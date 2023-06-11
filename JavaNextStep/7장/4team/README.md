# 7장 DB를 활용해 데이터를 영구적으로 저장하기

- HTTP 웹 서버의 문제
    - 사용자가 입력한 데이터가 서버를 재시작하면 사라짐
    - Database Server 도입으로 해결
- JDBC
    - 구현 코드 거의 없고, 인터페이스만 정의
    - 데이터베이스 통신을 위한 규약만 정하고, 구현체는 서비스하는 회사가 제공
    - JDBC API 사용하는 소스 코드에 많은 중복 → JDBC 공통 라이브러리를 직접 구현해 해결
- Servlet
    - 인터페이스만 정의하고 서블릿 컨테이너를 만들어 제공하는 회사가 인터페이스 구현체 제공
    ![image](https://github.com/BookBBu/JavaWebProgrammingNextStep/assets/80014833/6bdce7fa-df95-40ab-b20a-8fa9b66a97ea)

 <br>
 
 
## 7.1 회원 데이터를 DB에 저장하기 실습
1. **데이터베이스 관련 설정**
    - H2 데이터베이스
        - 자바로 구현된 데이터베이스
        - jar 파일만 추가하면 별도의 설치 없이 사용 가능
2. **데이터베이스 스키마 초기화**
    - 톰캣 서버가 시작할 DB 초기화 작업 진행
        
        ```java
        @WebListener
        public class ContextLoaderListener implements ServletContextListener {
            private static final Logger logger = LoggerFactory.getLogger(ContextLoaderListener.class);
        
            // 톰켓서버가 시작할 때 contextInitialized 메소드를 호출해 초기화 작업을 할 수 있음
            
            @Override
            public void contextInitialized(ServletContextEvent sce) {
            
                ResourceDatabasePopulator populator = new ResourceDatabasePopulator();
                populator.addScript(new ClassPathResource("jwp.sql"));
                DatabasePopulatorUtils.execute(populator, ConnectionManager.getDataSource());
        
                logger.info("Completed Load ServletContext!");
            }
        
            @Override
            public void contextDestroyed(ServletContextEvent sce) {
            }
        }
        ```
        
    - 실무에서는 DBA가 혹은 DB Migration 도구 활용해 테이블 스키마 관리
3. **dao 구문**
    - UserDao 예시
        
        ```java
        public class UserDao {
        	public void insert(User user) throws SQLException {
        		Connection con = null;
        		PreparedStatement pstmt = null;
        		try {
        			con = ConnectionManager.getConnection();
        			String sql = "INSERT INTO USERS VALUES(?, ?, ?, ?)";
        			pstmt.setString(1, user.getUserId());
        			pstmt.setString(2, user.getPassword());
        			pstmt.setString(3, user.getName());
        			pstmt.setString(4, user.getEmail());
        
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
        		...
        }	
        ```
      
💡 **자바에서의 Database 처리**
```  
1. JDBC API
2. Spring JDBC 
   - DBCP를 pom.xml에 등록
   - bean으로 DataSource 등록해 객체화
3. iBatis/MyBatis
   - SQLMapper를 통해 DB 객체와 연결 (DB 종속적)
4. ORM (hibernate)
   - 자체 내장 SQL 함수 이용해 쿼리 결과  data에 직접 매칭 (객체지향적)
```

 <br>

## 7.2~7.4  DAO 리팩토링 실습

1. **extract method**
    
    
    | 작업 | 공통 라이브러리 | 개발자가 구현할 부분 |
    | --- | --- | --- |
    | Connection 관리 | O | X |
    | SQL | X | O |
    | Statement 관리 | O | X |
    | ResultSet관리 | O | X |
    | Row 데이터 추출 | X | O |
    | 파라미터 선언 | X | O |
    | 파라미터 Setting | O | X |
    | 트랜잭션 관리 | O | X |
    - 변화가 발생하는 부분 = 개발자가 구현해야 하는 부분
        
        → 추출한 메소드의 접근 제한자는 default로 변경
        
    - 변화가 없는 부분 = 공통 라이브러리로 분리할 수 있는 부분
        
        → JdbcTemplate
2. **(특정 dao와 변수에 대한) template 의존성 제거** 
    - 익명 클래스 이용
        - JdbcTemplate.java의 추상 메소드
            
            ![image](https://github.com/BookBBu/JavaWebProgrammingNextStep/assets/80014833/0b6643fd-9e2d-44e8-b6f9-8e0d06a6e24e)
            
        - UserDao.java의 익명 클래스
            
            ![image](https://github.com/BookBBu/JavaWebProgrammingNextStep/assets/80014833/ee19f706-1826-447e-85f5-6b078df09085)
            
        
   
        💡 **익명 클래스**
        ```
        이름이 없는 클래스
        객체 사용시에 클래스의 선언과 객체 생성이 동시에
        일회성으로 딱 하나의 객체만 필요할 경우 사용 
        익명 클래스 생성 시에는 상속 받을 부모 클래스 또는 구현할 인터페이스 이름과 함께 선언
        - 부모클래스이름 변수명 = new 부모클래스 이름 { ... 내용 구현 ... };
        - 인터페이스이름 변수명 = new 인터페이스 이름 { ... 내용 구현 ... };
        ```

        💡 **template method pattern**
        ```
        중복 로직을 상위 클래스가 구현, 변화가 발생하는 부분만 추상 메소드로 만들어 구현하는 디자인 패턴 
        ```
        
    - 반환 형 통일
        - Object로 변환
            - Casting 필수
            
           ![image](https://github.com/BookBBu/JavaWebProgrammingNextStep/assets/80014833/e51f873b-2e72-4ffa-950c-3b627efba83b)
           
        - 제네릭 이용
            - Casting을 하지 않아도 됨
            
            ![image](https://github.com/BookBBu/JavaWebProgrammingNextStep/assets/80014833/bca2df8c-dafc-499f-bd51-fec6b5400bd5)
            
            
            💡 **Generic T : `<T>`**
             ```
            사실 T는 변수 명과 같이 제네릭의 이름일 뿐 A / B.... 등 어떤 걸로 하든 크게 상관은 없음
            타입 T는 객체를 생성할 때 해당 타입으로 변경
            ```
                        
          
            💡 **Genric Method**
            ```
            매개변수의 타입과 리턴 타입을 제네릭 타입으로 받는 메소드
            public static <T,T2> void info(T t, T2 t2){ …
            ```
            
        - 가변 인자
            - SQL문에 따라 전달할 값의 개수가 달라지는데 가변 인자를 통해 똑같은 매개변수로 인자를 받을 수 있음
            
            ![image](https://github.com/BookBBu/JavaWebProgrammingNextStep/assets/80014833/d0fdb5ec-f51d-4f9f-960a-f48c6f83aee1)
3. **jdbcTemplate 하나로 통합**
    - executeQuery / executeUpdate로 메소드 통합
        - executeQuery : select
        - executeUpdate : insert, update, delete
    - 사용자 커스텀 Exception.class 생성
        - Dao의 각 메소드 마다, 컴파일 타임 Exception인 SQLException을 Throw
        - template에서 SQLException 처리 부분을 DataAcessException으로 넘겨 RuntimeException으로 처리하게 됨
        
        ```java
        public class DataAccessException **extends RuntimeException** {
        
        	private static final long serialVersionUID = 1L;
        	
        	public DataAccessException() {
        		super();
        	}
        	
        	public DataAccessException(String message, Throwable cause,
        			boolean enableSuppression , boolean writableStackTrace) {
        		super(message, cause,enableSuppression,writableStackTrace);
        	}
        	
        	public DataAccessException(String message, Throwable cause) {
        		super(message, cause);
        	}
        	
        	public DataAccessException(String message) {
        		super(message);
        	}
        	
        	public DataAccessException(Throwable cause) {
        		super( cause);
        	}
        }
        ```
        
       
        💡 **SQLException의 문제**
        ```
        SQLException은 Complie Exception이기 때문에 매번 try/catch절을 통해 Exception 처리가 필수
        catch문에서 특별히 처리할 코딩이 없음
        ```

        💡 **Exception 처리 가이드**
        ```
        - API를 사용하는 모든 곳에서 이 예외를 처리해야 함
            → Compile Exception
        - 예외가 반드시 메소드에 대한 반환 값이 되어야 함
            → Compile Exception
        - API를 사용하는 소수 중 이 예외를 처리해야 함
            → Runtime Exception
        - 큰 문제 발생 하고 문제 복구 방법이 없음
            → Runtime Exception 
            → 에러에 대한 정보를 통보 받는 것 밖에 방법이 없음
        - 불명확
            → Runtime Exception
            → 문서화하고 API를 사용하는 곳에서 예외 처리
        ```
        
    - try with resources 사용
        - 자원 반납할 때 close() 대신, java.io.AutoClosable 인터페이스를 구현하는 클래스인 try-with-resource 활용
            - BufferedReader, Connection, PreparedStatement 등 AutoClosable 상속 한 것
            
            ```java
            public  class JdbcTemplate {
            
            	public void update(String sql,PreparedStatementSetter pss) throws DataAccessException {
            		**try (Connection con = ConnectionManager.getConnection()**;
            				**PreparedStatement 	pstmt = con.prepareStatement(sql))**{
            		
            			pss.setValues(pstmt);
            			pstmt.executeUpdate();
            
            		} catch (SQLException e) {
            			throw new DataAccessException(e);
            		}
            	}
            ```
            
        - finally 절의 복잡성 낮아짐
            
            ```java
            public  class JdbcTemplate {
            
            	public void update(String sql,PreparedStatementSetter pss) throws DataAccessException {
            		Connection conn = null;
            		PreparedStatement pstmt = null;
            
            		try {
            			conn = ConnectionManager.getConnection();
            			pstmt = conn.prepareStatement(sql);
            			pss.setValues(pstmt);
            			pstmt.executeUpdate();
            
            		} catch (SQLException e) {
            			throw new DataAccessException(e);
            		} finally {
            			if (pstmt != null) {
            				**try {
            					pstmt.close();					
            				} catch(SQLException e) {
            					throw new DataAccessException(e);
            				}**
            			}
            
            			if (conn != null) {
            				try {
            					conn.close();
            				} catch(SQLException e) {
            					throw new DataAccessException(e);
            				}
            			}
            		}
            	}
            }
            ```
            
    - 콜백 인터페이스 사용
        - 변화 시점이 다른 부분을 서로 다른 인터페이스로 분리함으로써 공통 라이브러리에 대한 유연함을 높일 수 있게 됨
        - 두 개의 추상 메소드를 같은 클래스가 가지도록 구현x, 각각의 추상 메소드를 인터페이스를 통해 분리
        - rowMapper, prepareStateSetter interface로 구현해서 dao에서 인자로 template에 전하면 abstract method 제거 가능
            
            ![image](https://github.com/BookBBu/JavaWebProgrammingNextStep/assets/80014833/554424e8-8c50-4853-8178-54cb6f6693b0)
            
            ![image](https://github.com/BookBBu/JavaWebProgrammingNextStep/assets/80014833/38626a0a-9262-4df9-83c9-25d1924741c7)
            
        
       
        💡 **Callback Interface**
        ```
        일반적으로 Caller가 Callee 호출하지만, Callback은 Callee가 Caller를 호출하는 행위
        Caller는 Callee가 자신을 호출할 수 있도록 Callback이라는 Interface를 전달
        Callee는 Caller가 요청한 임무를 수행하던 도중 Event가 발생하면 Callback을 통해 Caller를 호출할 수 있음
        ```
 <br>

## 7.5.1 데이터베이스
- 관계형 데이터베이스의 SQL
- 성능을 고려한 설계 및 인덱스 활용
- ER 다이어그램
- 인덱스
- 정규화
- 트랜잭션
- NoSQL
    - 특징
        - RDBMS와 달리 데이터 간의 관계를 정의하지 않음
        - RDBMS에 비해 대용량(페타바이트 급)의 데이터를 저장 가능
        - 분산형 구조
        - 관계형 데이터베이스와 NoSQL을 같이 사용하는 경우가 많기 때문에, 백엔드 개발자가 데이터베이스 선정, 설계에 참여하는 경우가 많아짐
    - 종류
        - 카산드라
        - 카우치DB
        - 몽고DB
        - 레디스
       
        <br>
       
## 7.5.2 디자인 패턴
- 요구사항 정의
    - non funtional : 아키텍처 분석 설계
    - functional : 시스템 분석 설계 (디자인 패턴)
- 디자인 패턴
    1. 생성 패턴
        - 객체의 생성에 관련된 패턴
        - 종류
            - Abstract Factory
            - Builder
            - Factory Method
            - Prototype
            - Singleton
    2. 구조 패턴
        - 클래스를 조합해 더 큰 구조를 만드는 패턴
            - 종류
                - Adaptor
                - Bridge
                - Composite
                - Decorator
                - facade
                - Flyweight
                - Proxy
    3. 행위 패턴
        - 알고리즘이나 책임의 분배에 관한 패턴
        - 종류
            - Chain of Responsibility
            - Command
            - Interprter
            - Iterator
            - Mediator
            - Memento
            - Observer
            - State
            - Strategy
            - Template
            - Visitor

 <br>

## 결론
1. **우리가 얼마나 편하게 db를 접근할 수 있는지**
    - JDBC API → Spring JDBC → SQLMapper(iBatis(旧)/MyBatis(新)) → ORM
2. **리팩토링 방법**
    - 메소드 추출
    - 중복 제거 → 라이브러리 생성  (안정성 효과도 얻을 수 있음, sql 설정 분리 가능)
        - JdbcTemplate.class로 sql 연결 부분 통합
        - DataAccessException.class로 exception부분 통합
        - object/캐스팅  혹은 제네릭 <T> 사용
        - 가변인자 사용
    - 의존성 제거
        - 익명 클래스 / 추상클래스
        - 콜백 인터페이스
