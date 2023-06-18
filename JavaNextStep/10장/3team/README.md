# 10장 새로운 프레임워크 구현을 통한 점진적 개선

Annotation과 Reflection을 활용해 ReqeusetMapping을 구현해보자


## 10.1 MVC 프레임워크 요구사항 3단계

**현재 클라이언트-서버 통신 과정**
```
//DispatcherServlet 클래스
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
1. 클라이언트 요청
2. Ddspatcher Servlet이 요청을 service에서 처리
    <br>2.1 Ddspatcher Servlet 객체가 없다면 객체생성
    <br>2.2 request mapping 등록(init)
3. 요청 url에 맞는 컨트롤러를 해시맵을 이용하여 검색(findController)
4. 컨트롤러 실행 및 view 객체 반환(execute)
5. view 객체 실행(render) 

**기존 controller mapping 초기화 코드**

```
public class RequestMapping{
  private Map<String,Controller> mappings = new HashMap<>();

  void initMapping(){
    mappings.put("/", new HomeController());
    mappings.put("/users/form", new ForwardController("/user/form.jsp"));
    mappings.put("/users", new ListUserController());
    mappings.put("/users/create", new CreateUserController());
    ...
  }
}

```

컨트롤러가 추가될 때마다 RequestMapping의 initMapping을 추가해야 됨 (개방-폐쇄의 원칙 위반).
<br>
-> RequestMapping 코드를 건들지 않고 컨트롤러 추가를 할 수 있는 방법 필요

해결법 : Annotaion(@Controller, @RequestMapping)를 활용한 컨트롤러의 핸들러 자동 등록

**요구사항 분리**
1. @Controller를 가지고 있는 모든 클래스를 탐색하는 ControllerScanner 필요
2. Controller내부의 @RequestMapping을 가지고 있는 메소드(핸들러)를 url과 요청 타입으로 구성된 key로 mappings에 등록하는 HandlerMapping 필요
3. 요청 객체인 Request를 타입(GET,POST ...)과 URL를 조합한 mapping key로 변환하는 기능 필요
4. 기존 initMapping을 통한 등록된 핸들러와 어노테이션을 통한 등록된 핸들러들의 실행기 필요


**Annotation**

소스 코드에 추가해서 사용할 수 있는 메타데이터

메타 데이터 : 데이터의 구조 및 기능을 설명해 주는 데이터

@Target : 어노테이션을 붙일 수 있는 위치

- ElementType.TYPE : 클래스, 인터페이스, enum
- ElementType.FIELD : 필드
- ElementType.METHOD : 메서드
- ElementType.PARAMETER : 파라미터
- ElementType.CONSTRUCTOR : 생성자
- ElementType.LOCAL_VARIABLE : 지역 변수 
- ElementType.ANNOTATION_TYPE : 어노테이션 
- ElementType.PACKAGE : 패키지
- ElementType.TYPE_PARAMETER : 타입 파라미터
- ElementType.TYPE_USE : 타입
- ElementType.MODULE : 모듈


@Retention : 어노테이션이 유지되는 시점

- RetentionPolicy.RUNTIME : 런타임까지
- RetentionPolicy.CLASS : 컴파일까지
- RetentionPolicy.SOURCE : 컴파일 전 소스레벨 까지


```
  //컨트롤러 annotation 구현

  @Target({ ElementType.TYPE })
  @Retention(RetentionPolicy.RUNTIME)
  public @interface Controller {
      String value() default "";
  } 

  //RequestMapping annotation 구현
  //@RequestMapping(value="/user",method="POST")와 같이 사용 가능

  @Target({ ElementType.METHOD, ElementType.TYPE })
  @Retention(RetentionPolicy.RUNTIME)
  public @interface RequestMapping {
    String value() default "";

    RequestMethod method() default RequestMethod.GET;
  }

  //RequestMethod 구현
  public enum RequestMethod {
    GET, HEAD, POST, PUT, PATCH, DELETE, OPTIONS, TRACE
  }
