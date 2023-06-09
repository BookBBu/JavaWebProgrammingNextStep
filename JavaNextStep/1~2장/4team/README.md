# 1장 첫 번쨰 양파 껍질 벗기기
> 과거에는 대다수가 자바를 기반으로 하는 웹 서버 개발자였는데 현재는 다양하다.
## 1.1 대한민국 IT 개발자 직군의 종류
- 웹 백엔드 개발자
  - 자바, C#, 루비, 파이썬 등의 언어로 서버 쪽의 로직을 개발하는 역할을 한다.
- 웹 프론트엔드 개발자
  - HTML/CSS, 자바스크립트를 주로 사용하며 디자이너와 협업을 하는 개발자이다.
- 모바일 앱 개발자
  - 자바 기반의 안드로이드 개발자와 오브젝티브 C(또는 Swift) 기반의 iOS 개발자가 이 범주에 속한다.
- 기타
  - 시스템 프로그래머, 모바일 게임 개발자, 게임 서버 및 게임 클라이언트 개발자가 있다.
- 비 개발자 직군
  - 중요한 비개발자 직군으로 DBA, 시스템 엔지니어, 빅데이터 전문가가 있다.
  
## 1.2 개발자들에게 유용한 웹사이트들
> 소프트웨어 분야는 정말 빠르게 발전하고 있기 때문에 모든 지식을 알 수 없다.
> 빠르게 바뀌는 기술의 흐름을 파악하려면 온라인을 통해 지식을 습득하고 다양한 개발자와 소통해야 한다.
> 또한 모르는 문제는 검색을 통해 해결할 수 있어야 한다.
- google.com
- stackoverflow.com
- github.com
- slideshare.net
- trello.com
- 페이스북 그룹, 다양한 온라인 커뮤니티
- MOOC 사이트들

## 1.3 처음에 배워야 하는 것들
- 맥 / 리눅스 사용법
- 다양한 프로그래밍 언어
- 내 전문 분야에 대한 방향성을 결정

