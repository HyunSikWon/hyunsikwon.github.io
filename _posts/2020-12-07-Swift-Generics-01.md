---
title: Swift - 제네릭을 알아보자!!
layout: single
comments: true
share: true
categories:
- Swift
tag:
- Generics
last_modified_at: 2020-12-07 T22:00:08+08:00
---

## Generics

*Generic code*는 유연하고 재사용 가능한 함수와 타입을 작성할 수 있게 해준다. 중복 작성을 피하고, 의도가 명확하고 추상화된 방식으로 코드를 작성할 수 있다.

## The Problem That Generics Solve

다음 코드는 두 정수 값을 바꾸는 전형적인 nongeneric 함수 `swapTwoInts(_:_:)` 이다.

```swift
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

이 함수는 매우 유용하고 기능에는 전혀 문제가 없지만, 오직 `Int` 값만 사용할 수 있다. 

만약 `String`, `Double` 타입을 swap 하기 위해선, 다음과 같은 새로운 함수를 만들어야 한다.

```swift
func swapTwoStrings(_ a: inout String, _ b: inout String) {
    let temporaryA = a
    a = b
    b = temporaryA
}

func swapTwoDoubles(_ a: inout Double, _ b: inout Double) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

그런데 세 함수 `swapTwoInts(_:_:)`, `swapTwoStrings(_:_:)`, `swapTwoDoubles(_:_:)` 의  본체는 완전히 같으며 세 함수의 유일한 차이는, 파라미터로 받는 값의 타입 뿐이다.

어떤 타입이든 두개의 값을 swap 하는 **하나의 함수만 작성하는 것**이 더 유용하고 유연한 코드 작성이라는 것을 모두다 알 것이다. Generic을 사용하면 이를 해결할 수 있다.

## Generic Functions

Generic functions은 타입의 제한 없이 사용가능하다. 다음의 예제코드를 확인해보자.

```swift
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

`swapTwoValues(_:_:)` 함수의 본체는 위에서 본 함수들과 동일하지만, 함수 선언부에 약간의 차이가 있다. 비교해보자.

```swift
func swapTwoInts(_ a: inout Int, _ b: inout Int)
func swapTwoValues<T>(_ a: inout T, _ b: inout T)
```

제네릭 버전의 함수는 `Int`, `Double` 같은 실제 타입 대신 *placeholder* 타입을 사용한다.(예제의 경우 `T`). 여기서 *placeholder* 는 `T` 가 무엇이어야 하는지는 알려주지 않지만, 두 개의 파라미터는 모두 타입 `T` 로 같아야 한다는 점은 알려준다.

제네렉 함수 `swapTwoValues(_:_:)`는 다음과 같이 이제 다양한 타입과 함께 사용할 수있다.

```swift
var someInt = 3
var anotherInt = 107
swapTwoValues(&someInt, &anotherInt)
// someInt is now 107, and anotherInt is now 3

