---
title: Swift - 클로저!
layout: single
comments: true
share: true
categories: 
- Swift
tag:
- Closures
last_modified_at: 2020-12-12 T22:00:00+08:00
---

## Closure

클로저는 특정 기능을 하는 코드들을 하나의 블록으로 모아놓은 것을 말한다. C언어의 블록(Block) 혹은 다른 언어의 람다와 유사하다. 

클로저는 상수나 변수의 참조를 캡쳐(capture)하고 저장할 수 있는데, 이를 변수나 상수를 *closing* 한다고 말하고, 클로저는 여기서 착안된 이름이다. 이와 관련한(capturing) 메모리 관리는 Swift가 처리해준다.

전역 함수나 중첩 함수 역시 클로저의 특별한 경우 중 하나이며, 클로저는 다음과 같은 세가지 형식 중 하나의 모습으로 나타난다.

- 이름을 갖고 어떤 값도 캡쳐하지 않는 전역 함수의 형태
- 이름을 갖고 함수 내부의 값을 캡쳐할 수 있는 중첩 함수의 형태
- 이름을 갖지 않고 주변의 문맥에 따라 값을 캡쳐할 수 있는 축약된 형태

클로저는 깔끔하고 간략한 형태로 표현되고 다음과 같은 특성을 갖는다.

- 파라미터와 리턴 값의 타입을 맥락을 통해 추론
- 한 줄의 표현만 있다면 암시적으로 이를 반환 값으로 취급한다.
- 축약된 인자.
- 후행 클로저 문법.

## Closure Expressions

Swift 표준 라이브러리에는 정렬 클로저의 출력에 기반하여 배열의 값을 정렬하는 `sorted(by:)` 메소드가 있다. 이 메소드는 정렬 과정이 완료되면 기존의 배열과 같은 크기와 타입을 갖는 정렬된 메소드를 반환한다. (기존의 배열을 이 메소드의 영향을 받지 않는다.)

`sorted(by:)` 메소드를 통해 클로저의 표현 방법을 알아보자. 

다음의 문자열 배열을 알파벳 순으로 정렬하도록 하자.

```swift
let names = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
```

문자열을 정렬해야하므로, 정렬 클로저의 타입은 `(String, String) -> Bool` 이어야 한다. 정렬 클로저를 제공하는 방식은 보통의 함수를 정의하여 `sorted(by:)` 메소드의 인자로 전달하는 것이다:

```swift
func backward(_ s1: String, _ s2: String) -> Bool {
    return s1 > s2
}
var reversedNames = names.sorted(by: backward)
// reversedNames is equal to ["Ewa", "Daniella", "Chris", "Barry", "Alex"]
```

한줄의 표현(`a > b`)을 위해 함수를 작성하는 대신, 클로저 문법을 이용해서 더 간단하게 표현해보자.

### Closure Expression Syntax

먼저, 클로저의 기본 형태는 다음과 같다.

```swift
{ (parameters) -> return type in
    statements
}
```

그럼, 클로저를 이용해서 앞서 살펴본 `backward(_:_:)`함수를 표현해보자.

```swift
reversedNames = names.sorted(by: { (s1: String, s2: String) -> Bool in
    return s1 > s2
})
```

`backward(_:_:)` 함수와 파라미터의 선언과 리턴타입은 동일하지만, 인라인 클로저는  파라미터와 리턴 타입이 모두 `{ }` 안에 작성되어 있다는 점에서 차이가 있다. 클로저의 몸체는 `in` 키워드 이후에 시작된다. 

### Inferring Type From Context

Swift는 맥락에 따라 파라미터와 리턴 값의 타입을 추론할 수 있다. 따라서 `(String, String)` 과 `Bool` 타입을 생략하는 것이 가능하다.

```swift
reversedNames = names.sorted(by: { s1, s2 in return s1 > s2 } )
```

타입의 표현 여부는 상황에 따라 작성하거나 생략하면 된다. 생략할 경우 클로저의 기능이 명확히 들어나지 않는 단점이 있지만 그만큼 간결하게 코드를 작성할 수 있다. 

### Implicit Returns from Single-Expression Closures

한 줄 표현의 클로저는 `return` 키워드 마저 생략할 수 있다. 

```swift
reversedNames = names.sorted(by: { s1, s2 in s1 > s2 } )
```

`sorted(by:)` 메소드의 인자는 `Bool` 타입의 값이 반환되는 것이 명확하고 한줄의 실행뭄ㄴ(`s1 > s2`) 만 존재하는 클로저이므로 `return` 키워드를 생략할 수 있다.

### Shorthand Argument Names

Swift는 단축 인자 이름을 제공하여 앞서 살펴본 것보다 더 축약하여 클로저를 표현할 수 있다. 단축 인자 이름은 `$0`, `$1`, `$2` 순서로 표현된다. 이를 사용하면 클로저의 매개변수 리스트를 생략할 수 있고, `in` 키워드 역시 생략할 수 있다.  

