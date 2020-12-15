---
title: Swift - 구조체와 클래스
layout: single
comments: true
share: true
categories: 
- Swift
tag:
- Structures and Classes
last_modified_at: 2020-12-15 T17:12:00+08:00
---


구조체와 클래스는 상황에 맞게 데이터를 묶어 표현하고자 할 때 유용하다. 구조체와 클래스에 프로퍼티와 메소드를 정의하여 용도에 맞는 기능을 추가할 수 있다. 

## Comparing Structures and Classes

구조체와 클래스는 많은 공통점이 있다. 구조체와 클래스는 다음과 같은 기능을 수행할 수 있다.

- 값을 저장하기 위해 프로퍼티를 정의한다.
- 특정 기능을 수행하는 메소드를 정의한다.
- 서브스크립트 문법을 이용하요 값에 접근할 수 있게 서브스크립트를 정의한다.
- 초기 상태를 설정하기 위해 이니셜라이저를 정의한다.
- 기본 정의사항의 기능이 확장되거나 추가될 수 있다.
- 프로토콜을 준수할 수 있다.

클래스는 구조체가 할 수 없는 추가적인 기능을 갖는다.

- ***Inheritance***: 클래스는 다른 클래스를 상속을 할 수 있다.
- ***Type casting***: 타입 캐스팅을 통해 클래스의 타입을 검사할 수 있다.
- ***Deinitailizer***: Deinitailizer를 통해서 할당된 자원을 해제할 수 있다.
- ***Reference counting***: 클래스 인스턴스에 대한 하나 이상의 참조가 가능하다.

### Definition Syntax

구조체와 클래스의 선언 문법은 매우 비슷하다. 각각 `struct` 와 `class` 키워드를 통해 선언하다. 이 둘은 모두 swift의 새로운 타입을 생성하므로 대문자로 시작해야한다.

```swift
struct SomeStructure {
    // structure definition goes here
}
class SomeClass {
    // class definition goes here
}
```

다음은 구조체와 클래스를 정의하는 코드이다.

```swift
struct Resolution {
    var width = 0
    var height = 0
}
class VideoMode {
    var resolution = Resolution()
    var interlaced = false
    var frameRate = 0.0
    var name: String?
}
```

### Structure and Class Instances

구조체와 클래스의 인스턴스 생성 문법

```swift
let someResolution = Resolution()
let someVideoMode = VideoMode()
```

### Accessing Properties

마침표 `.` 를 통해서 프로퍼티에 접근할 수 있다.

```swift
print("The width of someResolution is \(someResolution.width)")
// Prints "The width of someResolution is 0"
```

하위 레벨의 프로퍼티에도 접근할 수 있다.

```swift
print("The width of someVideoMode is \(someVideoMode.resolution.width)")
// Prints "The width of someVideoMode is 0"
```

프로퍼티에 새로운 값을 할당할 수도 있다.

```swift
someVideoMode.resolution.width = 1280
print("The width of someVideoMode is now \(someVideoMode.resolution.width)")
// Prints "The width of someVideoMode is now 1280"
```

### Memberwise Initializers for Structure Types

모든 구조체는 자동으로 *meberwise initializer* 를 생성한다. 새로운 인스턴스의 프로퍼티 초기 값은 이 이니셜라이저에 전달되어 생성된다.

```swift
let vga = Resolution(width: 640, height: 480)
```

구조체와는 달리, 클래스는 기본 멤버와이즈 이니셜라이저를 갖지 않는다.

## Structures and Enumerations Are Value Types

*Value Type(값 타입)* 은 변수나 상수에 할당될 때 혹은 함수에 전달될 때 값이 복사되어 잔달된다. Swift의 모든 기본 타입(`Int`, `Float`, `Bool`, `String` 등)은 값 타입으로 모두 구조체로 구현되어 있다.

모든 구조체와 열거형은 값 타입이며, 이는 모든 구조체와 열거형의 인스턴스, 값 타입의 프로퍼티는 언제나 복사되어 전달된다는 뜻이다.

다음 예제를 살펴보자:

```swift
let hd = Resolution(width: 1920, height: 1080)
var cinema = hd
```

