# 자뿌발표준비(8장)

작성일시: 2023년 6월 10일

## 📌8.2 AJAX 활용해 답변 추가, 삭제 실습

![image](https://github.com/BookBBu/JavaWebProgrammingNextStep/assets/98890934/551b3e10-5380-4bf6-b68b-66583dc2f979)

[서버에서 HTML응답 찾기 및 처리]

- 서버는 요청을 처리하고 응답으로 HTML 문서를 생성한다.
- 서버는 적절한 HTTP 헤더와 함께 응답 본문에 HTML 문서를 포함한다.
- 브라우저는 응답을 받고 처리를 시작
- 브라우저는 HTML 문서를 구문 분석하고 문서 개체 모델(DOM) 트리를 구성하며 CSS 스타일을 적용하여 웹 페이지를 렌더링
- HTML에 포함된 모든 JavaScript 코드가 실행되어 DOM을 수정하거나 페이지에서 다른 작업을 수행
- 브라우저는 최종 웹 페이지를 렌더링하여 사용자에게 표시

> 브라우저는 HTML 문서를 요청하고 서버는 **완전한 HTML 페이지로 응답**합니다. 그런 다음 브라우저는 전체 페이지를 렌더링합니다. → 많은 단계를 거치고 더 많은 비용 발생
> 

[AJAX를 사용한 처리]

- AJAX를 사용하면 전체 페이지를 다시 로드하지 않고도 웹 페이지가 서버에서 비동기적으로 데이터를 가져올 수 있다.
- 전체 HTML 페이지를 요청하는 대신 브라우저는 일반적으로 XMLHttpRequest 개체 또는 최신 Fetch API를 사용하여 서버에 AJAX 요청
- 서버는 요청을 처리하고 일반적으로 XML 또는 JSON과 같은 구조화된 형식의 응답을 생성
- 서버는 적절한 HTTP 헤더와 함께 응답을 브라우저로 다시 보낸다.
- 브라우저는 AJAX 응답을 수신하고 응용 프로그램의 요구 사항에 따라 다양한 방식으로 처리
    
    (일반적으로 브라우저의 JavaScript 코드는 AJAX 응답을 처리)
    
- JavaScript 코드는 응답에서 데이터를 추출하거나 DOM을 동적으로 조작하거나 전체 페이지를 다시 로드하지 않고 웹 페이지의 특정 부분을 업데이트
- 업데이트된 콘텐츠는 웹 페이지와의 현재 상호 작용을 중단하지 않고 사용자에게 표시

> AJAX를 사용하면 브라우저가 데이터에 대한 비동기 요청(일반적으로 XML 또는 JSON과 같은 구조화된 형식)을 보내고 요청된 데이터만 응답으로 받습니다. 
브라우저의 JavaScript 코드는 이 응답을 처리하고 전체 페이지를 다시 로드하지 않고 웹 페이지의 일부를 선택적으로 업데이트할 수 있습니다. 
이를 통해 **사용자의 현재 컨텍스트를 방해하지 않고 백그라운드에서 콘텐츠를 업데이트할 수 있으므로 보다 동적이고 상호 작용적인 웹 경험이 가능**합니다.
> 

### AJAX 장단점

**장점**

- 서버를 기다리지 않고, 비동기 요청 가능
    
    <aside>
    💡 **비동기 요청** 
    브라우저는 요청을 기다리지 않고 동시에 여러 요청 보낼수 있음, 동시작업 가능
    
    </aside>
    
- 페이지 이동 없이 고속으로 화면 전환 가능
- 대역폭 소비를 줄입니다. 이는 모바일 사용자와 같이 인터넷 연결이 제한된 사용자에게 특히 유용(필요한 데이터만 검색하기 때문)

**단점**

- JavaScript 종속성 : 자바스크립트에 크게 의존하기 때문에 JavaScript가 비활성화되어 있거나 사용자 브라우저에서 지원되지 않는 경우 기능 작동이 안됌
- 검색 엔진 최적화문제 : 검색 엔진 크롤러는 AJAX를 통해 동적으로 로드된 콘텐츠를 구문 분석하는 것은 어려움. 검색 엔진이 AJAX 기반 콘텐츠를 인덱싱하는 데 발전을 이루었지만 검색 엔진 최적화(SEO) 노력에 여전히 문제.
- 복잡한 구현 : 숙련된 JavaScript 기술과 비동기 작업필요
- 개발 시간 증가 : 개발자는 비동기 요청을 처리하고, 응답을 관리하고, DOM을 동적으로 업데이트해 → 기존의 서버 측 렌더링보다 더 복잡
- 보안 고려 사항 : AJAX가 제대로 구현되지 않으면 보안 위험이 발생
    - XSS(교차 사이트 스크립팅)
        
        공격자가 악의적인 스크립트를 삽입하여 사용자의 브라우저에서 실행되게 하는 공격
        
        XSS는 주로 사용자로부터 입력을 받아 웹 페이지에 동적으로 출력되는 곳에서 발생
        
        공격자는 입력 필드, 코멘트 섹션, 게시판 등의 공격 대상이 될 수 있는 부분에 악성 스크립트를 삽입 →  서버로부터 받은 페이지를 요청한 사용자의 브라우저에서 실행되어 추가적인 악의적인 행위를 수행
        
    - CSRF(교차 사이트 요청 위조)
        
        인증된 사용자의 의지와 무관하게 공격자가 특정 요청을 위조하여 실행
        

> **이런 단점에도 불구하고 왜 쓰는 걸까?**
AJAX는 향상된 사용자 경험, 성능 최적화, 향상된 상호 작용과 같은 이점을 제공하므로 최신 ****웹 응용 프로그램을 만드는 데 유용
> 

### 8.2.1 답변하기

답변하기 클릭 → 입력 정보를 서버로 전송 → 데이터베이스에 저장 → 클라이언트에 JSON형태로 전송 → 전송받은 JSON데이터를 HTML로 변환해서 화면에 출력

1. 답변하기 클릭 → addAnswer( )함수 실행

```jsx
$(".answerWrite input[type=submit]").click(addAnswer);
```

1. 입력한 정보를 서버로 전송할 수 있도록 추출

```jsx
function addAnswer(e) {
  e.preventDefault();   //submit 이 자동으로 동작하는 것을 방지한다.

  var queryString = $("form[name=answer]").serialize();

function onSuccess(json, status){
  var answerTemplate = $("#answerTemplate").html();
  var template = answerTemplate.format(json.writer, new Date(json.createdDate), json.contents, json.answerId, json.answerId);
  $(".qna-comment-slipp-articles").prepend(template);
}

function onError(xhr, status) {
  alert("error");
}

String.prototype.format = function() {
  var args = arguments;
  return this.replace(/{(\d+)}/g, function(match, number) {
    return typeof args[number] != 'undefined'
        ? args[number]
        : match
        ;
  });
};
```

**[AddAnswerController.java] :** 클라이언트 요청을 처리하는 서버

```java
public class AddAnswerController implements Controller {
    private static final Logger log = LoggerFactory.getLogger(AddAnswerController.class);

    @Override
    public String execute(HttpServletRequest req, HttpServletResponse resp) throws Exception {
        Answer answer = new Answer(req.getParameter("writer"), req.getParameter("contents"),
                Long.parseLong(req.getParameter("questionId")));
        log.debug("answer : {}", answer);

        AnswerDao answerDao = new AnswerDao();
        Answer savedAnswer = answerDao.insert(answer);
        ObjectMapper mapper = new ObjectMapper();
        resp.setContentType(**"application/json;charset=UTF-8"**);
        PrintWriter out = resp.getWriter();
        out.print(mapper.writeValueAsString(savedAnswer));
        return null;
    }
}
```

> 응답을 할때 HTML이 아닌 JSON(XML)형태로만 데이터를 전달 한다.
> 

**[RequestMapping.java]**

```java
public class RequestMapping {
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);
    private Map<String, Controller> mappings = new HashMap<>();

    void initMapping() {
        ...
        mappings.put("/api/qna/addAnswer", new AddAnswerController());
        ...
	}
}
```

> JSON의 형태로 클라이언트 javascript에 전달된다.
> 

### 8.2.2 답변 삭제하기

---

- 모든 삭제 링크에 클릭 이벤트 발생시 함수를 호출 하도록 구현

```java
$(".qna-comment").on("click", ".form-delete", deleteAnswer);
```

- jQuery의 ajax( ) 함수를 통해 서버에 삭제 요청을 보낸다.

```java
function deleteAnswer(e) {
  e.preventDefault();

  var deleteBtn = $(this);
  var queryString = deleteBtn.closest("form").serialize();

  $.ajax({
    type: 'post',
    url: "/api/qna/deleteAnswer",
    data: queryString,
    dataType: 'json',
    error: function (xhr, status) {
      alert("error");
    },
    success: function (json, status) {
      if (json.status) {
        deleteBtn.closest('article').remove();
      }
    }
  });
}

String.prototype.format = function() {
  var args = arguments;
  return this.replace(/{(\d+)}/g, function(match, number) {
    return typeof args[number] != 'undefined'
        ? args[number]
        : match
        ;
  });
};
```

- 서버는 클라이언트 요청에 대해 답변을 삭제한다. 삭제 결과는 next.model.Result결과를 활용한다.

```java
public class DeleteAnswerController implements Controller {
    @Override
    public String execute(HttpServletRequest req, HttpServletResponse resp) throws Exception {
        Long answerId = Long.parseLong(req.getParameter("answerId"));
        AnswerDao answerDao = new AnswerDao();

        answerDao.delete(answerId);

        ObjectMapper mapper = new ObjectMapper();
        resp.setContentType("application/json;charset=UTF-8");
        PrintWriter out = resp.getWriter();
        out.print(mapper.writeValueAsString(Result.ok()));
        return null;
    }
}
```

requstMapping

```java
public class RequestMapping {
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);
    private Map<String, Controller> mappings = new HashMap<>();

    void initMapping() {
        ...
        mappings.put("/api/qna/deleteAnswer", new DeleteAnswerController());
        ...
	}
}
```

### 🤷‍♂️ 자바스크립트의 this? 자바의 this?

---

- java : 클래스의 현재 인스턴스
- javaScript
    1. 함수가 호출되는 방식에 따라 달라진다. 동적으로 변경 가능하다.
    2. call( ), apply( ), bind( )메소드를 사용하여 명시적으로 설정가능하다. 함수가 객체의 메서드가 아닌 경우 전역 객체(예: 브라우저의 창 객체)를 참조할 수도 있음
    3. 익명 함수의 "this" 값은 함수가 실행되는 컨텍스트에 따라 다릅니다. 함수가 호출되는 방식이나 주변 코드가 구성되는 방식에 영향을 받을 수 있음
    4. 정적 메서드 또는 정적 컨텍스트라는 개념이 없다. 따라서 "this"는 클래스 자체를 가리키는 것이 아니라 현재 코드를 실행 중인 개체를 나타낸다.
    
    | 함수 | 설명 |
    | --- | --- |
    | call( )  | call() 메서드는 this에 대해 지정된 값으로 함수를 호출하고 인수를 개별적으로 제공 |
    | apply ( ) | call()과 유사하지만 인수를 배열 또는 배열과 유사한 객체로 취한다. |
    | bind( ) | this에 대해 지정된 값과 선택적 사전 설정 인수를 사용하여 새 함수를 생성. 
    나중에 호출할 수 있는 새 함수를 반환합니다. |

## 📌8.3 MVC 프레임 워크 요구사항 2단계

**MVC 프레임워크 의도와 맞지 않은 부분 발생**

1. JSON으로 응답을 보내는 경우 이동할 JSP페이지가 없어 불필요하게 null을 반환해야한다.
    
    (변환 값이 굳이 필요 없음)
    
2. AnswerController와 DeleteAnswerController에서 자바 객체를 JSON으로 변환하고 응답하는 부분에서 중복 발생

**View 인터페이스를 추가해 구조를 변경하자!**

1. View 인터페이스를 구현하는 JspView, JsonView를 추가해 컨드롤러에서 전달하는 모델 데이터를 활용해 응답하도록 개선
2. HttpServletRequest을 사용하지 않고 별도의 Map을 활용해 전달하도록 구조를 변경
3. 모델 데이터와 뷰를 추상화한 ModelAndView를 추가해 Controller 인터페이스를 변경

## 📌8.4 MVC 프레임 워크 구현 2단계

### 8.4.1 View인터페이스 추가

---

```java
public interface View {
    void render(HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```

### 8.4.2 JspView와 JsonView추가

---

- JSP에 대한 페이지 이동 처리를 담당하는 JspView 추가
- 이동할 뷰 이름을 생성자로 받은 후 render()메소드 호출 시 해당 페이지로 이동

```java
public class JspView implements View {
    private static final String DEFAULT_REDIRECT_PREFIX = "redirect:";

    private String viewName;

    public JspView(String viewName) {
        if (viewName == null) {
            throw new NullPointerException("viewName is null. 이동할 URL을 입력하세요.");
        }
        this.viewName = viewName;
    }

    @Override
    public void render(HttpServletRequest req HttpServletResponse response)
            throws Exception {
        if (viewName.startsWith(DEFAULT_REDIRECT_PREFIX)) {
            response.sendRedirect(viewName.substring(DEFAULT_REDIRECT_PREFIX.length()));
            return;
        }

        RequestDispatcher rd = req.getRequestDispatcher(viewName);
        rd.forward(req, response);
    }
}
```

```java
public class JsonView implements View {
    @Override
    public void render(HttpServletRequest req, HttpServletResponse response)
            throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        response.setContentType("application/json;charset=UTF-8");
        PrintWriter out = response.getWriter();
        out.print(mapper.writeValueAsString(createModel(req));
    }

    private Map<String, Object> createModel(HttpServletRequest req) {
      Enumeration<String> names = req.getAttributeNames();
      Map<String, Object> model = new HashMap<>();
      while(names.hasMoreElements()) {
        String name = names.nextElement();
        model.put(name, request.getAttribute(name));
      }
      return model;
    }
}
```

**[얻을 수 있는 장점]**

1. 유연성 : 예를 들어, JSP 뷰, Thymeleaf 뷰, JSON 뷰 등 다양한 종류의 뷰를 각각의 클래스로 구현할 수있어 유연성이 높아 진다.
2. 확장성 : 새로운 형태의 뷰를 추가하는 것이 쉬워짐
3. 의존성 분리 : 해당 인터페이스에만 의존하도록 코드를 작성할 수 있다.  이는 뷰 엔진에 대한 구체적인 의존성을 제거하고, 뷰의 구현과 애플리케이션 로직을 분리시킴으로써 코드의 가독성과 유지보수성을 향상시킨다.
4. 단위 테스트의 용이성 : 뷰와 관련된 로직을 독립적으로 테스트 가능

### 8.4.3 Controller 반환 값을 String에서 View로 수정

---

```java
public interface Controller {
    ModelAndView execute(HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```

### 8.4.4 Controller 구현체가  String대신 View를 반환하도록 수정

---

**HomeController.java**

```java
public class HomeController extends AbstractController {
    private QuestionDao questionDao = new QuestionDao();

    @Override
    public ModelAndView execute(HttpServletRequest request, HttpServletResponse response) throws Exception {
        return jspView("home.jsp").addObject("questions", questionDao.findAll());
    }
}
```

**AddAnswerController.java**

```java
public class AddAnswerController extends AbstractController {
    private static final Logger log = LoggerFactory.getLogger(AddAnswerController.class);

    private AnswerDao answerDao = new AnswerDao();

    @Override
    public ModelAndView execute(HttpServletRequest req, HttpServletResponse response) throws Exception {
        Answer answer = new Answer(req.getParameter("writer"), req.getParameter("contents"),
                Long.parseLong(req.getParameter("questionId")));
        log.debug("answer : {}", answer);

        Answer savedAnswer = answerDao.insert(answer);
        return jsonView().addObject("answer", savedAnswer);
    }
}
```

### 8.4.5 DispatcherServlet이 View에 작업을 위임하도록 수정

---

```java
@WebServlet(name = "dispatcher", urlPatterns = "/", loadOnStartup = 1)
public class DispatcherServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);

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

        Controller controller = rm.findController(reqestUri);
        try {
            View view = controller.execute(req, resp)
            view.render(req, resp);
        } catch (Throwable e) {
            logger.error("Exception : {}", e);
            throw new ServletException(e.getMessage());
        }
    }
}
```

> 페이지 이동 작업(move ( ) )을 JspView의 render() 메소드로 이동 → 깔끔
> 

### 8.4.6 ModelAndView추가를 통한 모델 추상화

---

**[ModelAndView.java] : 모델 데이터에 대한 추상화 담당**

```java
public class ModelAndView {
    private View view;
    private Map<String, Object> model = new HashMap<>();

    public ModelAndView(View view) {
        this.view = view;
    }

    public ModelAndView addObject(String attributeName, Object attributeValue) {
        model.put(attributeName, attributeValue);
        return this;
    }

    public Map<String, Object> getModel() {
        return Collections.unmodifiableMap(model);
    }

    public View getView() {
        return view;
    }
}
```

**[View.interface] : render( )메소드에 모델 데이터를 인자로 전달 가능하도록 변경**

```java
public interface View {
    void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```

**[JspView.java]**

```java
public class JspView implements View {
    private static final String DEFAULT_REDIRECT_PREFIX = "redirect:";

    private String viewName;

    public JspView(String viewName) {
        if (viewName == null) {
            throw new NullPointerException("viewName is null. 이동할 URL을 입력하세요.");
        }
        this.viewName = viewName;
    }

    @Override
    public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response)
            throws Exception {
        if (viewName.startsWith(DEFAULT_REDIRECT_PREFIX)) {
            response.sendRedirect(viewName.substring(DEFAULT_REDIRECT_PREFIX.length()));
            return;
        }

        Set<String> keys = model.keySet();
        for (String key : keys) {
            request.setAttribute(key, model.get(key));
        }

        RequestDispatcher rd = request.getRequestDispatcher(viewName);
        rd.forward(request, response);
    }
}
```

**[JsonView.java]**

```java
public class JsonView implements View {
    @Override
    public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response)
            throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        response.setContentType("application/json;charset=UTF-8");
        PrintWriter out = response.getWriter();
        out.print(mapper.writeValueAsString(model));
    }
}
```

**[Controller.java] : 인터페이스 반환값을 View → ModelAndView로 수정**

```java
public interface Controller {
    ModelAndView execute(HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```

**[AbstractController.java] : ModelAndView생성을 도와준다. 반환값 ModelAndView이다.**

```java
public abstract class AbstractController implements Controller {
    protected ModelAndView jspView(String forwardUrl) {
        return new ModelAndView(new JspView(forwardUrl));
    }

    protected ModelAndView jsonView() {
        return new ModelAndView(new JsonView());
    }
}
```

**[HomeController.java]**

```java
public class HomeController extends AbstractController {
    private QuestionDao questionDao = new QuestionDao();

    @Override
    public ModelAndView execute(HttpServletRequest request, HttpServletResponse response) throws Exception {
        return jspView("home.jsp").addObject("questions", questionDao.findAll());
    }
}
```

**[AddAnswerController.java]**

```java
public class AddAnswerController extends AbstractController {
    private static final Logger log = LoggerFactory.getLogger(AddAnswerController.class);

    private AnswerDao answerDao = new AnswerDao();

    @Override
    public ModelAndView execute(HttpServletRequest req, HttpServletResponse response) throws Exception {
        Answer answer = new Answer(req.getParameter("writer"), req.getParameter("contents"),
                Long.parseLong(req.getParameter("questionId")));
        log.debug("answer : {}", answer);

        Answer savedAnswer = answerDao.insert(answer);
        return jsonView().addObject("answer", savedAnswer);
    }
}
```

**[DeleteAnswerController.java]**

```java
public class DeleteAnswerController extends AbstractController {
    private AnswerDao answerDao = new AnswerDao();

    @Override
    public ModelAndView execute(HttpServletRequest request, HttpServletResponse response) throws Exception {
        Long answerId = Long.parseLong(request.getParameter("answerId"));

        ModelAndView mav = jsonView();
        try {
            answerDao.delete(answerId);
            mav.addObject("result", Result.ok());
        } catch (DataAccessException e) {
            mav.addObject("result", Result.fail(e.getMessage()));
        }
        return mav;
    }
}
```

**[DispatcherServlet.java] : View가 아닌 ModelAndView를 사용하도록 리펙토링**

```java
@WebServlet(name = "dispatcher", urlPatterns = "/", loadOnStartup = 1)
public class DispatcherServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);

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

        Controller controller = rm.findController(req.getRequestURI());
        ModelAndView mav;
        try {
            mav = controller.execute(req, resp);
            View view = mav.getView();
            view.render(mav.getModel(), req, resp);
        } catch (Throwable e) {
            logger.error("Exception : {}", e);
            throw new ServletException(e.getMessage());
        }
    }
}
```

### 🤷‍♂️ 함수형 프로그래밍을 해야하는 이유?

---

코드의 안정성, 유지 관리 가능성, 테스트 가능성, 확장 성 및 모듈성과 같은 이점을 제공하기 때문.

깨끗하고 표현력이 좋은 코드를 만들기 좋다.

## 📌8.5 추가 학습 자료

### 8.5.1 REST API 설계 및 개발

---

- REST API란?
    
    네트워크 애플리케이션을 설계하기 위한 아키텍처 스타일
    
- REST API의 특성은?
    1. 리소스 : 모든 것이 리소스로 간주된다.
        
        리소스는 API를 통해 액세스하거나 조작할 수 있는 모든 정보가 될 수 있다.
        
    2. 상태 비저장 : 클라이언트에서 서버로의 각 요청에는 서버가 요청을 이해하고 처리하는 데 필요한 모든 정보가 포함
    3. 표현 : JSON 또는 XML과 같은 다양한 형식을 사용하여 표현 된다.
    4. 확장성 : 확장 가능하도록 설계되었으며 상태 비저장 특성과 균일한 인터페이스를 활용하여 많은 수의 동시 요청을 처리