```swift
reversedNames = names.sorted(by: { $0 > $1 } )
```

## Trailing Closures

함수나 메소드의 마지막 인자로 클로저를 전달하고, 클로저의 표현이 길다면 후행 클로저로 작성하는 것이 유용하다. 후행 클로저는 함수나 메소드의 소괄호를 닫은 후 작성해도 된다. 

```swift
reversedNames = names.sorted() { $0 > $1 }
```

만약 클로저가 함수나 메소드의 유일한 인자라면, 괄호 `()` 를 생략해도 된다.

```swift
reversedNames = names.sorted { $0 > $1 }
```

함수가 여러 클로저를 인자로 갖을 경우에 첫번째 후행 클로저의 인자의 이름을 생략하고 나머지 후행 클로저의 이름은 남겨두어야 한다. 

```swift
func loadPicture(from server: Server, completion: (Picture) -> Void, onFailure: () -> Void) {
    if let picture = download("photo.jpg", from: server) {
        completion(picture)
    } else {
        onFailure()
    }
}

loadPicture(from: someServer) { picture in
    someView.currentPicture = picture
} onFailure: {
    print("Couldn't download the next picture.")
}
```

## Capturing Values

클로저는 주변 문맥을 통해 변수나 상수를 캡쳐할 수 있고, 변수나 상수가 더 이상 존재하지 않더라도 클로저 내부에서 해당 변수나 상수의 값을 참조하거나 수정할 수 있다.

값을 캡쳐하는 클로저의 가장 간단한 형태는 중첩 함수이다. 중첩 함수는 바깥 함수의 인자, 상수, 변수를 캡쳐할 수 있다.

다음의 `makeIncrementer` 함수는 중첩 함수 `incrementer` 을 포함하고 있다. `incrementer()` 함수는 `runningTotal` 과 `amount` 를 캡쳐하고, 캡쳐 후에 `makeIncrementer` 에 의해 반환된다.

```swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementer
}
```

`incrementer()` 중첩 함수를 살펴보자.

```swift
func incrementer() -> Int {
    runningTotal += amount
    return runningTotal
}
```

이 함수는 파라미터를 갖지 않지만, `runningTotal`과 `amount`에 대한 참조를 캡쳐하여 이를 함수 내에서 사용한다. 참조에 의한 캡쳐는 `runningTotal`과 `amount`가 `makeIncrementer` 함수의 호출이 끝나도 사라지지 않는 것을 보장한다. 이로인해 `incrementer` 함수가 다음에 호출되었을때 `runningTotal`을 사용할 수 있다.

`makeIncrementer` 의 동작을 살펴보자.

호출될 때마다 10씩 증가하는 `incrementByTen`

```swift
let incrementByTen = makeIncrementer(forIncrement: 10)
incrementByTen()
// returns a value of 10
incrementByTen()
// returns a value of 20
incrementByTen()
// returns a value of 30
```

 

새로운 증가 incrementer을 생성하면 이는 `incrementByTen`과는 별개로 자신의 새로운 참조를 갖는다.

```swift
let incrementBySeven = makeIncrementer(forIncrement: 7)
incrementBySeven()
// returns a value of 7
```

## Closures Are Reference Types

앞선 예제에서 살펴본 `incrementBySeven` 과 `incrementByTen`은 상수 이지만 이 상수의 클로저는 여전히 캡쳐한 `runningTotal` 을 증가시킬 수 있다. 이는 함수와 클로저가 참조 타입이기 때문에 가능한 것이다. 

상수나 변수에 함수 혹은 클로저를 할당하는 것은 실제로는 함수나 클로저에 대한 참조를 설정하는 것이다. 이는 하나의 클로저를 두 개의 다른 상수나 변수에 할당하면, 두 상수 혹은 변수가 같은 클로저를 참조한다는 의미이다.

```swift
let alsoIncrementByTen = incrementByTen
alsoIncrementByTen()
// returns a value of 50

incrementByTen()
// returns a value of 60
```

## Escaping Closures

클로저가 함수의 인자로 전달되어 함수 반환 후에 호출되는 것을 *escape* 이라고 부른다. 파라미터 중 하나로 클로저를 갖는 함수를 선언할 때, `@escaping` 키워드를 타입 앞에 명시하면 클로저가 탈출하는 것을 허용한다는 것을 의미한다. 

클로저가 탈출할 수 있는 경우 중 하나는 함수 외부에 정의된 변수에 저장되는 것이다. 예를들어, 비동기 작업을 수행하는 많은 함수가 컴플리션 핸들러(completion handler)로 클로저 인자를 갖는다. 함수는 작업이 끝난 후에(함수 return 후)에 호출되는 클로저는 나중에 호출되기 위해선 함수에서 탈출되어 있어야 한다.

