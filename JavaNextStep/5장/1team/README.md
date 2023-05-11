# 5장 웹 서버 리팩토링, 서블릿 컨테이너와 서블릿의 관계

# 5-1 HTTP 웹 서버 리팩토링 실습

## 5.1.1 리팩토링할 부분 찾기

- 나쁜 냄새가 나는 코드를 찾는 능력 키우기
- 리팩토링이 필요한 시점에 대한 정확한 기준을 세우기 보다 직관에 의존하기
- 직관을 키우려면 다른 개발자가 구현한 코드를 많이 많이 읽어보고 구현해보고 연습하기

## 5.1.2 리팩토링 1단계 힌트

### 5.1.2.1 요청 데이터 처리 로직을 별도의 클래스로 분리(HttpRequest)

- 클라이언트 요청 데이터를 담은 InputStream을 생성자로 받아 HTTP 메소드, URL, 헤더, 본문 분리
- 헤더는 Map에 저장하고 관리
- GET/POST에 따라 전달되는 파라미터도 Map에 저장하고 관리
- getHeader(”필드이름”), getParameter(”인자이름”)으로 접근 가능하게 구현

### 5.1.2.2 응답 데이터 처리 로직을 별도의 클래스로 분리(HttpResponse)

- 응답 데이터 처리 코드 중복 제거
- 응답 헤더 정보 Map으로 관리
- 응답을 보낼 때 HTML, CSS, JS 파일을 직접 읽어 응답으로 보내는 메소드는 forward(), 다른 URL로 리다이렉트하는 메소드는 sendRedirect()로 분리해 구현

### 5.1.2.3 다형성을 활용해 클라이언트 요청 URL에 대한 분기 처리 제거

- 각 요청과 응답에 대한 처리를 담당하는 부분을 인터페이스로 만들기

```java
public interface Controller{
	void service(HttpRequest request, HttpResponse response);
}
```

- controller 인터페이스를 구현하는(implements) 클래스를 만들어 분리
- 이렇게 생성한 Controller 구현체를 Map<String, Controller>에 저장(String은 요청URL)
- Controller 인터페이스를 구현하는 추상클래스를 추가해 중복 제거
- service() 메소드에서 GET과 POST에 따라 doGet(), doPost() 호출

# 5.2 웹 서버 리팩토링 구현 및 설명

## 5.2.1 요청 데이터를 처리하는 로직을 별도의 클래스로 분리

- HttpRequest의 책임: 클라이언트 요청 데이터를 읽은 후, 각 데이터를 사용하기 좋은 형태로 분리

```java
public HttpRequest(InputStream in) {
		try {
			BufferedReader br = new BufferedReader(new InputStreamReader(in, "UTF-8"));
			String line = br.readLine();
			if(line == null) {
				return;
			}
			
			processRequestLine(line);
			
			line = br.readLine();
			while(!line.equals("")) {
				log.debug("header : {}", line);
				String[] tokens = line.split(":");
				headers.put(tokens[0].trim(), tokens[1].trim());
				line = br.readLine();
			}
			
			if ("POST".equals(method)) {
				String body = IOUtils.readData(br, Integer.parseInt(headers.get("Content-Length")));
				params = HttpRequestUtils.parseQueryString(body);
			}
		} catch (IOException e) {
			log.debug(e.getMessage());
		}
	}
	
	private void processRequestLine(String requestLine) {
		log.debug("request line : {}", requestLine);
		String[] tokens = requestLine.split(" ");
		// method 종류: GET or POST
		method = tokens[0];
		
		// POST
		if("POST".equals(method)) {
			path = tokens[1];
			return;
		}
		
		// GET
		int index = tokens[1].indexOf("?");
		if(index==-1) {
			path = tokens[1];
		}else {
			path = tokens[1].substring(0, index);
			params = HttpRequestUtils.parseQueryString(tokens[1].substring(index+1));
		}
	}
	
// 저장한 값에 접근할 수 있도록 하는 get 메소 
	public String getMethod() {
		return method;
	}
	
	public String getPath() {
		return path;
	}
	public String getHeader(String name) {
		return headers.get(name);
	}
	
	public String getParameter(String name) {
		return params.get(name);
	}
```

