# 7장 DB를 활용해 데이터를 영구적으로 저장하기

- 데이터베이스 서버 도입
    - 서버를 재시작하면 사용자가 입력한 데이터가 사라지는 문제 (HTTP 웹 서버)
- 자바 진영은 JDBC 라는 표준을 통해 데이터베이스와의 통신을 담당하도록 지원
    - **JDBC**

        - Java Database Connectivity의 약자로 JDBC는 자바에서 데이터베이스에 접속할 수 있도록 하는 자바 API
        - JDBC는 DBMS에 종속되지 않는 관련 API를 제공한다. 즉, 여러 가지 DBMS에 대해 동일한 인터페이스를 제공하여 개발자가 특정 DBMS에 종속되지 않고 코드를 작성할 수 있도록 합니다.
        - JDBC API는 JDK에서 제공하며, 따라서 JDK를 설치하면 JDBC API를 사용할 수 있다.
        - JDBC프로그래밍을 위해서는 JDBC드라이버가 필요하다.
        - JDBC는 DBMS에 연결하고 쿼리를 실행하기 위한 도구입니다. 개발자들은 실제 구동 코드를 작성하는 대신 JDBC를 사용하여 DBMS에 연결하고 쿼리를 실행할 수 있습니다.
        - JDBC는 각 DBMS에 대한 차이점을 추상화하고 단일 인터페이스를 제공하기 때문에 개발자가 특정 DBMS를 사용할 때 발생하는 차이를 신경 쓰지 않고 일관된 방식으로 작업할 수 있도록 도와줍니다.
    - **JDBC드라이버**
        - 각 DBMS 제조업체는 자체적으로 JDBC 드라이버를 제공한다. 드라이버는 해당 DBMS와의 통신을 담당하며, JDBC API를 사용하여 DBMS에 접속하고 쿼리를 실행할 수 있게 해줍니다.
        - 오라클을 사용하면 오라클용 JDBC드라이버, mysql을 사용하면 mysql JDBC드라이버
    
    → JDBC는 DBMS와의 통신을 담당하는 중간 계층으로 볼 수 있다. 각 DBMS마다 별도의 JDBC 드라이버를 제공하며, JDBC는 이러한 드라이버를 통해 표준화된 방식으로 DBMS와 통신할 수 있도록 합니다.
    
    - 
        - jdbc는 오라클을 연결하기위한, 접속하기 위한 도구들을 어플리케이션을 만드는 사용자들이 직접 쓰지 않게 하려고 하는 것이다. 
        실제 구동하기위한 코드는 jdbc driver에 있다.
        우리는 직접 쓰지 않고 단일화 시키기위한 도구를 만들었는데 그게 jdbc
        자바가 oracle을 사용할때랑 mysql을 사용할때 차이가 나는 부분들을 단일화시키는 도움
        jdbc는 깡통이라고 생각하면 된다. 역할은 다른나라 여행할 때 그 나라마다 어댑터를 가져가는데 jdbc가 그런 역할이라 생각하면 된다.
        우리가 만들때는 jdbc driver을 직접쓰지않고 jdbc를 통해서 간접적으로 쓰게되면은 각각의 dbms들이 가지고 있는 차이를 알 필요도 없다. jdbc는 항상 같은 함수 이름을 갖고있는 기능을 제공할거라서
        - JDBC 사용 절차
            1. 드라이버 로드 : jdbc가 깡통이니까 실질적인 구동장치인 jdbc driver  로드해야함
            2. 연결  생성하기
            3. 쿼리 실행하기
            4. 결과집합 사용하기
- 이번 장 목표
    - 회원 데이터를 데이터베이스 서버에서 관리하고 JDBC API를 통해 접근하도록 구현

## 7.1 회원 데이터를 DB에 저장하기 실습

### 7.1.1 실습 코드 리뷰 및 JDBC 복습

- 서버가 시작하는 시점에 회원 정보를 저장할 테이블 초기화 → 톰캣 서버가 시작할 때 초기화되도록 ContextLoaderListener 클래스에 구현
- extends, implements(interface 구현)
    - extends : 클래스 상속을 위한 키워드, 자식 클래스는 부모 클래스의 모든 멤버(필드, 메서드)를 상속 → 자식 클래스에서 부모 클래스의 멤버를 직접 사용하고 수정할 수 있음
    상속을 통해 코드의 재사용성과 구조화
    - implements : 클래스가 인터페이스(=메소드 선언만 포함)에서 정의한 메소드를 구현할 때 implements키워드 사용