full HD 비디오의 너비와 높이로 초기화된 `Resolution` 인스턴스 `hd`를 생성하고, `hd`를 할당한 `cinema` 변수를 정의했다. 구조체의 특성에 따라 `hd` 인스턴스의 복사본이 만들어지고, 이 복사본이 `cinema` 에 할당된 것이다. 

두 인스턴스는 완전이 분리됐기 때문에, `cinema` 의 프로퍼티 값을 변경해도 `hd` 에는 영향을 주지 않는다.

```swift
cinema.width = 2048

print("cinema is now \(cinema.width) pixels wide")
// Prints "cinema is now 2048 pixels wide"

print("hd is still \(hd.width) pixels wide")
// Prints "hd is still 1920 pixels wide"
```

<img width="650" alt="sharedStateStruct_2x" src="https://user-images.githubusercontent.com/48352065/102188588-d61d7b80-3ef8-11eb-9406-e3fdbd8102e0.png">

열거형도 구조체와 마찬가지.

```swift
enum CompassPoint {
    case north, south, east, west
    mutating func turnNorth() {
        self = .north
    }
}
var currentDirection = CompassPoint.west
let rememberedDirection = currentDirection
currentDirection.turnNorth()

print("The current direction is \(currentDirection)")
print("The remembered direction is \(rememberedDirection)")
// Prints "The current direction is north"
// Prints "The remembered direction is west"
```

## Classes Are Reference Types

값 타입과는 달리, *reference type*은 변수나 상수에 할당될 때, 함수에 전달될 때 복사되지 않는다. 대신 존재하는 인스턴스에대한 참조가 사용된다.

코드를 통해 알아보자:

```swift
let tenEighty = VideoMode()
tenEighty.resolution = hd
tenEighty.interlaced = true
tenEighty.name = "1080i"
tenEighty.frameRate = 25.0

let alsoTenEighty = tenEighty
alsoTenEighty.frameRate = 30.0
```

`ViewMode` 클래스의 인스턴스 `tenEighty`를 선언하여 프로퍼티에 값을 할당하고 `tenEighty` 를 새로운 상수 `alsoTenEighty`에 할당했다. 클래스는 참조 타입이므로 `tenEighty` 와 `alsoTenEighty`는 같은 인스턴스를 참조한다. 

<img width="670" alt="sharedStateClass_2x" src="https://user-images.githubusercontent.com/48352065/102188580-d453b800-3ef8-11eb-917a-7c17bf6b27fc.png">

같은 참조로 인해서 프로퍼티의 변화는 두 상수에 모두 영향을 준다.

```swift
print("The frameRate property of tenEighty is now \(tenEighty.frameRate)")
// Prints "The frameRate property of tenEighty is now 30.0"
```

여기서 주목할 것은 `tenEighty` 와 `alsoTenEighty`는 상수로 선언되었다는 것이다. 상수로 선언했음에도 불구하고 `tenEighty.frameRate`와 `alsoTenEighty.frameRate`의 값이 변할 수 있는 것은 `tenEighty` 와 `alsoTenEighty`는 실제로 `ViewMode` 인스턴스를 저장하는 것이 아니라 인스턴스를 참조하고 있기 때문이다.

### Identity Operators

클래스는 참조 타입이기 때문에, 여러 상수와 변수가 동일한 하나의 인스턴스를 참조할 수 있다. 

이런 특성을 살려, 두 상수나 변수가 같은 클래스 인스턴스를 참조하는지 확인할 수 있다. 이를 위해 우리는 두 개의 식별 연산자(dentity operators)를 사용한다.

- Identical (`===`)
- Not identical (`!==`)

예제 코드:

```swift
if tenEighty === alsoTenEighty {
    print("tenEighty and alsoTenEighty refer to the same VideoMode instance.")
}
// Prints "tenEighty and alsoTenEighty refer to the same VideoMode instance."
```

### Pointers

C, C++ 언어는 메모리 주소를 참조하는 포인터를 사용한다. Swift에서 상수나 변수가 참조 타입의 인스턴스를 참조하는 것도 포인터와 비슷하지만, 실제로 메모리의 주소를 직접 참조하지 않고, 참조를 생성했다는 것을 알리기 위해 `*` 를 작성할 필요도 없다. 

## Reference

[docs.swift.org](https://docs.swift.org/swift-book/LanguageGuide/ClassesAndStructures.html)
