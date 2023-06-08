# 8장 AJAX를 활용해 새로고침 없이 데이터 갱신하기

직접 구현한 프레임워크 및 라이브러리와 AJAX 기술을 활용해 질문/답변 게시판에 답변을 추가 및 삭제 기능을 구현한다.

브라우저가 서버에서 HTML 응답을 받아 처리하는 과정은 다음과 같다.

1. 브라우저가 HTML 응답을 받으면 브라우저는 먼저 HTML을 라인 단위로 읽는다.
2. 서버에 재요청이 필요한 부분(CSS, 자바스크립트, 이미지 등)을 찾아 서버에 다시 요청을 보낸다.
3. 서버에서 자원을 다운로드하면서 HTML DOM 트리를 구성한다.
4. CSS 파일을 다운로드하면 앞에서 생성한 HTML DOM 트리에 CSS 스타일을 적용한 후 모니터 화면에 그린다.

이와 같이 사용자에게 화면을 보여줄 때까지 많은 단계를 거치며 많은 비용이 발생한다.

그렇다면 답변 추가 및 삭제 기능의 경우에는 매번 서버에 요청을 보내 위의 과정 전체를 실행하는 것보다 변경되는 부분만 처리하면 효율적일 것이다. 이 때, 사용하는 기술이 AJAX이다.

- **AJAX(Asynchronous JavaScript and XML)**

  - 장점

    - 페이지 이동없이 고속으로 화면을 전환할 수 있다.
    - 서버 처리를 기다리지 않고, 비동기 요청이 가능하다.
    - 수신하는 데이터 양을 줄일 수 있고, 클라이언트에게 처리를 위임할 수도 있다.
    - 플러그인 없이도 인터렉티브한 웹페이지 구현할 수 있다.

  - 단점

    - Ajax를 쓸 수 없는 브라우저에 대한 문제가 있다.
    - HTTP 클라이언트의 기능이 한정되어 있다.
    - 페이지 이동없는 통신으로 인한 보안상의 문제가 있다.
    - 지원하는 문자집합(Charset)이 한정되어 있다.
    - 스크립트로 작성되므로 디버깅이 용이하지 않다.
    - 요청을 남발하면 역으로 서버 부하가 늘 수 있다.
    - 동일-출처 정책으로 인해 다른 도메인과는 통신이 불가능하다.

## 8.2 AJAX 활용해 답변 추가, 삭제 실습

### 8.2.1 답변하기

1. 사용자가 답변하기 버튼을 클릭.
2. 사용자가 입력한 데이터를 서버로 전송.
3. 서버는 사용자가 입력한 데이터를 데이터베이스에 저장.
4. 저장한 데이터를 클라이언트에 JSON 형태로 전송.
5. 클라이언트는 서버가 응답한 JSON 데이터를 HTML로 변환해 화면에 출력.

### HTML의 버튼

```html
<div class="answerWrite">
  <form name="answer" method="post">
    <input type="hidden" name="questionId" value="${question.questionId}" />
    <div class="form-group col-lg-4" style="padding-top:10px;">
      <input class="form-control" id="writer" name="writer" placeholder="이름" />
    </div>
    <div class="form-group col-lg-12">
      <textarea name="contents" id="contents" class="form-control" placeholder=""></textarea>
    </div>
    <input class="btn btn-success pull-right" type="submit" value="답변하기" />
    <div class="clearfix" />
  </form>
</div>
```

### 답변하기 버튼의 클릭이벤트 -> addAnswer() 실행

```javascript
$(".answerWrite input[type=submit]").click(addAnswer);
```

### addAnswer()

```javascript
function addAnswer(e) {
  e.preventDefault();

  var queryString = $("form[name=answer]").serialize();

  $.ajax({
    type: "post",
    url: "/api/qna/addAnswer",
    data: queryString,
    dataType: "json",
    error: onError,
    success: onSuccess,
  });
}
```

### onSuccess()

```javascript
function onSuccess(json, status) {
  var answerTemplate = $("#answerTemplate").html();
  var template = answerTemplate.format(
    json.writer,
    new Date(json.createdDate),
    json.contents,
    json.answerId,
    json.answerId
  );
  $(".qna-comment-slipp-articles").prepend(template);
}
```

### onError()

```javascript
function onError(xhr, status) {
  alert("error");
}
```

