# 3.4. 웹 서버 실습

📌 HTTP의 request/response 의 약속된 규약

- 웹 클라이언트가 웹 서버에 요청을 보내기 위한 규약
    
    ```
    **POST /user/create HTTP/1.1**  ⇒ 요청 라인
    HOST : localhost:8080       ⇒ 요청 헤더
    Connection-Length : 59      ⇒ 요청 헤더
    Content-Type: application/x-wwww-form-urlencoded   ⇒ 요청 헤더
    Accept: */*   ⇒ 요청 헤더
    							⇒ 헤더와 본문 사이의 빈 공백 라인
    userId=yejin1234&password=1234 ⇒ 요청 본문
    ```
    
    - 요청 라인
        
        `HTTP-메소드 URI HTTP-버전` 으로 구성되어 있다.
        
        - HTTP 메소드는 요청의 종류를 나타낸다. Ex) GET, POST
        - URI와 URL이 혼용되어 사용되는데 거의 같은 의미다.
        - HTTP-버전은 현재 요청의 HTTP 버전으로 현재 HTTP/1.1이 주로 사용되고 있다.
        
    - 요청 헤더
        
        `<필드 이름> : <필드 값> 쌍`으로 이루어져 있다.
        
        이때, 필드 이름 하나에 여러 개의 필드 값을 전달하고 싶다면 쉼표(,)를 구분자로 전달할 수 있다. Ex) <필드 이름> : <필드 값1>, <필드 값2>
        
- 웹 서버가 웹 클라이언트에게 응답을 보내기 위한 규약
    
    ```
    HTTP/1.1 200 OK ⇒ 상태 라인
    Content-Type: text/html; charset=utf-8 ⇒ 응답 헤더
    Content-Length: 20 ⇒ 응답 헤더
    									 ⇒ 헤더와 본문 사이의 빈 공백 라인
    <h1>Hello World</h1> 	⇒ 응답 본문 
    ```
    
    - 상태 라인
        
        `HTTP-버전 상태코드 응답구문` 으로 구성되어 있다.
        
        💁🏻‍♀️ HTTP 상태 코드
        
        - 2xx(Successful) : **성공** 	⇒ 요청을 정상적으로 처리했음을 의미한다.
        - 3xx(Redirection) : **리다이렉션** 완료  ⇒ 요청 완료를 위해 추가 작업 조치가 필요함을 의미한다. 주로 리다이렉트 할 때 많이 사용한다.
        Ex) 302는 리다이렉트 시킬 때 HTTP 메서드를 GET으로 바꾸고, body 없이 전송한다.
        - 4xx(Client Error) : 요청 오류 	⇒ **클라이언트 오류**로 인해 서버가 요청을 처리할 수 없음을 의미한다.
        - 5xx(Server Error) : 서버 오류 	⇒ **서버 오류**로 인해 서버가 정상 요청을 처리하지 못함을 의미한다.

---

### 3.4.1 요구사항 1 - index.html 응답하기

**✔️ `http://localhost:8080/index.html` 로 접속했을 때 webapp 디렉토리의 index.html 파일을 읽어 클라이언트에 응답한다.**

1. BufferedReader.readLine() 메소드를 활용해 라인별로 HTTP 요청 정보를 읽는다.
    
    ```java
    // 1단계
    InputStreamReader isr = new InputStreamReader(in);
    BufferedReader br = new BufferedReader(isr);
    String line = br.readLine();
            	
    // line이 null 값인 경우에 대한 예외 처리를 해야 한다. 아니면 무한 루프에 빠진다.
    if(line == null) {
          return;
    }
    ```
    

1. HTTP 요청 라인에서 요청 URL(우리는 `/index.html` 을 추출한다)
    
    **GET /index.html HTTP/1.1  ⇒ 요청 라인**
    
    ```java
    // 2단계
    String[] lines = line.split(" ");
    String url = lines[1]; // /index.html
    ```
    

1. 요청 URL에 해당하는 파일을 webapp 디렉토리에서 읽어 전달한다.
    
    ```java
    // 3단계
    DataOutputStream dos = new DataOutputStream(out);
    byte[] body = Files.readAllBytes(new File("./webapp" + url).toPath()); // index.html의 태그가 body[]에 담긴다.
    response200Header(dos, body.length);
    responseBody(dos, body);
    
    **⇒ 이때 url은 /index.html 이다.**
    ```
    

