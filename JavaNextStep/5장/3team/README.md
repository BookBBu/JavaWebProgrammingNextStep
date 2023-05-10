# 📂 5장 웹 서버 리팩토링, 서블릿 컨테이너와 서블릿의 관계

> ** 설계는 필요하지만 설계를 한 번만 해야한다는 생각을 가질 필요는 없다 **

설계는 한 번의 작업으로 끝내야 하는 것이 아니라 애플리케이션을 개발하고 배포해 운영하는 동안 끊임없이 진행해야 한다.
리팩토링은 설계를 개선하기 위한 일련의 활동이다.

## 5.1 HTTP 웹 서버 리팩토링 실습

### 5.1.1 리팩토링할 부분 찾기 

리팩토링이 필요한 시점과 종료해야 하는 시점을 판단하는 능력이 중요하다. 
    - 코드를 가리지 말고 다른 개발자가 구현한 많은 코드를 읽는다.
    - 소스코드를 직접 구현한다.

### 5.1.2 리팩토링 1단계 힌트
```
<힌트>
 - 클라이언트 요청 데이터를 담고 있는 InputStream을 생성자로 받아 HTTP 메소드, URL, 헤더, 본문을 분리하는 작업을 한다.
 - 헤더는 Map<String, String>에 저장해 관리하고 getHeader("필드 이름") 메소드를 통해 접근 가능하도록 구현한다.
 - GET과 POST 메소드에 따라 전달되는 인자를 Map<String, String>에 저장해 관리하고 geParageter("인자 이름") 메소드를 통해 접근 가능하도록 구현한다.
```
#### 5.1.2.1 요청 데이터를 처리하는 로직을 별도의 클래스로 분리한다(HttpRequest)
 Http_GET.txt, Http_POST.txt라는 이름으로 요청 데이터를 담고 있는 테스트 파일을 추가하여 테스트 코드를 기반으로 개발할 수 있다.

GET.txt
```
GET /user/create?userId=javajigi&password=password&name=JaeSung HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Accept: */*

```
> 마지막 라인에 빈 공백 문자열을 포함해야 한다.

POST.txt
```
POST /user/create HTTP/1.1
Host: localhost:8080
Connection: keep-alive
Content-Length: 46
Content-Type: application/x-www-form-urlencoded
Accept: */*

userId=javajigi&password=password&name=JaeSung
```

```java
public class HttpRequestTest {
	private String testDirectory = "./src/test/resources/";

	@Test
	public void request_GET() throws Exception {
		InputStream in = new FileInputStream(new File(testDirectory + "Http_GET.txt"));
		HttpRequest request = new HttpRequest(in);

		assertEquals("GET", request.getMethod());
		assertEquals("/user/create", request.getPath());
		assertEquals("keep-alive", request.getHeader("Connection"));
		assertEquals("javajigi", request.getParameter("userId"));
	}

	@Test
	public void request_POST() throws Exception {
		InputStream in = new FileInputStream(new File(testDirectory + "Http_POST.txt"));
		HttpRequest request = new HttpRequest(in);

		assertEquals("POST", request.getMethod());
		assertEquals("/user/create", request.getPath());
		assertEquals("keep-alive", request.getHeader("Connection"));
		assertEquals("javajigi", request.getParameter("userId"));
	}
}
```
GET과 POST에 대한 테스트 코드를 만족하는 HttpRequest 코드를 구현하면 된다. 

#### 5.1.2.2 응답 데이터를 처리하는 로직을 별도의 클래스로 분리한다(HttpResponse)
```
<힌트>
 - RequestHandler 클래스를 보면 응답 데이터 처리를 위한 많은 중복이 있다. 이 중복을 제거해 본다.
 - 응답 헤더 정보를 Map<String, String>으로 관리한다.
 - 응답을 보낼 때 HTML, CSS, 자바스크립트 파일을 직접 읽어 응답으로 보내는 메소드는 forward(). 다른 URL로 리다이렉트하는 메소드는 sendRedirect() 메소드를 나누어 구현한다.
```
요청 데이터에 대한 처리를 담당하고 있는 HttpResponse가 정상적으로 동작하는지 아래의 HttpResponseTest를 통해 확인할 수 있다.