- jQuery의 ajax() 함수를 활용해 서버에 요청을 보냄.
- 요청 메소드는 POST, 요청 URL은 "/api/qna/addAnswer"
- 응답데이터 타입은 JSON
- 서버에 응답이 성공하면 onSuccess() 함수를 호출하면서 응답 데이터를 전달받음.
- 서버에 응답이 실패하면 onError() 함수를 호출하면서 실패 원인을 전달받음.

### AddAnswerController

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
        resp.setContentType("application/json;charset=UTF-8");
        PrintWriter out = resp.getWriter();
        out.print(mapper.writeValueAsString(savedAnswer));
        return null;
    }
}
```

- 응답할 때 HTML이 아닌 JSON 형태로 데이터를 전달하기 위해 Jackson 라이브러리를 사용하여 자바 객체의 데이터를 JSON으로 변환하여 사용함.
- JSON 데이터 생성 후 바로 응답을 보내기 때문에 이동할 페이지가 없어 null을 반환.
- DispatcherServlet이 null처리를 하지 않고 있기 때문에 뷰 이름이 null인 경우 페이지 이동을 하지 않도록 null 처리를 해줌.

### RequestMapping

```java
public class RequestMapping {
    private Map<String, Controller> mappings = new HashMap<>();

    void initMapping() {
        mappings.put("/api/qna/addAnswer", new AddAnswerController());
    }
}
```

- RequestMapping "/api/qna/addAnswer" URL로 AddAnswerController를 등록함.
- 위와 같이 구현한 응답 결과는 다음과 같은 JSON 형태의 데이터로 클라이언트 자바스크립트에 전달됨.

```javascript
{"answer":6, "writer":"재성","contents":"테스트","createdDate":1457066690411,"questionId":8,"timeFromCreateDate":1457066690411}
```

- 이 JSON데이터는 onSuccess() 함수가 동적으로 HTML을 생성한 후 화면에 출력함.

```javascript
function onSuccess(json, status) {
  var answerTemplate = $("#answerTemplate").html();
  var template = answerTemplate.format(
    json.writer,
    new Date(json.createdDate),
    json.contents,
    json.answerId,
    json.answerId
  );
  $(".qna-comment-slipp-articles").prepend(template);
}
```

onSuccess() 함수가 간단하게 구현될 수 있는 이유는 간단한 HTML 템플릿과 이 템플릿에 값을 전달하는 template() 함수를 다음과 같이 구현했기 때문이다. (format() 함수는 webapp/js/scripts.js에 구현됨.)

```javascript
String.prototype.format = function () {
  var args = arguments;
  return this.replace(/{(\d+)}/g, function (match, number) {
    return typeof args[number] != "undefined" ? args[number] : match;
  });
};
```

다음과 같은 HTML 템플릿을 추가하면 위 template() 함수를 활용해 동적인 HTML을 간편하게 생성할 수 있다.

```html
<script type="text/template" id="answerTemplate">
  <article class="article">
  	<div class="article-header">
  		<div class="article-header-thumb">
  			<img src="https://graph.facebook.com/v2.3/1324855987/picture" class="article-author-thumb" alt="">
  		</div>
  		<div class="article-header-text">
  			{0}
  			<div class="article-header-time">{1}</div>
  		</div>
  	</div>
  	<div class="article-doc comment-doc">
  		{2}
  	</div>
  	<div class="article-util">
  	<ul class="article-util-list">
  		<li>
  			<a class="link-modify-article" href="/api/qna/updateAnswer/{3}">수정</a>
  		</li>
  		<li>
  			<form class="form-delete" action="/api/qna/deleteAnswer" method="POST">
  				<input type="hidden" name="answerId" value="{4}" />
  				<button type="submit" class="link-delete-article">삭제</button>
  			</form>
  		</li>
  	</ul>
  	</div>
  </article>
