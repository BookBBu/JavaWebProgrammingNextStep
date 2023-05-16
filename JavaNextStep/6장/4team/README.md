# 6. 서블릿/ JSP를 활용해 동적인 웹 애플리케이션 개발하기

작성일시: 2023년 5월 17일

## 📌6.1 서블릿/JSP로 회원관리 기능 다시 개발하기

---

서블릿이 HTTP지원과 관련해 많은 부분을 제공하고 있지만 서블릿만으로는 웹 애플리케이션을 빠르게 개발하는데 한계가 있다.

왜? 무슨 단점 ?

이 같은 단점을 보완해 좀 더 효과적인 개발이 가능하도록 프레임워크를 만드는데 이때 거의 모든 웹 프레임워크는 MVC패턴을 기반으로 하고 있다.

그래서 이 장에서는 MVC 패턴 기반으로 프레임워크를 만들면서 MVC에 대한 개념을 경험한다.

### 6.1.1 서블릿/JSP복습

---

사용자가 입력한 데이터를 추출한 후 데이터베이스에 데이터를 추가하는 회원가입 코드이다.

```java
public class CreateUserController extends AbstractController {
    private static final Logger log = LoggerFactory.getLogger(CreateUserController.class);

    @Override
    public void doPost(HttpRequest request, HttpResponse response) {
        User user = new User(request.getParameter("userId"), request.getParameter("password"),
                request.getParameter("name"), request.getParameter("email"));
        log.debug("user : {}", user);
        DataBase.addUser(user);
        response.sendRedirect("/index.html");
    }
}
```

```java
@WebServlet("/user/create")
public class CreateUserServlet extends HttpServlet {

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        User user = new User(
            req.getParameter("userId"),
            req.getParameter("password"),
            req.getParameter("name"),
            req.getParameter("email"));
        DataBase.addUser(user);
        resp.sendRedirect("/user/list");
    }
}
```

controller로 구현했을 때랑 별 차이가 없다.

---

다음은 회원가입을 완료 후 사용자 목록을 출력하는 코드이다.

```java
public class ListUserController extends AbstractController {
    @Override
    public void doGet(HttpRequest request, HttpResponse response) {
        if (!isLogined(request.getSession())) {
            response.sendRedirect("/user/login.html");
            return;
        }

        Collection<User> users = DataBase.findAll();
        StringBuilder sb = new StringBuilder();
        sb.append("<table border='1'>");
        for (User user : users) {
            sb.append("<tr>");
            sb.append("<td>" + user.getUserId() + "</td>");
            sb.append("<td>" + user.getName() + "</td>");
            sb.append("<td>" + user.getEmail() + "</td>");
            sb.append("</tr>");
        }
        sb.append("</table>");
        response.forwardBody(sb.toString());
    }

    private static boolean isLogined(HttpSession session) {
        Object user = session.getAttribute("user");
        if (user == null) {
            return false;
        }
        return true;
    }
}
```

```java
@WebServlet("/user/list")
public class ListUserServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        req.setAttribute("users", DataBase.findAll());
        RequestDispatcher rd = req.getRequestDispatcher("/user/list.jsp");
        rd.forward(req, resp);
    }
}
```

Controller는 사용자 목록을 HTML을 StringBuilder를 활용해 동적으로 생성했지만 Servlet은 JSP파일로 해당 기능을 위임하였고 이로 인해 코드가 간단해 지는 결과를 보이고 있다.

JSP는 Java Server Page라는 이름으로 자바 구문을 그대로 사용할 수 있다. 따라서 JKP 초창기에는 사용자 목록을 출력하기 위해 직접 사용해 구현했다.

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@page import="java.util.*"%>
<%@page import="next.model.*"%>

<%
Collection<User> users = (Collection<USer>)request.getAttribute("users");
for(User user : users){
%>
<tr>
    <td><%=user.getUserId()%></td>
    <td><%=user.getName()%></td>
    <td><%=user.getEmail()%></td>
    <td><a href="#" class="btn btn-success" role="button">수정</a>
    </td>
</tr>
<%
}
%>
```

JSP에서는 스크립틀릿이라고 하는 <% %>내에 자바 구문을 그대로 사용할 수 있게 되었다.

그러나 웹 애플리케이션 요구사항의 복잡도가 증가하면서 JSP를 유지보수하기 너무 힘들어졌다.
이를 극복하기 위해 등장한 기술이 JSTL과 EL이다.

또한 JSP의 복잡도를 낮춰 유지보수를 쉽게 하자는 목적으로 MVC 패턴을 적용한 프레임워크가 등장하게 되었다.

위의 코드를 JSTL과 EL을 사용하여 구현한 코드이다.

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
 <c:forEach items="${users}" var="user" varStatus="status">
                    <tr>
                        <th scope="row">${status.count}</th>
                        <td>${user.userId}</td>
                        <td>${user.name}</td>
                        <td>${user.email}</td>
                        <td><a href="#" class="btn btn-success" role="button">수정</a>
                        </td>
                    </tr>
                </c:forEach>
```

