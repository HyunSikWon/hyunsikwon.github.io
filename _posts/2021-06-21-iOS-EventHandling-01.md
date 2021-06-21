---
title: Using Responders and the Responder Chain to Handle Events
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- Event Handling
last_modified_at: 2021-06-21 T15:04:40+08:00
---

## Overview

앱은 responder 객체를 사용해서 이벤트를 받고 처리한다. Responder 객체는 `UIResponder` 클래스의 인스턴스를 말하며 서브클래스인 `UIView`, `UIViewController`, `UIApplication`도 이에 해당된다. Responder는 이벤트 데이터를 받아 이벤트를 처리하거나 다른 responder 객체에 전달하는 역할을 한다. 앱이 이벤트를 받으면 가장 적절한 responder 객체에게 이벤트를 전달하는데 이를 *first responder*라고 부른다.

first responder에서 이벤트를 처리하지 않는다면 해당 이벤트는 responder chain를 거쳐 다른 responder에게 전달된다. Responder chain에 이벤트를 처리할 responder가 존재하면 해당 객체가 적절한 이벤트 처리 작업을 수행하지만 그렇지 않다면 이벤트는 사라진다. 다음 그림은 responder chain에서 어떻게 이벤트가 responder에게 전달되는지를 보여준다.

![f17df5bc-d80b-4e17-81cf-4277b1e0f6e4](https://user-images.githubusercontent.com/48352065/122714102-c022b600-d2a1-11eb-8c70-4549b813b528.png)

만약 텍스트 필드가 이벤트를 처리하지 않는다면 UIKit은 이벤트를 부모 객체인 `UIView` 객체와 `UIWindow`의 루트 뷰를 거쳐 전달된다. 루트 뷰에서는 responder chain이 `UIWindow`로 바로 향하지 않고 `UIViewController` 로 우회하여 전달된다.

## First Responder 결정

UIKit은 다음과 같이 여러 이벤트 타입에 따라 first responder를 결정한다.
| 이벤트 타입 | First Responder |
|----------|-----------------|
| 터치 이벤트 | 터치가 발생한 뷰 |
|프레스 이벤트 | 포커스를 가진 뷰|
|흔들기 이벤트 | 사용자나 UIKit이 지정한 뷰|
|원격 이벤트 | 사용자나 UIKit이 지정한 뷰|
|편집 메뉴 이벤트 | 사용자나 UIKit이 지정한 뷰|



## UIControl

UIControl은 특정 액션이나 사용자의 의도(드래그, 버튼 클릭 등등)를 전달하는 시각적인 요소들의 기반이 되는 클래스이다. UIControl 클래스를 상속하는 클래스로는 대표적으로 UIButton 클래스가 있다. 즉, 사용자에게 보여지는 뷰가 사용자와 상호작용할 수 있게끔 능력을 부여해주는 클래스라고 설명할 수 있다. UIControl은 Target-Action이라는 매커니즘을 이용해 사용자의 액션들을 앱에 전달한다. 비록 액션 메시지는 이벤트는 아니자만, Responder Chain을 이용하게 되는데, 만약 특정 컨트롤에서의 타겟이 nil이라면 UIKit은 해당 컨트롤을 가진 뷰에서부터 시작해서 Responder Chain을 따라가며 적절한 액션 메소드를 찾게 된다.

## 어떤 Responder가 이벤트를 가지고 있는지 확인

UIKit은 hit-testing을 통해 터치 이벤트가 발생한 최상단 뷰를 찾으며 그렇게 찾은 뷰가 해당 이벤트를 처리할 수 있는 첫 번째 뷰, 즉 First Responder가 되는 것이다. 터치가 발생한 위치와 계층 내 뷰 객체의 bounds를 비교하는 방식으로 수행된다. 

터치 이벤트가 발생하면 UIKit은 `UITouch` 객체를 만들어서 뷰와 연결한다. 터치 위치가 바뀌거나 기타 매개 변수들이 바뀌어도 동일한 `UITouch` 객체를 사용하여 정보만 변경해준다. 바뀌지 않는 프로퍼티는 view로, 터치 위치가 뷰를 벗어나더라도 이 값은 변하지 않는다. 터치가 끝나게 되면 UIKit은 사용했던 `UITouch` 객체를 해제합니다.

### Hit Testing

Hit-Testing을 진행할 때는 뷰 계층에 있어서 최상위 뷰(`UIWindow`)가 `hitTest(:with:)` 메소드를 호출하며 검사를 진행한다. `hitTest(:with:)` 메소드는 내부적으로 `point(inside:with:)` 메소드를 호출하고 이 메소드를 통해 현재 검사하고 있는 뷰가 터치 이벤트가 발생한 지점(Point)을 포함하는지 검사한다. 만일 포함하지 않는다면 `false`를 반환하고 해당 뷰의 subviews들은 검사하지 않고 다음 뷰로 넘어가게 됩니다. 포함한다면 `true`를 반환한다. 이렇게 `true`가 반환된다면 해당 뷰의 subviews들을 순회하며 `hitTest(:with:)`를 호출하면서 터치가 일어난 최상단 뷰를 찾는 것이다. 그래서 최종적으로 `hitTest(:with:)`는 터치가 일어난 최상단 뷰를 반환하게 된다. 여기서 최상단은 뷰 계층의 최상단이 아닌 뷰 스택에서의 최상단을 말한다. 

> `hitTest(_:with:)` 메소드는 기본적으로 `isHidden = true`, `isUserInteractionEnabled = false` 또는 `alpha` 값이 0.01 미만인 뷰의 검사는 진행하지 않고 해당 뷰의 subviews 또한 검사를 진행하지 않는다.

## References

[https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events)

[https://baked-corn.tistory.com/125](https://baked-corn.tistory.com/125)
