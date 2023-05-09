# 자뿌발표준비

작성일시: 2023년 5월 10일

## 📌5.1 HTTP 웹 서버 리팩토링 실습

### 5.1.1 리팩토링할 부분 찾기

---

리팩토링을 하는 데 있어 리팩토링을 어떻게 하느냐는 능력 보다는 리팩토링이 **필요한 시점와 종료해야 하는 시점을 판단**해야 하는 능력이 중요.

[마틴 파울러 저/김지원 역, 한빛미디어/2012년]

- 리팩토링시, 아래 두 가지 구별된 작업을 위해 시간을 나누어야 한다.
    - 기능 추가시, 기존 코드를 건드려서는 안되고 단지 새로운 기능만 추가해야한다.
        
        테스트를 추가하고, 테스트가 잘 작동하는 지를 확인함으로써 진행상황을 알 수 있다.
        
    - 리팩토링시 기능을 추가해서는 안되고, 단지 코드의 구조에만 집중,  테스트 케이스 추가하지 않는다.
- 리팩토링 시기
    - 삼진규칙 : 중복(비슷한) 된 것을 세번째로 하게 될 때
    - 기능을 추가할 때 : 수정해야할 코드에 대한 이해를 돕기 위해서, 기능 추가가 쉽지 않은 디자인인 경우
    - 버그를 수정해야할 때 : 버그가 있었다는 것을 몰랐을 정도로 코드가 명확하지 않았다는 뜻이므로
    - 코드 리뷰를 할 때

### 5.1.2 리팩토링

---

**5.1.2.1 요청 데이터를 처리하는 로직을 별도의 클래스로 분리한다. (HttpRequest)**

**5.1.2.2 응답 데이터를 처리하는 로직을 별도의 클래스로 분리한다(HttpResponse)**

**5.1.2.3 다형헝을 활용해 클라이언트 요청 URL에 대한 분기 처리를 제거한다.**

### **5.1.2.3 다형성을 활용해 클라이언트 요청 URL에 대한 분기 처리를 제거한다.**

---

메소드에 따른 실행을 모든 컨트롤러에게 위임

**[Controller interface의 service 메소드]**

```java
public interface Controller{
	void service(HttpRequest request, HttpResponse response) throws IOException;
}
```

**[AbstractController를 통한 다형성]**

```java
package controller;

import http.HttpMethod;
import http.HttpRequest;
import http.HttpResponse;

public abstract class AbstractController implements Controller {
    @Override
    public void service(HttpRequest request, HttpResponse response) {
        HttpMethod method = request.getMethod();

        if (method.isPost()) {
            doPost(request, response);
        } else {
            doGet(request, response);
        }
    }

    protected void doPost(HttpRequest request, HttpResponse response) {
    }

    protected void doGet(HttpRequest request, HttpResponse response) {
    }
}
```

- sevice를 구현한 추상 컨트롤러는 Http Method에 따른 실행을 위해 doGet(), doPost() 메소드를 가지고 있다. service 메소드 내부에서는 FrontController로 부터 전달받은  HttpRequest 객체를 d이용해 HttpMethod를 검사해 doGet, doPost를 실행한다.
- 컨트롤러의 service만 호출해도 각 컨트롤러의 상위 클래스인 AbstractController가 구현한 service에서 자동으로 Http Method를 검사하고 알맞은 메소드를 실행
- 호출된 doGet, doPost를 실행한 컨트롤러는 AbstractController를 구현한 Controller에서 반드시 구현하고 있으므로 해당 메소드를 실행

## 📌5.2 웹 서버 리팩토링 구현 및 설명

### 🔎 테스트 코드를 기반으로 개발할 경우의 장점

---

1. 클래스에 버그가 있는 지를 빨리 찾아 구현가능
    
    수동적으로 확인하는 과정을 거치면서 개발 가능
    
2. 디버깅 하기 쉽다.
    
    클래스에 대한 단위 테스트 → 디버깅을 쉽고 빠르게 함 → 개발의 생산성을 높여줌
    
3. 테스트 코드를 기반으로 리팩토링하기 쉽다
    
    리팩토링을 꺼리는 이유 중 하나는 지금까지 했던 테스트를 다시 반복해야한다는 것이 가장 큰 이유이다. 하지만 테스트코드가 이미 존대한다면 리팩토링 완료 후 실행하기만 하면 된다.
    

