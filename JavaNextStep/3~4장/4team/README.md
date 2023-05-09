# 3장 개발 환경 구축 및 웹 서버 실습 및 요구사항

## **개요**

- 라이브러리를 만들고, 프레임워크를 구현하기 위한 첫 번째 단계로 로컬에서 웹 애플리케이션을 개발, 버전 관리 시스템에서 소스코드를 관리, 개발한 웹 애플리케이션을 원격 서버에 실제 배포하는 경험을 하고, 질문/답변 게시판에 대한 서비스 요구사항과 실습 요구사항을 제시하고 있다.
- 또한 HTTP 웹 서버를 직접 구현하는 경험을 함으로써 웹 클라이언트와 서버 간에 데이터를 어떻게 주고 받는지에 대해 학습한다.

---

## **실습 환경 세팅 및 소스코드 분석**

- webserver.WebServer 를 실행한 후 브라우저에서 http://localhost:8080으로 접근했을 때 "Hello World" 메시지가 출력되는지 확인한다.
- WebServer 클래스는 웹 서버를 시작하고, 사용자의 요청이 있을 때까지 대기 상태에 있다가 사용자의 요청이 있을 경우 사용자의 요청을 RequestHandler에 작업을 위임하는 역할을 한다.
- 사용자 요청에 대한 모든 처리는 RequestHandler 클래스의 run() 메서드가 담당한다.

---

## **HTTP**

### 1. 웹 클라이언트가 웹 서버에 요청을 보내기 위한 규약

```
POST /user/create HTTP/1.1    → 요청 라인
HOST : localhost:8080         → 요청 헤더
Content-Length : 59           → 요청 헤더
Content-Type: application/x-wwww-form-urlencoded → 요청 헤더
Accept: */*                   → 요청 헤더
							  → 헤더와 본문 사이의 빈 공백 라인
userId=yejin1234&password=1234 → 요청 본문
```

- 요청 라인 (Reqeust Line)

  - 요청 데이터의 첫 번째 라인
  - "HTTP-메소드 URI HTTP-버전" 으로 구성되어 있다.
  - HTTP-버전은 현재 요청의 HTTP 버전으로 현재 HTTP/1.1이 주로 사용되고 있다.

- 요청 헤더 (Request Headers)
  - 두 번째 라인부터 빈 공백 문자열 라인
  - <필드 이름> : <필드 값> 쌍으로 이루어져 있다.

### 2. 웹 서버가 웹 클라이언트에게 응답을 보내기 위한 규약

```
TTP/1.1 200 OK                         → 상태 라인
Content-Type: text/html; charset=utf-8 → 응답 헤더
Content-Length: 20                     → 응답 헤더
					   → 헤더와 본문 사이의 빈 공백 라인
<h1>Hello World</h1> 	               → 응답 본문
```

- 상태 라인
  - 응답 메시지의 첫 번째 라인
  - HTTP-버전 상태코드 응답구문 으로 구성되어 있다.
  - 대표적으로 사용되는 상태 코드
    > 1. 2xx(Successful) : 성공 ⇒ 요청을 정상적으로 처리했음을 의미한다. <br>
    > 2. 3xx(Redirection) : 리다이렉션 완료 ⇒ 요청 완료를 위해 추가 작업 조치가 필요함을 의미한다. 주로 리다이렉트 할 때 많이 사용한다. Ex) 302는 리다이렉트 시킬 때 HTTP 메서드를 GET으로 바꾸고, body 없이 전송한다. <br>
    > 3. 4xx(Client Error) : 요청 오류 ⇒ 클라이언트 오류로 인해 서버가 요청을 처리할 수 없음을 의미한다. <br>
    > 4. 5xx(Server Error) : 서버 오류 ⇒ 서버 오류로 인해 서버가 정상 요청을 처리하지 못함을 의미한다.

---

## **요구사항 1 - index.html 응답하기**

> 요구사항: http://localhost:8080/index.html로 접속했을 때 webapp 디렉토리의 index.html 파일을 읽어 클라이언트에 응답한다.

