
# 11장. 의존관계 주입(이하DI)을 통한 테스트하기 쉬운코드 만들기
작성일시: 2023년 6월 22일

## 📌11.1 왜 DI가 필요한가?

---

### 의존관계(dependency)

-  객체 혼자 모든일을 처리하기 힘들기 때문에 내가 해야 할 작업을 다른 객체에게 위임하면서 발생
-  즉, 내가 가지고 있는 책임과 역할을 다른 객체에 위임하는 순간 발생
- ex) 예를 들어 QnaService는 질문을 삭제하는 기능을 수행하기 위해 QuestionDao, AnswerDao에게 데이터베이스 접근 로직을 위임, QnaService는 삭제 작업을 완료하기 위해 QuestionDao, AnswerDao에 의존

### DI(Dependency Injection)

- DI는 객체 간의 의존관계를 어떻게 해결하느냐에 따른 새로운 접근 방식
- 지금까지 의존관계에 있는 객체를 사용하기 위해 객체를 직접 생성하고 사용하는 방식으로 구현
- ex) QnaService의 경우 의존관계에 있는 QuestionDao AnswerDao를 사용하기 위해 이 클래스의 인스턴스를 직접 생성하고 사용하는 방식으로 구현
- 이렇게 구현할 경우 유연한 개발을 하는데 한계가 있기 때문에 인스턴스를 생성하는 책임과 사용하는 책임을 분리하자는 것이 핵심
- ex)  QnaService는 QuestionDao 와 AnswerDao에 대한 인스턴스 생성에 대한 책임은 없애고 단순히 사용만 함으로써 유연성을 높이자는 것이 DI의 접근 방식
---
### 왜 DI를 활용하지 않으면 유연성이 떨어지는가?

현재 시스템 시간에 따라 "오전" 또는 "오후"를 반환하는 DateMessageProvider 클래스

```java
import java.util.Calendar;

public class DateMessageProvider {
	public String getDataMessage() {
		Calendar now = Calendar.getInstance();
		int hour = now.get(Calendar.HOUR_OF_DAY);
	
		if(hour <12 ) {
			return "오전";
		}
		return "오후";
	}
}
```

테스트 클래스


```java
public class DateMessageProviderTest {
	@Test
	public void 오전() throws Exception {
		DateMessageProvider provider = new DateMessageProvider();
		assertEquals("오전", provider.getDateMessage());
	}

	@Test
	public void 오후() throws Exception {
		DateMessageProvider provider = new DateMessageProvider();
		assertEquals("오후", provider.getDateMessage());
	}
}
```
테스트를 실행하면 컴퓨터의 현재 시간에 따라 테스트 메소드 중 둘중하나는 반드시 실패

두 개의 테스트가 모두 성공 할 수 없는 이유는 DateMessageProvider가 Calendar와 의존관계를 가지는데 테스트를 위해 Calendar의 시간을 변경할 수 있는 방법이 없음

 즉, 테스트를 모두 성공하려면 Calendar를 통해 조회하는 컴퓨터 시간을 변경할 수 있어야 하는데 변경할 방법이 없음. 
 
 소스코드를 컴파일 하는 시점에 Calendar 인스턴스가 이미 결정되어 버리기 때문 이와 같은 의존관계를 강하게 결합 되어 있다고 함.

 테스트 클래스를 모두 성공하려면 Calendar인스턴스를 생성한 후 전달하는 구조로 바궈야함

DateMessageProvider 외부에서 Calendar 인스턴스를 전달하는 방법은 생성자를 통해 전달하거나, getDateMessage() 메소드 인자로 전달하는 두 가지 방법

getDateMessage() 인자로 전달하는 방식
```java
import java.util.Calendar;

public class DateMessageProvider {
	public String getDataMessage(Calendar now) { // Calendar 인스턴스를 외부에서 전달 
		int hour = now.get(Calendar.HOUR_OF_DAY);
		if(hour <12 ) {
			return "오전";
		}
		return "오후";
	}
}


```
이에 대한 테스트 클래스
```java
public class DateMessageProviderTest {
	@Test
	public void 오전() throws Exception {
		DateMessageProvider provider = new DateMessageProvider();
		assertEquals("오전", provider.getDateMessage(createCurrentDate(11)));
	}

	@Test
	public void 오후() throws Exception {
		DateMessageProvider provider = new DateMessageProvider();
		assertEquals("오후", provider.getDateMessage(createCurrentDate(13)));
	}
  private Calendar creteCurrentDate(int hourOfDay) {
    Calendar now = Calendar.getInstance();
    now.set(Calendar.HOUR_OF_DAY, hourofDay);
    return now;
  }
}
```
이와 같이 의존하고 있는 Calendar인스턴스의 생성을 DateMessageProvider가 결정하지 않고 외부로부터 전달받음으로써 테스트가 가능

객체간의 의존관계에 대한 결정권을 의존관계를 가지는 객체(예시에서는 DateMessageProvider)가 가지는 것이 아니라 외부의 누군가가 담당하도록 맡겨 버림으로써 좀 더 유연한 구조로 개발하는 것이 DI라고함

유연한 구조의 애플리케이션은 변화를 최소화하면서 확장하기도 쉽고, 테스트하기도 쉽다는 것을 의미