```

**Java Reflection**

클러스 로더를 통해 JVM에 저장되어 있는 클래스 정보를 가져올 수 있게 도와주는 클래스

- 특징
	- 클래스의 메소드, 필드의 값을 가져올 수 있음
	- 특정 어노테이션이 있는 클래스의 정보만 추출할 수 있음

- 종류
	- getFields/getDeclaredFields : public/모든 필드를 가져오는 메소드
	- getConstructors/getDeclaredConstructors : public/모든 생성자를 가져오는 메소드
	- getMethods/getDeclaredMethods : public/모든 메소드를 가져오는 메소드


## 10.2 MVC 프레임워크 구현 3단계

### 10.2.1 @Controller annotiation 설정 클래스 스캔

@controller가 붙은 클래스를 찾기 위해 reflection을 활용해 모든 컨트롤러를 스캔한다.

```
//@Controller 어노테이션이 붙은 클래스(컨트롤러)를 찾는 기능 
public class ControllerScanner {
    private static final Logger log = LoggerFactory.getLogger(ControllerScanner.class);

    private Reflections reflections; //클래스의 정보를 얻기 위한 reflection 변수

    //기본 경로를 등록하여 하위의 경로에서 컨트롤러를 찾도록 함
    public ControllerScanner(Object... basePackage) {
        reflections = new Reflections(basePackage);
    }

    //@controller 어노테이션이 붙은 모든 메소드를 중복없이 찾아내는 메소드
    public Map<Class<?>, Object> getControllers() {
       //getTypesAnnotatedWith(어노테이션 이름.class) : 어노테이션을 가지고 있는 모든 클래스를 리턴
        Set<Class<?>> preInitiatedControllers = reflections.getTypesAnnotatedWith(Controller.class);
        return instantiateControllers(preInitiatedControllers);
    }

    // 컨트롤러의 내부 메소드를 사용하기 위해 컨트롤러 클래스를 인스턴스화 시켜 저장하는 메소드
    Map<Class<?>, Object> instantiateControllers(Set<Class<?>> preInitiatedControllers) {
        //클래스 이름을 키로 가져 해당 클래스의 인스턴스를 가져오는 map 변수
        Map<Class<?>, Object> controllers = Maps.newHashMap();
        try {
            //모든 클래스를 인스턴스로 만들어 map어 저장
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

### 10.2.2 @RequestMapping annotiation 설정을 활용한 매핑

@RequestMapping과 일치하는 '요청'을 찾아서 '실행'시키는 기능이 필요하다.
빠른 검색을 위해 Map<요청,실행> 으로 구성된 map으로 RequestMapping을 구현한다.

1) 요청 key(HandlerKey)

@RequsetMapping 예제 : @RequestMapping(value = "/hello", method = RequestMethod.GET)
위와 같이 모든 요청의 핸들러는 url(value)과 요청 타입(method)으로 결정된다.
2가지의 복합키로 핸들러를 찾아야 하기 때문에 복합키 클래스 HandleKey를 생성한다.

```
public class HandlerKey {
    private String url; //url
    private RequestMethod requestMethod; //요청 타입 ex)get, post, put

    public HandlerKey(String url, RequestMethod requestMethod) {
        this.url = url;
        this.requestMethod = requestMethod;
    }
}
```

2) 실행 value(HandlerExecution)

@RequsetMapping이 붙은 핸들러를 실행 시키는 부분이다.
Method 클래스의 invoke를 활용하면 클래스의 이름과 파라미터로 메소드 실행이 가능하다.

```
//핸들로를 실행시키는 클래스
public class HandlerExecution {
    private static final Logger logger = LoggerFactory.getLogger(HandlerExecution.class);

    private Object declaredObject; //실행시킬 메소드가 있는 클래스
    private Method method;  //실행시킬 메소드

    public HandlerExecution(Object declaredObject, Method method) {
        this.declaredObject = declaredObject;
        this.method = method;
    }

    //핸들러를 실행시켜 modelAndView를 생성하는 메소드
    public ModelAndView handle(HttpServletRequest request, HttpServletResponse response) throws Exception {
        try {
            //method.invoke(클래스 이름, 파라미터1,파라미터2,...) : 메소드를 실행시키는 메소드
            return (ModelAndView) method.invoke(declaredObject, request, response);
        } catch (IllegalAccessException | IllegalArgumentException | InvocationTargetException e) {
            logger.error("{} method invoke fail. error message : {}", method, e.getMessage());
            throw new RuntimeException(e);
        }
    }
}
```

3) @RequestMapping이 붙은 모든 메소드를 스캔(AnnotationHandlerMapping)