💁🏻‍♀️ client-server가 socket 통신을 할 때 Object가 아닌 byte 단위로 변환해서 보내는 이유?

: client-server 모두 Java로 구현되어 있을 때는 Object로 데이터를 주고 받아도 상관 없지만, 상대방이 c++이라면 Java의 Object를 이해하지 못한다(c++의 구조체를 자바에서 이해 못하니까 서로 byte 단위로 정보를 주고 받아야 한다)

💁🏻‍♀️ InputStream ⇒ InputStreamReader ⇒ BufferedReader

| InputStream | byte | InputStream in = System.in; | read() |
| --- | --- | --- | --- |
| InputStreamReader | char | InputStreamReader isr = new InputStreamReader(in); | read() |
| BufferedReader | String | BufferedReader br = new BufferedReader(isr); | readLine() |
- InputStream : 1byte 읽기
    
    : InputStream 객체의 System.in 객체를 사용해서, read() 메소드로 입력을 받는다.
    
    🖍️ read() 메소드는 1byte의 int 자료형으로 저장된다. (이때의 int는 아스키코드 값)
    
    ```java
    InputStream in = System.in;
    int a = in.read();
    ```
    
- InputStreamReader : 문자로 읽기
    
    ```java
    InputStreamReader isr = new InputStreamReader(in);
    char[] c = new char[3];
    isr.read(c);
    System.out.println(c);
    ```
    
    ⇒ 배열 크기를 일일이 정해야 하는 문제점이 있음
    
- BufferedReader : 통째로 읽기
    
    ```java
    BufferedReader br = new BufferedReader(isr);
    System.out.println(br.readLine());
    ```
    

---

### 3.4.3.2 요구사항 2 - GET 방식으로 회원가입하기

**✔️ “회원가입” 메뉴를 클릭하면 `http://localhost:8080/user/form.html` 으로 이동하면서 회원가입할 수 있다.** 

```java
private void responseResource(OutputStream out, String url) throws IOException{
		DataOutputStream dos = new DataOutputStream(out);
		byte[] body = Files.readAllBytes(new File("./webapp" + url).toPath());
		response200Header(dos, body.length);
		responseBody(dos, body);
}
**⇒ 이때 url은 user/form.html 이다.**	

```

---

### 3.4.3.3 요구사항 3 - POST 방식으로 회원 가입하기

**✔️ `http://localhost:8080/user.form.html` 파일의 form 태그 method를 get에서 post로 수정한 후 회원가입 기능이 정상적으로 동작하도록 구현한다.**