- ServletContextListener
    - 웹 어플리케이션이 시작되거나 종료될 때  호출할 메서드를 정의한 인터페이스
    - public void contextInitialized(ServletContextEvent sce) : 웹어플리케이션을 초기화할 때 호출
    - public void contextDestroyed(ServletContextEvent sce) : 웹 어플리케이션을 종료할 때 호출
- @WebListener 애노테이션: servlet-api 라이브러리를 가져와서 사용할 수 있는 애노테이션으로 클래스 위에 @WebListener를 붙임으로써 톰캣서버에게 Listener 클래스임을 알려주는 것이다.
- 서블릿 컨테이너는 ServletContextListener인터페이스 구현체 중 @WebListener 애노테이션이 설정되어 있으면 서블릿 컨테이너를 시작하는 과정에서 contextInitialized()메소드를 호출해 초기화 작업을 진행
- DAO : 데이터베이스에 대한 접근 로직 처리를 담당하는 객체
- 기존에 DataBase 클래스를 사용하던 코드를 UserDao를 사용하도록 변경
- 


### 7.1.2 회원 목록 실습

```java
public List<User> findAll() throws SQLException {
    	List<User> userlist = new ArrayList<>();
        Connection con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;
        try {
            con = ConnectionManager.getConnection();
            String sql = "SELECT userId, password, name, email FROM USERS";
            pstmt = con.prepareStatement(sql);

            rs = pstmt.executeQuery();

            User user = null;
            if (rs.next()) {
                user = new User(rs.getString("userId"), rs.getString("password"), rs.getString("name"),
                        rs.getString("email"));
                userlist.add(user);
            }

        } finally {
            if (rs != null) {
                rs.close();
            }
            if (pstmt != null) {
                pstmt.close();
            }
            if (con != null) {
                con.close();
            }
        }
        return userlist;
    }
```

### 7.1.3 개인정보 수정 실습

```java
public void update(User user) throws SQLException {
        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = ConnectionManager.getConnection();
            String sql = "UPDATE users SET password=?, name=?, email=? WHERE userid=?";
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, user.getPassword());
            pstmt.setString(2, user.getName());
            pstmt.setString(3, user.getEmail());
            pstmt.setString(4, user.getUserId());

           pstmt.executeUpdate();

        } finally {

            if (pstmt != null) {
                pstmt.close();
            }
            if (con != null) {
                con.close();
            }
        }
    }
```

## 7.2 DAO 리팩토링 실습

- 중복 코드를 리팩토링 하기위해 
변화가 발생하는 부분(깨발자가 구현할 수 밖에 없는 부분)
변화가 없는 부분(공통 라이브러리로 분리할 부분)
으로 분리
- 개발자가 구현할 부분 : SQL 쿼리, 쿼리에 전달할 인자, SELECT 구문의 경우 조회한 데이터를 추출
- 나머지는 라이브러리로 위임가능
- 리팩토링시 SQLException : SQLException은 컴파일타임 Exception
- 에러와 예외
    - 에러(error) : 프로그램 코드에 의해서 수습될 수 없는 심각한 오류 (예: JVM의 메모리 부족, 스택 오버플로우)
    - 예외(exception) : 프로그램 코드에 의해서 수습될 수 있는 다소 미약한 오류
    → 예외는 에러와 달리 문제가 발생하더라도 이에 대한 **대응 코드**를 미리 작성해 놓음으로써 어느정도 프로그램의 비정상적인 종료 혹은 동작을 막을 수 있다.
    예외에 대한 대응 코드가 바로 예외 처리 try - catch
- 컴파일에러와 런타임 에러


- **자바에서 예외**는 크게 두 가지로 분류됩니다: 컴파일타임 예외(Checked Exception)와 런타임 예외(Unchecked Exception)입니다.
- **컴파일타임 예외(Checked Exception)**: Exception 클래스를 상속받고 RuntimeException 클래스를 상속받지 않은 예외
컴파일러에 의해 강제적으로 예외 처리가 요구된다.
개발자는 try-catch 블록을 사용하거나 메서드 선언에 throws 절을 추가하여 컴파일타임 예외를 처리해야 한다.
    - Exception 및 하위 클래스 : 사용자의 실수와 같은 외적인 요인에 의해 발생하는 컴파일시 발생하는 예외
        - 존재하지 않는 파일의 이름을 입력 (FileNotFoundException)
        - 실수로 클래스의 이름을 잘못 기재 (ClassNotFoundException)
        - 입력한 데이터 형식이 잘못된 경우 (DataFormatException)
