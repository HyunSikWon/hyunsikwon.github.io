---
title: Swift - Associated Type이란 무엇인가??!
layout: single
comments: true
share: true
categories:
- Swift
tag:
- Associated Type
last_modified_at: 2020-12-01 T22:36:00+08:00

---


iOS 개발 공부 중 `associatedtype` 을 사용하게 되었는데, 이를 이해하고 공부하기 위해서 Swift Document를 통해 공부했다.

## Associated Type

프로토콜을 정의할 때, 몇몇을 associated type으로 선언하면 매우 유용할 때가 있다. *associated type* 은 프로토콜의 일부로 사용되는 타입을 위한 placeholder 역할을 한다. associated type은 정의하는 프로토콜이 채택되기 전까지 실제 타입이 명시되지 않는다. Associated type은 `associatedtype` 키워드를 사용해서 정의한다.

## Associated Types in Action

여기 `Container` 프로토콜이 associated type의 `Item` 과 함께 정의되어 있다. 

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
```

이 프로토콜은 3가지의 요구사항을 정의한다.

1. `append(_:)` 메소드를 통해서 컨테이너에 새로운 항목를 추가할 수 있어야한다. 
2. `Int` 타입을 리턴하는 `count` 프로퍼티를 통해서 컨테이너 들어있는 항목의 개수를 알 수 있어야한다.
3. `Int` index 값을 취하는 subscript를 통해서 컨테이너의 각각의 항목을 가져올 수 있어야한다.

 `Container` 프로토콜은 항목들이 컨테이너에 어떻게 저장되어야 하는지, 어떤 타입이어야 하는지는 정의하고 있지 않고, 3가지 필수 요구사항만 정의한다. 이 프로토콜을 준수하는 타입은 세가지 필수 요구사항만 준수하면 원하는 기능을 더할 수 있다.

  `Container` 프로토콜을 **준수하는 모든 타입은,** 저장하는 값의 타입을 명시해야 하고, 이 값은 컨테이너에 추가될 수 있는 타입임을 보장해야한다. 뿐만 아니라 subscript를 통해 반환될 수 있는 타입이어야 함도 보장해야한다. 이를 위해서, `Container` 프로토콜은 구체적인 컨테이너의 타입을 알지 못하더라도, 컨테이너가 가질 요소들의 타입을 언급할 필요가 있다. 다시 말해, `Container` 프로토콜은 `append(_:)` 메소드를 통해 전달되는 값이 컨테이너의 요소와 같은 타입이어야 한다는 것과 컨테이너의 subscript 역시 컨테이너의 요소와 같은 타입이어야 한다는 것을 명시해야한다. **이를 위해서 `Container` 프로토콜은 associated type인 Item을 활용하는 것이다.**

예제 코드:

```swift
struct IntStack: Container {
    // original IntStack implementation
    var items = [Int]()
    mutating func push(_ item: Int) {
        items.append(item)
    }
    mutating func pop() -> Int {
        return items.removeLast()
    }
    // conformance to the Container protocol
    typealias Item = Int
    mutating func append(_ item: Int) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Int {
        return items[i]
    }
}
```

`IntStack` 타입은 `Container` 프로토콜을 체택하고 세 가지 필수 요구사항을 준수하고 있고, `associatedtype`인 `Item` 사용하기 위해 Int 타입을 사용하고 있다. `typealias Item = Int` 는 프로토콜을 준수하기 위해서 추상 타입인 `Item`을 `Int` 로 바꿔 사용하기 위한 구문이다. Swift의 타입 추론 덕분에 이 구문은 생략 가능하다.

generic `Stack` 타입을 통해서도 `Container` 프로토콜을 준수할 수 있다.

```swift
struct Stack<Element>: Container {
    // original Stack<Element> implementation
    var items = [Element]()
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
    // conformance to the Container protocol
    mutating func append(_ item: Element) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Element {
        return items[i]
    }
}
```

## Adding Constraints to an Associated Type

Associated type에 제약조건을 줄 수 있다. 예를들어, 다음 코드는 컨테이너의 항목들이 `equatable`해야 하는 `Container` 프로토콜을 정의한다.

```swift
protocol Container {
    associatedtype Item: Equatable
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
```

이 프로토콜을 준수하기 위해서, 컨테이너의 `Item` 타입은 `Equatable` 프로토콬ㄹ을 준수해야 한다.

## Using a Protocl in Its Associated Type's Constraints

프로토콜은 자기 자신의 요구사항의 일부로 표현될 수 있다. 예를들어, 다음은 `Container` 프로토콜에 `suffix(*:)`*  메소드를 추가하여 기존의 프로토콜을 개선한 코드이다. **`suffix(_:)`는 컨테이너의 끝에서부터 주어진 개수의 요소를 반환하여, `Suffix` 타입의 인스턴스에 저장하는 메소드이다. 

```swift
protocol SuffixableContainer: Container {
    associatedtype Suffix: SuffixableContainer where Suffix.Item == Item
    func suffix(_ size: Int) -> Suffix
}
```

이 프로토콜에서 `Suffix` 는 associated type 이고, 두가지 제약조건을 가진다. 먼저 `SuffixableContainer` 프로토콜을 준수해야하고, 그것의 `Item` 은 컨테이너의 Item 타입과 같아야한다. 

다음은 `SuffixableContainer` 프로토콜을 준수한  `Stack` 타입의 익스텐션 코드이다.

```swift
extension Stack: SuffixableContainer {
    func suffix(_ size: Int) -> Stack {
        var result = Stack()
        for index in (count-size)..<count {
            result.append(self[index])
        }
        return result
    }
    // Inferred that Suffix is Stack.
}
var stackOfInts = Stack<Int>()
stackOfInts.append(10)
stackOfInts.append(20)
stackOfInts.append(30)
let suffix = stackOfInts.suffix(2)
// suffix contains 20 and 30
```

이 코드에서 `Stack`의 `Suffix` associated type은 또한 `Stack` 이고, 따라서 `Stack`의suffix 작업은 또 다른 `Stack` 을 반환한다.

## Reference

[The Swift Programming Guide](https://docs.swift.org/swift-book/LanguageGuide/Generics.html)
