# 10장 새로운 MVC 프레임워크 구현을 통한 점진적 개선
**목표**
- 지금까지 구현한 MVC 프레임워크의 문제점을 파악한 후 새로운 MVC 프레임워크를 구현하자
- 단, 이전에 구현한 MVC 프레임워크 기반의 컨트롤러가 정상적으로 동작하는 상태에서 새로운 MVC 프레임워크로 점진적으로 전환해 가는 과정에 대해 살펴보자
## 10.1 MVC 프레임워크 요구사항 3단계
### 10.1.1 요구사항
**기존 문제점**
- 새로운 컨트롤러가 추가될 때마다 매번 RequestMapping 클래스에 요청 URL과 컨트롤러를 추가하는 작업 필요
  (유지보수 차원에서 좋지 않음)
<br/><br/>
```java
@Controller
public class UserController extends AbstractNewController {
    private static final Logger logger = LoggerFactory.getLogger(UserController.class);

    private UserDao userDao = UserDao.getInstance();

    @RequestMapping(value = "/users", method = RequestMethod.POST )
    public ModelAndView list(HttpServletRequest request, HttpServletResponse response) throws Exception {
        logger.debug("users create");
        return new ModelAndView(new JspView("redirect:/users"));
    }
    
    [...]
}
```
그래서 새로운 기능이 추가될 때마다 
- 매번 컨트롤러를 추가하는 것이 아니라 메소드를 추가
- 요청 URL 매핑시 HTTP 메소드 활용
<br/>

=> 위 코드와 같이 애노테이션 기반으로 MVC 프레임워크를 통합하자

**커스텀 어노테이션 구현**
- Annotation : 자바 소스 코드에 추가하여 사용할 수 있는 메타데이터의 일종
```java
// @Controller
@Target({ ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
public @interface Controller {
    String value() default "";
}
// @RequestMapping
@Target({ ElementType.METHOD, ElementType.TYPE })
@Retention(RetentionPolicy.RUNTIME)
public @interface RequestMapping {
  String value() default "";

  RequestMethod method() default RequestMethod.GET;
}
// RequestMethod enum 클래스
public enum RequestMethod {
  GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE
}
```
- @Target : 적용대상 지정 
  - PACKAGE : 패키지
  - TYPE : 타입
  - ANNOTATION_TYPE : 어노테이션 타입
  - CONSTRUCTOR : 생성자
  - FIELD : 필드
  - LOCAL_VARIABLE : 지역 변수 
  - METHOD : 메서드 
  - PARAMETER : 파라미터
  - TYPE_PARAMETER : 파라미터 타입
  - TYPE_USE : 타입 


- @Retention : 애노테이션 유지정책 설정
  - SOURCE: 소스상에서만 애노테이션 유지
  - CLASS : 바이트 코드 파일까지 애노테이션 유지
  - RUNTIME : 바이트 코드 파일까지 애노테이션을 유지, 런타임시 애노테이션 정보 확인 가능
  - 리플렉션 기능으로 런타임시 애노테이션 정보를 얻을라면 애노테이션 유지정책을 RUNTIME으로 설정해야 함
### 10.1.2 자바 리플렉션
- Reflection
  - 구체적인 클래스 타입을 알지 못하더라도 그 클래스의 메서드, 타입, 변수들에 접근할 수 있도록 해주는 자바 API
  - 컴파일 시간이 아닌 실행 시간에 동적으로 특정 클래스의 정보를 추출할 수 있는 프로그래밍 기법


- getXXX() vs getDeclaredXXX()
  - getXXX() : 상속받은 클래스와 인터페이스를 포함하여 모든 public 요소 접근
  - getDeclaredXXX() : 상속받은 클래스와 인터페이스를 제외하고 해당 클래스에 직접 정의된 내용만 접근 (private, protect로 선언된 요소 접근)


우리는 실습으로 `getMethods()`, `getDeclaredConstructor()`, `getDeclaredFields()` 등의 함수를 사용해 런타임시간에 클래스의 함수, 생성자, 필드에 대한 정보들을 가져오는 것을 확인할 수 있다. <br/>
이러한 리플렉션 API 를 사용해 애노테이션을 구현한다.<br/>
한가지 예시로 리플렉션을 사용해 클래스와 메소드에 어떤 어노테이션이 붙어 있는지 확인할 수 있다.

