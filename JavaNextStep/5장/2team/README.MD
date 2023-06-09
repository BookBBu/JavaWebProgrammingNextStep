# 🐢 5장 웹 서버 리팩토링, 서블릿 컨테이너와 서블릿의 관계

> #### 저자가 작성한 브랜치의 최종 파일로 진행 했습니다. 이번 챕터에선 리팩토링을 메인으로 다뤄봅시다.
- - -

## 5.1 HTTP 웹서버 리팩토링 실습

### 5.1.1 리팩토링할 부분 찾기

> 냄새나는 코드를 찾아내 수정하자!!

### 5.1.2 리팩토링 1단계 힌트

#### 5.1.2.1 요청 데이터를 처리하는 로직을 별도의 클래스로 분리한다(HttpRequest)

> 1. 클라이언트 요청 데이터를 담고 있는 InputStream을 생성자로 받아 HTTP 메소드, URL, 헤더, 본문을 분리하는 작업을 한다.
> 2. 헤더는 Map<String, String>에 저장해 관리하고 getHeader("필드 이름") 메소드를 통해 접근 가능하도록 구현한다.
> 3. GET과 POST 메소드에 따라 전달되는 인자를 Map<String, String>에 저장해 관리하고 getParameter("인자 이름") 메소드를 통해 접근 가능하도록 구현한다.

--> 생성자 단계에서 모든 Http Request를 읽어들이고, GET방식과 POST방식에서 우리가 취해야할 차이점은 Parameter를 어떤 위치에서 받는가이다.  
private String path, method; private String Map<String, String> hedaerMap, parameterMap; 을 이용해서 request를 읽는 과정에서 미리 모든
정보를 기록했다가  
get~~ 메서드 요청이 들어오면 그 값을 리턴해주기만 하면 된다.

#### 내가 구현한 코드

```java
public class HttpRequest {

    private String method, path;
    private Map<String, String> headerMap = new HashMap<>();
    private Map<String, String> parameterMap = new HashMap<>();

    public HttpRequest(InputStream in) {
        try {
            BufferedReader br = new BufferedReader(new InputStreamReader(in, "UTF-8"));
            String line = br.readLine();
            method = line.split(" ")[0];
            if (method.equals("GET")) {
                String[] url = line.split(" ")[1].split("[?]");
                path = url[0];
                String[] parameterSplit = url[1].split("&");
                for (String param : parameterSplit) {
                    String key = param.split("=")[0];
                    String value = param.split("=")[1];
                    parameterMap.put(key, value);
                }
            }
            if (method.equals("POST")) {
                path = line.split(" ")[1];
            }
            while (line != null && !line.equals("")) {
                if (line.contains(":")) {
                    String[] s = line.split(":");
                    headerMap.put(s[0].trim(), s[1].trim());
                }
                line = br.readLine();
            }

            if (method.equals("POST")) {
                line = br.readLine();
                String[] parameterSplit = line.split("&");
                for (String param : parameterSplit) {
                    String key = param.split("=")[0];
                    String value = param.split("=")[1];
                    parameterMap.put(key, value);
                }

            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public String getMethod() {
        return method;
    }

    public String getPath() {
        return path;
    }

    public String getHeader(String header) {
        return headerMap.get(header);
    }

    public String getParameter(String parameter) {
        return parameterMap.get(parameter);
    }
}
```

이상 없이 모든 테스트 케이스를 통과한다.
- - - 

#### 5.1.2.2 응답 데이터를 처리하는 로직을 별도의 클래스로 분리한다(HttpResponse)

> 1. RequestHandler 클래스를 보면 응답 데이터 처리를 위한 많은 중복이 있다. 이 중복을 제거해 본다.
> 2. 응답 헤더 정보를 Map<String, String>으로 관리한다.
> 3. 응답을 보낼 때 HTML, CSS, 자바스크립트 파일을 직접 읽어 응답으로 보내는 메소드는 forward(), 다른 URL로 리다이렉트하는 메소드는 sendRedirect() 메소드를 나누어 구현한다.