그러나 자바 구문을 JSP에서 완벽하게 제거하려면 출력할 데이터를 전달해 줄 컨트롤러가 필요하기에 결국은 MVC패턴 기반으로 개발해야 자바 구문이 완전히 제거 된다.

---

## 해당 실습 코드는 책에서 설명하는 github에서 제공

### **6.1.2 개인정보수정 실습**

### **6.1.3 로그인/로그아웃 기능 실습**

### **6.1.4 회원목록 및 개인정보 수정 보안 강화 실습**

### **6.1.5 중복 코드 제거**

---

## 📌6.2 세션(HttpSession)요구사항 및 실습

---

- Http는 클라이언트와 서버가 연결된 후 상태를 유지할 수 없다.(무상태 프로토콜)
- 그러나 로그인과 같이 상태를 유지할 필요가 있을 때 쿠키헤더를 사용한다.

**쿠키헤더를 사용하는 방법**

- "Set-Cookie"헤더를 통해 쿠키를 생성하면 이후 발생하는 모든 요청에"Set-Cookie"로 추가한 값을 "Cookie"헤더로 전달하는 방식 - 브라우저 개발자 도구나 HTTP 분석 도구를 활용해 응답 헤더를 눈으로 확인 가능해 쿠키를 사용해 개인 정보를 전달할 경우 보안에 취약함

**쿠키의 단점을 보완하기 위해 세션의 등장**

- 세션은 상태 값으로 유지하고 싶은 정보를 클라이언트인 브라우저에 저장하는 것이 아니라 서버에 저장한다.
- 서버에 저장한 후 각 클라이언트마다 고유한 아이디를 발급해 이 아이디를 "Set-Cookie"헤더를 통해 전달

**HTTP에서 상태를 유지하는 방법은 쿠키밖에 없다.**

- 세션은 HTTP의 쿠키를 기반으로 동작한다.
- 세션이 상태 데이터를 웹 서버에서 관리한다는 점만 다를 뿐 HTTP에서 상태 값을 유지하기 위한 값을 전달할 때는 쿠키를 사용한다.

---

### **6.2.1 요구사항**

**서블릿에서 지원하는 HttpSession API의 일부를 구현**

- String getId(): 현재 세션에 할당되어 있는 고유한 세션 아이디를 반환
- void setAttribute(String name, Object value): 현재 세션에 value 인자로 전달되는 객체를 name 인자 이름으로 저장
- Object get Attribute(String name): 현재 세션에 name 인자로 저장되어 있는 객체 값을 찾아 반환
- void removeAttribute(String name): 현재 세션에 name 인자로 저장되어 있는 객ㅊ 값을 삭제
- void invalidate():현재 세션에 저장되어 있는 모든 값을 삭제

### **6.2.2 요구사항 분리 및 힌트**

1. 클라이언트와 서버 간에 주고 받을 고유한 아이디를 생성해야 한다. 고유한 아이디는 쉽게 예측할 수 없어야 한다. 예측하기 쉬우면 쿠키 값을 조작해 다른 사용자처럼 속일 수 있다.

```
Hint
JDK에서 제공하는 UUID클래스를 사용해 고유한 아이디를 생성할 수 있다.
UUID uuid = UUID.randomUUID();

- UUID는 고유한 값을 식별하기 위한 아이디 값으로 32개의 16진수로 표현되며 총 36개 문자로 된 8-4-4-4-12라는 5개의 그룹을 하이픈으로 구분한다. ex)2580f09d-fca8-42ac-a318-dcbe8680730c
```

2. 앞 단계에서 생성한 고유한 아이디를 쿠키를 통해 전달한다.

```
Hint
쿠키는 Set-Cookie헤더를 통해 전달되며 name1=value1; name2=value2 형태로 전달된다.
자바 진영에서 세션 아이디를 전달하는 이름으로 JSESSIONID를 사용한다.
```

