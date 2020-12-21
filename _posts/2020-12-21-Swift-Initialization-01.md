---
title: Swift - Initialization PART1
layout: single
comments: true
share: true
categories: 
- Swift
tag:
- Initialization
last_modified_at: 2020-12-21 T15:27:00+08:00
---

## Initialization

*Initialization*는 클래스, 구조체, 열거형의 인스턴스를 사용하기 위한 준비 과정이다. 이 과정에서 인스턴스의 저장 프로퍼티의 초기 값을 설정하고, 인스턴스를 사용하기 전에 요구되는 설정이나 초기 작업을 수행한다. 

초기화는 특정 타입의 인스턴스를 생성하기 전에 호출되는 *initializers*를 정의하여 수행한다.  이니셜라이저는 별다른 리턴 값이 없으며, 인스턴스를 처음 사용하기 전에 정확하게 초기화 되었는 지를 보장하는 것이 주 역할이다.

클래스 타입의 인스턴스는 *deinitializer* 또한 수행할 수 있는데, 여기에는 클래스의 인스턴스가 deallocated 직전에 해야할 일들을 정의할 수 있다.

## Setting Initial Values for Stored Properties

클래스와 구조체는 인스턴스가 생성될 때 모든 저장 프로퍼티의 초기값을 반드시 설정해야 한다. 저장 프로퍼티는 indeterminate(불확정) 상태가 되면 안된다. 저장 프로퍼티의 초기 값은 이니셜라이저를 통해서나기본 값을 할당하여 설정할 수 있다.

### Initializers

이니셜라이저는 특정 타입의 새로운 인스턴스를 생성하기 위해 호출된다. 가장 단순한 형태의 이니셜라이저는 파라미터가 없는 인스턴스 메소드와 유사하다.  `init` 키워드를 사용하여 작성한다

```swift
struct Fahrenheit {
    var temperature: Double
    init() {
        temperature = 32.0
    }
}
var f = Fahrenheit()
print("The default temperature is \(f.temperature)° Fahrenheit")
// Prints "The default temperature is 32.0° Fahrenheit"
```

### Default Property Values

프로퍼티를 정의할 때 초기 값을 할당하여 프로퍼티의 기본 값을 명시할 수 있다.

```swift
struct Fahrenheit {
    var temperature = 32.0
}
```

> 프로퍼티가 항상 같은 초기 값을 갖는다면, 기본 값을 통해 초기 값을 설정해라. 결과는 똑같지만 코드가 더 짧고 깔끔할 뿐더러 기본 값을 통해 프로퍼티의 타입을 추론할 수 있게 해준다. 또한, 앞으로 살펴볼 default initializers와 initializer inheritance의 이점을 쉽게 활용할 수 있다.

## Customizing Initialization

다양한 방법을 통해서 초기화 과정을 목적에 맞게 구성할 수 있다.

### Initialization Parameters

이니셜라이저에 파라미터를 추가할 수 있다. 이니셜라이저 파라미터는 함수, 메소드 파라미터와 완전히 같은 문법이고 같은 기능을 한다.

```swift
struct Celsius {
    var temperatureInCelsius: Double
    init(fromFahrenheit fahrenheit: Double) {
        temperatureInCelsius = (fahrenheit - 32.0) / 1.8
    }
    init(fromKelvin kelvin: Double) {
        temperatureInCelsius = kelvin - 273.15
    }
}
let boilingPointOfWater = Celsius(fromFahrenheit: 212.0)
// boilingPointOfWater.temperatureInCelsius is 100.0
let freezingPointOfWater = Celsius(fromKelvin: 273.15)
// freezingPointOfWater.temperatureInCelsius is 0.0
```

### Parameter Names and Argument Labels

함수, 메소드와 마찬가지로, 이니셜라이저도 내부에서 사용하는 파라미터 이름과 호출시에 사용하는 argument label을 모두 가질 수 있다. 

이니셜라이저는 함수의 이름처럼 식별 가능하지 않아서, 파라미터의 이름과 타입이 이니셜라이저를 식별하는데 매우 중요한 역할을 한다. 이러한 이유로 만약 따로 argument label을 작성하지 않으면, Swift는 자동으로 argument label을 제공한다.

```swift
struct Color {
    let red, green, blue: Double
    init(red: Double, green: Double, blue: Double) {
        self.red   = red
        self.green = green
        self.blue  = blue
    }
    init(white: Double) {
        red   = white
        green = white
        blue  = white
    }
}
```

