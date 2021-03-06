---
title: Swift Access Control
layout: single
comments: true
share: true
categories: 
- Swift
tag:
- Access Control
last_modified_at: 2021-02-18 T16:30:00+08:00
---

접근 제한(Access control)은 다른 소스파일이나 모듈에서 코드 일부에 접근하는 것을 제한한다. 이를 통해 코드의 상세 구현은 숨기고 허용된 기능에만 접근하고 사용할 수 있는 인터페이스를 구현할 수 있다. 

스위프트의 접근 제한 모델은 모듈과 소스파일의 개념에 기초한다. 모듈은 배포할 코드의 묶음 단위이며, 통상 하나의 프레임워크나 어플리케이션이 모듈 단위가 될 수 있다. 스위프트에서 `import` 키워드를 통해 불러올 수 있다. 소스파일은 하나의 소스 코드 파일을 의미한다. 스위프트에서 어떠한 새로운 타입, 함수, 프로퍼티를 정의할 땐 기본 접근 수준으로 `internal`이 지정된다. 즉, 이들은 모두 같은 모듈내에 다른 코드에서 접근할 수 있다는 것이다.

예제를 통해 살펴보자. 우리는 쇼핑 앱을 개발 중이고 제품 배열을 통해 전체 가격을 계산하는 `PriceCalculator` 클래스를 다음과 같이 정의했다.

```swift
class PriceCalculator {
    func calculatePrice(for products: [Product]) -> Int {
        // The reduce function enables us to reduce a collection,
        // in this case an array of products, into a single value:
        products.reduce(into: 0) { totalPrice, product in
            totalPrice += product.price
        }
    }
}
```

현재 명시적으로 어떠한 접근 레벨도 지정하지 않았고, 따라서 이 클래스는 앱 내의 어디에서든 접근 가능하다. 그러나 우리가 이 클래스를 다른 모듈과 공유하고 싶다면 이 클래스를 `public`으로 지정해야 한다.

```swift
public class PriceCalculator {
    public func calculatePrice(for products: [Product]) -> Int {
        products.reduce(into: 0) { totalPrice, product in
            totalPrice += product.price
        }
    }
}
```

이제 `PriceCalculator` 클래스를 다른 모듈에서 찾을 수는 있지만 클래스의 이니셜라이저가 기본(default) 접근 레벨인 `internal`로 지정되어 있기 때문에 클래스의 인스턴스를 생성할 수는 없다. 이를 해결하기 위해 `public` 이니셜라이저를 정의하자. 실제 작업은 이루어지지 않기 때문에 이니셜라이저 내부는 비워둔다.

```swift
public class PriceCalculator {
    public init() {}
    ...
}
```

이제 모듈 내외부 모두에서 접근, 초기화, 호출할 수 있다. 그러나 우리가 이 클래스를 수정하기 위해 상속하거나 기능을 추가하고 싶기도 하다. 이는 같은 모듈 내에서는 가능하지만 모듈 밖에서는 불가능하다. 이를 위해서 현재 스위프트에서 가장 낮은(open) 수준의 접근 레벨을 지정하자. `open` 키워드를 사용한다.

```swift
open class PriceCalculator {
    ...
}
```

이제 위의 변경 사항을 적용하여 어디에서나 `PriceCalculator`의 사용자 지정 하위 클래스를 만들 수 있다. 여기에는 새 이니셜라이저, 새 프로퍼티 및 새 메소드가 있을 수 있다. 이를 사용하여 모든 가격 계산에 지정된 할인을 적용 할 수 있는 `DiscountedPriceCalculator`를 구현하였다.

```swift
class DiscountedPriceCalculator: PriceCalculator { 
    let discount: Int

    init(discount: Int) {
        self.discount = discount
        super.init()
    }

    func calculateDiscountedPrice(for products: [Product]) -> Int {
        let price = calculatePrice(for: products)
        return price - discount
    }
}
```