--> 기존 RequestHandler의 많은 부분을 차지하던 코드는 바로 응답과 관련한 코드였다. 200응답이냐 302응답이냐에 따라 메서드를 달리했고,
로그인 실패, 성공에 따라 분리되는 부분도 있었다. 이런 부분을 따로 클래스로 빼내서 리팩토링해보자.  
이 부분은 전혀 감이 잡히지 않아서 직접 하진 못하고 코드를 따라쳤다.

```java
package http;

import java.io.DataOutputStream;
import java.io.File;
import java.io.IOException;
import java.io.OutputStream;
import java.nio.file.Files;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HttpResponse {
    private static final Logger log = LoggerFactory.getLogger(HttpResponse.class);

    private DataOutputStream dos = null;

    private Map<String, String> headers = new HashMap<String, String>();

    public HttpResponse(OutputStream out) {
        dos = new DataOutputStream(out);
    }

    public void addHeader(String key, String value) {
        headers.put(key, value);
    }

    public void forward(String url) {
        try {
            byte[] body = Files.readAllBytes(new File("./webapp" + url).toPath());
            if (url.endsWith(".css")) {
                headers.put("Content-Type", "text/css");
            } else if (url.endsWith(".js")) {
                headers.put("Content-Type", "application/javascript");
            } else {
                headers.put("Content-Type", "text/html;charset=utf-8");
            }
            headers.put("Content-Length", body.length + "");
            response200Header(body.length);
            responseBody(body);
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }

    public void forwardBody(String body) {
        byte[] contents = body.getBytes();
        headers.put("Content-Type", "text/html;charset=utf-8");
        headers.put("Content-Length", contents.length + "");
        response200Header(contents.length);
        responseBody(contents);
    }

    private void response200Header(int lengthOfBodyContent) {
        try {
            dos.writeBytes("HTTP/1.1 200 OK \r\n");
            processHeaders();
            dos.writeBytes("\r\n");
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }

    private void responseBody(byte[] body) {
        try {
            dos.write(body, 0, body.length);
            dos.writeBytes("\r\n");
            dos.flush();
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }

    public void sendRedirect(String redirectUrl) {
        try {
            dos.writeBytes("HTTP/1.1 302 Found \r\n");
            processHeaders();
            dos.writeBytes("Location: " + redirectUrl + " \r\n");
            dos.writeBytes("\r\n");
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }

    private void processHeaders() {
        try {
            Set<String> keys = headers.keySet();
            for (String key : keys) {
                dos.writeBytes(key + ": " + headers.get(key) + " \r\n");
            }
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }
}

```

forward, forwardBody에서 응답의 틀을 만들고 response~~와 sendRedriect에서 응답을 보낸다.
- - -

#### 5.1.2.3 다형성을 활용해 클라이언트 요청 URL에 대한 분기 처리를 제거한다.

RequestHandler 클래스를 보면 기능이 추가될 때마다 분기분 (if/else)이 하나씩 추가되기때문에 이런 분기문을 제거하기 위해 다형성을 활용한다.
> 1. 각 요청과 응답에 대한 처리를 담당하는 부분을 추상화해 인터페이스로 만든다.
> 2. 각 분기문을 Controller 인터페이스를 구현하는 클래스를 만들어 분리한다.
> 3. 이렇게 생성한 Controller 구현체를 Map<String, Controller>에 저장한다. Map의 key에 해당하는 String은 요청 URL, value에 해당하는 Controller는
     Controller 구현체이다.
> 4. 클라이언트 요청 URL에 해당하는 Controller를 찾아 service() 메소드를 호출한다.
> 5. Controller 인터페이스를 구현하는 AbstractController 추상 클래스를 추가해 중복을 제거하고, service() 메소드에서 GET과 POST HTTP 메소드에 따라 doGet(),
     doPost() 메소드를 호출하도록 한다.

--> 이 부분도 혼자는 못하겠어서 따라쳤다. 파일이 너무 많아서 코드는 생략

## 5.2 웹 서버 리팩토링 구현 및 설명