var someString = "hello"
var anotherString = "world"
swapTwoValues(&someString, &anotherString)
// someString is now "world", and anotherString is now "hello"
```

## Type Parameters

예제에서 본 `swapTwoValues(_:_:)` 함수의 `T` 는 *type parameter* 의 예이다. *type parameter*는 placeholder type을 명시하고, 함수의 이름 뒤에 바로 작성된다. 타입 파라미터를 명시하면, 해당 함수의 파라미터 타입으로 즉시 정의 가능하다. 이는 함수의 리턴 타입, 혹은 함수 본체에서 타입으로 사용될 수 있다. `,` 로 구분하여 여러 타입 파라미터를 정의할 수도 있다.

## Naming Type Parameters

많은 경우에 타입 파라미터는, `Dictionary<Key, Value>`, `Array<Element>` 처럼 타입 파라미터와 제네릭 타입 혹은 함수사이의 관계를 알려주기 위해 서술식 이름을 갖는다. 다만, 의미있는 관계가 없는 경우에는 `T`, `U` 와 같이 한 글자로 표기한다.

## Generic Tpyes

 함수와 더불어, 사용자 정의 제네릭 타입을 정의할 수 있다. 이는 `Array`,`Dictionary` 처럼, 어느 타입과 함께 사용할 수 있는 커스텀 클래스, 구조체, 열겨형이다.

`Stack` 자료구조를 제네릭 컬렉션 타입으로 작성해보자. 

다음은 논제네릭 버전의 스택이다.

```swift
struct IntStack {
    var items = [Int]()
    mutating func push(_ item: Int) {
        items.append(item)
    }
    mutating func pop() -> Int {
        return items.removeLast()
    }
}
```

기본적인 `push`, `pop` 기능이 들어간 `Int` 형 스택임을 알 수 있다. 

제네릭 버전의  `Stack`을 만들어보자

```swift
struct Stack<Element> {
    var items = [Element]()
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
}
```

`Element` 타입 파라미터는 세 위치에서 placeholder 역할은 한다.

- `Element` 타입의 빈 배열로 초기화되는 `items` 프로퍼티를 생성하기 위해서.
- `push(_:)` 메소드의 파라미터 타입을 명시하기 위해서.
- `pop()` 메소드가 반환하는 값의 타입을 명시하기 위해서.

이제 다음과 같이 `Stack` 인스턴스를 생성할 수 있다.

```swift
var stackOfStrings = Stack<String>()
stackOfStrings.push("uno")
stackOfStrings.push("dos")
stackOfStrings.push("tres")
stackOfStrings.push("cuatro")
// the stack now contains 4 strings
```

## Extending a Generic Type

제네릭 타입을 확장할 때, 타입 파라미터 리스트를 익스텐션 정의의 일부로 제공하지 않는다. 기존의 타입을 정의할 때 명시한 타입 파라미터를 익스텐션의 몸체에서 사용할 수 있다. 

다음은 제네릭 `Stack` 타입에 read-only 계산 프로퍼티 `topItem` 을 추가하는 예제 코드이다. 

```swift
extension Stack {
    var topItem: Element? {
        return items.isEmpty ? nil : items[items.count - 1]
    }
}
```

코드를 보면, 타입 파라미터를 정의하지 않았다는 것을 알 수 있다. 대신 `Stack` 타입의 기존 파라미터 타입인 `Element` 가 익스텐션 내부에서 사용되는 것을 볼 수 있다.

이제 `topItem` 프로퍼티는 모든 `Stack` 인스턴스에서 사용될 수 있다.

```swift
if let topItem = stackOfStrings.topItem {
    print("The top item on the stack is \(topItem).")
}
// Prints "The top item on the stack is tres."
```

## Type Constraints

위에서 본 swap 함수와 `Stack` 은 모든 타입과 함께 사용할 수 있다. 다만, 때때로 제네릭 함수 혹은 타입에 제약조건을 주는것이 유용하다. 타입 제약조건은 타입 파라미터가 특정 클래스를 상속하거나, 특정 프로토콜을 준수하는 것을 명시한다.

예를들어 Swift의 `Dictionary` 타입은 딕셔너리의 key로 사용될 수 있는 타입에 제한이 있다. `Dictionary` 의 키 타입은 반드시 `hashable`이어야 한다. 즉, `Dictionary`의 key 타입은`Hashable` 프로토콜을 준수해야만 한다는 타입 제약이 존재한다.

### Type Constraint Syntax

타입 제약은 파라미터 이름 뒤에 클래스 혹은 프로토콜 제약을 위치시키면 된다. 

```swift
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // function body goes here
}
```

여기서 `T` 타입 파라미터는 반드시 `SomeClass` 의 서브클래스여야 하고, `U` 타입 파라미터는 `SomeProtocol` 을 준수해야 한다는 타입 제약이 존재한다.

### Type Constraints in Action

여기 논제네릭 함수 `findIndex(ofString:in:)`가 있다. 배열에 해당 문자열이 존재하는지 확인하고 위치를 반환하는 함수이며, 존재하지 않으면 `nil` 을 반환한다.

```swift
func findIndex(ofString valueToFind: String, in array: [String]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}