3. 서버 측에서 모든 클라이언트의 세션 값을 관리하는 저장소 클래스를 추가한다.

```
Hint
HttpSession과 같은 이름을 가지는 클래스를 추가한다.
이 클래스는 Map<String, HttpSession>와 같은 저장소를 통해 모든 클라이언트별 세션을 관리해야 한다. 이 저장소의 키는 앞에서 UUID로 생성한 고유한 아이디이다.
```

4. 클라이언트별 세션 데이터를 관리할 수 있는 클래스(HttpSession)를 추가한다.

```
Hint
HttpSession 클래스는 요구사항에 있는 5개의 메소드를 구현해야 하며, 상태 데이터를 저장할 Map<String, Object>가 필요하다.
```

---

## 📌6.3 세션(HttpSession) 구현

### **6.3.1 고유한 아이디 생성**

랜덤으로 임의 값을 생성

```java
import java.util.UUID;
import org.junit.Test;

public class UUIDTest{
    @Test
    public void uuid(){
        System.out.println(UUID.randomUUID());
    }
}
```

### **6.3.2 쿠키를 활용해 아이디 전달**

클라이언트가 처음 접근하는 경우 클라이언트가 사용할 세션 아이디를 생성한 후 쿠키를 통해 전달한다.

이렇게 세션 아이디를 한번 전달하면 이후 요청부터는 상태 값을 공유하기 위해 해당 세션 아이디를 사용하면 된다.

```java
public class RequestHandler extends Thread {
    private static final Logger log = LoggerFactory.getLogger(RequestHandler.class);

    private Socket connection;

    public RequestHandler(Socket connectionSocket) {
        this.connection = connectionSocket;
    }

    public void run() {
        log.debug("New Client Connect! Connected IP : {}, Port : {}", connection.getInetAddress(),
                connection.getPort());

        try (InputStream in = connection.getInputStream(); OutputStream out = connection.getOutputStream()) {
            HttpRequest request = new HttpRequest(in);
            HttpResponse response = new HttpResponse(out);

           if(getSessionID(request.getHeader("Cookie"))==null){
            response.addHeader("Set-cookie", "JESSIONID=" + UUID.randomUUID());
           }
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }

    private String getSessionId(String cookieValue) {
        Map<String, String> cookies =
        HttpRequestUtils.parseCookies(cookieValue);
        return cookies.get("JSESSIONID");
    }
}

```

### **6.3.3 모든 클라이언트의 세션 데이터에 대한 저장소 추가**

서버는 다수의 클라이언트 세션을 지원해야 해서 모든 클라이언트의 세션을 관리할 수 있는 저장소가 필요하다.

이 저장소는 모든 세션을 매번 생성하는 것이 아니라 한번 생성한 후 재사용할 수 있어야 한다.

```java
package http;

import java.util.HashMap;
import java.util.Map;

public class HttpSessions {
    public static final String SESSION_ID_NAME = "JSESSIONID";

    private static Map<String, HttpSession> sessions = new HashMap<String, HttpSession>();

    public static HttpSession getSession(String id) {
        HttpSession session = sessions.get(id);

        if (session == null) {
            session = new HttpSession(id);
            sessions.put(id, session);
            return session;
        }

        return session;
    }

    static void remove(String id) {
        sessions.remove(id);
    }
}
```

### **6.3.4 클라이언트별 세션 저장소 추가**

```java
package http;

import java.util.HashMap;
import java.util.Map;

public class HttpSession {
    private Map<String, Object> values = new HashMap<String, Object>();

    private String id;

    public HttpSession(String id) {
        this.id = id;
    }

    public String getId() {
        return id;
    }

    public void setAttribute(String name, Object value) {
        values.put(name, value);
    }

    public Object getAttribute(String name) {
        return values.get(name);
    }

    public void removeAttribute(String name) {
        values.remove(name);
    }

    public void invalidate() {
        HttpSessions.remove(id);
    }
}
```

---

지금까지 구현 과정을 보면 세션을 활용하면 클라이언트와 서버 사이에 상태 공유를 위해 전달하는 데이터는 세션 아이디 뿐이다.

따라서 세션 아이디를 예측할 수 없도록 생성하는 것은 보안 측면에서 중요하다.

쿠키는 보안을 좀 더 강화하기 위해 domain, path, max-age, expires, secure 속성을 사용할 수 있다.