- **런타임 예외(Unchecked Exception)**: RuntimeException 클래스와 그 하위 클래스들로 이루어진 예외
컴파일러에 의해 예외 처리를 강제하지 않는다.
개발자는 선택적으로 예외를 처리할 수 있다. 
즉, 예외를 처리하기 위해 try-catch 블록을 사용하지 않아도 컴파일 오류가 발생하지 않는다.
    - RuntimeException 클래스 : 프로그래머의 실수로 발생하는 예외, 주로 실행 중에 발생
        - 배열의 범위를 벗어남 (IndexOutOfBoundsException)
        - 값이 null인 참조 변수의 멤버를 호출 (NullPointerException)
        - 클래스 간의 형 변환을 잘못함 (ClassCastException)
        - 정수를 0으로 나누는 산술 오류 (ArithmeticException)

| API를 사용하는 모든 곳에서 이 예외를 처리해야하는가? | 예 | 컴파일타임 Exception |
| --- | --- | --- |
| 예외가 반드시 메소드에 대한 반환 값이 되어야 하는가? | 예 | 컴파일타임 Exception |
| API를 사용하는 소수 중 이 예외를 처리해야 하는가? | 예 | 런타임 Exception |
| 무엇인가 큰 문제가 발생했는가? 이 문제를 복구할 방법이 없는가? | 예 | 런타임 Exception |
| 아직도 불명확한가? | 예 | 런타임 Exception |

→ 대부분 **런타임 Exception**

⇒ JDBC에 대한 공통 라이브러리를 만드는 과정에서 SQLException을 런타임 Exception으로 변환할 것임

### 7.2.1 요구사항

- 목표 : JDBC에 대한 공통 라이브러리를 만들기
→ 이를통해 개발자가 sql 쿼리, 쿼리에 전달할 인자, select 구문의 경우 조회한 데이터 추출
이 3가지 구현에만 집중할 수 있도록 함
→ SQLException을 런타임 Exception으로 변환하여 try/catch절로 인해 가독성을 해치지 않도록

### 7.2.2 요구사항 분리 및 힌트

- UserDao
- insert(User), update(User), findAll(), findByUserId(String)
1. INSERT,UPDATE 쿼리를 가지는 중복메소드의 중복 제거 작업 진행
2. 분리한 메소드 중에서 변화가 발생하지 않는 부분(즉, 공통 라이브러리로 구현한 코드)를 새로운 클래스로 추가후 이동
    - private접근 제어자라면 default 접근 제어자로 리팩토링