![Untitled (1)](https://user-images.githubusercontent.com/81174840/235858486-c64fdf8c-d51d-421b-8d53-efc62ec9c4bb.png)

**⇒ form.html에서 method를 post로 변경하기!**

```java
// TODO : POST 방식으로 회원가입

if(lines[0].equals("POST")) { // POST 방식으로 요청이 들어오는 경우
				int contentLength = 0; // pathVariable의 길이

				while (!"".equals(line)) {
					System.out.println(line);
					
					
					line = br.readLine();

					if (line.contains("Content-Length"))
						contentLength = Integer.parseInt(line.split(": ")[1]);

				}
				
				// br은 요청 본문을 가리키고 있다.
				String pathVariable = IOUtils.readData(br, contentLength);
				
				// 데이터를 map에 넣어서 받아온다.
				Map<String, String> params = HttpRequestUtils.parseQueryString(pathVariable);
				
				
				if(url.equals("/user/create")) {  // post 방식으로 회원 가입하는 경우
					User user = new User(params.get("userId"), params.get("password"), params.get("name"),
							params.get("email")); // user 객체를 만든다.
					
					DataBase.addUser(user); // DB에 user 객체를 넣는다.
				}
}
```

---

### 3.4.3.4 요구사항 4 - 302 status code 적용

**✔️ “회원가입”을 완료하면 index.html 페이지로 이동하고 HHTP 응답 헤더의 status code를 200이 아니라 302 code를 사용한다.**

```java
DataOutputStream dos = new DataOutputStream(out);
response302Header(dos, "/index.html");
```

```java
private void response302Header(DataOutputStream dos, String url) {
		try {
			dos.writeBytes("HTTP/1.1 302 Redirect \r\n");
			dos.writeBytes("Location: "+url+"\r\n");
			dos.writeBytes("\r\n");
		} catch (IOException e) {
			log.error(e.getMessage());
		}
	}
```

- 클라이언트는 첫 라인의 상태 코드를 확인한 후 302라면 Location의 값을 읽어 서버에 재요청을 보내게 된다.
- 이와 같은 과정으로 요청을 보내면 클라이언트의 요청은 회원가입 처리를 위한 /user/create 요청이 아니라 /index.html 요청으로 변경된다.
- 302 상태 코드를 활용한 페이지 이동 방식은 많은 라이브러리와 프레임워크에서 리다이렉트 이동 방식으로 알려져 있다.
    
    

---

### 3.4.3.5 요구사항 5 -로그인하기

**✔️ “로그인” 메뉴를 클릭하면 `http://localhost:8080/user/login.html` 으로 이동해 로그인 할 수 있다 → 로그인이 성공하면 /index.html로 이동하고, 로그인이 실패하면 /user/login_failed.html로 이동해야 한다.**

```java
if(url.equals("/user/login")) { 

			User user = DataBase.findUserById(params.get("userId"));
					
			if(user == null) { // DB에 회원의 정보가 없는 경우
				responseResource(out, "/user/login_failed.html");
				return;
			}
					
			if(user.getPassword().equals(params.get("password"))) { // DB에 저장한 비밀번호와 사용자가 입력한 비밀번호가 동일한 경우
				DataOutputStream dos = new DataOutputStream(out);
				response302LoginSuccessHeader(dos); // Header 요청 라인의 상태 코드를 302로 변경하고 Location을 추가한다.
			}else {
				responseResource(out, "/user/login_failed.html"); // 로그인에 실패한 경우 
			}					
}
```

- 사용자의 아이디를 비교하여 DB에 회원의 정보가 있는지 확인 ⇒ DB에 저장된 회원의 비밀번호와 사용자의 비밀번호가 같은 지 비교한다.
- 로그인에 실패한 경우 /user/login_failed.html 페이지로 이동한다.

---

### 3.4.3.6 요구사항 6 - 사용자 목록 출력

**✔️ 접근하고 있는 사용자가 로그인 상태일 경우 (즉, Cookie 값이 logined = true) 에는`http://localhost:8080/user/list`로 접근했을 때 사용자 목록을 출력한다. 만약 로그인하지 않은 상태라면 로그인 페이지로 이동한다.**

```java
if(url.equals("/user/list")) { // 사용자의 목록을 조회한다.
		if(!logined) {
			responseResource(out, "/user/login.html");
			return;
		}
		Collection<User> userList = DataBase.findAll();
		StringBuilder sb = new StringBuilder();
		sb.append("<table border='1'>");
		sb.append("<tr> <td>아이디</td> <td>이름</td> <td>이메일</td> </tr>");
		for(User user : userList) {
			sb.append("<tr>");
			sb.append("<td>"+user.getUserId()+"</td>");
			sb.append("<td>"+user.getName()+"</td>");
			sb.append("<td>"+user.getEmail()+"</td>");
			sb.append("</tr>");
		}
		sb.append("</table>");
		byte[] body = sb.toString().getBytes();
		DataOutputStream dos = new DataOutputStream(out);
		response200Header(dos, body.length);
		responseBody(dos, body);
}
```

- HTTP는 기본적으로 무상태 프로토콜이라 각 요청 간에 상태 데이터를 공유하지 못한다.
    
    💁🏻‍♀️ 무상태 프로토콜?
    
    - 통신은 기본적으로 클라이언트와 서버 사이의 의사소통인데 서로가 통신을 할 때 상태정보, 세션 등을 요구하지 않는 것이 무상태의 정의가 된다.
        
        [장점]
        
        - 서버 디자인을 단순하게 만들어 주며, 리소스의 소비를 억제해준다.
        
        [단점]
        
        - 인증정보에 대해 매 요청마다 인증을 해야하는 불편함이 발생할 수 있다.
        
        [무상태 프로토콜의 단점을 극복할 수 있는 방안]
        
        - 쿠키를 이용해서 최초에 서버가 response를 보낼 때 쿠키정보를 클라이언트에게 전달하고, 클라이언트는 서버와 다시 통신할 때 쿠키정보를 헤더에 담아 전달한다.
            
            서버는 이를 통해 클라이언트의 정보를 확인한다.
            
- 쿠키를 사용하여 각 요청 간에 상태 정보를 공유한다. 이 방법은 HTTP에서 각 요청 간의 상태를 공유할 수 있는 유일한 방법이다.
- 쿠키를 사용할 경우 쿠키의 정보를 클라이언트에 저장해 관리하기 때문에 보안적으로 문제가 발생한다. 이 같은 단점을 보안하기 위해 세션을 사용해 상태 데이터를 서버에 저장한다.

---

### 3.4.3.7 요구사항 7 - CSS 지원하기

**✔️ CSS파일을 적용하기**

💁🏻‍♀️ 1~6번의 요구사항을 진행하면서 css 파일을 적용할 수 없었던 이유?

: 응답을 보낼 때 모든 컨텐츠의 타입을 text/html 로 보내는 것 때문이다.

  (브라우저는 응답을 받은 후 Content-Type 헤더 값을 통해 응답 본문(body)에 포함되어 있는 

   텐츠가 어떤 컨텐츠인지를 판단한다. 그런데 지금까지 구현한 모든 응답은 text/html 로 고정

   어 있어 브라우저는 css 파일도 HTML로 인식했기 때문에 정상적으로 동작하지 않았던 것이다)

⇒ 따라서 Content-Type 헤더 값을 text/html이 아니라 text/css 로 응답을 보내면 css 파일을 인식할 수 있다.

```java
// css 파일을 설정한다.
if(url.endsWith(".css")) {
		DataOutputStream dos = new DataOutputStream(out);
		byte[] body = Files.readAllBytes(new File("./webapp" + url).toPath());
		response200CssHeader(dos, body.length);
		responseBody(dos, body);
}
```

```java
private void response200CssHeader(DataOutputStream dos, int lengthOfBodyContent) {
		try {
			dos.writeBytes("HTTP/1.1 200 OK \r\n");
			dos.writeBytes("Content-Type: text/css\r\n");
			dos.writeBytes("Content-Length: "+lengthOfBodyContent+"\r\n");
			dos.writeBytes("\r\n");
		}catch(Exception e) {
			e.printStackTrace();
		}
	}
```

---

- RequestHandler.java
    
    ```java
    package webserver;
    
    import java.io.BufferedReader;
    import java.io.DataOutputStream;
    import java.io.File;
    import java.io.IOException;
    import java.io.InputStream;
    import java.io.InputStreamReader;
    import java.io.OutputStream;
    import java.net.Socket;
    import java.nio.file.Files;
    import java.util.Collection;
    import java.util.List;
    import java.util.Map;
    
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    
    import db.DataBase;
    import model.User;
    import util.HttpRequestUtils;
    import util.IOUtils;
    
    public class RequestHandler extends Thread {
    	private static final Logger log = LoggerFactory.getLogger(RequestHandler.class);
    
    	private Socket connection;
    
    	public RequestHandler(Socket connectionSocket) {
    		this.connection = connectionSocket;
    	}
    
    	public void run() {
    		log.debug("New Client Connect! Connected IP : {}, Port : {}", connection.getInetAddress(),
    				connection.getPort());
    
    		try (InputStream in = connection.getInputStream(); OutputStream out = connection.getOutputStream()) {
    			// TODO 사용자 요청에 대한 처리는 이 곳에 구현하면 된다.
    
    			// 1단계
    			InputStreamReader isr = new InputStreamReader(in);
    
    			BufferedReader br = new BufferedReader(isr);
    
    			String line = br.readLine();
    
    			// line이 null 값인 경우에 대한 예외 처리를 해야 한다. 아니면 무한 루프에 빠진다.
    			if (line == null) {
    				return;
    			}
    
    			// 2단계
    			String[] lines = line.split(" "); // Http header를 나눈다.
    
    			String url = lines[1]; // 요청 url
    
    			String[] root = url.split("/");
    
    			// Http Header를 분석한다.
    			int contentLen = 0;
    			boolean logined = false;
    			while (!"".equals(line)) {
    
    				line = br.readLine();
    
    				if (line.contains("Content-Length")) {
    					contentLen = contentLength(line);
    				} else if (line.contains("Cookie")) {
    					logined = isLogin(line);
    				}
    
    			}
    
    			if (url.equals("/user/create")) { // 회원가입
    				// byte형식으로 받아온 pathVariable을 String 타입으로 반환해서 변수에 저장한다.
    				String pathVariable = IOUtils.readData(br, contentLen);
    				
    				// pathVariable을 parsing한다.
    				Map<String, String> params = HttpRequestUtils.parseQueryString(pathVariable);
    				
    				User user = new User(params.get("userId"), params.get("password"), params.get("name"),
    						params.get("email"));
    
    				DataBase.addUser(user);
    
    				DataOutputStream dos = new DataOutputStream(out);
    
    				response302Header(dos, "/index.html");
    
    			} else if (url.equals("/user/login")) { // 로그인
    				String pathVariable = IOUtils.readData(br, contentLen);
    
    				Map<String, String> params = HttpRequestUtils.parseQueryString(pathVariable);
    
    				User user = DataBase.findUserById(params.get("userId"));
    
    				if (user == null) {// DB에 회원의 정보가 없는 경우
    					responseResource(out, "/user/login_failed.html");
    					return;
    				}
    
    				if (user.getPassword().equals(params.get("password"))) { // DB에 저장한 비밀번호와 사용자가 입력한 비밀번호가 동일한 경우
    					DataOutputStream dos = new DataOutputStream(out);
    					response302LoginSuccessHeader(dos); // Header 요청 라인의 상태 코드를 302로 변경하고 Location을 추가한다.
    				} else {
    					responseResource(out, "/user/login_failed.html"); // 로그인에 실패한 경우
    				}
    			} else if (url.equals("/user/list")) { // 사용자의 목록을 조회한다.
    				if (!logined) {
    					responseResource(out, "/user/login.html");
    					return;
    				}
    				Collection<User> userList = DataBase.findAll();
    				StringBuilder sb = new StringBuilder();
    				sb.append("<table border='1'>");
    				sb.append("<tr> <td>아이디</td> <td>이름</td> <td>이메일</td> </tr>");
    				for (User user : userList) {
    					sb.append("<tr>");
    					sb.append("<td>" + user.getUserId() + "</td>");
    					sb.append("<td>" + user.getName() + "</td>");
    					sb.append("<td>" + user.getEmail() + "</td>");
    					sb.append("</tr>");
    				}
    				sb.append("</table>");
    				byte[] body = sb.toString().getBytes();
    				DataOutputStream dos = new DataOutputStream(out);
    				response200Header(dos, body.length);
    				responseBody(dos, body);
    			} else if (url.endsWith(".css")) {
    				DataOutputStream dos = new DataOutputStream(out);
    				byte[] body = Files.readAllBytes(new File("./webapp" + url).toPath());
    				response200CssHeader(dos, body.length);
    				responseBody(dos, body);
    			} else {
    				responseResource(out, url);
    			}
    
    		} catch (IOException e) {
    			log.error(e.getMessage());
    		}
    	}
    	
    	// ● contentLength : content-Length의 길이를 반환하는 메소드
    	public int contentLength(String line) {
    		return Integer.parseInt(line.split(": ")[1]);
    	}
    
    	// ● isLogin : Cookie의 상태를 반환하는 메소드
    	// Ex) line의 형태 → Cookie: logined=true
    	public boolean isLogin(String line) {
    		String state = line.split(": ")[1].split("=")[1];
    		return Boolean.parseBoolean(state);
    	}
    
    	// ● response302Header : Header의 상태 코드를 302으로 하고 url의 경로로 redirect 하는 메소드
    	private void response302Header(DataOutputStream dos, String url) {
    		try {
    			dos.writeBytes("HTTP/1.1 302 Redirect \r\n");
    			dos.writeBytes("Location: " + url + "\r\n");
    			dos.writeBytes("\r\n");
    		} catch (IOException e) {
    			log.error(e.getMessage());
    		}
    	}
    
    	// ● response200Header : Header의 상태 코드를 200으로 하고 pathVariable의 길이를 반환하는 메소드
    	private void response200Header(DataOutputStream dos, int lengthOfBodyContent) {
    		try {
    			dos.writeBytes("HTTP/1.1 200 OK \r\n");
    			dos.writeBytes("Content-Type: text/html;charset=utf-8\r\n");
    			dos.writeBytes("Content-Length: " + lengthOfBodyContent + "\r\n");
    			dos.writeBytes("\r\n");
    		} catch (IOException e) {
    			log.error(e.getMessage());
    		}
    	}
    	
    	// ● responseBody : 응답 body를 출력스트림에 쓰고 server로 HTTP header, body를 보낸다.
    	private void responseBody(DataOutputStream dos, byte[] body) {
    		try {
    			dos.write(body, 0, body.length); // body를 출력스트림에 쓴다.
    			dos.flush(); // 보낸다.
    		} catch (IOException e) {
    			log.error(e.getMessage());
    		}
    	}
    	
    	// ● responseResource : HTTP header와 Body를 생성한다. 
    	private void responseResource(OutputStream out, String url) throws IOException {
    		DataOutputStream dos = new DataOutputStream(out);
    		byte[] body = Files.readAllBytes(new File("./webapp" + url).toPath());
    		response200Header(dos, body.length);
    		responseBody(dos, body);
    	}
    
    	// ● response302LoginSuccessHeader : 로그인을 했으므로 Cookie의 상태를 true로 만들고 index.html로 redirect하는 메소드
    	private void response302LoginSuccessHeader(DataOutputStream dos) {
    		try {
    			dos.writeBytes("HTTP/1.1 302 Redirect \r\n");
    			dos.writeBytes("Set-Cookie: logined=true \r\n");
    			dos.writeBytes("Location: /index.html \r\n");
    		} catch (IOException e) {
    			e.printStackTrace();
    		}
    	}
    	
    	// ● response200CssHeader : Http Header를 생성하는 메소드
    	private void response200CssHeader(DataOutputStream dos, int lengthOfBodyContent) {
    		try {
    			dos.writeBytes("HTTP/1.1 200 OK \r\n");
    			dos.writeBytes("Content-Type: text/css\r\n");
    			dos.writeBytes("Content-Length: " + lengthOfBodyContent + "\r\n");
    			dos.writeBytes("\r\n");
    		} catch (Exception e) {
    			e.printStackTrace();
    		}
    	}
    }
    ```
    

# 3.5. 추가 학습 자료

## 3.5.3 디버깅을 위한 로깅(logging)

- System.out.println()으로 메세지를 출력하는 방법?
    
    : System.out.println() 으로 메세지를 출력하는 방법은 어플리케이션 성능을 저하 시키는 원인이 된다.
    
    - 성능을 저하 시키는 원인은❓
        
        : System.out.println()으로 디버깅 메세지를 출력하면 파일로 메세지가 출력하게 되는데 파일에 메세지를 출력하는 작업은 상당한 비용을 발생 시키는 작업이다.
        
    
    ⇒ 위와 같은 단점을 보완하기 위해 등장한 라이브러리가 로깅 라이브러리이다.
    

- 로깅(logging) 라이브러리?
    - 과거에는 Log4J를 많이 사용했지만 최근에는 더 좋은 성능을 자랑하는 Logback을 사용할 것을 추천한다.
    
    [동작 방식]
    
    자바 소스 코드가 SLF4J 라이브러리를 사용해 디버깅 메세지를 남기면 실제로 디버깅 메세지를 출력하는 구현체는 Log4J, Logback이 담당하는 방식으로 동작한다.
    
    [장점] 추후 Logback보다 더 좋은 로깅 라이브러리가 등장할 경우 소스 코드는 수정할 필요 없이 구현체를 담당할 로깅 라이브러리만 교체하면 된다.
    
    - 자바 진영의 로깅 라이브러리는 메세지 출력 여부를 로그 레벨을 통해 관리한다.
        - 로그 레벨 : `TRACE < DEBUG < INFO < WARN < ERROR` 순으로 높아진다.
            
                 → 로그 레벨이 높을수록 출력 되는 메세지는 적어진다.
            
            Ex) WARN을 로그 레벨로 설정하면 WARN, ERROR 레벨의 메세지만 출력된다.
            
            ```
            [로그 레벨을 구현할 때 사용하는 메소드]
            1. TRACE - log.trace()
            2. INFO - log.info()
            3. WARN - log.warn()
            4. ERROR - log.error() 
            ```
            
            📌 주의
            
            ```java
            log.debug("New Client Connect! Connected IP : " + connection.getInetAddress() + ", Port : " + cnnection.getPort());
            ```
            
            ⇒ 이때는 INFO, WARN인 경우 굳이 debug() 메소드에 인자를 전달하기 위해 문자열을 더하는 부분이 실행될 필요가 없다.
            
***

# ● 서버 배포
![Untitled](https://user-images.githubusercontent.com/81174840/235858228-7e8b5cf4-9e9c-4eb2-b89d-91832785e35e.png)

