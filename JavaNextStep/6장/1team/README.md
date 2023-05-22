- 6장에서 배울 것
    - 쿠키의 문제점 살펴보기
    - 세션 구현
    - 세션의 동작 원리 이해
    - MVC패턴

# 6.1 서블릿/JSP로 회원관리 기능 다시 개발하기

## 6.1.1 서블릿/JSP 복습

---

- JSP (Java Server Page)
    - 서블릿의 한계를 극복하기 위해 등장
    - 정적인 HTML은 그대로 두고 동적으로 변경되는 부분만 JSP구문을 활용해 구현
    - 스크립틀릿 <% %> 내에 자바 구문을 그대로 사용 가능
    - 웹 애플리케이션 요구사항의 복잡도가 증가하면서 많은 로직이 JSP에 자바 코드로 구현되어 유지보수하기 힘들다는 한계점
- JSTL (JavaServer Pages Standard Tag Library)와 EL(Expression Language)
    - JSP의 한계를 극복하기 위해 등장
    - JSTL : 자바코드를 html태그형식으로 간편하게 사용하기 위해 나온 라이브러리
    - EL : 기존의 Script tag의 표현식(**<%= 정보 %>**) tag에서 업그레이드된 버전 ( **${ 정보 }** )
- MVC 패턴 적용한 프레임워크
    - JSP의 복잡도를 낮춰 유지보수를 쉽게 하기 위한 목적

→ JSTL + EL + 컨트롤러를 활용하면 JSP에서 자바 구문을 완전히 제거 가능

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<c:forEach items="${users}" var="user" varStatus="status">
                    <tr>
                        <th scope="row">${status.count}</th>
                        <td>${user.userId}</td>
                        <td>${user.name}</td>
                        <td>${user.email}</td>
                        <td><a href="/users/updateForm?userId=${user.userId}" class="btn btn-success" role="button">수정</a>
                        </td>
                    </tr>
                </c:forEach>
```

## 6.1.2 개인정보수정 실습

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<!DOCTYPE html>
<html lang="kr">
<head>
    <%@ include file="/include/header.jspf" %>
</head>
<body>
<%@ include file="/include/navigation.jspf" %>

<div class="container" id="main">
    <div class="col-md-10 col-md-offset-1">
        <div class="panel panel-default">
            <table class="table table-hover">
                <thead>
                <tr>
                    <th>#</th> <th>사용자 아이디</th> <th>이름</th> <th>이메일</th><th></th>
                </tr>
                </thead>
                <tbody>
                <c:forEach items="${users}" var="user" varStatus="status">
                    <tr>
                        <th scope="row">${status.count}</th>
                        <td>${user.userId}</td>
                        <td>${user.name}</td>
                        <td>${user.email}</td>
                        **<td><a href="/users/updateForm?userId=${user.userId}" class="btn btn-success" role="button">수정</a>**
                        </td>
                    </tr>
                </c:forEach>
                </tbody>
            </table>
        </div>
    </div>
</div>

<%@ include file="/include/footer.jspf" %>
</body>
</html>
```

```java
@WebServlet(value = { "/users/update", "/users/updateForm" })
**public class UpdateUserController extends HttpServlet {**
    private static final long serialVersionUID = 1L;
    private static final Logger log = LoggerFactory.getLogger(UpdateUserController.class);

    **@Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {**
        String userId = req.getParameter("userId");
        User user = DataBase.findUserById(userId);
        if (!UserSessionUtils.isSameUser(req.getSession(), user)) {
            throw new IllegalStateException("다른 사용자의 정보를 수정할 수 없습니다.");
        }
        **req.setAttribute("user", user);
        RequestDispatcher rd = req.getRequestDispatcher("/user/updateForm.jsp");**
        **rd.forward(req, resp);**
    }

    @Override
    **protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {**
        User user = DataBase.findUserById(req.getParameter("userId"));
        if (!UserSessionUtils.isSameUser(req.getSession(), user)) {
            throw new IllegalStateException("다른 사용자의 정보를 수정할 수 없습니다.");
        }

        User updateUser = new User(req.getParameter("userId"), req.getParameter("password"), req.getParameter("name"),
                req.getParameter("email"));
        log.debug("Update User : {}", updateUser);
        user.update(updateUser);
        **resp.sendRedirect("/");**
    }
}
```

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<!DOCTYPE html>
<html lang="kr">
<head>
    <%@ include file="/include/header.jspf" %>