### 5.2.1 요청 데이터를 처리하는 로직을 별도의 클래스로 분리한다.

---

클라이언트 요청 데이터에서 요청 라인을 읽고, 헤더를 읽는 로직을 HttpRequest 클래스를 추가해 구현한다.

HttpRequest의 책임 : 클라이언트 요청 데이터를 읽은 후 각 데이터를 사용하기 좋은 형태로 분리하는 역할만 한다. → 사용은 RequestHandler만 한다.

➡ 데이터 파싱작업 (HttpRequest), 데이터 사용(RequestHandler)하는 부분을 나눈다.

```java
private void processRequestLine(String requestLine) {
		log.debug("request line : {}", requestLine);
		String[] tokens = requestLine.split("");
		
		if("POST".equals(method)) {
			path = tokens[1];
			return;
		}
		
		int index = tokens[1].indexOf("?");
		if(index==-1) {
			path = tokens[1];
		}else {
			path = tokens[1].substring(0,index);
			params = HttpRequestUtils.parseQueryString(tokens[1].substring(index+1));
		}
	}
```

**🤦‍♀️ private method는 복잡도가 높아 추가적인 테스트가 필요하지만 이 메소드만 분리해서 테스트 하기 힘들다**

### 🔎 **private method가 복잡도가 높은 이유**

---

1. Private 메소드는 클래스의 내부 구현 세부 사항에 대한 책임을 지기 때문입니다. 따라서, 해당 메소드는 클래스의 다른 부분과 강하게 결합될 수 있으며, 이로 인해 코드의 수정이 어려워질 수 있습니다.
2. Private 메소드는 다른 클래스에서 사용될 필요가 없으므로, 외부에서 테스트하기 어려울 수 있습니다. 따라서, 이러한 메소드를 테스트하는 데는 클래스의 다른 부분과 동일한 수준의 노력이 필요할 수 있습니다.
3. Private 메소드는 클래스의 책임을 분리하는 데 도움이 되지만, 이것은 클래스가 더 작고 단순하게 유지되도록 돕는 것이 아니라, 클래스의 구현 복잡도를 높이는 경향이 있습니다.

따라서, private 메소드의 복잡도를 낮추기 위해서는, 클래스의 구현을 단순화하고, private 메소드의 역할을 분리하여 클래스의 다른 부분과의 결합을 최소화해야 합니다. 또한, private 메소드에 대한 테스트 코드를 작성하고, 외부에서 테스트하기 쉽도록 만들어야 합니다.

**➕ 따로 분리해서 테스팅 하기 어려운 이유 : 클래스 외부에서 액세스할 수 없기 때문**

⇒ 해결방법에는 크게 2가지가 존재

1. private → defualt로 변경
2. 메소드 로직을 새로운 class로 분리

> **⁉ 그냥 defualt로 변경하면 안돼?**
클래서의 상태나 동작에 밀접하게 연결되어 있지 않으면 class로 분리하는 것이 코드의 재사용과 중복 제거 측면에서 더 바람직 하다!
> 

```java
package webserver;

import java.util.HashMap;
import java.util.Map;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import util.HttpRequestUtils;

public class RequestLine {
	private static final Logger log = LoggerFactory.getLogger(HttpRequest.class);

	private String method;
	private String path;
	private Map<String, String> headers = new HashMap<>();
	private Map<String, String> params = new HashMap<>();

	public RequestLine(String requestLine) {
		log.debug("request line : {}", requestLine);
		String[] tokens = requestLine.split(" ");
		if (tokens.length != 3) {
			throw new IllegalArgumentException(requestLine + "이 형식에 맞지 않습니다.");
		}
		method = tokens[0];
		if ("POST".equals(method)) {
			path = tokens[1];
			return;
		}

		int index = tokens[1].indexOf("?");
		if (index == -1) {
			path = tokens[1];
		} else {
			path = tokens[1].substring(0, index);
			params = HttpRequestUtils.parseQueryString(tokens[1].substring(index + 1));
		}
	}

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
}
```

**[문자열이 하드코딩 되어 있다 → 자바의 enum을 쓰자]**

