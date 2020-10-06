---
title: iOS - Unit Test
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- Unit Test
last_modified_at: 2020-10-06 T15:23:00+08:00

---

iOS 환경에서의 Unit Test를 공부해보자!


## Figuring Out What to Test

테스트를 다루기 전에 기초를 다지는 것이 중요하다. 

만약, 기존의 앱을 확장하는 것이 목표라면 변경할 모든 요소들에 대해서 테스트를 작성해야 한다. 

일반적으로 테스트는 다음과 같은 것들을 다룬다:

- 핵심 기능: 모델 클래스, 메소드 그리고 이들과 controller와의 상호작용.
- 가장 흔한 UI 흐름(Workflows)
- 경계 조건
- 버그 수정

## Best Practices for Testing

두문자어 `FIRST` 는 효율적인 테스트를 위한 간결한 기준들을 설명한다.

- `Fast` : 테스트는 빠르게 진행되어야 한다.
- `Independent/Isolated` : 테스트는 서로 상태를 공유해선 안된다.
- `Repeatable` : 테스트는 수행할 때마다 항상 같은 결과를 내야한다. 외부 데이터 제공자와 동시성 문제는 일시적인 오류를 유발할 수 있다.
- `Self-validating` : 테스트는 완전히 자동이어야 한다. 로그 파일에 대한 개발자의 해석에 의존하기 보단 결과는 항상 "성공" 혹은 "실패" 가 되어야 한다.
- `Timely` : 이상적으로, 테스트는 프로덕션 코드를 작성하기 전에 작성되어야 한다. (Test-Driven Development)

`FIRST` 원칙을 따르면 깔끔하고 유용한 테스트가 유지될 수 있다.

# Getting Started

`BullsEye` 프로젝트로 테스트를 진행할 것이다.

## Unit Testing in Xcode

**Test Navigator**는 테스트를 위한 가장 쉬운 방법을 제공한다. 이를 이용해 테스트 target을 생성하고 테스트를 실행할 수 있다.

Xcode 14 부터는 자동으로 생성해주는 듯 하다.

## Creating a Unit Test Target

프로젝트를 열고 `command+6` 을 눌러 Test navigator을 열어라. 왼쪽 아래의 `+` 버튼을 누른 후 `New Unit Test Target…` 를 선택한다.