@RequestMapping이 붙은 모든 메소드를 스캔하여 HandlerKey와 HandlerExecution로 이루어진 map을 생성한다.
AnnotationHandlerMapping 클래스를 DispatcherServlet이 생성될 때 초기화하면 모든 RequestMapping이 붙은 핸들러를 탐색해 저장할 수 있다.

```
public class AnnotationHandlerMapping implements HandlerMapping {
    private static final Logger logger = LoggerFactory.getLogger(AnnotationHandlerMapping.class);

    private Object[] basePackage;

    private Map<HandlerKey, HandlerExecution> handlerExecutions = Maps.newHashMap();

	//@RequestMapping을 탐색할 클래스의 루트 디렉토리를 지정
    public AnnotationHandlerMapping(Object... basePackage) {
        this.basePackage = basePackage;
    }
	//모든 컨트롤러에서 @RequestMapping을 추출해 handlerExecution을 생성하는 메소드
    public void initialize() {
        ControllerScanner controllerScanner = new ControllerScanner(basePackage);
		//모든 컨트롤러 탐색
        Map<Class<?>, Object> controllers = controllerScanner.getControllers();
        
		//모든 @RequestMapping이 붙은 메소드,클래스 탐색
		Set<Method> methods = getRequestMappingMethods(controllers.keySet());
		
		//모든 메소드의 handlerKey와 handlerExecution 생성
        for (Method method : methods) {
           //@RequestMapping 어노테이션 추출
            RequestMapping rm = method.getAnnotation(RequestMapping.class);
            logger.debug("register handlerExecution : url is {}, method is {}", rm.value(), method);
            
            //해당 메소드의 handlerKey와 handlerExecution 생성
            handlerExecutions.put(createHandlerKey(rm),
                    new HandlerExecution(controllers.get(method.getDeclaringClass()), method));
        }

        logger.info("Initialized AnnotationHandlerMapping!");
    }

    //핸들러 키를 생성하는 메소드
    private HandlerKey createHandlerKey(RequestMapping rm) {
        //value : url, method : 요청 타입
        return new HandlerKey(rm.value(), rm.method());
    }

    //모든 @RequestMapping이 붙은 메소드 탐색하는 메소드
    @SuppressWarnings("unchecked")
    private Set<Method> getRequestMappingMethods(Set<Class<?>> controlleers) {
        //모든 requestMapping이 담긴 HashSet 생성
        Set<Method> requestMappingMethods = Sets.newHashSet();
       
        //모든 컨트롤러에서 @RequestMapping이 붙은 메소드 추출
        for (Class<?> clazz : controlleers) {
            //ReflectionUtils.getAllMethods(클래스 명, 어노테이션 조건) : 어노테이션 조건에 맞는 모든 메소드를 클래스에서 찾아 set으로 반환하는 메소드
            requestMappingMethods
                    .addAll(ReflectionUtils.getAllMethods(clazz, ReflectionUtils.withAnnotation(RequestMapping.class)));
        }
        return requestMappingMethods;
    }
}
```

### 10.2.3 클라이언트 요청에 해당하는 HandlerExecution 반환

이제 클라이언트 요청을 handlerKey로 만들어 해당 키로 HandlerExecution을 찾으면 요청 별 핸들러를 실행시킬 수 있게 된다.
<br>
우선 클라이언트 요청(HttpServletRequest)를 handlerKey로 변환하는 기능을 AnnotationHandlerMapping에 추가한다.
```
public class AnnotationHandlerMapping implements HandlerMapping {
    ...
    //요청으로 HandlerKey를 생성해 해당 키에 맞는 handlerExecution이 있는지 확인하는 메솓드
    @Override
    public HandlerExecution getHandler(HttpServletRequest request) {
        String requestUri = request.getRequestURI();
        //요청 method를 대문자로 만들어 RequestMethod enum값으로 변경
        RequestMethod rm = RequestMethod.valueOf(request.getMethod().toUpperCase());
        logger.debug("requestUri : {}, requestMethod : {}", requestUri, rm);
        
        //HandlerKey를 uri와 RequestMethod를 이용해 생성해 해당 키에 맞는 handlerExecution을 리턴
        //없다면 null 리턴
        return handlerExecutions.get(new HandlerKey(requestUri, rm));
    }
}
```
### 10.2.4 DispatcherServlet과 AnnotationHandlerMapping 통합