자바의 enum은 상수들의 집합을 정희하는 데이터 유형, 특정 데이터 값만 포함하는 데이터 유형으로 상수들의 집합을 만들기 위해 좋은 방법

```java
public enum HttpMethod{
	GET,
	POST;
}
```

```java
package webserver;

import java.util.HashMap;
import java.util.Map;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import util.HttpRequestUtils;

public class RequestLine {
	private static final Logger log = LoggerFactory.getLogger(HttpRequest.class);

	**private HttpMethod method;**

	private String method;
	private String path;
	private Map<String, String> headers = new HashMap<>();
	private Map<String, String> params = new HashMap<>();

	public RequestLine(String requestLine) {
		log.debug("request line : {}", requestLine);
		String[] tokens = requestLine.split(" ");
		if (tokens.length != 3) {
			throw new IllegalArgumentException(requestLine + "이 형식에 맞지 않습니다.");
		}
		//method = tokens[0];
		**method = HttpMethod.valueOf(tokens[0]);**
		//if ("POST".equals(method)) {
			**if(method == HttpMethod.POST){**
			path = tokens[1];
			return;
		}

		int index = tokens[1].indexOf("?");
		if (index == -1) {
			path = tokens[1];
		} else {
			path = tokens[1].substring(0, index);
			params = HttpRequestUtils.parseQueryString(tokens[1].substring(index + 1));
		}
	}

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
}
```

```java
		**if(method == HttpMethod.POST){=> isPost() 로 수정 가능하다.**
			path = tokens[1];
			return;
		}
// 이 부분을 더 간결하게 할 수 없을까?
public enum HttpMethod{
	GET,
	POST;

	public boolean isPost(){
			return this==POST;
	}
}
```

### 5.2.2 응답 데이터를 처리하는 로직을 별도의 클래스로 분리한다.

---

1. HttpResponse 클래스를 추가

```java
package http;

import java.io.DataOutputStream;
import java.io.File;
import java.io.IOException;
import java.io.OutputStream;
import java.nio.file.Files;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

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
            } else if (url.endsWith(".js")) {
                headers.put("Content-Type", "application/javascript");
            } else {
                headers.put("Content-Type", "text/html;charset=utf-8");
            }
            headers.put("Content-Length", body.length + "");
            response200Header(body.length);
            responseBody(body);
        } catch (IOException e) {
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

    public void sendRedirect(String redirectUrl) {
        try {
            dos.writeBytes("HTTP/1.1 302 Found \r\n");
            processHeaders();
            dos.writeBytes("Location: " + redirectUrl + " \r\n");
            dos.writeBytes("\r\n");
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }

    private void processHeaders() {
        try {
            Set<String> keys = headers.keySet();
            for (String key : keys) {
                dos.writeBytes(key + ": " + headers.get(key) + " \r\n");
            }
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }
}
```

⇒ RequestHandler.class에서 요청을 처리하는 private method를 제거한다.

### 5.2.3 다형성을 활용해 클라이언트 요청 URL에 대한 분기 처리를 제거한다.

---

현재 RequestHandler.class 의 run( ) 메소드를 살펴보면  기능이 추가될 때마다 else if문이 추가되는 구조로 구현되어 있다는 것이다.

객체지향 설계 원칙 중 요구사항의 변경이나 추가사항이 발생하더라고, 기존 구성요소는 수정이 일어나지 말아야 하며, 기존 구성요소를 쉽게 재사용 해야한다는 개방폐쇠의 원칙을 위반하고 있다.

> 객체 지향 설계 원칙(SOLID 원칙)
1. SRP(Single Responsibility Principle): 클래스는 하나의 책임이나 할 일만 가져야 합니다.
2. OCP(Open/Closed Principle): 기존 코드를 수정하지 않고도 시스템의 동작을 확장할 수 있어야 합니다.
3. Liskov 대체 원칙(LSP): 클래스 계층 구조가 있는 경우 하위 클래스의 모든 인스턴스는 코드를 손상시키지 않고 상위 클래스의 인스턴스를 대체할 수 있어야 합니다.
4. ISP(Interface Segregation Principle): 클라이언트가 불필요한 메서드를 강제로 구현하지 않고 사용할 수 있는 작고 응집력 있는 인터페이스를 만들어야 합니다.
5.DIP(Dependency Inversion Principle): 고수준 모듈은 저수준 모듈에 의존해서는 안 됩니다. 대신 둘 다 추상화에 의존해야 합니다. 이 원칙은 모듈을 분리하여 유지 관리 및 수정을 더 쉽게 만드는 것입니다.
> 
1. **run ( )메소드의 복잡도가 높아, 각 분기문을 별도의 메소드로 분리하는 리팩토링 진행**

