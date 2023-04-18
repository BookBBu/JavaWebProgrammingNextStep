# 📕 JavaWebProgrammingNextStep
**자바 웹 프로그래밍 Next Step**을 읽고, 생각을 공유합시다.

## 🚩 진행 방식
**정기 회의** : 매주 1회 (팀별 조율)

### 커밋 규칙
| 1. **Repository clone**
```bash
git clone https://github.com/BookBBu/JavaWebProgrammingNextStep.git
```

| 2. **Repository open**
- vscode

| 3. **발표자 본인의 Branch 생성**
- branch는 주차별, 팀별로 생성한다.

```bash
git checkout -b {본인의 깃허브 이름}/{주차}week/{팀 번호}team
```
> ex. git checkout -b SUbbb/1week/1team

| 4. **주차별 디렉토리 생성 및 README 저장**
```
{책 이름}/{챕터}장/{팀 번호}team/README.md
```
> ex. JavaNextStep/1~2장/1team/README.md

| 5. **Push**
```bash
git add .
git commit -m "[{주차}week/{팀 번호}team] 발표자 본인의 깃허브 이름 : {챕터}장 챕터명 또는 수정 사항"
git push origin {생성한 브랜치}
```

> ex. git commit -m "[1week/1team] SUbbb : 1장 첫 번째 양파 껍질 벗기기"

> ex. git commit -m "[1week/1team] SUbbb : 2장 문자열 계산기 구현을 통한 테스트와 리팩토링 2.3 추가"

| 6. **Pull request 생성**
- Pull Request Name : [{주차}week/{팀 번호}team] {본인의 깃허브 이름} : {진행한 챕터}장
  > ex. [1week/1team] SUbbb : 1 ~ 2장
- Content : 해당 주차 내용을 간략하게 2-3줄 요약
- Assignees : 본인
- Label : 스터디 진행 도서

| 7. **스터디 후, merge**

| 8. **공동 노션에 질문, 답변, 궁금한 점 등 기재**

### ✏ 리뷰 규칙
스터디 시작 전, 최대한 많은 생각과 고민들을 자유롭게 남깁시다!

### 📆 일정

|주차|Chapter|내용|
|:---:|:---:|:---:|
|1주차|**1 & 2장**<br>|첫번째 양파 껍질 벗기기 & 문자열 계산기 구현을 통한 테스트와 리팩토링|
|2주차|**3장**<br>|개발 환경 구축 및 서버 실습 요구사항|
|3주차|**4장**<br>|HTTP 웹 서버 구현을 통해 HTTP 이해하기|
|4주차|**5장**<br>|웹 서버 리팩토링, 서블릿 컨테이너와 서블릿의 관계|
|5주차|**6장**<br>|서블릿/JSP를 활용해 동적인 웹 애플리케이션 개발하기|
|6주차|**7장**<br>|DB를 활용해 데이터를 영구적으로 저장하기|
|7주차|**8장**<br>|AJAX를 활용해 새로고침 없이 데이터 갱신하기|
|8주차|**9장**<br>|두번째 양파 껍질을 벗기기 위한 중간 점검|
|9주차|**10장**<br>|새로운 MVC 프레임워크 구현을 통한 점진적 개선|
|10주차|**11장**<br>|의존관계 주입(DI)을 통한 테스트하기 쉬운 코드 만들기|
|11주차|**12장**<br>|확장성 있는 DI 프레임워크로 개선|
