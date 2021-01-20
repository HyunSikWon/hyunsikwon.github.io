---
title: Swift 고차함수(Map, Filter, Reduce, compactMap, flatMap)
layout: single
comments: true
share: true
categories: 
- Swift
tag:
- Functions
last_modified_at: 2021-01-20 T13:35:00+08:00
---


스위프트 표준 라이브러리의 대표적인 고차함수(Higher-order function)를 알아보자. 고차함수란 매개변수로 함수를 갖는 함수를 말한다.

## **map(_:)**

맵은 자신을 호출할 때 매개변수로 전달된 함수를 실행하여 그 결과를 다시 반환해주는 함수이다. 스위프트의 `Sequence`, `Collection` 프로토콜을 따르는 타입은 모두 맵을 사용할 수 있다. 맵을 사용하면 컨테이너가 담고 있던 각각의 값을 매개변수를 통해 받은 함수에 적용한 후 다시 컨테이너에 포장하여 반환한다. 기존의 컨테이너의 값은 변경되지 않고 새로운 컨테이너가 생성되어 반환된다. 

```swift
let cast = ["Vivien", "Marlon", "Kim", "Karl"]
let lowercaseNames = cast.map { $0.lowercased() }
// 'lowercaseNames' == ["vivien", "marlon", "kim", "karl"]
let letterCounts = cast.map { $0.count }
// 'letterCounts' == [6, 6, 3, 4]
```

## **filter(_:)**

필터는 말 그대로 컨테이너의 내부의 값을 걸러서 추출하는 역할을 한다. 맵과 마찬가지로 새로운 컨테이너의 값을 담아 반환한다. 다만 맵처럼 기존 컨텐츠를 변형하지 않고 특정 조건에 맞게 걸러내는 역할을 한다. 이 함수의 매개변수를 전달되는 함수의 반환 타입은 `Bool` 이다.

```swift
let cast = ["Vivien", "Marlon", "Kim", "Karl"]
let shortNames = cast.filter { $0.count < 5 }
print(shortNames)
// Prints "["Kim", "Karl"]"
```

## **reduce(_ :_ :)**

리듀스는 컨테이너 내부의 컨텐츠를 하나로 합하는 기능을 실행한다. 배열이라면 배열의 모든 값을 전달인자로 받은 클로저의 연산 결과로 합해준다. 스위프트의 리듀스는 두 가지 형태로 구현되어 있다. 첫번째 리듀스는 클로저가 각 요소를 전달받아 연산한 후 값을 다음 클로저 실행을 위해 반환하며 컨테이너를 순환하는 형태이다. 두번째 리듀스는 컨테이너를 순환하며 클로저가 실행되지만, 클로저가 따로 결과값을 반환하지 않는 형태이다. 대신 `inout` 매개변수를 사용하여 초기값에 직접 연산을 실행한다.

```swift
let numbers = [1, 2, 3, 4]
let numberSum = numbers.reduce(0, { x, y in
    x + y
})
// numberSum == 10
```

## **compactMap(_:)**

컴팩트 맵은 매개변수로 전달된 클로저가 옵셔널 값을 생산할때(produce) 사용한다. 시퀀스의 각 요소를 전달받은 클로저에 적용하고 `nil`이 아닌 값들만 배열 추가한 후 반환한다. 

`map(_:)`과 `compactMap(_:)`의 차이점

```swift
let possibleNumbers = ["1", "2", "three", "///4///", "5"]

let mapped: [Int?] = possibleNumbers.map { str in Int(str) }
// [1, 2, nil, nil, 5]

let compactMapped: [Int] = possibleNumbers.compactMap { str in Int(str) }
// [1, 2, 5]
```

## **flatMap(_:)**

플랫 맵은 시퀀스의 각 요소들에 매개변수로 전달된 클로저을 적용하여 연속적인(concatenated) 값을 가지는 배열을 반환한다. 매개변수 클로저가 각 요소에 대한 시퀀스 혹은 컬렉션을 생산할때(produce), 일차원 컬렉션을 얻고자 사용한다.  

`map(_:)`과 `flatMap(_:)`의 차이점

```swift
let numbers = [1, 2, 3, 4]

let mapped = numbers.map { Array(repeating: $0, count: $0) }
// [[1], [2, 2], [3, 3, 3], [4, 4, 4, 4]]

let flatMapped = numbers.flatMap { Array(repeating: $0, count: $0) }
// [1, 2, 2, 3, 3, 3, 4, 4, 4, 4]
```

## Reference
Apple Developer Documentation

스위프트 프로그래밍 - 야곰

