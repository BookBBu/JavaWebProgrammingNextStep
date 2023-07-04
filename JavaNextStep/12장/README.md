# 12. 확장성있는 DI 프레임워크로 개선


### 一，DI 방식

1. **생성자** (필수적 의존성 주입)
    - 장점
        - 객체의 일관성과 불변성을 보장
        - 객체 생성 시점에 모든 필수 의존성이 완전히 설정되어 안전한 객체 인스턴스를 생성
        - 코드에서 명시적으로 의존성을 표현해, 코드 가독성과 유지보수성이 향상
        - testing에도 용이해서 제일 많이 사용
            - spring과 독립
        - 순환 참조 방지
            - 순환 참조 : 의존성 주입 시, 재귀적으로 계속 참조되는 경우
    - 단점
        - 생성자 매개변수가 많아질 수 있으며, 매개변수의 순서와 타입에 주의
        - 의존성이 변경되거나 추가되는 경우 생성자 시그니처를 변경해야 하므로, 일부 코드 수정이 필요
    
    ```java
    @Service
    public class MyQnaService {
    	private UserRepository userRepository;
    	private QuestionRepository questionRepository;
    
    	@Inject
    	public void MyQnaService(UserRepository userRepository, QuestionRepository questionRepository) {
    		this.userRepository = userRepository;
    		this.QuestionRepository = questionRepository;
    	}
    }
    ```
    
2. **필드**
    - 장점
        - 코드의 간결성과 가독성
        - 메소드를 통한 별도의 초기화 코드가 필요 없음
        - 중복 코드를 줄일 수 있음
    - 단점
        - 객체의 일관성과 불변성을 보장하기 어려움
            - 필드 값 변동 돼서
            - 스프링에서 필드 주입할 때 final 안되게 설계
        - 외부에서 필드에 직접 접근할 수 있어 캡슐화가 약화
        - 테스트 작성이 어려울 수 있으며, 모의(mock) 객체 주입 등의 특정 테스트 시나리오에서 문제가 발생할 수 있음
            - spring 만들고 나서 필드 객체 생성되는데 test할 때 spring 사용 안 함
    
    ```java
    @Service
    public class MyUserService {
    	@Inject
    	private UserRepository userRepository;
    
    	public void setUserRepository(UserRepository userRepository) {
    		this.userRepository = userRepository;
    	}
    }
    ```
    
1. **Setter 메소드** (선택적 의존성 주입)
    - 장점
        - 유연성을 제공하여 의존성을 동적으로 변경 가능
        - 필요한 의존성만 주입 가능
    - 단점
        - 일부 의존성이 누락 가능
        - setter 메소드를 통해 의존성이 주입되기 때문에 객체의 일관성이 보장되지 않을 수 있습니다.
        - setter 메소드를 공개해야하므로 캡슐화가 약화
        - 자동으로 안 만들어줘서 잘 안씀
    
    ```java
    @Controller
    public class MyUserController {
    	private MyUserService myUserService;
    
    	@Inject	
    	public void setUserService(MyUserService myUserService) {
    		this.myUserService = myUserService;
    	}
    }
    ```
    

### 二， (3가지 방식으로) DI 구현

- **구현 방법**
    1. @Inject가 설정되어 있는 모든 생성자를 가진 클래스/필드/setter 메소드를 찾는다
    2. 해당되는 빈이 BeanFactory에 등록되어 있으면 빈을 활용하고, 등록되어 있지 않으면 빈 인스턴스 생성한 후 DI를 한다
- **역할 분리**
    - BeanFactory : 빈을 추가하고 조회하는 역할
    - Injector를 구현한 클래스 : 생성자/필드/setter로 DI하고 인스턴스 생성하는 역할

💡 **Injector Interface**<br>
   객체 간의 의존성 관리와 주입을 담당
```java
public interface Injector {
  void inject(Class<?> clazz);
}
```

