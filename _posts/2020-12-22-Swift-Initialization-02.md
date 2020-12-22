---
title: Initialization part2
layout: single
comments: true
share: true
categories: 
- Swift
tag:
- Initialization
last_modified_at: 2020-12-22 T15:27:00+08:00
---

## Class Inheritance and Initialization

슈퍼 클래스로부터 상속 받은 프로퍼티를 포함한 모든 클래스의 저장 프로퍼티는 초기화 과정에서 반드시 초기 값을 할당 받아야한다. Swift는 클래스의 모든 저장 프로퍼티가 초기 값을 가지는 것을 보장하기 위해 두 가지의 이니셜라이저를 제공한다.

### Designated Initializers and Convenience Initializers

*Designated initializers*는 클래스의 주요 이니셜라이저이다. 지정 이니셜라이저는 클래스의 모든 프로퍼티를 초기화 하고, 모든 관련된 슈퍼클래스의 이니셜라이저를 호출한다. 모든 클래스는 최소한 하나의 지정 이니셜라이저를 가져야하며, 몇몇의 경우에는 하나 이상의 지정 이니셜라이저를 슈퍼클래스로부터 상속하여 이를 만족한다. 

*Convenience initializers*는 초기화를 도와주는 이니셜라이저이다. 편의 이니셜라이저는 필수가 아니며, 보통의 초기화 패턴보다 간단한 방식을 통해서 시간을 절약하거나 의도를 명확히하기 위해서 사용된다.

### Syntax for Designated and Convenience Initializers

클래스의 지정 이니셜라이저는 값 타입의 이니셜라이저와 같은 방식으로 작성한다.

```swift
 init(parameters) {
    statements
}
```

편의 이니셜라이저는 `convenience` 수식어를 붙여준다.

```swift
convenience init(parameters) {
    statements
}
```

### Initializer Delegation for Class Types

지정 이니셜라이저와 편의 이니셜라이저 사이의 관계를 단순화하기 위해서, Swift는 이니셜라이저 사이의 위임에 대한 세 가지 **규칙**을 적용한다. 

1. 지정 이니셜라이저는 반드시 가장 인접한 슈퍼클래스의 지정 이니셜라이저를 호출해아 한다.
2. 편의 이니셜라이저는 반드시 같은 클래스의 다른 이니셜라이저를  호출해야 한다.
3. 편의 이니셜라이저는 궁극적으로는 반드시 지정 이니셜라이저를 호출해야 한다.

이를 쉽게 기억하는 방법은 다음과 같다:

- 지정 이니셜라이저는 반드시 윗 방향으로 위임 한다.
- 편의 이니셜라이저는 반드시 옆 방향으로 위임 한다.