</head>
<body>
<%@ include file="/include/navigation.jspf" %>

<div class="container" id="main">
    <div class="col-md-6 col-md-offset-3">
        <div class="panel panel-default content-main">
            **<form name="question" method="post" action="/users/update">**
                <input type="hidden" name="userId" value="${user.userId}" />
                <div class="form-group">
                    <label>사용자 아이디</label>
                    ${user.userId}
                </div>
                <div class="form-group">
                    <label for="password">비밀번호</label>
                    <input type="password" class="form-control" id="password" name="password" placeholder="Password">
                </div>
                <div class="form-group">
                    <label for="name">이름</label>
                    <input class="form-control" id="name" name="name" placeholder="Name" value="${user.name}">
                </div>
                <div class="form-group">
                    <label for="email">이메일</label>
                    <input type="email" class="form-control" id="email" name="email" placeholder="Email" value="${user.email}">
                </div>
                <button type="submit" class="btn btn-success clearfix pull-right">개인정보수정</button>
                <div class="clearfix" />
            </form>
        </div>
    </div>
</div>

<%@ include file="/include/footer.jspf" %>
</body>
</html>
```

- **`RequestDispatcher`**의 **`forward()`** 메서드
    - 현재 요청(**`req`**)과 응답(**`resp`**)을 다른 서블릿이나 JSP로 전달하여 해당 컴포넌트에서 추가적인 처리를 할 수 있도록 한다.
    - 전달된 요청과 응답 객체는 전달된 컴포넌트에서 직접 접근하여 처리할 수 있다.
- **`sendRedirect()`** 메서드
    - 클라이언트(웹 브라우저)에게 지정된 URL로 리다이렉트 요청
    - **`/`** 경로는 보통 웹 애플리케이션의 루트 경로

## 6.1.3 로그인/로그아웃 기능 실습

```java
@WebServlet(value = { "/users/login", "/users/loginForm" })
public class **LoginController** extends HttpServlet {
    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        forward("/user/login.jsp", req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String userId = req.getParameter("userId");
        String password = req.getParameter("password");
        User user = DataBase.findUserById(userId);
        if (user == null) {
            req.setAttribute("loginFailed", true);
            forward("/user/login.jsp", req, resp);
            return;
        }

        if (user.matchPassword(password)) {
            **HttpSession session = req.getSession();
            session.setAttribute(UserSessionUtils.USER_SESSION_KEY, user);**
            resp.sendRedirect("/");
        } else {
            req.setAttribute("loginFailed", true);
            forward("/user/login.jsp", req, resp);
        }
    }

    private void forward(String forwardUrl, HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        RequestDispatcher rd = req.getRequestDispatcher(forwardUrl);
        rd.forward(req, resp);
    }
}
```

```java
import javax.servlet.http.HttpSession;

import next.model.User;

public class UserSessionUtils {
    public static final String USER_SESSION_KEY = "user";

    public static User getUserFromSession(HttpSession session) {
        Object user = session.getAttribute(USER_SESSION_KEY);
        if (user == null) {
            return null;
        }
        return (User) user;
    }

    public static boolean isLogined(HttpSession session) {
        if (getUserFromSession(session) == null) {
            return false;
        }
        return true;
    }

    public static boolean isSameUser(HttpSession session, User user) {
        if (!isLogined(session)) {
            return false;
        }

        if (user == null) {
            return false;
        }

        return user.isSameUser(getUserFromSession(session));
    }
}
```

```java
<c:choose>
        <c:when test="${not empty sessionScope.user}">
	        <li><a href="/users/logout" role="button">로그아웃</a></li>
	        <li><a href="/users/updateForm?userId=${sessionScope.user.userId}" role="button">개인정보수정</a></li>
        </c:when>
        <c:otherwise>
	        <li><a href="/users/loginForm" role="button">로그인</a></li>
	        <li><a href="/users/form" role="button">회원가입</a></li>
        </c:otherwise>
