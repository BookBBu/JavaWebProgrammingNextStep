
# 7. DB를 활용해 데이터를 영구적으로 저장하기
작성일시: 2023년 5월 30일

## 📌7.1 회원 데이터를 DB에 저장하기

---

HTTP 웹서버가 가지고 있는 문제 중의 하나는 "사용자가 입력한 데이터가 서버를 재시작하면 사라진다"


데이터를 영구적으로 저장하고 조회할 필요가 있는데 이를 데이터베이스 서버를 도입해 해결

**JDBC**

- JDBC라는 표준을 통해 데이터베이스와 통신을 담당하도록 지원
- JDBC 소스코드를 보면 구현 코드는 거의 없고 인터페이스만 정의
- 즉, 데이터베이스 통신을 위한 규약만 제공하고 이에 대한 구현체는 데이터베이스를 만들어 서비스하는 회사가 제공
- 이와 같이 표준만 정의함으로써 데이터베이스에 대한 연결 설정만 변경해 다른 데이터베이스를 지원하여 소스코드의 변경 최소화
 
회원데이터를 데이터베이스 서버에서 관리하고 JDBC API를 통해 접근하도록 구현 
JDBC API를 이용하는 과정에서 
중복 코드 발생 이를 제거 함으로써 여러프로젝트에서 공통으로 사용할 수 있는 JDBC 공통 라이브러리를 구현해보자




### 7.1.1 실습 코드 리뷰 및 JDBC 복습

---

톰켓 서버가 시작할 때 데이터베이스에 대한 초기화 작업을 위한 코드

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

ContextLoaderListener가 servleContextListener 인터페이스를 구현하고 있고 @WebListener 어노테이션이 설정되어 있으면 서블릿 컨테이너를 시작하는 과정에서 contextInitialized()메소드를 호출해 초기화 작업을 진행
서블릿의 초기화가 해당 서블릿과 관련한 초기화를 담당한다면 SevleContextListener초기화는 웹 애플리케이션 전체에 영향을 미치는 초기화가 필요한 경우 활용


---

### **7.1.2 회원 목록 실습**
```java
   public List<User> findAll() throws SQLException {
        // TODO 구현 필요함.
    	Connection con = null;
    	PreparedStatement pstmt = null;
    	ResultSet rs = null;
    	List<User> list = new ArrayList<>();
    	try {
    		con = ConnectionManager.getConnection();
    		StringBuilder sql = new StringBuilder("SELECT userId,password,name,email FROM USERS");
    		pstmt = con.prepareStatement(sql.toString());
    		rs = pstmt.executeQuery();
    		while(rs.next()) {
    			User user =  new User(rs.getString("userId"),
    					rs.getString("password"),
    					rs.getString("name"),
    					rs.getString("email"));
    			list.add(user);
    			
        }
    	} catch(Exception e) {
    		e.printStackTrace();
    	}
    	finally {
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
    	
        return list;
    }

```
---