</script>
```

### 8.2.2 답변 삭제하기 실습

코드 참조

- **자바의 this와 자바스크립트의 this의 차이**

  - 자바에서의 this

    Java에서의 this는 객체 자신(self)을 가리키는 참조변수로, this가 객체 자신에 대한 참조 값을 가지고 있다는 뜻이다. 주로 매개변수와 객체 자신이 가지고 있는 멤버변수명이 같을 경우 이를 구분하기 위해서 사용된다.

  - 자바스크립트에서의 this

    자바스크립트의 경우 함수 호출 방식에 의해 this에 바인딩할 어떤 객체가 동적으로 결정된다.

    - 함수 호출

      내부함수는 일반함수, 메소드, 콜백함수 어디에서 선언되었는지에 관계없이 this는 전역객체를 바인딩한다.

    - call, apply, bind 메소드

      호출에 따라 바뀌는 this를 피하기 위해 명시적으로 this를 지정해주는 메소드이다.

    - 생성자 함수 호출

      JavaScript의 생성자 함수는 JAVA와 같은 객체지향 언어의 생성자 함수와 다르게 그 형식이 정해져 있는 것이 아닌, 기존 함수에 new연산자를 붙여서 호출하면 해당 함수가 생성자 함수로 동작한다.

## 8.4 MVC 프레임워크 구현 2단계

**AddAnswerController의 2가지 문제점**

- JSON으로 응답을 보내는 경우 이동할 JSP 페이지가 없어 불필요하게 null을 반환해야 한다.

  - 컨트롤러에서 응답할 뷰가 JSP 하나에서 JSP와 JSON 두 가지로 증가했기 때문이다. 뷰가 JSP(또는 서블릿)일 경우는 String을 항상 반환해야 했지만 JSON일 경우는 반환 값이 필요없다. 이 문제를 해결하는 가장 쉬운 방법은 DispatcherServlet에서 execute() 메소드의 반환 값이 null일 때 아무런 일도 하지 않도록 if/else 형태로 구현할 수 있다. 그렇다면 이후에 PDF, 엑셀과 같은 뷰가 추가된다면 또 다시 그에 따른 예외 처리를 해야한다. 쉬운 해결책이기는 하지만 근본적인 해결책은 될 수 없다.

- AddAnswerController와 DeleteAnswerController에서 자바 객체를 JSON으로 변환하고 응답하는 부분에서 중복이 발생한다.

  - View 인터페이스를 추가해 구조를 변경한다. 지금까지는 뷰가 JSP 하나였는데 JSON 응답을 담당하는 다른 종류의 뷰가 하나 추가되었기 때문에 이를 추상화한 View 인터페이스를 추가한다.

### 8.4.1 View 인터페이스 추가

JSP와 JSON 뷰를 추상화한 View 인터페이스를 추가한다.

```java
public interface View {
    void render(HttpServletRequest req, HttpServletResponse resp) throws Exception;
}
```

### 8.4.2 JspView와 JsonView 추가

**JspView**

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
    public void render(HttpServletRequest req, HttpServletResponse resp) throws Exception {
        if (viewName.startsWith(DEFAULT_REDIRECT_PREFIX)) {
            response.sendRedirect(viewName.substring(DEFAULT_REDIRECT_PREFIX.length()));
            return;
        }

        RequestDispatcher rd = request.getRequestDispatcher(viewName);
        rd.forward(request, response);
    }
}
```

- 이동할 뷰 이름을 생성자로 받은 후 render() 메소드 호출 시 해당 페이지로 이동한다. 이 부분은 DispatcherServlet의 move 메소드 구현 부분을 render() 메소드에 구현하면 된다.

**JsonView**

```java
public class JsonView implements View {
    @Override
    public void render(HttpServletRequest req, HttpServletResponse resp) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        response.setContentType("application/json;charset=UTF-8");
        PrintWriter out = response.getWriter();
        out.print(mapper.writeValueAsString(model));
    }
}
```

- JsonView는 이동할 URL이 없다. render() 메소드는 HttpServletRequest를 통해 전달되는 자바 객체를 JSON으로 변환한 후 응답하는 기능을 가지도록 구현한다.
- 이와 같이 구현하고 보니 MVC 프레임워크가 가지고 있던 두 번째 문제인 중복 코드를 제거하는 효과까지 볼 수 있게 되었다.

### 8.4.3 Controller 반환 값을 String에서 View로 수정

더 이상 Controller가 String을 반환하지 않아도 된다. View를 반환하도록 수정한다.

```java
public interface Controller {
    View execute(HttpServletRequest req, HttpServletResponse resp) throws Exception;
}
```

### 8.4.4 Controller 구현체가 String 대신 View를 반환하도록 수정