```java
public void run() {
        log.debug("New Client Connect! Connected IP : {}, Port : {}", connection.getInetAddress(),
                connection.getPort());

        try (InputStream in = connection.getInputStream(); OutputStream out = connection.getOutputStream()) {
            HttpRequest request = new HttpRequest(in);
            HttpResponse response = new HttpResponse(out);
            String path = getDefaultPath(request.getPath());

            if ("/user/create".equals(path)) {
               createUser(request,response);
            } else if ("/user/login".equals(path)) {
               login(request,response);
            } else if ("/user/list".equals(path)) {
               list(request, reponse);
            } else {
                response.forward(path);
            }
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }

		private void createUser(HttpRequest request, HttpResponse reponse){ ... }
		private void login(HttpRequest request, HttpResponse reponse){ ... }
		private void listUser(HttpRequest request, HttpResponse reponse){ ... }
```

➡️ 각 메소드는 앞에 request, response만 인자로 받는 것을 확인할 수 있다. 
      메소드 원형이 동일하기 때문에 인터페이스로 추출이 가능하다. ⇒ Controller(interface)

```java
package controller;

import http.HttpRequest;
import http.HttpResponse;

public interface Controller {
    void service(HttpRequest request, HttpResponse response);
}
```

1. **앞에서 분리했던 메소드의 구현 코드를 Controller 인터페이스에 대한 구현 클래스로 이동한다.**
    1. CreateUserController
    2. LoginUserController
    3. ListUserController
