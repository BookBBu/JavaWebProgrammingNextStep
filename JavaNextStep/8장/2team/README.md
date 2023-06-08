# 8장 AJAX를 활용해 새로고침 없이 데이터 갱신하기

## 8.1 질문/답변 게시판 구현
## 8.2 AJAX 활용 답변 추가, 삭제

### 1. 브라우저가 서버에서 HTML 응답을 받아 처리하는 과정


1) HTML 응답을 받으면 브라우저는 HTML을 라인 단위로 읽어 내려가면서 서버에 재 요청이 필요한 부분을 찾아 서버에 다시 요청을 보낸다.

2) 서버에서 자원을 다운로드하면서 HTML DOM 트리를 구성한다.

3) CSS 파일을 다운로드하면 앞에서 생성한 HTML DOM 트리에 CSS 스타일을 적용한 후 모니터 화면에 그린다.

…..

→ 서버에 요청을 보내고 응답을 받아 사용자에게 화면을 보여줄 때까지 많은 단계를 거치고, 많은 비용이 발생한다.

→ 이를 보안하기 위해 등장한 기술 AJAX

### 2. AJAX 장단점


**장점**

- 페이지 이동없이 고속으로 화면을 전환할 수 있다.
- 서버 처리를 기다리지 않고, 비동기 요청이 가능하다.
- 수신하는 데이터 양을 줄일 수 있고, 클라이언트에게 처리를 위임할 수도 있다.

**단점**

- Ajax를 쓸 수 없는 브라우저에 대한 문제가 있다.
- HTTP 클라이언트의 기능이 한정되어 있다.
- 페이지 이동없는 통신으로 인한 보안상의 문제
- 지원하는 문자집합(Charset)이 한정되어 있다.
- 스크립트로 작성되므로 디버깅이 용이하지 않다.
- 요청을 남발하면 역으로 서버 부하가 늘 수 있음.

## 8.2.1  답변하기

- 아래 코드로 answerWrite의 클릭 이벤트 발생 시 addAnswer() 함수 실행

```jsx
$(".answerWrite input[type=submit]").click(addAnswer);
```

  ```jsx
  function addAnswer(e) {
    e.preventDefault();   //submit 이 자동으로 동작하는 것을 막아줌

    var queryString = $("form[name=answer]").serialize();

    $.ajax({
      type : 'post',
      url : '/api/qna/addAnswer',
      data : queryString,
      dataType : 'json',
      error: onError,
      success : onSuccess,
    });
  }

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

- 클라이언트 요청을 처리하는 서버 코드

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

- request mapping

  ```java
  public class RequestMapping {
      private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);
      private Map<String, Controller> mappings = new HashMap<>();

      void initMapping() {
          ...
          mappings.put("/api/qna/addAnswer", new AddAnswerController());
          ...
  ```

## 8.2.2 답변 삭제하기

- 아래 코드로 answerWrite의 클릭 이벤트 발생 시 addAnswer() 함수 실행

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

- 클라이언트 요청을 처리하는 서버 코드

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

- request mapping

  ```java
  public class RequestMapping {
      private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);
      private Map<String, Controller> mappings = new HashMap<>();

      void initMapping() {
          ...
          mappings.put("/api/qna/deleteAnswer", new DeleteAnswerController());
          ...
  ```

  > java, javascript의  this
  > 
  > java : 자신의 객체에 접근할 때 사용
  > 
  > javascript : 해당 함수를 누가 호출 하는지에 따라 this가 달라짐(https://eunjinii.tistory.com/104)

  </aside>

# 8.3~8.4 MVC 프레임워크

### **8.2 코드의 문제점**

- JSON으로 응답을 보내는 경우 이동할 JSP페이지가 없다보니 불필요하게  null을 반환
- ..AnswerController 에서 자바 객체를 JSON으로 변환하고 응답하는 부분에서 중복이 발생

→ (추상화한)View 인터페이스를 추가하여 구조 변경

### 8.4.1 View 인터페이스 추가

  ```java
  public interface View {
      void render(HttpServletRequest request, HttpServletResponse response) throws Exception;
  }
  ```

### 8.4.2 JspView 와 JsonView 추가

- JSP와 JSON의 처리를 각각 알맞은 View 가 처리하도록 분리

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

- View 분리에 대한 효과
    - View 인터페이스를 구현한 View 구현체들은 사용자들이 원하는 View 반환을 선택할 수 있게 할 수 있다.
    - 다른 렌더링 방식을 원한다면 View 인터페이스를 구현하게되면 얼마든지 새로운 View를 추가할 수 있다.
    - DispatcherServlet에서 View 렌더링에 대한 책임을 각 View 구현체에게 위임하여 책임을 분리한다.
    - 각 View 구현체에게 렌더링에 대한 책임을 위임하여 Controller 코드가 간결해진다.
        - jsonView 도입 전
            
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
            
        - jsonView  도입 후
            
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
            

### 8.4.3 Controller 반환 값을 String에서 View로 수정

  ```java
  public interface Controller {
      ModelAndView execute(HttpServletRequest request, HttpServletResponse response) throws Exception;
  }
  ```

### 8.4.4 Controller 구현체가 String 대신 View를 반환 하도록 수정

- 반환값에 따라 JsonView, JspView로 반환 하도록 수정
    
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
    

### 8.4.5 DispatcherServlet이 View에 작업을 위임하도록 수정

- 기존
    
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
                if (viewName != null) {
                    move(viewName, req, resp);
                }
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
    

- 수정 후
    
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
    
    - 페이지 이동 작업이  JspView의 render() 메소드로 이동

### 8.4.6 ModelAndView 추가를 통한 모델 추상화

- ModelAndView → View를 포함해 모델 데이터에 대한 추상화를 담당
    
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
    
    - View
        
        ```java
        public interface View {
            void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception;
        }
        ```
        
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
        
    - Controller
        
        ```java
        public interface Controller {
            ModelAndView execute(HttpServletRequest request, HttpServletResponse response) throws Exception;
        }
        ```
        
        - ModelAndView 생성을 조금 더 쉽게 도와주기 위한 AbstractController
            
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
        
    - DispatcherServlet
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