## 10.2 MVC 프레임워크 구현 3단계
### 10.2.1 @Controller 애노테이션 설정 클래스 스캔
reflections 라이브러리를 활용해 @Controller이 설정되어 있는 모든 클래스를 찾고,
각 클래스에 대한 인스턴스 생성을 담당하는 ControllerScanner 클래스를 추가한다.
```java
public class ControllerScanner {
    private static final Logger log = LoggerFactory.getLogger(ControllerScanner.class);

    private Reflections reflections; 
  
    public ControllerScanner(Object... basePackage) {
        reflections = new Reflections(basePackage);
    }

  // @Controller 어노테이션이 선언된 클래스 목록을 찾아 클래스 인스턴스가 담긴 Map 반환한다.
    public Map<Class<?>, Object> getControllers() {
       // 어노테이션을 가지고 있는 모든 클래스를 Set에 저장
        Set<Class<?>> preInitiatedControllers = reflections.getTypesAnnotatedWith(Controller.class);
        return instantiateControllers(preInitiatedControllers);
    }

  // 클래스에 대한 인스턴스를 생성해 Map<Class<?>, Object>에 추가한다.
    Map<Class<?>, Object> instantiateControllers(Set<Class<?>> preInitiatedControllers) {
        Map<Class<?>, Object> controllers = Maps.newHashMap();
        try {
            for (Class<?> clazz : preInitiatedControllers) {
                controllers.put(clazz, clazz.newInstance());
            }
        } catch (InstantiationException | IllegalAccessException e) {
            log.error(e.getMessage());
        }
        return controllers;
    }
}
```
- Maps.newHashMap() 을 아시나요?
  <br/>
  - Google에서 만든 오픈소스 라이브러리
  - 가독성 있는 코드를 작성하기 위한 많은 유틸 클래스 제공
  - [(참고)Guava를 쓰는 이유](https://www.javacodegeeks.com/2013/06/5-reasons-to-use-guava.html?amp=1)
  
### 10.2.2 @RequestMapping 애노테이션 설정을 활용한 매핑
ControllerScanner를 통해 찾은 @Controller 클래스의 메소드 중 @RequestMapping이 설정되어 있는 모든 메소드를 찾는다.
<br/>
1. HandlerKey
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
        [...]
    }

    @Override
    public boolean equals(Object obj) {
        [...]
    }
}
```
@RequestMapping이 붙은 메서드의 Key값이 되는 클래스이다.<br/>
Controller를 저장한 것과 다르게 Map의 Key값으로 요청 URL과 HTTP 메소드 조합으로 구성한다.
<br/>

2. HandlerExecution
```java
public class HandlerExecution {
    private static final Logger logger = LoggerFactory.getLogger(HandlerExecution.class);

    private Object declaredObject; // 메서드를 생성한 클래스 인스턴스
    private Method method; // 실행할 메서드

    public HandlerExecution(Object declaredObject, Method method){
        this.declaredObject = declaredObject;
        this.method = method;
    }
    // 메서드를 실행시키는 함수
    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response) throws Exception {
       try {
           return (ModelAndView) method.invoke(declaredObject, request, response);
       }catch (IllegalAccessException | IllegalArgumentException e){
           logger.error("{} method invoke fail. error message : {}", method, e.getMessage());
           throw new RuntimeException(e);
       }
    }
}
```
메서드를 실행시키기 위한 정보를 저장하는 클래스이다.<br/>
메서드를 생성한 클래스 인스턴스와 메서드의 정보를 가지고 있어 
`invoke()`를 통해 메서드를 실행시킬 수 있다.

3. AnonotationHandlerMapping
<br/>
   AnonotationHandlerMapping클래스에서 HandlerKey와 HandlerExecution을 연결한다.
```java
public class AnnotationHandlerMapping implements HandlerMapping {
    private static final Logger logger = LoggerFactory.getLogger(AnnotationHandlerMapping.class);

    private Object[] basePackage;

    private Map<HandlerKey, HandlerExecution> handlerExecutions = Maps.newHashMap();