- **중복 제거**
    
    
    💡 **템플릿 메소드 패턴**
    
    - 정의
    상위 클래스에서 알고리즘의 뼈대를 정의하고, 하위 클래스에서 알고리즘의 구체적인 단계를 구현하는 방법을 제공
    - 특징
        1. 코드 중복 제거
            
            공통된 알고리즘의 구조를 상위 클래스에 두어 코드 중복을 방지
            
        2. 확장성과 유연성
            
            알고리즘의 구조는 유지되면서, 각 단계를 하위 클래스에서 오버라이딩하여 변경 가능
            
        3. 캡슐화
            
            알고리즘의 구조를 상위 클래스에 숨겨두고, 구체적인 구현은 하위 클래스에 위임
            
    - 구조
        1. AbstractClass
        템플릿 메소드 패턴의 상위 클래스, 알고리즘의 뼈대를 정의
        2. ConcreteClass
        템플릿 메소드 패턴의 하위 클래스, AbstractClass를 상속
    </aside>
    
    - 클래스 구조 및 역할
        
        ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/82045f75-9167-4cb6-b80f-b0871940ff07/Untitled.png)
        
    - BeanFactory
        - DI 컨테이너의 핵심 동작
        - Bean 인스턴스의 생성, 주입, 조회, 초기화
    - BeanFactoryUtils
        - BeanFactory를 사용할 때 유용한 유틸리티 클래스
        - BeanFactory를 사용하여 DI 작업을 수행할 때 필요한 정보를 조회하거나 처리하기 위해 사용
    - BeanScanner
        - Reflections를 사용하여 주어진 패키지에서 어노테이션이 설정된 클래스들을 스캔
    - AbstractInjector
        
        Injector를 구현한 추상클래스
        
        ```java
        public abstract class AbstractInjector implements Injector {
            private static final Logger logger = LoggerFactory.getLogger(AbstractInjector.class);
        
            private BeanFactory beanFactory;
        
            public AbstractInjector(BeanFactory beanFactory) {
                this.beanFactory = beanFactory;
            }
        
            @Override
            public void inject(Class<?> clazz) {
                instantiateClass(clazz);
                Set<?> injectedBeans = getInjectedBeans(clazz);
                for (Object injectedBean : injectedBeans) {
                    Class<?> beanClass = getBeanClass(injectedBean);
                    inject(injectedBean, instantiateClass(beanClass), beanFactory);
                }
            }
        
            abstract Set<?> getInjectedBeans(Class<?> clazz);
        
            abstract Class<?> getBeanClass(Object injectedBean);
        
            abstract void inject(Object injectedBean, Object bean, BeanFactory beanFactory);
        
            private Object instantiateClass(Class<?> clazz) {
                Class<?> concreteClass = findBeanClass(clazz, beanFactory.getPreInstanticateBeans());
                Object bean = beanFactory.getBean(concreteClass);
                if (bean != null) {
                    return bean;
                }
        
                Constructor<?> injectedConstructor = BeanFactoryUtils.getInjectedConstructor(concreteClass);
                if (injectedConstructor == null) {
                    bean = BeanUtils.instantiate(concreteClass);
                    beanFactory.registerBean(concreteClass, bean);
                    return bean;
                }
        
                logger.debug("Constructor : {}", injectedConstructor);
                bean = instantiateConstructor(injectedConstructor);
                beanFactory.registerBean(concreteClass, bean);
                return bean;
            }
        
            private Object instantiateConstructor(Constructor<?> constructor) {
                Class<?>[] pTypes = constructor.getParameterTypes();
                List<Object> args = Lists.newArrayList();
                for (Class<?> clazz : pTypes) {
                    Class<?> concreteClazz = findBeanClass(clazz, beanFactory.getPreInstanticateBeans());
                    Object bean = beanFactory.getBean(concreteClazz);
                    if (bean == null) {
                        bean = instantiateClass(concreteClazz);
                    }
                    args.add(bean);
                }
                return BeanUtils.instantiateClass(constructor, args.toArray());
            }
        
            private Class<?> findBeanClass(Class<?> clazz, Set<Class<?>> preInstanticateBeans) {
                Class<?> concreteClazz = BeanFactoryUtils.findConcreteClass(clazz, preInstanticateBeans);
                if (!preInstanticateBeans.contains(concreteClazz)) {
                    throw new IllegalStateException(clazz + "는 Bean이 아니다.");
                }
                return concreteClazz;
            }
        }
        ```
        
    - ConstructorInjector
        
        ```java
        public class ConstructorInjector extends AbstractInjector {
            public ConstructorInjector(BeanFactory beanFactory) {
                super(beanFactory);
            }
        
            @Override
            Set<?> getInjectedBeans(Class<?> clazz) {
                return Sets.newHashSet();
            }
        
            @Override
            Class<?> getBeanClass(Object injectedBean) {
                return null;
            }
        
            @Override
            void inject(Object injectedBean, Object bean, BeanFactory beanFactory) {
            }
        }
        ```
        
    - FieldInjector
        
        ```java
        public class FieldInjector extends AbstractInjector {
            private static final Logger logger = LoggerFactory.getLogger(FieldInjector.class);
        
            public FieldInjector(BeanFactory beanFactory) {
                super(beanFactory);
            }
        
        		@Override
            Set<?> getInjectedBeans(Class<?> clazz) {
                return BeanFactoryUtils.getInjectedFields(clazz);
            }
        
        		@Override
            void inject(Object injectedBean, Object bean, BeanFactory beanFactory) {
                Field field = (Field) injectedBean;
                try {
                    field.setAccessible(true);
                    field.set(beanFactory.getBean(field.getDeclaringClass()), bean);
                } catch (IllegalAccessException | IllegalArgumentException e) {
                    logger.error(e.getMessage());
                }
            }
        		
        		@Override
            Class<?> getBeanClass(Object injectedBean) {
                Field field = (Field) injectedBean;
                return field.getType();
            }
        }
        ```
        
    - SetterInjector
        
        ```java
        public class FieldInjector extends AbstractInjector {
            private static final Logger logger = LoggerFactory.getLogger(FieldInjector.class);
        
            public FieldInjector(BeanFactory beanFactory) {
                super(beanFactory);
            }
        		
        		@Override
            Set<?> getInjectedBeans(Class<?> clazz) {
                return BeanFactoryUtils.getInjectedFields(clazz);
            }
        		
        		@Override
            void inject(Object injectedBean, Object bean, BeanFactory beanFactory) {
                Field field = (Field) injectedBean;
                try {
                    field.setAccessible(true);
                    field.set(beanFactory.getBean(field.getDeclaringClass()), bean);
                } catch (IllegalAccessException | IllegalArgumentException e) {
                    logger.error(e.getMessage());
                }
            }
        
        		@Override
            Class<?> getBeanClass(Object injectedBean) {
                Field field = (Field) injectedBean;
                return field.getType();
            }
        }
        ```
        
    