## 📌6.4 MVC 프레임워크 요구사항 1단계
**MVC 패턴의 등장**
- **JSP의 단점 발생**

- 많은 애플리케이션이 웹으로 개발되고, 요구사항의 복잡도는 점점 더 증가

- 또한 웹 애플리케이션의 수명이 길어지면서 유지보수 업무 증가

- JSP에 상당 부분의 로직을 포함하는 것이 초기 개발 속도는 빨랐지만 유지보수 비용은 증가했다.

이러한 단점을 보완해 유지보수 비용을 줄이기 위해 MVC패턴 기반의 개발이 등장

## **MVC 패턴의 요청과 응답 흐름**
---
클라이언트 ->(요청) 컨트롤러 ->(요청) 모델 ->(응답) 컨트롤러 ->(모델) 뷰 ->(응답) 클라이언트

MVC패턴을 적용하기 전엔 JSP가 클라이언트 요청이 처음으로 진입하는 부분이었으나 

MVC패턴을 적용한 후 컨트롤러가 처음으로 받게 된다. 

이렇게 MVC패턴을 적용하면 대부분의 로직은 컨트롤러와 모델이 담당하고, 

뷰에 해당하는 JSP는 컨트롤러에서 전달한 데이터를 출력하는 로직만 포함하게 되고 이렇게 되면 JSP의 복잡도는 상당히 낮아진다.

---

프레임워크를 사용하는 이유
- 개발자의 역량의 차이에 따라 MVC패턴을 적용한 코드와 그렇지 않은 코드가 섞여 있을 경우 일관성이 떨어져 유지보수에 어려움이 있다.
- 프레임워크는 이러한 차이가 있더라도 일관성 있는 코드를 구현하도록 강제하는 역할을 한다.
- 또한 애플리케이션 개발에서 발생하는 중복 코드를 제거해 재사용성을 높임으로써 개발 속도를 빠르게도 한다. 이는 라이브러리도 동일하다.

**프레임워크와 라이브러리의 차이점**

프레임워크는 특정 패턴 기반으로 개발하도록 강제하는 역할을 하고 라이브러리는 강제하는 부분이 없다.