### 결합도와 응집도 
- 결합도 : 결합도는 서로 다른 모듈 간에 상호 의존하는 정도 또는 연관된 관계를 나타내는 척도
- 응집도 : 모듈이 독립적인 기능을 수행하는지 또는 하나의 기능을 중심으로 책임이 잘 뭉쳐있는지 나타내는 척도
- 결합도는 낮추고 응집도는 높이게 설계하는 것이 좋음



## 11.2 DI를 사용하면 쌓이는 불편함(불만)


---
DI가 유연한 구조의 코드를 만들지만

개발하기 시작하면 이전보다 코딩량 증가, 불편한 점, 고려해야할 점 생김

질문/답변 게시판의 QnaService를 DI기반으로 변경하면서 기존의 개발 방시과 어떻게 다른지, 어떤 불편함이 있는지 살펴보자

QnaService
```java
publi class QnaService {
	private static QnaService qnaService;

	private QustionDao questionDao = QuestionDao.getInstance();
	private AnswerDao answerDao = AnswerDao.getInstance();

	private QnaService() {}

	public static QnaService getInstance(){
		if( qnaService ==null) {
			qnaService = new QnaService();
		}
		return qnaService;
	}
}
```

QnaService는 QuestionDao와 AnswerDao에 대한 의존관계를 가짐 이를 DI구조로 변경

```java
public class QnaService {
    private QuestionDao questionDao;
    private AnswerDao answerDao;

		// 생성자를 통해 필드로  관리
    public QnaService(QuestionDao questionDao, AnswerDao answerDao) {
        this.questionDao = questionDao;
        this.answerDao = answerDao;
    }

    public Question findById(long questionId) {
        return questionDao.findById(questionId);
    }

    public List<Answer> findAllByQuestionId(long questionId) {
        return answerDao.findAllByQuestionId(questionId);
    }

    public void deleteQuestion(long questionId, User user) throws CannotDeleteException {
        Question question = questionDao.findById(questionId);
        if (question == null) {
            throw new CannotDeleteException("존재하지 않는 질문입니다.");
        }

        List<Answer> answers = answerDao.findAllByQuestionId(questionId);
        if (question.canDelete(user, answers)) {
            questionDao.delete(questionId);
        }
    }
}
```
QnaService의 AnswerDao, QuestionDao는 메소드 전체에서 사용되기 때문에 생성자를 통해 필드로 관리

QuestionDao와 AnswerDao가 데이터베이스와 의존관계에 있기 때문에 의존관계를 가지지 않도록 변경할 수 있어야 데이터베이스에 의존하지 않는 테스트가 가능

QuestionDao와 AnswerDao를 상속해서 메소드를 오버라이딩


1. MockQuestionDao 이름의 클래스를 추가하고 QuestionDao 를 상속
  a. QuestionDao의 기본생성자가 싱글톤 패턴을 적용하면서 private으로 구현 했기 때문에 상속을 할 수 없음
  b. 객체 간의 의존관계에 대한 강한결합을 줄이기 위해 DI를 적용하면서 인터페이스를 사용

2. QuestionDao ,AnswerDao를 JdbcQuestionDao, JdbcAnswerDao로 이름을 바꾸고 인터페이스 이름을 QuestionDao, AnswerDao로 사용하도록 리팩토링
3. JdbcQuestionDao, JdbcAnswerDao에 의존하고 있던 QnaService 코드를 새로 추가한 QuestionDao, AnswerDao 인터페이스에 의존하도록 변경
4. MockQuestionDao , MockAnswerDao를 구현

```java
public class MockAnswerDao implements AnswerDao {
	private Map<Long, Answer> answers = Maps.newHashMap();

	@Override
	public Answer insert(Answer answer) {
		return answers.put(answer.getAnaswerId(), answer);
	}

	@Override
	public Answer findById(long answerId) {
		return answers.get(answerId);
	}

	@Override
	public List<Answer> findAllByQuestionId(long questionId){
		return answers.values().stream().filter(a-> a.getQuestionId == questionId)
			.collect(Clooectors.toList());
	}
	@Override
	public void delete (Long answerId){
		answers.remove(answerId);
	}
}

```


```java

public class QnaServiceTest {
    private QuestionDao questionDao;
    private AnswerDao answerDao;
    private QnaService qnaService;

    @Before
    public void setup() {
			questionDao = new MockQuestionDao();
			answerDao = new MockAnswerDao();
			qnaService = QnaService.getInstance(questionDao,answerDao);
    }

    @Test(expected = CannotDeleteException.class)
    public void deleteQuestion_없는_질문() throws Exception {
        qnaService.deleteQuestion(1L, newUser("userId"));
    }

    @Test(expected = CannotDeleteException.class)
    public void deleteQuestion_다른_사용자() throws Exception {
				Question question = new Question(1L, "javajigi");
				questionDao.insert(question);
        qnaService.deleteQuestion(1L, newUser("userId"));
    } 
		
		@Test(expected = CannotDeleteException.class)
    public void deleteQuestion_같은_사용자_답변없음() throws Exception {
				Question question = new Question(1L, "javajigi");
				questionDao.insert(question);
        qnaService.deleteQuestion(1L, newUser("javajigi"));
    }
		
}
```
세 번째 테스트를 추가하는 순간 에러 발생