### 三，DI 리팩토링

- **객체지향적 코드**
    - 객체의 책임과 역할을 분리
    - 한 가지 역할과 책임을 가지도록
    - 지나친 분리는 복잡도만 높아질 수 있기 때문에 주의
- **문제**
    - BeanFactory는 모든 빈과 관련한 정보를 관리
    - 빈 인스턴스 생성과 주입 작업을 Injector 구현 클래스가 담당
    - BeanFactory가 일 하는 게 아니라, 단순히 빈 정보를 조회
- **해결 방안**
    - BeanFactory가 빈 인스턴스 생성과 주입 작업 담당
    - 빈 클래스의 상태 정보를 별도의 클래스로 추상화해 관리
- **클래스 구조**
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/28c20572-b6d8-4f04-8b20-9f00c50c9d91/Untitled.png)
    
- **ClasspathBeanDefinitionScanner**
    - BeanScanner → ClasspathBeanDefinitionScanner 리네임
        - 클래스 조회 역할 강조
    - 클래스패스에서 어노테이션 설정된 클래스 조회 후, BeanDefinition을 생성해 BeanFactory에 전달
        
        <aside>
        💡 **Class Path
        
        -** 자바 애플리케이션에서 클래스와 리소스를 찾는 데 사용되는 경로
        - 디렉토리 및 JAR 파일의 집합
        - 자바 애플리케이션은 클래스패스에서 클래스 파일을 로드하여 해당 클래스를 사용하거나 리소스 파일을 찾아서 사용
        
        </aside>
        
- **BeanDefinitionRegistry**
    - BeanDefinition 저장소
    - BeanDefinition과의 의존 관계를 느슨하게 하기 위해
    
    ```java
    public interface BeanDefinitionRegistry {
        void registerBeanDefinition(Class<?> clazz, BeanDefinition beanDefinition);
    }
    ```
    