    public AnnotationHandlerMapping(Object... basePackage) {
        this.basePackage = basePackage;
    }
    // 매핑 초기화 작업
    public void initialize() {
        ControllerScanner controllerScanner = new ControllerScanner(basePackage);
        // @Cotroller가 붙은 class 인스턴스 가져오기
        Map<Class<?>, Object> controllers = controllerScanner.getControllers();
        // @RequestMapping이 붙은 method 가져오기
        Set<Method> methods = getRequestMappingMethods(controllers.keySet());
        for(Method method : methods) {
            // @RequestMapping이 붙은 메서드의 애노테이션 정보 가져오기
            RequestMapping rm = method.getAnnotation(RequestMapping.class);
            logger.debug("register handlerExcution : url is {}, method is {}",rm.value(),method);
            // Map<HandlerKey, HandlerExecution> 만들기
            handlerExecutions.put(createHandlerKey(rm),
                    new HandlerExecution(controllers.get(method.getDeclaringClass()),method));
        }
    }

    @SuppressWarnings("unchecked")
    private Set<Method> getRequestMappingMethods(Set<Class<?>> controllers) {
        Set<Method> requestMappingMethods = Sets.newHashSet();
      // 모든 controller를 확인하며 @RequestMapping이 붙은 메서드 구하기
        for(Class<?> clazz : controllers){
            requestMappingMethods.addAll(ReflectionUtils.getAllMethods(clazz,
                    ReflectionUtils.withAnnotation(RequestMapping.class)));
        }
        return requestMappingMethods;
    }

    private HandlerKey createHandlerKey(RequestMapping rm) {
        return new HandlerKey(rm.value(), rm.method());
    }

    public HandlerExecution getHandler(HttpServletRequest request) {
        String requestUri = request.getRequestURI();
        RequestMethod rm = RequestMethod.valueOf(request.getMethod().toUpperCase());
        return handlerExecutions.get(new HandlerKey(requestUri, rm));
    }
}
```
위 과정을 통해 요청을 컨트롤러에 매핑하는 기능을 만들었다.<br/>
기존의 `RequestMapping`과 다른 점은 `RequestMapping`은 개발자가 수동으로 등록하고, `AnnotationHandlerMapping`은 애노테이션을 설정하면 자동으로 매핑된다.

### 10.2.4 DispatcherServlet과 AnnotationHandlerMapping 통합
기존에 사용하던 `RequestMapping`과 새로 추가한 `AnnotationHandlerMapping`이 같이 동작하도록 구현한다.
<br/>
이 둘을 모두 지원하기 위해 분리해서 관리할 수도 있지만
**일관된 인터페이스를 구현하도록 통합 후 관리하는 것이 향후 확장성을 위해** 좋다.
1. 두 Mapping 클래스를 추상화한 인터페이스 추가
```java
public interface HandlerMapping {
    Object getHandler(HttpServletRequest request);
}
```

2. RequestMapping과 AnnotationHandlerMapping 클래스가 HandlerMapping 인터페이스 구현
```java
// RequestMapping 클래스를 LegacyHandlerMapping으로 변경
// HandlerMapping 인터페이스 상속 후 getHandler() 구현
public class LegacyHandlerMapping implements HandlerMapping {
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);
    private Map<String, Controller> mappings = new HashMap<>();

    [...]

    @Override
    public Object getHandler(HttpServletRequest request) {
        return mappings.get(request.getRequestURI());
    }
}
```
- AnnotationHandlerMapping은 getHandler()가 구현되어 있어 HandlerMapping 인터페이스 상속만 한다.

3. DispatcherServlet에서 두 개의 HandlerMapping이 동작하도록 수정

```java
@WebServlet(name = "dispatcher", urlPatterns = "/", loadOnStartup = 1)
public class DispatcherServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);

    private List<HandlerMapping> mappings = Lists.newArrayList();
    private LegacyHandlerMapping rm;

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
        // 요청에 맞는 handler 찾기
        Object handler = getHandler(req);
        if(handler == null) {
            throw new IllegalArgumentException("존재하지 않는 URL입니다.");
        }

        try {
            ModelAndView mav = execute(handler, req, resp);
            View view = mav.getView();
            view.render(mav.getModel(), req, resp);
        } catch (Throwable e) {
            logger.error("Exception : {}", e);
            throw new ServletException(e.getMessage());
        }
    }
    // Legacy는 URL만 확인하고 Annotation은 URL과 HTTP Method를 확인해 맞는 handler 반환한다.
    private Object getHandler(HttpServletRequest req) {
        for(HandlerMapping handlerMapping : mappings){
            Object handler = handlerMapping.getHandler(req);
            if(handler != null){
                return handler;
            }
        }
        return null;
    }

    private ModelAndView execute(Object handler, HttpServletRequest req, HttpServletResponse resp) throws Exception{
        // Legacy는 Controller 클래스 반환, Annotation은 HandlerExecution 반환 
        if(handler instanceof Controller){
            return ((Controller)handler).execute(req,resp);
        }else{
            return ((HandlerExecution)handler).handle(req, resp);
        }
    }
}
```
통합은 초기화가 끝난 HandlerMapping을 List로 관리하여 Request에 해당하는 컨트롤러를 찾아 컨트롤러에게 작업을 위임하도록 구현한다.

## 10.3 인터페이스가 다른 경우 확장성 있는 설계
기존 컨트롤러인 Controller 인터페이스와 새로 추가한 컨트롤러인 Handler Execution 클래스를 보면 메소드 인자와 반환 값이 같아 둘을 통합하는 것이 가능하다.
- HandlerExecution 클래스도 Controller 인터페이스를 구현하는 경우
  - 장점
     <br/>
    같은 인터페이스(Controller)를 구현함으로 캐스팅 작업이 빠져 소스코드가 깔끔해짐
  - 단점
  <br/>
    외부 라이브러리, 프레임워크 기반의 컨트롤러를 개발할 경우 하나의 인터페이스로 통합하는게 불가능하기 때문에 확장성이 떨어짐

그래서 새로운 유형의 컨트롤러가 추가되더라도 서로 간의 영향을 주지 않으면서 확장할 수 있어야 한다.
<br/>
여러 개의 프레임워크 컨트롤러는 역할이 같기 때문에 **하나의 추상화 단계를 구현**하면 된다.

```java
public interface HandlerAdapter {
    // 핸들러를 지원하는 클래스인지 확인한다.
    boolean supports(Object handler);
    // 핸들러를 실행시킨다.
    ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
}
```

이후 각 컨트롤러에 맞는 HandlerAdapter를 구현한다.
```java
// LegacyHandler(Controller)를 지원하는 Adapter
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