이유: QnaService가 싱글톤 인스턴스이기 때문에 발생 
QnaService 가 참조하고 있는 QuestionDao(처음 인스턴스 생성시 인스턴스) 와 테스트 메소드에서 생성한 인스턴스가 다르기 때문에 발생하는 에러 → 테스트 데이터를 초기화 하는 작업을 setUp 메소드에서만 하도록 변경


DI를 적용해 유연함을 얻을 수 있을지 모르지만 추가 작어발 부분이 너무 많음

유연함을 가슴으로 느끼려면 같은 서비스를 일정 기간 유지보수하면서 사용자의 요구사항이 바뀌고 새로운 기능을 추가하는 경험을 해봐야함

---

## ** 11.3 불만 해소하기 **
문제점들
1. 싱클톤 패턴을 사용함으로 인해 테스트에 어려움이 있음
2. 테스트를 위해 매번 Mock 객체를 만드는 것은 많은 비용이 들고 귀찮은 작업




---

### ** 11.3.1 싱글톤 패턴을 제거한 DI **

싱글톤 패턴은 디자인 패턴 중에서 이해하기 가장 쉬워 널리 사용

- 단점
1. 글톤 패턴으로 구현된 클래스와 의존관계를 가지는 경우 해당 클래스와 강한 의존관계를 가지기 때문에 테스트하기 어려우며, 생성자를 private으로 구현하기 때문에 상속할 수 없음
2. 싱글톤 패턴 기반으로 개발하는 경우 객체지향 설계 원칙에 따라 개발하는 것을 저해하는 요인

개방-폐쇄 원칙 위배: 싱글톤 인스턴스가 혼자 너무 많은 일을 하거나, 많은 데이터를 공유시키면 다른 클래스 간의 결합도가 높아지게 되는데 이때 개방-폐쇄 원칙이 위배

개방-폐쇄 원칙: 기존의 코드를 변경하지 않으면서, 기능을 추가할 수 있도록 설계가 되어야 한다는 원칙

결합도가 높아지게 되면 유지보수 힘들고 테스트도 원할하게 진행할 수 없는 문제 발생


싱글톤 패턴을 적용하지 않으면 어떻게 인스턴스 하나만 생성해 재사용할 수 있을까?

컨트롤러는 싱글톤 패턴을 적용하지 않고 같은 효과를 낼수 있었다. ⇒ 가능한 이유: 서블릿 컨테이너가 DispatcherServlet을 초기화하는 시점에 컨트롤러 인스턴스를 생성한 후 재사용 가능하도록 구현했기 때문에..?

다른 클래스도 같은 방식으로 인스턴스를 관리한다면 굳이 싱글톤 패턴을 적용하지 않으면서 인스턴스 하나를 재사용할 수 있게 된다. 

각 인스턴스간의 의존관계는 DI 기반으로 구현하는 방식으로 해결


QnaService의 QuestionDao, AnswerDao를 이 방법으로 구현

싱글톤 패턴을 위한 코드를 모두 제거하고 생성자를 public으로 변경

```java
public class QnaService {
		private QuestionDao questionDao;
		private AnswerDao answerDao;

		public QnaService(QuestionDao questionDao, AnswerDao answerDao) {
				this.questionDao = questionDao;
				this.answerDao = answerDao;
		}
}
```
QnaService와 의존관계에 있는 DeleteQuestionController와 ApiDeleteQuestionController 수정
```java
public class DeleteQuestionController extends AbstractController {
    private QnaService qnaService;

    public DeleteQuestionController(QnaService qnaService) {
        this.qnaService = qnaService;
    }
}
```
마지막으로 두 개의 컨트롤러를 생성할 때 QnaService를 DI로 전달
```java
public class LegacyHandlerMapping implements HandlerMapping {
    private static final Logger logger = LoggerFactory.getLogger(DispatcherServlet.class);
    private Map<String, Controller> mappings = new HashMap<>();

    public void initMapping() {
        QuestionDao questionDao = new JdbcQuestionDao();
        AnswerDao answerDao = new JdbcAnswerDao();
        QnaService qnaService = new QnaService(questionDao, answerDao);
        mappings.put("/", new HomeController(questionDao));
        mappings.put("/qna/show", new ShowQuestionController(questionDao, answerDao));
        mappings.put("/qna/form", new CreateFormQuestionController());
        mappings.put("/qna/create", new CreateQuestionController(questionDao));
        mappings.put("/qna/updateForm", new UpdateFormQuestionController(questionDao));
        mappings.put("/qna/update", new UpdateQuestionController(questionDao));
        mappings.put("/qna/delete", new DeleteQuestionController(qnaService));
        mappings.put("/api/qna/deleteQuestion", new ApiDeleteQuestionController(qnaService));
        mappings.put("/api/qna/list", new ApiListQuestionController(questionDao));
        mappings.put("/api/qna/addAnswer", new AddAnswerController(questionDao, answerDao));
        mappings.put("/api/qna/deleteAnswer", new DeleteAnswerController(answerDao));

        logger.info("Initialized Request Mapping!");
    }

  }
```

싱글톤 패턴을 제거하는 작업이 예상보다 수정할 부분이 많음

싱글톤 패턴을 사용하면서 인스턴스 하나를 사용하도록 하려면 컨트롤러에서 시작해 꼬리에 꼬리를 물고 DI하는 구조로 구현해야 함