- ### **요구사항 구현**

1. BufferedReader.readLine() 메소드를 활용해 라인별로 HTTP 요청 정보를 읽는다.

```java
BufferedReader br = new BufferedReader(new InputStreamReader(in));
String line = br.readLine();
```

2. HTTP 요청 라인에서 요청 URL(/index.html)을 추출한다.

```java
String[] tokens = line.split(" "); // line: GET /index.html HTTP/1.1
String url = tokens[1]; // /index.html
```

3. 요청 URL에 해당하는 파일을 webapp 디렉토리에서 읽어 전달한다.

```java
DataOutputStream dos = new DataOutputStream(out);
byte[] body = Files.readAllBytes(new File("./webapp" + url).toPath());
response200Header(dos, body.length);
responseBody(dos, body);
```

## **요구사항 2 - GET 방식으로 회원가입하기**

> 요구사항: "회원가입" 메뉴를 클릭하면 http://localhost:8080/user/form.html 으로 이동하면서 회원가입할 수 있다. 회원가입을 하면 다음과 같은 형태로 사용자가 입력한 값이 서버에 전달된다. /user/create?userId=id&password=pw&name=name&email=email%40email.com HTML과 URL을 비교해 보고 사용자가 입력한 값을 파싱(문자열을 원하는 형태로 분리하거나 조작하는 것을 의미)해 model.User 클래스에 저장한다.

1. HTTP 요청의 첫 번째 라인에서 요청 URL을 추출하고, URL에서 접근 경로와 `이름=값`으로 전달되는 데이터를 추출해 User 클래스에 담는다.

```java
if (url.startsWith("/user/create")) {
				int index = url.indexOf("?");

				String params = url.substring(index + 1);

				Map<String, String> map = util.HttpRequestUtils.parseQueryString(params);
				User user = new User(map.get("userId"), map.get("password"), map.get("name"), map.get("email"));
}
```

- `이름=값` 파싱은 util.HttpRequestUtils 클래스의 parseQueryString() 메소드를 활용한다.
- Get 방식으로 사용자가 입력한 데이터를 전달하는 데는 몇 가지 문제점이 있다.
  1. 사용자가 입력한 데이터가 브라우저 URL에 표시되기 때문에 보안 측면에서 좋지 않다.
  2. 요청 라인의 길이에 제한이 있다. 따라서 GET 방식으로 사용자가 입력할 수 있는 데이터 크기에도 제한이 있다.
- 따라서 GET 방식은 사용자가 입력한 데이터를 서버에 전송해 데이터를 추가할 때는 적합하지 않고, 이와 같은 한계를 극복하기 위해 HTTP는 POST 방식을 지원한다.

---

## **요구사항 3 - POST 방식으로 회원가입하기**

> 요구사항: http://localhost:8080/user/form.html 파일의 form 태그 method를 get에서 post로 수정한 후 회원가입 기능이 정상적으로 동작하도록 구현한다.

1. http://localhost:8080/user/form.html 파일의 form 태그 method를 get에서 post로 수정한다.

```html
<!-- form 태그 method를 get -> post 변경 -->
<!-- <form name="question" method="get" action="/user/create"> -->
<form name="question" method="post" action="/user/create"></form>
```

2. BufferedReader에서 본문 데이터는 util.IOUtils 클래스의 readData() 메소드를 활용한다. 본문의 길이는 HTTP 헤더의 Content-Length의 값이다.

```java
String bodyData = IOUtils.readData(br, contentLength);
```

---

## **요구사항 4 - 302 status code 적용**

> 요구사항: "회원가입"을 완료하면 /index.html 페이지로 이동하고 싶다. 현재는 URL이 /user/create 로 유지되는 상태로 읽어서 전달할 파일이 없다. 따라서 회원가입을 완료한 후 /index.html 페이지로 이동한다. 브라우저의 URL도 /user/create가 아니라 /index.html 로 변경해야 한다.

1. HTTP 응답 헤더의 status code를 200이 아니라 302 code를 사용한다.