```java
public class HttpResponseTest {
	private String testDirectory = "./src/test/resources/";

	@Test
	public void responseForward() throws Exception {
        // Http_Forward.txt 결과는 응답 body에 index.html이 포함되어 있어야 한다.
		HttpResponse response = new HttpResponse(createOutputStream("Http_Forward.txt"));
		response.forward("/index.html");
	}

	@Test
	public void responseRedirect() throws Exception {
        // Http_Redirect.txt결과는 응답 header에 
        // Location 정보가 /index.html로 포함되어 있어야 한다.
		HttpResponse response = new HttpResponse(createOutputStream("Http_Redirect.txt"));
		response.sendRedirect("/index.html");
	}

	@Test
	public void responseCookies() throws Exception {
        // Http_Cookie.txt 결과는 응답 header에 Set-Cookie 값으로 
        // logined=true 값이 포함되어 있어야 한다.
		HttpResponse response = new HttpResponse(createOutputStream("Http_Cookie.txt"));
		response.addHeader("Set-Cookie", "logined=true");
		response.sendRedirect("/index.html");
	}

	private OutputStream createOutputStream(String filename) throws FileNotFoundException {
		return new FileOutputStream(new File(testDirectory + filename));
	}
}
```
#### 5.1.2.3 다형성을 활용해 클라이언트 요청 URL에 대한 분기 처리를 제거한다.
기능이 많아질수록 분기문의 수는 증가한다. 자바의 **다형성** 을 활용해  **분기문을 제거** 한다. 
→ 인터페이스와 상속을 활용하자!

---

## 5.2 웹 서버 리팩토링 구현 및 설명
4장에서 구현한 RequestHandler 클래스는, 
- 클라이언트 요청에 대한 헤더와 본문 처리 
- 클라이언트 요청에 따른 로직 처리(회원가입, 로그인 등) 
- 로직 처리 완료 후 클라이언트에 대한 응답 헤더와 본문 데이터 처리 작업 
을 수행한다. 
클래스 하나가 너무 많은 일을 하기 때문에 

**`⭐한 객체가 한 가지 책임⭐`** 을 가지도록 리팩토링 진행 할 것이다.

먼저, 클라이언트 요청 데이터와 응답 데이터 처리를 별도의 클래스로 분리한다.

### 5.2.1 요청 데이터를 처리하는 로직을 별도의 클래스로 분리한다.
> 데이터 파싱하는 작업 => HttpRequest(추가), 사용하는 부분 => RequestHandler(수정)

**HttpRequest**
: (1)InputStream을 생성자의 인자로 받고 (2)InputStream에 담겨있는 데이터를 필요한 형태로 분리한 후, (3)객체의 필드에 저장하는 역할만 한다.
: 4가지 종류의 get()메소드 ```getMethod(), getPath(), getHeader(), getParameter()``` 를 통해 값에 접근할 수 있도록한다. (코드: p.154-156 참고)
    
하지만, 리팩토링 후에도 여전히 요청 라인(request line)을 처리하는 메소드(processRequestLine())의 복잡도가 높아 보인다.

(해결방법)
1. private 접근 제어자인 메소드를 default 접근 제어자로 수정하고 메소드 처리 결과를 반환하도록 수정
2. **메소드 구현 로직을 새로운 클래스로 분리 (V)**


[ ❓Private 메소드를 테스트하는 방법과 이를 지양해야 하는 이유 ]
1. Private 메소드를 테스트하는 방법
- 방법 1 : 테스트하고 싶은 메서드의 접근 제어자 범위(package-private, protected)를 바꾸어 테스트한다.
- 방법 2: 자바의 리플렉션을 활용하여 강제적으로 private메서드를 호출한다. 
	- 2.1. PowerMock의 Whitebox.invokeMethod
	- 2.2. Junit Addons의 PrivateAccessor 
	- 2.3. 직접 리플렉션 사용
2. Private 메소드 테스트를 지양해야 하는 이유
	- private 메소드에 대한 테스트는 깨지 쉬운 테스트가 된다. 
	- private 메소드는 내부를 감추어 클라이언트와의 결합도를 낮춰주는데, 클라이언트인 테스트 클래스가 내부 메소드를 알고 있으니 결합도가 높아진다. 그리고 이는 유지보수할 때 테스트에 대한 비용을 증가시키는 요인이 될 수 있는데, 메소드 이름이나 파라미터 등을 변경할 때 실패하게 된다. 또한 리플렉션 자체 역시 컴파일 에러를 유발하지 못하므로 최대한 사용을 자제해야 한다. 