- **BeanFactory**
    - BeanDefinition 저장
    - BeanDefinition 활용해 빈 인스턴스 생성
    - 의존관계 주입
    - BeanDefinitionRegistry 구현 클래스
        - BeanDefinition 정보 관리
        
        ```java
        public class BeanFactory implements BeanDefinitionRegistry {
            private static final Logger log = LoggerFactory.getLogger(BeanFactory.class);
        
            private Map<Class<?>, Object> beans = Maps.newHashMap();
        
            **private Map<Class<?>, BeanDefinition> beanDefinitions = Maps.newHashMap();**
        
            public void initialize() {
                for (Class<?> clazz : getBeanClasses()) {
                    getBean(clazz);
                }
            }
        
            public Set<Class<?>> getBeanClasses() {
                return beanDefinitions.keySet();
            }
        
            @SuppressWarnings("unchecked")
            public <T> T getBean(Class<T> clazz) {
                Object bean = beans.get(clazz);
                if (bean != null) {
                    return (T) bean;
                }
        
                Class<?> concreteClass = findConcreteClass(clazz);
                BeanDefinition beanDefinition = beanDefinitions.get(concreteClass);
                bean = inject(beanDefinition);
                beans.put(concreteClass, bean);
                return (T) bean;
            }
        
            private Class<?> findConcreteClass(Class<?> clazz) {
                Set<Class<?>> beanClasses = getBeanClasses();
                Class<?> concreteClazz = BeanFactoryUtils.findConcreteClass(clazz, beanClasses);
                if (!beanClasses.contains(concreteClazz)) {
                    throw new IllegalStateException(clazz + "는 Bean이 아니다.");
                }
                return concreteClazz;
            }
        
            private Object inject(BeanDefinition beanDefinition) {
                if (beanDefinition.getResolvedInjectMode() == InjectType.INJECT_NO) {
                    return BeanUtils.instantiate(beanDefinition.getBeanClass());
                } else if (beanDefinition.getResolvedInjectMode() == InjectType.INJECT_FIELD) {
                    return injectFields(beanDefinition);
                } else {
                    return injectConstructor(beanDefinition);
                }
            }
        
            private Object injectConstructor(BeanDefinition beanDefinition) {
                Constructor<?> constructor = beanDefinition.getInjectConstructor();
                List<Object> args = Lists.newArrayList();
                for (Class<?> clazz : constructor.getParameterTypes()) {
                    args.add(getBean(clazz));
                }
                return BeanUtils.instantiateClass(constructor, args.toArray());
            }
        
            private Object injectFields(BeanDefinition beanDefinition) {
                Object bean = BeanUtils.instantiate(beanDefinition.getBeanClass());
                Set<Field> injectFields = beanDefinition.getInjectFields();
                for (Field field : injectFields) {
                    injectField(bean, field);
                }
                return bean;
            }
        
            private void injectField(Object bean, Field field) {
                log.debug("Inject Bean : {}, Field : {}", bean, field);
                try {
                    field.setAccessible(true);
                    field.set(bean, getBean(field.getType()));
                } catch (IllegalAccessException | IllegalArgumentException e) {
                    log.error(e.getMessage());
                }
            }
        
            public void clear() {
                beanDefinitions.clear();
                beans.clear();
            }
        
            **@Override
            public void registerBeanDefinition(Class<?> clazz, BeanDefinition beanDefinition) {
                log.debug("register bean : {}", clazz);
                beanDefinitions.put(clazz, beanDefinition);
            }**
        }
        ```
        
- **ApplicationContext**
    - BeanFactory와 ClasspathBeanDefinitionScanner 간의 DI를 담당
    - 의존 관계 연결하고 초기화
    
    ```java
    public class ApplicationContext {
        private BeanFactory beanFactory;
    
        public ApplicationContext(Object... basePackages) {
            beanFactory = new BeanFactory();
            ClasspathBeanDefinitionScanner scanner = new ClasspathBeanDefinitionScanner(beanFactory);
            scanner.doScan(basePackages);
            beanFactory.initialize();
        }
    
        public <T> T getBean(Class<T> clazz) {
            return beanFactory.getBean(clazz);
        }
    
        public Set<Class<?>> getBeanClasses() {
            return beanFactory.getBeanClasses();
        }
    }
    ```
    