```java
DataOutputStream dos = new DataOutputStream(out);
response302Header(dos, "/index.html");
```

```java
private void response302Header(DataOutputStream dos, String url) {
		try {
			log.debug("302실행");
			dos.writeBytes("HTTP/1.1 302 Found \r\n"); // \r\n : http 표준 줄바꿈 :CRLF
			dos.writeBytes("Location: " + url + "\r\n");
			dos.writeBytes("\r\n");
		} catch (IOException e) {
			log.error(e.getMessage());
		}
	}
```

- 위와 같이 응답을 보내면 클라이언트는 첫 라인의 상태 코드를 확인한 후 302라면 Location의 값을 읽어 서버에 재요청을 보내게 된다. 따라서 302 상태 코드를 활용해 페이지를 이동할 경우 요청과 응답이 두 번 발생한다.
- 302 상태 코드를 활용한 페이지 이동 방식은 많은 라이브러리와 프레임워크에서 리다이텍트 이동 방식으로 알려져 있다.

---

## **요구사항 5 - 로그인하기**

> 요구사항: "로그인" 메뉴를 클릭하면 http://localhost:8080/user/login.html 로 이동해 로그인할 수 있다. 로그인이 성공하면 /index.html 로 이동하고, 로그인이 실패하면 /user/login_failed.html 로 이동해야 한다. <br>
> 앞에서 회원가입한 사용자로 로그인할 수 있어야 한다. 로그인이 성공하면 로그인 상태를 유지할 수 있어야 한다. 로그인이 성공할 경우 요청 헤더의 Cookie 헤더 값이 logined=true, 로그인이 실패하면 Cookie 헤더 값이 logined=false로 전달되어야 한다.

1. 로그인 성공시 /index.html로 이동하고, 실패하면 /user/login_failed.html로 이동한다.

```java
String bodyData = IOUtils.readData(br, contentLength);
				log.debug("HTTP Request bodyData : {}", bodyData);
				Map<String, String> queryParams = util.HttpRequestUtils.parseQueryString(bodyData);
				User user = DataBase.findUserById(queryParams.get("userId"));
				if (user == null) {
					log.debug("Login Null Fail !");
				    responseLoginFailed(out);
				    return;
				}
				if (user.getPassword().equals(queryParams.get("password"))) {
				    log.debug("Login Success !");
				    DataOutputStream dos = new DataOutputStream(out);
				    response302HeaderLoginSuccess(dos);
				} else {
					log.debug("Login Fail !");
				    responseLoginFailed(out);
				    return;
				}
```

- userId와 password를 비교하여 로그인을한다.

2. 로그인 성공할 경우 요청 헤더의 Cookie 헤더 값을 logined=true 로 전달한다.

```java
private void response302HeaderLoginSuccess(DataOutputStream dos) {
	    try {
	    	log.debug("302 로그인 성공");
	        dos.writeBytes("HTTP/1.1 302 Found \r\n");
	        dos.writeBytes("Set-Cookie: logined=true; Path=/ \r\n");
	        dos.writeBytes("Location: /index.html \r\n");

	        dos.writeBytes("\r\n");
	    } catch (IOException e) {
	        log.error(e.getMessage());
	    }
	}
```

3. 로그인에 실패하면 Cookie 헤더 값이 logined=false로 전달되어야 한다.

```java
private void responseLoginFailed(OutputStream out) {
	    try {
	    	log.debug("로그인 실패");
	        DataOutputStream dos = new DataOutputStream(out);
	        byte[] body = Files.readAllBytes(new File("./webapp" + "/user/login_failed.html").toPath());
	        int lengthOfBodyContent = body.length;
	        dos.writeBytes("HTTP/1.1 200 OK \r\n");
	        dos.writeBytes("Content-Type: text/html;charset=utf-8\r\n");
	        dos.writeBytes("Set-Cookie: logined=false\r\n");
	        dos.writeBytes("Content-Length: " + lengthOfBodyContent + "\r\n");
	        dos.writeBytes("\r\n");
	        responseBody(dos, body);
	    } catch (IOException e) {
	        log.error(e.getMessage());
	    }
	}
```

