# 11장 의존관계 주입(이하 DI)을 통한 테스트하기 쉬운 코드 만들기

## 11.1) 왜 DI가 필요한가?
---
의존관계
- 객체 혼자 모든 일을 처리하기 힘들기 때문에 내가 해야 할 작업을 다른 객체에게 위임하면서 발생한다.
- 예를 들어, 앞에서 구현한 QnaService는 질문을 삭제하는 기능을 구현하기 위해 QuestionDao, AnswerDao에게 데이터베이스 접근 로직을 위임하고 있다. 이는 QnaService가 삭제 작업을 완료하기 위해 QuestionDao, AnswerDao에 의존하고 있다. 즉, 의존관계를 가지고 있는 것이다.

지금까지의 구현 방식
- 앞에서 예로 든 QnaService의 경우 의존관계에 있는 QuestionDao와 AnswerDao를 사용하기 위해 이 클래스의 인스턴스를 직접 생성하고 사용하는 방식으로 구현했다. 하지만 이는 유연한 개발을 하는데 한계가 있다.

```java
import java.util.Calendar;

public class DateMessageProvider{
    public StringgetDateMessage(){
        Calendar now = Calendar.getinstance(); // 인스턴스 생성
        int hour = now.get(Calendar.HOUR_OF_DAY); // 인스턴스 사용

        if(hour < 12){
            return "오전";
        }
        return "오후";
    }
}
```
따라서 인스턴스를 생성하는 책임과 사용하는 책임을 분리하자는 것이 DI의 핵심이다.

--- 

현재 시스템 시간에 따라 "오전" 또는 "오후"를 반환하는 DateMessageProvider라는 클래스가 있다.
```java
public class DateMessageProviderTest{
    @Test
    public void 오전() throws Exception{
        DateMessageProvider provider = new DateMessageProvice();
        assertEquals("오전", provider.getDateMessage());
    }

    @Test
    public void 오후() throws Exception{
        DateMessageProvider provider = new DateMessageProvice();
        assertEquals("오후", provider.getDateMessage());
    }
}
```
DateMessageProviderTest의 테스트를 실행하면 컴퓨터의 현재 시간에 따라 테스트 메소드 중의 하나는 반드시 실패한다. 요구사항은 이 두개의 테스트가 모두 성공하도록 소스코드를 리팩토링 하는 것이다.


두 개의 테스트가 모두 성공할 수 없는 이유는 DateMessageProvider가 Calendar와 의존관계를 가지는데 테스트를 위해 Calendar의 시간을 변경할 수 있는 방법이 없기 때문이다. 테스트를 모두 성공하려면 Calendar를 통해 조회하는 컴퓨터 시간을 변경할 수 있어야 하는데 소스코드를 컴파일하는 시점에 Calendar 인스턴스가 이미 결정되어 버린다. 

이러한 의존관계를 강하게 결합(tightley coupling)되어 있다고 한다. 

--- 

테스트를 모두 통과하려면 Calendar 인스턴스를 DateMessageProvider 외부에서 생성한 후 전달하는 구조로 바꾸어야 한다. 전달하는 방법은 생성자를 통해 전달하거나 getDateMessage() 메소드 인자로 전달하는 두 가지 방법이 있다.

그 중 getDateMessage() 메소드 인자로 전달하는 방법으로 리팩토링을 진행한다.
```java
import java.util.Calendar;

public class DateMessageProvider{
    public String getDateMessage(Calendar now){ // 외부에서 생성한 Calendar 객체 메소드 인자로 전달
        int hour = now.get(Calendar.HOUR_OF_DAY);

        if(hour < 12){
            return "오전";
        }
        return "오후";
    }
}
```
```java
public class DateMessageProviderTest{
    @Test
    public void 오전() throws Exception{
        DateMessageProvider provider = new DateMessageProvice();
        assertEquals("오전", provider.getDateMessage(createCurrentDate(11)));
    }

    @Test
    public void 오후() throws Exception{
        DateMessageProvider provider = new DateMessageProvice();
        assertEquals("오후", provider.getDateMessage(createCurrentDate(13)));
    }

    private Calendar createCurrentDate(int hourOfDay){
        Calendar now = Calendar.getInstance();
        now.set(Calendar.HOUR_OF_DAY, hourOfDay);
        return now;
    }
}
```
DateMessageProvider가 의존하고 있는 Calendar 인스턴스의 생성을 DateMessageProvider가 결정하지 않고 전달받음으로써 테스트가 가능해졌다. 이처럼 객체 간의 의존관계에 대한 결정권을, 의존관계를 가지는 객체가 가지는 것이 아니라 외부의 누군가가 담당하도록 맡겨 버림으로써 좀 더 유연한 구조로 개발하는 것을 DI라고 한다.


