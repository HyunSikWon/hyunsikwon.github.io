---

title: iOS Architecture Pattern(MVC, MVP, MVVP, VIPER)를 알아보자

layout: single

comments: true

share: true

categories:

-  iOS

tag:

- Architecture Pattern

last_modified_at: 2021-03-24 T13:00:00+08:00

---

## 좋은 Architecture의 특징

1. 엄격한 룰에 따라 개체들 간의 책임이 균형있게 분배되어야 한다. (distribution)
2. 첫번째 특징으로 분리된 개체들에 테스트를 적용할 수 있어야 한다. (testability)
3. 사용과 유지 보수가 쉬워야 한다. (Ease of use)

## MV(X)

대표적인 아키텍쳐 패턴은 다음과 같다:

- MVC
- MVP
- MVVM
- VIPER

MV(X) 패턴의 경우 앱의 개체를 3개의 레이어로 나눈다.

- Models: 데이터 혹은 데이터를 조작하는 데이터 접근 레이어를 위한 역할을 한다.
- Views: 표현 레이어에 대한 역할을 한다. iOS 환경의 경우 'UI' 접두사로 시작하는 모든 것들을 생각하면 된다.
- Controller/Presenter/ViewModel: Model과 View 사이에서 연결 혹은 중재자 역할을 한다. 보통 View에서의 사용자의 동작에 반응하여 Model에 알리고, Model의 변화로 부터 View를 수정하는 역할을 한다.

이를 통해 우리는 분리된 개체들을 더 잘 이해할 수 있고, 재사용이 가능하며, 독립적으로 테스트 할 수 있다. 이제 아키텍쳐 패턴을 하나씩 살펴보자.


## MVC