현재 프로젝트는 1)직접 mapping에 핸들러를 넣는 방법과 2)annotation으로 mapping에 handler를 넣는 방법 2가지를 가지고있다.
<br>2가지 방법 모두 통용되게 하기 위해 1)과 2)를 통합시킬 필요가 있다.
<br>1)과 2)의 통합을 위해 인터페이스로 추상화를 진행한다.

```
//요청 객체를 이용해 맞는 핸들러를 찾는 기능을 가진 인터페이스
public interface HandlerMapping {
    Object getHandler(HttpServletRequest request);
}

```

1)의 방법은 uri로 handlerKey를 생성하고 2)의 방법은 uri와 요청 method를 이용해 handlerKey를 생성한다.
기존 1)의 방법을 HalderMapping을 상속하게 하여 재구성한다.

```
public class LegacyHandlerMapping implements HandlerMapping {
    //기존 초기 mapping을 추가하는 방식
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
    //기존 getHandler 부분(더이상 사용하지 않는다.)
    public Controller findController(String url) {
        return mappings.get(url);
    }
    //기존 mapping을 추가하는 방식
    void put(String url, Controller controller) {
        mappings.put(url, controller);
    }

  //새로 만든 getHandler
  @Override
    public Controller getHandler(HttpServletRequest request) {
        return mappings.get(request.getRequestURI());
    }
}
```

이제 기존 방법과 어노테이션 방법이 getHandler라는 메소드 하나로 request객체를 이용해 요청에 맞는 핸들러를 찾을 수 있게 됐다.
<br>DispatcherServlet에서 어노테이션 방식으로 mapping을 등록할 수 있도록 수정한다.

```
@WebServlet(name = "dispatcher", urlPatterns = "/", loadOnStartup = 1)
public class DispatcherServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);

    private List<HandlerMapping> mappings = Lists.newArrayList();
    private List<HandlerAdapter> handlerAdapters = Lists.newArrayList();

    @Override
    public void init() throws ServletException {
        LegacyHandlerMapping lhm = new LegacyHandlerMapping();
        //기존 방식으로 mapping 등록
        lhm.initMapping();
        AnnotationHandlerMapping ahm = new AnnotationHandlerMapping("next.controller");
        
        //어노테이션 방식으로 mapping 등록
        ahm.initialize();

        //mapping에 핸들러 추가
        mappings.add(lhm);
        mappings.add(ahm);
    }

    //모든 요청을 받아 기능을 시작하는 메소드
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //핸들러 타입이 Controller인지 HandlerExecution인지 모르므로 Object타입으로 핸들러를 탐색 
        Object handler = getHandler(req);
        if (handler == null) {
            throw new IllegalArgumentException("존재하지 않는 URL입니다.");
        }

        try {
            //핸들러를 실행하여 ModelAndView생성
            ModelAndView mav = execute(handler, req, resp);

            if (mav != null) {
                View view = mav.getView();
                //ModelAndView를 이용해 클라이언트에게 해당 페이지 송신 
                view.render(mav.getModel(), req, resp);
            }
        } catch (Throwable e) {
            logger.error("Exception : {}", e);
            throw new ServletException(e.getMessage());
        }
    }
    //요청에 맞는 핸들러가 있는지 탐색하는 메소드
    private Object getHandler(HttpServletRequest req) {
        //모든 mapping에서 핸들러가 있는지 판단하는 부분
        for (HandlerMapping handlerMapping : mappings) {
            Object handler = handlerMapping.getHandler(req);
            //핸들러를 찾으면 해당 핸들러 리턴
            if (handler != null) {
                return handler;
            }
        }
        return null;
    }
    //핸들러를 실행시키는 부분
    private ModelAndView execute(Object handler, HttpServletRequest req, HttpServletResponse resp) throws Exception {
        //기존 방식의 핸들러 실행
        if(handler instanceof Controller){
          return ((Contoller)handler).execute(req, resp);
        }
        //어노테이션 방식의 핸들러 실행
        else {
          return ((HandlerExecution)handler).handle(req, resp);
        }
    }
}
```

## 10.3 인터페이스가 다른 경우 확장성 있는 설계

DispatcherServlet의 execute를 다시보면 handlerExecution의 종류가 많아진다면 if문이 늘어가게 될 수 있다.

