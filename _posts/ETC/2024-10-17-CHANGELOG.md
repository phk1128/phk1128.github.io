---
title: CHANGELOG.md 자동 생성하기
date: 2024-10-16
categories: [ ETC, git ]
tags: [ CHANGELOG, git, commit, log, 우테코 ]
image: assets/posts/img_64.png
---

## 🎬 Intro
> CHANGELOG.md 파일을 생성하는 방법을 알아봅시다.

- 우테코 7기 프리코스를 통해 CHANGELOG.md를 알게 되었습니다.
- 프로젝트 관리 시 용이한 방법 같아 정리해봅니다.

### ✅ CHANGELOG.md
`CHANGELOG.md`는 릴리즈 시점에 commit 메세지를 `type` 및 `scope` 별로 정리해놓은 마크다운 형식의 파일입니다.
따라서 프로젝트의 기능 추가, 버그 수정, 기타 변화를 한눈에 파악할 수 있습니다.

### ✅ 자동생성 과정 

#### 1. Node.js 설치 (macOS & Linux)

```shell
brew install node
```

#### 2. generate-changelog 설치 
```shell
#글로벌
npm i generate-changelog -g

#특정 프로젝트 종속
npm i generate-changelog -D
```
- generate-changelog는 Node.js 기반의 CHANGELOG.md를 자동으로 생성해주는 tool 입니다.
- 저는 다른 프로젝트에도 사용하고 싶어서 글로벌로 설치하였습니다.

#### 3. 프로젝트 루트 경로에 package.json 파일 생성
```json
{

}
```
- generate-changelog는 Node.js 기반의 tool 이므로 package.json이 없으면 오류가 발생합니다.
- 따라서 루트 경로에 package.json 생성 후 `{}` 를 작성합니다.

```text
Error: valid package.json not found
    at /usr/local/lib/node_modules/generate-changelog/lib/package.js:18:11
    at tryCatcher (/usr/local/lib/node_modules/generate-changelog/node_modules/bluebird/js/release/util.js:16:23)
    at Promise._settlePromiseFromHandler (/usr/local/lib/node_modules/generate-changelog/node_modules/bluebird/js/release/promise.js:547:31)
    at Promise._settlePromise (/usr/local/lib/node_modules/generate-changelog/node_modules/bluebird/js/release/promise.js:604:18)
    at Promise._settlePromise0 (/usr/local/lib/node_modules/generate-changelog/node_modules/bluebird/js/release/promise.js:649:10)
    at Promise._settlePromises (/usr/local/lib/node_modules/generate-changelog/node_modules/bluebird/js/release/promise.js:725:18)
    at _drainQueueStep (/usr/local/lib/node_modules/generate-changelog/node_modules/bluebird/js/release/async.js:93:12)
    at _drainQueue (/usr/local/lib/node_modules/generate-changelog/node_modules/bluebird/js/release/async.js:86:9)
    at Async._drainQueues (/usr/local/lib/node_modules/generate-changelog/node_modules/bluebird/js/release/async.js:102:5)
    at Async.drainQueues [as _onImmediate] (/usr/local/lib/node_modules/generate-changelog/node_modules/bluebird/js/release/async.js:15:14)
    at process.processImmediate (node:internal/timers:478:21)

```
- package.json이 없으면 위와 같은 오류가 발생합니다.


#### 4. changelog generate 명령어 실행

```shell
changelog generate
```
- 터미널 프로젝트 루트 경로에서 위 명령어를 실행해줍니다. 

#### 5. 생성파일 확인

![img.png](/assets/posts/img_63.png)

- 프로젝트 루트 경로에 CHANGELOG.md 파일이 생성되었습니다.