---

## **요구사항 6 - 로그인 여부에 따른 사용자 목록 출력**

> 요구사항: 접근하고 있는 사용자가 "로그인" 상태일 경우 (Cookie 값이 logined=true) http://localhost:8080/user/list 로 접근했을 때 사용자 목록을 출력한다. 만약 로그인하지 않은 상태라면 로그인 페이지(login.html)로 이동한다.

1. 우선 로그인 여부를 확인하기 위해 Cookie 값을 확인한다.

```java
if(line.contains("Cookie")) {
  isLogined = loginCheck(line);
  log.debug("isLogined : {}", isLogined);
}
```

```java
private boolean loginCheck(String line) {
	String temps[] = line.split(":");
	Map<String, String> cookies = util.HttpRequestUtils.parseCookies(temps[1].trim());
	String valueCookie = cookies.get("logined");
	log.debug("valueCookie: {}", valueCookie);

	if(valueCookie == null) return false;
	return Boolean.parseBoolean(valueCookie);
}
```

2. 로그인 상태인 경우, 사용자 목록을 출력한다.

```java
Collection<User> userList = DataBase.findAll();

StringBuilder sb = new StringBuilder();

sb.append("<table border='1'>");
for(User u : userList) {
  sb.append("<tr>");
  sb.append("<td>" + u.getUserId() + "</td>");
  sb.append("<td>" + u.getName() + "</td>");
  sb.append("<td>" + u.getEmail() + "</td>");
  sb.append("</tr>");
}
sb.append("</table>");

DataOutputStream dos = new DataOutputStream(out);
byte[] body = sb.toString().getBytes();

response200Header(dos, body.length);
responseBody(dos, body);
```

3. 로그인하지 않은 상태인 경우, 로그인 페이지로 이동한다.

```java
if(!isLogined) {
	log.debug("Not Logined ! ");
	responseResource(out, "/user/login.html");
	return;
}
```

- Cookie는 웹 서버를 종료 후 재실행 하여도 남아 있기 때문에, 개발자 도구에서 Cookie를 직접 제거해 주어야 한다.

---

## **요구사항 7 - CSS 지원하기**

> 요구사항: 지금까지 구현한 소스코드는 CSS 파일을 지원하지 못하고 있다. CSS 파일을 지원하도록 구현한다.

1. 브라우저가 CSS 파일을 인식하도록 Content-Type 헤더 값을 수정한다.

```java
private void response200HeaderCss(DataOutputStream dos, int lengthOfBodyContent) {
  try {
    dos.writeBytes("HTTP/1.1 200 OK \r\n");
    dos.writeBytes("Content-Type: text/css\r\n");
    dos.writeBytes("Content-Length: " + lengthOfBodyContent + "\r\n");
    dos.writeBytes("\r\n");
  } catch (IOException e) {
    log.error(e.getMessage());
  }
}
```

- 지금까지 구현한 모든 응답은 text/html로 고정되어 있어 브라우저는 CSS 파일도 HTML로 인식했기 때문에 정상적으로 동작하지 않았다.

---

## **전체 코드**