---

### ** 11.3.2 Mockito를 활용한 테스트 **

---
데이터베이스가 없는 상태에서도 테스트가 가능하도록 QuestionDao 와 AnswerDao에 대한 가짜(Mock)클래스를 구현 

만약 QnaService와 의존관계에 있는 DeleteQuestionContoller와 APiDeleteQuestionController에 대한 테스트를 진행하려면 mock 클래스를 구현해야 함 ⇒ 너무 번거로움

Mock테스트 프레임워크 사용

pom.xml에 의존관계 추가

```XML
<dependency>
			<groupId>org.mockito</groupId>
			<artifactId>mockito-core</artifactId>
			<version>1.10.19</version>
			<scope>test</scope>
</dependency>

```
Mockito를 활용해 QnaService의 deleteQuestion() 테스트하면  다음과 같다

```java


@RunWith(MockitoJUnitRunner.class)
public class QnaServiceTest {
    @Mock
    private QuestionDao questionDao;
    @Mock
    private AnswerDao answerDao;

    private QnaService qnaService;

    @Before
    public void setup() {
        qnaService = new QnaService(questionDao, answerDao);
    }

    @Test(expected = CannotDeleteException.class)
    public void deleteQuestion_없는_질문() throws Exception {
        when(questionDao.findById(1L)).thenReturn(null);

        qnaService.deleteQuestion(1L, newUser("userId"));
    }

    @Test
    public void deleteQuestion_삭제할수_있음() throws Exception {
        User user = newUser("userId");
        Question question = new Question(1L, user.getUserId(), "title", "contents", new Date(), 0) {
            public boolean canDelete(User user, List<Answer> answers) throws CannotDeleteException {
                return true;
            };
        };
        when(questionDao.findById(1L)).thenReturn(question);

        qnaService.deleteQuestion(1L, newUser("userId"));
        verify(questionDao).delete(question.getQuestionId());
    }

    @Test(expected = CannotDeleteException.class)
    public void deleteQuestion_삭제할수_없음() throws Exception {
        User user = newUser("userId");
        Question question = new Question(1L, user.getUserId(), "title", "contents", new Date(), 0) {
            public boolean canDelete(User user, List<Answer> answers) throws CannotDeleteException {
                throw new CannotDeleteException("삭제할 수 없음");
            };
        };
        when(questionDao.findById(1L)).thenReturn(question);

        qnaService.deleteQuestion(1L, newUser("userId"));
    }
}
```

위 코드를 보면 MockQuestionDao와 MockAnswerDao를 이용하지 않고도 테스트를 진행

@Mock 애노테이션으로 설정한 클래스의 메소드를 호출했을 때 반환 값을 지정할 수 있음.

클래스의 메소드가 호출되는지의 여부를 verify() 메소드를 통해 검증할 수 있음

when 함수 Mock 객체의 행동 설정 (어떤 행동을 할 때)

컨트롤러, 서비스와 같이 다른 클래스와 의존관계를 가지는 클래스를 테스트하는 경우 Mockito와 같은 Mock프레임워크를 사용할 것을 추천

---

### ** 11.3.3 DI보다 우선하는 객체지향 개발 **

---

9장에서 질문 삭제 기능의 중복을 제거하기 위해 QnaService를 추가

컨트롤러에 있던 중복 로직은 QnaService의 deleteQuestion()메소드를 통해 중복을 제거

계층형 아키텍쳐 관점과 객체지향 설계 관점에서 핵심 비즈니스 로직을 구현해야 하는 역할은 누가 담당할까?

```java
import java.util.Calendar;

public class DateMessageProvider {
	public String getDataMessage(Calendar now) { // Calendar 인스턴스를 외부에서 전달 
		int hour = now.get(Calendar.HOUR_OF_DAY);
		if(hour <12 ) {
			return "오전";
		}
		return "오후";
	}
}

```
DateMessageProviderTest
```java
public class DateMessageProviderTest {
	@Test
	public void 오전() throws Exception {
		DateMessageProvider provider = new DateMessageProvider();
		assertEquals("오전", provider.getDateMessage());
	}

	@Test
	public void 오후() throws Exception {
		DateMessageProvider provider = new DateMessageProvider();
		assertEquals("오후", provider.getDateMessage());
	}
	private Calendar createCurrentDate(int hourOfDay) {
		Calendar now = Calendar now = Calendar.getInstance();
		now.set(Calendar.HOUR_OF_DAY, hourOfDay);
		return now;
	}
}

```
DateMessageProvider는 현재 시간에 따라 오전/오후를 출력하는 간단한 로직

현재 시간에 따라 오전/오후를 반환하는 간단한 로직을 구현하는 데 이 정도의 테스트 코드를 작성해야 한다면 누가 테스트 함?


DateMessageProvider의 구조를 개선해 좀더 테스트하기 쉽도록 하자

좀 더 객체지향 적인 개발

시간을 추상화 하는 Hour클래스를 추가
```java
public class Hour{
	private int hour;

	public Hour(int hour){
		this.hour = hour;
	}
	public String getMessage(){
		if(hour <12 ) {
			return "오전";
		}
		return "오후";
	}
	@Override
	public boolean equals(Object obj){
		if(this==obj) return true;
		if(obj==null) return false;
		if(getClass() != obj.getClass() return false;
		Hour other = (Hour) obj;
		if (hour != other.hour) return false;
		return true;
	}

}
```

