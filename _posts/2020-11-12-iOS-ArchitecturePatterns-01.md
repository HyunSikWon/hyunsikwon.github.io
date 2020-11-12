---

title: iOS Architecture Pattern(MVC, MVP, MVVP)을 알아보자

layout: single

comments: true

share: true

categories:

-  iOS

tag:

- Architecture Pattern

last_modified_at: 2020-11-12 T14:00:00+08:00

---

## 좋은 Architecture의 특징

1. 엄격한 룰에 따라 개체들 간의 책임이 균형있게 **분배되어야 한다. -(distribution)**
2. 첫번째 특징으로 분리된 개체들에 **테스트를 적용**할 수 있어야 한다. - **testability**
3. 사용과 유지 보수가 쉬워야 한다.- **Ease of use**

## MV(X)

아키텍쳐 디자인 패턴에는 몇 가지가 존재한다.

- MVC
- MVP
- MVVM
- VIPER

처음 3개의 경우 앱의 개체를 3개의 카테고리로 나눈다.

- **Models:** 데이터 혹은 데이터를 조작하는 데이터 접근 계층을 위한 역할을 한다.
- **Views:** 표현 계층에 대한 역할, iOS 환경의 경우 'UI' 접두사로 시작하는 모든 경우를 생각하면 된다.
- **Controller/Presenter/ViewModel:** Model과 View 사이에서 연결 혹은 중재자 역할을 한다. 보통 View에서의 사용자의 동작에 반응하여 Model에 알리고, Model의 변화로 부터 View를 수정하는 역할을 한다.

위와 같은 분리를 통해서 우리는 분리된 개체들을 **더 잘 이해할 수 있고.  재사용이 가능하며. 독립적으로 테스트 할 수 있다.**

이제 아키텍쳐 패턴을 하나씩 살펴보자.

## MVC

