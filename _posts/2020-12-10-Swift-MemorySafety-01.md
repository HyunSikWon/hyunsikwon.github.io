---
title: Swift - 메모리 안전성
layout: single
comments: true
share: true
categories: 
- Swift
tag:
- Memory Safety
last_modified_at: 2020-12-10 T15:42:00+08:00
---

## Memory Safety

Swift는 기본적으로 코드에서 발생하는 안전하지 않은 동작들을 방지한다. 

- **변수는 사용되기 전에 초기화 된다.**
- **해제된 메모리에는 접근하지 않는다.**
- **배열의 인덱스는 범위에서 벗어나지 않도록 검사된다.**

Swift는 메모리를 자동적으로 관리하기 때문에, 메모리 접근에 대해 크게 신경쓰지 않아도 되지만, 어디서 잠재적으로 충돌이 발생할 수 있는지 이해하고, 이를 예방할 수 있는 코드를 작성하는 것은 중요하다.

## Understanding Conflicting Access to Memory

Swift에서는 변수의 값을 저장하거나, 함수에 인자를 전달하는 것과 같은 경우에 메모리 접근이 발생한다. 예를들어 다음의 코드에는 *read access*와 *write access*가 포함된다.

```swift
// A write access to the memory where one is stored.
var one = 1

// A read access from the memory where one is stored.
print("We're number \(one)!")
```

메모리 접근 충돌은 코드 내에 **서로 다른 부분에서 같은 메모리 공간에 동시 접근할 때 발생**한다. 코드의 여러 줄에 걸친 값이 변경되는 상황에서 수정 중인 값에 메모리 접근이 발생하는 경우가 생길 수 있다.

종이에 적힌 예산을 업데이트 하는 상황을 가정해보자. 예산을 업데이트 하는 것은 두 단계를 거친다: 먼저, 아이템의 이름과 가격을 추가한다, 그리고 현재 총액을, 추가된 아이템을 포함하여 계산하고 변경한다. 다음 그림처럼, 업데이트 전, 후로는 예산에서 모든 정보를 읽을 수 있고, 정확한 답을 얻을 수 있다.

<img width="430" alt="memory_shopping_2x" src="https://user-images.githubusercontent.com/48352065/101731033-5f4e4000-3afe-11eb-9476-0cda94bbcc63.png">

반면에, 아이템을 추가하는 중에는 총액이 반영되지 않아 일시적으로 유효하지 않은 상황이 발생한다.  이 같은 상황은 메모리 접근 충돌 문제를 수정하며 직면할 상황이기도 하다.

### Characteristics of Memory Access

접근 충돌의 상황에서 고려야해야할 메모리 접근의 특징은 3가지가 존재한다: 

- 읽기위한 접근인지 쓰기 위한 접근인지 여부
- 접근 기간
- 메모리의 위치

먼저, **읽기 접근과 쓰기 접근의 차이**는 분명하다. 쓰기 접근은 메모리의 위치를 변경하지만, 읽기 접근은 그렇지 않다. **메모리의 위치**는 액세스중인 항목 (변수, 상수 또는 프로퍼티)를 나타낸다. 그리고 메모리 접근의 기간은 즉시 접근(instantaneous)과 장기 접근(long-term)으로 나뉜다.

- **즉시 접근(Instantaneous):** 접근이 시작되고 끝나기 전까지 다른 코드가 수행될 수 없다.
- **장기 접근(long-term):** 접근이 끝나기 전에도 다른 코드가 수행될 수 있다. 다른 코드의 메모리 접근과 겹치는(overlap) 문제가 발생할 수 있다.

 특히 다음 접근 조건 중 2가지가 만족하면 충돌이 발생한다.

- 최소 하나의 접근이 쓰기 접근이거나 nonatomic 접근인 경우.
- 메모리의 같은 공간에 접근할 경우.
- 메모리 접근 기간이 겹칠 때.

접근이 겹치는(overlapping) 것은 주로 함수나 메소드에서 `inout` 파라미터를 사용하거나 구조체에서 `mutating` 메소드를 사용할 때 발생한다. 

## Conflicting Access to In-Out Parameters

함수는 자신의 모든 `inout` 파라미터에대한 장기 쓰기 접근(ong-term write access)을 갖는다. `inout` 파라미터에 대한 쓰기 접근은 모든 non-in-out 파라미터의 값이 정해지면 시작되어 함수의 호출이 종료될 때까지 지속된다. 만약 여러개의 `inout` 파라미터가 존재하면 파라미터의 순서대로 쓰기 접근이 시작된다.

장기 쓰기 접근 때문에 `inout`으로 전달한 기존 변수에 접근할 수 없는 문제가 발생한다. 변수의 범위와 접근제어를 만족하더라도 접근할 수 없다.

```swift
var stepSize = 1

func increment(_ number: inout Int) {
    number += stepSize
}

increment(&stepSize)
// Error: conflicting accesses to stepSize
```

`stepSize`는 전역변수기 때문에 `increment(_:)` 에서 접근할 수 있지만, `stepSize` 에 대한 읽기 접근은 `number` 에대한 쓰기 접근과 겹치게 된다. 다음 그림과 같이, `number` 와 `stepSize`가 같은 메모리 공간을 참조하게 된다. 이로인해 충돌이 발생된다.