4장에서 구현한 HTTP 웹 서버 코드의 핵심은 RequestHandler 클래스이다. RequestHandler 클래스는 클라이언트 요청에 대한 헤더와 본문 데이터 처리, 클라이언트
요청에 따른 로직 처리(회원가입, 로그인 등), 로직 처리 완료 후 클라이언트에 대한 응답 헤더와 본문 데이터 처리 작업으로 나뉜다.
클래스 하나가 너무 많은 일을 하므로 각 객체가 한가지 책임을 가지도록 설계를 개선하는 리팩토링을 진행한다.

### 최종 구현 코드(RequestHandler)

```java
package webserver;

import http.HttpRequest;
import http.HttpResponse;

import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import controller.Controller;

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
            HttpRequest request = new HttpRequest(in);
            HttpResponse response = new HttpResponse(out);

            Controller controller = RequestMapping.getController(request.getPath());
            if (controller == null) {
                String path = getDefaultPath(request.getPath());
                response.forward(path);
            } else {
                controller.service(request, response);
            }
        } catch (IOException e) {
            log.error(e.getMessage());
        }
    }

    private String getDefaultPath(String path) {
        if (path.equals("/")) {
            return "/index.html";
        }
        return path;
    }
}
```

## 요청 흐름 :

1. 웹페이지가 변경되면서 새로운 HttpRequest가 발생하면 먼저 RequestHandler에서 HttpRequest 객체를 생성한다. 동시에 response객체도
   생성해 응답을 대기한다.
2. HttpRequest와 HttpResponse 클래스에서는 in과 out 값에 따라 해야할 일을 수행한다(header, body값 분석, 응답 뼈대 만들기)
3. HttpRequest의 url을 토대로 RequestMapping에서 해당하는 url을 찾아 맞는 다형성으로 Controller를 찾아준다.
4. Controller 내부에서는 url의 요청이 get이냐 post냐에 따라 맞는 로직을 실행해준다.(여기서 응답값이 마무리된다.)

mvc 패턴에서 model부분과 controller부분이 RequestHandler와 Controller에 모두 포함되어있고, view 부분은 html파일로 진행된다.  
이 때 중요한 점은, HttpRequest와 HttpResponse 부분은 개발자가 중요하게 신경쓰지 않아야 할 부분이지만, 현재 코드는 이 부분을 구현하는 데에 많은 시간을 투자해야 한다.

### 5.2.4 HTTP 웹 서버의 문제점

1. HTTP 요청과 응답 헤더, 본문 처리와 같은 데 시간을 투자함으로써 정작 중요한 로직을 구현하는 데 투자할 시간이 상대적으로 적다.
2. 동적인 HTML을 지원하는 데 한계가 있다. 동적으로 HTML을 생성할 수 있지만 많은 코딩량을 필요로 한다.
3. 사용자가 입력한 데이터가 서버를 재시작하면 사라진다. 사용자가 입력한 데이터를 유지하고 싶다.

### 내가 생각하는 각 문제를 해결하는 방법

1. HTTP 요청을 처리하는 데 필요한 노동은 지금으로썬 딱히 방법이 없다. 서블릿이나 스프링같은 것을 사용해 이미 만들어진 것을 사용하는  
   게 좋을 것같다. 만약 직접 구현하는 것을 목표로 한다면 지금처럼 계속 해야할 듯 하다.
2. 동적 HTML을 구현하기 위해서는 JSP같은 스크립트용 뭔가가 필요하다. 만약 직접 구현하는 것을 목표로 한다면 지금처럼 계속 해야할 듯 하다.
3. 데이터가 서버가 켜진동안만 유지되는 문제는 어떤 데이터베이스를 연결해야할 듯 하다. 하지만 데이터베이스를 연결하는 로직을
   직접 짤 수 있을까? 아니라고 본다. 역시 jdbc, mybatis, jpa등이 필요할 듯하다.

## 5.3 서블릿 컨테이너, 서블릿/JSP를 활용한 문제 해결