![Traditional MVC](https://user-images.githubusercontent.com/48352065/98896270-ef8f6a00-24eb-11eb-8b66-bec549982df2.png)

 전통적인 MVC는 3개의 개체가 강하게 결합되어 있기 때문에 각각의 개체는 다른 두개의 개체를 알고 있어야한다. 이는 각각의 개체의 재사용성을 떨어트리기 때문에 앱에 적용하기 적절치 않다.

> 전통적인 MVC는 최신 iOS 앱에 적용하기에는 적절하지 않다.


## Apple's MVC

![Cocoa MVC](https://user-images.githubusercontent.com/48352065/98896269-eef6d380-24eb-11eb-8a80-44dccd21e856.png)

**Controller**는 **View**와 **Model** 사이의 중재자 역할을 하고 따라서 **View**와 **Model**을 서로에 대해 알고 있을 필요가 없다. 여기서 가장 재사용성이 떨어지는 것은 **Controller**인데, 우리는 Model에 넣기에는 적절하지 않은 복잡한 로직들을 위한 장소가 필요하기 때문에 상관 없다.

*하지만 현실은....*

![Realistic Cocoa MVC](https://user-images.githubusercontent.com/48352065/98896268-ee5e3d00-24eb-11eb-95ef-ce7074afd105.png)

하지만 **View Controller**는 따로 분리하기 어려운 **View**의 life cycle과  얽혀있기 때문에 Cocoa MVC는 **Massive** View Controller를 유발한다. 비록 몇몇 로직과 데이터 변환을 **Model**에 넘겼지만, 대부분의 경우 **View**의 역할은 **Controller**에 action을 보내는 것이기 때문에 **View**와 관련된 작업을 분리하는 것은 쉽지 않다. **View controller**는 결국 여러 **View**와 관련한 작업과 네트워킹 작업 등으로 인해 **Massive** **View Controller**가 될수 밖에 없다.

아마 다음과 같은 코드를 본적 있을 것이다.

```swift
var userCell = tableView.dequeueReusableCellWithIdentifier("identifier") as UserCell
userCell.configureWithUser(user)
```

여기서 Cell(**View**)은 직접 **Model**을 통해서 구성되어 MVC 가이드라인을 위반한다. 하지만 이는 모든 경우에 발생하고 대부분의 사람들이 잘못됐다고 생각하지 않는다. MVC를 정확히 따르기 위해서는 Cell(**View**)을 **Controller**를 통해 구성해야하고, **Model**을 **View**에 전달하지 않아야 하는데, 이는 **Controller**의 크기를 증가시키게 된다.

이 문제는 Unit Test를 수행하기 전까지는 명확하게 드러나지 않는다. View controller는 View와 강하게 결합되어 있어 테스트를 위한 View와 life cycle을 만들어야 하기 때문에 테스트하기 어려워 질 것이다. 

앞선 내용을 보면 Cocoa MVC는 선택하기에 좋지 않은 패턴처럼 보인다. 글 초반에 살펴본 **특징**의 관점에서 Cocoa MVC를 살펴보자:

- **Distribution: View**와 **Model**은 분리되어 있지만 **View**와 **Controller**는 강하게 결합되어 있다.
- **Testability:** 분리가 잘 되어있지 않기 때문에 **Model**에 대한 테스트만 가능할 것이다.
- **Ease of use:** 다른 패턴들 중 가장 작은 양의 코드를 갖는다. 또한 많은 사람들이 친숙하고 경험이 적은 개발자도 쉽게 사용 가능하다.

Cocoa MVC는 아키텍쳐에 투자할 시간이 적을 때, 혹은 작은 프로젝트에 높은 유지 보수 비용이 과하다고 느껴질 때 적절한 아키텍쳐 패턴이다. 

> Cocoa MVC는 빠른 개발 속도의 관점에서는 좋은 아키텍쳐 패턴이다.


## MVP

![Passive View variant of MVP ](https://user-images.githubusercontent.com/48352065/98896265-ed2d1000-24eb-11eb-8316-c018fdcb2e08.png)

Apple의 MVC와 매우 흡사한 모습을 보이지만 Apple의 MVC가 사실 MVP라는 의미는 아니다. Cocoa MVC는 **View**와 **Controller**가 강하게 결합되는데 반해, MVP의 중재자 **Presenter**는 view controller의 생명주기와 아무런 관계가 없다. 따라서 테스트를 위한 **View**를 쉽게 만들 수 있고, layout과 관련한 코드는 더 이상 **Presenter**에 존재하지 않는다. **Presenter**는 단지 **View**의 데이터와 상태를 갱신하기위한 역할만 한다.

**MVP**의 관점에서 UIViewController 서브클래스는 **View**이며 **Presenter**가 아니다. 이러한 분리로 인해 테스트를 적용하기에 아주 좋다.

MVP 패턴은 실제 3개의 레이어로 분리 됐음에도 불구하고 결합 문제가 들어난 첫번째 패턴이다.

**MVP의 특징:**

- **Distribution:** 대부분의 책임이 Presenter와 Model로 나누어져 있고 View는 거의 아무런 작업을 안한다.(Dumb라고 원문에서 표현됨)
- **Testability:** 테스트하는데 매우 훌륭한 코드이다. 대부분의 비지니스 로직들을 테스트할 수 있다.
- **Ease of use:** MVC와 비교했을 때 많은 양의 코드가 필요하다. 하지만 그와 동시에 MVP에 대한 아이디어는 매우 분명하다. - 각 개체의 역할이 분명하게 나뉨.

> iOS에서 MVP는 많은 양의 코드가 필요하지만 테스트하기 아주 적합한 형태이다.


## MVVM

**MV(X) 중 가장 최신이자 훌륭한 방식**

MVVM에서 **Model**과 **View**는 우리가 지금까지 살펴본 방식과 같고, 중재자의 역할은 **ViewModel**이 담당한다. 또한, MVVM에서는 View와 ViewModel 사이에는 데이터 바인딩이 존재한다. 

![MVVM](https://user-images.githubusercontent.com/48352065/98896260-eacab600-24eb-11eb-9291-54bf5bfdf2b1.png)

MVVM은 MVP와 유사한 형태를 띈다.

- MVVM은 view controller를 **View**로 취급한다.
- **View**와 **Model** 사이에 강한 결합이 존재하지 않는다.

MVVM의 기본 개념은 View와 View의 상태를 UIKit 독립적으로 표현하는 것이다. **ViewModel**은 **Model**의 변경을 일으키고, **변경된** **Model**을 사용하여 자기 자신의 상태를 갱신한다. View와 ViewModel 사이에는 데이터 바인딩이 존재하기 때문에 **View** 역시 갱신된다.

**MVVM의 특징:**

- **Distribution:** MVP의 **View**보다 MVVM의 View가 가진 책임이 더 크다. **MVVM**의 **View**는 바인딩을 이용하여 **ViewModel**의 상태에 따라 **자신(View)**의 상태를 직접 갱신한다. 반면 **MVP**는 발생하는 모든 이벤트를 **Presenter**에 넘기고 **자신(View)**의 상태를 직접 갱신하지 않는다.
- **Testability: ViewModel**은 **View**에 대해 아무것도 알지 못하기 때문에 테스트를 적용하기 쉽고, **View** 역시 테스트하기 수월하다.
- **Ease of use:** 바인딩을 사용하면 모든 View의 이벤트를 Presenter에 알리고 수동으로 View를 갱신해야하는 MVP와 달리 MVVM은 보다 간결하게 작성 가능하다.

> MVVM은 앞서 살펴본 방법들의 이점들을 결합한 매력적인 방법이다. 또한, 바인딩을 사용하여 View의 갱신을 위한 추가적인 코드를 작성할 필요가 없다. 테스트를 적용하기 매우 좋은 방식이다.

## Reference

[iOS Architecture Patterns](https://medium.com/ios-os-x-development/ios-architecture-patterns-ecba4c38de52)