### **7.1.3 개인정보 수정 실습**
```java
public void update(User user) throws SQLException {
        // TODO 구현 필요함.
    	Connection con = null;
    	PreparedStatement pstmt = null;
    	
    	try {
    		con = ConnectionManager.getConnection();
            String sql = "UPDATE USERS SET password=?, name =? ,email=? WHERE userId =?";
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, user.getPassword());
            pstmt.setString(2, user.getName());
            pstmt.setString(3, user.getEmail());
            pstmt.setString(4, user.getUserId());
            pstmt.executeUpdate();

    		
    	} catch(Exception e) {
    		e.printStackTrace();
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

---

## 📌7.2 DAO 리팩토링 실습

---
UserDao 코드 분석
- JDBC를 사용하는 UserDao는 많은 중복 코드 존재
- 쿼리 하나를 실행하기 위해서 개발자가 구현해야 할 코드가 많음
- 구현해야 할 코드가 대부분 매번 반복되는 부분
- 반복적인 부분을 공통 라이브러리를 만들어 제거 가능

1. 변화가 발생하는 부분(개발자가 구현할 수 밖에 없는 부분)과 변화가 없는 부분(공통 라이브러리로 분리할 부분)으로 분리 

|작업|공통 라이브러리|개발자가 구현할 부분|
|------|:---:|:---:|
|Connection 관리|O|X|
|SQL|X|O|
|Statement 관리|O|X|
|ResultSet관리|O|X|
|Row 데이터 추출|X|O|
|파라미터 선언|X|O|
|파라미터 세팅|O|X|
|트랜잭션 관리|O|X|

쿼리마다 개발자가 구현할 부분은 SQL 쿼리, 쿼리에 전달할 인자, 조회한 데이터를 추출하는 부분 3가지
나머지 작업은 공통 라이브러리로 위임 가능

2. SQLException에 대한 처리
- SQLException은 Complie Exception이기 때문에 매번 try/catch절을 통해 Exception 처리를 해야한다.
- 개발자 입장에서는 catch를 한다고 해서 에러 로그를 남기는 것외에 별달리 다른 작업을 한 부분이 생각나지 않는다.
- 컴파일 Exception으로 설계되지 않아도 되는 부분을 컴파일 Exception 으로 사용함으로써 불필요하게 try/catch절로 감싸야 하며 소스코드의 가독성 저하를 가져옴.



**Runtime Exception**
- 컴파일 된 후 실행시에 발생하는 예외(데이터 유효성 검증 등)
- 예외 처리(try catch 혹은 throw exception) 혹은 값 변경 중 선택하여 해결

**Compile Exception**
- 문제가 생기면 컴파일 자체가 불가, 컴파일 과정에서 발생하는 예외(문법적 오류)
- 예외 처리(try catch 혹은 throw exception) 필수

**컴파일 Exception과 Runtime 을 사용해야 되는 가이드 라인**
- API를 사용하는 모든 곳에서 이 예외를 처리해야 하는가? 예 => Compile Exception 구현 
- 예외가 반드시 메소드에 대한 반환 값이 되어야 하는가? 예 => Compile Exception 구현 (컴파일러의 도움)
- API를 사용하는 소수 중 이 예외를 처리해야 하는가? 예 => Runtime Exception 구현 (Exception을 Catch하도록 강제하지 않음)
- 큰 문제 발생 했는가? 문제 복구방법이 없는가? 예 => Runtime Exception 구현 (에러에 대한 정보를 통보받는것 외에 할수 있는것이 없음)
- 불명확한가? Runtime Exception으로 구현(문서화 하고 API를 사용하는 곳에서 예외처리 결정)

### **7.2.1 요구사항**

**JDBC에 대한 공통 라이브러리 만들기**

- 개발자가 SQL 쿼리, 쿼리에 전달할 인자, 조회한 데이터를 추출하는 3가지 구현에 집중하도록 해야 함.


## 📌7.4 DAO 리팩토링


### **7.4.1 메소드 분리**

메소드가 한 가지 작업만 처리하도록 작은 단위로 분리하다보면 중복 코드가 명확하게 들어남
비슷한 구조로 되어있는 insert() update() 메소드 분리
코드는 다음과 같다.
```java

