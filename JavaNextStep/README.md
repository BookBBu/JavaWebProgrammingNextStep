<h1 align="center">Hi 👋, I'm sorryday </h1>
<h3 align="center">A passionate beginner Java and Back-End (and Android) developer from Republic of Korea</h3>

- 🔭 I’m currently working on my **![Java](https://img.shields.io/badge/java-%23ED8B00.svg?style=for-the-badge&logo=java&logoColor=white)   ![Spring](https://img.shields.io/badge/spring-%236DB33F.svg?style=for-the-badge&logo=spring&logoColor=white)   ![Android Studio](https://img.shields.io/badge/Android%20Studio-3DDC84.svg?style=for-the-badge&logo=android-studio&logoColor=white)**

- 👯 I will be a growing developer.

- 📫 How to reach me: **rptqoxmf1@gmail.com OR rptqoxmf5886@naver.com**

<h3 align="center">Connect with me:</h3>
<div align="center">

[![image](https://img.shields.io/badge/Instagram-E4405F?style=for-the-badge&logo=instagram&logoColor=white)](https://www.instagram.com/ssh_seunghyeon/?hl=ko)

</div>


-------------------------------------------------------------
# 3장: 개발 환경 구축 및 웹 서버 십습 요구사항

## 3-0. 서론

- 3장의 전반적인 구성 내용
    1. 질문/답변 게시판에 대한 서비스 요구사항과 실습 요구사항 제시<br>
    2. Web Client와 서버간의 데이터 통신을 위한 HTTP Web Application Server 구축<br>
    3. Local에서 Web Application 개발<br>
    4. 버전 관리 시스템에서 소스 코드 관리<br>
    5. 개발한 Application을 원격 서버에 배포<br>
	<br>
	
## 3-1. 서비스 요구 사항

- 질문/답변 게시판의 요구사항을 사용자의 흐름 순으로 살펴보자
    1. 질문/답변 게시판<br>
        1. Header에서 회원가입, 로그인, 로그아웃, 개인정보 수정이 가능<br>
    2. 회원가입 클릭 시 회원 가입 페이지로 이동<br>
    3. 로그인 버튼을 누르면 로그인 페이지로 이동 및 로그인 가능<br>
    4. 질문 목록 화면에서 질문 클릭 시 각 질문의 상세보기 화면으로 이동<br>
        1. 답변 추가 및 수정/삭제 가능<br>
        
	<br>
## 3-2. 로컬 개발 환경 구축

- 기본적으로 Java 8 Version과 이클립스를 이용하여 구성된다. (물론 변경도 가능)
- 이클립스의 Git Perspective를 이용한 방법
1. https://github.com/slipp/web-application-server 에서 Project를 자신의 Repository에 Fork한다.
    
   <img width="1663" alt="1" src="https://user-images.githubusercontent.com/49806698/235552016-ecd2e486-6035-459a-997f-859e08c55382.png">
	<br>
    
2. 이클립스의 Git Perspective에서 Fork한 Repository를 Git clone 한다.
    <img width="520" alt="2" src="https://user-images.githubusercontent.com/49806698/235552036-a099c762-6dbe-4aad-9aee-dce3153435f0.png">


   
3. clone한 프로젝트를 로컬 폴더에 Import한다.<br>
	<img width="520" alt="3" src="https://user-images.githubusercontent.com/49806698/235552059-653e66ab-15c3-44c5-b38d-6f6e2ef362b5.png">
    


    <br>
4. Import한 프로젝트에서 **WebServer.java**를 Java Application으로 실행한다.
    <img width="1035" alt="4" src="https://user-images.githubusercontent.com/49806698/235552065-03067ae8-7ea6-4eec-bff3-7df27e16cbee.png">

   
    <br>
5. “Hello world”가 잘 출력되는지 확인
    <img width="809" alt="5" src="https://user-images.githubusercontent.com/49806698/235552078-f9dc6692-45a9-404e-8d81-72d9bd0800df.png">

 
    <br>

- 터미널에서 Git Clone한 후 Import 하는 방법
    - [https://www.youtube.com/watch?v=5hjYe_PggJI](https://www.youtube.com/watch?v=5hjYe_PggJI)
    

## 3-3. 원격 서버에 배포

- 참고 유튜브 영상
    1. 한글 인코딩 설정, 자바 8 설치 및 설정, Maven 설치 및 과정
        1. [https://www.youtube.com/watch?v=dWGzApCuF9M](https://www.youtube.com/watch?v=dWGzApCuF9M)
    2. Git 설치, Github 저장소 Clone, 메이븐을 활용한 빌드, HTTP 웹 서버 시작, 소스 코드 수정 시 재배포 과정
        1. [https://www.youtube.com/watch?v=N8iLAuAo-Qw](https://www.youtube.com/watch?v=N8iLAuAo-Qw)

### 3-3-0. AWS EC2 설정

- EC2
    - EC2는 AWS에서 제공하는 **클라우드 컴퓨팅 서비스**로 아마존이 각 세계에 구축한 데이터 센터의 서버용 컴퓨터들의 자원을 원격으로 사용할 수 있는 서비스이다. (즉 한 대의 컴퓨터를 임대 하는 것!!)
    
- EC2 설정
    1. 아마존 웹 서비스(AWS)에 접속하여 로그인을 하면 다음과 같은 화면을 볼 수 있는데 여기서 EC2를 선택하여 “인스턴스 시작”을 클릭 한다.
    ![6](https://user-images.githubusercontent.com/49806698/235552093-0822e812-a3c1-4bd1-8836-da4fd588ea7d.png)

    
    ![7](https://user-images.githubusercontent.com/49806698/235552102-ad4e8d1f-85c1-42bf-9b09-50704b767fc5.png)

    
    2. 인스턴스 시작을 누르면 Amazon Machine Image(AMI) 및 인스턴스 유형을 선택할 수 있는 페이지로 넘어간다. 이 부분에서 프리 티어의 Ubuntu 22.04 LTS 버전을 선택한다.
    프리 티어를 사용한다면 인스턴스의 유형은 **t2.mircro**로 고정이다. <br>
    <img width="800" alt="8" src="https://user-images.githubusercontent.com/49806698/235553459-a5f6ebd5-4b02-414e-a59a-56534318df00.png"><br>
    <img width="800" alt="asv" src="https://user-images.githubusercontent.com/49806698/235553626-e0d9958d-eb86-40ac-993a-5f02e1e2bc04.png"><br>
    <img width="793" alt="9" src="https://user-images.githubusercontent.com/49806698/235553717-634b5759-e3c8-4f09-8385-addba151a09a.png"><br>


    3. 키 페어는 EC2 서버에 SSH 접속을 위해 필수적인 부분이다.
    **새 키페어 생성**”을 통해 키 페어를 생성 하면 `[key이름].pem` 파일이 다운로드 되는데 이 파일의 위치에 가서 ssh 명령을 수행하면 된다. 단!! 한 번 다운받은 후에는 재 다운로드가 불가능하기 때문에 백업이 필요하다.<br>
        
     <img width="594" alt="10" src="https://user-images.githubusercontent.com/49806698/235553752-94f4c18e-5eb6-4bda-b9a6-5eb897891e7c.png"><br>


        
    4. 네트워크는 보안 그룹을 따로 생성하도록 하였으며 SSH 트래픽의 경우 **위치 무관**으로 열어놓았다. 보안 상 특정 ip에서 접속하도록 설정 해야 하지만 현재는 스터디를 위한 것이므로 어느 IP에서도 접속이 가능하도록 하였다. 스토리지 구성의 경우 프리 티어는 최대 30GB 범용 SSD를 사용할 수 있어서 29GB를 할당하였다.<br>
        ![11](https://user-images.githubusercontent.com/49806698/235553782-fa8387f6-6307-4738-bc24-9171f285fc70.png)<br>

       

        
    5. 설정이 끝난 후 인스턴스 시작을 누르면 인스턴스 생성 완료 메시지가 나오며, 인스턴스 홈으로 가 보면 생성한 인스턴스를 볼 수 있다.<br>
        <img width="1479" alt="12" src="https://user-images.githubusercontent.com/49806698/235553851-39334b5a-5c82-4464-a346-6abef61c1c81.png"><br>

      

        
    6. AWS EC2 인스턴스는 서버를 중지하고 다시 실행하면 Public IP가 변경되기 때문에 Reboot을 해도 사용할 수 있는 IP가 필요하다.
        
        탄력적 IP는 만들어놓고 사용하지 않더라도 과금이 되기 때문에 필요한 만큼만 생성하자!
        메뉴의 “탄력적 IP”를 클릭하여 “탄력적 IP 주소 할당”을 선택한다.<br>
        
	![13](https://user-images.githubusercontent.com/49806698/235553880-3334847a-baa8-4b1a-bedd-df98d8a6c255.png)<br>

        
    7. 기본 Default로 할당을 해주면 된다.<br>
        <img width="870" alt="14" src="https://user-images.githubusercontent.com/49806698/235553895-8e6cc865-8373-4e18-876a-43916ff66f0c.png"><br>


        
    8. 그 후 생성한 탄력적 IP와 생성한 EC2 인스턴스를 연결하면 인스턴스의 IP가 탄력적 IP로 고정 됨을 알 수 있다.<br>
        
    <img width="310" alt="15" src="https://user-images.githubusercontent.com/49806698/235553912-924d5dbd-0f2f-4968-99b4-f7b3ed605c25.png"><br>

        
    9. 이제 인스턴스를 연결해보자.
    선택한 인스턴스에서 오른쪽 마우스 클릭 시 인스턴스 연결 정보를 알려주긴 하지만 매번 이런식으로 연결하면 매우 불편하다. 이때 **호스트를 등록하는 방법**을 사용하면 간단하게 접속할 수 있다.<br><br>
    
    - 우선 키 파일이 있는 폴더에 가서 터미널 실행 후 키 파일이 권한을 변경한다.<br>
    
    ```bash
    $ chmod 400 "본인의 키 파일 이름".pem
    ```
    
    - 그 후 키 페어 파일을 `~/.ssh/`로 복사하고(백업) 복사한 키 파일 권한을 변경한다.<br>
    
    ```bash
    $ cp "본인의 키 파일 이름".pem ~/.ssh/
    $ cd ~/.ssh
    $ chmod 600 "본인의 키 파일 이름".pem
    ```
    
    - 그 후 `~/.ssh/` 디렉터리에 `config` 파일을 생성해서 다음의 코드를 입력한다.
    작성자는 이미 만들어놓은 설정 정보가 존재하여 밑에 추가를 해주었다.<br>
    
    ```
    $ vi ~/.ssh/config
    
    # 아래는 파일 내용
    # ssh -i {키 페어 파일} {유저 이름}@{탄력적 IP}
    Host {원하는 호스트 이름}
    User {유저 이름}
    HostName {탄력적 IP}
    IdentityFile {키 페어 파일 위치}
    ```
    
    - 이제 키 페어 파일이 없는 곳에서도 SSH 접속이 가능하다.
    
    ```bash
    $ ssh "설정한 호스트이름"
    ```
    
    10. 마지막으로 보안그룹을 설정해주자. 우선 AWS에서 쓸 수 있는 보안 그룹은 두 가지 종류가 있다.
        - 인바운드 (Inbound): 외부 -> EC2 인스턴스 내부 허용
        - 아웃바운드 (Outbound): EC2 인스턴스 내부 -> 외부 허용
        
        보안 그룹은 **인스턴스의 보안 Tab**에서 볼 수 있다.
        
        - 인바운드 규칙에서는 외부에서 EC2로 요청할 때 허용할 IP 대역을 설정할 수 있다.
        EC2를 생성할 때 보안 그룹이 Default로 만들어졌지만, 새로 생성한 규칙을 통해 적용할 것이다.
        
        유형을 먼저 선택하면 그에 맞게 프로토콜과 포트의 범위가 고정되며, 특정 IP나 대역을 넣으려면 “사용자 지정”으로 추가할 수 있다.
        
        우선 가장 많이 사용하는 “SSH”, “HTTP”, “HTTPS”를 추가한다. **0.0.0.0/0**은 모든 IP의 접속을 허용한다는 얘기이다.<br>
            ![16](https://user-images.githubusercontent.com/49806698/235554065-fc881b4b-c638-4965-8fc4-bac395eca1f3.png)


            
        - 규칙을 다 설정하고 저장했다면 인스턴스로 와서 생성한 보안그룹으로 변경하면 정상적으로 적용된다.<br>
            
	![17](https://user-images.githubusercontent.com/49806698/235554081-d69f4464-80a6-47fc-915f-f7d5e70a2cd2.png)

            
    

### 3-3-1. 계정 추가 및 sudo 권한 할당

- AWS EC2로 실습을 진행할 경우 다음 단계부터 진행하면 된다.
- [AWS EC2가 아닌 다른 ubuntu 서버를 이용할 경우](https://kyungsnim.net/176)

### 3-3-2. 각 계정별 UTF-8 인코딩을 설정하여 한글 이슈 해결

- 다음 명령을 실행하여 시스템 전체 계정에서 한글과 관련한 인코딩을 사용할 수 있도록 설정한다.

```bash
sudo locale-gen ko_KR.EUC-KR ko_KR.UTF-8
sudo dpkg-reconfigure locales
```

- 각 계정의 디렉토리의 .bash_profile에 다음과 같은 설정을 추가한다.
- 만약 .bash_profile 파일이 없다면 vi 명령을 통해 만들어준다.

```bash
(만약 .bash_profile 파일이 없다면)
vi .bash_profile

(.bash_profile 파일)
LANG="ko_KR.UTF-8"
LANGUAGE="ko_KR:ko:en_US:en"

(저장 후 서버를 재시작해야 반영하지만 source 명령을 이용하여 Reboot 없이 바로 서버에 반영할 수 있다.)
source .bash_profile

(env 명령을 통해 반영 결과를 확인할 수 있다.)
(env: 현재 지정되어 있는 환경 변수들을 출력하거나, 새로운 환경 변수를 설정하고 적용된 내용을 출력하는 명령어)
env
```
<br>
<img width="167" alt="18" src="https://user-images.githubusercontent.com/49806698/235554217-98547f8e-98e9-4c72-95ea-c2b162eb659c.png"><br>




### 3-3-3. JDK, Maven 설치

- JDK 8 설치
    
    ```xml
    wget --no-cookies --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" https://javadl.oracle.com/webapps/download/GetFile/1.8.0_301-b09/d3c52aa6bfa54d3ca74e617f18309292/linux-i586/jdk-8u301-linux-x64.tar.gz
    ```
    
    - EC2 인스턴스에서 wget 명령을 통해 ubuntu 버전 jdk를 다운로드 받고 압축을 풀어준다.
    
    ```bash
    wget [다운로드 주소]
    
    tar -xvf [압축을 해제할 파일 이름]
    ```
    
    - 계정 Home 디렉 토리의 .bash_profile 파일에 JAVA_HOME/bin 디렉토리를 PATH로 등록한다.
    
    ```bash
    (심볼릭 링크를 연결)
    ln -s [자바 디렉토리 이름]/ java
    
    vi .bash_profile
    
    (.bahs_profile 안에서)
    export JAVA_HOME=~/java
    export PATH=$PATH:$JAVA_HOME/bin
    ```
    
    - PATH 설정을 완료 후 java -version 명령을 실행해 설치한 자바 버전을 확인한다.
    
    ```bash
    java -version
    ```
    
- Maven 설치
    - [https://maven.apache.org/](https://maven.apache.org/)
    - 위 링크에서 maven을 다운로드 한다. (tar.gz 파일)
    - 압축을 해제하면 설치가 완료된다.
    
    ```bash
    wget [다운로드 주소]
    
    tar -xvf [압축을 해제할 파일 이름]
    ```
    
    - .bash_profile에 MAVEN_HOME/bin 디렉토리를 PATH로 설정한다.
    
    ```bash
    (심볼릭 링크를 연결)
    ln -s [메이븐 디렉토리 이름]/ maven
    
    vi .bash_profile
    
    (.bahs_profile 안에서)
    export JAVA_HOME=~/maven
    export PATH=$PATH:$JAVA_HOME/bin:$MAVEN_HOME/bin
    ```
    
    - PATH 설정 완료 후 mvn -version 명령을 통해 설치한 Maven 버전을 확인한다.
    
    ```bash
    mvn -version
    ```
    
      
    
- 서버 실행
    
    ```bash
    java -cp target/classes/:target/dependency/* webserver.WebServer 8080
    
    // 백그라운드 실행
    java -cp target/classes/:target/dependency/* webserver.WebServer 8080 &
    ```
    
      
    

### 3-3-4. Git 설치. clone 및 빌드

- git 설치

```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install git
```

- git clone

```bash
git clone [clone할 url]
```

## 3-4. 웹 서버 실습

- 3장에서의 실습 요구 사항은 다음과 같은 구성으로 되어있다.
    - index.html 응답하기
    - GET 방식으로 회원가입하기
    - POST 방식으로 회원가입하기
    - 302 status code 적용
    - 로그인하기
    - 로그인 여부에 따른 사용자 목록 출력
    - CSS 지원하기
    
- 소스 코드 분석
    1. WebServer.java
        1. 웹 서버를 시작하고, 사용자의 요청이 있을 때까지 대기 상태에 있다가 사용자 요청이 있을 경우 사용자의 요청을 RequestHandler 클래스에 위임
    2. RequestHandler.java
        1. 사용자의 요청에 대한 처리와 응답에 대한 처리를 담당하는 클래스

### 3-4-1. 요구사항 (1) - index.html 응답하기

- WebServer.java를 실행하고 브라우저에서 [localhost:8080/index.html](http://localhost:8080/index.html을) 로 접속해보자
- 접속하고 개발자 도구 → Network → localhost의 Header를 보면 다음과 같은  Request Header 메시지를 볼 수 있다.<br>
    
   ![19](https://user-images.githubusercontent.com/49806698/235554296-abaec389-0cc9-4dd3-b11b-adb07c234de0.png)<br>


    
- 이 요청 값은 InputStream을 통해 들어오고 이를 BufferdReader를 통해 값을 출력할 수 있다.

```java
// 요청 라인 추출하기 -> HTTP Header : GET /index.html HTTP 1.1
BufferedReader br = new BufferedReader(new InputStreamReader(in, "UTF-8"));
String headerMessage = br.readLine();
log.debug("HTTP Request Header : {}", headerMessage);

// 전체 값 추출하기
if (headerMessage == null) return;
        	
while (!headerMessage.equals("")) {
	headerMessage = br.readLine();
    log.debug(" : {}", headerMessage);
}
```

- 요청 라인 값에서 /index.html을 추출하기 위해서 공백을 구분자로 split 함수를 이용한다

```java
String tokens[] = headerMessage.split(" ");
```

- 추출한 /index.html을 이용하여 웹 서버에 요청을 보내면 index.html의 내용을 응답받을 수 있다.

```java
byte[] body = Files.readAllBytes(new File("./webapp" + tokens[1]).toPath());    	
DataOutputStream dos = new DataOutputStream(out);

response200Header(dos, body.length);
responseBody(dos, body);
```

### 3-4-2. 요구사항 (2) - GET 방식으로 회원 가입하기

- 지금 상태에서 웹 서버를 띄운 상태에서 회원 가입을 누르면 아래와 같은 HTTP Header 요청을 볼 수 있다.<br>
<img width="824" alt="20" src="https://user-images.githubusercontent.com/49806698/235554329-2b434ef0-85ec-4f6e-8a48-92142d30e7c0.png"><br>

  

    
- 이 요청에서 알 수 있는건 **GET 방식으로 요청이 온다는 점**과 경로가 **/user/create?** 로 시작한다는 점이다.
- 이 URL 정보를 통해 우리는 회원가입을 진행할 수 있는 힌트를 얻을 수 있다.

```java
// 요구사항 2. GET 방식으로 회원 가입하기

// http header 요청 메세지
String url = tokens[1];

// startWith([String]) -> String 문자열로 시작하면 True를, 아니라면 false를 반환하는 함수
if (url.startsWith("/user/create")) {

	// url에서 ?의 위치를 반환
	int index = url.indexOf("?");
	String queryString = url.substring(index + 1);
	
// parseQueryString()
// &를 기준으로 나누며, =를 기준으로 key : value로 나누어 Map에 저장한다.
	Map<String, String> params = 
			HttpRequestUtils.parseQueryString(queryString);
	
	User user = new User(params.get("userId"), params.get("password"),
			params.get("name"), params.get("email"));
	
	log.debug("User : {}", user);
```

- 이 실습까지는 회원가입 처리 후 응답 메세지를 보내지 않았기 때문에 브라우저는 “응답 없음”을 띄울 것이다.<br>
![21](https://user-images.githubusercontent.com/49806698/235554370-2b048b0e-db2d-4f77-ab13-a64af84786af.png)<br>




- GET 방식은 사용자가 입력한 데이터가 URL에 표시 되기 때문에 보안적인 측면에서 취약하다.
- 또 브라우저의 정책마다 다르지만 크롬 브라우저의 경우 2MB의 길이로 제한(주소창 표시는 32KB)을 둔다고 한다.<br>
    
    ![22](https://user-images.githubusercontent.com/49806698/235554401-e72515a1-83b5-4b01-aa6f-a846c43101cd.png)<br>

    
    <출처 : [Chromium Docs](https://chromium.googlesource.com/chromium/src/+/main/docs/security/url_display_guidelines/url_display_guidelines.md#URL-Length) >
    

### 3-4-3. 요구사항 (3) - POST 방식으로 회원 가입하기

- GET 방식을 POST 방식으로 변경하기 위해 form.html의 form 태그 method 속성을 get에서 post로 수정한다.

```html
<form name="question" method="post" action="/user/create">
```

- POST 방식은 GET 방식과 다르게 URL에 쿼리 스트링이 표시되지 않으며, 이 쿼리 스트링은 HTTP 요청의 Body를 통해 전달된다.
- POST 방식으로 데이터를 전달하면서 헤더에 본문 데이터의 길이가 **Content-Length**라는 필드 이름으로 전달된다.
- 데이터의 파싱 방법은 “요구사항 (2)”와 같다.

```java
// 요구사항 3. POST 방식으로 회원 가입하기
	int contentLength = 0;
	while (!headerMessage.equals("")) {
		headerMessage = br.readLine();
		log.debug(" : {}", headerMessage);
		
		if (headerMessage.contains("Content-Length")) {
			String[] headerTokens = headerMessage.split(":");
			contentLength = Integer.parseInt(headerTokens[1].trim());
		}
	}
	
	String url = tokens[1];
	if (url.startsWith("/user/create")) {
		String queryString = IOUtils.readData(br, contentLength);
		
		Map<String, String> params = 
				HttpRequestUtils.parseQueryString(queryString);
		
		User user = new User(params.get("userId"), params.get("password"),
				params.get("name"), params.get("email"));
		
		log.debug("User : {}", user);
	} else {
		DataOutputStream dos = new DataOutputStream(out);
		byte[] body = Files.readAllBytes(new File("./webapp" + tokens[1]).toPath());
		response200Header(dos, body.length);
        responseBody(dos, body);
	}
```


![23](https://user-images.githubusercontent.com/49806698/235554834-c4cde14b-c8cc-4ccc-9a5f-14d7ed538001.png)

![24](https://user-images.githubusercontent.com/49806698/235554840-9cf7c277-c1b0-48b6-9a0b-6e7f6edcbb26.png)

<br>


- HTML에서는 기본으로 GET과 POST만을 지원한다.
    - GET
        - 서버에 존재하는 데이터 또는 자원을 가져오는 것
    - POST
        - 서버에 요청을 보내 데이터 추가, 수정, 삭제와 같은 작업을 실행하도록 하는 것
        
- 그럼 PUT과 DELETE는 왜 지원하지 않는걸까?
    - 2010년도에 [버그 리포트](https://www.w3.org/Bugs/Public/show_bug.cgi?id=10671#c0) 를 이슈로 등록하고 PR을 올림
    - 실제로 파이어 폭스 베타 버전에서는 관련 스펙이 추가되기도 했음
    - 그러나 공식적으로 추가되지는 않음
        - 코멘트에는 “PUT에는 Form의 Patyoad를 넣는 것을 기대하지 않고 delete 역시 payload가 있는 것이 맞지 않다.”는 내용이 있다.

### 3-4-4. 요구사항 (4) - 302 status code 적용

- 302 status Code란
    - Redirection 클래스에 속하며 일시 (**Temporarily)** 리다이렉션이라고 부른다.
    - 보통 리다이렉트 방식으로 이동한다고 한다면 내부적으로 302 상태 코드를 이용하여 이동한다고 생각하면 된다.
    
- 저번 실습까지 회원 가입을 완료하면 콘솔에만 log가 찍힐 뿐 페이지가 이동하지 않았다.
- 페이지를 이동하려면 “/index.html”을 담고 요청 URL 값을 이 값으로 변경하면 웹 서버가 이를 읽어 응답으로 보낼 수 있다.

```java
DataOutputStream dos = new DataOutputStream(out);
byte[] body = Files.readAllBytes(new File("./webapp" + url).toPath());
response200Header(dos, body.length);
responseBody(dos, body);
```

- 그러나 이 방식을 사용하면 ****************************브라우저가 이전 요청 정보를 유지 하고 있기 때문에**************************** 회원 가입 요청이 다시 실행된다.
- url을 보면 index.html로 이동했음에도 /user/create로 되어 있는 것을 알 수 있다.<br>
    
   ![25](https://user-images.githubusercontent.com/49806698/235554494-9305be02-217f-40f8-a751-2e3560c5fb42.png)<br>

    
- 이러면 데이터의 중복이 발생하므로 회원 가입을 처리하는 /user/create 요청과 index.html을 보여주는 요청을 분리한 후 HTTP의 302 상태 코드를 활용하여 해결할 수 있다.
- /index.html로 이동하도록 응답을 보낼 때 응답 헤더의 Location으로 다음과 같은 응답을 보내면 클라이언트는 상태 코드를 확인 한 후 302라면 Location의 값으로 서버에 재요청을 전송한다.
- 이러면 /user/create 요청이 아니라 /index.html 요청이 보내진다.

```java
HTTP/1.1 302 Found
Location: /index.html
```

```java
response302Header(dos, "/index.html");

private void response302Header(DataOutputStream dos, String url) {
    	try {
    		dos.writeBytes("HTTP/1.1 302 Redirect \r\n");
    		dos.writeBytes("Location: " + url + "\r\n");
    		dos.writeBytes("HTTP/1.1 302 Redirect \r\n");
    	} catch (Exception e) {
    		log.error(e.getMessage());
		}
}
```

![26](https://user-images.githubusercontent.com/49806698/235554563-5e82eef2-b40a-45d7-8dbf-c2babb9c892d.png)
<br>

- 상태 코드 정리
    - 2XX  : 성공. 클라이언트가 요청한 동작을 수신하여 이해했고 승낙햇으며, 성공적으로 처리
    - 3XX : 리다이렉션. 클라이언트는 요청을 마치기 위해 추가 동작이 필요함
    - 4XX : 요청 오류. 클라이언트에 오류가 발생
    - 5XX : 서버 오류. 서버가 유효한 요청을 수행하지 못했을 때 발생
    

### 3-4-5. 요구사항 (5) - 로그인하기

- 로그인 페이지로 이동하여 로그인을 한다.
- 로그인에 성공하면 “index.html”페이지로 이동하고 로그인이 실패하면 “/user/login_failed.html”로 이동한다.

- HTTP는 기본적으로 stateless(무상태) 방식이다.
- 즉 한번 요청을 보내그 응답을 받으면 클라이언트와 서버간의 연결을 끊기 때문에 각 요청 사이의 상태를 공유할 수 없다.
- 이 때 서버가 클라이언트를 식별할 수 없는 문제가 발생하는데, 이를 **쿠키**를 통해 해결할 수 있다.
- 이 실습에서는

- /user/create url로 요청이 오면 user를 만든후 addUser()를 통해 User를 저장한다.

```java
if (url.startsWith("/user/create")) {
	String queryString = IOUtils.readData(br, contentLength);
	
	Map<String, String> params = 
			HttpRequestUtils.parseQueryString(queryString);
	
	User user = new User(params.get("userId"), params.get("password"),
			params.get("name"), params.get("email"));
	
	DataBase.addUser(user);
}
```

- 로그인 요청이 오면 가입되어 있는 유저중 form으로 넘어온 id 값과 일치한 유저가 있고 password가 동일하다면 response302LoginSuccessHeader()를 실행하여 쿠키를 설정하고 아니라면 responseResource()를 실행하여 로그인 실패 페이지로 이동한다.

```java
if (url.startsWith("/user/create")) {
	String queryString = IOUtils.readData(br, contentLength);
	
	Map<String, String> params = 
			HttpRequestUtils.parseQueryString(queryString);
	
	User user = new User(params.get("userId"), params.get("password"),
			params.get("name"), params.get("email"));
	
	DataBase.addUser(user);
}
```

```java
private void response302LoginSuccessHeader(DataOutputStream dos) {
  	try {
		dos.writeBytes("HTTP/1.1 302 OK \r\n");
		dos.writeBytes("Set-Cookie: logined=true \r\n");
        dos.writeBytes("Location: /index.html \r\n");
        dos.writeBytes("\r\n");
	} catch (IOException e) {
		log.error(e.getMessage());
	}
}
```

```java
private void responseResource(OutputStream out, String url) throws IOException{
	DataOutputStream dos = new DataOutputStream(out);
	byte[] body = Files.readAllBytes(new File("./webapp" + url).toPath());
	response200Header(dos, body.length);
	responseBody(dos, body);
}
```

![27](https://user-images.githubusercontent.com/49806698/235554603-ff2e9135-2e92-4506-9b31-e0d6df804b4d.png)
<br>

### 3-4-6. 요구사항 (6) - 사용자 목록 출력

- 접근하고 있는 사용자가 “로그인” 상태일 경우 (이때는 쿠기 값이 logined=true) 사용자 목록 URL(http://localhost:8080/user/list)로 접근했을 대 사용자 목록을 출력한다. 만약 로그인한 사용자가 아니라면 로그인 페이지(login.html)로 이동한다.

- 우선 로그인한 상태인지 아닌지를 boolean 변수를 통해 판단한다.

```java
while (!headerMessage.equals("")) {
	headerMessage = br.readLine();
	
	if (headerMessage.contains("Cookie")) {
		logined = isLogin(headerMessage);
	}
}

// 쿠키가 있는지 판별하는 함수
private boolean isLogin(String headerMessage) {
  	String[] headerTokens = headerMessage.split(":");
  	Map<String, String> cookies = HttpRequestUtils.parseCookies(headerTokens[1].trim());
  	String value = cookies.get("logined");
  	
  	if (value == null) return false;
	return Boolean.parseBoolean(value);
}
```

- 만약 로그인한 상태라면 테이블을 만드는 html 코드를 데이터로 담아 요청을 보내며 로그인 하지 않았다면 로그인 페이지로 이동한다

```java
else if ("/user/list".equals(url)) {
	if (!logined) {
		responseResource(out, "/user/login.html");
	}
	
	Collection<User> users = DataBase.findAll();
	
	StringBuilder sb = new StringBuilder();
	sb.append("<table border='1'>");
	
	for (User user : users) {
		sb.append("<tr>");
		sb.append("<td>" + user.getUserId() + "</td>");
		sb.append("<td>" + user.getName() + "</td>");
		sb.append("<td>" + user.getEmail() + "</td>");
		sb.append("<tr>");
	}
	
	sb.append("</table>");
	byte[] body = sb.toString().getBytes();
	DataOutputStream dos = new DataOutputStream(out);
	
	response200Header(dos, body.length);
	responseBody(dos, body);
	
}
```

- 이때 쿠키는 프로젝트를 종료하고 다시 웹 서버를 실행하도 남아 있기 때문에 개발자 도구를 이용하여 쿠키를 지워줘야 한다.
    - 개발자 도구 → Application → Cookies → URL에 마우스 우클릭 → Clear
    
- 본 예제에서는 로그인한 정보를 쿠키에 담아 클라이언트에 보관했다. 이는 보안 이슈로 인해 좋지 않은 방법이며 보통 로그인 정보는 상태 데이터를 서버에 저장하는 **Session**에 담는 것이 관례이다.

### 3-4-6. 요구사항 (7) - CSS 지원하기

- 이클립스 log Message를 보면 CSS 파일을 정상적으로 요청함을 알 수 있다.<br>
    
   ![28](https://user-images.githubusercontent.com/49806698/235554621-bfe656fe-a5dd-4ddb-9ef2-8e9ea1f0bbf2.png)

    
- 하지만 CSS 적용이 안되는 것을 알 수 있는데, 이는 **모든 컨텐츠의 타입을 text/html로 보내기 때문**이다.
- 브라우저는 응답을 받은 후 Content-Type 헤더 값을 통해 Body에 포함되어 있는 컨텐츠가 어떤 것인지 판단한다.
- 그러나 지금까지 구현한 응답은 text/html로 고정되어 있어서 브라우저는 CSS 파일도 HTML로 인식했기 대문에 동작하지 않았다.
- 이는 CSS 요청에 대해 Content-Type 헤더 값을 text/html이 아니라 **text/css**로 응답을 보내면 해결할 수 있다.

```java
else if (url.endsWith(".css")) {
	DataOutputStream dos = new DataOutputStream(out);
	byte[] body = Files.readAllBytes(new File("./webapp" + url).toPath());
	
	response200CssHeader(dos, body.length);
	responseBody(dos, body);
}

private void response200CssHeader(DataOutputStream dos, int length) {
	try {
		dos.writeBytes("HTTP/1.1 200 OK \r\n");
		dos.writeBytes("Content-Type: text/css\r\n");
		dos.writeBytes("Content-Length:" + length + " \r\n");
		dos.writeBytes("\r\n");
	} catch (IOException e) {
		log.error(e.getMessage());
	}
}
```

### 3-5. Maven과 Gradle의 차이

- 빌드 관리 도구
    - 프로젝트와 관련한 설정을 관리하면서 소스코드에 대한 컴파일, 컴파일을 위해 필요한 라이브러리 관리, 테스트, 배포를 위한 패키징 작업 등의 작업을 자동화할 수 있도록 지원하는 도구
    
- Maven
    - **Java 전용 프로젝트 관리 도구**로, **Lifecycle 관리 목적 빌드 도구**이며, Apache Ant의 대안으로 만들어졌다.
    - XML을 기반으로 만들어진다.
    - 상속 구조를 가진다.
    - 다음의 8단계의 LifeCycle을 가진다.
        1. clean : 빌드 시 생성되어있었던 파일들을 삭제한다.
        2.  validate : 프로젝트가 올바른지 확인하고 필요한 모든 정보를 사용할 수 있는지 확인하는 단계
        3. compile : 프로젝트 소스코드를 컴파일 하는 단계
        4. test : 단위 테스트를 수행하는 단계. 테스트 실패 시 빌드 실패로 처리하며, 스킵이 가능하다.
        5. package : 실제 컴파일된 소스 코드와 리소스들을 jar, war 등의 파일의 배포를 위한 패키지로 만든다.
        6. verify : 통합 테스트 결과에 대한 검사를 실행하여 품질 기준을 충족하는지 확인한다.
        7. site : 프로젝트 문서와 사이트 작성, 생성하는 단계
        8. deploy : 만들어진 package를 원격 저장소에 release하는 단계
    - 필요한 라이브러리를 `pom.xml` 에 기술하며 **Project Modeling**이라고 한다.
    - pom → (Project Object Model)
    
- Gradle
    - Maven을 대체할 수 있는 프로젝트 구성 관리 및 범용 빌드 툴이며 Ant Builder와 Groovy Script를 기반으로 구축되어 기존 Ant의 역할과 배포 스크립의 기능을 모두 사용가능하며 스프링부트와 안드로이드에서 사용된다.
    - XML을 사용하는 Maven보다 간결한 정의가 가능하다.
    - 프로젝트 설정 주입 방식으로 정의하여 Maven의 상속 구조보다 재사용에 용의하다.
    - Gradle 설치 없이 `Gradle Wrapper`를 이용하여 빌드를지원

### 3-6. 디버깅을 위한 로깅(logging)

- Application이 정상적으로 동작하는지, 혹은 문제가 발생했을 떄 원인을 파악하기 위해 System.out.print API를 많이 사용한다.
- 하지만 이는 파일로 메시지를 출력하는 작업이기 때문에 많은 비용이 발생하여 성능 저하를 일으킬 수 있다.
- 또 디버깅을 목적으로 사용하는 경우 원복해야하는 문제점도 있다.
- 이를 보완하기 위해 등장한 라이브러리가 `Logging` 라이브러리이다.
- 이 중 SLF4J 라이브러리를 이용해 로깅 API에 대한 창구를 일원화했다.
- 즉 SLF4J를 이용하여 디버깅 메세지를 남기면 디버깅 메시지를 출력하는 구현체는 Log4J, Logback이 담당하는 방식으로 동작하며, 추후 다른 라이브러리로 교체한다 해도 소스코드를 수정할 필요 없이 구현체를 담당하는 라이브러리만 교체하면 된다.

```xml
<!-- Log Back 의존성 -->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.1.2</version>
</dependency>
```

```java
// SLF4J를 import 하여 log 라이브러리 사용
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

private static final Logger log = LoggerFactory.getLogger(RequestHandler.class);
```

- 자바 진영의 로깅 라이브러리는 메시지 출력 여부를 로그 레벨로 관리한다.
    - TRACE < DEBUG < INFO < WARN < ERROR가 있으며, 이 순서대로 로그 레벨이 높아진다.
    - 로그 레벨이 높아질 수록 출력되는 메시지는 적어지고, 로그 레벨이 낮을 수록 더 많은 로깅 레벨이 출력된다.
    - 실습에서 진행했던 Handler의 `log.debug()` 는 DEBUG 레벨이며 l`og.trace()`, `log.info()`, `log.warn()`, `log.error()`메소드가 있다.
    
- 이 로그 레벨 설정은 logback.xml 파일에서 설정할 수 있다.

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