![initializerDelegation01_2x](https://user-images.githubusercontent.com/48352065/102856754-6a905c80-446a-11eb-829f-21cbe298a06c.png)

슈퍼클래스는 하나의 지정 이니셜라이저와 두 개의 편의 이니셜라이저를 가진다. 한 개의 편의 이니셜라이저는 지정 이니셜라이저를 호출하는 같은 클래스의 다른 편의 이니셜라이저를 호출한다. 이는 위에서 본 **규칙 2,3**을 만족한다. 슈퍼클래스는 자신의 슈퍼클래스를 가지지 않으므로 **규칙 1**은 적용되지 않는다.

그림의 서브클래스는 두 개의 지정 이니셜라이저와 한 개의 편의 이니셜라이저를 가진다. 편의 이니셜라이저는 같은 클래스의 이니셜라이저만 호출할 수 있기 때문에, 두 지정 이니셜라이저 중 하나를 만드시 호출해야한다. 이는 **규칙 2,3**을 만족한다. 두 지정 이니셜라이저는 슈퍼클래스의 지정 이니셜라이저를 호출해야하고, 이는 **규칙 1**을 만족한다. 

다음의 그림은 조금 더 복잡한 클래스 계층 구조를 보여준다. 이 그림을 보면 지정 이니셜라이저가 계층 구조 내에서 "funnel(깔때기)" 포인트의 역할을 하는지 볼 수 있다.

![initializerDelegation02_2x](https://user-images.githubusercontent.com/48352065/102856751-69f7c600-446a-11eb-97e1-1574bf5588ba.png)

### Two-Phase Initialization

Swift에서 클래스 초기화는 두 단계를 거친다. 첫번째 단계에서 각각의 저장 프로퍼티는 초기 값을 할당 받는다. 모든 저장 프로퍼티의 초기 상태가 결정되면 두번째 단계가 시작되고, 이때 각각의 클래스는 인스턴스가 사용되기 전에 자신의 저장 프로퍼티를 목적에 맞게 설정할 수 있는 기회를 갖게 된다. 두 단계의 초기화는 프로퍼티가 초기화되기 전에 접근되는 것을 막아주고, 프로퍼티가 다른 이니셜라이저에 의해 다른 값이 설정되는 것을 막아준다.

Swift 컴파일러는 두 단계 초기화가 에러 없이 이루어질 수 있도록 4개의 안정성 검사를 수행한다.

**Safety check1**

지정 이니셜라이저는 슈퍼클래스의 이니셜라이저에 위임하기 전에 클래스의 모든 프로퍼티가 초기화되도록 해야한다. 

> 객체를 위한 메모리는 모든 저장 프로퍼티의 초기 상태를 알아야만 완전히 초기화 되었다고 간주된다. 이를 위해서 지정 이니셜라이저는 모든 자신의 프로퍼티가 위로 전달되기 전에 초기화 되었다는 것을 보장해야한다.

**Safety check2**

지정 이니셜라이저는 상속받은 프로퍼티에 값을 할당하기 전에 슈퍼클래스의 이니셜라이저에 위임해야 한다. 그렇지 않으면 지정 이니셜라이저가 할당한 새로운 값이 슈퍼클래스에 의해 덮어씌워질 것이다.

**Safety check3**

편의 이니셜라이저는 프로퍼티에 값을 할당하기 전에 다른 이니셜라이저에 위임해야한다. 그렇지 않으면 편의 이니셜라이저가 할당한 값이 지정 이니셜라이저에 의해 덮어씌워질 것이다.

**Safety check4**

이니셜라이저는 초기화의 첫 단계가 끝나기 전까지 메소드를 호출하거나 프로퍼티의 값을 읽거나 할 수 없다. 또한 `self`를 값으로 참조할 수 없다.

네 가지 안정성 검사를 기반으로 초기화의 두 단계는 다음과 같이 진행된다.

**Phase 1**

- 지정/편의 이니셜라이저가 호출된다.
- 클래스의 새로운 인스턴스를 위한 메모리가 할당된다. 메모리는 초기화된 상태는 아니다.
- 클래스의 지정 이니셜라이저가 모든 저장 프로퍼티가 값을 가지는지 확인하면, 이제 이 저장 프로퍼티를 위한 메모리가 초기화된다.
- 지정 이니셜라이저는 슈퍼클래스도 저장 프로퍼티를 위한 같은 작업을 수행하도록 슈퍼클래스의 이니셜라이저에 위임한다.
- 이는 클래스 계층 구조의 맨 꼭대기에 도착할 때까지 진행된다.
- 가장 꼭대기에 도착하면, 마지막 클래스는 자신의 모든 저장 프로퍼티가 값을 가진다는 것을 보장하고, 인스턴스의 메모리는 완전히 초기화 된 것으로 간주되며 첫 단계가 끝난다.

**Phase 2**

- 다시 클래스 계층 구조의 위에서 아래로 진행되며, 지정 이니셜라이저는 인스턴스를 목적에 맞게 조정할 수 있다. 이니셜라이저는 이제 `self`에 접근할 수 있고, 프로퍼티를 수정하고, 인스턴스 메소드를 호출할 수 있다.
- 마지막으로, 모든 편의 이니셜라이저는 인스턴스를 목적에 맞게 조장할 수 있고 `self`에 접근할 수 있다.

그림을 통해 다시 한번 진행과정을 살펴보자.

![twoPhaseInitialization01_2x](https://user-images.githubusercontent.com/48352065/102856749-69f7c600-446a-11eb-981e-8f6dc3ad72ab.png)


초기화는 편의 이니셜라이저를 호출하며 시작된다. 편의 이니셜라이저는 아직 프로퍼티을 수정할 수 없고, 같은 클래스의 지정 이니셜라이저에 위임한다. 지정 이니셜라이저는 **safety check 1**에 따라 서브 클래스의 모든 프로퍼티가 값을 가진다는 것을 확인한다. 이후에 슈퍼클래스의 지정 이니셜라이저를 호출하여 초기화 과정을 진행한다.

슈퍼클래스의 지정이니셜라이저는 자신의 모든 프로퍼티가 값을 가지는지 확인한다. 초기화할 더 이상의 슈퍼클래스가 존재하지 않고 따라서 더 이상의 위임도 필요 없다. 슈퍼클래스의 모든 프로퍼티가 초기값을 가지면, 메모리는 완전이 초기화되고 첫 단계가 끝나게 된다.

이제 두번째 단계가 진행된다. 

![twoPhaseInitialization02_2x](https://user-images.githubusercontent.com/48352065/102856748-695f2f80-446a-11eb-8eec-e1bd0425fad5.png)


슈퍼클래스의 지정 이니셜라이저는 이제 인스턴스를 목적에 맞게 조작할 기회를 얻게된다. (안할 수도 있다.) 슈퍼클래스의 지정 이니셜라이저가 완료되면 서브 클래스의 지정 이니셜라이저 역시 같은 기회를 얻는다. 이 과정이 끝나면, 마지막으로 편의 이니셜라이저도 같은 기회를 얻으며 초기화 과정을 마치게 된다.

### Initializer Inheritance and Overriding

Obecjtive-C와는 달리 Swift의 서브클래스는 슈퍼클래스의 이니셜라이저를 디폴트로 상속하지 않는다. 이는 슈퍼클래스의 아주 간단한 이니셜라이저가 더 특화된 서브클래스에 의해 상속되는 것이나, 슈퍼클래스의 이니셜라이저로 인해 완전히 혹은 정확히 초기화되지 않은 서브클래스의 인스턴스가 생성되는 것을 예방한다.

슈퍼클래스의 이니셜라이저와 동일한 이니셜라이저를 서브클래스에서 사용하고 싶다면 서브클래스에서 구현하면 되는데, 만약 슈퍼클래스와 동일한 지정 이니셜라이저를 구현하고 싶다면 서브클래스에서 재정의를 하면 된다. 슈퍼클래스의 기본 이니셜라이저도 서브클래스에서 재정의 할 수 있다. 재정의를 위해선 `override`와 함께 이니셜라이저를 작성하면 된다.

오버라이딩한 프로퍼티, 메소드, 서브스크립트와 마찬가지로 `override` 수식어는 Swift로 하여금 슈퍼클래스에 동일한 지정 이니셜라이저가 존재하는지와 오버라이딩한 이니셜아저가 유요한지를 검사하게 한다.

> 지정 이니셜라이저를 오버라이딩 할 땐 반드시 `override` 수식어를 붙여주어야 한다.

그러나, 슈퍼클래스의 편의 이니셜라이저를 서브클래스에 작성할 때는 `override` 수식어를 붙이지 않는다. 위에서 본 **규칙**에 따라 슈퍼클래스의 편의 이니셜라이저는 서브클래스에 의해 직접 호출되지 않기 때문이다. 서브클래스에 의해 직접 호출되는 이니셜라이저는 지정 이니셜라이저 뿐이다.

코드를 통해서 살펴보자. 먼저, 기본 클래스인 `Vehicle`은 기본 값이 `0` 인 `numberOfWheels` 저장 프로퍼티와, `description` 계산프로퍼티를 선언한다.

```swift
class Vehicle {
    var numberOfWheels = 0
    var description: String {
        return "\(numberOfWheels) wheel(s)"
    }
} 
```

`Vehicle` 클래스는 유일한 저장프로퍼티 `numberOfWheels`에 기본 값을 제공하고, 따로 이니셜라이저를 구현하지 않았기 때문에, 자동으로 기본 이니셜라이저(default initializer)를 갖게된다. 기본 이니셜라이저는 언제나 클래스의 지정 이니셜라이저이므로, 다음과 같이 인스턴스를 생성하는데 사용될 수 있다.

```swift
let vehicle = Vehicle()
print("Vehicle: \(vehicle.description)")
// Vehicle: 0 wheel(s)
```

다음은 `Vehicle`의 서브클래스 `Bicycle` 이다.

```swift
class Bicycle: Vehicle {
    override init() {
        super.init()
        numberOfWheels = 2
    }
}
```

`Bicycle`은 지정 이니셜라이저를 정의하고, 이는 슈퍼클래스의 지정 이니셜라이저와 동일하다. 따라서 이니셜라이저에 `override` 수식어를 붙여주었다. 여기서 슈퍼클래스의 기본 이니셜라이저를 `super.init()`을 통해 호출하는 것은 상속된 `numberOfWheels` 프로퍼티가 슈퍼클래스에 의해 먼저 초기화되는 것을 보장한다. `super.init()`이 호출된 후에 `numberOfWheels` 값은 2로 대체된다.

`Bicycle`의 인스턴스를 생성하면 상속된 `description` 계산 프로퍼티를 호출할 수 있다. `numberOfWheels` 프로퍼티의 값을 주목해라.

```swift
let bicycle = Bicycle()
print("Bicycle: \(bicycle.description)")
// Bicycle: 2 wheel(s)
```

만약 서브클래스의 이니셜라이저가 초기화 과정의 2번째 단계에서 위처럼 상속된 프로피티를 새로 정의하는 과정이 없고, 슈퍼클래스의 지정 이니셜라이저가 어떠한 인자를 갖지 않는다면 `super.init()`을 생략할 수 있다.

다음의 `Vehicle`의 서브클래스 `Hoverboard`는 이니셜라이저에서 오직 자신의 프로퍼티인 `color` 만 값을 설정한다. 따라서 `super.init()`를 명시적으로 호출할 필요가 없다.

```swift
class Hoverboard: Vehicle {
    var color: String
    init(color: String) {
        self.color = color
        // super.init() implicitly called here
    }
    override var description: String {
        return "\(super.description) in a beautiful \(color)"
    }
} 
```

`Hoverboard`의 인스턴스는 `Vehicle`의 이니셜라이저가 제공하는 `numberOfWheels`의 기본 값을 사용한다.

```swift
let hoverboard = Hoverboard(color: "silver")
print("Hoverboard: \(hoverboard.description)")
// Hoverboard: 0 wheel(s) in a beautiful silver
```

### Automatic Initializer Inheritance

앞서 이야기 했듯이 서브클래스는 디폴드로 슈퍼클래스의 이니셜라이저를 상속하지 않는다. 그러나 만약 특정 조건을 만족하면 슈퍼클래스의 이니셜라이저가 자동으로 상속되는 경우가 있다. 따라서 특정 조건을 만족하면 이니셜라이저를 재정의할 필요 없고, 적은 노력으로 슈퍼클래스의 이니셜라이저를 상속할 수 있다.

**서브클래스의 모든 프로퍼티에 기본 값을 제공한다고 가정할 때**, 다음 두가지 규칙에 따라서 슈퍼클래스의 이니셜라이저가 자동으로 상속된다:

**Rule 1** 

서브클래스가 어떠한 지정 이니셜라이저도 정의하지 않는다면, 슈퍼클래스의 모든 지정 이니셜라이저를 자동으로 상속한다.

**Rule 2**

서브클래스가 슈퍼클래스의 모든 지정 이니셜라이저를 구현한 경우 - **규칙 1**에 따라서 자동 상속 받은 경우 or 모든 슈퍼클래스의 지정 이니셜라이저를 재정의한 경우 - 슈퍼클래스의 모든 편의 이니셜라이저를 자동으로 상속한다.

### Designated and Convenience Initializers in Action

다음의 예제는 지정 이니셜라이저, 편의 이니셜라이저, 자동 이니셜라이저 상속이 어떻게 동작하는 지 보여준다. 이 예제에는 3개의 클래스(`Food`, `RecipeIngredient`, `ShoppingListItem`) 계층을 정의한다. 

기본 클래스는 `Food`이며, 음식의 이름을 캡슐화한 간단한 클래스이다. 한 개의 문자열 타입의 프로퍼티 `name`과 인스턴스 생성을 위한 두 개의 이니셜라이저가 정의되어 있다.

```swift
class Food {
    var name: String
    init(name: String) {
        self.name = name
    }
    convenience init() {
        self.init(name: "[Unnamed]")
    }
}
```

다음 그림은 `Food` 클래스의 이니셜라이저 구조를 보여준다:

![initializersExample01_2x](https://user-images.githubusercontent.com/48352065/102856747-68c69900-446a-11eb-9cd4-db68395a8656.png)

`Food` 클래스는 `name` 인자 하나를 가지는 지정 이니셜라이저를 제공한다. 이 이니셜라이저는 특정한 이름과 함께 인스턴스를 생성하기 위해 사용된다.

```swift
let namedMeat = Food(name: "Bacon")
// namedMeat's name is "Bacon"
```

`init(name: String)` 이니셜라이저는 지정 이니셜라이저로 제공된다. 이 이니셜라이저가 `Food` 인스턴스의 모든 저장 프로퍼티가 완전히 초기화 되는 것을 보장하기 때문이다. 추가로 `Food` 클래스는 슈퍼클래스가 없으므로 `super.init()`을 호출하지 않아도 된다.

`Food` 클래스는 편의 이니셜라이저도 제공한다. 이는 기본 이름을 제공하는 역할을 하고 `[Unnamed]` 값과 함께 `init(name: String)`에 초기화를 위임한다.

```swift
let mysteryMeat = Food()
// mysteryMeat's name is "[Unnamed]"
```

두번째 클래스는 `Food`의 서브클래스 `RecipeIngredient` 이다. 

```swift
class RecipeIngredient: Food {
    var quantity: Int
    init(name: String, quantity: Int) {
        self.quantity = quantity
        super.init(name: name)
    }
    override convenience init(name: String) {
        self.init(name: name, quantity: 1)
    }
}
```

두번째 클래스의 이니셜라이저 구조를 보여주는 그림:

![initializersExample02_2x](https://user-images.githubusercontent.com/48352065/102856745-682e0280-446a-11eb-957f-ada3f4b68b58.png)

`RecipeIngredient` 클래스는 한 개의 지정 이니셜라이저를 가진다. 이 지정 이니셜라이저는 `RecipeIngredient`의 유일한 프로퍼티인 `quantity` 에 값을 할당하면서 시작하고, `Food` 클래스의 `init(name: String)` 이니셜라이저에 초기화 위임을 진행한다. 이 과정은 **안정성 검사 1**을 만족한다. **안정성 검사 1** - *지정 이니셜라이저는 슈퍼클래스의 이니셜라이저(*`init(name: String)`*)에 위임하기 전에 클래스의 모든 프로퍼티(*`quantity`*)가 초기화되도록 해야한다.* 

`RecipeIngredient`는 편의 이니셜라이저도 정의한다. 이는 이름만 사용해서 인스턴스를 생성하는데 사용된다. 이 편의 이니셜라이저는 수량이 명시되지 않은 인스턴스의 수량을 `1`로 추정하여 초기화를 진행한다. 편의 이니셜라이저를 사용하면 빠르고 편리하게 인스턴스를 생성 할 수 있다. 이 편의 이니셜라이저는 `quntity` 값 1과 함꼐 같은 클래스의 지정 이니셜라이저에 초기화를 위임한다. 슈퍼클래스의 지정 이니셜라이저를 재정의 한 것이므로 `override` 수식어를 반드시 붙여야 한다.

더해서 `RecipeIngredient`는 슈퍼클래스의 모든 지정 이니셜라이저를 제공하므로, 슈퍼클래스의 모든 편의 이니셜라이저도 자동으로 상속한다. 즉, 슈퍼클래스의 편의 이니셜라이저 `init()`을 상속받고, 같은 방식으로 동작한다. 다만 상속된 편의 이니셜라이저는 초기화를 슈퍼클래스 버전이 아닌, `RecipeIngredient`버전의 `init(name: String)`에 위임한다.

```swift
let oneMysteryItem = RecipeIngredient() // 상속된 편의 이니셜라이저
let oneBacon = RecipeIngredient(name: "Bacon") // override 한 이니셜라이저
let sixEggs = RecipeIngredient(name: "Eggs", quantity: 6) // 지정 이니셜라이저
```

세번째 클래스 `ShoppingListItem`는 클래스 계층의 마지막 클래스이며, `RecipeIngredient`의 서브클래스이다.

```swift
class ShoppingListItem: RecipeIngredient {
    var purchased = false
    var description: String {
        var output = "\(quantity) x \(name)"
        output += purchased ? " ✔" : " ✘"
        return output
    }
}
```

`ShoppingListItem` 클래스는 모든 프로퍼티의 기본 값을 제공하고 따로 이니셜라이저를 정의하고 있지 않다. 따라서 `ShoppingListItem`는 자동으로 슈퍼클래스의 모든 지정 이니셜라이저와 편의 이니셜라이저를 상속받는다. - **Rule 1**

다음 그림과 같은 구조를 가진다.

![initializersExample03_2x](https://user-images.githubusercontent.com/48352065/102856734-66643f00-446a-11eb-952b-bde52fa6205e.png)

상속된 3가지의 이니셜라이저를 모두 사용하여 인스턴스를 생성할 수 있다.

```swift
var breakfastList = [
    ShoppingListItem(),
    ShoppingListItem(name: "Bacon"),
    ShoppingListItem(name: "Eggs", quantity: 6),
]
breakfastList[0].name = "Orange juice"
breakfastList[0].purchased = true
for item in breakfastList {
    print(item.description)
}
// 1 x Orange juice ✔
// 1 x Bacon ✘
// 6 x Eggs ✘
```

## Reference

[docs.swift.org](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html)