## 11.2) DI를 적용하면서 쌓이는 불편함(불만)
---
막상 DI 구조로 개발하면 이전보다 코딩량도 많아지고, 불편한 점도 생기고, 고려해야 할 부분도 생긴다.

질문/답변 게시판의 QnaSwervice를 DI 기반으로 변경하면서 기존의 개발 방식과 다르게 어떤 불편함이 있는지 살펴보도록 하겠다.

먼저 QnaService 코드를 살펴보자.
```java
public class QnaService {
	private static QnaService qnaService;

	private QustionDao questionDao = QuestionDao.getInstance(); // 인스턴스 생성
	private AnswerDao answerDao = AnswerDao.getInstance(); // 인스턴스 생성

	private QnaService() {}

	public static QnaService getInstance(){
		if(qnaService == null) {
			qnaService = new QnaService();
		}
		return qnaService;
	}
}
```
QnaService는 QuesionDao와 AnswerDao에 대한 의존관계를 가진다. 이 의존관계를 DI 구조로 변경해보자.

```java
public class QnaService {
    private static QnaService qnaService;

    private QuestionDao questionDao;
    private AnswerDao answerDao;

    // 1.
    public QnaService(QuestionDao questionDao, AnswerDao answerDao) {
        this.questionDao = questionDao;
        this.answerDao = answerDao;
    }

    // 2.
    public static QnaService getInstance(QuestionDao questionDao, AnswerDao answerDao){ 
        if(qnaService == null){
            qnaService = new QnaService(questionDao, answerDao);
        }
        return qnaService;
    }

    [...]
}
```
1. QuestionDao와 AnswerDao는 QnaService 메소드 전체에서 사용되고 있기 때문에 생성자를 통해 필드로 관리한다.
2. DI를 적용하기 위해 getInstance() 메소드 인자를 변경한다. 

하지만 QuestionDao와 AnswerDao가 데이터베이스와 의존관계에 있기 때문에 의존관계를 가지지 않도록 변경할 수 있어야 데이터베이스에 의존하지 않고, 데이터베이스가 없는 상태에서도 테스트가 가능해진다.

그렇다면 해결방법은?

1. QuestionDao와 AnswerDao를 상속해서 메소드를 오버라이드 한다.
    - QuestionDao의 기본 생성자가 싱글톤 패턴을 적용하면서 Private으로 구현했기 때문에 상속을 할 수 없다.
2. private을 protected로 변경한다.
    - 싱글톤 패턴 구현 원칙에 어긋난다.
3. 인터페이스를 추가한다.
    - 객체 간의 의존관계에 대한 강한 결합을 줄이기 위해 DI를 적용하면서 인터페이스를 사용하라고 한다.

이클립스의 "Extract interface" 리팩토링 기능을 활용하여 QuestionDao와 AnswerDao에서 인터페이스를 추출하고 JdbcQuestionDao, JdbcAnswerDao로 이름을 바꾼 후 인터페이스 이름을 QuestionDao, AnswerDao로 사용하도록 리팩토링 한다.

JdbcQuestionDao, JdbcAnswerDao에 의존하고 있던 QnaService 코드를 새로 추가한 QuestionDao, AnswerDao 인터페이스에 의존하도록 변경한다. 

마지막으로 두 개의 테스트용 Mock 클래스 MockQuestionDao, MockAnswerDao를 구현한다.

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
		return answers.values().stream()
            .filter(a -> a.getQuestionId() == questionId)
			.collect(Collectors.toList());
	}

	@Override
	public void delete(Long answerId){
		answers.remove(answerId);
	}
}
```
Map을 활용해 간단한 메모리 데이터베이스 역할을 하도록 구현했다.

마지막으로 테스트 코드를 추가하여 테스트한다.
```java
public class QnaServiceTest {
    private QuestionDao questionDao;
    private AnswerDao answerDao;
    private QnaService qnaService;