let strings = ["cat", "dog", "llama", "parakeet", "terrapin"]
if let foundIndex = findIndex(ofString: "llama", in: strings) {
    print("The index of llama is \(foundIndex)")
}
// Prints "The index of llama is 2"
```

배열에서 값의 인덱스를 찾는 원리는 문자열에만 적용되는 것이 아니다. 따라서 이 함수를 제네릭 버전으로 작성하자.

```swift
func findIndex<T>(of valueToFind: T, in array:[T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

위 함수에는 한 가지 문제가 있다. `if value == valueToFind` 에 문제가 있는데, Swift에는`==` 연산자를 통해 비교될 수 없는 타입이 존재한다. 예를들어, 복잡한 데이터 모델을 표현하는 클래스나 구조체는 비교 연산을 사용할 수 없을 것이다. 따라서 이 코드는 모든 `T` 타입에서 정확히 수행되지 않는다.

그러나, Swift 기본 라이브러리는 `Equatable` 프로토콜을 정의하고, 이는 이를 준수하는 모든 타입이 `==`,`!=` 연산자를 두 값의 비교를 위해 사용할 수 있다. 따라서 `Equatable` 프로토콜을 준수하는 타입들은 `findIndex(of:in:)` 메소드를 안전하게 사용할 수 있다. 이를 코드로 적용하기 위해서, 타입 파라미터 정의에 제약조건을 명시하자.

```swift
func findIndex<T: Equatable>(of valueToFind: T, in array:[T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
let doubleIndex = findIndex(of: 9.3, in: [3.14159, 0.1, 0.25])
// doubleIndex is an optional Int with no value, because 9.3 isn't in the array
let stringIndex = findIndex(of: "Andrea", in: ["Mike", "Malcolm", "Andrea"])
// stringIndex is an optional Int containing a value of 2
```

"`<T: Equatable>` 은 `Equatable` 프로토콜을 준수하는 모든 `T`"를 의미한다.

## Associated Types

이전에 정리한 글이 이미 있습니다. [여기서](https://hyunsikwon.github.io/swift/Swift-AssociatedType/) 

## Generic Where Clauses

`Where`절을 사용하여 associated type이 특정 프로토콜을 준수해야하는 것이나, associated type과 타입 파라미터가 같아야하는 것과 같은 요구사항을 정의할 수 있다. `Where` 키워드로 시작하여 여러 제약조건들을 뒤에 작성하는 방식으로 명시한다.

다음의 제네릭 함수 `allItemMatch`는 두 개의 `Container` 인스턴스가 같은 요소를 같은 순서로 가지는지 확인하는 함수이다. 이 함수는 `Boolean` 값을 반환한다.

```swift
func allItemsMatch<C1: Container, C2: Container>
    (_ someContainer: C1, _ anotherContainer: C2) -> Bool
    where C1.Item == C2.Item, C1.Item: Equatable {

        // Check that both containers contain the same number of items.
        if someContainer.count != anotherContainer.count {
            return false
        }

        // Check each pair of items to see if they're equivalent.
        for i in 0..<someContainer.count {
            if someContainer[i] != anotherContainer[i] {
                return false
            }
        }

        // All items match, so return true.
        return true
}

var stackOfStrings = Stack<String>()
stackOfStrings.push("uno")
stackOfStrings.push("dos")
stackOfStrings.push("tres")

var arrayOfStrings = ["uno", "dos", "tres"]

if allItemsMatch(stackOfStrings, arrayOfStrings) {
    print("All items match.")
} else {
    print("Not all items match.")
}
// Prints "All items match."
```

- `C1` , `C2`는 `Container` 프로토콜을 준수해야한다. (`<C1: Container, C2: Container>` 로 명시)
- `C1`의 `Item` 과 `C2`의 `Item` 은 같아야한다.(`C1.Item == C2.Item`) - `Container`의 `Item` 타입이 같아야한다.
- `C1`의 `Item` 은 `Equatable` 프로토콜을 준수해야 한다. (`C1.Item: Equatable`)

`stackOfStrings` 와 `arrayOfStrings` 는 서로 다른타입의 배열이지만, 모두 `Container` 프로토콜을 준수하고, 같은 타입의 값을 가지기 때문에 함수가 정상적으로 작동한다.

*문서에서 사용된 코드는 앞선 설명에서 활용된 여러 프로토콜과 타입이 사용된 것. `Container` 프로토콜은 Swift 기본 라이브러리에 정의되어 있지 않다.*

## Extensions with a Generic Where Clause

`where` 절을 `extension` 에서도 사용할 수 있다. 다음 코드는 제네릭 `Stack` 구조체에 `isTop(_:)` 메소드를 추가한다.

```swift
extension Stack where Element: Equatable {
    func isTop(_ item: Element) -> Bool {
        guard let topItem = items.last else {
            return false
        }
        return topItem == item
    }
}

if stackOfStrings.isTop("tres") {
    print("Top element is tres.")
} else {
    print("Top element is something else.")
}
// Prints "Top element is tres."
```

만약 `where`절 없이 이 메소드를 사용한다면 문제가 발생한다: `isTop(_:)` 메소드는 `==` 연산자를 사용하는데, `Stack`은 요소들이 *equatable* 해야한다고 제한을 두지 않았다. 따라서 `==` 연산자는 complie-time 에러를 유발한다.

만약 elements 가 `Equatable`을 준수하지 않는 `Stack`에서 `isTop(_:)`을 호출하면 complie-time 에러를 유발한다.

```swift
struct NotEquatable { }
var notEquatableStack = Stack<NotEquatable>()
let notEquatableValue = NotEquatable()
notEquatableStack.push(notEquatableValue)
notEquatableStack.isTop(notEquatableValue)  // Error
```

`where` 절을 프로토콜 익스텐션에서 사용할 수 있다. 

```swift
extension Container where Item: Equatable {
    func startsWith(_ item: Item) -> Bool {
        return count >= 1 && self[0] == item
    }
}

if [9, 9, 9].startsWith(42) {
    print("Starts with 42.")
} else {
    print("Starts with something else.")
}
// Prints "Starts with something else."
```

이 메소드는 `Container` 프로토콜을 준수하는 타입 중, 요소들이 `Equatable` 하다면 사용할 수 있다.

또한 `where`절을 사용하여 더 구체적인 타입으로 제약을 줄 수 있다.

```swift
extension Container where Item == Double {
    func average() -> Double {
        var sum = 0.0
        for index in 0..<count {
            sum += self[index]
        }
        return sum / Double(count)
    }
}
print([1260.0, 1200.0, 98.6, 37.0].average())
// Prints "648.9"
```

또한, 다른 곳에 `where`절을 작성하는 것처럼,  `extension`의 `where`절에도 여러개의 요구사항을 정의할 수 있고, `,` 를 이용하여 구분한다.

## Contextual Where Clauses

`where`절을 자신의 제네릭 타입 제약을 갖지 않는 선언부에 작성할 수 있다. 예를들어 제네릭 타입의 서브스크립트 혹은 메소드에 작성할 수 있다.

```swift
extension Container {
    func average() -> Double where Item == Int {
        var sum = 0.0
        for index in 0..<count {
            sum += Double(self[index])
        }
        return sum / Double(count)
    }
    func endsWith(_ item: Item) -> Bool where Item: Equatable {
        return count >= 1 && self[count-1] == item
    }
}
let numbers = [1260, 1200, 98, 37]
print(numbers.average())
// Prints "648.75"
print(numbers.endsWith(37))
// Prints "true"
```

위의 코드를 *contextual where cluases* 없이 작성한다면 다음과 같이 각각의 `where` 절을 갖는, 두개의 `extension`으로 작성할 수 있다.

```swift
extension Container where Item == Int {
    func average() -> Double {
        var sum = 0.0
        for index in 0..<count {
            sum += Double(self[index])
        }
        return sum / Double(count)
    }
}
extension Container where Item: Equatable {
    func endsWith(_ item: Item) -> Bool {
        return count >= 1 && self[count-1] == item
    }
}
```

## Associated Types with a Generic Where Clause

Associated type에도 `where` 절을 추가할 수 있다. 예를들어 iterator를 포함하는 버전의 `Container` 를 만든다고 가정하자. 

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }

    associatedtype Iterator: IteratorProtocol where Iterator.Element == Item
    func makeIterator() -> Iterator
}
```

`Iterator` 의 `where` 절은 iterator가 element와 container의 item과 같은 타입이어야 횡단한다는 것을 요구한다.

## Generic Subscripts

서브스크립트 역시 제네릭일 수 있고, `where` 절을 포함할 수 있다. 

```swift
extension Container {
    subscript<Indices: Sequence>(indices: Indices) -> [Item]
        where Indices.Iterator.Element == Int {
            var result = [Item]()
            for index in indices {
                result.append(self[index])
            }
            return result
    }
}
```

익스텐션으로 정의된 서브스크립트는 여러 index 값들을 받아서 해당 인덱스에 위치한 값들을 포함한 배열을 반환한다. 이 제네릭 서브스크립트는 다음의 제약을 지닌다.

- `Indices` 제네릭 파라미터는 `Sequence` 프로토콜을 준수해야 한다. `Sequence` 프로토콜은 표준 라이브러리에 포함되어 있다.
- 서브스크립트는 `Indices` 타입의 인스턴스인 하나의 파라미터 `indices`를 갖는다.
- `where`절은 iterator가 `Int` 타입의 요소들만 탐색해야 함을 의미한다.

## Reference

[The Swift Language Guide](https://docs.swift.org/swift-book/LanguageGuide/Generics.html)