</c:choose>
```

- jspf 파일
    - navigation.jspf
    - JSP Fragment 파일로서, 재사용 가능한 코드 블록이나 일부 컴포넌트를 담고 있는 파일
    - 일반적으로 JSP 파일에서 반복적으로 사용되는 코드나 특정 기능을 가진 컴포넌트를 별도의 jspf 파일로 분리하여 관리
        - jspf 파일은 일반적으로 다른 JSP 파일에서 **`<jsp:include>`** 또는 **`<jsp:directive.include>`**와 같은 지시자를 사용하여 포함됩니다.1
    
    ```java
    <head>
        <%@ include file="/include/header.jspf" %>
    </head>
    <body>
    	<%@ include file="/include/navigation.jspf" %> 
    ...
    ```
    

```java
@WebServlet("/users/logout")
public class LogoutController extends HttpServlet {
    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        **HttpSession session = req.getSession();
        session.removeAttribute(UserSessionUtils.USER_SESSION_KEY);
        resp.sendRedirect("/");**
    }
}
```

## 6.1.3 회원 목록 및 개인정보 수정 보안 강화 실습

```java
@WebServlet("/users")
public class ListUserController extends HttpServlet {
    private static final long serialVersionUID = 1L;

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        **if (!UserSessionUtils.isLogined(req.getSession())) {
            resp.sendRedirect("/users/loginForm");
            return;
        }**

        req.setAttribute("users", DataBase.findAll());

        RequestDispatcher rd = req.getRequestDispatcher("/user/list.jsp");
        rd.forward(req, resp);
    }
}
```

```java
public class UserSessionUtils {
    public static final String USER_SESSION_KEY = "user";

    **public static User getUserFromSession(HttpSession session) {**
        Object user = session.getAttribute(USER_SESSION_KEY);
        if (user == null) {
            return null;
        }
        **return (User) user**;
    }

    **public static boolean isLogined(HttpSession session) {**
        if (**getUserFromSession(session) == null**) {
            return false;
        }
        return true;
    }

    **public static boolean isSameUser(HttpSession session, User user) {**
        if (!isLogined(session)) {
            return false;
        }

        if (user == null) {
            return false;
        }

        return user.isSameUser(getUserFromSession(session));
    }
}
```

## 6.1.5 중복 코드 제거

```java
<head>
    **<%@ include file="/include/header.jspf" %>**
</head>
<body>
	<%@ include file="/include/navigation.jspf" %> 
