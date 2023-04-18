# 📂 1장 첫 번째 양파 껍질 벗기기

> **"양파 껍질을 벗기듯이 학습하라"**

즉, 한 번에 한 가지 지식을 깊이 있게 학습하는 것에 집중하기보다 다양한 분야의 얕은 지식을 학습 후 일정 수준이 되면 다음 단계의 깊은 지식으로 서서히 깊이를 더해가라.

## 🚩 대한민국 IT 개발자 직군의 종류

과거에는 대다수가 **자바**를 기반으로 하는 웹 서버 개발자였다.

**웹 백엔드 개발자**

: 자바, C#, 루비, 파이썬 등의 언어로 서버 쪽의 로직을 개발한다. 대부분, 데이터베이스도 잘 알아야 한다. 일부 프론트엔드 개발 작업도 담당하는 경우가 일반적이다.

**웹 프론트엔드 개발자**

: HTML/CSS, JS를 주로 사용하며 디자이너와 협업하는 개발자이다. 최근에는 Angular.js, React.js, Vue.js와 같은 라이브러리도 잘 사용해야 하고, node.js를 통해 웹 백엔드 개발까지 가능하다.

**모바일 앱 개발자**

: 자바 기반의 안드로이드 개발자와 오브젝브 C(또는 Swift) 기반의 iOS 개발자가 속한다.

**기타**

: 시스템 프로그래머, 모바일 게임 개발자, 게임 서버 및 게임 클라이언트 개발자가 속한다.

**비개발자 직군**

: DBA(Database Administration), 시스템 엔지니어, 빅데이터 전문가 등이 있다.

---

## 🚩 개발자들에게 유용한 웹 사이트들