```
 private ModelAndView execute(Object handler, HttpServletRequest req, HttpServletResponse resp) throws Exception {
        //기존 방식의 핸들러 실행
        if(handler instanceof Controller){
          return ((Contoller)handler).execute(req, resp);
        }
        //어노테이션 방식의 핸들러 실행
        else {
          return ((HandlerExecution)handler).handle(req, resp);
        }
        //만약 handlerExecution의 종류가 많아지면 아래로 계속 추가해야한다.
    }
```

이렇게 되면 execute를 수정해야 하므로 개방-폐쇄의 원칙에 어긋나게 되어 확장성이 떨어진다.
<br>이를 해결하기위해 인터페이스로 추상화하고 각 종류가 다른 handler를 실행시키는 handlerAdaptor 클래스를 생성한다.

```
//핸들러를 실행시키는 인터페이스
public interface HandlerAdapter {
    //목표 핸들러를 실행시킬 수 있는 클래스인지 확인하는 메소드
    boolean supports(Object handler);

    //목표 핸들러를 실행시키는 메소드
    ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;
}
```
현재 handler의 종류는 Controller와 HandlerExecution 2가지가 있으므로 2가지의 HandlerAdaptor를 생성한다.

```
//어노테이션 방식의 handler(HandlerExecution)를 실행시키는 HandlerAdapter
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

```
//기존 방식의 handler(Controller)를 실행시키는 HandlerAdapter
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

이제 handlerAdaptor를 DispatcherServlet에 적용하자.

```
@WebServlet(name = "dispatcher", urlPatterns = "/", loadOnStartup = 1)
public class DispatcherServlet extends HttpServlet {
    private static final long serialVersionUID = 1L;
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);

    private List<HandlerMapping> mappings = Lists.newArrayList();
    private List<HandlerAdapter> handlerAdapters = Lists.newArrayList();

    @Override
    public void init() throws ServletException {
      LegacyHandlerMapping lhm = new LegacyHandlerMapping();
      //기존 방식으로 mapping 등록
      lhm.initMapping();
      AnnotationHandlerMapping ahm = new AnnotationHandlerMapping("next.controller");
        
      //어노테이션 방식으로 mapping 등록
      ahm.initialize();

      //mapping에 핸들러 추가
      mappings.add(lhm);
      mappings.add(ahm);
      
      //지원하는 모든 핸들러 어댑터 추가
      handlerAdapters.add(new ControllerHandlerAdapter());
      handlerAdapters.add(new HandlerExecutionHandlerAdapter());
    }

    //모든 요청을 받아 기능을 시작하는 메소드
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //핸들러 타입이 Controller인지 HandlerExecution인지 모르므로 Object타입으로 핸들러를 탐색 
        Object handler = getHandler(req);
        if (handler == null) {
            throw new IllegalArgumentException("존재하지 않는 URL입니다.");
        }

        try {
            //핸들러를 실행하여 ModelAndView생성
            ModelAndView mav = execute(handler, req, resp);

            if (mav != null) {
                View view = mav.getView();
                //ModelAndView를 이용해 클라이언트에게 해당 페이지 송신 
                view.render(mav.getModel(), req, resp);
            }
        } catch (Throwable e) {
            logger.error("Exception : {}", e);
            throw new ServletException(e.getMessage());
        }
    }
    //요청에 맞는 핸들러가 있는지 탐색하는 메소드
    private Object getHandler(HttpServletRequest req) {
        //모든 mapping에서 핸들러가 있는지 판단하는 부분
        for (HandlerMapping handlerMapping : mappings) {
            Object handler = handlerMapping.getHandler(req);
            //핸들러를 찾으면 해당 핸들러 리턴
            if (handler != null) {
                return handler;
            }
        }
        return null;
    }
    //핸들러를 실행시키는 부분
    private ModelAndView execute(Object handler, HttpServletRequest req, HttpServletResponse resp) throws Exception {
        //해당 핸들러어댑터가 지원하는 핸들러면 실행시키는 메소드
        for (HandlerAdapter handlerAdapter : handlerAdapters) {
            if (handlerAdapter.supports(handler)) {
                return handlerAdapter.handle(req, resp, handler);
            }
        }
        return null;
    }
}
```

## 10.4 배포 자동화를 위한 쉘 스크립트 개선

**tomcat 구조**