...
```

# 6.2 세션(HttpSession)요구사항 및 실습

- HTTP : 클라이언트와 서버가 연결된 후 상태를 유지할 수 없음 → 무상태 프로토콜
- 로그인과 같이 상태를 유지할 필요가 있는 요구사항 발생 → 쿠키 헤더 사용
- 쿠키(Cookie)
    - “Set-Cookie” 헤더를 통해 쿠키를 생성하면 이후 발생하는 모든 요청에 “Set-Cookie”로 추가한 값을 “Cookie”헤더로 전달하는 방식
    - 보안상 취약하다는 문제점 → 브라우저 개발자 도구나 HTTP 분석 도구를 활용하여 HTTP 요청, 응답 헤더가 노출됨
- 세션
    - 쿠키의 단점 보완
    - 세션은 상태 값으로 유지하고싶은 정보를 클라이언트인 브라우저가 아닌 서버에 저장
    - 서버에 저장한 후 각 클라이언트마다 고유한 아이디를 발급해 아이디를 “Set-Cookie” 헤더를 통해 전달
    - 구체적인 메커니즘
        
        클라이언트가 웹 서버에 접속하면 서버는 해당 클라이언트를 위한 세션을 생성하고, 클라이언트에게 고유한 세션 아이디를 발급합니다. 이 세션 아이디는 일반적으로 "Set-Cookie" 헤더를 통해 클라이언트로 전달됩니다.
        
        클라이언트가 서버로 요청을 보낼 때, 세션 아이디는 "Cookie" 헤더에 포함되어 서버로 전달됩니다. 서버는 이 세션 아이디를 사용하여 클라이언트를 식별하고, 해당 세션에 저장된 상태 정보를 가져올 수 있습니다.
        
- HTTP에서 상태를 유지하는 방법은 쿠키뿐이다.
    - 세션이 상태 데이터를 서버에 관리한다는 점만 다를 뿐,
    HTTP에서 상태 값을 유지하기 위한 값을 전달할 때는 쿠키 사용

## 6.2.1 요구사항

HttpSession의 핵심이 되는 메소드 5가지

- String getId() : 현재 세션에 할당되어 있는 고유한 세션 아이디 반환
- void setAttribute(String name, Object value): 현재 세션에 value인자로 전달되는 객체를 name인자 이름으로 저장
- Object getAttribute(String name) : 현재 세션에 name인자로 저장되어 있는 객체 값을 찾아 반환
- void removeAttribute(String name) : 현재 세션에 name인자로 저장되어 있는 객체 값을 삭제
- void invalidate() : 현재 세션에 저장되어 있는 모든 값을 삭제
- 세션은 클라이언트와 서버 간에 상태값을 공유하기 위해 고유한 아이디를 활용 → 고유한 아이디는 쿠키를 활용해 공유

## 6.2.2 요구사항 분리 및 힌트

- 클라이언트와 서버 간에 주고 받을 수 있는 고유한 아이디 생성
- 고유한 아이디를 쿠키를 통해 전달
- 서버 측에서 모든 클라이언트의 세션 값을 관리하는 저장소 클래스 추가
- 클라이언트별 세션 데이터를 관리할 수 있는 클래스(HttpSession) 추가

# 6.3 세션(HttpSession) 구현

## 6.3.1 고유한 아이디 생성

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

- UUID (Universally Unique Identifier)
    - UUID는 128비트 숫자로 구성되며, 일반적으로 32개의 16진수 숫자로 표현됩니다.
    - UUID는 전 세계적으로 고유한 값을 생성하기 위해 사용됩니다. 이를 통해 여러 시스템이나 장치에서 개별적으로 식별될 수 있다.
    - UUID는 일반적으로 고유성과 임의성을 가지고 있어 충돌이 발생할 확률이 매우 낮다.
    - UUID는 여러 분야에서 다양한 용도로 활용된다. 주로 데이터베이스의 레코드 식별자, 세션 관리, 토큰 생성, 고유한 파일 이름 생성 등에 사용됩니다. 또한, 분산 시스템이나 병렬 처리 환경에서 고유성이 필요한 경우에도 사용된다.

## 6.3.2 쿠키를 활용해 아이디 전달

클라이언트가 처음 접근하는 경우 → 클라이언트가 사용할 세션 아이디를 생성한 후 쿠키를 통해 전달

한번 전달했으면 이후부터는 상태 값을 공유하기 위해 해당 세션 아이디를 활용

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
            **HttpRequest request = new HttpRequest(in);**
            HttpResponse response = new HttpResponse(out);

           ** if (request.getCookies().getCookie(HttpSessions.SESSION_ID_NAME) == null) {
                response.addHeader("Set-Cookie", HttpSessions.SESSION_ID_NAME + "=" + UUID.randomUUID());
            }**
```

```java
public HttpCookie getCookies() {
		return headers.getCookies();
	}
```

```java
private static final String COOKIE = "Cookie";

HttpCookie getCookies() {
    	return new HttpCookie(getHeader(COOKIE));
    }

String getHeader(String name) {
        return headers.get(name);
    }
```

```java
HttpCookie(String cookieValue) {
        cookies = HttpRequestUtils.parseCookies(cookieValue);
    }
```

## 6.3.3 모든 클라이언트의 세션 데이터에 대한 저장소 추가

- 서버는 다수의 클라이언트 세션을 지원해야한다. → 클라이언트 세션들을 저장할 저장소 필요
- 저장소는 모든 세션을 매번 생성하는 것이 아니라 한번 생성한 후 재사용해야한다.
    - static으로 Map을 생성해 구현

```java
public class HttpSessions {
    public static final String SESSION_ID_NAME = "JSESSIONID";

    **private static Map<String, HttpSession> sessions = new HashMap<String, HttpSession>();**

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

## 6.3.4 클라이언트별 세션 저장소 추가

- 각 클라이언트별 세션을 담당할 HttpSession 클래스
- 서블릿에서 세션 데이터에 접근할 때 사용한 클래스

```java
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

- HttpRequest에서 클라이언트에 해당하는 HttpSession에 접근할 수 있도록 메소드 추가