    @Before
    public void setup() {
        questionDao = new MockQuestionDao();
        answerDao = new MockAnswerDao();
        qnaService = QnaService.getInstance(questionDao, answerDao);
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
그런데 세 번째 테스트를 추가하는 순간 "존재하지 않는 질문입니다."는 에러가 발생한다.

QnaService가 참조하고 있는 QuestionDao와 테스트 메소드에서 생성한 인스턴스가 다르기 때문에 발생하는 에러로, 해당 에러는 QnaService가 싱글톤 인스턴스이기 때문에 발생하는 에러라는 것을 확인할 수 있었다. 

테스트 데이터 초기화 하는 작업을 setup() 메소드에서만 하도록 변경함으로써 문제를 해결했다.

## 11.3) 불만 해소하기
--- 
이와 같이 DI를 적용해 유연함을 얻을 수 있을지 모르겠지만 추가적으로 작업할 부분이 너무 많다는 느낌이 든다. 앞에서 발생한 이슈들을 살펴보자.

1. 싱글톤 패턴을 사용함으로 인해 테스트에 어려움이 있다.
2. 테스트를 위해 매번 Mock 객체를 만드는 것은 많은 비용이 들고 귀찮은 작업이다.

이 두 가지 이슈를 해결해보자.

### 11.3.1) 싱글톤 패턴을 제거한 DI

싱글톤 패턴의 단점
- 싱글톤 패턴으로 구현된 클래스와 의존관계를 가지는 경우 해당 클래스와 강한 의존관계를 가지기 때문에 테스트하기 어렵다.
- 생성자를 private으로 구현하기 때문에 상속할 수 없다.

따라서 싱글톤 패턴을 사용하지 않으면서 인스턴스 하나만 유지할 수 있는 다른 해결책을 찾으면 좋겠다.

---
컨트롤러는 싱글톤 패턴을 적용하지 않고 같은 효과를 낼 수 있었다. 이것이 가능한 이유는 서블릿 컨테이너가 DispatcherServlet을 초기화하는 시점에 컨트롤러 인스턴스를 생성한 후 재사용이 가능하도록 구현했기 때문이다.

그렇다면 같은 방식으로 싱글톤 패턴을 적용하지 않으면서 인스턴스 하나를 재사용하고, 각 인스턴스 간의 의존관계를 DI 기반으로 구현하는 방식으로 해결해보자.

먼저 QnaService의 QuestionDao와 AnswerDao를 이 방법으로 구현하고 테스트도 계속 진행해보자. 싱글톤 패턴을 위한 코드를 모두 제거하고 생성자를 public으로 변경한다.

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

다음으로 QnaService와 의존관계에 있는 DeleteQuestionController와 ApiDeleteQuestionController를 수정한다. 

```java
public class DeleteQuestionController extends AbstractController {
    private QnaService qnaService;

    public DeleteQuestionController(QnaService qnaService) {
        this.qnaService = qnaService;
    }
}

public class ApiDeleteQuestionController extends AbstractController {
    private QnaService qnaService;

    public ApiDeleteQuestionController(QnaService qnaService) {
        this.qnaService = qnaService;
    }
}
```

마지막으로 두 개의 컨트롤러를 생성할 때 QnaService를 DI로 전달하도록 수정해야 한다.
```java
public class LegacyhandlerMapping implements HandlerMapping{
    private Map<String, Controller> mappings = new HashMap<>();

    public void initMapping(){
        QnaService qnaService = QnaService(JdbcQuestionDao.getinstance(), JdbcAnswerDao.getInstane());
        mappings.put("/qna/delete", new DeleteQuestionController(qnaService));
        mappings.put("/api/qna/deleteQuestion", new ApiDeleteQuestionController(qnaService));
    }
}
```
하지만 아직도 JdbcQuestionDao와 JdbcAnswerDao는 싱글톤 패턴을 사용하고 있다. 이 두 개의 DAO도 싱글톤 패턴을 적용하지 않도록 수정한다.

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

싱글톤 패턴을 사용하지 않으면서 인스턴스 하나를 사용하려면 위와 같이 컨트롤러에서 시작해 꼬리에 꼬리를 물고 DI하는 구조로 구현해야 한다. 의존관계 깊이가 길어질수록 객체 간의 의존관계를 연결하는 작업이 복잡해질 것이다.

### 11.3.2) Mockito를 활용한 테스트

앞에서 데이터베이스가 없는 상태에서도 테스트가 가능하도록 QuestionDao와 AnswerDao에 대한 Mock클래스를 구현했다. 만약 QnaService와 의존관계에 있는 DeleteQuestionController에 대한 테스트를 진행하려면 또한 QnaService에 대한 Mock 클래스를 구현해야 한다. 

테스트 코드와 Mock 클래스 코드까지 구현하는 것은 개발자에게 큰 부담이 된다. 이 같은 단점을 보완하기 위해서 Mock 클래스를 구현하지 않아도 테스트가 가능하도록 지원하는 Mock 프레임워크가 있다.

---
Mock
- 한글로 "모의, 가짜의"라는 뜻으로 주로 객체 지향 프로그래밍으로 개발한 프로그램을 테스트할 때 테스트를 수행할 모듈과 연결되는 외부의 다른 서비스나 모듈들을 실제 사용하지 않고 필요한 실제 객체와 동일한 모의 객체를 만들어 테스트의 효용성을 높이기 위해 사용한다.
- 자바 진영에서 많이 사용하는 프레임워크에는 Mockito, jMock, EasyMock 등이 있다.

Mockito 활용
- 메이븐 설정 파일(pom.xml)에 Mockito에 대한 의존관계를 추가
```XML
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>1.10.19</version>
    <scope>test</scope>
</dependency>
```
Mockito는 @Mock 애노테이션으로 설정한 클래스의 메소드를 호출했을 때 반환 값을 지정할 수 있다. 또한 @Mock으로 설정한 클래스의 메소드가 호출되는지의 여부를 verify() 메소드를 통해 검증하는 작업 또한 가능하다.

```java
import static org.mockito.Mockito.when;

import org.mockito.Mock;
import org.mockito.runners.MockitoJUnitRunner;

[...]

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

    [...]
}
```
컨트롤러, 서비스와 같이 다른 클래스와 의존관계를 가지는 클래스를 테스트하는 경우 Mockito와 같은 Mock 프레임워크를 사용할 것을 추천한다.

### 11.3.3) DI보다 우선하는 객체지향 개발

"계층형 아키텍처 관점과 객체지향 설계 관점에서 핵심적인 비즈니스 로직을 구현해야 하는 역할은 누가 담당해야 할까?"

이에 대한 답을 제시하기 전에 앞에서 구현한 DateMessageProvider를 다시 한 번 살펴보자.

DateMessateProvider는 현재 시간에 따라 오전/오후를 출력한느 정말 간단한 로직을 구현하고 있다. 그런데 이 간단한 로직을 테스트하기 위해 비교적 많은 코드를 구현해야 했다. DateMessageProvider 구조를 개선해 좀 더 간단히 테스트할 수 있는 방법은 없을까?

이에 대한 해결책은 좀 더 객체지향적인 개발에 있다.

---
먼저 시간을 추상화하는 Hour 클래스를 추가한다.

```java
public class Hour{
	private int hour;

	public Hour(int hour){
		this.hour = hour;
	}

	public String getMessage(){
		if(hour < 12) {
			return "오전";
		}
		return "오후";
	}

	@Override
	public boolean equals(Object obj){
		if(this == obj)
            return true;
		if(obj == null)
            return false;
		if(getClass() != obj.getClass())
            return false;
		Hour other = (Hour) obj;
		if (hour != other.hour)
            return false;
		return true;
	}
}
```
Hour가 DateMessageProvider가 구현하던 오전/오후를 반환하는 로직을 담당하고 있다.
이에 대한 테스트 코드는 다음과 같다.

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
이와 같이 로직 처리를 담당하는 새로운 객체를 추출함으로써 역할을 분리할 수 있으며, 테스트 또한 더 쉽게 할 수 있다.

---
다시 돌아와서, QnaService에 대한 테스트 코드를 살펴보면 Mockito를 활용해 테스트하는 과정이 복잡하다. 이 부분을 좀 더 개선할 수 없을까?

이에 대한 해결책 또한 객체지향 개발에 답이 있다.

"계층형 아키텍처 관점과 객체지향 설계 관점에서 핵심적인 비즈니스 로직을 구현해야 하는 역할은 누가 담당해야 할까?"

도메인 객체
- 핵심적인 비즈니스 로직 구현

서비스 레이어의 핵심 역할
- 도메인 객체들이 비즈니스 로직을 구현할 수 있도록 도메인 객체를 조합
- 로직 처리를 완료했을 때의 상태값을 DAO를 활용해 데이터베이스에 영구 저장 

QnaService의 deleteQuestion() 메소드가 구현하고 있는 로직은 Question과 Answer가 담당하도록 구현해 봄으로써 도메인 객체의 진정한 역할이 무엇인지 느껴보자.

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

먼저 삭제가 가능한지의 여부를 Question에 메시지를 보내(canDelte() 메소드) 확인하는 방식으로 리팩토링 할 수 있다.

```java
public class Question{
    private String writer;

    [...]

    public boolean canDelete(User user, List<Answer> answers) throws CannotDeleteException{
        if(!user.isSameUser(this.writer)){
            throw new CannotDeleteException("다른 사용자가 쓴 글을 삭제할 수 없습니다.");
        }

        for(Answer answer : answer){
            if(!answer.canDelete(user)){
                throw new CannotDeleteException("다른 사용자가 추가한 댓글이 존재해 삭제할 수 없습니다.");
            }
        }
        return true;
    }
}
```
Question은 자신이 모든 로직을 처리하지 않고 인자로 전달된 User, Answer와 협력해 삭제 가능 여부를 판단한다. 

User의 구현 코드는 다음과 같다. Answer도 같은 방식으로 구현한다.
```java
public class User{
    private String userId;

    [...]

    public boolean isSameUser(String newUserId){
        return userId.equals(newUserId);
    }
}
```

User, Answer에도 복잡한 로직이 구현되어 있는 것은 아니다. QnaService의 deleteQuestion() 메소드의 로직이 Question, Answer, User로 분산된 결과 코드의 복잡도가 많이 낮아졌다.

그렇다면 deleteQuestion() 메소드는 어떻게 바뀌었을까?

```java
public class QnaService{
    public void deleteQuestion(long questionId, User user) throws CannotDeleteException{
        Question question = questionDao.findById(questionId);
        if(question == null){
            throw new CannotDeleteException("존재하지 않는 질문입니다.");
        }

        List<Answer> answers = answerDao.findAllByQuestionId(questionId);
        if(question.canDelete(user, answers)){
            questionDao.delete(questionId);
        }
    }
}
```
객체지향 개발을 하기 위한 좋은 연습은 상태 값을 가지는 도메인 객체에서 값을 꺼내려고 하지 말고 객체에 메세지를 보내 작업을 위임한다는 생각으로부터 시작할 수 있다.

또한 객체지향적으로 개발할 때 얻을 수 있는 장점 중의 하나는 테스트하기 쉽다는 것이다. 

다음은 Question의 canDelete() 메소드에 대한 테스트 코드이다.
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
QuestionDao, AnswerDao에 대한 의존관계가 없기 때문에 굳이 Mockito와 같은 Mock 프레임워크를 사용할 필요도 없으며, 테스트 구현 또한 Mockito를 사용할 때보다 간단하다. 

이와 같이 핵심  로직을 도메인 객체에 구현함으로써 로직 구현의 복잡도를 낮추고, 테스트하기 쉬운 코드로 구현할 수 있다. 

## 11.4) DI 프레임워크 실습
---

질문/답변 기능에 DI를 적용하는 작업은 LegacyhandlerMapping에서 프로그래밍을 통해 직접 구현했다. DI가 필요할 때마다 매번 직접 인스턴스를 생성해 전달하기 귀찮다. 자체적인 DI 프레임워클르 구현해보자.

### 11.4.1) 요구사항
 - 각 클래스에 대한 인스턴스 생성 및 의존관계 설정을 애노테이션으로 자동화한다.
 - 애노테이션은 각 클래스 역할에 맞도록 컨트롤러는 @Controller, 서비스는 @Service, DAO는 @Repository로 설정한다. 이 3개의 설정으로 생성된 각 인스턴스 간의 의존관계는 @Inject 애노테이션을 사용한다.
 - 개발자간의 원활한 소통을 위해 DI 프레임워크 를 통해 생성된 인스턴스는 빈(Bean)이라는 이름을 사용한다.

제약사항
- 빈은 @Inject 애노테이션을 가지는 생성자는 하나만 존재하며, @Inject 애노테이션이 설정되어 있지 않으면 인자가 없는 기본 생성자를 제공하는 것으로 한다.

### 11.4.2) 1단계 힌트
재귀함수를 활용하여 구현하기
- @Inject 애노테이션이 설정되어 있는 생성자를 통해 빈을 생성해야 하는데, 이 생성자의 인자로 전달할 빈도 다른 빈과 의존관계에 있다. 이와 같이 꼬리에 꼬리를 물고 빈 간의 의존관계가 발생할 때에 다른 빈과 의존관계를 가지지 않는 빈을 찾아 인스턴스를 생성할 때까지 재귀를 실행하는 방식으로 구현할 수 있다. 

### 11.4.3) 2단계 힌트
빈 인스턴스를 생성하기 위한 재귀 함수를 지원하려면 Class에 대한 빈 인스턴스를 생성하는 메소드와 Constructor에 대한 빈 인스턴스를 생성하는 메소드가 필요하다.

### 11.4.4) 추가 요구사항 및 힌트
- 이 장에서 구현한 DI 프레임워크를 활용할 경우 10장에서 @Controller가 설정되어 있는 클래스를 찾는 ControllerScanner를 DI 프레임워크가 있는 패키지로 이동해 @Controller, @service, @Repository에 대한 지원이 가능하도록 개선한다.
- MVC 프레임워크의 AnnotationhandlerMapping이 BeanFactory와 BeanScanner를 활용해 동작하도록 리팩토링한다. 

## 11.5) DI 프레임워크 구현
---
이미 대부분의 기반 코드는 제공하고 있기 때문에, BeanFacory의 initalize() 메소드만 구현하면 된다.

```java
public class BeanFactory {
    private static final Logger logger = LoggerFactory.getLogger(BeanFactory.class);
    private Set<Class<?>> preInstanticateBeans;
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

    private Object instantiateClass(Class<?> clazz) {
        Object bean = beans.get(clazz);
        if (bean != null) {
            return bean;
        }

        Constructor<?> injectedConstructor = BeanFactoryUtils.getInjectedConstructor(clazz);
        if (injectedConstructor == null) {
            bean = BeanUtils.instantiate(clazz);
            beans.put(clazz, bean);
            return bean;
        }

        logger.debug("Constructor : {}", injectedConstructor);
        bean = instantiateConstructor(injectedConstructor);
        beans.put(clazz, bean);
        return bean;
    }

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

다음으로 @Controller, @Service, @Repository로 설정되어 있는 빈을 찾는 BeanScanner를 다음과 같이 구현한다.

```java
public class BeanScanner {
    private Reflections reflections;

    public BeanScanner(Object... basePackage) {
        reflections = new Reflections(basePackage);
    }

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

마지막 구현은 AnnotationHandlerMapping이 BeanFactory를 사용하도록 리팩토링한다.

```java
public class BeanFacotry{
    private Set<Class<?>> preInstanticateBeans;
    private Map<Class<?>, Object> beans = Maps.newHashMap();

    [...]

    public Map<Class<?>, Object> getControllers() {
        Map<Class<?>, Object> controllers = Maps.newHashMap();
        for (Class<?> clazz : preInstanticateBeans) {
            if (clazz.isAnnotationPresent(Controller.class)) {
                controllers.put(clazz, beans.get(clazz));
            }
        }
        return controllers;
    }
}
```

AnnotationHandlerMapping에서 리팩토링할 부분은 BeanFactory를 초기화한 후 @Controller로 설정한 빈을 사용하면 된다.

```java
public class AnnotationHandlerMapping implements HandlerMapping {
    private Object[] basePackage;

    public AnnotationHandlerMapping(Object... basePackage) {
        this.basePackage = basePackage;
    }

    public void initialize() {
        BeanScanner scanner = new BeanScanner(basePackage);
        BeanFactory beanFactory = new BeanFactory(scanner.scan());
        beanFactory.initialize();
        Map<Class<?>, Object> controllers = beanFactory.getControllers();
       
       [...]
    }
}
```
## 11.6) 추가 학습 자료
---
### 11.6.1) 의존관계 주입(DI)
1. 의존관계가 무엇이고 왜 필요한 것인지에 대해 그림을 통해 쉽게 설명 : http://www.slideshare.net/baejjae93/dependency-injection-36867592

2. DI와 DI를 적용했을 때의 테스트 방법에 대해 설명 : "토비의 스프링 3.1" 1권 1,2장