### **6.4.1 요구사항**
전에 서블릿/JSP로 구현한 기능들을 MVC패턴으로 구현하기
### **6.4.2 요구사항 분리 및 힌트**
- 모든 요청을 서블릿 하나(예를 들어 DIspatcherServlet)가 받을 수 있도록 URL매핑한다.
```
Hint
@WebServlet(name="dispatcher", urlPatterns="/", loadOnStartup = 1)
loadOnStartup속성이 무슨 역할을 하는지 학습해 본다.
서블릿은 "/"로 설정함으로써 모든 요청을 하나의 서블릿으로 매핑할 수 있다.
```
- Controller 인터페이스를 추가한다.
```java
Hint
public interface Controller{
    String execute(HttpServletRequest request, HttpServletResponse response) throws Exception;
}
execute() 메소드의 반환 값이 String이라는 것을 눈여겨 보자.
```
- 서블릿으로 구현되어 있는 회원관리 기능을 앞 단계에서 추가한 Controller 인터페이스 기반으로 다시 구현한다. execute() 메소드의 반환 값은 리다이렉트 방식으로 이동할 경우 redirect:로 시작하고 포워드 방식으로 이동할 경우 JSP 경로를 반환한다.
```java
Hint
public class ListUserController implements Controller{
    @Override
    public String execute(HTtpServletRequest req, HttpServletResponse resp) throws Exception{
        if(!UserSessionUtils.isLogined(req.getSession())){
            return "redirect:/users/loginForm";
        }

        req.setAttribute("users", DataBase.findAll());
        return "/user/list.jsp";
    }
}
```
- RequestMapping클래스를 추가해 요청 URL과 컨트롤러 매핑을 설정한다.
```
Hint
요청 URL과 컨트롤러를 매핑할 때 Map<String, Controller>에 설정한다.
```
- 특별한 로직 없이 뷰에 대한 이동만을 담당하는 ForwardController를 추가한다.
- DispatcherServlet에서 요청 URL에 해당하는 Controller를 찾아 execute() 메소드를 호출해 실질적인 작업을 위임한다.
- Controller의 execute() 메소드 반환 값 String을 받아 서블릿에서 JSP로 이동할 때의 중복을 제거한다.
```
Hint
반환 값이 redirect:로 시작할 경우 sendRedirect()로 이동하고, redirect:이 아닌 경우 RequestDispatcher의 forward방식으로 이동한다.
예를 들어 redirect:/user/list라면 /user/list URL로 리다이렉트하도록 구현한다.
```
## 📌6.5 MVC 프레임워크 구현 1단계
- 서블릿에서 JSP로 이동을 할 때 구현해야 하는 중복 코드가 제거 되었다.
```java
public class ListUserController implements Controller {
    @Override
    public String execute(HttpServletRequest req, HttpServletResponse resp) throws Exception {
        if (!UserSessionUtils.isLogined(req.getSession())) {
            return "redirect:/users/loginForm";
        }

        req.setAttribute("users", DataBase.findAll());
        return "/user/list.jsp";
    }
}
```
- 특별한 로직 없이 뷰에 대한 이동만을 담당하는 ForwardController 추가
```java
public class ForwardController implements Controller {
    private String forwardUrl;

    public ForwardController(String forwardUrl) {
        this.forwardUrl = forwardUrl;
        if (forwardUrl == null) {
            throw new NullPointerException("forwardUrl is null. 이동할 URL을 입력하세요.");
        }
    }

    @Override
    public String execute(HttpServletRequest req, HttpServletResponse resp) throws Exception {
        return forwardUrl;
    }
}
```
- 서비스에서 발생하는 모든 요청 URL과 각 URL에 대한 서비스를 다 ㅁ당할 컨트롤러를 연결하는 작업을 한다.
```java
public class RequestMapping {
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);
    private Map<String, Controller> mappings = new HashMap<>();

    void initMapping() {
        mappings.put("/", new HomeController());
        mappings.put("/users/form", new ForwardController("/user/form.jsp"));
        mappings.put("/users/loginForm", new ForwardController("/user/login.jsp"));
        mappings.put("/users", new ListUserController());
        mappings.put("/users/login", new LoginController());
        mappings.put("/users/profile", new ProfileController());
        mappings.put("/users/logout", new LogoutController());
        mappings.put("/users/create", new CreateUserController());
        mappings.put("/users/updateForm", new UpdateFormUserController());
        mappings.put("/users/update", new UpdateUserController());

        logger.info("Initialized Request Mapping!");
    }

    public Controller findController(String url) {
        return mappings.get(url);
    }

    void put(String url, Controller controller) {
        mappings.put(url, controller);
    }
}
```
- 클라이언트의 모든 요청을 받아 URL에 해당하는 컨트롤러로 작업을 위임하고, 실행된 결과 페이지로 이동하는 작업
```java
@WebServlet(name = "dispatcher", urlPatterns = "/", loadOnStartup = 1)
public class DispatcherServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);
    private static final String DEFAULT_REDIRECT_PREFIX = "redirect:";

    private RequestMapping rm;

    @Override
    public void init() throws ServletException {
        rm = new RequestMapping();
        rm.initMapping();
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String requestUri = req.getRequestURI();
        logger.debug("Method : {}, Request URI : {}", req.getMethod(), requestUri);

        Controller controller = rm.findController(requestUri);
        try {
            String viewName = controller.execute(req, resp);
            move(viewName, req, resp);
        } catch (Throwable e) {
            logger.error("Exception : {}", e);
            throw new ServletException(e.getMessage());
        }
    }

    private void move(String viewName, HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        if (viewName.startsWith(DEFAULT_REDIRECT_PREFIX)) {
            resp.sendRedirect(viewName.substring(DEFAULT_REDIRECT_PREFIX.length()));
            return;
        }

        RequestDispatcher rd = req.getRequestDispatcher(viewName);
        rd.forward(req, resp);
    }
}
```

