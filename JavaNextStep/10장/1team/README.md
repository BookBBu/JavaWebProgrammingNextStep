# 10장 새로운 프레임워크 구현을 통한 점진적 개선

자바의 리플렉션과 어노테이션을 사용하여 기존의 레거시 MVC 프레임워크와 어노테이션 기반의 MVC 프레임 워크를 동시에 서비스 해보자!

## 10.1 MVC 프레임워크 요구사항 3단계

1. 새로운 기능이 추가 될 때 마다 컨트롤러를 추가하는 것이 아닌 메소드를 추가 해보자
2. URL을 매핑할때 HTTP 메소드를 활용하자
   -> 아래 코드와 같은 기능을 지원하는 어노테이션 기반 MVC 프레임워크를 만들어 보자!

```java
@Controller
public class MyController {
    private static final Logger logger = LoggerFactory.getLogger(MyController.class);

    @RequestMapping("/users")
    public ModelAndView list(HttpServletRequest request, HttpServletResponse response) {
        logger.debug("users findUserId");
        return new ModelAndView(new JspView("/users/list.jsp"));
    }

    @RequestMapping(value = "/users/show", method = RequestMethod.GET)
    public ModelAndView show(HttpServletRequest request, HttpServletResponse response) {
        logger.debug("users findUserId");
        return new ModelAndView(new JspView("/users/show.jsp"));
    }

    @RequestMapping(value = "/users", method = RequestMethod.POST)
    public ModelAndView create(HttpServletRequest request, HttpServletResponse response) {
        logger.debug("users create");
        return new ModelAndView(new JspView("redirect:/users"));
    }
}

```

- @Controller 어노테이션 구현

```java
package core.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
public @interface Controller {
    String value() default "";
}

```

- @RequestMapping 어노테이션 구현

```java
package core.annotation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target({ ElementType.METHOD, ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
public @interface RequestMapping {
    String value() default "";

    RequestMethod method() default RequestMethod.GET;
}
```

@Target : @interface 어노테이션의 적용 위치를 설정하는 옵션이다.

- Type : 클래스 및 인터페이스
- FIELD : 클래스의 멤버 변수
- METHOD : 함수
- PARAMETER : 파라미터
- CONSTRUCTURE : 생성자
- LOCAL_VARIABLE : 지역 변수
- ANNOTATION_TYPE : 어노테이션 타입
- PACKAGE : 패키지
- TYPE_PARAMETER : 타입 파라미터
- TYPE_USE : 타입 사용

@Retention : 어노테이션이 유지되는 시점

- RetentionPolicy.RUNTIME : 런타임까지
- RetentionPolicy.CLASS : 컴파일까지
- RetentionPolicy.SOURCE : 컴파일 전 소스레벨 까지

### 10.1.2. 자바 리플렉션

- 자바 리플렉션(Reflection) ?

  - 리플렉션은 구체적인 클래스 타입을 알지 못하더라도 그 클래스의 메서드, 타입, 변수들에 접근할 수 있도록 해주는 자바 API를 말한다.

  - 컴파일 시간이 아닌 실행 시간에 동적으로 특정 클래스의 정보를 추출할 수 있는 프로그래밍 기법이라 할 수 있다.

  - 실습 코드

  ```java
    @Test
    public void showClass() {
        //JVM의 클래스 로더에서 클래스 파일에 대한 로딩을 완료한 후, 해당 클래스의 정보를 담은 Class 타입의 객체를 생성하여 메모리의 힙 영역에 저장해 둔 것
        Class<Question> clazz = Question.class;
        logger.debug(clazz.getName());
    }
  ```

  ```java
    public class Junit3TestRunner {
        private static final Logger logger = LoggerFactory.getLogger(Junit3TestRunner.class);

        @Test
        public void run() throws Exception {
            Class<Junit3Test> clazz = Junit3Test.class;
            Method[] methods = clazz.getMethods();
            for(Method method : methods) {
                if (method.getName().startsWith("test")) {
                    method.invoke(clazz.getDeclaredConstructor().newInstance());
                }
            }
        }
    }
  ```

  ```java
    public class Junit4TestRunner {
    @Test
    public void run() throws Exception {
        Class<Junit4Test> clazz = Junit4Test.class;

        Method[] methods = clazz.getMethods();
        for(Method method : methods) {
            if(method.isAnnotationPresent(MyTest.class)) {
                method.invoke(clazz.getDeclaredConstructor().newInstance());
            }
        }
    }
  ```

  ```java
        @Test
    public void newInstanceWithConstructorArgs() throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Class<User> clazz = User.class;

        Constructor<User> cons = clazz.getDeclaredConstructor(String.class, String.class, String.class, String.class);
        User user = cons.newInstance("efef", "efef", "efef", "efef");

        logger.debug(clazz.getName());
        logger.debug(user.toString());
    }
  ```

  ```java
      @Test
    public void privateFieldAccess() throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Class<Student> clazz = Student.class;

        Student student = new Student();
        Field[] fields = clazz.getDeclaredFields();

        for (Field field : fields) {
            field.setAccessible(true);
            if (field.getType() == String.class) {
                field.set(student, "ㄱㄷㄱㄷ");
                continue;
            }

            if (field.getType() == Integer.class) {
                field.set(student, new Integer(32));
            }
        }

        logger.debug(clazz.getName());
        logger.debug(student.getName() + " " + student.getAge());
    }
  ```