3. InsertJdbcTemplate 과 UpdateJdbcTemplate이 UserDao에 의한 의존관계를 가진다.
UserDao에 대한 의존관계를 끊는다.
    - 추상메소드 (abstract method)
        - 자식 클래스에서 반드시 오버라이딩해야만 사용할 수 있는 메소드
        - 추상 메소드를 선언하여 사용하는 **목적** : 추상 메소드가 포함된 클래스를 상속받는 자식 클래스가 반드시 추상 메소드를 구현하도록 하기 위함
        만약 일반 메소드로 구현한다면 사용자에 따라 해당 메소드를 구현할 수도 있고, 안 할 수도 있다. 하지만 추상 메소드가 포함된 추상 클래스를 상속받은 모든 자식 클래스는 추상 메소드를 구현해야만 인스턴스를 생성할 수 있으므로, 반드시 구현하게 된다.
        - 선언부만이 존재하며, 구현부는 작성되어 있지 않습니다. → 작성되어 있지 않은 구현부를 자식 클래스에서 오버라이딩하여 사용하는 것입니다.
        
        
        ```
        💡 abstract 반환타입 메소드이름();
       ```
        
    - 추상클래스
        - 하나 이상의 추상 메소드를 포함하는 클래스를 가리켜 추상 클래스(abstract class)
        
        ```
        💡 abstract class 클래스이름 {
        
        ...
        
        **abstract** 반환타입 **메소드이름**();
        
        ...
        
        }
        
        ```
        
        - 추상 클래스는 동작이 정의되어 있지 않은 추상 메소드를 포함하고 있으므로, 인스턴스를 생성할 수 없다.
        - 추상 클래스는 먼저 상속을 통해 자식 클래스를 만들고, 만든 자식 클래스에서 추상 클래스의 모든 추상 메소드를 오버라이딩하고 나서야 비로소 자식 클래스의 인스턴스를 생성할 수 있게 된다.
    - 익명클래스
        - 이름이 없는 클래스
        - 익명, 이름이 없다는 것은 별로 기억되지 않아도 된다는 것이며, 나중에 다시 불러질 이유가 없다는 뜻을 내포한다.  즉, 프로그램에서 일시적으로 한번만 사용되고 버려지는 객체라고 보면 된다. (일회용 클래스)
        
        - 어떤 클래스의 자원을 상속 받아 재정의하여 사용하기 위해서는 먼저 자식이 될 클래스를 만들고 상속(extends) 후에 객체 인스턴스 초기화를 통해 가능하다.
        
        ```java
        // 부모 클래스
        class Animal {
            public String bark() {
                return "동물이 웁니다";
            }
        }
        
        // 자식 클래스
        class Dog extends Animal {
        	@Override
            public String bark() {
                return "개가 짖습니다";
            }
        }
        
        public class Main {
            public static void main(String[] args) {
                Animal a = new Dog();
                a.bark();
            }
        }
        ```
        
        - 클래스 정의 없이 메소드 내에서 바로 클래스를 생성해 인스턴스화 할 수 있다.
        - 클래스의 선언과 객체의 생성을 동시에 하기 때문에 **단 한 번만 사용**
        - **부모 클래스의 자원을 상속받아 재정의하여 사용할 자식 클래스가 한번만 사용**되고 버려질 자료형이면, 굳이 상단에 클래스를 정의하기보다는, **지역 변수처럼** 익명 클래스로 정의하고 스택이 끝나면 삭제되도록 한다.
        
        ```java
        // 부모 클래스
        class Animal {
            public String bark() {
                return "동물이 웁니다";
            }
        }
        
        public class Main {
            public static void main(String[] args) {
                // 익명 클래스 : 클래스 정의와 객체화를 동시에. 일회성으로 사용
                Animal dog = new Animal() {
                	@Override
                    public String bark() {
                        return "개가 짖습니다";
                    }
                }; // 단 익명 클래스는 끝에 세미콜론을 반드시 붙여 주어야 한다.
                	
                // 익명 클래스 객체 사용
                dog.bark();
            }
        }
        ```
        
4. InsertJdbcTemplate과 UpdateJdbcTemplate의 구현 부분이 다른 부분이 없다. 둘 중의 하나를 사용하도록 리팩토링한다.
5. JdbcTemplate은 아직도 User와 의존관계를 가지기 때문에 DAO클래스에서 재사용할 수 없다.
User와 의존관계를 끊는다.
6. 더이상 JdbcTemplate은 특정 DAO클래스에 종속적이지 않다. 이와 똑같은 방법으로 selectJdbcTemplate을 생성해 반복 코드를 분리한다.
7. JdbcTemplate과 SelectJdbcTemplate을 보니 중복코드가 보인다. 또한 굳이 2개의 클래스를 제공하고 싶지 않다. JdbcTemplate과 같은 한 개의 클래스만을 제공하도록 리팩토링해본다.
8. 위와같이 SelectJdbcTemplate클래스로 통합했을 때의 문제점을 찾아보고 이를 해결하기 위한 방법을 찾아본다.
9. SQLException을 런타임Exception으로 변환해 throw하도록 한다. Connection, PrparedStatement자원 반납을 close()메소드를 사용하지 말고 try-with-resource구문을 적용해 해결한다.
    - try-with-resources : 자바 7버전부터 사용가능
        - try에 자원 객체를 전달하면, try 코드 블록이 끝나면 자동으로 자원을 종료해주는 기능
        - 즉, 따로 finally 블록이나 모든 catch 블록에 종료 처리를 하지 않아도 된다.
        
        ```java
        **try** (SomeResource resource = getResource()) {
            use(resource);
        } **catch**(...) {
            ...
        }
        ```
        
10. Select 문의 경우 조회한 데이터를 캐스팅하는 부분이 있다. 캐스팅하지 않고 구현하도록 계산한다.
    - 자바 제너릭
        - 데이터의 타입(data type)을 일반화한다(generalize)는 것을 의미합니다.
        - 제네릭은 클래스나 메소드에서 사용할 내부 데이터 타입을 컴파일 시에 미리 지정하는 방법입니다.
        - 자바에서 제네릭은 클래스와 메소드에만 다음과 같은 방법으로 선언
        
        ```java
        class MyArray<T> {  // T : 타입변수, 임의의 참조형 타입, T말고 다른 문자 사용가능
            T element;
            void setElement(T element) { this.element = element; }
            T getElement() { return element; }
        } 
        ```
        
        - 제네릭 클래스(generic class)를 생성할 때에는 타입 변수 자리에 사용할 실제 타입을 명시
        
        ```java
        MyArray<Integer> myArr = new MyArray<Integer>();
        ```
        