해당 코드에서 서블릿 매핑을 "/"로 하면 모든 요청 URL이 연결된다. 그러나 보통 모든 요청을 연결하면 *을 사용하여 /*을 사용한다 생각한다.

/*을 사용할 수 있으나 이와 같이 매핑을 하게 되면 모든 JSP에 대한 요청이 연결되므로 정상적으로 처리되지 않는 문제가 발생한다.
## "/"매핑
톰캣 서버의 기본 설정을 보면 "/" 설정은 "default"라는 이름을 가지는 서블릿을 매핑해 정적 자원을 처리하도록 구현하고 있다.

이 설정을 DispatcherSevlet에서 다시 재정의함으로써 JSP에 대한 요청은 처리하지 않으면서 그 외의 모든 요청을 담당하도록 구현되어있다.

**그렇다면 "default"처리는 어떻게 된것인가?**
이에 대한 처리는 DispatcherSevlet으로 요청하기 전에 core.web.filter.ResourceFilter에서 처리하도록 구현되어있다.
이와 같이 설정할 경우 주의할 점이 하나 있다.

HomeController에서 "/"로 매핑할 경우 index.jsp가 존재하는 경우  HomeController로 가는 것이 아니라 index.jsp로 요청이 처리된다. 

**이는 path가 없는 경우 처리를 담당하는 기본 파일로 설정되어 있기 때문이다.**

---
**서블릿을 매핑할때 loadOnStartup설정을 추가할 수 있다.**

이 설정은 서블릿의 인스턴스를 생성하는 시점과 초기화를 담당하는 init() 메소드를 어느 시점에 호출할 것인가를 결정하는 설정이다.

해당 설정을 하지 않을 경우 서블릿 인스턴스 생성과 초기화는 서블릿 컨테이너가 시작을 완료한 후 클라이언트의 요청이 최초로 발생하는 시점에 진행된다. 이때 초기화 순서는 loadOnStartup설정 숫자 값이 낮은 순으로 진행된다.

---
DispatcherServlet의 move()메소드를 보면 각 서블릿에서 서블릿과 JSP 사이를 이동하기 위해 구현한 모든 중복 코드를 담당하고 있다. 이는 리팩토링을 함으로 써 서비스의 복잡도가 증가하더라도 유지보수하기 좋은 코드로 만들 수 있다.

**이 같은 구조로 MVC 프레임 워크를 구현하는 패턴을 프론트 컨트롤러 패턴이라고 한다.**

프론트 컨트롤러 패턴 - 각 콘트롤러의 앞에 모든 요청을 받아 각 컨트롤러에 작업을 위임하는 방식
## 📌6.6 쉘 스크립트를 활용한 배포 자동화

### **6.6.1 요구사항**
- 지금까지 구현한 기능을 개발 서버에 톰캣 서버를 설치한 후 배포한다.
- 서버가 정상적으로 실행되고 있는지 톰캣 로그 파일을 통해 모니터링한다.
- 쉘 스크립트를 만들어 배포 과정을 자동화한다.
### **6.6.2 힌트**

1. 톰캣 서버 설치
2. 실습 코드 배포
3. 톰캣 서버 로그 모니터링
4. 쉘 스크립트 통해 배포 자동화

### **6.6.3 동영상을 참고한 배포 자동화 실습**

**자바 웹 애플리케이션을 배포하는 방법**

1. war파일을 만들어 배포하는 방법
2. war파일로 묶지 않고 디렉토리 자체를 배포하는 방법

## 📌6.7 추가 학습 자료

### **6.7.1 쿠키와 세션, 보안**

쿠키와 세션의 차이를 이해하고 사용하는 것은 안전한 웹을 만들기 위한 첫 걸음

1. **프로가 되기 위한 웹기술 입문(고모라 유스케 저/ 김정환 역, 위키북스/2012)**
   - 공격 방법 SQL 인젝션, 크로스 사이트 스크립팅(XSS), 세션 하이재킹, 크로스 사이트 요청 위조(CSRF)

안전한 웹 애플리케이션 개발을 위한 책

2. **HTTP & Network : 그림으로 배우는 책으로 학습(우에노 센 저/이병억 역, 영진닷컴/2015)"**

### **6.7.2 쉘 스크립트와 배포 자동화**
굳이 쉘 스크립트를 활용하지 않고 같은 기능을 지원하는 다른 도구를 사용해도 된다. 즉 개발자가 단순, 반복적으로 하는 작업을 최소한의 시간 투자로 자동화할 수 있다면 그 자체로 목적을 달성하는 것이다.

쉘 스크립트를 활용하면 개발자의 무수히 많은 단순, 반복적인 수동 작업을 자동화하는 것이 가능하다. 특히 웹 백엔드 개발자에게 자동화된 배포 환경을 구축하는 것은 많은 시간을 아낄 수 있는 중요한 작업이다.

쉘 스크립트에 대한 기본적인 학습은 "리눅스 커맨드라인 완벽입문서" 책을 읽어보되 책으로는 한계까 있으므로 작은 것이라도 자기 주변에서 발생하는 불편한 점을 쉘 스크립트를 통해 개선해 나가는 경험을 하는 것이 좋은 학습 방법이다.