### 10.2.3 클라이언트 요청에 해당하는 HandlerExecution 반환

-

```
public class AnnotationHandlerMapping implements HandlerMapping {
    private static final Logger logger = LoggerFactory.getLogger(AnnotationHandlerMapping.class);

    private Object[] basePackage;

    private Map<HandlerKey, HandlerExecution> handlerExecutions = Maps.newHashMap();

    public AnnotationHandlerMapping(Object... basePackage) {
        this.basePackage = basePackage;
    }

    public void initialize() {
        ControllerScanner controllerScanner = new ControllerScanner(basePackage);
        Map<Class<?>, Object> controllers = controllerScanner.getControllers();
        Set<Method> methods = getRequestMappingMethods(controllers.keySet());
        for (Method method : methods) {
            RequestMapping rm = method.getAnnotation(RequestMapping.class);
            logger.debug("register handlerExecution : url is {}, method is {}", rm.value(), method);
            handlerExecutions.put(createHandlerKey(rm),
                    new HandlerExecution(controllers.get(method.getDeclaringClass()), method));
        }

        logger.info("Initialized AnnotationHandlerMapping!");
    }
}

```

- initialize() : @controller 어노테이션이 붙은 클래스에서 @requestMapping이 붙은 메소드를 찾는다.

```java
public class HandlerKey {
    private String url;
    private RequestMethod requestMethod;

    public HandlerKey(String url, RequestMethod requestMethod) {
        this.url = url;
        this.requestMethod = requestMethod;
    }

    @Override
    public String toString() {
        return "HandlerKey [url=" + url + ", requestMethod=" + requestMethod + "]";
    }

    @Override
    public int hashCode() {
        final int prime = 31;
        int result = 1;
        result = prime * result + ((requestMethod == null) ? 0 : requestMethod.hashCode());
        result = prime * result + ((url == null) ? 0 : url.hashCode());
        return result;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj)
            return true;
        if (obj == null)
            return false;
        if (getClass() != obj.getClass())
            return false;
        HandlerKey other = (HandlerKey) obj;
        if (requestMethod != other.requestMethod)
            return false;
        if (url == null) {
            if (other.url != null)
                return false;
        } else if (!url.equals(other.url))
            return false;
        return true;
    }
}

```

- url, HttpRequest를 하나의 Key로 관리하는 HandlerKey이다.
- 두가지를 하나의 Key로 관리하기 위해 equals와 hashCode의 override가 필요하다.

```java
public class HandlerExecution {
    private static final Logger logger = LoggerFactory.getLogger(HandlerExecution.class);

    private Object declaredObject;
    private Method method;

    public HandlerExecution(Object declaredObject, Method method) {
        this.declaredObject = declaredObject;
        this.method = method;
    }

    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response) throws Exception {
        try {
            return (ModelAndView) method.invoke(declaredObject, request, response);
        } catch (IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
            logger.error("{} method invoke fail. error message : {}", method, e.getMessage());
            throw new RuntimeException(e);
        }
    }
}
```

- 하나의 handlerKey에 Mapping 되는 HadlerExecution에 대한 코드

### 10.2.4 DispatcherServlet과 AnnotationHandlerMapping 통합

기존의 Mapping 방법과 Annotation을 사용한 Mapping 두가지의 Handler를 인터페이스로 관리하여 추상화를 진행 할 수 있다.

```
public interface HandlerMapping {
    Object getHandler(HttpServletRequest request);
}

```

- HandlerMapping을 interface로 구현한 코드