- **BeanDefinition**
    - 빈 클래스 정보
    
    ```java
    public class BeanDefinition {
        private Class<?> beanClazz;
        private Constructor<?> injectConstructor;
        private Set<Field> injectFields;
    
        public BeanDefinition(Class<?> clazz) {
            this.beanClazz = clazz;
            injectConstructor = getInjectConstructor(clazz);
            injectFields = getInjectFields(clazz, injectConstructor);
        }
    
    		// constructor
        private static Constructor<?> getInjectConstructor(Class<?> clazz) {
            return BeanFactoryUtils.getInjectedConstructor(clazz);
        }
    		
    		// 필드, setter
        private Set<Field> getInjectFields(Class<?> clazz, Constructor<?> constructor) {
            // 생성자 존재 여부 판단
    				if (constructor != null) {
                return Sets.newHashSet();
            }
    
            Set<Field> injectFields = Sets.newHashSet();
            Set<Class<?>> injectProperties = getInjectPropertiesType(clazz);
            Field[] fields = clazz.getDeclaredFields();
            for (Field field : fields) {
                if (injectProperties.contains(field.getType())) {
                    injectFields.add(field);
                }
            }
            return injectFields;
        }
    		
    		// 주입할 대상의 객체 타입을 결정
        private static Set<Class<?>> getInjectPropertiesType(Class<?> clazz) {
            Set<Class<?>> injectProperties = Sets.newHashSet();
            Set<Method> injectMethod = BeanFactoryUtils.getInjectedMethods(clazz);
            for (Method method : injectMethod) {
                Class<?>[] paramTypes = method.getParameterTypes();
                if (paramTypes.length != 1) {
                    throw new IllegalStateException("DI할 메소드 인자는 하나여야 합니다.");
                }
    
                injectProperties.add(paramTypes[0]);
            }
    
            Set<Field> injectField = BeanFactoryUtils.getInjectedFields(clazz);
            for (Field field : injectField) {
                injectProperties.add(field.getType());
            }
            return injectProperties;
        }
    
        public Constructor<?> getInjectConstructor() {
            return injectConstructor;
        }
    
        public Set<Field> getInjectFields() {
            return this.injectFields;
        }
    
        public Class<?> getBeanClass() {
            return this.beanClazz;
        }
    
        public InjectType getResolvedInjectMode() {
            if (injectConstructor != null) {
                return InjectType.INJECT_CONSTRUCTOR;
            }
    
            if (!injectFields.isEmpty()) {
                return InjectType.INJECT_FIELD;
            }
    
            return InjectType.INJECT_NO; // 주입 필요 없음
        }
    }
    ```
    
    <aside>
    💡 **`Sets.newHashSet()` vs `new HashSet<>()`**
    
    - 둘 다 **`HashSet`**의 인스턴스를 생성하는 메서드
    - **`new HashSet<>()`**
        - **`HashSet`**의 생성과 관련된 모든 결정을 개발자가 수행
        - 생성자에 매개변수를 추가하거나 다른 하위 클래스를 사용 가능
    - **`Sets.newHashSet()`**
        - Guava 라이브러리의 **`Sets`** 클래스의 정적 팩토리 메서드인 **`newHashSet()`**를 사용하여 **`HashSet`**의 인스턴스를 생성
        - **`Sets`** 클래스는 **`HashSet`**뿐만 아니라 **`LinkedHashSet`**, **`TreeSet`** 등 다양한 집합 구현을 생성할 수 있는 팩토리 메서드를 제공
        - **`Sets.newHashSet()`**를 사용하면 향후 구현을 변경하거나 다른 구현을 사용 가능(유연성)
    </aside>
    

### 四，추가적인 리팩토링