정의된 두 개의 이니셜라이저는 파라미터 이름을 사용해서 인스턴스를 생성한다.

```swift
let magenta = Color(red: 1.0, green: 0.0, blue: 1.0)
let halfGray = Color(white: 0.5)
```

argument label을 사용하지 않고 이니셜라이저를 호출하는 것은 불가능 하다.

```swift
let veryGreen = Color(0.0, 1.0, 0.0)
// this reports a compile-time error - argument labels are required
```

### Initializer Parameters Without Argument Labels

만약 argument label을 사용하고 싶지 않다면, `_` 를 argument label 대신 작성하면 된다.

```swift
struct Celsius {
    var temperatureInCelsius: Double
    init(fromFahrenheit fahrenheit: Double) {
        temperatureInCelsius = (fahrenheit - 32.0) / 1.8
    }
    init(fromKelvin kelvin: Double) {
        temperatureInCelsius = kelvin - 273.15
    }
    init(_ celsius: Double) {
        temperatureInCelsius = celsius
    }
}
let bodyTemperature = Celsius(37.0)
// bodyTemperature.temperatureInCelsius is 37.0
```

다음과 같은 경우는 이니셜라이저를 구분할 수 없기 때문에 불가능하다. 어떤 방식으로든 이니셜라이저가 구분될 수 있어야 한다.

```swift
struct Test {
    var temp: Int
    
    init(_ a: Int) {
        self.temp = a
    }
    
    init(_ b: Int) {
        self.temp = b
    }
}
```

### Optional Property Types

만약 타입이 `nil`이 되는 것을 논리적으로 허용하거나, 추후에 `nil`을 가질 수 있는 저장 프로퍼티를 가진다면, 이 프로퍼티는 *optional* 타입으로 선언해야 한다. 옵셔널 타입의 프로퍼티는 자동적으로 `nil`로 초기화된다.

```swift
class SurveyQuestion {
    var text: String
    var response: String?
    init(text: String) {
        self.text = text
    }
    func ask() {
        print(text)
    }
}
let cheeseQuestion = SurveyQuestion(text: "Do you like cheese?")
cheeseQuestion.ask()
// Prints "Do you like cheese?"
cheeseQuestion.response = "Yes, I do like cheese."
```

### Assigning Constant Properties During Initialization

초기화 과정 어느 곳에서든, 상수 프로퍼티에도 값을 할당할 수 있다. 그러나, 일단 상수 프로퍼티에 값이 할당되면, 후에는 수정될 수 없다.

위에서 본 `SurveyQuestion` 클래스의 `text` 프로퍼티를 상수로 수정해보자. 이는 일단`SurveyQuestion`의 인스턴스가 생성되면 앞으로 값이 변하지 않음을 알려준다.

```swift
class SurveyQuestion {
    let text: String
    var response: String?
    init(text: String) {
        self.text = text
    }
    func ask() {
        print(text)
    }
}
let beetsQuestion = SurveyQuestion(text: "How about beets?")
beetsQuestion.ask()
// Prints "How about beets?"
beetsQuestion.response = "I also like beets. (But not with cheese.)"
```

## Default Initializers

Swift는 모든 구조체와 클래스의 *default initializer(기본 이니셜라이저)*를 제공한다. 이 기본 이니셜라이저는 모든 프로퍼티를 기본 값으로 설정하여 인스턴스를 생성한다.

```swift
class ShoppingListItem {
    var name: String?
    var quantity = 1
    var purchased = false
}
var item = ShoppingListItem()
```

`ShoppingListItem` 클래스는 모든 프로퍼티가 기본 값을 갖고, 슈퍼 클래스가 없는 기본 클래스기 때문에, 자동으로 기본 이니셜라이저를 갖게 된다. (`name` 프로퍼티는 옵셔널 `String` 프로퍼티기 때문에 기본 값으로 `nil` 을 갖게된다.)

### Memberwise Initializers for Structure Types

구조체 타입은 따로 이니셜라이저를 정의하지 않는다면 자동으로 *memberwise initializer*를 갖게된다. 기본 이니셜라이저(default initializer)와는 달리, 타입에 기본 값을 갖지 않는 저장 프로퍼티가 있더라도 멤버와이즈 이니셜라이저를 가지게 된다.

멤버와이즈 이니셜라이저는 구조체 인스턴스의 프로퍼를 초기화하는 간단한 방법으로, 프로퍼티의 초기 값이 멤버와이즈 이니셜라이저를 통해 전달된다.

