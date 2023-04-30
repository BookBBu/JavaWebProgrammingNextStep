# 3장_ 개발 환경 구축 및 웹 서버 실습 요구사항

## **개요**
- 로컬에서 웹 어플리케이션을 개발, 버전 관리 시스템에서 소스코드를 관리, 개발한 웹 어플리케이션을 원격 서버에 실제 배포하는 경험을 하고 12장까지 구현해야 할 질문/답변 게시판에 대한 서비스 요구사항과 실습 요구사항을 제시하고 있다.   

- 또한, HTTP 웹 서버를 직접 구현하며 웹 클라이언트와 서버 간에 데이터를 어떻게 주고 받는지에 대해 학습한다.

- 3장 초반에서 제시하는 원격 서버 배포는 각자 진행하는 것을 권장함.
  * AWS EC2 사용 (우분투 20.04)
  * SSH 프로그램 : XShell
  * JAVA 설치 (https://i5i5.tistory.com/266)
    ```
    $ sudo apt-get update 
    $ sudo apt-get install openjdk-8-jdk
    ```
    ```
    readlink -f $(which java)
    /usr/lib/jvm/java-8-openjdk-amd64/bin/java
    ```

    ```
    sudo vi /etc/profile​
    ```
    `i` 눌러서 수정모드, 저장은 ESC 후, `:wp` 도중에 종료하고싶으면 `:q!`

    ```
    export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
    export PATH=$PATH:$JAVA_HOME/bin
    ```

    ```
    source /etc/profile
    ```

    ```
    $ echo $JAVA_HOME
    /usr/lib/jvm/java-8-openjdk-amd64

    $ echo $PATH | grep java
    ......:/usr/lib/jvm/java-8-openjdk-amd64/bin
    ```
  * maven 설치 (https://jjeongil.tistory.com/1995)
  * EC2 외부 접속을 위한 인바운드 설정

* * *   

## **요구사항 및 구현**
- 3장에서의 실습 요구사항은 아래와 같다.
  * [index.html 응답하기](#요구사항-1)
  * [GET 방식으로 회원가입하기](#요구사항-2)
  * [POST 방식으로 회원가입하기](#요구사항-3)
  * [302 status code 적용](#요구사항-4)
  * [로그인하기](#요구사항-5)
  * [로그인 여부에 따른 사용자 목록 출력](#요구사항-6)
  * [CSS 지원하기](#요구사항-7)
  * [Logger](#logger)
  * [SLF4J 효율적인 사용법](#slf4j-효율적인-사용법)   


### **요구사항 구현 전 제공 소스코드 분석**
요구사항을 구현하기 전 간단하게 소스코드를 분석하자면 WebServer 클래스의 Main method 에 의해 서버 소켓을 생성하고 RequestHandler 클래스로 사용자 요청에 대한 처리와 응답에 대한 처리를 하고있다.

이번 실습에선 RequestHandler 클래스의 run() 메서드 안에서 구현하도록 한다. 필요한 기능을 구현하고 리팩토링 전 후를 비교하고자 한다.


### **요구사항 1 : index.html 응답하기**

- 요구사항 구현   
  * 브라우저에서 `localhost:8080/index.html` 로 접속 시, InputStream을 통해 HttpRequest 값이 들어옴.   
들어오는 값은 <img src=./1-2.png> <br>
개발자 도구의 Network 탭에서 `Request Header`의 값을 확인할 수 있다.   
Header 값이 InputStream 을 통해 들어오는데 이 InputStream을 가져오기 위해 BufferedReader를 이용해 HTTP 요청 정보를 가져올 수 있다.   
`BufferedReader br = new BufferedReader(new InputStreamReader(in, "UTF-8"));` (UTF-8로 받아오기 위해 decoding)   
첫번째 라인에는 요청 라인값이며 이 상황에서는 `GET /index.html HTTP/1.1` 이다.

   
  * 이 값을 이용하여 /index.html 을 추출할 수 있다.   
    ```Java
    BufferedReader br = new BufferedReader(new InputStreamReader(in, "UTF-8"));

    String lineString = br.readLine();
    log.debug("HTTP Header : {}", lineString);
    // HTTP Header : GET /index.html HTTP/1.1
    ```   
    여기서, `/index.html` 을 추출하기 위해 split 메서드를 이용한다.
    `String tokens[] = lineString.split(" ");`   
    `tokens[1] : "/index.html" ` 가 들어있게 됨.   


  * 또한, HTTP 요청에서는 Header 와 Body 의 구분은 빈줄로 구분하게 된다. 이를 이용해 Header 에 관한 정보를 가져올 수 있다.
    ``` java
    while (true) {
        lineString = br.readLine();
        if (lineString == null || lineString.equals("")) {
            break;
        }
        log.debug("HTTP Request Header : {}", lineString);
    }
    ```

  * 요청 URL에 맞는 곳으로 이동시키기 위해 
    ```java
    DataOutputStream dos = new DataOutputStream(out);
    byte[] body = Files.readAllBytes(new File("./webapp" + tokens[1]).toPath());
    // File 클래스의 File 객체를 Path객체로 변환시키는 toPath() 메서드 사용
    // readAllBytes는 Parameter 타입을 Path로 받는다.
    // 파일 시스템에서 파일의 위치를 가진 Path 객체를 얻어내기 위해 사용한다.
    // System.out.println(new File("./webapp" + tokens[1]).toString());
    // .\webapp\index.html
    
    response200Header(dos, body.length);
    responseBody(dos, body); 
    ```
    파일의 내용(HTML)을 수신하고, 이 코드를 사용해 페이지를 렌더링



* * *
### **요구사항 2 : GET 방식으로 회원가입하기**
- 요구사항 구현
  * 우선, 회원가입 클릭 시,    <img src=./2_1.png>   
  콘솔 창에 위와 같은 값들이 출력되며, 여기서 여겨봐야할 것은 HTTP Header 요청 라인이다. `GET /user/create?userId=...(생략)`    
  [요구사항 1] 에서 구현하였 듯이, tokens[1] 의 값은 위 요청 라인의 `/user/create?userId=...(생략)` 이 부분인데, 
  구분을 하자면 `/user/create?`, `userId=aa&password=aa...`으로 구분할 수 있다.   
  즉, `/user/create?` 로 시작하는 점을 이용해 회원가입에 대한 작업을 할 수 있다. 
    ``` java
    if (tokens[1].startsWith("/user/create?")) {
        ...
    } // startsWith("str") : str로 시작하는 경우 True
    ```

    추가로, 요청 쿼리에 대해 값을 가져오기 위해 구현된 메서드인 `parseQueryString`을 이용한다.
    ```java
    Map<String, String> queryParams = HttpRequestUtils.parseQueryString(tokens[1].substring("/user/create?".length()));
    // &를 기준으로 나누고, = 를 기준으로 key-value 값을 가진다.
    ```
  * 문제에서 요구하는 사항을 구현하면 아래와 같다.
    ```java
    if (tokens[1].startsWith("/user/create?")) {
        Map<String, String> queryParams = HttpRequestUtils.parseQueryString(tokens[1].substring("/user/create?".length()));
    
        User user = new User(
            queryParams.get("userId"),
            queryParams.get("password"),
            queryParams.get("name"),
            queryParams.get("email")
        ); 
        log.debug("user : {}", user.toString());
    }
    ```
    <img src=./2_2.png>


* * *
### **요구사항 3 : POST 방식으로 회원가입하기**
- 요구사항 구현   
  * 현재는 GET 방식으로 구현되어 있어, 회원가입시 사용되는 form tag의 method 방식을 변경해야 한다.
    ```HTML 
    <form name="question" method="get" action="/user/create">
    <form name="question" method="post" action="/user/create">
    <!-- "get" 을 "post" 으로 -->
    ```

  * GET 방식과 다른 점은, Header에 회원가입 관련 정보가 담기지 않고 HTTP Body에 정보가 담기는 차이가 있다.  그래서, [요구사항 1]과 유사하게, Header 와 Body를 구분한 후, Body 값을 제어한다. (Header와 Body는 빈줄로 구분하는 점을 이용한다.)

  * Body에 값이 어떤 식으로 들어오는 지 확인하기 위해 log.debug를 사용해 확인한다.   
    ```java
    String bodyData = IOUtils.readData(br, contentLength);
    log.debug("HTTP Request bodyData : {}", bodyData);
    // userId=123&password=12332&name=123&email=123%4022
    ```
    POST 의 길이는 Header의 `Content-Length` 를 통해 확인할 수 있다.   
    <img src=./2_3.png>   
    그래서, Header의 값을 읽어올 때 미리 값을 확인해야한다.
    ```java
    if(lineString.contains("Content-Length")) {
        String temps[] = lineString.split(": "); 
        contentLength = Integer.parseInt(temps[1]);
    }
    ```
  * 앞서 구현한, [요구사항 2]와 파싱할 값이 같음을 확인할 수 있다.
    ```java
    Map<String, String> queryParams = HttpRequestUtils.parseQueryString(bodyData);
    User user = new User(
        queryParams.get("userId"),
        queryParams.get("password"),
        queryParams.get("name"),
        queryParams.get("email")
    );
    log.debug("user : {}", user.toString());
    // user : User [userId=123, password=12332, name=123, email=123%4022]
    DataBase.addUser(user); // 로그인할 때 사용할 코드.
    ```
  * 비밀번호 값이 포함되어 있으니, 그나마 POST 방식이 안전하다고 볼 수 있다.


* * *
   
### **요구사항 4 : 302 status code 적용**
https://developer.mozilla.org/ko/docs/Web/HTTP/Status/302   
https://en.wikipedia.org/wiki/HTTP_302   
301 302의 차이점 : 
- 요구사항 구현 : 회원가입을 완료하면 `/index.html` 페이지 이동
  * HTTP의 302 코드를 사용해서 구현하는데, HTTP Reponse Status Code 302 Found 는 헤더에 지정된 URL로 일시적으로 이동시키는 Redirection 이다.
  Server Reponse 방법은 아래와 같다.
    ```
    HTTP/1.1 302 Found
    Location: http://www.iana.org/domains/example/
    ```
    
    우선, 302Header로 response하는 Method를 만들어 준다.
    ```java
    private void response302Header(DataOutputStream dos, String url) {
        try {
            dos.writeBytes("HTTP/1.1 302 Found \r\n"); // \r\n : http 표준 줄바꿈 :CRLF
            dos.writeBytes("Location: " + url + "\r\n");

        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }
    ```
  * 회원가입 완료 시, 앞서 구현한 `response302Header` 를 통해 index.html 페이지로 이동한다.
    ```java
    DataOutputStream dos = new DataOutputStream(out);
    response302Header(dos, "/index.html");
    ```
  * `writeBytes("..\r\n")` 을 할 때, 캐리지 리턴 후 줄바꿈을 하는 이유는 http 표준 줄바꿈이기 때문이다.(CRLF)
     (그리고, CRLF의 취약점을 이용하는 HTTP Response Splitting 기법이 있다. 입력에 대한 CR, LF 제거 등 유효성 검사를 통해 대응할 수 있음..)

* * *
### **요구사항 5 : 로그인하기**
- 요구사항 구현 
  * 로그인 성공 시, /index.html 로 이동하고 실패할 경우, /user/login_failed.html로 이동한다.
    ```java
    String bodyData = IOUtils.readData(br, contentLength);
    log.debug("HTTP Request bodyData : {}", bodyData);
    Map<String, String> queryParams = HttpRequestUtils.parseQueryString(bodyData);
    ```
    POST 방식으로 들어오므로, userId와 password의 일치여부를 통해 로그인할 수 있다.
    ```java
    User user = DataBase.findUserById(queryParams.get("userId"));
    if (user == null) {;
        responseLoginFailed(out);
        return;
    }
    
    if (user.getPassword().equals(queryParams.get("password"))) {
        log.debug("Login Success !");
        DataOutputStream dos = new DataOutputStream(out);
        response302HeaderLoginSuccess(dos);
    } else {
        responseLoginFailed(out);
        return;
    }
    ```


  * 회원가입한 사용자로 로그인에 성공한다면, Cookie 값을 통해 로그인 상태를 유지할 수 있어야한다. 앞서 구현한 response302Header에 Set-Cookie로 요구하는 쿠키의 값을 넣어준다.
    ```java
    private void response302HeaderLoginSuccess(DataOutputStream dos) {
        try {
            dos.writeBytes("HTTP/1.1 302 Found \r\n");
            dos.writeBytes("Set-Cookie: logined=true\r\n");
            dos.writeBytes("Location: /index.html \r\n");
        
            dos.writeBytes("\r\n");
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }
    ```

  * 반대로 실패할 경우, /user/login_failed.html 로 이동함과 동시에 Set-Cookie: logined=false 를 하면되므로
  response200HeaderLoginFailed 메서드를 구현해주어 사용한다.
    ```java
    private void response200HeaderLoginFailed(DataOutputStream dos, int lengthOfBodyContent) {
        try {
            dos.writeBytes("HTTP/1.1 200 OK \r\n");
            dos.writeBytes("Content-Type: text/html;charset=utf-8\r\n");
            dos.writeBytes("Set-Cookie: logined=false\r\n");
            dos.writeBytes("Content-Length: " + lengthOfBodyContent + "\r\n");
            dos.writeBytes("\r\n");
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }
    ```
    실패할 경우, 아래와 같은 코드를 실행하게 된다. 
    ```java
    DataOutputStream dos = new DataOutputStream(out);
    byte[] body = Files.readAllBytes(new File("./webapp" + "/user/login_failed.html").toPath());

    response200HeaderLoginFailed(dos, body.length);
    responseBody(dos, body);
    ```
    실패하는 경우는 `로그인하려는 ID가 없는 경우`, `password가 일치하지 않는 경우` 이다. 이 두번의 경우에서 중복되어 사용되게 되므로,
    ```java
    private void responseLoginFailed(OutputStream out) {
        try {
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
    `responseLoginFailed(out);` 만을 사용하여 로그인 실패 처리를 하였다.   
  * 쿠키의 값   
    <img src=./2_4.png>

* * *
### **요구사항 6 : 로그인 했을 경우, 목록 출력**
  * 우선 로그인 여부를 Cookie 를 통해 확인해야한다. Cookie 는 헤더에 포함되어 있으므로, Header를 읽을 때 값을 가져오면 된다. 
    ```java
    if(lineString.contains("Cookie")) {
        String temps[] = lineString.split(":");
        Map<String, String> cookies = HttpRequestUtils.parseCookies(temps[1].trim());
        String valueCookie = cookies.get("logined");
        log.debug("valueCookie: {}", valueCookie);
        if(valueCookie == null) {
            isLogined = false;
        } else {     
            isLogined = Boolean.parseBoolean(valueCookie);
        }
        log.debug("isLogined : {}", isLogined);
    }
    ```
    이때, `parseCookies` 는 저자가 구현한 메서드인데 쿠키에서 "logined" 와 "true" 또는 "false" 를 Map을 이용해 파싱하는 메서드이다.

    위 코드도 리팩토링한다면...
    ```java
    private boolean loginCheck(String lineString) {
		String temps[] = lineString.split(":");
		Map<String, String> cookies = HttpRequestUtils.parseCookies(temps[1].trim());
		String valueCookie = cookies.get("logined");
		log.debug("valueCookie: {}", valueCookie);
        
		if(valueCookie == null) return false;
		return Boolean.parseBoolean(valueCookie);		
    }
    ```
    ```java
    isLogined = loginCheck(lineString);
    log.debug("isLogined : {}", isLogined);
    ```
  * isLogined 값을 통해 리스트 출력 여부를 결정하면 된다. 로그인 되어있지 않을 경우, 
    ```java
    if(!isLogined) {
        DataOutputStream dos = new DataOutputStream(out);
        byte[] body = Files.readAllBytes(new File("./webapp" + "/user/login.html").toPath());
        
        response200HeaderLoginFailed(dos, body.length);
        responseBody(dos, body); 
        return;
    }
    ```
    로그인 되어 있을 경우,
    ```java

    log.debug("User Logined ! ");

    Collection<User> userList = DataBase.findAll();
    
    // HTML 동적으로 넣어라고?
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
    HTML 코드를 body에 실어 화면에 뿌리도록 하였다.   
    로그인 여부를 쿠키에 저장하였기 때문에 다시 프로젝트를 종료하고 실행하여도 쿠키에 값이 그대로 남아있다. 개발자 도구로 손수 직접 쿠키를 지워야 로그인 하지 않았을 경우를 확인할 수 있다.




* * *
### **요구사항 7 : CSS 적용**
[DEBUG] [Thread-122] [webserver.RequestHandler] - HTTP Header : GET /css/styles.css HTTP/1.1
- CSS 적용하기 위해서 `text/html` 로 모든 게 보내졌던 것을 `.css` 파일은 따로 Content-Type을 지정하여 응답을 보내야한다. 즉, 이때까진 Header 에서 css 를 따로 처리하지 않았고 각 요구사항에서 구현한 if else {...} 의 마지막 구문에서 Content-Type: text/html 형식으로 body로 보내졌었다.
    ```Java
    DataOutputStream dos = new DataOutputStream(out); 
    byte[] body = Files.readAllBytes(new File("./webapp" + tokens[1]).toPath());  
    response200Header(dos, body.length); // dos.writeBytes("Content-Type: text/html;charset=utf-8\r\n");
    responseBody(dos, body);
    ```

* 그래서, tokens[1]의 마지막이 `.css`로 끝나는 것이 `css` 파일임을 이용하여 따로처리하도록 하였다. 
    ```java
    else if(tokens[1].endsWith(".css")) {

        DataOutputStream dos = new DataOutputStream(out);  
        byte[] body = Files.readAllBytes(new File("./webapp" + tokens[1]).toPath());
        
        response200HeaderCss(dos, body.length);
        responseBody(dos, body);      	
    }
    ```
    `response200HeaderCss`
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
    Content-Type을 text/css 로 보내면 css 적용이 된다.

* * *
지금까지 구현한 상황을 본다면 tokens[1] 로 요청을 확인해 아래와 같은 방식으로 구현하였다. 
```java
value = ...
if (value = ?) {
    ...
} else if (value = ?) {
    ...
} else if (..) {
    ..
} else if (..) {
    ...
} .
  .
  .
  else {

}
```
```java
value = ...
if (value = ?) {
    ...
    return;
}

if (value = ?) {
    ...
    return;
}
```
위와 같은 방법으로 리팩토링을 한다면 조금 더 가독성이 있을 것이다.
또한, Controller 로 doGet, doPost로 분리할 수 있을 것이다.

* * * 
### **Logger**
- **dependency 추가** 
    ``` 
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.1.2</version>
    </dependency>
    ```
- **Log level 설정**   
    로그 레벨은 TRACE < DEBUG < INFO < WARN < ERROR 순서로 되어있다.
    개발 단계에서 로그 레벨을 잘 나누어 구현하게 되면 로그 레벨의 조절만으로도 원하는 로깅을 확인할 수 있다.

    개발 단계에선 주로 DEBUG로 두고, 실서비스에선 INFO 레벨로 둔다.
    (DEBUG 레벨로 실서비스를 운영하면 굳이 노출되지 않아도 될 디버깅 내용들이 출력되므로 다른 로그를 확인하는데 불편함이 존재)

    로그 레벨 설정은 logback.xml 파일에서 할 수 있다.
    ```
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
    `%d{HH:mm:ss.SSS} [%-5level] [%thread] [%logger{36}] - %m%n` : 로그 메시지의 패턴을 설정   
    `<root level="DEBUG">` : 로그 레벨을 설정할 수 있다.

* * * 
### **SLF4J 효율적인 사용법**
http://dveamer.github.io/backend/HowToUseSlf4j.html

요약   
1. `log.debug()` 안에서 문자열 연산을 하면 log.debug() 메서드 시작 전에 문자열 연산을 하므로, 문자열 연산만큼 성능 악화 발생   
    ```java
    logger.debug("Hello " + userName + "."); // X
    ```
2. 문자열 연산을 하고 싶다면 우선 가용 레벨을 확인
    ```java
    if(logger.isDebugEnabled()) {
        logger.debug("Hello " + userName + ".");
    } // 가용 로그 레벨이 INFO 일 경우, 문자열 연산이 되지 않아 절약
    ```
3. 위 방법은 문자열 연산의 가독성 저하 → 치환문자 사용 `{}`
    ```java
    logger.debug("Hello {}.", userName);
    ```
    가용 로그 레벨을 확인해도 괜찮으나 코드 상에서 간결성을 해침.   
    너무 긴 log를 출력하는 경우에 가용 로그 레벨을 확인할 것

4. 파라미터 개수가 3개 이상이면 가용 로그 레벨을 체크하기 전에 Object[] 를 생성하는 비용이 발생함.  
    최대한 파라미터의 개수를 2개 이하로 맞추기 위해 노력해야 하며 클래스는 toString() 메서드를 활용.
    
* * * 
### **Maven과 Gradle의 차이점**
https://dev-coco.tistory.com/65    
요약   
- **Maven**   
Maven의 기능을 이용하기 위해 POM.XML 사용 

  - pom.xml에서 주요하게 다루는 기능   
프로젝트 정보 : 프로젝트의 이름, 라이센스 등   
빌드 설정 : 소스, 리소스, 라이프사이클별 실행한 플로그인 등 빌드와 관련된 설정   
빌드 환경 : 사용자 환경 별로 달라질 수 있는 프로파일 정보   
pom 연관 정보 : 의존 프로젝트(모듈), 상위 프로젝트, 포함하고 있는 하위 모듈 등

- **Gradle**   
빌드 속도가 Maven에 비해 10~100배 가량 빠름   
JAVA, C/C++M Python 등을 지원   
빌트툴인 Ant Builder와 Groovy 스크립트 기반으로 만들어져 기존    Ant의 역할과 배포 스크립트의 기능을 모두 사용 가능   

pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.2</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example2</groupId>
    <artifactId>demo-maven</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo-maven</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>11</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
 
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
 
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
 
</project>
```

gradle
```groovy
plugins {
    id 'org.springframework.boot' version '2.5.2'
    id 'io.spring.dependency-management' version '1.0.11.RELEASE'
    id 'java'
}
 
group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'
 
repositories {
    mavenCentral()
}
 
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
 
test {
    useJUnitPlatform()
}
```