- **문제1 : DispatcherServlet의 init()에 패키지 이름 하드 코딩**
    - 해결 방안
        - 기본 패키지명을 외부에서 전달
        - 서블릿에 인자를 전달
    - **방법1 : web.xml 설정으로 서블릿에 인자 전달**
    - **방법2 :ServletContainerInitializer 인터페이스 구현**
        - **ServletContainerInitializer**
            - 웹 애플리케이션의 초기화 과정을 커스터마이즈
            - 서블릿 컨테이너에게 필요한 구성 요소들을 동적으로 제공
        - **WebApplicationInitializer**
            - 확장을 위해 자체적으로 사용할 인터페이스
        - **MyServletContainerInitializer**
            - ServletContainerInitializer의 구현체
        - **@HandlesTypes**
            - 애플리케이션 클래스 타입을 찾아내고, 해당 클래스 타입에 대한 초기화 작업
            - WebApplicationInitializer를 추가
        - **서블릿 컨테이너**
            - 클래스패스에 존재하는 클래스 중 WebApplicationInitializer의 구현체를 찾아 onStartUp()의 인자로 전달
            - WebApplicationInitializer의 구현체의 onStartUp() 사용하려면 클래스패스에 추가
            - WebApplicationInitializer 인터페이스를 구현함으로써 서블릿 컨테이너 초기화 과정 확장 가능
        
        ```java
        **@HandlesTypes(WebApplicationInitializer.class)**
        public class MyServletContainerInitializer implements ServletContainerInitializer {
            @Override
            public void onStartup(Set<Class<?>> webAppInitializerClasses, ServletContext servletContext)
                    throws ServletException {
                List<WebApplicationInitializer> initializers = new LinkedList<WebApplicationInitializer>();
        
                if (webAppInitializerClasses != null) {
                    for (Class<?> waiClass : webAppInitializerClasses) {
                        try {
                            initializers.add((WebApplicationInitializer) waiClass.newInstance());
                        } catch (Throwable ex) {
                            throw new ServletException("Failed to instantiate WebApplicationInitializer class", ex);
                        }
                    }
                }
        
                if (initializers.isEmpty()) {
                    servletContext.log("No Spring WebApplicationInitializer types detected on classpath");
                    return;
                }
        
                for (WebApplicationInitializer initializer : initializers) {
                    initializer.onStartup(servletContext);
                }
            }
        }
        ```
        
    
    <aside>
    💡 **core, next**
    
    Spring 프레임워크의 아키텍처 및 관례에 따른 것
    ****core **:** 주요 비즈니스 로직이나 핵심 기능을 담고 있는 모듈
    next : core 모듈에 추가적인 기능이나 확장을 제공하는 모듈들을 담을 수 있는 공간
    
    ![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bf0db937-4b81-4fe2-94e8-f5ccb9cba9e3/Untitled.png)
    
    </aside>
    
- **문제 2 : 외부라이브러리 클래스에서 빈 등록**
    - 외부라이브러리는 클래스가 아닌 메소드를 사용해야하기 때문에, 메소드 어노테이션으로 등록

### 五，유연성과 확장성

- 중복 제거 방법
    - 상속
    - 조합
        - 인터페이스 활용하고, **DI기반**으로 객체 간의 의존관계 연결

<aside>
💡 객체 간의 의존관계를 인터페이스를 통해 분리한 후, DI를 통해 연결 → 유연성, 확장성

</aside>

- 인터페이스로 분리
    - 확장이 필요한 부분을 인터페이스로 구현
    - 구현 클래스가 아닌 인터페이스 통해서 의존 관계 갖도록 연결
    - 서로 다르게 변화하는 구현 클래스 사이의 연결은 팩토리 클래스가 담당
        - 팩토리 클래스 : ApplicationContext
    - 기능마다 변화가 발생하는 시점이 다른 부분은 서로 다르게 변화, 발전 가능
- 예시
    - 스프링 프레임워크가 버전 업데이트하면서 하위 버전 지원
    - BeanDefinitionReader는 BeanDefinition 생성을 위한 어노테이션 설정만 지원하는 것이 아니라, XML, Property 설정도 지원 가능
    - BeanDefinitionRegistry는 BeaDefinitionReader의 변화에 영향 받지 않고 독립적으로 확장 가능

<aside>
💡 이 정도 수준의 유연성과 확장성이 필요한 경우는 거의 없고, 어플리케이션 자체에서만 활용해도 충분한 코드가 대분

</aside>

- DI 컨테이너
    - 클래스간의 의존관계 파악해 매번 DI를 하는 것은 불편하고, 의존관계를 잘못 연결하는 경우 문제가 발생할 가능성 큼
    - DI 컨테이너로 객체 간의 의존관계 연결을 좀 더 쉽고, 안전하게 연결해 사능
    - 스프링 프레임워크는 DI 컨테이너 기능 지원