### RequestHandler.java

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
import java.net.URLDecoder;
import java.nio.file.Files;
import java.util.Collection;
import java.util.Map;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import db.DataBase;
import model.User;
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
//			DataOutputStream dos = new DataOutputStream(out);
//            byte[] body = "Hello World".getBytes();

		// (p.96) 요구사항1 - [Hint]1단계: BufferedReader 생성.
		BufferedReader br = new BufferedReader(new InputStreamReader(in));

		// (p.96) 요구사항1 - [Hint]2단계: HTTP 요청 정보의 첫 번째 라인에서 요청 URL을 추출한다.
		String line = br.readLine();
		String[] tokens = line.split(" ");
		String url = tokens[1];
		// URL 디코딩을 해준다. @가 %40으로 나오기 때문!!
		url = URLDecoder.decode(url);

		// (p.101) 요구사항6 - 로그인 여부 확인
		Boolean isLogined = false;

		// (p.96) 요구사항1 - [Hint]1단계: HTTP 요청 정보 전체를 라인별로 출력한다.
		int contentLength = 0;
		while (line != null && !"".equals(line)) {
			log.debug(line);
			line = br.readLine();
			// (p.99) 요구사항3 - [Hint] 본문 데이터의 길이를 가져온다.
			if (line.contains("Content-Length")) {
				String temps[] = line.split(": ");
				contentLength = Integer.parseInt(temps[1].trim());
			}
			// (p.101) 요구사항6 - 로그인 여부 확인
			if(line.contains("Cookie")) {
		    isLogined = loginCheck(line);
		    log.debug("isLogined : {}", isLogined);
      }
		}

		// (p.98) 요구사항2 - [Hint]요청 URL과 이름 값을 분리.
		if (url.startsWith("/user/create")) {
			DataOutputStream dos = new DataOutputStream(out);
			int index = url.indexOf("?");
//				log.debug(index+"\n");
//				String requestPath = url.substring(0, index);
			String params = url.substring(index + 1);
//				log.debug(requestPath);
//				log.debug(params);
			String bodyData = IOUtils.readData(br, contentLength);
			log.debug("HTTP Request bodyData : {}", bodyData);

				// (p.98) 요구사항2 - [Hint]데이터를 추출해 User 클래스에 담는다.
				// (p.99) 요구사항3 - [Hint]본문 데이터를 추출해 User 클래스에 담는다.
				// get 전송방식: params, post 전송방식: bodyData
//				Map<String, String> map = util.HttpRequestUtils.parseQueryString(params);
			Map<String, String> map = util.HttpRequestUtils.parseQueryString(bodyData);
			User user = new User(map.get("userId"), map.get("password"), map.get("name"), map.get("email"));
			log.debug("user : {}", user.toString());
				// user : User [userId=id, password=pw, name=na, email=em%40em.com]
//				log.debug("UserId: " + user.getUserId()); // UserId: id
//				log.debug("Password: " + user.getPassword()); // Password: pw
//				log.debug("Name: " + user.getName()); // Name: na
//				log.debug("Email: " + user.getEmail()); // Email: em@em.com
				// (p.100) 요구사항5 - [Hint]2단계: 회원가입 때 생성한 User 객체를 DataBase.addUser() 메소드를 활용해서
				// 저장한다.
			DataBase.addUser(user);
				// (p.99) 요구사항4 - [Hint] HTTP 응답 헤더의 status code를 200 -> 302로 변경
//				url = "/index.html"; // 이 방식으로 처리할 경우 새로고침 시 이전과 똑같은 회원가입 요청이 발생하는 문제.
			response302Header(dos, "/index.html");
		} else if ("/user/login".equals(url)) {
	  		String bodyData = IOUtils.readData(br, contentLength);
				log.debug("HTTP Request bodyData : {}", bodyData);
				Map<String, String> queryParams = util.HttpRequestUtils.parseQueryString(bodyData);
				User user = DataBase.findUserById(queryParams.get("userId"));
				if (user == null) {
					log.debug("Login Null Fail !");
				    responseLoginFailed(out);
				    return;
				}
				if (user.getPassword().equals(queryParams.get("password"))) {
				    log.debug("Login Success !");
				    DataOutputStream dos = new DataOutputStream(out);
				    response302HeaderLoginSuccess(dos);
				} else {
					log.debug("Login Fail !");
				    responseLoginFailed(out);
				    return;
				}
			}
			// (p.101) 요구사항6 - 사용자 목록 출력
			else if("/user/list".equals(url)) {
				if(!isLogined) {
					log.debug("Not Logined ! ");
					responseResource(out, "/user/login.html");
				    return;
				}
				log.debug("User Logined ! ");

				Collection<User> userList = DataBase.findAll();

				StringBuilder sb = new StringBuilder();

				sb.append("<table border='1'>");
				for(User u : userList) {
				    sb.append("<tr>");
				    sb.append("<td>" + u.getUserId() + "</td>");
				    sb.append("<td>" + u.getName() + "</td>");
				    sb.append("<td>" + u.getEmail() + "</td>");
				    sb.append("</tr>");
				}
				sb.append("</table>");

				DataOutputStream dos = new DataOutputStream(out);
				byte[] body = sb.toString().getBytes();

				response200Header(dos, body.length);
				responseBody(dos, body);
			}
			else if(tokens[1].endsWith(".css")) {

			    DataOutputStream dos = new DataOutputStream(out);
			    byte[] body = Files.readAllBytes(new File("./webapp" + tokens[1]).toPath());

			    response200HeaderCss(dos, body.length);
			    responseBody(dos, body);
			}
			else {
				// (p.96) 요구사항1 - [Hint]3단계: 요청 URL에 해당하는 파일을 webapp 디렉토리에서 읽어 전달한다.
//				DataOutputStream dos = new DataOutputStream(out);
//				byte[] body = Files.readAllBytes(new File("./webapp" + url).toPath());
//				response200Header(dos, body.length);
//				responseBody(dos, body);
				responseResource(out, url);
			}
		} catch (IOException e) {
			log.error(e.getMessage());
		}
	}

	private void responseResource(OutputStream out, String url) throws IOException{
		DataOutputStream dos = new DataOutputStream(out);
		byte[] body = Files.readAllBytes(new File("./webapp" + url).toPath());
		response200Header(dos, body.length);
		responseBody(dos, body);
	}

	private void response200Header(DataOutputStream dos, int lengthOfBodyContent) {
		try {
//			log.debug("200실행");
			dos.writeBytes("HTTP/1.1 200 OK \r\n");
			dos.writeBytes("Content-Type: text/html;charset=utf-8\r\n");
			dos.writeBytes("Content-Length: " + lengthOfBodyContent + "\r\n");
			dos.writeBytes("\r\n");
		} catch (IOException e) {
			log.error(e.getMessage());
		}
	}

	private void response302Header(DataOutputStream dos, String url) {
		try {
			log.debug("302실행");
			dos.writeBytes("HTTP/1.1 302 Found \r\n"); // \r\n : http 표준 줄바꿈 :CRLF
			dos.writeBytes("Location: " + url + "\r\n");
			dos.writeBytes("\r\n");
		} catch (IOException e) {
			log.error(e.getMessage());
		}
	}

	private void response302HeaderLoginSuccess(DataOutputStream dos) {
	  try {
	   	// (p.100) 요구사항5 - [Hint]1단계 : 로그인 성공시 HTTP 응답 헤더에 Set-Cookie를 추가해 로그인 성공 여부를 전달한다.
	  	log.debug("302 로그인 성공");
      dos.writeBytes("HTTP/1.1 302 Found \r\n");
	    dos.writeBytes("Set-Cookie: logined=true; Path=/ \r\n");
	    dos.writeBytes("Location: /index.html \r\n");

	    dos.writeBytes("\r\n");
	  } catch (IOException e) {
      log.error(e.getMessage());
	  }
	}

	private void responseBody(DataOutputStream dos, byte[] body) {
		try {
//			log.debug("body 실행");
			dos.write(body, 0, body.length);
			dos.flush();
		} catch (IOException e) {
			log.error(e.getMessage());
		}
	}

	private void responseLoginFailed(OutputStream out) {
	  try {
	    log.debug("로그인 실패");
	    DataOutputStream dos = new DataOutputStream(out);
	    byte[] body = Files.readAllBytes(new File("./webapp" + "/user/login_failed.html").toPath());
	    int lengthOfBodyContent = body.length;
	    dos.writeBytes("HTTP/1.1 200 OK \r\n");
	    dos.writeBytes("Content-Type: text/html;charset=utf-8\r\n");
	    // (p.100) 요구사항5 - [Hint]2단계: 아이디와 비밀번호가 같은지를 확인해
	    // 로그인이 성공하면 응답 헤더의 Set-Cookie 값을 logined=true,
	    // 로그인이 실패하면 응답 헤더의 Set-Cookie 값을 logined=false 로 설정한다.
	    dos.writeBytes("Set-Cookie: logined=false\r\n");
	    dos.writeBytes("Content-Length: " + lengthOfBodyContent + "\r\n");
	    dos.writeBytes("\r\n");
	    responseBody(dos, body);
	    } catch (IOException e) {
	      log.error(e.getMessage());
	    }
	}

	private boolean loginCheck(String line) {
		String temps[] = line.split(":");
		Map<String, String> cookies = util.HttpRequestUtils.parseCookies(temps[1].trim());
		String valueCookie = cookies.get("logined");
		log.debug("valueCookie: {}", valueCookie);

		if(valueCookie == null) return false;
		return Boolean.parseBoolean(valueCookie);
	}

	private void response200HeaderCss(DataOutputStream dos, int lengthOfBodyContent) {
	  try {
	    dos.writeBytes("HTTP/1.1 200 OK \r\n");
	    dos.writeBytes("Content-Type: text/css\r\n");
	    dos.writeBytes("Content-Length: " + lengthOfBodyContent + "\r\n");
	    dos.writeBytes("\r\n");
	  } catch (IOException e) {
	    log.error(e.getMessage());
	  }
	}
}