- processRequestLine() 메소드의 복잡도를 줄이는 방법, 테스트할 수 있도록 하는 방법
    1. private 접근 제어자인 메소드를 default 접근 제어자로 수정, 메소드 처리 결과를 반환하도록 수정하기 ⇒ 반환해야하는 상태 값이 여러 개라 부적합
    2. 메소드 구현 로직을 새로운 클래스로 분리하기 ⇒ 얘로 리팩토링 해보자

```java
public RequestLine(String requestLine) {
		log.debug("request line : {}", requestLine);
		String[] tokens = requestLine.split(" ");
		
		if(tokens.length !=3) {
			throw new IllegalArgumentException(requestLine +"이 형식에 맞지 않습니다.");
		}
		
		// method 종류: GET or POST
		method = tokens[0];
		
		// POST
		if("POST".equals(method)) {
			path = tokens[1];
			return;
		}
		
		// GET
		int index = tokens[1].indexOf("?");
		if(index==-1) {
			path = tokens[1];
		}else {
			path = tokens[1].substring(0, index);
			params = HttpRequestUtils.parseQueryString(tokens[1].substring(index+1));
		}
	}
```

- GET, POST 문자열 하드코딩 문제 ⇒ enum 사용하도록 리팩토링
- 하드코딩: 데이터를 코드에 직접 입력함
- 하드코딩의 장단점에 대해 이야기 해보장

```java
public enum HttpMethod {
	GET,
	POST;
	public boolean isPost() {
		return this == POST;
	}
}
```

## 5.2.2 응답 데이터를 처리하는 로직을 별도의 클래스로 분리

- 응답 데이터를 처리하는 책임을 HttpResponse로 위임해서 RequestHandler를 깔끔하게 만들기

```java
public class HttpResponse {
	private static final Logger log = LoggerFactory.getLogger(HttpResponse.class);
	private DataOutputStream dos = null;
	private Map<String, String> headers = new HashMap<String, String>();

	public HttpResponse(OutputStream out) {
		dos = new DataOutputStream(out);
	}
	
	public void addHeader(String key, String value) {
		headers.put(key, value);
	}
	
	public void forward(String url) {
		try {
			byte[] body = Files.readAllBytes(new File("./webapp" + url).toPath());
			if (url.endsWith(".css")) {
				headers.put("Content-Type", "text/css");
			}else if(url.endsWith(".js")) {
				headers.put("Content-Type", "application/javascript");
			}else {
				headers.put("Content-Type", "text/html;charset=utf-8");
			}
			headers.put("Content-Length", body.length + "");
			response200Header(body.length);
			responseBody(body);
		}catch(IOException e) {
			log.error(e.getMessage());
		}
	}
	
	public void forwardBody(String body) {
		byte[] contents = body.getBytes();
		headers.put("Content-Type", "text/html;charset=utf-8");
		headers.put("Content-Length", contents.length + "");
		response200Header(contents.length);
		responseBody(contents);
	}
	
	public void sendRedirect(String redirectUrl) {
		try {
    		dos.writeBytes("HTTP/1.1 302 Found \r\n");
    		processHeaders();
    		dos.writeBytes("Location: " + redirectUrl + "\r\n");
    		dos.writeBytes("\r\n");
		} catch (IOException e) {
			e.getMessage();
		}
	}
	
// RequestHandler에 있던 private 메소드들
    private void response200Header(int lengthOfBodyContent) {
        try {
            dos.writeBytes("HTTP/1.1 200 OK \r\n");
            processHeaders();
            dos.writeBytes("\r\n");
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }

    private void responseBody(byte[] body) {
        try {
            dos.write(body, 0, body.length);
            dos.writeBytes("\r\n");
            dos.flush();
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }
    
    private void processHeaders() {
    	try {
			Set<String> keys = headers.keySet();
			for(String key: keys) {
				dos.writeBytes(key + ": " + headers.get(keys) + "\r\n");
			}
			} catch (IOException e) {
				e.getMessage();
			}
    }
}
```