- [private 메서드도 테스트를 해야 할까? (private 메서드 테스트 하고 싶을 때...)](https://kukim.tistory.com/84?category=908201)
- [[Java] Private 메소드를 테스트하는 방법과 이를 지양해야 하는 이유](https://mangkyu.tistory.com/235)


🛠️방법2에 따라서 **```RequestLine```** 라는 새로운 클래스를 추가하여 리팩토링!
- 결과) 요청라인을 처리하는 책임을 분리했지만 **HttpRequest의 메소드 원형은 바뀌지 않았다.** 따라서 기존의 HttpRequestTest도 변경없이 테스트할 수 있다.
```java
public class RequestLine {
	private static final Logger log = LoggerFactory.getLogger(RequestLine.class);
	
	private HttpMethod method;
	private String path;
	private Map<String, String> params = new HashMap<>();
	
	public RequestLine(String requestLine) {
		log.debug("request line : {}", requestLine);
		String[] tokens = requestLine.split(" ");
		
		if (tokens.length != 3) {
			throw new IllegalArgumentException(requestLine + "이 형식에 맞지 않습니다.");
		}
		
		method = HttpMethod.valueOf(tokens[0]);
		
		if (method.isPost()) {
			path = tokens[1];
			return;
		}
		
		int index = tokens[1].indexOf("?");
		if (index == -1) {
			path = tokens[1];
		} else {
			path = tokens[1].substring(0, index);
			params = HttpRequestUtils.parseQueryString(tokens[1].substring(index+1));
		}
	}
	
	public HttpMethod getMethod() {
		return method;
	}
	
	public boolean isPost() {
		return method.isPost();
	}
	
	public String getPath() {
		return path;
	}
	
	public Map<String, String> getParams() {
		return params;
	}
}
```
🛠️추가 리팩토링) **```HttpMethod```** 라는 이름의 enum으로 추가하는 리팩토링!
- 상수 값이 서로 연관되어 있는 경우, 자바의 ```enum```을 쓰기 적합하다. 따라서 String으로 구현되어 있던 method 필드(GET, POST)를 HttpMethod의 enum을 사용하도록 변경한다. => ```HttpRequest```, ```RequestLineTest```, ```HttpRequestTest```에서도 HttpMethod를 사용하도록 변경
```java
public enum HttpMethod {
	GET,
	POST;
	
	public boolean isPost() {
		return this == POST;
	}
}
```
🛠️ **```RequestHandler```** 에서 ```HttpRequest```를 사용하도록 리팩토링!
- 클라이언트 요청 데이터에 대한 처리를 모두 ```HttpRequest```로 위임했기 때문에 ```RequestHandler```는 요청 데이터를 처리하는 모든 로직을 제거할 수 있었다.

**<테스트 코드를 기반으로 개발할 경우 효과>**
```
1. 클래스에 버그가 있는지 빨리 찾아 구현할 수 있다.
2. 디버깅을 좀 더 쉽고 빠르게 할 수 있으며, 개발 생산성을 높여준다.
3. 리팩토링을 마음 놓고 할 수 있다.
```
### 5.2.2 응답 데이터를 처리하는 로직을 별도의 클래스로 분리한다.
> 응답 데이터 처리하는 작업 => HttpResponse(추가)

🛠️ HttpResponse 요구사항
- 응답 데이터의 상태에 따라 적절한 HTTP 헤더를 처리
- HTML, CSS, 자바스크립트 파일을 읽어 반환
- 302 상태 코드를 처리
- 쿠키 추가와 같이 HTTP헤더에 임의의 값을 추가

- HttpResponse 테스트: 2단계 힌트에서 제시한 HttpResponseTest를 실행해 생성된 파일을 통해 수동으로 테스트 가능

🛠️ **```RequestHandler```** 에서 ```HttpResponse```를 사용하도록 리팩토링!
```java
public class RequestHandler extends Thread {
    [...]

    public RequestHandler(Socket connectionSocket) {
        this.connection = connectionSocket;
    }

    public void run() {
        log.debug("New Client Connect! Connected IP : {}, Port : {}", connection.getInetAddress(),connection.getPort());

        try (InputStream in = connection.getInputStream(); OutputStream out = connection.getOutputStream()) {
        	HttpRequest request = new HttpRequest(in);
        	HttpResponse response = new HttpResponse(out);

        	Controller controller = RequestMapping.getController(request.getPath());
        	
        	if (controller == null) {
        		String path = getDefaultPath(request.getPath());
        		response.forward(path);
        	} else {
        		controller.service(request, response);
        	}
        		
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }

    private String getDefaultPath(String path) {
    	if (path.equals("/")) {
    		return "/index.html";
    	}
    	
    	return path;
    }
}
```

