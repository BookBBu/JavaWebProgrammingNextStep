# [7장] DB를 활용해 데이터를 영구적으로 저장하기

HTTP 웹 서버가 가지고 있었던 문제

- 사용자가 입력한 데이터가 서버를 재시작하면 사라진다는 것
- 해결책 : 데이터베이스 서버 도입하여 데이터를 영구적으로 저장하고 조회 가능

## **📌 7장 목표**
--- 
지금까지 구현한 회원 데이터를 데이터베이스 서버에서 관리하고 JDBC API를 통해 접근하도록 구현

## 7.1) 회원 데이터를 DB에 저장하기

---

이 책은 경량 데이터베이스 중의 하나인 H2 데이터베이스를 사용하고 있다

### H2 데이터베이스

- 자바 기반의 오픈소스 관계형 데이터베이스 관리시스템(RDBMS)
- 따로 설치가 필요없고 용량이 매우 가볍다
- 스트링 부트가 지원하는 인메모리 관계형 데이터베이스(주 메모리에 저장)
- 어플리케이션 개발 단계의 테스트 DB로서 많이 사용된다
- Http://www.h2database.com/

### 7.1.1 실습 코드 리뷰 및 JDBC 복습

---

1. ContextLoaderListener 클래스

- 톰캣 서버가 시작하는 시점에 회원 정보를 저장할 테이블을 초기화하도록 구현