- private 메소드 테스트를 지양해야할 이유에 대해 생각해보장

## 5.2.3 다형성을 활용해 클라이언트 요청 URL에 대한 분기 처리 제거

- run() 메소드의 문제점 : 기능이 추가될 때마다 새로운 else if 절이 추가되는 구조로 구현되어 있음
- 가상폐쇄의 원칙(Open-Closed Principle)을 위반하는 코드 ..
    
    (가상폐쇄의 원칙: 요구사항의 변경이나 추가사항이 발생하더라도 기존 구성요소는 수정이 일어나지 말아야 하며, 기존 구성요소를 쉽게 확장해서 재사용할 수 있어야 한다)
    
- 각 분기문 구현을 별도의 메소드로 분리하도록 리팩토링 해보자 !

```java
// 리팩토링 전 run() 메소드
public void run() {
        log.debug("New Client Connect! Connected IP : {}, Port : {}", connection.getInetAddress(), connection.getPort());

        try (InputStream in = connection.getInputStream(); OutputStream out = connection.getOutputStream()) {
        	HttpRequest request = new HttpRequest(in);
        	HttpResponse response = new HttpResponse(out);
        	String path = getDefaultPath(request.getPath());
        	
            if("/user/create".equals(path)) {
            	User user = new User(	request.getParameter("userId"), request.getParameter("password"), 
            							request.getParameter("name"), request.getParameter("email"));
            	log.debug("User : {}", user);
            	DataBase.addUser(user);
            	response.sendRedirect("/index.html");
            }else if("/user/login".equals(path)) {
            	User user = DataBase.findUserById(request.getParameter("userId"));
            	if(user != null) {
                	if(user.login(request.getParameter("password"))) {
                		response.addHeader("Set-Cookie", "logined=true");
                		response.sendRedirect("/index.html");
                	}else {
                		response.sendRedirect("/user/login_failed.html");
                	}
            	}else {
            		response.sendRedirect("/user/login_failed.html");
            	}
            }else if("/user/list".equals(path)) {
            	if(!isLogin(request.getHeader("Cookie"))) {
            		response.sendRedirect("/user/login.html");
            		return;
            	}
            	
            	Collection<User> users = DataBase.findAll();
            	StringBuilder sb = new StringBuilder();
            	for(User user: users) {
            		sb.append("<tr>");
            		sb.append("<td>" + user.getUserId() + "</td>");
            		sb.append("<td>" + user.getName() + "</td>");
            		sb.append("<td>" + user.getEmail() + "</td>");
            		sb.append("</tr>");
            	}
            	response.forwardBody(sb.toString());
            }else {
            	response.forward(path);
            }
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }
```

```java

// 리팩토링 후
private void listUser(HttpRequest request, HttpResponse response) {
	if(!isLogin(request.getHeader("Cookie"))) {
		response.sendRedirect("/user/login.html");
    return;
	}
    	
  Collection<User> users = DataBase.findAll();
  StringBuilder sb = new StringBuilder();
  for(User user: users) {
	  sb.append("<tr>");
    sb.append("<td>" + user.getUserId() + "</td>");
    sb.append("<td>" + user.getName() + "</td>");
    sb.append("<td>" + user.getEmail() + "</td>");
    sb.append("</tr>");
	}
  response.forwardBody(sb.toString());
}

private void login(HttpRequest request, HttpResponse response) {
	User user = DataBase.findUserById(request.getParameter("userId"));
  if(user != null) {
	  if(user.login(request.getParameter("password"))) {
	    response.addHeader("Set-Cookie", "logined=true");
	    response.sendRedirect("/index.html");
		}else {
			response.sendRedirect("/user/login_failed.html");
	}else {
	  response.sendRedirect("/user/login_failed.html");
  }
}
private void createUser(HttpRequest request, HttpResponse response) {
	User user = new User(	request.getParameter("userId"), request.getParameter("password"), 
				request.getParameter("name"), request.getParameter("email"));
	log.debug("User : {}", user);
	DataBase.addUser(user);
	response.sendRedirect("/index.html");
}
  
```