```java
public class HourTest {
	@Test
	public void 오전() throws Exception {
		Hour hour = new Hour(11);
		assertEquals("오전", hour.getDateMessage());
	}

	@Test
	public void 오후() throws Exception {
		Hour hour = new Hour(16);
		assertEquals("오후", hour.getDateMessage());
	}

}

```

로직 처리를 담당하는 새로운 객체를 추출함으로써 역할을 분리 테스트 또한 더 쉽게 할 수 있음

Hour를 추가한 후 자신만의 Calendar를 추가해 Hour를 생성하도록 지원할 수도 있다

```java
public class MyCalendar {
	private Calendar calendar:
	
	public MyCalendar(Calendar calendar) {
		this.calendar = calendar;
	}

	public Hour getHour() {
		return new Hour(Calendar.get(Calendar.HOUR_OF_DAY));
	}
}
public class MyCalendarTest {
	@Test
	public void getHour() throws Exception {
		Calendar now = Calendar.getInstance();
		now.set(Calendar.HOUR_OF_DAY,11);
		MyCalendar calendar = new MyCalendar(now);
		assertEquals(new Hour(11), calendar.getHour());
	}
}

```
“계층형 아키텍쳐 관점과 객체지향 설계 관점에서 핵심적인 비즈니스 로직을 구현해야 하는 역할은 누가 담당할까?”

많은 개발자들이 서비스 레이어를 담당하고 있는 QnaService에서 처리해야 한다고 생각

이는 서비스 레이어의 역할에 맞지 않음

핵심적인 비즈니스 로직 구현은 도메인 객체가 담당하는 것이 맞음 

서비스 레이어의 핵심적인 역할은 도메인 객체들이 비즈니스 로직을 구현할 수 있도록 도메인 객체를 조합하거나, 로직 처리를 완료했을 때의 상태값을 DAO를 활용해 데이터베이스에 영구 저장하는 등의 역할을 담당해야 함

많은 개발자들이 QnaService의 deleteQuestion() 메소드와 같이 구현하는 것이 일반적 이는 대표적인 절차지향적인 개발 방법

```java
public void deleteQuestion(long questionId, User user) throws CannotDeleteException {
        Question question = questionDao.findById(questionId);
        if (question == null) {
            throw new CannotDeleteException("존재하지 않는 질문입니다.");
        }

        if (!question.isSameUser(user)) {
            throw new CannotDeleteException("다른 사용자가 쓴 글을 삭제할 수 없습니다.");
        }

        List<Answer> answers = answerDao.findAllByQuestionId(questionId);
        if (answers.isEmpty()) {
            questionDao.delete(questionId);
            return;
        }

        boolean canDelete = true;
        for (Answer answer : answers) {
            String writer = question.getWriter();
            if (!writer.equals(answer.getWriter())) {
                canDelete = false;
                break;
            }
        }

        if (!canDelete) {
            throw new CannotDeleteException("다른 사용자가 추가한 댓글이 존재해 삭제할 수 없습니다.");
        }

        questionDao.delete(questionId);
    }
```


절차지향적으로 개발하면 서비스 레이어의 복잡도는 점점 더 증가하면서 유지보수와 테스트하기 힘든 상황이 발생함

또한 핵심 객체라 할 수 있는 도메인 객체는 사용자가 입력한 데이터를 DAO에 전달하거나 데이터베이스 데이터를 뷰에 전달하는 역할 밖에 하지 않음 그렇다 보니 도메인 객체가 값을 전달하는 Setter, getter 메소드만 가지는 상황이 발생

지금부터라도 도메인 객체에 더 많은 일을시키자

QnaService의 deleteQuestion() 메소드가 구현하고 있는 로직을 Question과 Answer가 담당하도록 구현해봄으로써 도메인 객체의 진정한 역할이 무엇인지 느껴보자.

먼저 삭제가 가능한지의 여부를 Question에 메시지를 보내 (canDelete()메소드) 확인하는 방식으로 리팩토링

```java
public class Question {
			private String writer;


			public boolean canDelete(User user, List<Answer> answers) throws CannotDeleteException {
		        if (!user.isSameUser(this.writer)) {
		            throw new CannotDeleteException("다른 사용자가 쓴 글을 삭제할 수 없습니다.");
		        }
		
		        for (Answer answer : answers) {
		            if (!answer.canDelete(user)) {
		                throw new CannotDeleteException("다른 사용자가 추가한 댓글이 존재해 삭제할 수 없습니다.");
		            }
		        }
		
		        return true;
		    }
		}
}
```

Question은 자신이 모든 로직을 처리하지 않고 인자로 전달된 User,Answer와 협력해  삭제 가능 여부를 판단

먼저 User에 질문한 사람 아이디를 전달해 사용자가 같은지 여부를 판단

글쓴이와 로그인 사용자가 같은 경우 각 답변이 삭제 가능한 상태인지 확인