[Stack Overflow - Where Developers Learn, Share, & Build Careers](http://www.stackoverflow.com)

: 개발자들의 지식인 같은 곳이다.

[GitHub: Where the world builds software](http://www.github.com)

: 소셜 코딩이라고 부르기도 한다. Git의 사용법과 함께 필수적으로 알아야 한다. 참고로 Git은 소스코드에 대한 버전관리용 도구이고, Github는 Git을 지원하는 웹 서비스이므로 엄연히 다른 것이다.

[Manage Your Team's Projects From Anywhere | Trello](http://www.trello.com)

: 칸반이라는 툴을 적용할 수 있는 도구이다. 프로젝트 진행 시 프로젝트 관리를 위한 협업 도구로 유용하다.

---

## 🚩 일단 시작해 보자

소프트웨어를 만든다는 거창한 계획을 세우기보다 일단 무엇이라도 만드는 작업을 시작해 보자. 온라인으로도 학습할 수 있는 좋은 컨텐츠들이 많기 때문에 이를 활용하여 시작해보도록 하자. 

> "소프트웨어를 학습하는 좋은 방법 중 하나는 일단 무엇인가 만들어보는 경험을 한 후 이론적인 개념을 학습하고, 다시 다음 단계의 경험을 하고 이론적인 개념을 학습하는 과정을 반복하는 것이라 생각한다."

---

## 🚩 본격적인 웹 프로그래밍 도전

### 🔧 책을 통한 학습

자신에게 맞는 책을 고르는 것도 능력이고, 연습이 필요하다. 누군가 추천해준 책을 무조건 구매하기보다 자신이 직접 선택할 수 있는 능력을 기를 것을 추천한다.

> "일단 시작하지 않으면 내가 무엇을 모르는지조차 모른다."

---

## 🚩 학습 방법

기초부터 탄탄하게 쌓는 것도 하나의 학습 방법이지만, 경험을 통해 즐거움을 느낀 후 이론적인 학습을 하는 것 또한 좋은 학습 방법 중 하나이다. 

사람들은 내가 현재 학습하고 있는 지식이 어느 곳에 활용될 것인지 공감될 때 깊이 있게 몰입할 수 있으며, 체화할 수 있다.

일단 무엇이라도 시작해 프로그래밍, 컴퓨터에 대한 두려움을 깨야 한다. 내가 원하는 무엇인가를 만들어낼 수 있다는 것에 대한 즐거움과 흥분된 경험을 해봐야 한다. 이 즐거움과 흥분이 있어야 앞으로 두 번째, 세 번째 양파 껍질을 벗기면서 경험하게 될 큰 산들을 슬기롭게 넘길 수 있다.

---
---

# 📂 2장 문자열 계산기 구현을 통한 테스트와 리팩토링
> **"테스트와 리팩토링은 개발자가 갖추어야 할 중요한 역량이다."**

## 🚩 main 메소드를 활용한 테스트의 문제점

일반적으로 작성한 코드를 테스트하는 방법은 `main()` 을 실행시켜보는 것이다.

그리고 그 결과를 콘솔에서 확인한다.

```java
public class Calculator {
    int add(int x, int y) {
        return x + y;
    }
    int subtract(int x, int y) {
        return x - y;
    }
    int multiply(int x, int y) {
        return x * y;
    }
    int divide(int x, int y) {
        return x / y;
    }

    public static void main(String[] args) {
        Calculator cal = new Calculator();

        System.out.println(cal.add(3,4));
        System.out.println(cal.subtract(5,4));
        System.out.println(cal.multiply(2,6));
        System.out.println(cal.divide(8,4));
    }
}
```

하지만 위와 같은 방법에서 `main()`은

- **프로그래밍을 실행**하기 위한 목적과,
- 프로덕션 코드가 정상적으로 동작하는지 확인하는 **테스트** 목적으로 나뉘게 된다.

이 방식의 문제점은,

1. **프로덕션 코드와 테스트 코드가 같은 클래스에 위치하고 있다.**
   - 테스트 코드의 경우는 테스트에서만 필요한 것이므로, **서비스하는 시점에 함께 배포될 필요가 없다.**
   - 프로덕션 코드와 테스트 코드가 한 클래스에 존재하면, 이를 분리하지 않는 이상 함께 배포가 될 것이다.

→ 따라서 **`클래스 분리`** 를 통해 이를 해결할 수 있다.

(`Calculator` 클래스, `CalculatorTest` 클래스)

```java
public class CalculatorTest {
    public static void main(String[] args) {
        Calculator cal = new Calculator();

        System.out.println(cal.add(3,4));
        System.out.println(cal.subtract(5,4));
        System.out.println(cal.multiply(2,6));
        System.out.println(cal.divide(8,4));
    }
}
```

하지만 여전히 `main()`에서 프로덕션 코드의 **여러 메소드를 동시에 테스트**하는 형태일 것이다.

- 이러한 방식은 장기적으로 **프로덕션 코드의 복잡도가 증가**할수록, 
**`main()`의 복잡도 또한 증가**시키고, 결과적으로 **유지보수에 부담**을 주게 된다.

→ **테스트 코드를 메소드별로 분리**하여 해결할 수 있다.

```java
public class CalculatorTest {
    public static void main(String[] args) {
        // main에서는 함수 호출만 수행
        Calculator cal = new Calculator();

        add(cal);
        subtract(cal);
        multiply(cal);
        divide(cal);
    }

    private static void divide(Calculator cal) {
        System.out.println(cal.divide(8,4));
    }
    private static void multiply(Calculator cal) {
        System.out.println(cal.multiply(2,6));
    }
    private static void subtract(Calculator cal) {
        System.out.println(cal.subtract(5,4));
    }
    private static void add(Calculator cal) {
        System.out.println(cal.add(3,4));
    }
}
```

> 💡 하지만 이 또한 최종적인 해결책은 될 수 없다.

개발자는 프로그래밍 시, **한 번에 메소드 하나의 구현에 집중**한다.

- **“내가 현재 구현하고 있는 그 하나의 메소드”**

하지만 위의 테스트 코드는 `Calculator` 클래스가 가진 모든 메소드를 테스트할 수밖에 없다.

- 그렇다고 해서 매번 주석처리하는 것은 비효율적이다.

2. **테스트 결과를 매번 콘솔에 출력되는 값을 통해 수동으로 확인해야 한다.**
- **로직의 복잡도가 높아질수록**, 또는 **구현을 완료한 후 많은 시간이 지난 경우**에 해당 출력값이 과연 올바른 값일지 예측하는 건 매우 번거롭다.

> `main()`을 활용한 테스트의 이러한 문제점을 해결하기 위해 등장한 라이브러리가 **JUnit**이다.

---

## 🚩 JUnit을 활용해 main() 메소드 문제점 극복

단위 테스트 프레임워크 중 하나이다. 현재 버전 5.9.1까지 출시되어 있다.

- Jar을 제공하기에 간편하게 사용할 수 있다.

### 한 번에 메소드 하나에만 집중

JUnit은 테스트 메소드에 `**@Test**`를 추가한다.

```java
import org.junit.Test;

public class CalculatorTest {

    @Test
    public void add() {
        Calculator cal = new Calculator();
        System.out.println(cal.add(6, 3));
    }
}
```

위와 같이 JUnit 기반의 테스트 코드 구현으로 테스트 클래스가 가지는 **전체 메소드를 한 번에** 실행할 수도 있고, **각 메소드를 따로** 실행할 수도 있다.

**⇒ 이 각 메소드를 따로 실행할 수 있다는 점이 한 번에 메소드 하나에만 집중할 수 있도록 돕는 기능이다.**

### 결과 값을 눈이 아닌 프로그램을 통해 자동화

```java
import static org.junit.Assert.assertEquals;
import org.junit.Test;

public class CalculatorTest {

    @Test
    public void add() {
        Calculator cal = new Calculator();
        assertEquals(9, cal.add(6,3));
    }

    @Test
    public void subtract() {
        Calculator cal = new Calculator();
        assertEquals(3, cal.subtract(6,3));
    }
}
```

> `assertEquals()`는 `static` 메소드이기에 `import static`으로 사용할 수 있다.
> 

해당 메소드는 2개의 인자를 받는다.

- 첫번째는 해당 테스트로부터 **기대하는 결과 값(expected)**
- 두번째는 **실제 프로덕션 코드의 결과 값(actual)**

두 값의 비교를 통해 테스트의 성공 여부를 화면을 통해 알려준다.

- 테스트 실패 시에는 실패한 원인을 알려준다.

**이렇게 JUnit의 메소드를 활용해 수동으로 확인했던 실행 결과를 자동화할 수 있다.**

- 해당 메소드 이외에도 `assertTrue()`, `assertFalse()`, `assertNull()`, `assertArrayEquals()` 등이 있다.

### 테스트 코드 중복 제거

> **개발자가 가져야 할 좋은 습관 중 하나는 중복 코드를 제거하는 것이다.**

앞서 구현한 테스트 클래스의 중복을 제거해보자.

- `Calculator` 클래스 인스턴스를 중복적으로 생성하는 부분

```java
public class CalculatorTest {
    private Calculator cal = new Calculator();

    @Test
    public void add() {
        assertEquals(9, cal.add(6,3));
    }

    @Test
    public void subtract() {
        assertEquals(3, cal.subtract(6,3));
    }
}
```

- 하지만 JUnit은 테스트를 실행하기 위한 초기화 작업을 위와 같이 구현하는 것을 추천하지 않고, 아래와 같이 `@Before` 어노테이션을 활용해 구현하는 것을 추천한다.

```java
import static org.junit.Assert.assertEquals;

import org.junit.Before;
import org.junit.Test;

public class CalculatorTest {
    private Calculator cal;

    @Before
    public void setup() {
        cal = new Calculator();
    }

    @Test
    public void add() {
        assertEquals(9, cal.add(6,3));
    }

    @Test
    public void subtract() {
        assertEquals(3, cal.subtract(6,3));
    }
}
```

- 결과적으로 각 테스트 메소드를 호출할 때마다, `Calculator` 인스턴스를 생성하는 것은 같다.
    - 매 테스트마다 생성하는 이유는, **다른 테스트 메소드에 영향을 미치지 않기 위해서**이다.
    - 특정 테스트 수행 이후, 만약 `Calculator`의 상태 값이 변경된다면, 다음 테스트의 결과가 매번 달라질 수 있기 때문이다.

어노테이션을 사용해 구현하는 방법은 알았으나, 이전 방식보다 코딩량이 조금은 더 많아졌다.

- **근데 왜 추천하는 것일까?**

JUnit은 `@RunWith`, `@Rule`과 같은 어노테이션을 이용해 기능을 확장할 수 있다.

- 그리고 `@Before` 안이어야만 해당 어노테이션에서 초기화된 객체에 접근할 수 있다는 제약사항이 있다.
- 따라서 이러한 제약 사항 준수를 통해 추후 테스트에서의 문제 발생 가능성을 없앨 수 있도록 이 방식을 추천하는 것이다.

`@Before`이 각 테스트 메소드 실행 전 초기화 작업을 수행하듯, **테스트 메소드 실행이 끝난 후 후처리 작업**을 담당하는 `@After` 어노테이션 또한 존재한다.

> 👨‍💻 **이 두 어노테이션을 통해 매번 초기화, 후처리 작업을 수행해서 각 테스트 간 영향을 미치지 않으면서 독립적인 실행이 가능하도록 지원한다.**

---

## 🚩 문자열 계산기 요구사항 및 실습
### 요구사항
: 전달하는 문자를 구분자로 분리한 후 각 숫자의 합을 구해 반환해야 한다.
- 쉼표 또는 콜론을 구분자로 가지는 문자열을 전달하는 경우 구분자를 기준으로 분리한 각 숫자의 합을 반환
- 위의 기본 구분자 외에 Custom 구분자를 지정할 수 있다. Custom 구분자는 문자열 앞부분의 "//"와 "\n" 사이에 위치하는 문자로 사용
- 문자열 계산기에 음수를 전달하는 경우 `RuntimeException` 으로 예외 처리

간단해보이지만 구현을 바로 시작하지 않고, 요구사항을 더 작은 단위로 나눠 테스트할 경우의 수를 분리해본다.

### 요구사항 분리 및 각 단계별 힌트
- `StringCalculator` 클래스 : 실제 프로덕션 코드를 구현할 클래스
- `StringCalculatorTest` 클래스 : 테스트 코드를 구현할 클래스

또한, StringCalculator 클래스의 add()에 모든 요구사항을 구현한다.

<details><summary>StringCalculator & StringCalculatorTest</summary>

    import java.util.regex.Matcher;
    import java.util.regex.Pattern;

    public class StringCalculator {
        public int add(String text) {
            if (text == null || text.isEmpty()) {
                return 0;
            }

            String delimiter = ",|:";

            // custom delimiter가 존재하는지 정규표현식으로 확인
            Matcher matcher = Pattern.compile("//(.)\n(.*)").matcher(text);
            if (matcher.find()) {
                String customDelimiter = matcher.group(1);
                delimiter += "|" + customDelimiter;
                text = matcher.group(2);
            }

            int sum = 0;
            String[] tokens = text.split(delimiter);
            for (String token : tokens) {
                int number = Integer.parseInt(token);
                if (number < 0) {
                    throw new RuntimeException();
                }
                sum += number;
            }

            return sum;
        }
    }

    import org.junit.jupiter.api.BeforeEach;
    import org.junit.jupiter.api.Test;

    import static org.junit.jupiter.api.Assertions.assertEquals;
    import static org.junit.jupiter.api.Assertions.assertThrows;

    class StringCalculatorTest {

        private StringCalculator stringCalculator;

        @BeforeEach
        void setup() {
            stringCalculator = new StringCalculator();
        }

        @Test
        void add_null_또는_빈문자() {
            // given
            String str1 = null;
            String str2 = "";

            // when
            int result1 = stringCalculator.add(str1);
            int result2 = stringCalculator.add(str2);

            // then
            assertEquals(0, result1);
            assertEquals(0, result2);
        }

        @Test
        void add_숫자_하나() {
            // given
            String str = "2";

            // when
            int result = stringCalculator.add(str);

            // then
            assertEquals(2, result);
        }

        @Test
        void add_쉼표구분자() {
            // given
            String str = "1,23,4";

            // when
            int result = stringCalculator.add(str);

            // then
            assertEquals(28, result);
        }

        @Test
        void add_콜론구분자() {
            // given
            String str = "1,21:3:4";

            // when
            int result = stringCalculator.add(str);

            // then
            assertEquals(29, result);
        }

        @Test
        void add_customDelimiter() {
            // given
            String str = "//;\n1;2;3";

            // when
            int result = stringCalculator.add(str);

            // then
            assertEquals(6, result);
        }

        @Test
        void add_with_runtimeException() {
            // given
            String str = "-1:23,-24";

            // when
            assertThrows(RuntimeException.class, () -> stringCalculator.add(str));
        }
    }
</details>

### 추가 요구사항

소스코드 구현을 완료했다면, 중복을 제거하고, **읽기 좋은 코드**를 구현하기 위해 구조를 변경하는 **리팩토링이 필수적이다!**

> 👨‍💻 **리팩토링이란?**
> 소스코드의 가독성을 높이고 유지보수를 편하게 하기 위해 소스코드의 구조를 변경하는 것.
> - 기능상의 결과는 변경이 없어야 한다.

이 책에서 중요시하는 리팩토링 원칙.

> **메소드가 한 가지 책임만 가지도록 구현한다.**

> **인덴트 깊이를 1로 유지한다.**

> **else를 사용하지 마라.**

—

## 🚩 테스트와 리팩토링을 통한 문자열 계산기 구현


—

### 🔗 출처
[https://ko.wikipedia.org/wiki/JUnit](https://ko.wikipedia.org/wiki/JUnit)