![TestNavigator1](https://user-images.githubusercontent.com/48352065/95166014-13260d00-07e8-11eb-97b1-e24989faadbb.png)

프로젝트 이름+Tests 라는 Default 이름이 생성된다. 테스트 번들을 열어보자. 자동으로 번들이 생성되지 않으면 다른 navigator에 갔다가 돌아오면된다.

![TestNavigator2](https://user-images.githubusercontent.com/48352065/95166013-13260d00-07e8-11eb-9752-afe444455312.png)


기본 템플릿은 XCTest framework를 import하고 XCTestCase의 하위 클래스를 정의한 후, `setUpWithError(),tearDownWithError()` 와 함께 예시 테스트 메소드를 정의한다.

테스트를 실행하기 위한 방법은 3가지가 있다.

- `Product -> Test` 혹은 `command+U` :  모든 테스트 클래스를 실행한다.
- 테스트 네비게이터의 화살표 버튼을 클릭.
- gutter의 다이아몬드 버튼을 클릭. gutter은 코드 라인 번호가 적인 곳.


![TestNavigator3](https://user-images.githubusercontent.com/48352065/95166012-128d7680-07e8-11eb-92d3-285c365c6469.png)


또한, 다이아몬드 버튼을 눌러 테스트 메소드를 따로 실행할 수 있다. 3가지 방식으로 테스트를 실행시켜 보자.

모든 테스트가 성공적으로 끝나면 다이아몬드는 체크 마크와 초록색이 채워진다. `testPerformanceExample()` 의 끝에 있는 회색 다이아몬드를 누르면 수행 결과를 볼 수 있다,

![TestNavigator4](https://user-images.githubusercontent.com/48352065/95166011-11f4e000-07e8-11eb-8789-b7d41e634a4f.png)


## Using XCTAssert to Test Models

먼저 `XCTestAssert` 를 이용해서 프로젝트 모델의 핵심 기능을 테스트 할 것이다.  `BullsEyeGame` 객체는 한 라운드의 점수를 정확하게 계산하는지 테스트 해보자.

`BullsEyeTests.swift`의 import문 아래에 다음과 같은 코드를 추가하자.

```swift
@testable import BullsEye
```

이는 unit 테스트가 BullsEye의 내부 타입과 함수에 접근할 수 있게 해준다.

`BullsEyeTests` 클래스 맨 위에 다음의 프로퍼티를 추가하자.

```swift
var sut: BullsEyeGame!
```

위 코드는 BullsEyeGame을 위한 placeholder를 생성하고 이는 `SUT(System Under Test)`, 즉 테스트와 연관된 테스트 케이스 클래스 객체이다.

이제 `setUpWithError()` 에 다음 코드를 추가하자.

```swift
super.setUp()
sut = BullsEyeGame()
sut.startNewGame()
```

이는 클래스 수준의 `BullsEyeGame` 객체 생성하고 이로인해, 이 테스트 클래스의 모든 테스트는 SUT객체의 프로퍼티와 메소드에 접근할 수 있다.

여기서는 `startNewGame()` 을 호출하여 `targetValue` 를 초기화 한다. 많은 테스트에서 게임이 점수를 정확히 계산하는지 테스트하기 위해 `targetValue` 를 사용할 것이다.

또한 잊지말고 SUT 객체를 release 해주자. `tearDownWithError()` 에 다음을 추가하자:

```swift
sut = nil
super.tearDown()
```

## Writing Your First Test

이제 본격적으로 테스트를 작성해보자!

`BullsEyeTests` 마지막에 다음 코드를 추가하자:

```swift
func testScoreIsComputed() {
  // 1. given
  let guess = sut.targetValue + 5

  // 2. when
  sut.check(guess: guess)

  // 3. then
  XCTAssertEqual(sut.scoreRound, 95, "Score computed from guess is wrong")
}
```

테스트 메소드의 이름은 항상 **test**로 시작해야하고, 무엇을 테스트 하는지에 대한 설명이 있어야 한다.



`Given` , `When` , `Then`  섹션으로 테스트 형식을 구성하는 것은 좋은 연습이 될 것이다.

1. `Given` : 여기선 필요한 값들을 지정할 것이다. 이 예제에선 `guess` 값을 생성하여  `targetValue` 와 얼만큼의 차이가 있는지 알 수 있다.
2. `When` : 여기선 테스트될 코드를 수행한다: `Call(guess:)` 를 호출한다.
3. `Then` : 이 섹션은 기대하는 결과와 테스트 실패시 출력된 메시지를 선언할 섹션이다. 예제의 경우 `sut.scoreRound` 는 항상 95가 되어야 한다.

이 게임은 100에서 targetValue와 guess 값의 차만큼을 뺀 값이 한 라운드의 점수로 주어진다.

ex) targetValue = 98, guess 90 → roundScore = 92


이제 다이아몬드 버튼을 눌러 테스트를 실행해보자. 빌드하고 앱을 실행시킬 것이며, 다이아몬드 아이콘은 초록색 체크로 변할 것이다.

## Debugging a Test

`BullsEyeGame` 에는 사실 의도된 버그가 있고, 이를 찾는 연습을 할 것이다. 먼저 버그를 보기 위해서 `targetValue` 에서 5를 빼는 테스트를 생성할 것이다.

다음 테스트를 추가하자:

```swift
func testScoreIsComputedWhenGuessLTTarget() {
  // 1. given
  let guess = sut.targetValue - 5

  // 2. when
  sut.check(guess: guess)

  // 3. then
  XCTAssertEqual(sut.scoreRound, 95, "Score computed from guess is wrong")
}
```

 

여기서도 `guess` 와 `targetValue` 는 5만큼 차이가 나고 따라서 점수는 95가 되어야 한다.

`Breakpoint Navigator` 에서 `Test Failure Breakpoint` 를 추가하자. 이제 테스트 메소드가 실패할 경우  테스트 실행이 중단된다.


![AddTestFailureBreakpoint](https://user-images.githubusercontent.com/48352065/95166009-115c4980-07e8-11eb-852a-e003ec1ca447.png)


테스트를 실행하면 `XCAssertEqual` 라인에서 멈출 것이다.

`sut` 과 `guess` 를 디버그 콘솔에서 확인해보자:

<img width="694" alt="TestFailure-1" src="https://user-images.githubusercontent.com/48352065/95166006-0f928600-07e8-11eb-98a9-b268143b2b17.png">


`guess` 는 `targetValue-5` 인데 `scoreRound` 는 95가 아니라 105다!

더 자세히 살펴보기 위해서, 일반적인 디버깅 프로세스를 사용하자: `when` 에 breakpoint를 설정하고 `BullsEyeGame.swift` 내부에 있는  `check(guess:)` 의 `difference` 를 생성하는 곳에도 breakpoint를 지정하자. 테스트를 다시 실행하고 `difference` 의 값을 살펴보기 위해  `let difference` 가 생성되는 과정을 단계별로 살펴보자:

![DebugConsole](https://user-images.githubusercontent.com/48352065/95165999-0dc8c280-07e8-11eb-8aa3-d260c3035d9a.png)


문제는 `difference` 가 음수가 되어 점수가 100 - (-5) 로 계산되는 것이다. 이를 고치기 위해 `difference` 의 절댓값을 구하도록 해야한다. `abs()` 를 이용해 수정하자.

breakpoint를 제거하고 테스트를 다시 실행하면 테스트가 성공적으로 진행되는 것을 볼 수 있다.

## Reference

[raywenderlich](https://www.raywenderlich.com/960290-ios-unit-testing-and-ui-testing-tutorial)