```java
public class User {
	private String userId;

	public boolean isSameUser(String newUserId) {
		return userId.equals(newUserId);
	}
}

public class Answer {
	private String writer;

	public boolean canDelete(User user) {
		return user.isSameUser(this.writer);
	}
}
```
QnaService의 deleteQuestion() 메소드의 로직이 Qustion, Answer, User로 분산된 결과 코드의 복잡도는 낮아짐

```java

public void deleteQuestion(long questionId, User user) throws CannotDeleteException {
        Question question = questionDao.findById(questionId);
        if (question == null) {
            throw new CannotDeleteException("존재하지 않는 질문입니다.");
        }

        List<Answer> answers = answerDao.findAllByQuestionId(questionId);
        if (question.canDelete(user, answers)) {
            questionDao.delete(questionId);
        }
    }
```
객체 지향 설계의 핵심은 여러 객체가 서로 협력하면서 로직을 구현하는 것

객체 지향적으로 개발할 때 얻을 수 있는 장점 중의 하나는 테스트 하기 쉽다는 것

```java

public class QuestionTest {
    public static Question newQuestion(String writer) {
        return new Question(1L, writer, "title", "contents", new Date(), 0);
    }

    public static Question newQuestion(long questionId, String writer) {
        return new Question(questionId, writer, "title", "contents", new Date(), 0);
    }

    @Test(expected = CannotDeleteException.class)
    public void canDelete_글쓴이_다르다() throws Exception {
        User user = newUser("javajigi");
        Question question = newQuestion("sanjigi");
        question.canDelete(user, new ArrayList<Answer>());
    }

    @Test
    public void canDelete_글쓴이_같음_답변_없음() throws Exception {
        User user = newUser("javajigi");
        Question question = newQuestion("javajigi");
        assertTrue(question.canDelete(user, new ArrayList<Answer>()));
    }

    @Test
    public void canDelete_같은_사용자_답변() throws Exception {
        String userId = "javajigi";
        User user = newUser(userId);
        Question question = newQuestion(userId);
        List<Answer> answers = Arrays.asList(newAnswer(userId));
        assertTrue(question.canDelete(user, answers));
    }

    @Test(expected = CannotDeleteException.class)
    public void canDelete_다른_사용자_답변() throws Exception {
        String userId = "javajigi";
        List<Answer> answers = Arrays.asList(newAnswer(userId), newAnswer("sanjigi"));
        newQuestion(userId).canDelete(newUser(userId), answers);
    }
}
```

QuestionTest를 보면 QuestionDao, AnswerDao에 대한 의존관계가 없어 Mock 프레임워크를 사용할 필요도 없으면 테스트 구현 또한 간단

이와 같이 핵심 로직을 도메인 객체에 구현함으로써 로직 구현의 복잡도를 낮추고, 테스트하기 쉬운 코드로 구현할 수 있다

QnaService에 대한 테스트는 핵심 비즈니스 로직은 QuestionTest에서 모두 끝낸 상태이기 때문에 모든 테스트를 반복할 필요가 없음