### 5.2.3 다형성을 활용해 클라이언트 요청 URL에 대한 분기 처리를 제거한다.
🛠️ RequestHandler의 run() 메소드의 복잡도를 더 낮추기 위해 **각 분기문 구현을 별도의 메소드로 분리** 하는 리팩토링
- 결과) 메소드 원형이 같기 때문에 자바의 인터페이스로 추출하는 것이 가능 => Controller 인터페이스 추가
```java
import http.HttpRequest;
import http.HttpResponse;

public interface Controller {
    void service(HttpRequest request, HttpResponse response);
}
```

🛠️ Controller 인터페이스에 대한 구현 클래스 추가
- 위에서 run()함수 안의 분기문에서 분리했던 메소드(createUser, login, listUser) 구현 코드를 Controller인터페이스에 대한 구현 클래스로 이동(CreateUserController, LoginController, ListUserController)
```java
public class CreateUserController extends Controller{
    private static final Logger log = LoggerFactory.getLogger(CreateUserController.class);
        
    @Override
    public void service(HttpRequest request, HttpResponse response) {
        User user = new User(
                request.getParameter("userId"),
                request.getParameter("password"),
                request.getParameter("name"),
                request.getParameter("email"));
        log.debug("User : {}", user);
        DataBase.addUser(user);
        response.sendRedirect("/index.html");   	
    }
}
```

🛠️ RequestMapping 클래스 추가
- 각 요청 URL과 URL에 대응하는 Controller를 연결하는 역할
- 웹 애플리케이션에서 서비스하는 모든 URL과 Controller를 관리
- 요청 URL에 해당하는 Controller를 반환
- RequestHandler 에서 요청 URL에 대한 Controller를 찾은 후 모든 작업을 해당 Controller가 처리하도록 위임
```java
public class RequestMapping {
    private static Map<String, Controller> controllers = new HashMap<>();
        
    static {
        controllers.put("/user/create", new CreateUserController());
        controllers.put("/user/login", new LoginController());
        controllers.put("/user/list", new ListUserController());
        }
        
    public static Controller getController(String requestUrl) {            
        return controllers.get(requestUrl);
    }
}
```

```
Controller controller = RequestMapping.getController(request.getPath());
if(controller == null) {
    String path = getDefaultPath(request.getPath());
    response.forward(path);
} else {
    controller.service(request, response);
}
```

🛠️추가 리팩토링) AbstractController 추상클래스 추가
- HTTP 메소드(GET, POST)에 따라 다른 처리를 할 수 있도록 추상 클래스를 추가
    - 각 Controller는 Controller 인터페이스를 직접 구현하는 것이 아닌 AbstractController를 상속해 각 HTTP 메소드에 맞는 메소드를 오버라이드 하도록 구현

```java
public class AbstractController implements Controller{
    @Override
	public void service(HttpRequest request, HttpResponse response) {
		HttpMethod method = request.getMethod();
		if (method.isPost()) {
            doPost(request, response);
        } else {
            doGet(request, response);
        }
    }
	
	public void doPost(HttpRequest request, HttpResponse response) {
	}
	
	public void doGet(HttpRequest request, HttpResponse response) {
	}
}
```
```java
public class CreateUserController extends AbstractController {
    @Override
    public void doPost(HttpRequest request, HttpResponse response) {
        [...]
    }
}
```
```java
public class ListUserController extends AbstractController {
    @Override
    public void doGet(HttpRequest request, HttpResponse response) {
        [...]
    }
}
```
해당 리팩토링시 장점
- ```요청 URL이 같더라도```, **```HTTP메소드가 다른 경우```** Controller하나로 GET(doGet메소드), POST(doPost메소드)를 모두 지원할 수 있다.

