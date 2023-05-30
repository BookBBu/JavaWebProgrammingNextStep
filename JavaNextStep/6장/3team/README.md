# 6. 서블릿/ JSP를 활용해 동적인 웹 애플리케이션 개발하기

작성일시: 2023년 5월 30일

## 📌6.1 서블릿/JSP로 회원관리 기능 다시 개발하기

---

Servlet은 HTTP 프로토콜 서비스를 통해 클라이언트의 요청을 처리하며 동적인 웹 애플리케이션을 가능하게 하나, 빠르게 개발하는데는 한계가 있다.

**Servlet Container**

서블릿을 관리해주는 컨테이너다. 서블릿은 스스로 작동하는 것이 아니고 Servlet Container에 의해 관리되며 작동한다.

**Servlet 동작 방식**

1. 클라이언트의 요청을 HTTP Request가 서블릿 컨테이너로 전송
2. 요청을 전송받은 서블릿 컨테이너가 HttpServletRequest, HttpServletResponse 객체를 생성
3. web.xml을 기반으로 서블릿을 매핑
4. 해당 서블릿의 인스턴스 존재 유무를 확인하여 없다면 init()
5. 요청 처리 후, 응답
6. distory()로 HttpServletRequest, HttpServletResponse 두 객체를 소멸

**Servlet**이 어떠한 역할을 수행하는 **정의서**라면, **Servlet Container**는 그 **정의서를 보고 수행**한다고 볼 수 있다.

**서블릿과 JSP의 한계**

- Servlet을 통해 사용자가 요청한 웹 페이지를 보여주려면 out 객체의 print() 메소드를 이용하여 HTML 문서를 작성해야 하는데, 이는 **유지보수가 어렵고 가독성이 매우 떨어진다.**
- 순수 Java 코드로만 이루어진 웹 서버용 클래스이므로, 테스트를 위해서 **항상 빌드를 다시 하여야 한다.** 이렇게 변경의 라이프 사이클이 다른 부분을 하나의 코드로 관리하는 것은 유지보수에 좋지않다.
- 웹 애플리케이션 요구사항의 복잡도가 증가하면서 많은 로직이 JSP에 자바 코드로 구현되어 유지보수하기 힘들어 진다.

### 6.1.1 서블릿/JSP복습

---

사용자가 입력한 데이터를 추출 후, DB에 데이터 추가하는 회원가입 서블릿 코드

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

/user/list와 매핑 되어있는 ListUserServlet 코드

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