QnaService로직에 대해서만 테스트 하면 된다.
```java

@RunWith(MockitoJUnitRunner.class)
public class QnaServiceTest {
 

    @Test(expected = CannotDeleteException.class)
    public void deleteQuestion_없는_질문() throws Exception {
        when(questionDao.findById(1L)).thenReturn(null);

        qnaService.deleteQuestion(1L, newUser("userId"));
    }

    @Test
    public void deleteQuestion_삭제할수_있음() throws Exception {
        User user = newUser("userId");
        Question question = new Question(1L, user.getUserId(), "title", "contents", new Date(), 0) {
            public boolean canDelete(User user, List<Answer> answers) throws CannotDeleteException {
                return true;
            };
        };
        when(questionDao.findById(1L)).thenReturn(question);

        qnaService.deleteQuestion(1L, newUser("userId"));
        verify(questionDao).delete(question.getQuestionId());
    }

    @Test(expected = CannotDeleteException.class)
    public void deleteQuestion_삭제할수_없음() throws Exception {
        User user = newUser("userId");
        Question question = new Question(1L, user.getUserId(), "title", "contents", new Date(), 0) {
            public boolean canDelete(User user, List<Answer> answers) throws CannotDeleteException {
                throw new CannotDeleteException("삭제할 수 없음");
            };
        };
        when(questionDao.findById(1L)).thenReturn(question);

        qnaService.deleteQuestion(1L, newUser("userId"));
    }
}
```
관계형 데이터베이스를 사용하면서 좀 더 객체지향적인 개발이 가능하도록 하려면 ORM(Oriented-Relational Mapping 프레임워크를 사용할 필요가 있다.

하지만 ORM 사용없이 메소드의 인자로 객체를 전달함으로써 일정 부분 객체지향적인 개발이 가능


---

## 📌11.4 DI 프레임워크  실습

---


### ** 11.4.1 요구사항 **

질문/답변 기능에 DI를 적용하는 작업은 LegacyHandlerMapping에서 프로그래밍을 통해 직접 구현 했음

DI를 매번 인스턴스를 생성해 전달하기 귀찮음

새로 추가한 MVC프레임워크는 프로그래밍을 통해 직접 DI할 수도 없음

자체적인 DI 프레임워크 구현

1. 애노테이션 각 클래스 역할에 맞도록 컨트롤러는 이미 추가되어있는 @Controller 서비스는 @Service DAO는 @Repository 애노테이션 설정
2. 3개의 설정으로 생성된 각 인스턴스 간의 의존관계는 @Inject 애노테이션을 사용
3. 개발자간의 원활한 소통을 위해 DI프레임워크를 통해 생성된 인스턴스와 개발자가 new 키워드 로 생성한 인스턴스를 분리, DI 프레임워크를 통해 생성된 인스턴스는 Bean이라 하자.
4. 앞으로 빈이라는 용어가 등장하면 DI 프레임워크가 자동으로 생성한 인스턴스로 정의.

위 요구사항을 만족하는 DI 프레임워크를 적용해 구현한 소스코드   

```java
@Controller
public class QnaController extends AbstractNewController {
		private MyQnaService qnaService;

		@Inject
		public QnaController (MyQnaService qnaService) {
				this.qnaService = qnaService;
		}

		@RequestMapping("/questions")
		public ModelAndView list(HttpServletRequest request, HttpServeletResponse response) throws Exception {
				return jspView("/qna/list.jsp");
		}
}

@Service
public class MyQnaService {
		private UserRespository userRepository;
		private QuestionRespository questionRespository
		
		@Inject
		public MyQnaService(UserRepository userRepository, QuestionRepository questionRepository) {
				this.userRepository = userRepository;
				this.questionRepository = questionRepository;
		}

}

@Repository
public class JdbcQuestionRespository implements QuestionRepository {}

@Repository
public class JdbcUserRespository implements UserRepository {}
	
```

위의 코드와 같이 각 클래스 역할에 맞는 애노테이션을 사용하도록 설정하고, 클래스간의 의존관계는 @Inject 애노테이션 사용이 가능한 DI 프레임워크를 구현

제약사항: 빈은 @Inject 애노테이션을 가지는 생성자는 하나만 존재 애노태이션이 설정되어있지 않으면 인자가 없는 기본 생성자를 제공


### ** 11.4.2 1단계 힌트 **

@Inject 애노테이션이 설정되어 있는 생성자를 통해 Bean 생성  

⇒ 생성자의 인자로 전달할 빈도 다른 빈과 의존 관계 

⇒ 꼬리에 꼬리를 물고 빈 간의 의존관계 발생 

⇒ 다른 빈과 의존관계를 가지지 않는 빈을 찾아 인스턴스를 생성할 때 까지 재귀를 실행하는 방식으로 구현

재귀를 통해 새로 생성한 빈은 BeanFactory의 Map<Class<?>,Object>에 추가해 관리

인스턴스를 생성하기 전에 먼저 Class<?> 에 해당하는 빈이 Map<Class<?>>,Object> 에 존재하는지 여부를 판단한 뒤 존재하지 않을 경우 생성하는 방식으로 구현


### ** 11.4.3 2단계 힌트 **

빈 인스턴스를 생성하기 위한 재귀 함수를 지원하려면 Class에 대한 빈 인스턴스를 생성하는 메소드와 Constructor에 대한 빈 인스턴스를 생성하는 메소드가 필요 

재귀함수의 시작은 instantiateclass()에서 시작

@Inject 애노테이션이 설정 되어 있는 생성자가 존재하면 instantiateConstructor() 메소드를 통해 인스턴스를 생성하고

@Inject 애노테이션이 설정 되어 있는 생성자가 존재하지 않을 경우 기본 생성자로 생성

instantiateConstructor()메소드는 생성자의 인자로 전달할 빈이 생성되어 Map<class<?>, Object>에 이미 존재하면 해당 빈을 활용

존재하지 않을경우 instantiateClass()메소드를 통해 빈을 생성



### ** 11.4.4 추가 요구사항 및 힌트 **


DI프레임워크를 완료 했다면 10장에서 구현한 MVC 프레임워크와의 통합

@Controller가 설정되어 있는 클래스를 찾는 ControllerScanner를 DI프레임워크가 있는 패키지로 이동해 @Controller 애노테치션만 찾던 역할에서 @Service, @Repository 애노테이션까지 확대 BeanScanner로 이름을 리팩토링

BeanScanner는 애노테이션이 설정되어있는 모든 클래스를 찾아 Set에 저장

---

## 📌11.5 DI프레임워크 구현

---

DI프레임워크를 구현하기 위한 대부분의 코드는 이미 제공 따라서 BeanFactory initialize()메소드만 구현

```java
public class BeanFactory {
    private static final Logger logger = LoggerFactory.getLogger(BeanFactory.class);

    private Set<Class<?>> preInstanticateBeans;
  
    // 재귀를 통해 생성한 빈을 관리 
    private Map<Class<?>, Object> beans = Maps.newHashMap();

    public BeanFactory(Set<Class<?>> preInstanticateBeans) {
        this.preInstanticateBeans = preInstanticateBeans;
    }

    @SuppressWarnings("unchecked")
    public <T> T getBean(Class<T> requiredType) {
        return (T) beans.get(requiredType);
    }

    public void initialize() {
    	for (Class<?> clazz: preInstanticateBeans) {
    		if(beans.get(clazz) == null) {
    			instantiateClass(clazz);
    		}
    	}

    }
    // 재귀 함수의 시작 
    private Object instantiateClass(Class<?> clazz) {
        Object bean = beans.get(clazz);
        if (bean != null) {
            return bean;
        }

      
        
        Constructor<?> injectedConstructor = BeanFactoryUtils.getInjectedConstructor(clazz);
        // Inject 애노테이션이 설정되어 있는 생성자가 존재하지 않으면 기본 생성자로 인스턴스 생성
        if (injectedConstructor == null) {
            bean = BeanUtils.instantiate(clazz);
            beans.put(clazz, bean);
            return bean;
        }
        // Inject 애노테이션이 설정되어 있는 생성자가 존재하면 instantiateConstructor 메소드를 통해 인스턴스 생성
        logger.debug("Constructor : {}", injectedConstructor);
        bean = instantiateConstructor(injectedConstructor);
        beans.put(clazz, bean);
        return bean;
    }

  
    //instantiateConstructor()메소드는 생성자의 인자로 전달할 빈이 생성되어 Map<class<?>, Object>에 이미 존재하면 해당 빈을 활용 존재하지 않을경우 instantiateClass()메소드를 통해 빈을 생성
    private Object instantiateConstructor(Constructor<?> constructor) {
        Class<?>[] pTypes = constructor.getParameterTypes();
        List<Object> args = Lists.newArrayList();
        for (Class<?> clazz : pTypes) {
            Class<?> concreteClazz = BeanFactoryUtils.findConcreteClass(clazz, preInstanticateBeans);
            if (!preInstanticateBeans.contains(concreteClazz)) {
                throw new IllegalStateException(clazz + "는 Bean이 아니다.");
            }

            Object bean = beans.get(concreteClazz);
            if (bean == null) {
                bean = instantiateClass(concreteClazz);
            }
            args.add(bean);
        }
        return BeanUtils.instantiateClass(constructor, args.toArray());
    }

    
}
```

BeanFactory 핵심 구현 로직은 InstatntiateClass(), instantiateConstructor() 메소드

두 메소드의 재귀 호출을 통해 복잡한 의존관계에 있는 빈을 생성하는 과정을 완료

@Controller @Service @Repository로 설정되어 있는 빈을 찾는 Bean Scanner 구현

```java
public class BeanScanner {
    private Reflections reflections;

    public BeanScanner(Object... basePackage) {
        reflections = new Reflections(basePackage);
    }
  
    // @Controller, @Service, @Repository로 설정되어 있는 빈을 찾음
    @SuppressWarnings("unchecked")
    public Set<Class<?>> scan() {
        return getTypesAnnotatedWith(Controller.class, Service.class, Repository.class);
    }

    @SuppressWarnings("unchecked")
    private Set<Class<?>> getTypesAnnotatedWith(Class<? extends Annotation>... annotations) {
        Set<Class<?>> preInstantiatedBeans = Sets.newHashSet();
        for (Class<? extends Annotation> annotation : annotations) {
            preInstantiatedBeans.addAll(reflections.getTypesAnnotatedWith(annotation));
        }
        return preInstantiatedBeans;
    }
}
```


마지막 구현 AnnotationHandlerMapping이 BeanFactory를 사용할 수 있도록 리팩토링

BeanFactory에 @Controller로 설정한 빈을 조회할 수 있도록 getControllers()메소드를 추가

```java

public Map<Class<?>, Object> getControllers() {
        Map<Class<?>, Object> controllers = Maps.newHashMap();
        for (Class<?> clazz : preInstanticateBeans) {
            if (clazz.isAnnotationPresent(Controller.class)) {
                controllers.put(clazz, beans.get(clazz));
            }
        }
        return controllers;
    }
```

AnnotationHandlerMapping에서 리팩토링할 부분은 BeanFactory를 초기화 한 후 @Controller 설정한 빈을 사용하면 끝

```java

public class AnnotationHandlerMapping implements HandlerMapping {
    private static final Logger logger = LoggerFactory.getLogger(AnnotationHandlerMapping.class);

   
    public AnnotationHandlerMapping(Object... basePackage) {
        this.basePackage = basePackage;
    }

    public void initialize() {
        BeanScanner scanner = new BeanScanner(basePackage);
        BeanFactory beanFactory = new BeanFactory(scanner.scan());
        beanFactory.initialize();
        Map<Class<?>, Object> controllers = beanFactory.getControllers();
        Set<Method> methods = getRequestMappingMethods(controllers.keySet());
        for (Method method : methods) {
            RequestMapping rm = method.getAnnotation(RequestMapping.class);
            logger.debug("register handlerExecution : url is {}, method is {}", rm.value(), method);
            handlerExecutions.put(createHandlerKey(rm),
                    new HandlerExecution(controllers.get(method.getDeclaringClass()), method));
        }

        logger.info("Initialized AnnotationHandlerMapping!");
    }

}
```

---
## 📌11.6 추가 학습 자료
---

### ** 11.6.1 의존관계 주입(DI)
1. ** [DI](https://www.slideshare.net/baejjae93/dependency-injection-36867592) **
   - 의존 관계가 무엇이고, 왜 필요한 것인지에 대해 그림을 통해 쉽게 설명하고 있는 문서


2. ** 토비의 스프링 3.1 (이일민 저 ,에이콘/2012) **
   - DI와 DI를 적용했을 때의 테스트 방법에 대해 잘 설명
   - DI, 객체지향 설계, 스프링 프레임워크 학습을 위한 다음 단계로 읽을 책 