```swift
var completionHandlers = [() -> Void]()
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
```

함수 외부에 선언된 배열에 클로저 인자를 추가하기 위해선 반드시 `@escaping` 키워드를 붙여줘야한다.

만약 탈출 클로저가 `self` 를 참조하고 `self`가 클래스의 인스턴스를 참조한다면 특별히 고려해야할 사항이 있다. 탈출 클로저에서 `self` 를 캡쳐하면 강한 참조 사이클을 유발할 수 있기 때문이다.

만약 `self`를 캡쳐하고 싶다면, `self`를 사용할 때 명시적으로 `self`를 적어주거나 클로저의 캡쳐 리스트에 `self` 를 포함시켜야 한다. `self` 를 명시적으로 표시하는 것은 의도를 명확히 표현하고 참조 사이클이 존재하지 않다는 것을 상기시켜준다. 

```swift
func someFunctionWithNonescapingClosure(closure: () -> Void) {
    closure()
}

class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}

let instance = SomeClass()
instance.doSomething()
print(instance.x)
// Prints "200"

completionHandlers.first?()
print(instance.x)
// Prints "100"
```

다음은 클로저의 캡쳐 리스트에 `self` 를 포함시켜 캡쳐하는 방식이다.

```swift
class SomeOtherClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { [self] in x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}
```

만약 `self` 가 구조체 혹은 열거형의 인스턴스인 경우에는 `self`를 명시하지 않아도 된다. 하지만 탈출 클로저는 `self` 가 구조체 혹은 열거형의 인스턴스인 경우, `self`에 대한 가변 참조를 캡쳐할 수 없다. 

```swift
struct SomeStruct {
    var x = 10
    mutating func doSomething() {
        someFunctionWithNonescapingClosure { x = 200 }  // Ok
        someFunctionWithEscapingClosure { x = 100 }     // Error
    }
}
```

## Autoclosures

오토 클로저는 인자를 갖지 않고, 호출될 때 감싸고 있는 표현식의 값을 반환한다. 자동 클로저는 함수로 전달하는 클로저를 복잡한 클로저 문법을 사용하지 않고도 클로저로 사용할 수 있는 문법적 편의성을 제공한다. 

```swift
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
print(customersInLine.count)
// Prints "5"

let customerProvider = { customersInLine.remove(at: 0) }
print(customersInLine.count)
// Prints "5"

print("Now serving \(customerProvider())!")
// Prints "Now serving Chris!"
print(customersInLine.count)
// Prints "4"
```

클로저 내부에서 `customersInLine`의 첫번째 값이 삭제되지만, 실제로 클로저가 호출되기 전까지는 배열의 요소가 삭제되지 않는다. 클로저가 영영 호출되지 않는다면 클로저 내부의 코드는 실행되지 않는다. `customersInLine` 의 타입은 `String`이 아니라 `() -> String` 임을 주목해라.

함수의 인자로 클로저를 전달하면 위와 똑같이 지연되어 코드를 실행하는 동작을 만들 수 있다.

```swift
// customersInLine is ["Alex", "Ewa", "Barry", "Daniella"]
func serve(customer customerProvider: () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: { customersInLine.remove(at: 0) } )
// Prints "Now serving Alex!"
```

이 함수는 고객의 이름을 반환하는 클로저를 인자로 받아 함수를 실행한다.

반면에 다음과 같은 코드를 살펴보자. 앞선 함수와 완전히 같은 동작을 하는 함수지만, 클로저를 인자로 받지 않고, `@autoclosure`를 인자로 갖는다. 이로인해 마치 함수가 `String` 인자를 받는 것처럼 사용할 수 있다. 함수의 인자는 자동적으로 클로저로 변환된다.

```swift
// customersInLine is ["Ewa", "Barry", "Daniella"]
func serve(customer customerProvider: @autoclosure () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: customersInLine.remove(at: 0))
// Prints "Now serving Ewa!"
```

자동 클로저가 탈출 가능하도록 만들 수 있다. `@autoclosure`와 `@escaping` 키워드를 함께 사용하면 된다.

```swift
// customersInLine is ["Barry", "Daniella"]
var customerProviders: [() -> String] = []
func collectCustomerProviders(_ customerProvider: @autoclosure @escaping () -> String) {
    customerProviders.append(customerProvider)
}
collectCustomerProviders(customersInLine.remove(at: 0))
collectCustomerProviders(customersInLine.remove(at: 0))

print("Collected \(customerProviders.count) closures.")
// Prints "Collected 2 closures."
for customerProvider in customerProviders {
    print("Now serving \(customerProvider())!")
}
// Prints "Now serving Barry!"
// Prints "Now serving Daniella!"
```

## Reference

[docs.swift.org](https://docs.swift.org/swift-book/LanguageGuide/Closures.html)