// AnnotationHandler(HandlerExecution)를 지원하는 Adapter
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

이제 DispatcherServlet을 다음과 같이 리팩토링이 가능하다.

```java
@WebServlet(name = "dispatcher", urlPatterns = "/", loadOnStartup = 1)
public class DispatcherServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);

    private List<HandlerMapping> mappings = Lists.newArrayList();
    private List<HandlerAdapter> handlerAdapters = Lists.newArrayList();

    @Override
    public void init() throws ServletException {
        
        [...]
        // HandlerAdapter를 리스트에 추가한다.
        handlerAdapters.add(new ControllerHandlerAdapter());
        handlerAdapters.add(new HandlerExecutionHandlerAdapter());
    }

    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String requestUri = req.getRequestURI();
        logger.debug("Method : {}, Request URI : {}", req.getMethod(), requestUri);
        // 요청에 맞는 handler(Controller 또는 HandlerExecution)를 가져온다.
        Object handler = getHandler(req);
        if (handler == null) {
            throw new IllegalArgumentException("존재하지 않는 URL입니다.");
        }

        try {
            // handler를 실행시킨다.
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
        for (HandlerAdapter handlerAdapter : handlerAdapters) {
            // handlerAdapter.supports()를 통해 Controller인지 HandlerExecution인지 구분한다.
            if (handlerAdapter.supports(handler)) {
                // 객체에 맞게 요청을 실행한다.
                return handlerAdapter.handle(req, resp, handler);
            }
        }
        return null;
    }
}
```
- 기존 execute()
```java
private ModelAndView execute(Object handler, HttpServletRequest req, HttpServletResponse resp) throws Exception {
    if(handler instanceof Controller){
      return ((Contoller)handler).execute(req, resp);
    }
    else {
      return ((HandlerExecution)handler).handle(req, resp);
    }
}
```
기존 execute() 메소드의 경우 새로운 컨트롤러 유형이 추가될 경우 else if 절이 추가되는 구조였다.<br/>
하지만 우리는 추상화 단계를 하나 더 만듦으로써
새로운 handler가 HandlerAdapter만 구현한다면 다른 HandlerAdapter에 영향을 미치지 않으면서
독립적으로 확장할 수 있다.