```java
@WebListener
public class ContextLoaderListener implements ServletContextListener {
    private static final Logger logger = LoggerFactory.getLogger(ContextLoaderListener.class);

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

- 톰캣 서버가 시작할 때 contextInitialized() 메소드를 호출함으로써 초기화 작업을 할 수 있다
- ContextLoaderListener가 ServletContextListener 인터페이스를 구현하고 있으며 @WebListener 애노테이션 설정이 있기 때문에, 서블릿 컨테이너를 시작하는 과정에서 contextInitialized() 메소드를 호출해 초기화 작업을 진행할 수 있다

2. UserDao 클래스

- 데이터베이스 접근 로직을 구현

```java
public class UserDao {
    public void insert(User user) throws SQLException {
        Connection con = null;
        PreparedStatement pstmt = null;
        try {
            con = ConnectionManager.getConnection();
            String sql = "INSERT INTO USERS VALUES (?, ?, ?, ?)";
            pstmt = con.prepareStatement(sql);
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

- 자바 진영은 데이터베이스에 대한 접근 로직 처리를 담당하는 객체를 별도로 분리해 구현하는 것을 추천, 이 객체를 DAO(Data Access Object)라고 부른다

3. CreateUserController 클래스

- 회원가입을 담당하고 있는 CreateUserController가 UserDao를 사용하도록 변경

```java
public class CreateUserController implements Controller {
    private static final Logger log = LoggerFactory.getLogger(CreateUserController.class);

    @Override
    public String execute(HttpServletRequest req, HttpServletResponse resp) throws Exception {
        User user = new User(req.getParameter("userId"), req.getParameter("password"), req.getParameter("name"),
                req.getParameter("email"));
        log.debug("User : {}", user);


        UserDao userDao = new UserDao();
        try {
        	userDao.insert(user);
        }catch(SQLException e) {
        	log.error(e.getMessage());
        }
        return "redirect:/";
    }
}

```

- 단, UserDao를 사용할 때 불편한 점 중의 하나는 UserDao의 모든 메소드가 SqlException을 throw하고 있기 때문에 컴파일 에러를 해결하기 위해 try/catch문으로 감싸줘야 한다는 것이다

### **7.1.2 회원 목록 실습**

---

```java

```

### **7.1.3 개인정보 수정 실습**

---

```java

```

## 7.2) DAO 리팩토링 실습

---

- JDBC를 사용하는 UserDao는 많은 중복 코드가 존재한다. 데이터베이스에 쿼리 하나를 실행하기 위해 개발자가 구현해야 할 코드가 너무 많고 대부분 매번 반복된다. 이와 같이 많은 중복이 있고, 반복적인 부분이 있는 코드는 공통 라이브러리를 만들어 제거할 수 있다.

### 📌 UserDao의 중복을 제거하는 작업을 통해 공통 라이브러리를 구현

1. **중복 코드를 리팩토링 하기 위해 UserDao 코드를 분리**
   - 변화가 발생하는 부분(개발자가 구현할 수 밖에 없는 부분) : SQL / Row 데이터 추출 / 패러미터 선언
   - 변화가 없는 부분(공통 라이브러리로 분리할 부분) : Connection 관리 / Statement 관리 / ResultSet 관리 / 패러미터 Setting / 트랜잭션 관리
2. **SQLExcetion 처리**
   - SQLExcetion은 컴파일 타임 Exception이기 때문에 매번 try/catch절을 통해 Exceprion 처리를 해야한다. 무부분별한 사용으로 불필요하게 try/catch절로 감싸게 되고, 코드의 가독성을 떨어뜨리게 된다.
   - 따라서, 'expert one-on-one J2EE 설계와 개발'에서 제공하는 가이드라인을 참고하여 JDBC의 SQLException을 런타임 Exception으로 변환하여 해결한다.

**컴파일/런타임 Exception 가이드라인**

- 컴파일타임 Exception
  - API를 사용하는 모든 곳에서 이 예외를 처리해야 하는가?
  - 예외가 반드시 메소드에 대한 반환 값이 되어야 하는가?
- 런타임 Exception
  - API를 사용하는 소수 중 이 예외를 처리해야 하는가?
  - 무엇인가 큰 문제가 발생했는가? 이 분제를 복구할 방법이 없는가?
  - 불명확한가?

### **7.2.1 요구사항**

---

- JDBC에 대한 공통 라이브러리를 만들어 개발자가 SQL 쿼리, 쿼리에 전달할 인자, SELECT 구문의 경우 조회한 데이터를 추출하는 3가지 구현에만 집중하도록 해야 함
- SQLException을 런타임 Exception으로 변환해 try/catch 절로 인해 소스코드의 가독성을 해치지 않도록 해야 함

### **7.2.2 요구사항 분리 및 힌트**

---

- 클래스 다이어그램을 활용해 UserDao 클래스의 각 단계별 리팩토링을 진행한다.
- 7.3의 동영상 참고

## 7.3) 동영상을 활용한 DAO 리팩토링 실습

---

1. INSERT, UPDATE, DELETE문에 대한 중복 제거 과정
   - http://youtu.be/ylrMBeakVnk
2. select 문 중복 제거
   - http://youtu.be/zfXAZkqPH44
3. JdbcTemplate과 SelectJdbcTemplate 통합
   - http://youtu.be/yEHUB97B62I
4. 라이브러리 리팩토링 및 목록 기능 추가
   - http://youtu.be/nkepkHJi7e8
5. SQLException을 DataAccessException으로 래핑
   - http://youtu.be/lFTyw7Uipyo
6. 람다 표현식을 사용하도록 리팩토링
   - http://youtu.be/0ax9jxfW9x4

## 7.4) DAO 리팩토링 및 설명

---

### **7.4.1 메소드 분리**

---

Extract Method 리팩토링을 통해 메소드 분리 작업

- 메소드가 한 가지 작업만 처리하도록 작은 단위로 분리
- 이 리팩토링에서는 위에서 분리한 두 가지 부분(변화가 발생하는 부분/변화가 발생하지 않는 부분)으로 나눈다.

```java
public class UserDao{
	public void insert(User user) throws SQLException{
		Connection con = null;
		PreparedStatemnet pstmt = null;
		try{
			con = ConnectionManager.getConnection();
			String sql = createQueryforInsert(); // 개발자가 매번 구현해야 할 부분
			pstmt = con.prepareStatement(sql);
			setValuesForInsert(user, pstmt); // 개발자가 매번 구현해야 할 부분

			pstmt.executeUpdate();
		}finally{
			if(pstmt != null){
				pstmt.close();
			}

			if(con != null){
				con.close();
			}
		}
	}
}
```

### **7.4.2 클래스 분리**

---

- UserDao 클래스는 라이브러리로 구현할 부분(insert() 메소드)와 개발자가 매번 구현해야 할 부분(createQueryForInsert(), setValuesForInsert() 메소드)로 나눠짐
- 따라서 insert() 메소드를 공통 라이브러리로 구현하기 위해 InsertJdbcTemplate 클래스를 추가한 후 UserDao의 insert() 메소드를 InsertJdbcTemplate로 이동한다.

```java
// InsertJdbcTemplate 클래스 : insert() 공통 라이브러리 구현
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
```

```java
// UserDao 클래스
public class UserDao{
	// UserDao의 insert()메소드는 InsertJdbcTemplate의 insert() 메소드를 호출
	public void insert(User user) throws SQLException{
		InsertJdbcTemplate jdbcTemplate = new InsertJdbcTemplate();
		jdbcTemplate.insert(user, this);
	}

	// InsertJdbcTemplate의 insert() 메소드가 접근 가능하도록 접근 제어자를 default로 변경
	void setValueForInsert(User user, PreparedStatement pstmt) throws SQLException{
		pstmt.setString(1, user.getUserId());
		pstmt.setString(2, user.getPassword());
		pstmt.setString(3, user.getName());
		pstmt.setString(4, user.getgetEmail());
	}

	String createQueryForInsert(){
		return "INSERT INTO USERS VALUES (?, ?, ?, ?, ?)";
	}
}

```

- insert() 메소드를 InsertJdbcTemplate으로 이동하는 경우 createQueryForInsert(), setValuesForInsert() 메소드가 없기 떄문에 컴팡리 에러가 발생한다. 이를 해결하기 위해 UserDao 인스턴스를 인자로 전달해 UserDao 메소드를 호출하도록 변경한다.

### **7.4.3 UserDao와 InsertJdbcTemplate의 의존관계 분리**

---

InsertJdbcTemplate와 UserDao는 의존관계를 가지고 있기 때문에 UserDao가 아닌 다른 곳에서는 사용할 수 없다.

- 따라서 의존관계를 가지지 않으려면 setValueForInsert(), createQueryForInsert() 두 메소드가 InsertJdbcTemplate 클래스 안에 존재해야 하는데, 메소드 구현은 UserDao가 담당해야 하기 때문에 두 개의 메소드를 추상 메소드로 구현한다.

```java
public abstract class InsertJdbcTemplate {
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

	// 두 메소드를 추상 메소드로 구현
	abstract void setValuesForInsert(User user, PreparedStatement pstmt) throws SQLException;
	abstract String createQueryForInsert();

}
```

- UserDao는 InsertJdbcTemplate 인스턴스를 생성해야 하는데 InsertJdbcTemplate가 추상 클래스이기 때문에 3가지 방법으로 해결할 수 있다.

  1.  2개의 추상 메서드 구현하기
  2.  InsertJdbcTemplate을 상속하는 새로운 클래스 추가하기
  3.  이름을 가지지 않는 익명 클래스 추가하기

- 3번의 익명클래스는 다른 곳에서 재사용할 필요가 없는 경우 사용하기 적합하므로 InsertJdbcTemplate 인스턴스를 생성할 때 적합하여 익명 클래스를 사용해 구현하여 해결한다.

```java
public class UserDao {
    public void insert(User user) throws SQLException {
		// 익명 클래스로 구현
        InsertJdbcTemplate jdbcTemplate = new InsertJdbcTemplate() {
			@Override
			void setValuesForInsert(User user, PreparedStatement pstmt) throws SQLException {
				pstmt.setString(1, user.getUserId());
		        pstmt.setString(2, user.getPassword());
		        pstmt.setString(3, user.getName());
		        pstmt.setString(4, user.getEmail());
			}

			@Override
			String createQueryForInsert() {
		    	return "INSERT INTO USERS VALUES (?, ?, ?, ?)";
			}
        };
        jdbcTemplate.insert(user);
    }
}
```

- 이와 같이 반복적으로 발생하는 중복 로직을 상위 클래스가 구현하고 변화가 발생하는 부분만 추상 메소드로 만들어 구현하도록 하는 디자인 패턴을 템플릿 메소드(Template Method) 패턴이라 한다.

### **7.4.4 InsertJdbcTemplate과 UpdateJdbcTemplate 통합**

---

1. 공통 라이브러리를 담당할 클래스를 분리하여 굳이 메소드 이름을 ForInsert, ForUpdate와 같이 붙일 필요가 없어졌다. 따라서, 메소드 이름을 createQuery(), setValues()로 Rename 리팩토링 한다.
2. JdbcTemplate 클래스도 마찬가지로 Insert, Update 두 개의 클래스로 분리할 필요없이 하나의 JdbcTemplate 클래스로 통합하여 사용하고, 메소드 이름도 insert(), update()로 사용하도록 한다.
3. 하나로 통합된 JdbcTemplate 클래스를 사용하므로 UserDao의 insert(), update()메소드가 JdbcTmplate을 사용하도록 리팩토링 한다.

```java
public abstract class JdbcTemplate { // 2. JdbcTemplate 하나의 클래스 사용
	public void update(User user) throws SQLException{ // 2. Rename
		Connection con = null;
    	PreparedStatement pstmt = null;

    	try {
    		con = ConnectionManager.getConnection();
            String sql = createQuery(); // 1. Rename
            pstmt = con.prepareStatement(sql);
            setValues(user, pstmt); // 1. Rename
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

```java
public class UserDao {
    public void insert(User user) throws SQLException {
        JdbcTemplate jdbcTemplate = new JdbcTemplate() {

			@Override
			void setValues(PreparedStatement pstmt) throws SQLException {
				pstmt.setString(1, user.getUserId());
		        pstmt.setString(2, user.getPassword());
		        pstmt.setString(3, user.getName());
		        pstmt.setString(4, user.getEmail());
			}

			@Override
			String createQuery() {
		    	return "INSERT INTO USERS VALUES (?, ?, ?, ?)";

			}
        };
        jdbcTemplate.update(); // 3. UserDao가 JdbcTemplate 사용
    }
}
```

### **7.4.5 User 의존관계 제거 및 SQL 쿼리 인자로 전달**

---

### 1. User 읜존관계 제거

- JdbcTempalte을 UserDao가 아닌 다른 곳에서 사용하려면 User에 대한 의존관계도 끊어야 한다.

- setValues() 메소드를 통해 User 인자를 전달하지 않고 UserDao insert(), update() 메소드의 User 인스턴스에 직접 접근하도록 리팩토링 한다.

```java
public abstract class JdbcTemplate {
	public void update() throws SQLException{
		Connection con = null;
    	PreparedStatement pstmt = null;

    	try {
    		con = ConnectionManager.getConnection();
            String sql = createQuery();
            pstmt = con.prepareStatement(sql);
            setValues(pstmt); // User 인자를 전달하지 않도록 리팩토링
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

	// User 인자를 전달하지 않도록 리팩토링
    abstract void setValues(PreparedStatement pstmt) throws SQLException ;

}
```

```java
public class UserDao {
    public void insert(User user) throws SQLException {
        JdbcTemplate jdbcTemplate = new JdbcTemplate() {

			// User 인자를 전달하지 않도록 리팩토링
			@Override
			void setValues(PreparedStatement pstmt) throws SQLException {
				pstmt.setString(1, user.getUserId());
		        pstmt.setString(2, user.getPassword());
		        pstmt.setString(3, user.getName());
		        pstmt.setString(4, user.getEmail());
			}

			@Override
			String createQuery() {
		    	return "INSERT INTO USERS VALUES (?, ?, ?, ?)";
			}
        };
        jdbcTemplate.update();
    }
}
```

### 2. SQL 쿼리 인자로 전달

- 매번 2개의 추상 메소드를 구현하지 않고 SQL 쿼리를 JdbcTemplate의 update() 메소드 인자로 전달하기

```java
public abstract class JdbcTemplate {
	// sql 쿼리를 update() 메소드의 인자로 전달
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
```

```java
public class UserDao {
    public void insert(User user) throws SQLException {
        JdbcTemplate jdbcTemplate = new JdbcTemplate() {

			@Override
			void setValues(PreparedStatement pstmt) throws SQLException {
				pstmt.setString(1, user.getUserId());
		        pstmt.setString(2, user.getPassword());
		        pstmt.setString(3, user.getName());
		        pstmt.setString(4, user.getEmail());
			}

        };

		// sql 쿼리를 update() 메소드의 인자로 전달
        String sql = "INSERT INTO USERS VALUES (?, ?, ?, ?)";
        jdbcTemplate.update(sql);
    }
}
```

### **7.4.6 SELECT문에 대한 리팩토링**

---

SELECT의 경우 리팩토링 방법은 같지만, 조회한 데이터를 자바 객체로 변환해야 하는 부분이 추가적으로 필요하다.

- mapRow() 메소드를 추상 메소드로 추가하여 조회한 데이터를 자바 객체로 변환하는 부분을 구현한다.

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

### **7.4.7 JdbcTemplate과 SelectJdbcTemplate 통합**

---

두 클래스의 구현 부분에서 중복 코드가 많아 두 개의 클래스를 JdbcTemplate 하나로 통합한다.

```java
public abstract class JdbcTemplate {
	public void update(String sql) throws SQLException {
		[...]
	}

	@SuppressWarnings("rawtypes")
	public List query(String sql) throws SQLException{
		[...]
	}

	@SuppressWarnings("rawtypes")
	public Object queryForObject(String sql) throws SQLException{
		[...]
	}

	abstract Object mapRow(ResultSet rs) throws SQLException;

	abstract void setValues(PreparedStatement pstmt) throws SQLException;
}
```

```java
public class UserDao {
	public void insert(User user) throws SQLException {
		JdbcTemplate jdbcTemplate = new JdbcTemplate() {

			@Override
			void setValues(PreparedStatement pstmt) throws SQLException {
				pstmt.setString(1, user.getUserId());
				pstmt.setString(2, user.getPassword());
				pstmt.setString(3, user.getName());
				pstmt.setString(4, user.getEmail());
			}

			// mapRow() 구현 필요!
			@Override
			Object mapRow(ResultSet rs) throws SQLException {
				return null;
			}

		};
		String sql = "INSERT INTO USERS VALUES (?, ?, ?, ?)";
		jdbcTemplate.update(sql);
	}

	...
}

```

- 두 개의 클래스를 하나로 통합한 결과 UserDao의 insert(), update() 메소드에서 다음과 같이 mapRow() 메소드를 구현해야 하는 수정 사항이 발생했다.

### **7.4.8 인터페이스 추가를 통한 문제점 해결**

---

**수정 사항 해결 방법**

- setValues() 메소드와 mapRow() 메소드를 분리해 서로 간의 의존관계를 끊어야 한다.
- 두 개의 추상 메소드를 같은 클래스가 가지도록 구현하지 말고, 각각의 추상 메소드를 인터페이스를 통해 분리한다.

```java
public interface PreparedStatementSetter { // 추상 메소드를 인터페이스로 분리
	void setValues(PreparedStatement pstmt) throws SQLException;
}
```

```java
public interface RowMapper { // 추상 메소드를 인터페이스로 분리
	Object mapRow(ResultSet rs) throws SQLException;
}
```

- 이와 같이 JdbcTemplate에 구현되어 있는 2개의 추상 메소드를 인터페이스로 분리한다.

```java
public  class JdbcTemplate {
	// PreparedStatementSetter 인터페이스 활용
	public void update(String sql,PreparedStatementSetter pss) throws SQLException {
		Connection con = null;
		PreparedStatement pstmt = null;

		try {
			con = ConnectionManager.getConnection();
			pstmt = con.prepareStatement(sql);
			pss.setValues(pstmt); // PreparedStatementSetter 인터페이스 활용
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
	// PreparedStatementSetter와 RowMapper 인터페이스 활용
	public List query(String sql, PreparedStatementSetter pss, RowMapper rowMapper) throws SQLException{
		Connection con = null;
		PreparedStatement pstmt = null;
		ResultSet rs =null;
		try {
			con = ConnectionManager.getConnection();
			pstmt = con.prepareStatement(sql);
			pss.setValues(pstmt); // PreparedStatementSetter 인터페이스 활용
			rs = pstmt.executeQuery();
			List<Object> result = new ArrayList<Object>();
			while(rs.next()) {
				result.add(rowMapper.mapRow(rs)); // RowMapper 인터페이스 활용
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
```

- 메소드 하나만 가지는 인터페이스를 생성한 후 필요에 따라 메소드의 인자로 전달하여 문제점 해결했다. 변화 시점이 다른 부분을 서로 다른 인터페이스로 분리하여 공통 라이브러리에 대한 유연함을 높였다.
- 이 예제에서 사용한 인터페이스를 콜백(Callback) 인터페이스라고 부른다.

### **7.4.9 런타임 Exception 추가 및 AutoClosable 활용한 자원 반환**

---

UserDao 문제점 중의 하나는 모든 메소드가 컴파일타임 Exception인 SQLException을 throw한다는 것이다

- 런타임 Excetion을 추가해 이 문제점을 해결한다.

1. RuntimeException을 상속하는 새로운 Exception 추가

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
		super(cause);
	}
}
```

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
		} finally { // 복잡도 증가
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

- DataAccessException을 추가한 후 JdbcTemplate을 다음과 같이 리팩토링함으로써 JdbcTemplate을 사용하는 곳에서 더 이상 SQLException을 처리하지 않게 되었다.
- 그러나 finally 절의 복잡도가 너무 높아졌다.

```java
public  class JdbcTemplate {
	public void update(String sql,PreparedStatementSetter pss) throws DataAccessException {
		try (Connection con = ConnectionManager.getConnection();
				PreparedStatement pstmt = con.prepareStatement(sql)){
			pss.setValues(pstmt);
			pstmt.executeUpdate();

		} catch (SQLException e) {
			throw new DataAccessException(e);
		}
	}
```

- close() 메소드 : java.io.AutoClosable 인터페이스를 활용해 해결(AutoClosable 인터페이스를 구현하는 클래스는 try-with-resources 구문을 활용해 자원을 자동으로 반납할 수 있다.)
- 위 코드의 경우 Connection과 PreparedStatement는 AutoClosable을 구현하고 있어서 자동으로 close된다.

### **7.4.10 제너릭(generic)을 활용한 개선**

---

데이터를 조회할 때 매번 캐스팅을 해야 한다는 불편함이 있다.

- 자바의 제너릭을 이용해 캐스팅을 하지 않도록 한다.

```java
public interface RowMapper<T>{ // 제너릭 이용
  T mapRow(ResultSet rs) throws SQLException;
}
```

- 자바의 제너릭 문법을 이용하도록 수정한다.

```java
public class JdbcTemplate {
	// 제너릭 이용
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

	// 제너릭 이용
    public <T> List<T> query(String sql,  PreparedStatementSetter pss,RowMapper<T> rowMapper)  throws DataAccessException {
      List<T> result = query(sql,pss,rowMapper);
      if(result.isEmpty()){
        return null;
      }
      return result.get(0);
    }

}
```

```java
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

		// 캐스팅을 하지 않아도 된다.
		RowMapper<User> rowMapper = new RowMapper<User>() {
			@Override
			// Object -> User
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

- 제너릭을 사용하도록 리팩토링하면 데이터를 조회할 때 캐스팅을 하지 않아도 된다.

### **7.4.11 가변인자를 활용해 쿼리에 인자 전달하기**

---

쿼리에 값을 전달할 때 PreparedStatement를 활용하지 않고 가변인자를 활용해 값을 전달할 수 있다.

- JdbcTemplate에 가변인자를 활용하는 새로운 update() 메소드를 추가한다.

```java
public class JdbcTemplate {
	// 새로운 update() 메소드 추가
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
	...
}
```

```java
public class UserDao{
  public void insert(User user) throws SQLException {
      JdbcTemplate jdbcTemplate = new JdbcTemplate();
      String sql = "INSERT INTO USERS VALUES (?, ?, ?, ?)";
	  // 새로 추가한 update() 메소드 사용
      jdbcTemplate.update(sql, user.getUserId(),user.getPassword(),user.getName(),user.getEmail());
    }

	...
}
```

- UserDao가 새로운 update() 메소드를 사용하도록 리팩토링한다.

### **7.4.12 람다를 활용한 구현**

---

UserDao에서 RowRapper에 대한 익명 클래스를 생성하던 부분을 다음과 같이 람다를 활용해 좀 더 깔끔하게 구현할 수 있다.

```java
public class UserDao {
 public User findByUserId(String userId) {
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        String sql = "SELECT userId, password, name, email FROM USERS WHERE userid=?";
        return jdbcTemplate.queryForObject(sql, (ResultSet rs) -> {
          return new User(rs.getString("userId"),rs.getString("password"),rs.getString("name"),rs.getString("email")}, userId);
    }
}
```

- 익명 클래스를 만들지 않고 람다를 사용할 경우, 메소드에 전달할 인자와 메소드 구현부만 구현하여 전달할 수 있다.
- 단, 람다를 사용하려면 RowMapper와 같이 인터페이스의 메소드가 하나만 존재해야 한다.

```java
@FunctionalInterface // 애노테이션으로 람다 표현식으로 사용할 인터페이스라고 표시
public interface RowMapper<T> {
	T mapRow(ResultSet rs) throws SQLException;
}
```

- @FunctionalInterface 애노테이션으로 람다 표현식으로 사용할 인터페이스라고 표시한다.

## **📌 이번 실습의 목표**

---

1. 콜백 인터페이스를 활용했을 떄의 유연함
   - 콜백 인터페이스는 라이브러리를 만들거나 코드에 유연함이 필요한 경우 유용하게 사용할 수 있다.
2. 리팩토링을 안전하게 진행해야 한다
   - 리팩토링은 새로운 기능을 추가하지 않으면서 설계를 개선하는 작업이므로, 리팩토링 과정에서 테스트를 통해 기능이 정상적으로 동작하는지 반드시 확인해야 한다.