```markdown
#### 2024-10-16

##### Documentation Changes

*  일부 내용 수정 (9e9c61cb)
*  클래스 다이어그램 이미지로 변경 (42da92c4)
*  기능 구현 목록 최종본 작성 (c1fc2445)
*  진행 과정 및 플로우 차트 추가 (510bdb70)
*  LottoService 기능 구현 목록 작성 (2d1b583d)
*  PrizeResult 내용 수정 (e5a390e3)
*  PrizeQuantity -> PrizeResult로 워딩 수정 (04ef8206)
*  기능 구현 목록 초안 작성 (0bfd65c8)

##### New Features

*  사용자가 입력한 숫자가 9자리가 넘으면 예외처리 (ffdae9df)
*  금액이 음수 일때 예외처리 추가 (69603c3b)
*  유틸성을 위해 ErrorMessage 분리 (95c37a1b)
*  유틸성을 위해 Constant 분리 (92889843)
*  유틸성을 위해 Validator분리 (13711381)
*  유틸성을 위해 Convertor 따로 분리 (43e042f6)
*  당첨번호와 일치하는 숫자 갯수 카운팅 기능, 보너스 번호 포함여부 확인 기능 추가 (b8eb3b9c)
*  setup precourse lotto project (666a0907)
* **LottoController:**  LottoController 구현 완료 (66d39e99)
* **InputView:**  InputView 구현 완료 (d99a3fc0)
* **OutputView:**  OutputView 구현 완료 (093996aa)
* **LottoService, LottoServiceTest:**  LottoService 구현 및 테스트 완료 (a6e8f5c6)
* **PrizeResult, PrizeResultTest:**  PrizeResult 도메인 구현 및 테스트 완료 (532d0ba4)
* **Prize, PrizeTest:**  Prize 도메인 구현 및 테스트 완료 (93e7952e)
* **Bonus, BonusTest:**  Bonus 도메인 구현 및 테스트 완료 (de4b9158)
* **Winning, WinningTest:**  Winning 도메인 구현 및 테스트 완료 (0c2d90f8)
* **Money, MoneyTest:**  Money 도메인 구현 및 테스트 완료 (44772d42)
* **LottoQuantity, LottoQuantityTest:**  LottoQuantity 도메인 구현 및 테스트 완료 (e8b3cbe0)
* **Lotto, LottoTest:**  Lotto 도메인 구현 및 테스트 완료 (77398e3c)

##### Other Changes

*  일급 컬렉션의 불변성을 보장하기 위해 코드 개선 (8d407369)
*  수익률 계산 및 출력 코드 수정 (02dc7a30)

##### Refactors

* **LottoService, LottoController:**
  *  Money를 생성하는 부분 이동 LottoController -> LottoService (a735015c)
  *  LottoService의 dcl제거로 인한 LottoController코드 수정 (7b1e8f04)
*  출력 메세지 하드 코딩 부분 필드 변수로 분리 (3093d563)
*  Convertor, Validator의 dcl 제거로 인한 코드 수정 (272bb66a)
*  LottoController는 stateless가 아니므로 dcl 제거 (c73ff74c)
*  유효성 검증 시 재할당 방지를 위해 각 매개변수에 final 키워드 추가 (8b6c2d75)
*  보너스 번호 재할당 방지를 위해 final 키워드 추가 (48274f33)
*  에러 메세지 하드 코딩 부분 개선 (d5e2d601)
*  숫자에 공백이 포함 되었을때 공백 제거 (af64703d)
*  coverte, validate 기능 Convertor, Validator로 이동 (8b3f5b7a)
* **LottorService, LottoController:**  LottoService에 dcl 적용 및 해당 부분 LottoController에 반영 (4f312de5)
* **Money, Validator:**  상수화 진행 (dbca4154)

##### Code Style Changes

*  formatting (3c078f3d)

##### Tests

*  App 최종 테스트 완료 (ef270d28)
*  금액이 음수인 경우 예외발생 테스트 완료 (fe777cb4)
* **MoneyTest, LottoSerivceTest:**  수익률 부분 다시 테스트 (2d8bd818)


```

- 파일 내용을 살펴보면 commit 메세지 type 및 scope 별로 나누어져 작성된것을 볼 수 있습니다.

```markdown
New Features (feat): 새롭게 추가된 기능을 나타내며, 새로운 기능이 포함된 커밋이 여기에 속합니다.

Bug Fixes (fix): 버그가 수정된 내용을 기록합니다.

Documentation (docs): 문서와 관련된 변경 사항을 기록합니다.

Performance Improvements (perf): 성능을 개선한 경우 여기에 포함됩니다.

Refactors (refactor): 코드 리팩토링으로 기능적인 변화는 없지만 코드 구조를 개선한 경우 기록됩니다.

Build (build): 빌드 관련된 시스템 변경이나 패키지 종속성 관련 변경을 기록합니다.

Continuous Integration (ci): CI 설정 파일이나 스크립트와 관련된 변경 사항을 나타냅니다.

Tests (test): 테스트 코드와 관련된 추가나 수정 사항을 기록합니다.

Chore (chore): 프로젝트 관리에 필요한 사소한 변경 사항(예: 코드 스타일 변경, 패키지 업데이트 등)을 기록합니다.

Reverts (revert): 이전에 커밋된 변경 사항을 되돌린 경우 기록됩니다.
```
- 각 섹션에 대한 설명 입니다.

## 📚 참고
> [AngularJS Git Commit Message Conventions](https://gist.github.com/stephenparish/9941e89d80e2bc58a153)
> 
> [우테코 7기 프리코스 1주차 미션](https://github.com/woowacourse-precourse/java-calculator-7)
>
> [우아한테크코스 4기 프리코스 후기 (2) - Commit Log 컨벤션](https://creampuffy.tistory.com/129)