- 메소드의 원형이 같기 때문에 자바의 인터페이스로 추출하는 것이 가능 ! ⇒ Controller 인터페이스 추가

```java
import http.HttpRequest;
import http.HttpResponse;

public interface Controller {
	void service(HttpRequest request, HttpResponse response);
}
```

- Controller 인터페이스에 대한 구현 클래스 추가

```java
public class CreateUserController implements Controller{
	private static final Logger log = LoggerFactory.getLogger(CreateUserController.class);

	@Override
	public void service(HttpRequest request, HttpResponse response) {
    	User user = new User(	request.getParameter("userId"), request.getParameter("password"), 
				request.getParameter("name"), request.getParameter("email"));
		log.debug("User : {}", user);
		DataBase.addUser(user);
		response.sendRedirect("/index.html");
	}
}
```

- 각 요청 URL에 해당하는 Controller( CreateUserController, LoginController, ListUserController .. )를 반환하 클래스 ⇒ RequestMapping 클래스 추가

```java
public class RequestMapping {
	private static Map<String, Controller> controllers = new HashMap<String, Controller>();
	
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

- 리팩토링 완료 후 run() 메소드
- 새로운 기능이 추가되어도 run() 메소드는 수정할 필요가 없고, Controller 인터페이스를 구현하는 새로운 클래스를 추가하고 RequestMapping의 Map에 요청 URL과 Controller 클래스를 추가하기만 하면 된다
- 변경 사항이 발생하면 Controller 클래스의 service() 메소드만 수정하면 된다

```java
public void run() {
        log.debug("New Client Connect! Connected IP : {}, Port : {}", connection.getInetAddress(), connection.getPort());

        try (InputStream in = connection.getInputStream(); OutputStream out = connection.getOutputStream()) {
        	HttpRequest request = new HttpRequest(in);
        	HttpResponse response = new HttpResponse(out);
        	
        	Controller controller = RequestMapping.getController(request.getPath());
        	if(controller==null) {
        		String path = getDefaultPath(request.getPath());
        		response.forward(path);
        	}else {
        		controller.service(request, response);
        	}
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }
```

- HTTP 메소드(GET/POST)에 따라 다른 처리를 할 수 있도록 추상 클래스 추가
- 각 Controller가 Controller 인터페이스를 직접 구현하는 것이 아닌 AbstractController를 상속해 각 HTTP 메소드에 맞는 메소드를 오버라이드하도록 구현
- 요청 URL이 같더라도 HTTP 메소드가 다른 경우 새로운 Controller를 추가하지 않고 GET과 POST 모두 지원 가능

```java
public abstract class AbstractController implements Controller {
	@Override
	public void service(HttpRequest request, HttpResponse response) {
		HttpMethod method = request.getMethod();
		
		if(method.isPost()) {
			doPost(request, response);
		}else{
			doGet(request, response);
		}
	}
	
	protected void doPost(HttpRequest request, HttpResponse response) {
		
	}
	protected void doGet(HttpRequest request, HttpResponse response) {
		
	}
}
```

## 5.2.4 HTTP 웹 서버의 문제점

- HTTP 요청과 응답 헤더, 본문 처리 등에 시간을 투자함으로써 **로직 구현에 투자할 시간이 상대적으로 적음**
- 동적인 HTML을 지원하는 데 한계 존재, 동적으로 HTML을 생성하려면 많은 코딩량 필요
- 사용자가 입력한 데이터가 서버를 재시작하면 사라짐, 유지할 방법은 ?

# 5.3 서블릿 컨테이너, 서블릿/JSP를 활용한 문제 해결

- 서블릿: 앞서 구현한 Controller, HttpRequest, HttpResponse를 추상화해 인터페이스로 정의해 놓은 표준. 즉, HTTP의 클라이언트 요청과 응답에 대한 표준을 정해 놓은 것
- 서블릿 컨테이너: 서블릿 표준에 대한 구현을 담당, 앞서 구현한 웹 서버의 역할
- 서블릿 컨테이너와 서블릿의 동작 방식
    
    1) 서블릿 컨테이너는 서버가 시작할 때 서블릿 인스턴스를 생성하고, 요청 URL과 서블릿 인스턴스를 연결해 놓음
    
    2) 클라이언트에서 요청이 오면 요청 URL에 해당하는 서블릿을 찾아 서블릿에 모든 작업을 위임
    

## 5.3.1 개발 환경 세팅 및 Hello World 출력

- 서블릿을 지원하는 서블릿 컨테이너 구현체는 톰캣(Tomcat), Jetty, JBoss 등 여러가지가 있음
- 오픈소스이며 무료로 사용할 수 있는 서블릿 컨테이너 중 가장 널리 사용되는 톰캣 사용

## 5.3.2 서블릿 컨테이너, 서블릿

- 서블릿은 앞서 구현한 Controller와 정확히 같은 역할을 하고, 똑같은 방식으로 동작함
- 더 정확히 연결하면 Controller 인터페이스는 서블릿의 Servlet 인터페이스, AbstractController는 HttpServlet과 같음

```java
public interface Servlet{

	// 서블릿 클래스의 인스턴스 생성, 
	// 요청 URL과 서블릿 인스턴스 매핑, 
	// 클라이언트 요청에 해당하는 서블릿을 찾은 후 서블릿에 작업을 위임
	// 이외에도 서블릿과 관련한 초기화(init)와 소멸(destroy) 작업도 담당
	public void init(ServletConfig config) throws ServletException;

	public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException;

	public void destroy();
	
	public SevletConfig getServletConfig();

	public String getServletInfo();
} 
```

- 서블릿 컨테이너가 시작하고 종료할 때의 과정을 단계적으로 살펴보기
    
    1) 서블릿 컨테이너 시작
    
    2) 클래스패스에 있는 Servlet 인터페이스를 구현하는 서블릿 클래스를 찾음
    
    3) ‘@WebServlet 설정을 통해 요청 URL과 서블릿 매핑
    
    4) 서블릿 인스턴스 생성
    
    5) init() 메소드 호출해 초기화
    
- 이와 같이 서블릿 생성, 초기화, 서비스, 소멸 과정을 거치는 전체 과정을 서블릿의 생명주기(라이프 사이클, life cycle)이라고 함
- 이외에도 멀티쓰레딩 지원, 설정 파일을 활용한 보안관리, JSP 지원 등을 지원함으로써 개발자가 로직 구현에 집중할 수 있도록 함

# 5.4 추가 학습 자료

## 5.4.1 객체지향 설계와 개발

- “객체지향과 사실의 오해”(조영호 저, 위키북스/2015)
- “개발자가 반드시 정복해야 할 객체지향과 디자인 패턴”(최범균 저, 인투북스/2013)

## 5.4.2 서블릿&JSP, 웹 애플리케이션 서버

- "Head First Servlet & JSP"(케이시 시에라, 버트 베이츠, 브라얀 바샴 저/김종호 역, 한빛미디어/2009)
- "프로가 되기 위한 웹기술 입문"(고모라 유스케 저/김정환 역, 위키북스/2012)5장

## 5.4.3 템플릿 엔진

- 최근 동적으로 HTML을 생성하기 위해 JSP 대신 템플릿 엔진을 사용하는 것이 일반적임
- 모바일 등 다양한 기기의 등장으로 최근 웹 백엔드는 JSON/XML과 같은 데이터만 제공하고 동적인 웹 UI는 클라이언트가 담당하는 방향으로 변화 중 ..
- 이런 방향으로 변화할 수록 JSP에 대한 필요성은 낮아지고 템플릿 엔진의 필요성은 높아짐