## 1.4 일단 시작해 보자
- [hour of code](http://code.org/learn)
- [칸 아카데미](https://khanacademy.org/)
- "기초 튼튼 코드 튼튼 다 함꼐 프로그래밍"(타나지리 카오리 저)
- [CS50](https://cs50.harvard.edu/)

## 1.5 본격적으로 웹 프로그래 밍에 도전하기
### 1.5.1 온라인 강의를 통한 학습
- [생활코딩](https://opentutorials.org/course/1688)
- [코드아카데미](https://www.codecademy.com/learn)
- [W3Schools](https://www.w3schools.com/)

### 1.5.2 책을 통한 학습
- 프로가 되기 위한 웹 기술 입문(고모리 유스케 저)
- 웹 프론트엔드 학습
  - "자바스크립트 & 제이쿼리: 인터랙티브 프론트엔드 웹 개발 교과서"(존 두켓 저)
  - "웹 표준 가이드: HTML5 + CSS3"(존 앨섭 저)
  - "자바스크립트를 말하다 : 가장 간결하면서도 완벽한 자바스크립트 입문서"(악셀 라우슈마이어 저)
- 웹 백엔드 학습
  - "자바 프로그래밍"(Jeff Langr 저)
  - "열혈 강의 자바 웹 개발 워크북 - MVC 아키텍쳐, 마이바티스, 스프링으로 만드는 실무형 개발자 로드맵"(엄진영 저)
  - "SQL 첫걸음 : 하루 30분 36강으로 배우는 완전 초보의 SQL 따라잡기"(아사이 아츠시 저)

## 1.6 학습 방법
1. 필요한 부분부터 흡수한다.
2. 대략적인 부분을 잡아서 조금씩 상세화한다.
3. 끝에서부터 차례대로 베껴간다.

# 2장 문자열 계산기 구현을 통한 테스트와 리팩토링
## 2.1 main() 메소드를 활용한 테스트의 문제점
- 프로덕션 코드와 테스트 코드(main() 메소드)가 같은 클래스에 위치하고 있다.
  - 이 문제를 해결하기 위한 첫 번째 단계로 프로덕션 코드와 테스트 코드를 분리할 수 있다.
- 프로덕션 코드의 복잡도가 증가하면 증가할수록, main() 메소드의 복잡도도 증가하고, 결과적으로 main() 메소드를 유지하는 데 부담이 된다.
  - 이 같은 문제를 해결하기 위해 테스트 코드를 각 메소드변로 분리할 수도 있다.
- 클래스가 가지고 있는 모든 메소드를 테스트할 수 밖에 없다.
- 테스트 결과를 매번 콘솔에 출력되는 값을 통해 수동으로 확인해야 한다는 것이다.
- 프로덕션 코드의 복잡한 로직을 머릿속으로 계산해 결과 값이 정상적으로 출력되는지 일일이 확인해야 하는 번거로움이 있다.

## 2.2 Junit을 활용해 main() 메소드 문제점 극복
- Junit은 단위 테스트 프레임워크 중 하나이다. JUnit은 앞 절에서 언급한 것처럼 main() 메소드의 한계를 해결해 줄 수 있는 도구이다.

### 2.2.1 한 번에 메소드 하나에만 집중
- 다른 메소드에 영향을 받지 않기 때문에 내가 현재 구현하고 있는 프로덕션 코드에 집중할 수 있는 효과를 얻을 수 있다.

### 2.2.2 결과 값을 눈이 아닌 프로그램을 통해 자동화
- main() 메소드의 두 번째 문제점은 실행 결과를 눈으로 직접 확인해야 한다는 것이다.
- JUnit은 이 같은 문제점을 극복하기 위해 assertEquals() 메소드를 제공한다.

### 2.2.3 테스트 코드 중복 제거
- 중복 코드는 프로그래밍의 가장 큰 적 중의 하나이다.
- 테스트 코드 또한 많은 중복이 발생한다.
- 각 테스트 메소드가 실행될 때 매번 @Before, @After 애노테이션으로 설정한 메소드가 실행되면서 초기화와 후처리 작업을 할 수 있다.
- 이와 같이 매번 초기화, 후 처리 작업을 통해 각 테스트 간에 영향을 미치지 않으면서 독립적인 실행이 가능하도록 지원한다.

## 2.3 문자열 계산기 요구사항 및 실습
- 실습 내용은 책 참고

## 2.4 테스트와 리팩토링을 통한 문자열 계산기 구현
- 복잡도가 증가하면 새로운 요구사항을 추가 구현하기도 쉽지 않고, 테스트가 깨질 경우 디버깅하기도 쉽지 않다.
- 위와 같이 복잡도가 쉽게 증가하는 원인은 요구사항의 복잡도가 높은 것이 가장 큰 원인이다.
- 복잡도를 낮출 수 있는 방법중의 하나가 끊임 없는없는 리팩토링을 통해 소스코드를 깔끔하게 구현하는 연습을 하는 것이다.

### 2.4.1 요구사항을 작은 단위로 나누기
- 복잡한 문제를 풀어가기 위해 첫 번째로 진행해야 하는 작업이 복잡한 문제를 작은 단위로 나눠 좀 더 쉬운 문제로 만드는 작업이다.

### 2.4.2 모든 단계의 끝은 리팩토링
- 소스코드의 복잡도가 쉽게 증가하는 이유는 하나의 요구사항을 완료한 후 리팩토링을 하지 않은 상태에서 다음 단계로 넘어가기 때문이다.
- 각 단계에서 다음 단계로 넘어가기 위한 작업의 끝은 내가 기대하는 결과를 확인했을 때가 아니라 결과를 확인한 후 리팩토링까지 완료했을 때이다.

### 2.4.3 동영상을 활용한 학습
- 동영상 참고

### 2.4.4 문자열 계산기 구현
- 메소드 이름은 영어로 작성하는 것이 일반적이지만 테스트 메소드가 어떤 테스트인지를 명확하게 전달하기 위해 영어로 작성하기 힘들다면 한글로 작성하는 것도 한 가지 방법이다.
- 세부 구현에 집중하도록 하지 않고 논리적인 로직을 쉽게 파악할 수 있도록 구현하는 것이 읽기 좋은 코드이다.
- 정규 표현식을 활용하면 문자열에서 원하는 문자열을 찾거나 특정한 패턴을 찾는 데 유용하다.
- 요구사항이 변경되면서 메소드 이름, 변수 이름을 변경하는 것 또한 중요한 리팩토링이다.

## 2.5 추가 학습 자료
### 2.5.1 테스트 주도 개발(Test Driven Development)과 리팩토링
- "테스트 주도 개발: 고품질 쾌속 개발을 위한 TDD 실천법과 도구"
- "테스트 주도 개발"(Kent Beck 저)
- "리팩토링 : 코드 품질을 개선하는 객체지향 사고법"(마틴 파울러 저)

### 2.5.5 정규 표현식
- 정규 표현식은 문자열을 조작을 지원하는 도구이다.
- "손에 잡히는 정규 표현식"(벤 포터 저)