11. 각 쿼리에 전달할 인자를 PreparedStatementSetter를 통해 전달할 수도 있지만 자바의 가변인자를 통해 전달할 수 있는 메소드를 추가한다.
    - 가변인자
        - 기존에는 메서드의 매개변수 개수가 고정적이었나 JDK1.5부터 동적으로 지정 가능
        - 메서드의 개수를 동적으로 지정할 수 있는것
        - 가변 인자는 `타입... 변수명`과 같은 형식으로 선언
        - 가변인자는 매개변수 중에서 제일 마지막에 선언해야 한다.
            - 마지막에 선언하지 않으면 *컴파일 에러 발생*
            
            ```
            java: varargs parameter must be the last parameter
            ```
            
            가변인자를 제일 마지막에 선언하지 않을 경우, 가변인자인지 아닌지를 구별할 방법이 없기 때문에 허용하지 않는 것이다.
            
            [[Java] 가변인자 (varargs)](https://velog.io/@minseojo/Java-가변인자-varargs)
            
12. UserDao에서 PreparedStatementSetter, RowMapper 인터페이스를 구현하는 부분을 JDK 8에서 추가한 람다 표현식을 활용하도록 리팩토링한다.
    - jdk 7 람다 : 익명클래스 대신 람다를 적용하여 구현
        - 함수형 프로그래밍에서 사용되는 개념으로, 간결하게 익명 함수를 표현하는 방법
        
        ```java
        (parameters) -> expression
        ```
        
        - **`parameters`**: 메서드에 전달되는 매개변수의 목록입니다. 매개변수가 없을 경우 비워둘 수 있습니다.
        - **`->`** 람다 화살표로 불리며, 매개변수와 람다 표현식의 바디를 구분합니다.
        - **`expression`**: 람다 표현식의 몸체입니다. 한 줄의 코드로 표현될 수도 있고, 여러 줄의 코드 블록으로 표현될 수도 있습니다.
        - 람다 표현식을 사용하면 코드를 간결하게 만들 수 있다.
        - 익명 클래스를 사용하지 않고도 함수형 인터페이스의 구현을 인라인으로 제공할 수 있다.
        
        ```java
        // 익명 클래스를 사용한 예제
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello, World!");
            }
        };
        
        // 람다 표현식을 사용한 예제
        Runnable runnable = () -> System.out.println("Hello, World!");
        ```
        

## 7.3 동영상을 활용한 DAO리팩토링 실습

- 리팩토링 과정에서 어려움 
- 지금까지 서비스하던 기능이 정상적으로 동작하는 상태에서 리팩토링을 진행해야한다는 점
- 소스코드 리팩토링, 데이터베이스 리팩토링 과정에서 안정적으로 리팩토링을 하기위해 과도기적 단계 필요

## 7.4 DAO 리팩토링 및 설명

### 7.4.1 메소드 분리


- 중복 코드를 제거하기 위한 첫 번째 단계 : Extract Method 리팩토링
- 개발자가 데이터베이스 접근 로직을 구현할 때 매번 구현해야 하는 부분 
그렇지 않은 부분을 기준으로 메소드 분리
- insert()와 update() 메소드 분리


### 7.4.2 클래스 분리


- UserDao

```java
public class UserDao {
	public void insert(User user) throws SQLException{
		JdbcTemplate jdbcTemplate = new JdbcTemplate();
		jdbcTemplate.insert(user, this);
	}
	**public** void setParameters(User user, PreparedStatement pstmt) throws SQLException {
		pstmt.setString(1, user.getUserId());
		pstmt.setString(2, user.getPassword());
		pstmt.setString(3, user.getName());
		pstmt.setString(4, user.getEmail());
	}

	**public** String createQuery() {
		return "INSERT INTO USERS VALUES (?, ?, ?, ?)";
	}
```

- JdbcTemplate

```java
package next.support.context;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

import core.jdbc.ConnectionManager;
import next.dao.UserDao;
import next.model.User;

public class JdbcTemplate {
	public void insert(User user, **UserDao userDao)** throws SQLException {
		Connection con = null;
		PreparedStatement pstmt = null;
		try {
			con = ConnectionManager.getConnection();
			String sql = userDao.createQuery();
			pstmt = con.prepareStatement(sql);
			userDao.setParameters(user, pstmt);
			pstmt.executeUpdate();
		} finally {
			if (pstmt != null) {
				pstmt.close();
			}
			if (con != null) {
				con.close();
			}
		}
	}
}
```

- insert() : 공통 라이브러리로 구현할 부분
- setParameters, createQuery() : 개발자가 매번 구현해야 할 부분
- 새로운 클래스를 추가하여 공통으로 구현할 부분을 구현한다.
    - 이때 setParameters,createQuery 와 같은 기존의 메소드가 없어서 컴파일 에러
    
    → 이를 해결하기 위해 UserDao 인스턴스를 인자로 전달하여 UserDao 메소드 호출하도록 변경
    
    - setParameters,createQuery에 접근 가능하도록 접근제한자를 default 혹은 public으로 변경

### 7.4.3 UserDao와 InsertJdbcTemplate의 의존관계 분리

```java
public **abstract class JdbcTemplate** {
	public void insert(User user) throws SQLException {
		Connection con = null;
		PreparedStatement pstmt = null;
		try {
			con = ConnectionManager.getConnection();
			String sql = createQuery();
			pstmt = con.prepareStatement(sql);
			setParameters(user, pstmt);
			pstmt.executeUpdate();
		} finally {
			if (pstmt != null) {
				pstmt.close();
			}
			if (con != null) {
				con.close();
			}
		}
	}
	**public abstract String createQuery();**
	**public abstract void setParameters(User user, PreparedStatement pstmt) throws SQLException;**
}
```

```java
public class UserDao {
	public void insert(User user) throws SQLException {
		**JdbcTemplate jdbcTemplate = new JdbcTemplate() {
			@Override**
			public String createQuery() {
				return "INSERT INTO USERS VALUES (?, ?, ?, ?)";
			}
			**@Override**
			public void setParameters(User user, PreparedStatement pstmt) throws SQLException {
				pstmt.setString(1, user.getUserId());
				pstmt.setString(2, user.getPassword());
				pstmt.setString(3, user.getName());
				pstmt.setString(4, user.getEmail());
			**}**
		**};**
		jdbcTemplate.insert(user);
	}
```

- 새로 추가한 클래스에서 UserDao와 의존관계를 가지고 있기 때문에 UserDao가 아닌 다른 곳에서 사용할 수 없다.
- UserDao 제거
- 메소드는 존재하지만 구현을 담당하지 않게하기위해 두 개의 메소드를 추상 메소드로 구현
- 추상클래스의 인스턴스를 생성하려면 추상 메소드를 구현
    - 추상메소드 구현방법
        1. InsertJdbcTemplate을 상속하는 새로운 클래스 추가
        2. 익명의 클래스 추가
- 템플릿 메소드 패턴
    - 반복적으로 발생하는 중복 로직을 상위 클래스가 구현하고 변화가 발생하는 부분만 추상 메소드로 만들어 구현하도록 하는 디자인패턴

### 7.4.4 InsertJdbeTemplate과 UpdateJdbcTemplate 통합

```java
public class UserDao {
	public void insert(User user) throws SQLException {
		JdbcTemplate jdbcTemplate = new JdbcTemplate() {
			@Override
			public void setParameters(PreparedStatement pstmt) throws SQLException {
				pstmt.setString(1, user.getUserId());
				pstmt.setString(2, user.getPassword());
				pstmt.setString(3, user.getName());
				pstmt.setString(4, user.getEmail());
			}
		};
		String sql =  "INSERT INTO USERS VALUES (?, ?, ?, ?)";
		jdbcTemplate.executeUpdate(sql);
	}

	public void update(User user) throws SQLException {
		JdbcTemplate template = new JdbcTemplate() {
			@Override
			public void setParameters(PreparedStatement pstmt) throws SQLException {
				pstmt.setString(1, user.getPassword());
				pstmt.setString(2, user.getName());
				pstmt.setString(3, user.getEmail());
				pstmt.setString(4, user.getUserId());	
			}
		};
		String sql =  "UPDATE users SET password=?, name=?, email=? WHERE userid=?";
		template.executeUpdate(sql);
	}
```

```java
public abstract class JdbcTemplate {
	public void executeUpdate(String sql) throws SQLException {
		Connection con = null;
		PreparedStatement pstmt = null;
		try {
			con = ConnectionManager.getConnection();
			pstmt = con.prepareStatement(sql);
			setParameters(pstmt);
			pstmt.executeUpdate();
		} finally {
			if (pstmt != null) {
				pstmt.close();
			}
			if (con != null) {
				con.close();
			}
		}
	}

	public abstract void setParameters(PreparedStatement pstmt) throws SQLException;
}
```

### 7.4.5 User 의존관계 제거 및 SQL 쿼리 인자로 전달

- public void insert(User user) throws SQLException{}
    - 모든 테이블의 insert문에 사용하고 싶은데 user라는 데이터한테 종속되어버린 상황.  
    (User user) user에 종속적이라는 뜻이 사람들이 insert할 때 공통으로 이걸 쓰고싶은데 , 즉 회원을 insert할때랑 게시글을 insert할 때 모두 template에서 쓰고싶은데 User때문에 회원만 추가할 수 있다.

### 7.4.6 SELECT문에 대한 리팩토링

```java
public abstract class SelectJdbcTemplate {
	public Object executeQuery(String sql) throws SQLException {
		Connection con = null;
		PreparedStatement pstmt = null;
		ResultSet rs = null;
		try {
			con = ConnectionManager.getConnection();
			pstmt = con.prepareStatement(sql);
			setParameters(pstmt);

			rs = pstmt.executeQuery();

			return mapRow(rs);
		} finally {
			if (rs != null) {
				rs.close();
			}
			if (pstmt != null) {
				pstmt.close();
			}
			if (con != null) {
				con.close();
			}
		}
	}
	public abstract void setParameters(PreparedStatement pstmt) throws SQLException;
	public abstract Object mapRow(ResultSet rs) throws SQLException;
}
```

```java
public Object findByUserId(String userId) throws SQLException {
		SelectJdbcTemplate template = new SelectJdbcTemplate() {
			
			@Override
			public void setParameters(PreparedStatement pstmt) throws SQLException {
				pstmt.setString(1, userId);
				
			}
			
			@Override
			public User mapRow(ResultSet rs) throws SQLException {
				User user = null;
				if (rs.next()) {
					user = new User(rs.getString("userId"), rs.getString("password"), rs.getString("name"),
							rs.getString("email"));
				}

				return user;
			}
		};
		String sql = "SELECT userId, password, name, email FROM USERS WHERE userid=?";
		return (User)template.executeQuery(sql);
	}
```

- → setParameter처럼 결정할 수 없는 부분을 추상 메소드로 만들고 클래스를 추상메소드를 만드는 디자인 패턴

### 7.4.7 JdbcTemplate과 SelectJdbcTemplate 통합하기

- 기존의 SelectJdbcTemplate  클래스 삭제 가능
- 데이터베이스 접근 로직 처리에 대한 공통 라이브러리를 JdbcTemplate 하나로 제공
    - insert,update문에 mapRow()메소드를 강제 구현해야하는 문제점 발생
    
    ⇒ 템플릿 메소드 패턴의 단점임
    
    : 부모클래스에서 새로운 추상메소드가 추가/변경되었을 때 추상클래스를 implements하고있던 모든 클래스에 영향을 받는다는 단점이 있음
    

```java
public class UserDao {
	public void insert(User user) throws SQLException {
		JdbcTemplate jdbcTemplate = new JdbcTemplate() {
			@Override
			public void setParameters(PreparedStatement pstmt) throws SQLException {
				pstmt.setString(1, user.getUserId());
				pstmt.setString(2, user.getPassword());
				pstmt.setString(3, user.getName());
				pstmt.setString(4, user.getEmail());
			}

			@Override
			public Object mapRow(ResultSet rs) throws SQLException {
				return null;
			}
		};
		String sql =  "INSERT INTO USERS VALUES (?, ?, ?, ?)";
		jdbcTemplate.executeUpdate(sql);
	}

	public void update(User user) throws SQLException {
		JdbcTemplate template = new JdbcTemplate() {
			@Override
			public void setParameters(PreparedStatement pstmt) throws SQLException {
				pstmt.setString(1, user.getPassword());
				pstmt.setString(2, user.getName());
				pstmt.setString(3, user.getEmail());
				pstmt.setString(4, user.getUserId());	
			}

			@Override
			public Object mapRow(ResultSet rs) throws SQLException {
				// TODO Auto-generated method stub
				return null;
			}
		};
		String sql =  "UPDATE users SET password=?, name=?, email=? WHERE userid=?";
		template.executeUpdate(sql);
	}

	public List<User> findAll() throws SQLException {
		List<User> userlist = new ArrayList<>();
		Connection con = null;
		PreparedStatement pstmt = null;
		ResultSet rs = null;
		try {
			con = ConnectionManager.getConnection();
			String sql = "SELECT userId, password, name, email FROM USERS";
			pstmt = con.prepareStatement(sql);

			rs = pstmt.executeQuery();

			User user = null;
			if (rs.next()) {
				user = new User(rs.getString("userId"), rs.getString("password"), rs.getString("name"),
						rs.getString("email"));
				userlist.add(user);
			}

		} finally {
			if (rs != null) {
				rs.close();
			}
			if (pstmt != null) {
				pstmt.close();
			}
			if (con != null) {
				con.close();
			}
		}
		return userlist;
	}

	public Object findByUserId(String userId) throws SQLException {
		JdbcTemplate template = new JdbcTemplate() {
			
			@Override
			public void setParameters(PreparedStatement pstmt) throws SQLException {
				pstmt.setString(1, userId);
				
			}
			
			@Override
			public User mapRow(ResultSet rs) throws SQLException {
				User user = null;
				if (rs.next()) {
					user = new User(rs.getString("userId"), rs.getString("password"), rs.getString("name"),
							rs.getString("email"));
				}

				return user;
			}
		};
		String sql = "SELECT userId, password, name, email FROM USERS WHERE userid=?";
		return (User)template.executeQuery(sql);
	}

}
```

### 7.4.8 인터페이스 추가를 통한 문제점 해결

```java
public interface RowMapper {
	Object mapRow(ResultSet rs) throws SQLException;
}

public interface PreparedStatementSetter {
	void setParameters(PreparedStatement pstmt) throws SQLException;
}
```

```java
public class JdbcTemplate {
	public void executeUpdate(String sql, PreparedStatementSetter pss) throws SQLException {
		Connection con = null;
		PreparedStatement pstmt = null;
		try {
			con = ConnectionManager.getConnection();
			pstmt = con.prepareStatement(sql);
			pss.setParameters(pstmt);
			pstmt.executeUpdate();
		} finally {
			if (pstmt != null) {
				pstmt.close();
			}
			if (con != null) {
				con.close();
			}
		}
	}
	
	public Object executeQuery(String sql, PreparedStatementSetter pss, RowMapper rm) throws SQLException {
		Connection con = null;
		PreparedStatement pstmt = null;
		ResultSet rs = null;
		try {
			con = ConnectionManager.getConnection();
			pstmt = con.prepareStatement(sql);
			pss.setParameters(pstmt);

			rs = pstmt.executeQuery();

			return rm.mapRow(rs);
		} finally {
			if (rs != null) {
				rs.close();
			}
			if (pstmt != null) {
				pstmt.close();
			}
			if (con != null) {
				con.close();
			}
		}
	}
	
}
```

- 추상클래스 제거됨
- 콜백 인터페이스

### 7.4.9 런타임 Exception 추가 및 AutoClosable 활용한 자원 반환

- UserDao 문제점 : 모든 메소드가 컴파일타임 Exception인 SQLException을 throw함
- 런타임 Exception을 추가하여 문제점 해결

### 7.4.10 제너릭(generic)을 활용한 개선

- 데이터를 조회할 때 매번 캐스팅을 해야하는 점이 단점
- 자바의 제너릭을 적용하여 캐스팅하지 않도록 개선

### 7.4.11 가변인자를 활용해 쿼리에 인자 전달하기

- 쿼리에 가변 인자를 활용해 값을 전달
- 가변인자는 인자로 전달할 값의 갯수가 고정되지 않고 동적으로 변경되는 경우 유용하게 사용
- SQL문에 따라 전달할 값의 갯수가 달라지기 때문에 가변인자 활용하기 적절

### 7.4.12 람다를 활용한 구현

- UserDao에서 RowMapper에 대한 익명 클래스를 생성하던 부분을 람다를 활용하여 깔끔하게 구현
- 람다를 사용하려면 인터페이스의 메소드가 하나만 존재해야하고, 람다 표현식으로 사용할 인터페이스라고 지정하려면 인터페이스에 @FunctionalInterface 애노테이션 추가해야한다.

# 7.5 추가 학습 자료

### 7.5.1 데이터베이스

- 관계형 데이터베이스의 SQL을 기본 학습
- 성능을 고려한 설계 및 인덱스 활용에 대한 학습

### 7.5.2 디자인패턴

- “Head First Design Patterns” 책 참고하여 학습