### 5.2.4 HTTP 웹 서버의 문제점
지금까지 구현한 웹 서버의 대표적인 문제점 3가지
1. HTTP 요청과 응답 헤더, 본문 처리와 같은 데 시간을 투자함으로써 정작 중요한 **로직을 구현하는데 투자할 시간이 상대적으로 적다.**
2. **동적인 HTML을 지원하는 데 한계가 있다.**. 동적으로 HTML을 생성할 수 있지만 많은 코딩량을 필요로 한다.
3. 사용자가 입력한 데이터가 서버를 재시작하면 사라진다. 즉, 사용자가 입력한 데이터를 유지할 수 없다.

---

## 5.3 서블릿 컨테이너, 서블릿/JSP를 활용한 문제 해결
> 자바 진영에서 표준으로 정한 것 : ```서블릿 컨테이너```, ```서블릿/JSP```
 - 🔎 **서블릿** : 위에서 구현한 웹서버의 Controller, HttpRequest, HttpResponse를 추상화해 인터페이스로 정의해놓은 표준이다. 즉, **HTTP의 클라이언트 요청과 응답에 대한 표준**이다.
 - 🔎 **서블릿 컨테이너** : 서블릿 표준에 대한 구현을 담당한다. (지금까지 구현한 웹 서버가 서블릿 컨테이너 역할과 같다고 생각하면 됨)

 ```
 [서블릿 컨테이너의 라이프 사이클 - 서블릿 컨테이너가 시작하고 종료할 때 까지의 과정]

 (서블릿 생성 및 초기화)
 1. 서블릿 컨테이너 시작
 2.클래스 패스에 있는 Servlet 인터페이스를 구현하는 서블릿 클래스를 찾음
 3. @WebServlet 설정을 통해 요청 URL과 서블릿 매핑
 4. 서블릿 인스턴스 생성
 5. init()메소드를 호출해 초기화

 (서블릿 서비스)
 이후 클라이언트 요청까지 대기. 클라이언트 요청이 있을 경우,
 6. URL에 해당하는 서블릿을 찾아 service()메소드를 호출

 (서블릿 소멸)
 7. 서비스 도중에 서블릿 컨테이너를 종료하면 서블릿 컨테이너가 관리하고 있는 모든 서블릿의 destroy()메소드를 호출
 ```
이 외에도 멀티쓰레딩 지원, 설정 파일을 활용한 보안관리, JSP 지원 등의 작업을 지원한다.

 - 🔎 **컨테이너** : 각 컨테이너마다 다른 기능을 지원할 수 있지만, 기본적으로 ```생명주기를 관리```하는 기능을 제공한다.
    - 예) EJB 컨테이너, Bean 컨테이너
    - 컨테이너에 의해 인스턴스가 관리되기 때문에 초기화, 소멸과 같은 작업을 위한 메소드를 인터페이스 규약으로 만들어 놓고 확장할 수 있도록 지원
    - 서블릿 컨테이너는 ```멀티스레드```로 동작 => 그러면 새로운 스레드가 생성될 때마다 새로운 서블릿 인스턴스를 생성할까? no. 서블릿 컨테이너가 시작할 때 한번 생성되면 모든 스레드가 같은 인스턴스를 재사용한다. 즉, ```멀티스레드가 하나의 인스턴스를 공유```한다.

## 5.4 추가 학습 자료
### 5.4.1 📖 객체지향 설계와 개발 
- "객체지향의 사실과 오해"(조영호 저, 위키북스/2015)
- "개발자가 반드시 정복해야 할 객체지향과 디자인 패턴"(최범균 저, 인투북스/2013)

### 5.4.2 서블릿 & JSP, 웹 애플리케이션 서버
- "Head First Servlet & JSP"(케이시 시에라, 버트 베이츠, 브라얀 바샴 저/김종호 역, 한빛미디어/2009)
- "프로가 되기 위한 웹기술 입문"(고모라 유스케 저/김정환 역, 위키북스/2012)5장

### 5.4.3 템플릿 엔진
JSP와 템플릿 엔진의 역할은 같다. 최근 동적으로 HTML을 생성하기 위해 JSP를 사용하는 대신 ```템플릿 엔진```을 사용한다.

→ 모바일과 같은 다양한 기기의 등장으로 웹 백엔드는 JSON/XML과 같은 데이터만 제공하고, 동적인 웹 UI는 클라이언트가 담당하는 방향으로 변화
- [[Web] 템플릿 엔진, JSP, Thymeleaf란? 서버 사이드 템플릿 엔진 vs 클라이언트 사이드 템플릿 엔진](https://code-lab1.tistory.com/211)