```

---

## **Logger**

### 1. System.out.println()

- 자바를 처음 학습할 때 콘솔에 값을 출력하기 위한 용도로 사용하는 API가 System.out.println()이다.
- 하지만 System.out.println()으로 메시지를 출력하는 방법은 파일로 메시지가 출력하게 되기 때문에 상당한 비용이 발생하여 애플리케이션 성능을 저하시키는 원인이 된다.
- 이와 같이 성능에 문제점이 있기 때문에 애플리케이션을 배포하기 전에 소스코드에 포함되어 있는 System.out.println()으로 구현한 코드를 삭제하거나 주석 처리하는 방법으로 해결하는 경우도 있었다. 하지만 이 또한 모두 비용이며, 디버깅을 목적으로 메시지를 출력하고 싶으면 또 다시 원복해야 하는 문제점이 있다.

### 2. logging 라이브러리

- 앞서 설명한 System.out.println() 문제점을 보완하기 위해 등장한 라이브러리가 logging 라이브러리다.
- 과거에는 Log4J 라이브러리를 사용했지만, 최근에는 더 좋은 성능을 자랑하는 Logback을 사용할 것을 추천한다.
- 자바 진영에는 많은 로깅 라이브러리 구현체가 존재하는데, 더 좋은 구현체가 등장할 때마다 로깅 라이브러리 구현 부분을 수정하는 어려움이 있다.
- 이 같은 어려움을 해소하기 위해 SLF4J라는 라이브러리를 활용해 로깅 API에 대한 창구를 일원화 했다.

```xml
<dependencies>
  <!-- logger -->
	<dependency>
		<groupId>ch.qos.logback</groupId>
		<artifactId>logback-classic</artifactId>
		<version>1.1.2</version>
	</dependency>
</dependencies>
```

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private static final Logger log = LoggerFactory.getLogger(RequestHandler.class);

```

### 로그 레벨 설정

- 자바 진영의 로깅 라이브러리는 메시지 출력 여부를 로그 레벨을 통해 관리한다.

  - 로그 레벨 : TRACE < DEBUG < INFO < WARN < ERROR 순으로 높아진다.
  - 로그 레벨이 높아질 수록 출력되는 메시지는 적어지고, 로그 레벨이 낮을 수록 더 많은 로깅 레벨이 출력된다.
  - 로그 레벨 설정은 logback.xml 파일에서 설정 가능하다.

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE configuration>
  <configuration>
  	<appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
  		<layout class="ch.qos.logback.classic.PatternLayout">
  			<Pattern>%d{HH:mm:ss.SSS} [%-5level] [%thread] [%logger{36}] - %m%n</Pattern>
  		</layout>
  	</appender>

  	<root level="DEBUG">
  		<appender-ref ref="STDOUT" />
  	</root>
  </configuration>
  ```

---