```java
public class HttpRequest{
 [ ... ] 
public HttpSession getSession(){
	return HttpSessions.getSession(getCookies().getCookie("JSESSIONID"));
 }
}
```

```java
public class **LoginController** extends AbstractController {
    @Override
    public void doPost(HttpRequest request, HttpResponse response) {
        **User user = DataBase.findUserById(request.getParameter("userId"));**
        if (user != null) {
            if (user.login(request.getParameter("password"))) {
                **HttpSession session = request.getSession();**
                **session.setAttribute("user", user);**
                response.sendRedirect("/index.html");
            } else {
                response.sendRedirect("/user/login_failed.html");
            }
        } else {
            response.sendRedirect("/user/login_failed.html");
        }
    }
}
```

→ 세션에 추가한 User : User객체를 추가하여 로그인 여부를 판단

```java
public class **ListUserController** extends AbstractController {
    @Override
    public void doGet(HttpRequest request, HttpResponse response) {
        if (**!isLogined(request.getSession())**) {
            response.sendRedirect("/user/login.html");
            return;
        }
...
    **private static boolean isLogined(HttpSession session) {
        Object user = session.getAttribute("user");
        if (user == null) {
            return false;
        }
        return true;
    }**
}
```

- 세션을 활용하면 클라이언트와 서버 사이에 상태 공유를 위해 전달하는 데이터는 “세션 아이디”뿐이다.
- 따라서 세션 아이디를 예측할 수 없도록 생성하는 것이 보안적 측면에서 중요
- 쿠키는 보안을 더 강화하기 위해 domain, path, max-age, expires,secure 속성 사용 가능
1. `Domain`: 쿠키를 전송할 수 있는 도메인을 지정합니다. 일반적으로 현재 도메인과 동일한 도메인에만 쿠키가 전송됩니다. 도메인 속성을 설정하면 해당 도메인과 그 하위 도메인에 대해서만 쿠키가 전송됩니다.
2. `Path`: 쿠키가 전송될 수 있는 경로를 지정합니다. 기본적으로 쿠키는 설정된 경로와 그 하위 경로에만 전송됩니다. 경로를 지정하면 해당 경로와 그 하위 경로에서만 쿠키가 전송됩니다.
3. `Max-Age`: 쿠키의 유효 기간을 지정합니다. Max-Age 속성은 쿠키가 만료될 때까지의 초 단위 시간을 설정합니다. 만료 기간이 지나면 브라우저는 쿠키를 삭제합니다.
4. `Expires`: 쿠키의 만료 일자를 지정합니다. Expires 속성은 쿠키가 만료되는 날짜와 시간을 설정합니다. Max-Age와 함께 사용되는 경우 Max-Age가 우선합니다.
5. `Secure`: 쿠키를 보안 연결(HTTPS)을 통해서만 전송하도록 지정합니다. Secure 속성이 설정된 쿠키는 암호화된 연결을 통해만 전송되므로 보안성이 향상됩니다.

# 6.4 MVC 프레임워크 요구사항 1단계

- MVC 패턴 기반 (Model, View, Controller)
    - JSP에 집중되었던 로직을 모델, 뷰, 컨트롤러 3개의 역할로 분리
    - 로직 : 컨트롤러, 모델
    - 데이터 출력 로직 : 뷰
- 웹 애플리케이션 개발에 MVC 패턴을 적용할 경우 기존과 다른 점
    - 클라이언트 요청이 처음 진입하는 부분이 컨트롤러이다.
    - 기존에는 뷰에 해당하는 JSP였음.