위 코드의 5장 version

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
}
```

---

## 해당 서블릿/JSP 실습 코드는 책에서 설명하는 github에서 제공

### **6.1.2 개인정보수정 실습**

회원가입한 사용자가 정보를 수정할 수 있는 수정 화면을 forword해주는 doGet 메소드와 사용자가 수정한 값을 수정하는 doPost 메소드를 구현한 코드

```java
@WebServlet(value = { "/users/update", "/users/updateForm" })
public class UpdateUserController extends HttpServlet {
    private static final long serialVersionUID = 1L;
    private static final Logger log = LoggerFactory.getLogger(UpdateUserController.class);

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {**
        String userId = req.getParameter("userId");
        User user = DataBase.findUserById(userId);
        if (!UserSessionUtils.isSameUser(req.getSession(), user)) {
            throw new IllegalStateException("다른 사용자의 정보를 수정할 수 없습니다.");
        }
        req.setAttribute("user", user);
        RequestDispatcher rd = req.getRequestDispatcher("/user/updateForm.jsp");**
        rd.forward(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {**
        User user = DataBase.findUserById(req.getParameter("userId"));
        if (!UserSessionUtils.isSameUser(req.getSession(), user)) {
            throw new IllegalStateException("다른 사용자의 정보를 수정할 수 없습니다.");
        }

        User updateUser = new User(req.getParameter("userId"), req.getParameter("password"), req.getParameter("name"),
                req.getParameter("email"));
        log.debug("Update User : {}", updateUser);
        user.update(updateUser);
        resp.sendRedirect("/");
    }
}
```

**Redirect와 forward의 차이는 ?** - forward : 이동할 URL로 요청정보를 그대로 전달한다. 따라서 사용자가 최조 요청한 요청정보는 다음 URL에서도 유효하다. - redirect : redirect로 클라이언트에 새로운 URL을 리턴하면 클라이언트에서 새로운 요청을 생성하여 요청을 보낸다. - **시스템(session, DB)에 변화가 생기는 요청**(로그인, 회원가입, 글쓰기)의 경우 **redirect방식으로 응답하는 것이 바람직하며, 시스템에 변화가 생기지 않는 단순조회**(리스트보기, 검색)의 경우 **forward방식으로 응답하는 것이 바람직하다.**

### **6.1.3 로그인/로그아웃 기능 실습**

### **6.1.4 회원목록 및 개인정보 수정 보안 강화 실습**

### **6.1.5 중복 코드 제거**

**jspf란 ?**
이 장에서 중복 되는 코드를 jspf파일로 만들었다.
jspf의 풀 네임은 **Java Server Page Fragment**로, JSP 파편이라는 뜻. **JSP 코드 조각을 소스 코드 차원에서 포함시키는 기능**을 제공하고, JSP 코드 조각(fragment)을 의미하는 확장자로 .jspf 를 사용한 것

---

## 📌6.2 세션(HttpSession)요구사항 및 실습

---

Http는 클라이언트와 서버가 연결된 후 상태를 유지할 수 없다.
→ Http가 **무상태 프로토콜**이라 불리는 이유

웹 개발을 하다보면 로그인과 같이 상태를 유지할 필요가 있다 !
→쿠키 헤더를 사용하여 해결 가능!

하지만 쿠키는 보안상 매우 취약하다!!!
그래서 세션 API로 상태 데이터를 관리하는 웹 서버를 구현해보자

### **6.2.1 요구사항**

**서블릿에서 지원하는 HttpSession API의 일부를 구현**

### **6.2.2 요구사항 분리 및 힌트**

---

## 📌6.3 세션(HttpSession) 구현

- **UUID란 ?**
  **네트워크 상에서 고유성이 보장되는 id를 만들기 위한 표준 규약.**
  - 128비트의 숫자이며 32자리 16진수로 표현되며, 8자리-4자리-4자리-4자리-12자리 패턴으로 관리된다.
  - UUID버전은 1, 3, 4, 5가 있으며 1과 4버전이 가장 많이 쓰인다.
  - 1버전은 타임스탬프를 기준으로 생성된다.
  - 4버전은 랜덤 생성이다.
  - 3, 5버전은 해쉬를 이용한 생성이다.

### **6.3.1 고유한 아이디 생성**

랜덤으로 UUID를 생성한 코드

```java
package util;

import java.util.UUID;

import org.junit.Test;

public class UUIDTest {
    @Test
    public void uuid() {
        System.out.println**(UUID.randomUUID())**;
    }
}
```

- **XXUtil 클래스들의 공통점 ?**
  보통 보안, 문자열처리, 날짜 처리 등등 특정 **비즈니스 로직과 관련 없는 독립적인 기능**들은 util 패키지에 넣고 XXXUtil 클래스로 만들어서 사용.
  만약에 RandomToken이 무조건 회원과 관련되서만 사용이 된다고 하면 회원 관련 패키지에 들어갈수도 있긴 하겠지만, 토큰을 만들어내는 것 자체는 비즈니스 로직과 관련이 없기 때문에 util 패키지에 들어가는게 좋다!

### **6.3.2 쿠키를 활용해 아이디 전달**

- 클라이언트가 처음 접근하는 경우 세션 아이디를 생성하고 쿠키를 통해 전달.
- 생성된 세션 아이디가 있다면, 상태값을 공유하기 위해 해당 세션 아이디를 사용

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

- 다수의 클라이언트 세션을 관리기 위해 Map<String, HttpSession> sessions를 생성
- static으로 선언하여 한번 생성한 후 재사용하게 함.

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

서블릿에서 세션 데이터에 접근할 때 사용할 클래스

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

## 📌6.4 MVC 프레임워크 요구사항 1단계

**MVC 패턴의 등장**
소프트웨어 디자인 패턴 없이 JSP에만 대부분의 로직을 포함하게 되면 코드가 상당히 길어지며 유지보수가 어려워진다.
→ MVC패턴을 기반으로 웹 애플리케이션을 개발하는 방향으로 발전했다.

### **6.4.1 요구사항**

전에 서블릿/JSP로 구현한 기능들을 MVC패턴으로 구현하기

### **6.4.2 요구사항 분리 및 힌트**

## 📌6.5 MVC 프레임워크 구현 1단계

- 요청 URL과 각 URL에 대한 서비스를 담당할 컨트롤러를 연결하는 작업을 함
- 기존 서블릿은 서블릿 컨테이너에서 매핑을 해주었지만, 여기선 일일이 다 해주어야함 ..

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

- 클라이언트의 모든 요청을 해당하는 컨트롤러로 작업을 위임하고, 결과 페이지로 이동시키는 DispatcherServlet
- /**\*로 매핑하면 JSP에 대한 요청 또한 여기로 연결**되기에 **/로 매핑하는것이 맞다 !**

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

- 위와 같은 frontController를 사용했을 때, 어떤 장점이 있을까 ?

  - 추적(Tracking)이나 보안(Security)를 적용할 때 하나의 컨트롤러(Controller)에 하기 때문에 편하다.
  - 파일 구조가 바뀌어도 URL을 유지할 수 있다.

- loadOnStartup을 사용하여 서버 시작과 동시에 초기화를 했을 때 어떤 장점이 있을까 ?
  - loadOnStartup은 DB 연결과 같이 다소 오버헤드가 발생 할 수 있는 곳에 사용된다고 한다.
  - 그래서 서버가 시작 될 때 오버헤드를 줄일 수 있다.