![image](https://github.com/BookBBu/JavaWebProgrammingNextStep/assets/37679352/9325bfba-2541-4fb8-84c5-6475de0e6936)

Server : 톰캣의 가장 큰 단위<br>
Service : connector와 engine을 묶은 단위<br>
Connector : 외부와의 연결을 하는 부분. 요청을 Engine에 넘기고 Engine에서 결과를 받아 외부에게 다시 전달한다.<br>
Engine : 외부의 요청을 host 이름에 따라 처리해서 Connector로 보내는 역할을 담당<br>
Host : 요청의 기능이 담긴 폴더 위치를 가상 주소를 통해 접근할 수 있도록 하는 부분. <br>
Context: 웹 어플리케이션 최소 단위. 아무 설정이 없다면 ROOT 폴더를 기준으로 한다.

ex) http:/[Host 이름]/[Context 경로]:[포트번호]

server.xml 구조

기본 톰캣 설정을 가지고 있는 설정 파일

```
<?xml version="1.0" encoding="UTF-8"?>
<Server port="8005" shutdown="SHUTDOWN">
  <Service name="Catalina">
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443"
               maxParameterCount="1000"
               />
    <Engine name="Catalina" defaultHost="localhost">
      <Host name="localhost"  appBase="webapps" unpackWARs="true" autoDeploy="true">
      </Host>
    </Engine>
  </Service>
</Server>
```
server.xml 해석)

8005번 포트를 톰캣이 사용하는데 현재 톰캣에서 관리하는 서비스는 Catalina 1개이다.
<br>
Catalina에서 사용할 수 있는 포트번호는 8080이며 사용할 수 있는 호스트명은 localhost이다.
<br>
즉 Catalina에 등록된 웹 어플리케이션은 http://localhost/[context 경로]:8080 이다.
<br>
현재 유일한 host인 localhost에 아무런 context 설정이 없으므로 /webapps/ROOT 폴더의 웹 어플리케이션만 이용할 수 있다.<br>

**자동 배포 스크립트 코드**

```
#이후 명령어는 bash 명령어로 실행
#!/bin/bash

#배포할 war파일이 있는 위치
REPOSITORY_DIR=~/repositories/jwp-basic
#톰캣 위치
TOMCAT_HOME = ~/tomcat
#war를 배포 시킬 폴더
RELEASE_DIR=~/release/jwp-basic

#배포 파일 위치로 이동
cd $REPOSITORY_DIR
#git 최신화
git pull
#기존 war파일 삭제
mvn clean package

#현재 시간 가져오기(폴더 이름 때문이지 기능에 영향 X)
C_TIME=${date+%s}

#war파일을 배포 폴더로 이동
mv $REPOSITORY_DIR/target/jwp-basic RELEASE_DIR/$C_TIME

#현재 톰캣 종료
$TOMCAT_HOME/bin/shutdown.sh

#현재 배포중인 war삭제
rm -rf $TOMCAT_HOME/webapps/ROOT

#새로 만든 war파일을 톰캣 ROOT폴더에 연결
ln -s $RELEASE_DIR/$C_TIME $TOMCAT_HOME/webapps/ROOT

#톰캣 실행
$TOMCAT_HOME/bin/startup.sh

#로그 출력
tail -500f $TOMCAT_HOME/logs/catalina.out
```

**자동 롤백 스크립트 코드**
```
#이후 명령어는 bash 명령어로 실행
#!/bin/bash

#배포할 war파일이 있는 위치
REPOSITORY_DIR=~/repositories/jwp-basic
#톰캣 위치
TOMCAT_HOME = ~/tomcat
#이름으로 내림차순 정렬한 문자열
RELEASE=$(ls -tr $REPOSITORY_DIR)
#줄바꿈을 기준으로 문자열을 배열로 변환
REVISIONS=(${REALSE//\n/})

if["${REVISIONS[@]}" -lt 2]; then
  echo "relase source length more then 2"
else
  #현재 톰캣 종료
  $TOMCAT_HOME/bin/shutdown.sh

  #현재 배포중인 war삭제
  rm -rf $TOMCAT_HOME/webapps/ROOT

  #2번째 최신 배포 파일로 변경
  ln -s $RELEASE_DIR/$REVISIONS[1] $TOMCAT_HOME/webapps/ROOT

  #톰캣 실행
  $TOMCAT_HOME/bin/startup.sh

#조건문 종료
fn

#로그 출력
tail -500f $TOMCAT_HOME/logs/catalina.out
```