- 프레임워크와 라이브러리
    - 프레임워크
        - 원하는 기능 구현에 집중하여 개발할 수 있도록 일정한 형태와 필요한 기능을 갖추고 있는 골격, 뼈대를 의미합니다.
        - 애플리케이션 개발 시 필수적인 코드, 알고리즘, DB 연동과 같은 기능들을 위해 어느 정도 뼈대(구조)를 제공하며 이러한 뼈대 위에서 사용자는 코드를 작성하여 애플리케이션을 개발합니다. 앱/서버 등의 구동, 메모리 관리, 이벤트 루프 등의 공통된 부분은 프레임워크가 관리하며, 사용자는 프레임워크가 정해준 방식대로 클래서, 메서드들을 구현하면 됩니다.
        - Java 서버 개발에 사용되는 Spring
        Python 서버 개발에 사용되는 Django, Flask
        안드로이드 앱 개발에 사용되는 Android
    - 라이브러리
        - 소프트웨어를 개발할 때 컴퓨터 프로그램이 사용하는 비휘발성 자원의 모임. 즉 특정 기능을 모와둔 코드, 함수들의 집합이며 코드 작성 시 활용 가능한 도구들을 의미합니다.
        - Python pip로 설치한 패키지/모듈 (tensorflow, pandas, beautifulsoup 등등)
        C++의 표준 템플릿 라이브러리 (STL)
        Node.js에서 npm으로 설치한 모듈
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/466cb8e9-e1a7-48a4-a46d-f323dfd13adf/Untitled.png)
        
    - 공통점
        - 개발에서 발생하는 중복 코드를 제거해 재사용성을 높여 개발속도를 빠르게한다.
    - 차이점
        - 프레임워크 : 특정 패턴 기반으로 개발하도록 강제하는 역할
        - 라이브러리는 강제하는 부분이 없다.

## 6.4.1 요구사항

- 요구사항 : MVC 패턴을 지원하는 프레임워크 구현
- MVC 패턴은 최초 진입 지점이 컨트롤러이고, 뷰에 직접 접근하는 것을 막고 항상 컨트롤러를 통해 접근해야한다.
- 서블릿 필터 : css, js, 이미지같은 정적 자원은 컨트롤러가 필요없는데 해당 정적 자원이 매핑되는  것을 막기 위해 css,js이미지를 처리함
    1. 정적 자원의 요청을 가로채기: 서블릿 필터는 웹 애플리케이션으로 들어오는 모든 요청을 가로챕니다. 따라서 CSS, JavaScript, 이미지 파일 등의 정적 자원에 대한 요청도 가로챌 수 있습니다.
    2. 요청 처리 여부 결정: 서블릿 필터는 가로챈 요청이 정적 자원인지 확인합니다. 이를 위해 요청된 URL 경로, 확장자 등을 분석합니다. 정적 자원인 경우 서블릿 필터는 해당 요청을 처리하지 않고 웹 서버로 바로 전달할 수 있습니다.
    3. 추가적인 처리: 필요에 따라 서블릿 필터는 정적 자원에 대한 추가적인 처리를 수행할 수 있습니다. 예를 들어, 캐싱 기능을 추가하여 웹 서버와의 통신을 최소화하고 성능을 향상시킬 수 있습니다. 또는 정적 자원에 대한 압축, 변환, 보안 등의 작업을 수행할 수도 있습니다.

# 6.5 MVC 프레임워크 구현 1단계

```java
**@WebServlet(name = "dispatcher", urlPatterns = "/", loadOnStartup = 1)**
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

RequestMapping : 서비스에서 발생하는 모든 요청 url과 각 url에 대한 서비스를 담당할 컨트롤러를 연결하는 작업

- “/”로 매핑한 후 [http://localhost:8080](http://localhost:8080) 요청할 때, 웹 자원을 관리하는 webapp 디렉토리에 index.js가 존재하는 경우 → index.jsp로 요청이 됨
    - path가 없는 경우 처리를 담당하는 기본파일로 설정되어있기 때문이다.
- 서블릿 매핑 시 loadOnStartup 설정
    - 서블릿의 인스턴스를 생성하는 시점과 초기화를 담당하는 init()메소드를 어느 시점에 호출할 것인가를 결정
    - 설정하지 않았을 때 : 서블릿 인스턴스 생성과 초기화는 클라이언트의 요청이 최초로 발생하는 시점
    - 설정을 했을 때 : 서블릿 컨테이너가 시작하는 시점에 서블릿 인스턴스 생성과 초기화가 진행
    - 숫자가 낮은 순으로 먼저 초기화 진행됨
- 프론트 컨트롤러

프론트 컨트롤러 서블릿 하나로 클라이언트의 요청을 받고, 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출한다. 즉 모든 요청의 입구를 하나로 통일하면서 공통 처리 기능이 가능해지고,
프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 되었다.

프론트 컨트롤러 패턴은 코드 중복을 줄이고 유지 보수를 용이하게 하며, 애플리케이션 전반적인 흐름을 중앙에서 관리할 수 있다.
