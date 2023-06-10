# 자뿌 8장

# 8장 - AJAX를 활용해 새로고침 없이 데이터 갱신하기

- 7장 까지 해오던 HTML을 이용한 데이터 처리를 이제는 AJAX로 해보자
- 그래서 이젠 JSON 형태의 데이터를 사용해서 처리를 할 것

---

- HTML은 라인 단위로 읽어서 명령어를 확인하고 CSS, JS, Img 등 을 찾아 다시 서버에 요청을 보낸다.
- 서버에서 CSS 파일을 다운 → 앞에서 생성한 HTML DOM 트리에 CSS 스타일 적용 후 화면에 렌더링 하게되는 구조.
- **→ 많은 단계를 거치게 되고, 많은 비용이 발생하게 됨**

간단한 추가, 삭제 기능에 대해서는 모든 화면이 바뀔 필요가 없음. 즉, 화면 대부분은 변경할 필요가 없다는 것. 추가 삭제에 해당하는 영역만 처리를 하면 됨. 즉, 매번 서버에 요청을 보내서 위에서 했던 과정 전체를 하는 것은 매우 비효율적이라고 말할 수 있음.

→ 이런 단점을 보완하기 위해 **AJAX(Asyn JavaScript and XML)** 이 등장하게 되었음.

![image](https://github.com/BookBBu/JavaWebProgrammingNextStep/assets/44702967/1e077b51-11f7-425c-9dbd-c6fd2ae6fa0e)

![image](https://github.com/BookBBu/JavaWebProgrammingNextStep/assets/44702967/39518803-0d84-4c01-9190-873241b1f9be)

- **AJAX 작동원리**
    
    ① : 사용자에 의한 **요청 이벤트**가 발생합니다.
    
    ② : 요청 이벤트가 발생하면 **이벤트 핸들러에 의해 자바스크립트가 호출**됩니다.
    
    ③ : 자바스크립트는 **XMLHttpRequest 객체를 사용하여 서버로 요청**을 보냅니다.
    
    - 이때 웹 브라우저는 요청을 보내고 나서, 서버의 응답을 기다릴 필요 없이 다른 작업을 처리할 수 있습니다.
    
    ④ : 서버는 전달받은 XMLHttpRequest 객체를 가지고 **Ajax 요청을 처리**합니다.
    
    ⑤와 ⑥ : 서버는 처리한 결과를 HTML, XML 또는 JSON 형태의 데이터로 웹 브라우저에 전달합니다. 이때 전달되는 응답은 새로운 페이지를 전부 보내는 것이 아니라 **필요한 데이터만을 전달**합니다.
    
    ⑦ : 서버로부터 전달받은 데이터를 가지고 **웹 페이지의 일부분만을 갱신하는 자바스크립트를 호출**합니다.
    
    ⑧ : 결과적으로 **웹 페이지의 일부분만이 다시 로딩되어 표시**됩니다.
    
    [코딩교육 티씨피스쿨](http://www.tcpschool.com/ajax/intro)
    

- 프론트에서 열심히 하던 axios

## 8.2 - AJAX 를 활용해 답변 추가, 삭제 실습

### 8.2.1 답변 추가 하기

- 답변하기 버튼 클릭 → 입력 데이터를 서버로 전송 → 데이터를 DB에 저장 → 저장한 데이터를 클라이언트에 JSON으로 전송 → 클라이언트는 서버가 응답해준 JSON 데이터를 HTML로 변환해 화면에 출력
- 지금은 `<form>` 형식으로 `method="post"` `<input>` 을 submit 하고 있다.

```jsx
$(".answerWrite input[type=submit]").click(addAnswer);
```

JS 로 구현한 단순 클릭 이벤트이다. 

클릭 이벤트를 발동시킬 때… submit의 기본 동작을 막고, form 태그에서 입력한 값을 서버에 전송할 수 있게 할꺼다

```jsx
function addAnswer(e) {
  e.preventDefault();   //submit 이 자동으로 동작하는 것을 방지한다.
  var queryString = $("form[name=answer]").serialize();
  //.serialize() 메서드는 표준 URL 인코딩 표기법으로 텍스트 문자열을 생성

	$.ajax({
    type : 'post',
    url : '/api/qna/addAnswer',
    data : queryString,
    dataType : 'json',
    error: onError,
    success : onSuccess,
  });
}
```

- ajax 함수로 서버에 요청을 보낸다. 메서드나 url 이런 것들은 위에 적힌거대로 간다.
- 성공, 실패했을 때onError, onSuccess를 구현해야겠다.

```jsx
function onSuccess(json, status){
  var answerTemplate = $("#answerTemplate").html();
  var template = answerTemplate.format(json.writer, new Date(json.createdDate), json.contents, json.answerId, json.answerId);
  $(".qna-comment-slipp-articles").prepend(template);
}

function onError(xhr, status) {
  alert("error");
}
```

그럼 클라이언트에서 넘어온 요청을 서버가 처리를 해야할텐데 그걸 구현해야한다.

```jsx
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
        resp.setContentType(**"**application/json;charset=UTF-8");
        PrintWriter out = resp.getWriter();
        out.print(mapper.writeValueAsString(savedAnswer));
        return null;
    }
}
```

- `ObjectMapper` 는 JSON 형태로 바꿔주는 라이브러리다.
- `out.print(mapper.writeValueAsString(savedAnswer));`
- 근데 지금 `return null;` ? 의미 없는게 있다 일단 인지하고 있어라. return 은 JSP 페이지 옮길 때 필요한데 지금 JSON 으로 처리하는 과정에서 페이지를 옮길일이 없다 그래서 null임.

url을 `/api/qna/addAnswer` 이렇게 넣었으니 서버쪽 컨트롤러에도 추가를 해야한다.

```jsx
public class RequestMapping {
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);
    private Map<String, Controller> mappings = new HashMap<>();

    void initMapping() {
        ...
        **mappings.put("/api/qna/addAnswer", new AddAnswerController());**
        ...
	}
}
```

답변을 추가했으면 삭제하는 기능도 만들어야겠다.

### 8.2.2 답변 삭제하기 실습

거의 뭐 비슷하다. 근데 이 파트에서 `this` 에 대해 다루고 있는데, Java와 JS의 this는 다르다.

- 자세히 알아보기
    
    **JAVA**
    자바에서 `this`는 현재 인스턴스를 참조하는 데 사용된다. 클래스의 인스턴스 메서드 내에서 **`this`**를 사용하면 해당 메서드를 호출한 인스턴스를 가리킨다. **`this`**를 사용하여 인스턴스 변수에 접근하거나 인스턴스 메서드를 호출할 수 있겠다
    
    ```java
    public class MyClass {
        private int value;
    
        public void setValue(int value) {
            this.value = value;
        }
    
        public int getValue() {
            return this.value;
        }
    }
    
    ```
    
    위의 코드에서*`this.value`는 현재 인스턴스의 **`value`** 변수를 참조함
    
    **JS**
    자바스크립트에서 `this`는 **호출된 함수의 컨텍스트**를 참조한다. `this`의 값은 함수를 호출하는 방법에 따라 결정되겠다.
    
    일반적으로 메서드 내에서 객체를 참조하기 위해 `this`를 사용하는데 메서드 내부에서 `this`는 해당 메서드가 속한 객체를 참조한다. 그러나 `this`의 값은 호출 방법에 따라 다를 수 있음.
    
    다음은 자바스크립트에서 `this`를 사용한 예다:
    
    ```jsx
    const obj = {
        value: 10,
        getValue: function() {
            return this.value;
        }
    };
    
    console.log(obj.getValue()); // 출력: 10
    ```
    
    위의 코드에서 `obj.getValue()`를 호출할 때 `this`는 `obj` 객체를 참조하므로 `this.value`는 10을 반환합니다.
    
    하지만 함수를 다른 방식으로 호출하면 `this`의 값이 달라질 수 있다. 예를 들어, 다음과 같이 함수를 변수에 할당하고 호출하는 경우:
    
    ```jsx
    const getValue = obj.getValue;
    console.log(getValue()); // 출력: undefined (or 에러 발생)
    ```
    
    위의 코드에서 `getValue` 함수를 변수에 할당하면 `this`는 더 이상 `obj`를 참조하지 않고 전역 객체를 참조하게 됨. 따라서 `this.value`는 정의되지 않은 변수를 참조하게 되므로 `undefined`를 반환하거나 에러가 발생할 수 있음
    
    자바스크립트에서 `this`의 동작은 호출한 방식(메서드, 생성자, 함수 등)과 함께 바인딩 규칙을 따르므로 주의해야 함
    
    자바스크립트의 **call, apply, bind 에 대해서 각자 알아보자**
    

`<form>` 태그에 맞게 클릭 이벤트를 달고

```jsx
$(".qna-comment").on("click", ".form-delete", deleteAnswer);
```

```jsx
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
```

거의 똑같다. 달라진 점은 sucesss할 때 인데 그냥 버튼과 가장 가까운 위치에 있는 <article> 태그를 지우기 위한 코드가 들어간 것 뿐이다.

다음 요구사항을 해결하러 가자

### 8.3 MVC 프레임워크 요구사항 2단계

AJAX를 이제 지금와서 막 도입하려고하니 이때까지 만들었던 MVC 프레임워크 구조에 허점이 많아졌다. 이제 뒷처리를 해야한다.

Delete 관련 컨트롤러를 통해서 어떤 부분을 봐야할지 고민해보고 한번 손보면 좋을 것 같다.

```jsx
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
				// JSON 응답을 클라이언트로 전송
        return null;
    }
}
```

아까 Add 랑 다를게 거의 없다.

참고로 `resp.setContentType("application/json;charset=UTF-8");`

응답의 컨텐츠 타입을 "application/json"으로 설정하고, 문자 인코딩을 "UTF-8"로 지정하는 것임. 클라이언트에게 JSON 형식의 응답을 제공한다고 말하는거

불편했던 `return null;`을 드디어 고쳐본다.

근데 add랑 delete 컨트롤러도 잘 찾아보면 이 부분도 중복된다. 리팩토링 해야할 게 생겨버렸다.

```jsx
        ObjectMapper mapper = new ObjectMapper();
        resp.setContentType(**"**application/json;charset=UTF-8");
        PrintWriter out = resp.getWriter();
        out.print(mapper.writeValueAsString(savedAnswer));
```

`return null;`

- 갑자기 필요 없어진 이유
    - JSP 페이지 옮길 때 썼는데요.. 이젠 JSON으로 할 경우는 반환값이 필요없기 때문에 필요가 없어졌다
    - 즉 JSP에서 JSP를 리턴했었다면 이젠 JSP에서 JSP 또는 JSON 을 처리해버리게 되어서 그렇다.
- 이 책 쓴사람이 제안하는 방법
    - View 인터페이스를 추가하는 것이다
        - JspView와 JsonView를 이용해 컨트롤러에서 전달하는 모델 데이터를 활용해 응답하도록 개선한 것이다.
    - 또한 모델 데이터를 HttpServletRequest 로 전달하는 모델 데이터 대신에 Map 을 활용해 전달하도록 했다.
    - 또 Model과 View 를 추상화한 ModelAndView 를 추가해 Controller 인터페이스를
        
        ```jsx
        public interface Controller {
            ModelAndView execute(HttpServletRequest request, HttpServletResponse response) 
        		throws Exception;
        }
        ```
        
        이렇게 바꿨다고 한다.
        

힌트는 여기까지고 직접 구현해보자

### 8.4 MVC 프레임워크 구현 2단계

**View 인터페이스 부터 정리를 해보자**

```jsx
public interface View {
    void render(HttpServletRequest request, HttpServletResponse response) 
		throws Exception;
}
```

- JSP와 JSON 뷰를 추상화하기 위해 View 인터페이스를 먼저 만든다

**JspView와 JsonView추가**

```jsx
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

- JSP는 진짜 그냥 페이지로 이동하면 된다.
    - `response.sendRedirect(viewName.substring(DEFAULT_REDIRECT_PREFIX.length()));`
        - `viewName`이 `redirect:`로 시작하는 경우, `response.sendRedirect`를 사용하여 리다이렉트를 처리하고 메서드를 종료
    - `rd.forward(req, response);`
        - `viewName`이 `DEFAULT_REDIRECT_PREFIX`로 시작하지 않는 경우, `RequestDispatcher`를 사용하여 `viewName`에 해당하는 JSP 파일로의 포워드

**JsonView**

```java
public class JsonView implements View {
    @Override
    public void render(HttpServletRequest req, HttpServletResponse response)
            throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        response.setContentType("application/json;charset=UTF-8");
        PrintWriter out = response.getWriter();
        out.print(mapper.writeValueAsString(createModel(req));
				// JSON 작업 부분
				// 리턴이 어디갔누 없어졌다
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
	  // Map으로 모델 데이터를 만들고 return 한다.
}
```

- JspView랑 구조가 다르다. 당연하다. 하는 일이 다르니.
- `return null;` 없앴다.
- 덩달아 `mapper` 관련 중복코드도 없앴다.

**Controller 반환 값을 String 에서 View 로 수정해보자**

컨트롤러가 더 이상 JSP 페이지 바꿀려고 String을 리턴안해도 되고 View 를 리턴하면 되기 때문이다.

```java
public interface Controller {
   View execute(HttpServletRequest request, HttpServletResponse response) 
		throws Exception;
}
```

그럼 기존에 컨트롤러로 `return "home.jsp"` 했던 코드들을 다 바꿔야 한다. String → View 로 바꿨으니까

```java
public class HomeController extends AbstractController {
    @Override
    public View execute(HttpServletRequest request, HttpServletResponse response) 
	throws Exception {
        **return new JspView("home.jsp")**
    }
}
```

JsonView 관련 컨트롤러는

```java
public class AddAnswerController extends AbstractController {
    private static final Logger log = LoggerFactory.getLogger(AddAnswerController.class);

    @Override
    public View execute(HttpServletRequest req, HttpServletResponse response) throws Exception {
        Answer answer = new Answer(req.getParameter("writer"), req.getParameter("contents"),
                Long.parseLong(req.getParameter("questionId")));
        log.debug("answer : {}", answer);

				AnswerDao answerDao = new AnswerDao();
        Answer savedAnswer = answerDao.insert(answer);
				req.setAttribute("answer", savedAnswer);
        **return new JsonView();**
    }
}
```

아까 JSON 데이터화 하는 부분을 JsonView로 위임했었다.

**DispatcherServlet이 View에 작업을 위임하도록 수정하자. String 값 대신에**

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
            **View view = controller.execute(req, resp)
            view.render(req, resp);**
        } catch (Throwable e) {
            logger.error("Exception : {}", e);
            throw new ServletException(e.getMessage());
        }
    }
}
```

**마지막으로 Model 데이터에 대한 추상화를 담당하는 ModelAndView 를 구현해보자**

```java
public class ModelAndView {
    private View view;
    **private Map<String, Object> model = new HashMap<>();**

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

- Map 을 이용해 Model을 할당하는 코드가 있다.

그럼 View의 `render` 메소드에 모델 데이터를 파라미터로 전달할 수 있게 변경한다

```java
public interface View {
    void render(**Map<String, ?> model**, HttpServletRequest request, HttpServletResponse response) 
		throws Exception;
}
```

그럼… View 인터페이스 파라미터가 바꼈으니까 앞에서 View로 구현했던 redner를 일부 바꿔야하겠다.

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
    public void render(**Map<String, ?> model**, HttpServletRequest request, HttpServletResponse response)
            throws Exception {
        if (viewName.startsWith(DEFAULT_REDIRECT_PREFIX)) {
            response.sendRedirect(viewName.substring(DEFAULT_REDIRECT_PREFIX.length()));
            return;
        }

        **Set<String> keys = model.keySet();
        for (String key : keys) {
            request.setAttribute(key, model.get(key));
        } // 객체를 "객체명" : 객체로 보내는 코드이다.**

        RequestDispatcher rd = request.getRequestDispatcher(viewName);
        rd.forward(request, response);
    }
}
```

**JsonView**

```java
public class JsonView implements View {
    @Override
    public void render(Map<String, ?> **model**, HttpServletRequest request, HttpServletResponse response)
            throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        response.setContentType("application/json;charset=UTF-8");
        PrintWriter out = response.getWriter();
        **out.print(mapper.writeValueAsString(model));
				// Json에 그냥 Map을 보내면 우리가 익숙하게 봤던 그 Json 형태로 해준다. 
				// {"name":"hello","age":"99"}**
    }
}
```

컨트롤러 인터페이스 리턴 값도 View 에서 ModelAndView로 바꿔준다

```java
public interface Controller {
    **ModelAndView** execute(HttpServletRequest request, HttpServletResponse response) throws Exception;
}
```

이런 ModelAndView를 생성하기 좀 더 쉽게 해주기위해 하나의 추상컨트롤러를 만들어준다.

```java
public abstract class AbstractController implements Controller {
    protected **ModelAndView** **jspView**(String forwardUrl) {
        return new ModelAndView(new JspView(forwardUrl));
    }

    protected **ModelAndView jsonView(**) {
        return new ModelAndView(new JsonView());
    }
}
```

그럼 이제 또 View로 해놨던 execute 메서드 들을 전부 ModelAndView를 통해 전달하도록 한다.

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
        **return jsonView().addObject("answer", savedAnswer);**
    }
}
```

DispatcherServlet도 View로 사용하던걸 ModelAndView로 바꿔준다.

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
            **mav = controller.execute(req, resp);
            View view = mav.getView();
            view.render(mav.getModel(), req, resp);**
        } catch (Throwable e) {
            logger.error("Exception : {}", e);
            throw new ServletException(e.getMessage());
        }
    }
}
```

리팩토링 하면은 자꾸 자꾸 바뀌는 것을 알 수 있다.
그래서 시작할 때 모든 요구사항을 파악해서 설계하고 바로 구현할 수 있다면 정말 좋은데 현실적으로 불가능하다. 그렇기 떄문에 직므 가능한 최선의 설계와 구현을 하고 추후 요구사항이 변경됐을 때 빠르게 대응할 수 있는 능력을 갖추는 것이 중요하다고 한다. 즉 리팩토링과 테스트 능력이 중요하다는 것.

**API 를 설계할 때 RESTful API 라고 많이들 얘기가 나온다.**
책에 있는 링크의 REST 관련 pdf 이다. 내용은
- tmi 주의
    1. **API란 무엇인가?** - API는 애플리케이션 프로그래밍 인터페이스의 약자로, 소프트웨어 컴포넌트 간에 상호 작용을 가능하게 하는 규약입니다. API는 데이터와 기능에 대한 액세스를 제공하며, 이를 통해 개발자들이 새로운 애플리케이션을 만들 수 있습니다.
    2. **API 디자인의 중요성** - API 디자인은 사용자 경험(UX) 디자인과 비슷한 중요성을 가지고 있습니다. 잘 디자인된 API는 개발자가 쉽게 이해하고 사용할 수 있으며, 이는 결국 더 나은 소프트웨어를 만드는 데 도움이 됩니다.
    3. **RESTful API 디자인** - REST는 웹 서비스를 위한 아키텍처 스타일입니다. RESTful API는 HTTP 메서드(GET, POST, PUT, DELETE 등)를 사용하여 리소스에 액세스하고, 이를 통해 클라이언트와 서버 간의 상호 작용을 단순화합니다.
    4. **API 버전 관리** - API는 시간이 지남에 따라 변화하고 발전합니다. 이러한 변화를 관리하기 위해 API 버전 관리 전략이 필요합니다. 이는 이전 버전의 API를 사용하는 애플리케이션을 보호하면서 새로운 기능을 추가할 수 있게 해줍니다.
    5. **API 보안** - API는 종종 민감한 데이터에 액세스하므로, 보안은 중요한 고려사항입니다. API 보안 전략은 인증, 권한 부여, 데이터 암호화 등을 포함해야 합니다.
    6. **API 문서화** - 문서화는 API의 사용성을 크게 향상시킵니다. API 문서는 API의 기능, 사용 방법, 예제 등을 제공해야 합니다.

일단 REST API url 명명 규칙은

- 소문자로
- 언더바 X → 긴 경로는 하이픈으로 (-)
- 마지막에 `/` X
- 동사 X
- 명사 O → 복수형, 자원같은 것들 ㅇㅇ
- 동사 대신에 GET, POST, PUT, DELETE 메서드로 동사 표현을 낸다.