![1*E9A5fOrSr0yVmc7Kly5C6A](https://user-images.githubusercontent.com/48352065/108462668-5cbb5400-72c0-11eb-8fa8-c6386808128f.png)

 전통적인 MVC는 3개의 개체가 강하게 결합되어 있기 때문에 각각의 개체는 다른 두개의 개체를 반드시 알고 있어야한다. 이는 각각의 개체의 재사용성을 떨어트리기 때문에 앱에 적용하기 적절치 않다.

> 전통적인 MVC는 최신 iOS 앱에 적용하기에는 적절하지 않다.

## Apple's MVC

![1*c0aGaDNX41qu6e8E4OEgwQ](https://user-images.githubusercontent.com/48352065/108462666-5c22bd80-72c0-11eb-99bf-7d536d5c0926.png)

`Controller`는 `View`와 `Model` 사이의 중재자 역할을 하며, 따라서 `View`와 `Model`은 서로에 대해 알고 있을 필요가 없다. 여기서 가장 재사용성이 떨어지는 것은 `Controller`인데, 우리는 `Model`에 넣기에는 적절하지 않은 복잡한 로직들을 위한 장소가 필요하기 때문에 상관 없다.

*하지만 현실은....*

![1*PkWjDU0jqGJOB972cMsrnA](https://user-images.githubusercontent.com/48352065/108462665-5c22bd80-72c0-11eb-8a1e-94f164a4ccac.png)

하지만 view controller는 따로 분리하기 어려운 `View`의 life cycle과  얽혀있기 때문에 Cocoa MVC는 Massive View Controller를 유발한다. 비록 몇몇 로직과 데이터 변환을 `Model`에 넘겼지만, 대부분의 경우 `View`의 역할은 `Controller`에 action을 보내는 것이기 때문에 `View`와 관련된 작업을 분리하는 것은 쉽지 않다. View controller는 결국 여러 `View`와 관련한 작업과 네트워킹 작업 등으로 인해 Massive View Controller가 될수 밖에 없다.

다음 코드를 살펴보자.

```swift
var userCell = tableView.dequeueReusableCellWithIdentifier("identifier") as UserCell
userCell.configureWithUser(user)
```

여기서 Cell(`View`)은 직접 `Model`을 통해 구성되어 MVC 가이드라인을 위반한다. 하지만 이는 많은 경우에서 흔히 일어나고 대부분의 사람들이 잘못됐다고 생각하지 않는다. MVC를 정확히 따르기 위해서는 Cell(`View`)을 `Controller`를 통해 구성해야하고, `Model`을 `View`에 전달하지 않아야 한다. 

### MVC의 특징

- Distribution: `View`와 `Model`은 분리되어 있지만 `View`와 `Controller`는 강하게 결합되어 있다.
- Testability: 분리가 잘 되어있지 않기 때문에 `Model`에 대한 테스트만 가능할 것이다.
- Ease of use: 다른 패턴들 중 가장 적은 양의 코드를 갖는다. 또한, 많은 사람들에게 친숙하고 경험이 적은 개발자도 쉽게 사용 가능하다.

> Cocoa MVC는 아키텍쳐에 투자할 시간이 적을 때, 혹은 작은 프로젝트에 높은 유지 보수 비용이 과하다고 느껴질 때 적절한 아키텍쳐 패턴이다. 따라서, Cocoa MVC는 빠른 개발 속도의 관점에서는 좋은 아키텍쳐 패턴이다.

## MVP

![1*hKUCPEHg6TDz6gtOlnFYwQ](https://user-images.githubusercontent.com/48352065/108462664-5af19080-72c0-11eb-84a3-768b3194a67a.png)

MVP 패턴은 Apple의 MVC와 매우 유사한 모습을 보이지만 Apple의 MVC가 사실 MVP라는 의미는 아니다. Cocoa MVC는 View와 Controller가 강하게 결합되는데 반해, MVP의 중재자 `Presenter`는 view controller의 생명주기와 아무런 관계가 없다. 따라서 테스트를 위한 `View`를 쉽게 만들 수 있고, layout과 관련한 코드는 더 이상 `Presenter`에 존재하지 않는다. `Presenter`는 단지 `View`의 데이터와 상태를 갱신하기위한 역할만 한다.

MVP의 관점에서 `UIViewController` 서브클래스는 `View`이며 `Presenter`가 아니다. 이러한 분리로 인해 테스트를 적용하기에 아주 좋다.

### MVP의 특징:

- Distribution: 대부분의 책임이 `Presenter`와 `Model`로 나누어져 있고 `View`는 거의 아무런 작업을 안한다.(Dumb라고 원문에서 표현됨)
- Testability: 테스트하는데 매우 훌륭한 코드이다. 대부분의 비지니스 로직들을 테스트할 수 있다.
- Ease of use: MVC와 비교했을 때 많은 양의 코드가 필요하다. 하지만 그와 동시에 MVP에 대한 아이디어는 매우 분명하다. - 각 개체의 역할이 분명하게 나뉜다.

> iOS에서 MVP는 많은 양의 코드가 필요하지만 테스트하기 아주 적합한 형태이다.

## MVVM

MV(X) 중 가장 최신이자 훌륭한 방식이다. MVVM에서 `Model`과 `View`는 우리가 지금까지 살펴본 방식과 같고, 중재자의 역할은 `ViewModel`이 담당한다. 또한, MVVM에서는 `View`와 `ViewModel `사이에는 데이터 바인딩이 존재한다. 

![1*uhPpTHYzTmHGrAZy8hiM7w](https://user-images.githubusercontent.com/48352065/108462655-588f3680-72c0-11eb-81aa-48edabeac6ce.png)

MVVM은 MVP와 유사한 형태를 띈다.

- MVVM은 view controller를 `View`로 취급한다.
- `View`와 `Model` 사이에 강한 결합이 존재하지 않는다.

MVVM의 기본 개념은 `View`와 `View`의 상태를 UIKit 독립적으로 표현하는 것이다. `ViewModel`은 `Model`의 변경을 일으키고, 변경된 `Model`을 사용하여 자기 자신의 상태를 갱신한다. `View`와 `ViewModel` 사이에는 데이터 바인딩이 존재하기 때문에 `View` 역시 갱신된다.

### MVVM의 특징:

- Distribution: MVP의 `View`보다 MVVM의 `View`가 가진 책임이 더 크다. MVVM의 `View`는 바인딩을 이용하여 `ViewModel`의 상태에 따라 자신(`View`)의 상태를 직접 갱신한다. 반면 MVP는 발생하는 모든 이벤트를 `Presenter`에 넘기고 자신(`View`)의 상태를 직접 갱신하지 않는다.
- Testability: `ViewModel`은 `View`에 대해 아무것도 알지 못하기 때문에 테스트를 적용하기 쉽고, `View` 역시 테스트하기 수월하다.
- Ease of use: 바인딩을 사용하면 모든 `View`의 이벤트를 `Presenter`에 알리고 수동으로 `View`를 갱신해야하는 MVP와 달리 MVVM은 보다 간결하게 작성 가능하다.

> MVVM은 앞서 살펴본 방법들의 이점들을 결합한 매력적인 방법이다. 또한, 바인딩을 사용하여 View의 갱신을 위한 추가적인 코드를 작성할 필요가 없다. 테스트를 적용하기 매우 좋은 방식이다.
> 

## VIPER

VIPER는 위에서 본 패턴들과 달리 개체간 책임을 나누기 위해 5가지 레이어를 가진다. 

![1*0pN3BNTXfwKbf08lhwutag](https://user-images.githubusercontent.com/48352065/112256687-8db7eb80-8ca7-11eb-9fcb-55990938cd22.png)

- Interactor: 데이터와 과련된 비지니스 로직 혹은 네트워킹을 수행한다. 엔티티의 새로운 인스턴스를 생성하거나 서버로부터 불러오는 작업 등을 한다.
- Presenter: UI와 관련된 비지니스 로직을 가진다.
- Entities: 단순한 데이터 객체이다. 데이터 액세스 레이어는 아니며 이는 Interactor가 수행한다.
- Router: VIPER 모듈 사이의 세그웨이를 위한 책임을 가진다.

MV(X) 패턴들과 비교해보면:

- Model 로직은 Entities와 함께 Interator로 이동했다.
- Controller/Presenter/ViewModel의 UI 표현 작업은 Presenter로 이동했다. 다만, Presenter는 데이터 변경 작업을 하지 않는다.
- VIPER는 네비게이션 책임을 명시적으로 다루는 첫번재 패턴이다. 이는 Router를 통해서 수행한다.

### VIPER의 특징

- Distribution: 아주 훌륭하게 책임들을 분리한다.
- Testability: 책임이 확실하게 분리되어 테스트를 적용하기 좋다.
- Ease of use: 다만, 작은 책임을 가진 클래스를 위한 많은 양의 인터페이스를 작성해야한다.


## Reference

[iOS Architecture Patterns](https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52)