2. **Controller를 연결하는 RequestMapping이라는 클래스 추가**
    
    : 웹 애플리케이션에서 서비스 하는 모든 URL과 Controller를 관리하고 있으며, 요청 URL에 해당하는 Controller를 반환하는 역할을 한다.
    
    ```java
    package webserver;
    
    import java.util.HashMap;
    import java.util.Map;
    
    import controller.Controller;
    import controller.CreateUserController;
    import controller.ListUserController;
    import controller.LoginController;
    
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
    
    ⭐ run ( ) 메소트 리펙토링 결과
    
    ```java
    public void run() {
            log.debug("New Client Connect! Connected IP : {}, Port : {}", connection.getInetAddress(),
                    connection.getPort());
    
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
    ```
    
3. HTTP 메소드에 따라 다른 처리를 할 수 있도록 추상 클래스를 추가 할 수 있다.
    
    ```jsx
    public abstract class AbstractController implements Controller {
        @Override
        public void service(HttpRequest request, HttpResponse response) {
            HttpMethod method = request.getMethod();
    
            if (method.isPost()) {
                doPost(request, response);
            } else {
                doGet(request, response);
            }
        }
    
        protected void doPost(HttpRequest request, HttpResponse response) {
        }
    
        protected void doGet(HttpRequest request, HttpResponse response) {
        }
    }
    ```
    
    ➡ Controller는 인터페이스를 직접 구현하는게 아니라 AbstractController를 상속해 각 HTTP 메소드에 맞는 메소드를 오버라이드 할 수 있도록 구현할 수 있다.
    
    **장점 : HTTP 메소드가 다른 경우 새로운 Controller클래스를 추가하지 않고 Controller하나로 모두 지원하는 것이 가능해진다!!**
    
    ### 5.2.4 HTTP 웹 서버의 문제점
    
    ---
    
    - HTTP 요청과 응답 헤더, 본문 처리와 같은데 시간을 투자함으로써 정작 중요한 로직을 구현하는데 투자할 시간이 상대적으로 적다
        
        → 껍질을 만드는데 시간이 많이 소요된다.
        
    - 동적인 HTML 을 지원하는 데 한계가 있다.
        
        → 동적 콘텐츠를 생성하기 위해 PHP또는 ASP와 같은 서버측 스크립팅 언어를 사용해야하기 때문이다. 이러한 언어는 복잡하며 효과적으로 사용할려면 전문적인 지식이 필요하다.
        
    - 사용자가 입력한 데이터가 서버를 재시작하면 사라진다.
    
    ## 📌 5.3 서블릿 컨테이너, 서블릿/ JSP를 활용한 문제 해결
    
    앞에서 언급한 문제점 중에서 첫 번째, 두번째 문제점을 해결하기 위해 서블릿 컨테이너와 서블릿/JSP 를 사용해야한다.
    
    **🔎 서블릿?**
    
    앞에서 구현한 웹 서버의 Controller, HttpRequest, Httpresponse를 추상화해 인터페이스로 정의해 놓은 표준이다.
    
    서블릿은 웹 애플리케이션에서 클라이언트와 서버 간의 요청과 응답을 처리하는 Java 클래스입니다. Apache Tomcat과 같은 웹 컨테이너 내에서 실행되며 HTTP 요청 처리 및 HTTP 응답 생성을 담당합니다.
    
    ### 5.3.2 서블릿 컨테이너, 서블릿
    
    ---
    
    ```java
    @WebServlet("/hello")
    public class HelloWorldServlet extends HttpServlet {
        @Override
        protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
            PrintWriter out = resp.getWriter();
            out.print("Hello World!");
        }
    }
    ```
    
    - HttpServletRequst = HttpRequest, HttpServletResponse  = HttpResponse 이다.
    - 서블릿 컨테이너는 서버를 시작할 때  HttpServlet를 상속 받는 클래스를 찾은 후 @WebServlet 어노테이션 값을 읽어 요청 URL과 서블릿을 연결하는 Map을 생성한다.
        
        ⇒ RequestMapping 의 Map에 서블릿을 추가하고, 요청 URL에 대한 서블릿을 찾아 서비스 하는 역할을 서블릿 컨데이너가 담당한다.
        
    
    ### ✏ 서블릿의 역할
    
    1. 서블릿 클래서의 인스턴스 생성
    2. 요청 URL과 서블릿 인스턴스 매핑
    3. 클라이언트 요청에 해당하는 서블릿을 찾은 후 서블릿에 작업을 위임
    4. 서블릿 관련 초기화와 소멸 작업
    
    ### 🖋 서블릿의 생성과 종료의 과정( = 서블릿의 Life Cycle)
    
    1. 서블릿 컨테이너 시작
    2. 클래스패스에 있는  Servlet인터페이스를 구현하는 서블릿 클래스를 찾는다.
    3. @WebServlet 설정을 통해 요청 URL과 서블릿을 매핑한다.
    4. 서블릿 인스턴스를 **하나** 생성한다
        
        **서블릿 컨테이너가 시작할 때 한번 생성되면 모든 스레트가 같은 인스턴스를 재사용한다.**
        
        하지만 서블릿 컨테이너는 멀티 스레드로 동작하므로 문제점이 생길 수 있다.
        
        1. 경쟁 조건 : 두 개 이상의 스레드가 동시에 동일한 데이터를 수정하려고 시도하여 예측할 수 없고 종종 잘못된 결과를 초래하는 경우에 발생할 수 있습니다.
        2. 불일치 : 여러 스레드가 적절한 동기화 없이 동일한 데이터를 수정하면 불일치 상태가 될 수 있습니다. 즉, 데이터가 예측할 수 없거나 유효하지 않은 상태로 남아 프로그램에 오류나 버그가 발생할 수 있습니다.
        3. 교착 상태: 여러 스레드가 동일한 개체에 대한 잠금을 유지하는 경우 교착 상태가 발생할 수 있습니다. 이것은 두 개 이상의 스레드가 서로 잠금을 해제하기를 기다리고 있을 때 발생하여 프로그램이 무기한 중단됩니다.
        4. 성능 저하: 여러 스레드가 동일한 개체에 액세스하면 경합이 발생하고 성능이 저하될 수 있습니다. 이는 각 스레드가 개체를 수정하기 전에 개체에 대한 잠금을 획득해야 하므로 지연 및 처리량 감소로 이어질 수 있기 때문에 발생합니다.
        
        ⇒ 기화하는 것이 중요합니다. 이는 한 번에 하나의 스레드만 공유 데이터에 액세스할 수 있도록 하는 잠금, 뮤텍스 및 세마포어와 같은 기술을 사용하여 수행
        
        > 뮤텍스: "상호 배제"의 줄임말인 뮤텍스는 여러 스레드가 공유 리소스에 동시에 액세스하는 것을 방지하는 데 사용되는 잠금
        > 
        
        > 세마포어 : 여러 스레드 간에 공유 리소스에 대한 액세스를 제어하는 데 사용할 수 있는 보다 일반적인 동기화 메커니즘
        > 
    5. init( )메소드를 호출해 초기화 한다.
    6. 서비스를 진행한다.
    7. 소멸한다.
    
    ### 🔎 static 키워드의 역할
    
    `static` 키워드는 클래스의 개별 인스턴스가 아닌 클래스 자체에 속하는 클래스 멤버 또는 변수를 정의하는 데 사용
    
    1. 정적 변수: 정적 변수는 클래스의 모든 인스턴스 간에 공유되는 클래스 수준 변수. 즉, 한 인스턴스에서 변수를 수정하면 다른 모든 인스턴스에서 볼 수 있습니다. 정적 변수는 클래스의 인스턴스가 아닌 클래스 이름을 사용하여 액세스할 수 있습니다.
    2. 정적 메서드: 정적 메서드는 클래스의 인스턴스를 생성하지 않고 호출할 수 있는 클래스 수준 메서드입니다. 정적 메서드는 인스턴스별 데이터에 대한 액세스 권한이 없으므로 정적 변수 및 기타 정적 메서드에만 액세스할 수 있습니다.
    3. 정적 클래스: 정적 클래스는 인스턴스화할 수 없고 정적 멤버만 포함하는 클래스입니다. 정적 클래스는 인스턴스 데이터에 의존하지 않는 유틸리티 함수 또는 상수를 정의하는 데 자주 사용됩니다.
    
    🔵 이점 : 개별 인스턴스를 생성하고 소멸시키는 오버헤드가 필요하지 않기 때문에 인스턴스 멤버보다 더 효율적으로 액세스
    
    🔴 주의할 점 : 공유상태, 경직성, 높은 결합도, 성능 문제
    
    - 상세설명
        - 공유 상태: '정적' 멤버는 클래스의 모든 인스턴스 간에 공유되므로 관리 및 디버그하기 어려운 공유 상태를 생성가능
        - 경직성: '정적' 멤버를 재정의하거나 하위 클래스에서 확장할 수 없기 때문에 클래스의 유연성이 떨어지고 수정하기가 더 어려워질 수 있음
        - 높은 결합도 : 인터페이스나 다른 추상화 계층을 통하지 않고 직접 액세스되는 경우가 많기 때문에 클래스 간에 긴밀한 결합을 만들 수 있음 ⇒ 테스틍 어려워짐
        - 성능 문제 : 래스의 모든 인스턴스 간에 공유되고 경합 상태 또는 기타 동시성 문제를 방지하기 위해 동기화해야 할 수 있으므로 성능에 영향을 줄 수 있음

☑  모든 인스턴스에서 공유되지 않으며 인스턴스 별 데이터 또는 동작에 의존하지 않는 경우에만 사용해야한다!

## 📌 5.4 추가 학습 자료

### 5.4.1 객체지향(OOD) 설계와 개발

---

OOD는 다음과 같은 몇 가지 핵심 원칙을 기반으로 합니다.

1. 캡슐화: 잘 정의된 인터페이스 또는 API만 노출하여 객체의 내부 세부 정보를 외부 세계로부터 숨기는 아이디어입니다.
    - 캡슐화를 해야하는 이유가 뭘까?
        1. 데이터 보호: 캡슐화는 개체 내의 데이터가 무단 액세스 또는 수정으로부터 보호되도록 합니다. 데이터를 캡슐화하면 액세스 방법을 제어하고 제어되고 안전한 방식으로만 수정되도록 할 수 있습니다.
        2. 코드 유지 관리: 캡슐화는 객체의 구현 세부 정보를 코드의 나머지 부분과 분리하여 시간이 지남에 따라 객체를 더 쉽게 수정하고 유지 관리할 수 있도록 합니다. 인터페이스가 동일하게 유지되는 한 외부 인터페이스에 영향을 주지 않고 개체의 내부 구현을 변경할 수 있습니다.
        3. 코드 재사용: 캡슐화를 사용하면 내부 구현에 관계없이 외부 인터페이스가 동일하게 유지되므로 객체를 다른 컨텍스트에서 재사용할 수 있습니다. 즉, 코드를 수정하지 않고도 개체를 다양한 응용 프로그램 및 시나리오에서 사용할 수 있습니다.
        4. 느슨한 결합: 객체가 서로의 내부 데이터에 직접 액세스하지 않고 공개 인터페이스를 통해서만 통신하기 때문에 캡슐화는 객체 간의 느슨한 결합을 촉진합니다. 이렇게 하면 한 개체에 대한 변경 사항이 다른 개체의 동작에 영향을 주지 않기 때문에 코드가 더 모듈화되고 유지 관리가 더 쉬워집니다.
        
        요약하면 캡슐화는 데이터 보호를 보장하고, 코드 유지 관리를 용이하게 하고, 코드 재사용을 촉진하고, 개체 간의 느슨한 결합을 촉진하는 데 필요합니다. 개체의 구현 세부 정보를 캡슐화하고 잘 정의된 인터페이스만 노출하면 다양한 컨텍스트에서 사용할 수 있는 보다 모듈화되고 유연하며 유지 관리 가능한 코드를 만들 수 있습니다.
        
2. 상속: 한 클래스가 다른 클래스의 속성과 동작을 상속하여 코드 재사용 및 특수화를 허용하는 기능입니다.
    - 상속을 사용해야 하는 이유가 뭘까?
        1. 코드 재사용: 상속을 통해 기존 클래스의 코드를 재사용할 수 있으므로 새 클래스를 개발하는 데 드는 시간과 노력을 절약할 수 있습니다. 기본 클래스에서 속성과 동작을 상속하면 많은 동일한 기능을 공유하는 새 클래스를 만들 수 있습니다.
        2. 다형성(Polymorphism): 상속을 통해 유형이나 클래스에 따라 다른 형식을 취하거나 다른 컨텍스트에서 다르게 동작할 수 있는 클래스를 만들 수 있습니다. 이를 다형성이라고 하며 인터페이스 생성 또는 다양한 유형의 데이터 처리와 같은 다양한 시나리오에서 유용할 수 있습니다.
        3. 모듈성: 상속은 복잡한 시스템을 더 작고 관리하기 쉬운 부분으로 분해할 수 있게 함으로써 코드의 모듈성을 촉진합니다. 공통 기능을 정의하는 기본 클래스를 만들고 해당 기본 클래스에서 새 클래스를 파생시키면 이해하고 유지 관리하기 쉬운 계층 구조를 만들 수 있습니다.
        4. 유연성: 상속을 통해 코드를 보다 유연하게 변경하고 적응할 수 있습니다. 클래스의 동작을 수정해야 하는 경우 기본 클래스를 직접 수정하는 대신 기본 클래스의 동작을 재정의하는 새 파생 클래스를 만들면 됩니다. 이 접근 방식을 사용하면 코드의 다른 부분에 영향을 주지 않고 변경할 수 있습니다.
        
        요약하면 상속은 코드 재사용, 다형성, 모듈성 및 유연성에 유용합니다. 기존 클래스에서 속성과 동작을 상속하는 새 클래스를 생성하면 새 코드를 개발하는 데 드는 시간과 노력을 절약하고, 보다 유연하고 적응 가능한 시스템을 만들고, 코드베이스에서 모듈성과 유지 관리 가능성을 높일 수 있습니다.
        
3. 다형성(Polymorphism): 개체가 유형이나 클래스에 따라 다른 형태를 취하거나 다른 맥락에서 다르게 행동하는 능력.
    - 다형성을 사용해야 하는 이유가 뭘까?
        
        상속과 같은 
        
        1. 코드 재사용: 다형성을 사용하면 공통 동작을 공유하는 새 클래스를 만들어 기존 클래스의 코드를 재사용할 수 있습니다. 공통 인터페이스 또는 추상 클래스를 정의하면 해당 인터페이스를 구현하거나 해당 클래스를 확장하는 새 클래스를 만든 다음 해당 클래스를 동일한 인터페이스 또는 기본 클래스를 공유하는 다른 개체와 상호 교환하여 사용할 수 있습니다.
        2. 유연성: 다형성은 코드를 더 유연하게 만들고 변화에 적응할 수 있게 해줍니다. 클래스의 동작을 수정해야 하는 경우 동일한 인터페이스를 구현하거나 동일한 기본 클래스를 확장하는 새 클래스를 만든 다음 코드에서 이전 클래스를 새 클래스로 대체하면 됩니다. 이 접근 방식을 사용하면 코드의 다른 부분에 영향을 주지 않고 변경할 수 있습니다.
        3. 단순화된 코드: 다형성은 특정 인터페이스를 구현하거나 특정 클래스를 확장하는 모든 개체와 함께 작동하는 일반 알고리즘 또는 메서드를 작성할 수 있도록 하여 코드를 단순화할 수 있습니다. 이렇게 하면 코드를 더 간결하고 이해하기 쉽게 만들 수 있습니다.
        4. 모듈성: 다형성은 복잡한 시스템을 더 작고 관리하기 쉬운 부분으로 분해하여 코드의 모듈성을 촉진합니다. 일반적인 동작을 정의하는 인터페이스 또는 추상 클래스를 생성하여 이해하고 유지 관리하기 쉬운 계층 구조를 생성할 수 있습니다.
        
        요약하면 다형성은 코드 재사용, 유연성, 단순화된 코드 및 모듈성에 유용합니다. 공통 인터페이스를 구현하거나 공통 기본 클래스를 확장하는 객체를 생성하면 보다 유연하고 적응 가능한 시스템을 생성하고 코드를 단순화하며 코드베이스에서 모듈성과 유지 관리성을 높일 수 있습니다.
        
4. 추상화: 관련 없는 세부 사항은 무시하면서 시스템의 가장 중요한 측면을 식별하고 집중하는 프로세스입니다.
    - 추상화를 해야하는 이유가 뭘까?
        1. 단순화: 추상화를 사용하면 필수 기능에 집중하고 관련 없는 세부 사항을 무시하여 복잡한 시스템을 단순화할 수 있습니다. 관련 개체 그룹의 공통 기능을 정의하는 추상 클래스 또는 인터페이스를 생성하여 이해하고 작업하기 쉬운 단순화된 시스템 보기를 생성할 수 있습니다.
        2. 캡슐화: 추상화는 개체가 추상 인터페이스 뒤에서 작동하는 방식에 대한 세부 정보를 숨겨서 캡슐화를 촉진합니다. 이렇게 하면 복잡성이 사용자에게 숨겨지기 때문에 사용 및 유지 관리가 더 쉬운 개체를 만들 수 있습니다.
        3. 유연성: 추상화는 코드를 보다 유연하고 변경에 적응할 수 있게 만듭니다. 추상 인터페이스 또는 클래스를 정의하면 더 모듈화되고 수정하기 쉬운 계층 구조를 만들 수 있습니다. 이렇게 하면 시스템을 쉽게 확장하거나 동작을 수정할 수 있습니다.
        4. 재사용성: 추상화는 여러 개체에서 구현할 수 있는 추상 클래스 또는 인터페이스를 생성할 수 있도록 하여 코드 재사용성을 촉진합니다. 이렇게 하면 추상 인터페이스를 구현하거나 추상 클래스를 확장하는 기존 코드를 재사용할 수 있으므로 새 코드를 개발하는 데 드는 시간과 노력을 절약할 수 있습니다.
        
        요약하면 추상화는 단순화, 캡슐화, 유연성 및 재사용성에 유용합니다. 관련 개체 그룹의 공통 기능을 정의하는 추상 클래스 또는 인터페이스를 생성하면 작업하기 쉬운 단순화된 시스템 보기를 생성하고 캡슐화 및 코드 재사용을 촉진하며 보다 유연하고 적응 가능한 시스템을 생성할 수 있습니다. 수정 및 확장이 더 쉽습니다.