```swift
struct Size {
    var width = 0.0, height = 0.0
}
let twoByTwo = Size(width: 2.0, height: 2.0)
```

멤버와이즈 이니셜라이저를 호출할 때, 기본 값을 가지는 프로퍼티라면 초기 값을 생략할 수 있다. 위 예제에서 `width`, `height` 프로퍼티는 기본 값을 가지기 때문에 둘 중 하나를 생략하거나 둘 다 생략할 수 있다. 

```swift
let zeroByTwo = Size(height: 2.0)
print(zeroByTwo.width, zeroByTwo.height)
// Prints "0.0 2.0"

let zeroByZero = Size()
print(zeroByZero.width, zeroByZero.height)
// Prints "0.0 0.0"
```

## Initializer Delegation for Value Types

이니셜라이저는 다른 이니셜라이저를 호출하여 인스턴스 초기화의 일부를 수행할 수 있다. 이 과정을 *initializer delegation*라고 하고 이를 통해서 여러 이니셜라이저 사이의 코드 중복을 방지할 수 있다.

어떻게 이니셜라이저 위임이 수행되는지, 어떤 형태의 위임이 허용되는지에 대한 규칙은 값 타입인지 참조 타입인지에 따라 다르다. 값 타입은 상속을 지원하지 않기 때문에 이니셜라이저 위임 과정은 비교적 단순하다. 그러나 클래스는 다른 클래스를 상속할 수 있고, 따라서 초기화 과정에서 상속한 저장 프로퍼티에 적절한 값을 할당해야 하는 추가적인 책임을 갖는다. 이는 PART2에서 자세히 알아보자.

갑 타입의 경우 자신의 이니셜라이저를 정의할 때 `self.init`을 사용하여 동일한 값 타입의 이니셜라이저를 참조한다. 오직 이니셜라이저 안에서만 `self.init`을 호출할 수 있다. 

코드를 통해 더 자세히 알아보자.

```swift
struct Size {
    var width = 0.0, height = 0.0
}
struct Point {
    var x = 0.0, y = 0.0
}

struct Rect {
    var origin = Point()
    var size = Size()
    init() {} // 1
    init(origin: Point, size: Size) { // 2
        self.origin = origin
        self.size = size
    }
    init(center: Point, size: Size) { // 3
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}
```

위의 `Rect` 구조체는 세 가지 방식 중 하나를 통해서 초기화 될 수 있고, 이 초기화 옵션은 `Rect`에서 3 가지의 이니셜라이저로 표현된다.  

첫번째 `Rect` 이니셜라이저 `init()`는 기본 이니셜라이저와 같은 기능을 한다. `{}` 본체 부분은 비어있고, 이 이니셜라이저를 호출하면 모두 기본 값으로 초기화 된  `origin`(`Point(x: 0.0, y: 0.0)`), `size`(`Size(width: 0.0, height: 0.0)`) 프로퍼티를 가진 `Rect` 인스턴가 반환된다.

```swift
let basicRect = Rect()
// basicRect's origin is (0.0, 0.0) and its size is (0.0, 0.0)
```

두번째 `Rect` 이니셜라이저 `init(origin:size:)`는 멤버와이즈 이니셜라이저와 같은 기능을 한다. 이 이니셜라이저는 단순히 `origin`과 `size` 인자 값을 저장 프로퍼티에 할당한다.

```swift
let originRect = Rect(origin: Point(x: 2.0, y: 2.0),
                      size: Size(width: 5.0, height: 5.0))
// originRect's origin is (2.0, 2.0) and its size is (5.0, 5.0)
```

세번째 이니셜라이저 `init(center:size:)`는 약간 복잡하다. 이 이니셜라이저는 먼저 `center`와 `size` 기반으로 적절한 값을(`originX`, `originY`) 계산한다. 다음엔 이 값을 가지고 `init(origin:size:)` 이니셜라이저에 초기화를 위임한다.

```swift
let centerRect = Rect(center: Point(x: 4.0, y: 4.0),
                      size: Size(width: 3.0, height: 3.0))
// centerRect's origin is (2.5, 2.5) and its size is (3.0, 3.0)
```

`init(center:size:)` 이니셜라이저가 새로운 `origin`, `size` 값을 직접 할당할 수 있다. 하지만 이미 존재하는 같은 기능을 하는 이니셜라이저를 활용하는 것이 더 편리하고 의도에 맞는 방법이다. 또한, 코드의 중복을 줄일 수 있다.

## Reference

[docs.swift.org](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html)