서블릿은 앞서 구현한 웹 서버의 Controller, HttpRequest, HttpResponse를 추상화 해 인터페이스로 정의해 놓은 표준이다.
즉, HTTP의 클라이언트 요청과 응답에 대한 표준을 정해 놓은 것을 서블릿이라 생각하면 된다. 서블릿 컨테이너는 이 서블릿 표준에
대한 구현을 담당하고 있으며 앞에서 구현한 웹 서버가 서블릿 컨테이너 역할과 같다고 생각하면 된다.
> 서블릿은 라이프사이클을 잘 알아야한다. 서블릿 라이프사이클을 알고 있다면 개발단계에 대한 더 깊은 이해로 개발 실력이 늘어날 것이다.
> 1. 서블릿 컨테이너 시작
> 2. 클래스패스에 있는 Servlet 인터페이스를 구현하는 서블릿 클래스를 찾음
> 3. @WebServlet 설정을 통해 요청 URL과 서블릿 매핑
> 4. 서블릿 인스턴스 생성
> 5. init() 메소드를 호출해 초기화
> 6. 이후 대기상태로 있다가 클라이언트 요청이 있을 경우 요청 URL에 해당하는 서블릿을 찾아 service() 메소드를 호출해 소멸 작업 진행
> 7. 서비스를 하다가 서블릿 컨테이너를 종료하면 서블릿 컨테이너가 관리하고 있는 모든 서블릿의 destroy() 메소드를 호출해 소멸 작업을 진행

### 5.4.1 객체지향 설계와 개발

객체 지향 프로그래밍이란?  
우리가 실생활에서 쓰는 모든 것을 객체라 하며, 객체 지향 프로그래밍은 프로그램 구현에 필요한 객체를 파악하고 각각의 객체들의 역할이 무엇인지를
정의하여 객체들 간의 상호작용을 통해 프로그램을 만드는 것을 말한다. 객체는 클래스라는 틀에서 생겨난 실체(instance)이다. 따라서 객체 지향 프로그램은 객체와
객체 간의 연결로 되어 있으며 각각의 객체 안에 자료구조와 알고리즘이 들어있는 것이다.

# 객체지향 프로그래밍의 특징

1. 추상화
    - 객체들의 공통적인 특징을 도출
    - 객체지향적 관점에서는 클래스를 정의하는 것을 추상화라 함
2. 캡슐화
    - 실제로 구현되는 부분을 외부에 드러나지 않도록해 정보를 은닉
    - 객체가 독립적으로 역할을 할 수 있도록 데이터와 기능을 하나로 묶어 관리
    - 코드가 묶여있어서 오류가 없어 편리함
    - 데이터를 보이지 않고 외부와 상호작용을 할 때는 메소드를 이용하여 통신
3. 상속성
    - 하나의 클래스가 가진 특징을 다른 클래스가 그대로 물려받는 것
    - 이미 작성된 클래스를 받아서 새로운 클래스를 생성하는 것
    - 기존 코드를 재활용해서 사용함으로써 객체지향 방법의 중요한 기능 중 하나에 속한다.
4. 다형성
    - 약간 다른 방법으로 동작하는 함수를 동일한 이름으로 호출하는 것
    - 동일한 명령의 해석을 연결된 객체에 의존하는 것
    - 오버라이딩, 오버로딩
        - 오버라이딩 - 부모클래스의 메소드와 같은 이름을 사용하며 매개변수도 같지만 내부 소스를 재정의하는 것
        - 오버로딩 - 같은 이름의 함수를 여러 개 정의한 후 매개변수를 다르게 하여 같은 이름을 경우에 따라 호출하여 사용하는 것
5. 동적 바인딩
    - 가상 함수를 호출하는 코드를 컴파일할 때, 바인딩을 실행시간에 결정하는 것
    - 파생 클래스의 객체에 대해, 기본 클래스의 포인터로 가상 함수가 호출될 때 일어난다.
    - 함수를 호출하면 동적 바인딩을 통해 파생 클래스에 오버라이딩 된 함수가 실행
    - 프로그래밍의 유연성을 높여주며 파생 클래스에서 재정의한 함수의 호출을 보장

### 우리가 이번 챕터에서 설계한 웹서버는 어떤 특징을 잘 보이고 어떤 특징은 잘 드러나지 않을까?