컴파일 에러를 해결하기 위해 먼저 모든 Controller 구현체가 String 대신 View를 반환하도록 수정한다. 변경 작업은 간단하다. String으로 반환하던 부분은 JspView 또는 JsonView로 반환하도록 수정한다.

```java
public class HomeController extends Controller {
		@Override
    public View execute(HttpServletRequest req, HttpServletResponse resp) throws Exception {
			QuestionDao questionDao = new QuestionDao();
			req.setAttribute("questions", questionDao.findAll());
      return new JspView("home.jsp");
    }
}
```

```java
public class AddAnswerController extends Controller {
    private static final Logger log = LoggerFactory.getLogger(AddAnswerController.class);

    private AnswerDao answerDao = new AnswerDao();

    @Override
    public View execute(HttpServletRequest req, HttpServletResponse resp) throws Exception {
        Answer answer = new Answer(req.getParameter("writer"), req.getParameter("contents"),
                Long.parseLong(req.getParameter("questionId")));
        log.debug("answer : {}", answer);

				AnswerDao answerDao = new AnswerDao();
        Answer savedAnswer = answerDao.insert(answer);
				req.setAttribute("answer", savedAnswer);
        return new JsonView();
    }
}
```

Json 데이터를 반환하는 AddAnswerController의 경우 자바 객체를 JSON 데이터로 변경하는 부분이 JsonView로 위임했기 때문에 훨씬 더 간단해졌다. 위와 같이 View를 추가하고 변경한 결과 컨트롤러를 일관된 구조하에서 개발할 수 있게 되었다.

### 8.4.5 DispatcherServlet이 View에 작업을 위임하도록 수정

마지막으로 수정할 부분은 DispatcherServlet에서 String 값을 받아 처리하는 작업을 View를 활용하도록 수정하면 된다.

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

        Controller controller = rm.findController(requestUri);
        try {
            View view = controller.execute(req, resp);
            view.render(req, resp);
        } catch (Throwable e) {
            logger.error("Exception : {}", e);
            throw new ServletException(e.getMessage());
        }
    }
}
```

DispacherServlet에서 처리하던 페이지 이동 작업을 JspView의 render() 메소드로 이동했기 때문에 이전보다 더 깔끔한 코드가 되었다.

지금까지의 과정을 통해 뷰가 다양해지면서 발생하던 MVC 프레임워크의 문제점을 해결했다. 이처럼 경우의 수가 하나가 아닌 2개 이상이 발생하는 경우 if/else를 통해 해결할 수 있지만 그보다는 인터페이스를 통해 문제를 해결하는 것이 좀 더 확장 가능하고 깔끔한 코드를 구현할 수 있다.

### 8.4.6 ModelAndView 추가를 통한 모델 추상화

뷰를 포함해 모델 데이터에 대한 추상화를 담당하는 ModelAndView를 다음과 같이 구현한다.

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

View의 render() 메소드에 모델 데이터를 인자로 전달할 수 있도록 변경한다.

```java
public interface View {
    void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```

View 인터페이스 변경에 따라 JspView와 JsonView를 다음과 같이 수정한다.

**JspView**

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

**JsonView**

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

Controller 인터페이스의 반환 값을 View에서 ModelAndView를 반환하도록 수정한다.

```java
public interface Controller {
    ModelAndView execute(HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```

ModelAndView 생성을 좀 더 쉽도록 도와주기 위해 AbstractController를 추가한 후 다음 두개의 메소드를 제공한다.

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

Controller 구현체가 View가 아닌 ModelAndView를 반환하도록 수정한다. 지금까지 HttpServletRequest를 통해 전달하던 모델 데이터를 ModelAndView를 통해 전달하도록 수정한다. 앞에서 살펴본 HomeController와 AddAnswerController 코드를 살펴보면 다음과 같다.

```java
public class HomeController extends AbstractController {
    private QuestionDao questionDao = new QuestionDao();

    @Override
    public ModelAndView execute(HttpServletRequest request, HttpServletResponse response) throws Exception {
        return jspView("home.jsp").addObject("questions", questionDao.findAll());
    }
}
```

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

Controller 구현체를 모두 수정했으면 마지막 작업은 DispatcherServlet이 View가 아닌 ModelAndView를 사용하도록 리팩토링한다.

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

## 8.5 추가 학습 자료

### 8.5.1 REST API 설계 및 개발