```java
public class LegacyHandlerMapping implements HandlerMapping {
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);
    private Map<String, Controller> mappings = new HashMap<>();

    public void initMapping() {
        mappings.put("/qna/show", new ShowQuestionController());
        mappings.put("/qna/form", new CreateFormQuestionController());
        mappings.put("/qna/create", new CreateQuestionController());
        mappings.put("/qna/updateForm", new UpdateFormQuestionController());
        mappings.put("/qna/update", new UpdateQuestionController());
        mappings.put("/qna/delete", new DeleteQuestionController());
        mappings.put("/api/qna/deleteQuestion", new ApiDeleteQuestionController());
        mappings.put("/api/qna/list", new ApiListQuestionController());
        mappings.put("/api/qna/addAnswer", new AddAnswerController());
        mappings.put("/api/qna/deleteAnswer", new DeleteAnswerController());

        logger.info("Initialized Request Mapping!");
    }

    public Controller findController(String url) {
        return mappings.get(url);
    }

    void put(String url, Controller controller) {
        mappings.put(url, controller);
    }

    @Override
    public Controller getHandler(HttpServletRequest request) {
        return mappings.get(request.getRequestURI());
    }
}
```

- 기존의 requestMapping 코드

```java
@WebServlet(name = "dispatcher", urlPatterns = "/", loadOnStartup = 1)
public class DispatcherServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);

    private List<HandlerMapping> mappings = Lists.newArrayList();
    private List<HandlerAdapter> handlerAdapters = Lists.newArrayList();

    @Override
    public void init() throws ServletException {
        LegacyHandlerMapping lhm = new LegacyHandlerMapping();
        lhm.initMapping();
        AnnotationHandlerMapping ahm = new AnnotationHandlerMapping("next.controller");

        ahm.initialize();

        mappings.add(lhm);
        mappings.add(ahm);
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Object handler = getHandler(req);
        if (handler == null) {
            throw new IllegalArgumentException("존재하지 않는 URL입니다.");
        }

        try {
            ModelAndView mav = execute(handler, req, resp);

            if (mav != null) {
                View view = mav.getView();
                view.render(mav.getModel(), req, resp);
            }
        } catch (Throwable e) {
            logger.error("Exception : {}", e);
            throw new ServletException(e.getMessage());
        }
    }
    private Object getHandler(HttpServletRequest req) {
        for (HandlerMapping handlerMapping : mappings) {
            Object handler = handlerMapping.getHandler(req);
            if (handler != null) {
                return handler;
            }
        }
        return null;
    }
    private ModelAndView execute(Object handler, HttpServletRequest req, HttpServletResponse resp) throws Exception {
        if(handler instanceof Controller){
          return ((Contoller)handler).execute(req, resp);
        }
        else {
          return ((HandlerExecution)handler).handle(req, resp);
        }
    }
}
```

-AnnotationMapping을 지원하도록 수정한 DispatcherServlet 코드

## 10.3 인터페이스가 다른 경우 확장성 있는 설계

```java
private ModelAndView getExecuteResult(final HttpServletRequest request, final HttpServletResponse response) {
    Object handler = getHandler(request);
    if (handler instanceof Controller) {
        return ((Controller) handler).execute(request, response);
    }
    if (handler instanceof HandlerExecution) {
        return ((HandlerExecution) handler).handle(request, response);
    }
    throw new IllegalArgumentException("Can not find execute handler");
}
```

- 위 코드는 새로운 controller가 등장하면 if문이 끝 없이 추가하게 된다.
- excute에 대한 수정도 필요함으로 개방-폐쇄원칙이 지켜지지 않는다.

```java
public interface HandlerAdapter {
    boolean supports(Object handler);

    ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
}
```

- adapter 패턴을 적용한 HandlerAdapter interface로 위 문제를 해결한다.

```java
public class HandlerExecutionHandlerAdapter implements HandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        return handler instanceof HandlerExecution;
    }

    @Override
    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        return ((HandlerExecution) handler).handle(request, response);
    }
}
```

- 어노테이션 방식의 handler를 처리하는 adapter

```java
public class ControllerHandlerAdapter implements HandlerAdapter {
    @Override
    public boolean supports(Object handler) {
        return handler instanceof Controller;
    }

    @Override
    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {
        return ((Controller) handler).execute(request, response);
    }
}
```

- 기존 방식의 handler를 처리하는 adapter

이제 handlerAdaptor를 DispatcherServlet에 적용하자.

```java
private ModelAndView getExecuteResult(final HttpServletRequest request, final HttpServletResponse response) {
    Object handler = getHandler(request);
    return handlerAdapters.stream()
            .filter(handlerAdapter -> handlerAdapter.support(handler))
            .findFirst()
            .orElseThrow(() -> new IllegalArgumentException("Can not find execute handler"))
            .execute(request, response, handler);
}
```

- 어떤 handler 구현체가 오더라도 위 코드에서 다 실행이 가능하다.