# 객체지향 프로그래밍의 5대 설계 원칙 SOLID
[출처 : 티스토리 '망나니개발자'](https://mangkyu.tistory.com/194)
## SOLID란?
SOLID란 객체 지향 프로그래밍을 하면서 지켜야하는 5대 원칙으로 각각 SRP, OCP, LSP,
DIP, ISP의 앞글자를 따서 만들어졌다. SOLID 원칙을 철저히 지키면 시간이 지나도 변경이
용이하고, 유지보수와 확장이 쉬운 소프트웨어를 개발하는데 도움이 되는것으로 알려져있다.
### 1. 단일 책임의 원칙(SRP, Single Responseibility Principle)
모듈이 변경되는 이유가 한가지여야 함으로 받아들여야 한다. 여기서 변경의 이유가 한가지라는
것은 해당 모듈이 여러 대상 또는 액터들에 대해 책임을 가져서는 안되고, 오직 하나의
액터에 대해서만 책임을 져야 한다는 것을 의미한다.
### 2. 개방 폐쇄 원칙(OCP, Open-Closed Principle)
개방 폐쇄 원칙은 확장에 대해 열려있고 수정에 대해서는 닫혀있어야 한다는 원칙으로,
각각이 갖는 의미는 다음과 같다.
- 확장에 대해 열려있다: 요구사항이 변경될 때 새로운 동작을 추가하여 애플리케이션의 기능을 확장할 수 있다.
- 수정에 대해 닫혀있다: 기존의 코드를 수정하지 않고 애플리케이션의 동작을 추가하거나 변경할 수 있다.
OCP가 본질적으로 얘기하는 것은 추상화이다.
### 3. 리스코프 치환 원칙(LSP, Liskov Substitution Principle)
하위 타입은 상위 타입을 대체할 수 있어야 한다는 것이다. 즉, 해당 객체를 사용하는 클라이언트는
상위 타입이 하위 타입으로 변경되어도, 차이점을 인식하지 못한 채 상위 타입의 퍼블릭 인터페이스를 통해
서브 클래스를 사용할 수 있어야 한다는 것이다.
### 4. 인터페이스 분리 원칙(ISP, Interface Segregation Principle)
객체가 충분히 높은 응집도의 작은 단위로 설계됐더라도, 목적과 관심이 각기 다른 클라이언트가
있다면 인터페이스를 통해 적절하게 분리해줄 필요가 있는데, 이를 인터페이스 분리 원칙이라고
부른다. 즉, 인터페이스 분리 원칙이란 클라이언트의 목적과 용도에 적합한 인터페이스만을
제공하는 것이다. 인터페이스 분리 원칙을 준수함으로써 모든 클라이언트가 자신의 관심에
맞는 퍼블릭 인터페이스(외부에서 접근 가능한 메시지)만을 접근해 불필요한 간섭을 최소화할
수 있으며, 기존 클라이언트에 영향을 주지 않은 채로 유연하게 객체의 기능을 확장하거나 수정할 수 있다.
### 5. 의존 역전 원칙(DIP, Dependency Inversion Principle)
고수준 묘둘은 저수준 묘둘의 구현에 의존해서는 안 되며, 저수준 모듈이 고수준 모듈에 의존해야 한다는 것이다.
객체지향 프로그래밍에서는 객체들 사이에 메시지를 주고 받기 위해 의존석이 생기는데,
의존성 역전의 원칙은 올바른 의존 관계를 위한 원칙에 해당된다. 여기서 각각 고수준 모듈과
저수준 모듈이란 다음을 의미한다.
- 고수준 모듈: 입력과 출력으로부터 먼(비지니스와 관련된) 추상화된 모듈
- 저수준 모듈: 입력과 출력으로부터 가까운(HTTP, 데이터베이스, 캐시 등과 관련된) 구현 모듈

의존 역전 원칙이란 결국 비즈니스와 관련된 부분이 세부 사항에는 의존하지 않는 설계 원칙을 의미한다.

> SOLID의 핵심은 결국 추상화와 다형성이다. 구체 클래스에 의존하지 않고 추상 클래스에
> 의존함으로써 우리는 유연하고 확장가능한 애플리케이션을 만들 수 있는 것이다.
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