![memory_increment_2x](https://user-images.githubusercontent.com/48352065/101731029-5eb5a980-3afe-11eb-8e4b-1db0437da0e3.png)
 

이를 해결하는 방법은 복사본을 만드는 것이다.

```swift
// Make an explicit copy.
var copyOfStepSize = stepSize
increment(&copyOfStepSize)

// Update the original.
stepSize = copyOfStepSize
// stepSize is now 2
```

또한, 여러개의 `inout` 파라미터를 가진 함수에 하나의 변수를 사용하여 전달하는 경우에 생긴다.

```swift
func balance(_ x: inout Int, _ y: inout Int) {
    let sum = x + y
    x = sum / 2
    y = sum - x
}
var playerOneScore = 42
var playerTwoScore = 30
balance(&playerOneScore, &playerTwoScore)  // OK
balance(&playerOneScore, &playerOneScore)
// Error: conflicting accesses to playerOneScore
```

`balance(_:_:)`에서 두 개의 `inout` 파라미터 `playerOneScore`에 대해, 같은 메모리 공간을 동시에 쓰기 접근 하므로 충돌이 발생한다. 

## Conflicting Access to self in Methods

구조체에서 mutating 메소드는 메소드의 호출 기간동안 `self` 에 대한 쓰기 접근을 갖는다. 

```swift
struct Player {
    var name: String
    var health: Int
    var energy: Int

    static let maxHealth = 10
    mutating func restoreHealth() {
        health = Player.maxHealth
    }
}
```

위 코드의 `restoreHealth()` 메소드가 시작될 때, `self`에대한 쓰기 접근이 발생하고, 메소드가 리턴될 때까지 지속된다. 이 경우에는 `Player` 인스턴스의 프로퍼티에 접근하는 다른 코드가 있지 않아 충돌이 발생하지 않는다.

다음과 같은 코드를 추가하면, `Player` 인스턴스를 `inout` 파라미터를 사용하기 때문에 잠재적으로 메모리 접근이 겹칠 가능성이 생긴다.

```swift
extension Player {
    mutating func shareHealth(with teammate: inout Player) {
        balance(&teammate.health, &health)
    }
}

var oscar = Player(name: "Oscar", health: 10, energy: 10)
var maria = Player(name: "Maria", health: 5, energy: 10)
oscar.shareHealth(with: &maria)  // OK
```

물론, 위의 예시에서는 다음 그림과 같이 서로 다른 메모리 공간에 접근하기 때문에 문제가 발생하지 않는다.

![memory_share_health_maria_2x](https://user-images.githubusercontent.com/48352065/101731027-5e1d1300-3afe-11eb-9a54-e78a989e7c4d.png)

그러나, 만약 `oscar`를 `shareHeath(with:)` 메소드의 인자로 전달하는 경우에는 충돌이 발생한다.

```swift
oscar.shareHealth(with: &oscar)
// Error: conflicting accesses to oscar
```

`mutating` 메소드는 함수가 호출되는 기간동안 `self` 에 대한 쓰기 접근을 유지하고, 같은 기간동안에 `inout` 파라미터도 `teammate`에 대한 쓰기 접근이 발생하는데, 여기서 `self` 와 `teammate` 가 같은 메모리 공간을 참조하기 때문에 충돌이 발생하는 것이다.

![memory_share_health_oscar_2x](https://user-images.githubusercontent.com/48352065/101731022-5c534f80-3afe-11eb-8b65-a6e5c38a5b2e.png)

## Conflicting Access to Properties

구조체, 튜플, 열거형 같은 값 타입(vlau types)은 프로퍼티들의 조합으로 구성되거나(구조체), 요소들의 조합으로 구성되기(튜플) 때문에 특정 값의 변화는 전체 값의 변화를 의미한다. 즉, 하나의 프로퍼티에 대한 읽기 혹은 쓰기 접근은 전체 값에대한 접근을 의미한다는 것이다. 따라서 다음과 같은 코드는 충돌을 유발한다. 

```swift
var playerInformation = (health: 10, energy: 20)
balance(&playerInformation.health, &playerInformation.energy)
// Error: conflicting access to properties of playerInformation
```

다만, 실제 사용에 있어 대부분의 구조체 프로퍼티에 접근하는 것은 안전할 수 있다. 예를들어, 다음과 같이 전역 변수 대신 지역변수로 사용할 경우 컴파일러가 구조체의 프로퍼티에 접근하는 것이 안전하다고 판단하여 정상적으로 작업이 수행된다.

```swift
func someFunction() {
    var oscar = Player(name: "Oscar", health: 10, energy: 10)
    balance(&oscar.health, &oscar.energy)  // OK
}
```

Swift는 배타적이지 않은 메모리 접근에도 메모리 안전성이 보장된다는 것을 입증할 수 있으면, 이와 같은 접근을 허용하는데, 이는 다음와 같은 조건을 만족해야 한다.

- 오직 인스턴스의 저장 프로퍼티에만 접근하는 경우.
- 구조체가 지역변수의 값인 경우.
- 구조체가 클로저에 의해 캡쳐되지 않거나, 오직 non-escaping 클로저에만 캡쳐된 경우.

만약 컴파일러가 접근이 안전하다는 것을 입증하지 못하면, 비배타적 접근을 허용하지 않는다.

## Reference

[docs.swift.org](https://docs.swift.org/swift-book/LanguageGuide/MemorySafety.html)