public class UserDao {
    public void insert(User user) throws SQLException {
        Connection con = null;
        PreparedStatement pstmt = null;
        try {
            con = ConnectionManager.getConnection();
            String sql = createQueryForInsert()
            pstmt = con.prepareStatement(sql);
            setValuesForInsert(user, pstmt);
           
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
    private void setValuesForInsert(User user,PreparedStatement pstmt) throws SQLException{
    	pstmt.setString(1, user.getUserId());
        pstmt.setString(2, user.getPassword());
        pstmt.setString(3, user.getName());
        pstmt.setString(4, user.getEmail());
    }
    public String createQueryForInsert() {
    	return "INSERT INTO USERS VALUES (?, ?, ?, ?)";
    }

    public void update(User user) throws SQLException {
        // TODO 구현 필요함.
    	Connection con = null;
    	PreparedStatement pstmt = null;
    	
    	try {
    		con = ConnectionManager.getConnection();
            String sql = createQueryForUpdate();
            pstmt = con.prepareStatement(sql);
            setValuesForUpdate(user, pstmt);
            pstmt.executeUpdate();

    		
    	} catch(Exception e) {
    		e.printStackTrace();
    	} finally {
            if (pstmt != null) {
                pstmt.close();
            }

            if (con != null) {
                con.close();
            }
        }
    }
    
    public String createQueryForUpdate() {
    	return "UPDATE USERS SET password=?, name =? ,email=? WHERE userId =?";
    }
    
    private void setValuesForUpdate(User user,PreparedStatement pstmt) throws SQLException {
        pstmt.setString(1, user.getPassword());
        pstmt.setString(2, user.getName());
        pstmt.setString(3, user.getEmail());
        pstmt.setString(4, user.getUserId());
    }


```

### **7.4.2 클래스 분리**

- 공통 라이브러리로 구현할 부분(insert(),update())과 개발자가 매번 구현해야 할 부분(createQueryFor...(),setValuesFor...() 로 명확히 나눠짐.
- InsertJdbcTemplate 클래스를 추가한 후 insert() 메소드르 InsertJdbc Template로 이동
- InsertJdbcTemplate으로 이동하는 경우 createQuery, setValues() 메소드가 없어 컴파일 에러 발생 => UserDao인스턴스를 인자로 전달
- update() 도 똑같이 구현

```java


public class InsertJdbcTemplate {
	public void insert(User user, UserDao userDao) throws SQLException{
		Connection con = null;
        PreparedStatement pstmt = null;
        try {
            con = ConnectionManager.getConnection();
            String sql = userDao.createQueryForInsert();
            pstmt = con.prepareStatement(sql);
            userDao.setValuesForInsert(user, pstmt);
           
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
public class UserDao {
  public void insert(User user) throws SQLException {
        InsertJdbcTemplate jdbcTemplate = new InsertJdbcTemplate();
        jdbcTemplate.insert(user, this);
    }
    void setValuesForInsert(User user,PreparedStatement pstmt) throws SQLException{
    	pstmt.setString(1, user.getUserId());
        pstmt.setString(2, user.getPassword());
        pstmt.setString(3, user.getName());
        pstmt.setString(4, user.getEmail());
    }
    public String createQueryForInsert() {
    	return "INSERT INTO USERS VALUES (?, ?, ?, ?)";
    }
}

```
### **7.4.3 UserDao와 InsertJdbcTemplate의 의존관계 분리**

- InsertJdbcTemplate은 UserDao와 의존관계를 가지고 있어 UserDao가 아닌 다른곳에서 사용할 수 없다.
- 의존 관계를 가지지 않으려면 createQueryForInsert(), setValuesForInsert()메소드가 InsertJdbcTemplate에 존재 해야함
- 메소드의 구현은 UserDao가 담당 해야하기 때문에 추상메소드로 구현

- update() 도 똑같이 구현

```java
public  abstract class InsertJdbcTemplate {
	public void insert(User user) throws SQLException{
		Connection con = null;
        PreparedStatement pstmt = null;
        try {
            con = ConnectionManager.getConnection();
            String sql = createQueryForInsert();
            pstmt = con.prepareStatement(sql);
            setValuesForInsert(user, pstmt);
           
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
	abstract void setValuesForInsert(User user, PreparedStatement pstmt) throws SQLException;
	abstract String createQueryForInsert();

}
```
- UserDao는 InsertJdbcTemplate 인스턴스를 생성 해야 함
- 추상 클래스이기 때문에 바로 생성 X 2개의 추상 메소드를 구현
- InsertJdbcTemplate을 상솟하는 새로운 클래스를 생성 or 익명클래스 사용
- 여기서는 익명클래스를 추가하는 방법 사용 (다른 곳에서 재사용할 필요가 없는 경우 사용 적합)

```java
public class UserDao {
    public void insert(User user) throws SQLException {
        InsertJdbcTemplate jdbcTemplate = new InsertJdbcTemplate() {

			@Override
			void setValuesForInsert(User user, PreparedStatement pstmt) throws SQLException {
				// TODO Auto-generated method stub
				pstmt.setString(1, user.getUserId());
		        pstmt.setString(2, user.getPassword());
		        pstmt.setString(3, user.getName());
		        pstmt.setString(4, user.getEmail());
			}

			@Override
			String createQueryForInsert() {
				// TODO Auto-generated method stub
		    	return "INSERT INTO USERS VALUES (?, ?, ?, ?)";

			}
        };
        jdbcTemplate.insert(user);
    }
}
```
- 이와 같이 반복적으로 발생하는 중복 로직을 상위 클래스가 구현 변화가 발생하는 부분만 추상 메소드로 만들어 구현 하는 디자인 패턴을 템플릿 메소드 패턴이라함

### **7.4.4 InsertJdbcTemplate 와 UpdateJdbcTemplate 통합**
- 공통 라이브러리를 담당할 클래스를 분리하니 굳이 ForInsert, ForUpdate와 같이 붙일 필요 X  => Rename리팩토링
- Insert,Update 구현부가 똑같음 => 클래스 합치기
```java
public abstract class JdbcTemplate {
	public void update(User user) throws SQLException{
		Connection con = null;
    	PreparedStatement pstmt = null;
    	
    	try {
    		con = ConnectionManager.getConnection();
            String sql = createQuery();
            pstmt = con.prepareStatement(sql);
            setValues(user, pstmt);
            pstmt.executeUpdate();

    		
    	} catch(Exception e) {
    		e.printStackTrace();
    	} finally {
            if (pstmt != null) {
                pstmt.close();
            }

            if (con != null) {
                con.close();
            }
        }
	}
    abstract String createQuery();
    
    
    abstract void setValues(User user,PreparedStatement pstmt) throws SQLException ;

}
```
- UserDao 수정 생략

### **7.4.5 User의존 관계 제거 및 SQL 쿼리 인자로 전달**
- JDBCTempalte을 다른 곳에서 사용하려면 User에 대한 의존관계도 끊어야 함.
- JdbcTemplate의 update()메소드를 살펴보면 굳이 User를 인자로 전달하지 않고 User인스턴스에 직접 접근하도록 리팩토링

```java
public abstract class JdbcTemplate {
	public void update() throws SQLException{
		Connection con = null;
    	PreparedStatement pstmt = null;
    	
    	try {
    		con = ConnectionManager.getConnection();
            String sql = createQuery();
            pstmt = con.prepareStatement(sql);
            setValues(pstmt);
            pstmt.executeUpdate();

    		
    	} catch(Exception e) {
    		e.printStackTrace();
    	} finally {
            if (pstmt != null) {
                pstmt.close();
            }

            if (con != null) {
                con.close();
            }
        }
	}
    abstract String createQuery();
    
    
    abstract void setValues(PreparedStatement pstmt) throws SQLException ;

}

public class UserDao {
    public void insert(User user) throws SQLException {
        JdbcTemplate jdbcTemplate = new JdbcTemplate() {

			@Override
			void setValues(PreparedStatement pstmt) throws SQLException {
				// TODO Auto-generated method stub
				pstmt.setString(1, user.getUserId());
		        pstmt.setString(2, user.getPassword());
		        pstmt.setString(3, user.getName());
		        pstmt.setString(4, user.getEmail());
			}

			@Override
			String createQuery() {
				// TODO Auto-generated method stub
		    	return "INSERT INTO USERS VALUES (?, ?, ?, ?)";

			}
        };
        jdbcTemplate.update();
    }
}
```

- 매번 2개의 추상 메소드를 구현할 필요가 있을까?
- setValues는 값을 전달하는 부분이 많아 어쩔 수 없이 구현해야 함
- SQL 쿼리는 메소드 인자로 전달하는 것이 사용성 측면에서 좋음
- 리팩토링 과정에서 컴파일 에러가 발생하지 않기 위해 update()메소드를 유지한 상태에서  update2(String sql ) 추가
- 문제 없이 돌아가는 것을 확인후 update() 삭제 후 update2(String sql) update로 rename
- 이렇게 리팩토링을 진행하는 이유는 점진적으로 진행하며 컴파일 에러 최소화
완성 코드
```java
public abstract class JdbcTemplate {
	
    
    public void update(String sql) throws SQLException{
		Connection con = null;
    	PreparedStatement pstmt = null;
    	
    	try {
    		con = ConnectionManager.getConnection();
            pstmt = con.prepareStatement(sql);
            setValues(pstmt);
            pstmt.executeUpdate();

    		
    	} catch(Exception e) {
    		e.printStackTrace();
    	} finally {
            if (pstmt != null) {
                pstmt.close();
            }

            if (con != null) {
                con.close();
            }
        }
	}
    abstract void setValues(PreparedStatement pstmt) throws SQLException ;

}
public class UserDao {
    public void insert(User user) throws SQLException {
        JdbcTemplate jdbcTemplate = new JdbcTemplate() {

			@Override
			void setValues(PreparedStatement pstmt) throws SQLException {
				// TODO Auto-generated method stub
				pstmt.setString(1, user.getUserId());
		        pstmt.setString(2, user.getPassword());
		        pstmt.setString(3, user.getName());
		        pstmt.setString(4, user.getEmail());
			}

        };
        String sql = "INSERT INTO USERS VALUES (?, ?, ?, ?)";
        jdbcTemplate.update(sql);
    }
}
public class UserDao {
  public List<User> findAll() throws SQLException {
		SelectJdbcTemplate jdbcTemplate = new SelectJdbcTemplate() {

			@Override
			void setValues(PreparedStatement pstmt) throws SQLException {
				// TODO Auto-generated method stub

			}

			@Override
			Object mapRow(ResultSet rs) throws SQLException {
				// TODO Auto-generated method stub
				return new User(rs.getString("userId"), rs.getString("password"), rs.getString("name"),
						rs.getString("email"));
			}
		};
		String sql = "SELECT userId, password,name, email FROM USERS";
		return (List<User>) jdbcTemplate.query(sql);
	}

	public User findByUserId(String userId) throws SQLException {
		SelectJdbcTemplate jdbcTemplate = new SelectJdbcTemplate() {

			@Override
			void setValues(PreparedStatement pstmt) throws SQLException {
				// TODO Auto-generated method stub
				pstmt.setString(1, userId);

			}

			@Override
			Object mapRow(ResultSet rs) throws SQLException {
				// TODO Auto-generated method stub
				return new User(rs.getString("userId"), rs.getString("password"), rs.getString("name"),
						rs.getString("email"));
			}
		};
		String sql = "SELECT userId, password, name, email FROM USERS WHERE userid=?";
		return (User) jdbcTemplate.queryForObject(sql);

	}

}
```
### **7.4.6 SELECT문에 대한 리팩토링 **
- SELECT에 대해서도 앞의 과정과 같은 방식으로 리팩토링 진행
- 조회한 데이터를 자바 객체로 변환하는 부분 추가 
- mapRow()라는 메소드를 추상메소드로 추가해 구현
- 한 건의 데이터를 조회하는 경우는 query()메소드를 이용해 
```java
public abstract class SelectJdbcTemplate {
	@SuppressWarnings("rawtypes")
	public List query(String sql) throws SQLException{
		Connection con = null;
		PreparedStatement pstmt = null;
		ResultSet rs =null;
		try {
			con = ConnectionManager.getConnection();
			pstmt = con.prepareStatement(sql);
			setValues(pstmt);
			rs = pstmt.executeQuery();
			List<Object> result = new ArrayList<Object>();
			while(rs.next()) {
				result.add(mapRow(rs));
			}
			return result;
			
		} finally {
			if(rs!=null) {
				rs.close();
			}
			if(pstmt != null) {
				pstmt.close();
			}
			if(con !=null) {
				con.close();
			}
		}
	}
	
	@SuppressWarnings("rawtypes")
	public Object queryForObject(String sql) throws SQLException{
		List result = query(sql);
		if(result.isEmpty()) {
			return null;
		}
		return result.get(0);
	}
	
	abstract void setValues(PreparedStatement pstmt) throws SQLException;
	abstract Object mapRow(ResultSet rs) throws SQLException;

}
```

### **7.4.7 JdbcTemplate과 SelectJdbcTemplate 통합 **
- 두 클래스의 구현 부분을 보면 중복 코드가 많아 하나로 통합하는 것이 좋음


```java
ublic abstract class JdbcTemplate {

	public void update(String sql) throws SQLException {
		Connection con = null;
		PreparedStatement pstmt = null;

		try {
			con = ConnectionManager.getConnection();
			pstmt = con.prepareStatement(sql);
			setValues(pstmt);
			pstmt.executeUpdate();

		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (pstmt != null) {
				pstmt.close();
			}

			if (con != null) {
				con.close();
			}
		}
	}
	@SuppressWarnings("rawtypes")
	public List query(String sql) throws SQLException{
		Connection con = null;
		PreparedStatement pstmt = null;
		ResultSet rs =null;
		try {
			con = ConnectionManager.getConnection();
			pstmt = con.prepareStatement(sql);
			setValues(pstmt);
			rs = pstmt.executeQuery();
			List<Object> result = new ArrayList<Object>();
			while(rs.next()) {
				result.add(mapRow(rs));
			}
			return result;
			
		} finally {
			if(rs!=null) {
				rs.close();
			}
			if(pstmt != null) {
				pstmt.close();
			}
			if(con !=null) {
				con.close();
			}
		}
	}
	
	@SuppressWarnings("rawtypes")
	public Object queryForObject(String sql) throws SQLException{
		List result = query(sql);
		if(result.isEmpty()) {
			return null;
		}
		return result.get(0);
	}
	
	abstract void setValues(PreparedStatement pstmt) throws SQLException;
	abstract Object mapRow(ResultSet rs) throws SQLException;



}

public class UserDao {
	public void insert(User user) throws SQLException {
		JdbcTemplate jdbcTemplate = new JdbcTemplate() {

			@Override
			void setValues(PreparedStatement pstmt) throws SQLException {
				// TODO Auto-generated method stub
				pstmt.setString(1, user.getUserId());
				pstmt.setString(2, user.getPassword());
				pstmt.setString(3, user.getName());
				pstmt.setString(4, user.getEmail());
			}

			@Override
			Object mapRow(ResultSet rs) throws SQLException {
				// TODO Auto-generated method stub
				return null;
			}

		};
		String sql = "INSERT INTO USERS VALUES (?, ?, ?, ?)";
		jdbcTemplate.update(sql);
	}
}

```

- 통합할 경우 UserDao 의 insert(),update() 메소드를 maprow()구현해야 하는 수정사항 발생

### **7.4.8 인터페이스 추가를 통한 문제점 해결 **
- Jdbc Template의 추상 메소드 변화 시점이 다를 수 있는데 항상 같이 변화하도록 의존 관계가 생김
- setValues()와 mapRow() 메소드를 분리해 서로  간의 의존 관계를 끊어버리면 좀더 유연한 개발이 가능
- 두 개의 추상 메소드를 같은 클래스가 가지도록 구현하지 말고 각각의 추상 메소드를 인터페이스를 통해 분리

```java

public interface PreparedStatementSetter {
	void setValues(PreparedStatement pstmt) throws SQLException;

}
public interface RowMapper {
	Object mapRow(ResultSet rs) throws SQLException;

}

public  class JdbcTemplate {

	public void update(String sql,PreparedStatementSetter pss) throws SQLException {
		Connection con = null;
		PreparedStatement pstmt = null;

		try {
			con = ConnectionManager.getConnection();
			pstmt = con.prepareStatement(sql);
			pss.setValues(pstmt);
			pstmt.executeUpdate();

		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (pstmt != null) {
				pstmt.close();
			}

			if (con != null) {
				con.close();
			}
		}
	}
	@SuppressWarnings("rawtypes")
	public List query(String sql,PreparedStatementSetter pss, RowMapper rowMapper) throws SQLException{
		Connection con = null;
		PreparedStatement pstmt = null;
		ResultSet rs =null;
		try {
			con = ConnectionManager.getConnection();
			pstmt = con.prepareStatement(sql);
			pss.setValues(pstmt);
			rs = pstmt.executeQuery();
			List<Object> result = new ArrayList<Object>();
			while(rs.next()) {
				result.add(rowMapper.mapRow(rs));
			}
			return result;
			
		} finally {
			if(rs!=null) {
				rs.close();
			}
			if(pstmt != null) {
				pstmt.close();
			}
			if(con !=null) {
				con.close();
			}
		}
	}
	
public class UserDao {
	public void insert(User user) throws SQLException {
		JdbcTemplate jdbcTemplate = new JdbcTemplate();
		PreparedStatementSetter pss = new PreparedStatementSetter() {

			@Override
			public void setValues(PreparedStatement pstmt) throws SQLException {
				// TODO Auto-generated method stub
				pstmt.setString(1, user.getUserId());
				pstmt.setString(2, user.getPassword());
				pstmt.setString(3, user.getName());
				pstmt.setString(4, user.getEmail());

			}
		};
		String sql = "INSERT INTO USERS VALUES (?, ?, ?, ?)";
		jdbcTemplate.update(sql, pss);
	}

	void update(User user) throws SQLException {
		JdbcTemplate jdbcTemplate = new JdbcTemplate();
		PreparedStatementSetter pss = new PreparedStatementSetter() {

			@Override
			public void setValues(PreparedStatement pstmt) throws SQLException {
				// TODO Auto-generated method stub
				pstmt.setString(1, user.getPassword());
				pstmt.setString(2, user.getName());
				pstmt.setString(3, user.getEmail());
				pstmt.setString(4, user.getUserId());
			}
		};
		String sql = "UPDATE USERS SET password=?, name =? ,email=? WHERE userId =?";
		jdbcTemplate.update(sql, pss);
	}

	public List<User> findAll() throws SQLException {
		JdbcTemplate jdbcTemplate = new JdbcTemplate();
		PreparedStatementSetter pss = new PreparedStatementSetter() {

			@Override
			public void setValues(PreparedStatement pstmt) throws SQLException {
				// TODO Auto-generated method stub

			}
		};
		RowMapper rowMapper = new RowMapper() {

			@Override
			public Object mapRow(ResultSet rs) throws SQLException {
				// TODO Auto-generated method stub
				return new User(rs.getString("userId"), rs.getString("password"), rs.getString("name"),
						rs.getString("email"));
			}
		};
		String sql = "SELECT userId, password,name, email FROM USERS";
		return (List<User>) jdbcTemplate.query(sql, pss, rowMapper);
	}

	public User findByUserId(String userId) throws SQLException {
		JdbcTemplate jdbcTemplate = new JdbcTemplate();
		PreparedStatementSetter pss = new PreparedStatementSetter() {

			@Override
			public void setValues(PreparedStatement pstmt) throws SQLException {
				// TODO Auto-generated method stub
				pstmt.setString(1, userId);

			}
		};
		RowMapper rowMapper = new RowMapper() {

			@Override
			public Object mapRow(ResultSet rs) throws SQLException {
				// TODO Auto-generated method stub
				return new User(rs.getString("userId"), rs.getString("password"), rs.getString("name"),
						rs.getString("email"));
			}
		};

		String sql = "SELECT userId, password, name, email FROM USERS WHERE userid=?";
		return (User) jdbcTemplate.queryForObject(sql, pss, rowMapper);

	}

}

```
- 이와 같이 메소드 하나만 가지는 인터페이스를 생성한 후 필요에 따라 메소드의 인자로 전달하여 문제점 해결
- 변화 시점이 다른 부분을 서로 다른 인터페이스로 분리함으로써 공통 라이브러리에 대한 유연함을 높임
- 이 예제에서 사용한 인터페이스를 콜백(Callback) 인터페이스라고 부른다.


### **7.4.9  Runtime Exception 추가 및 AutoClosable 활용한 자원 반환 **
- UserDao의 문제점 중 하나는 모든 메소드가 컴파일 타임 Exception인 SQLException 을 Throw
- runtime Exception 추가

먼저 RuntimeException을 상속 하는 새로운 Exception 추가

```java
public class DataAccessException extends RuntimeException {

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
- JdbcTemplate을 리팩토링하여 JdbcTemplate을 사용하는 곳에서 더 이상 SQLException을 처리하지 않게 되었다.

```java
public  class JdbcTemplate {

	public void update(String sql,PreparedStatementSetter pss) throws DataAccessException {
		Connection con = null;
		PreparedStatement pstmt = null;

		try {
			con = ConnectionManager.getConnection();
			pstmt = con.prepareStatement(sql);
			pss.setValues(pstmt);
			pstmt.executeUpdate();

		} catch (SQLException e) {
			throw new DataAccessException(e);
		} finally {
			if (pstmt != null) {
				try {
					pstmt.close();					
				} catch(SQLException e) {
					throw new DataAccessException(e);
				}
			}

			if (con != null) {
				try {
					con.close();
				} catch(SQLException e) {
					throw new DataAccessException(e);
				}
			}
		}
	}
}

```
- finally{} 절의 복잡도 증가 
```java
public  class JdbcTemplate {

	public void update(String sql,PreparedStatementSetter pss) throws DataAccessException {
		try (Connection con = ConnectionManager.getConnection();
				PreparedStatement 	pstmt = con.prepareStatement(sql)){
		
			pss.setValues(pstmt);
			pstmt.executeUpdate();

		} catch (SQLException e) {
			throw new DataAccessException(e);
		}
	}

```
- try-with-resources 구문을 이용하여 자원을 자동으로 반납
- java.io.AutoClosable 인터페이스를 구현하고 있어야 사용 가능
[java docs](https://docs.oracle.com/javase/8/docs/api/)

### **7.4.10  제너릭(generic)을 활용한 개선 **
- jdbcTemplate 사용시 매번 캐스팅을 해야한다는 점이 불편
- 제너릭을 사용하여 캐스팅을 하지 않도록 개선

```java
public interface RowMapper<T>{
  T mapRow(ResultSet rs) throws SQLException;
}

public class JdbcTemplate {
  
    public <T> List<T> query(String sql, PreparedStatementSetter pss, RowMapper<T> rowMapper) throws DataAccessException {
        ResultSet rs = null;
        try (Connection conn = ConnectionManager.getConnection();
                PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pss.setParameters(pstmt);
            rs = pstmt.executeQuery();

            List<T> list = new ArrayList<T>();
            while (rs.next()) {
                list.add(rowMapper.mapRow(rs));
            }
            return list;
        } catch (SQLException e) {
            throw new DataAccessException(e);
        } finally {
            try {
                if (rs != null) {
                    rs.close();
                }
            } catch (SQLException e) {
                throw new DataAccessException(e);
            }
        }
    }
  
    public <T> List<T> query(String sql,  PreparedStatementSetter pss,RowMapper<T> rowMapper)  throws DataAccessException {
      List<T> result = query(sql,pss,rowMapper);
      if(result.isEmpty()){
        return null;
      }
      return result.get(0);
    }

}
public class UserDao{
  
  	public User findByUserId(String userId) throws SQLException {
		JdbcTemplate jdbcTemplate = new JdbcTemplate();
		PreparedStatementSetter pss = new PreparedStatementSetter() {

			@Override
			public void setValues(PreparedStatement pstmt) throws SQLException {
				// TODO Auto-generated method stub
				pstmt.setString(1, userId);

			}
		};
		RowMapper<User> rowMapper = new RowMapper<User>() {

			@Override
			public User mapRow(ResultSet rs) throws SQLException {
				// TODO Auto-generated method stub
				return new User(rs.getString("userId"), rs.getString("password"), rs.getString("name"),
						rs.getString("email"));
			}
		};

		String sql = "SELECT userId, password, name, email FROM USERS WHERE userid=?";
		return jdbcTemplate.queryForObject(sql, pss, rowMapper);

	}

}

```
- 제너릭을 사용하면 데이터를 조회할 때 캐스팅이 하지 않아도 된다.


### **7.4.11  가변인자를 활용해 쿼리에 인자 전달하기  **
- PreparedStatementSetter를 활용가능하지만 인자로 전달할 값이 갯수가 고정되지 않고 동적으로 변경되는 경우 가변 인자를 활용가능


```java
public class JdbcTemplate {
  
  public void update(String sql,PreparedStatementSetter pss) throws DataAccessException {
      Connection con = null;
      PreparedStatement pstmt = null;

      try {
        con = ConnectionManager.getConnection();
        pstmt = con.prepareStatement(sql);
        pss.setValues(pstmt);
        pstmt.executeUpdate();

      } catch (SQLException e) {
        throw new DataAccessException(e);
      }
    }
    public void update(String sql,Object... parameters) throws DataAccessException {

      try (Connection con = ConnectionManager.getConnection();
        PreparedStatement pstmt = con.prepareStatement(sql)) {
          for(int i=0;i < parameters.length;i++) {
            pstmt.setObject(i+1, parameters[i]);
          }
          pstmt.executeUpdate();
      } catch (SQLException e) {
        throw new DataAccessException(e);
      }
    }
}



public class UserDao{
  

  public void insert(User user) throws SQLException {
      JdbcTemplate jdbcTemplate = new JdbcTemplate();
      String sql = "INSERT INTO USERS VALUES (?, ?, ?, ?)";
      jdbcTemplate.update(sql, user.getUserId(),user.getPassword(),user.getName(),user.getEmail());
    }

}
```

### **7.4.12  람다를 활용한 구현  **
- UserDao에서 RowMapper에 대한 익명 클래스를 생성하던 부분을 람다를 활용하여 깔끔하게 구현 가능

```java
public class UserDao {
 public User findByUserId(String userId) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        String sql = "SELECT userId, password, name, email FROM USERS WHERE userid=?";

        return jdbcTemplate.queryForObject(sql, (ResultSet rs) -> {
          return new User(rs.getString("userId"),rs.getString("password"),rs.getString("name"),rs.getString("email")}, userId);
    } 
}

@FunctionalInterface
public interface RowMapper<T> {
	T mapRow(ResultSet rs) throws SQLException;

}

```

- 람다를 사용하기 전에는 익명 클래스를 만들어 전달 했지만 람다를 사용할 경우 메소드에 전달할 인자와 메소드의 구현부만 전달하는 것이 가능
- 람다를 사용하려면 RowMapper와 같이 인터페이스의 메소드가 하나만 존재
- 람다 표현식으로 사용할 인터페이스라고 지정하기 위히ㅐ @FunctionInterface 어노테이션 추가


## 📌7.5 추가 학습 자료

### **7.5.1 데이터베이스**



1. **SQL 첫걸음: 하루 30분 36강으로 배우는 완전 초보의 SQL 따라 잡기(아사이 아츠시 저/ 박준용 역, 한빛미디어/2015)**
   - ER다이어그램, 인덱스, 정규화, 트랜잭션


2. **Real MySQL : 개발자와 DBA를 위한(이성욱 ,위키북스/2012)"**
   - MySQL에 대한 깊이 있는 학습  
3. **NOSQL: 빅데이터 세상으로 떠나는 간결한 안내서(프라모드 사달게이, 마틴 파울러 공저, 윤성준 역, 인사이트/2013)**
  - 다양한 NoSQL 에 대한 특징을 알아보기 

[데이터베이스 명명규칙](https://bestinu.tistory.com/62)



### **7.5.2 디자인 패턴**
- 디자인 패턴은 개발 경험을 쌓은 후 학습해도 괜찮다고 생각함.
- 먼저 리팩토링할 부분을 찾아 작은 부분이라도 리팩토링을 통해 소스 코드를 개선하는 경험이 우선

1. **GoF의 디자인 패턴: 재사용성을 지닌 객체 지향 소프트웨어의 핵심요소(에릭 감마 저, 김정아 역,프로텍미디어/2015)"**
  - 개발경험이 많은 4명의 개발자가 개발 과정에서 자주 나타내는 패턴을 정리한 책 : 정말 어려움

2. **Head First Design Patterns: 스토리가 있는 패턴 학습법(에릭 프리먼 저, 서환수 역,한빛미디어/2005)"**
  - 애플리케이션 개발에서 자주 사용하는 디자인 패턴 위주로 설명하고 있으며, 다양한 그림을 통해 쉽게 설명 
3. **실전 코드로 배우는 실용주의 디자인 패턴 (Allen Holub 저, 송치형 역 ,지앤선/2006)"**
  - 소스코드를 통해 디자인 패턴을 학습 