위에서는 완전히 새로운 가격 계산 방법을 정의했지만 사실, 상위 클래스에서 상속한 기존 `calculatePrice` 메소드를 재정의하여(override) 수정하는 것이 훨씬 더 적절할 것이다. 이렇게 하면 메소드 호출 과정에서 혼동이 없고 두 클래스를 일관되게 유지할 수 있다. 그렇게 하려면 `calculatePrice` 메소드를 `open`으로 지정해야 한다.

```swift
open class PriceCalculator {
    public init() {}

    open func calculatePrice(for products: [Product]) -> Int {
        ...
    }
}
```

이제 자유롭게 `calculatePrice`를 재정의할 수 있다.

```swift
class DiscountedPriceCalculator: PriceCalculator {
    let discount: Int

    init(discount: Int) {
        self.discount = discount
        super.init()
    }

    override func calculatePrice(for products: [Product]) -> Int {
        let price = super.calculatePrice(for: products)
        return price - discount
    }
}
```

다른 방법으로 코드의 일부가 발견되고 사용되지 않도록 숨길 수도 있다. 이 방법의 무슨 의미가 있는지 의심스러울 수 있지만 API를 훨씬 더 좁고 집중적으로 만드는데 도움이 될 수 있고 이로인해 API의 이해와, 테스트 및 사용이 더 쉬워 질 수 있다.

이제 접근 레벨 스펙트럼의 반대쪽으로 이동하여 가장 제한적인 수준인 `private`을 살펴 보자. `private`으로 표시된 모든 타입, 프로퍼티 또는 메소드는 자체 타입 내에서만 표시된다(동일한 파일 내에 정의된 해당 타입의 익스텐션 포함).

주어진 타입의 프라이빗한 구현 세부 사항은 분명하게 `private`으로 표시되어야 한다. 예를 들어 가격 계산기의 `discount` 프로퍼티는 실제로 자체 클래스 내에서만 사용하기 위한 것이므로 계속해서 해당 속성을 `private`으로 지정하자.

```swift
class DiscountedPriceCalculator: PriceCalculator {
    private let discount: Int
    ...
}
```

만약 이 프로퍼티에 대한 접근 레벨을 조금 확장하고 싶다면(같은 파일 내의 다른 타입에서 접근하고 싶다면) `fileprivate`을 사용한다. 

```swift
class DiscountedPriceCalculator: PriceCalculator {
    fileprivate let discount: Int
    ...
}
```

위의 변경 사항을 적용하여 이제 동일한 파일에 정의된 관련 코드 (예시 : UIAlertController의 익스텐션)에서 `discount` 프로퍼티에 액세스 할 수 있다. 이를 통해 알림(alert)에서 제품 배열에 대한 가격 설명을 쉽게 표시 할 수 있다.

```swift
extension UIAlertController {
    func showPriceDescription(
        for products: [Product],
        calculator: DiscountedPriceCalculator
    ) {
        let price = calculator.calculatePrice(for: products)

        // We can now access 'discount' even outside of the type
        // that it's declared in, thanks to 'fileprivate':
        message = """
        Your \(products.count) product(s) will cost \(price).
        Including a discount of \(calculator.discount).
        """
    }
}
```

> `private`과 `fileprivate`은 타입 내에 정의된 것들에만 차이가 있다.

앞서 살펴본 다섯가지 접근 레벨을 요약하자면:

- `private`은 동일한 파일 내에 정의된 해당 타입의 익스텐션을 포함하여 둘러싸는 타입 내에서 프로퍼티 또는 함수를 비공개로 유지한다. 최상위(top-level) 타입, 함수 또는 익스텐션에 적용하면 `fileprivate`과 동일한 방식으로 작동한다.
- `fileprivate`은 정의된 전체 파일 내에서 선언(declaration)을 표시하고 다른 모든 코드에서는 숨긴다.
- `internal`은 기본 액세스 수준이며 정의된 전체 모듈 내에서 선언을 표시한다.
- `public`은 모듈 외부에 함수, 타입, 익스텐션 또는 프로퍼티를 표시한다.
- `open`은 모듈 외부에서 클래스를 서브클래싱하고 함수 또는 프로퍼티를 재정의 할 수 있도록 한다.

## Reference

[Swift by Sundell](https://www.swiftbysundell.com/basics/access-control/)
