---
title: Custom View!!
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- Custom View
last_modified_at: 2021-01-18 T23:35:00+08:00
---


# Defining a Custom View

커스텀 뷰(Custom View)를 사용하면 앱의 컨텐츠의 외형(appearance)과 해당 컨텐츠와의 상호 작용이 처리되는 방식을 완벽하게 제어 할 수 있다.

## Checklist for Implementing a Custom View

커스텀 뷰의 역할은 컨텐츠를 표시하고 해당 컨텐츠와의 상호 작용을 관리하는 것이다. 물론 커스텀 뷰의 성공적인 구현은 이벤트를 그리고 처리하는 것 이상의 것이 포함된다. 다음 체크리스트에는 커스텀 뷰를 구현할 때 오버라이드 할 수 있는 중요한 메소드(및 제공 할 수있는 동작)이 포함되어 있다.

- 뷰를 위한 적절한 초기화 메소드를 정의한다.
    - 코드로 뷰를 생성하려면 (programmatically), `initWithFrame:` 메소드를 오버라이드 하거나, 커스텀 초기화 메소드를 정의해야한다.
    - nib 파일을 통해서 뷰를 생성하려면, `initWithCoder:` 메소드를 오버라이드 해야한다. 이 메소드를 사용하여 뷰를 초기화해야한다.
- 커스텀 데이터를 클린업하기 위해서 `dealloc` 메소드를 구현해야한다.
- 커스텀 드로일을 처리하기 위해서, `drawRect:` 메소드를 오버라이드하고 드로잉 작업을 해야한다.
- `autoresizingMask` 프로퍼티를 설정하여 autoresizing 동작을 정의해야 한다.
- 만약 뷰 클래스가 하나 이상의 필수(integral) 서브뷰를 관리한다면, 다음의 것들을 해야한다.
    - 뷰의 초기화 시퀀스 중 이 서브뷰들을 생성해야한다.
    - 생성 시점에서 각 서브뷰의 `autoresizingMask` 프로퍼티를 설정한다.
    - 만약 서브뷰들이 커스텀 레이아웃을 필요로 한다면, `layoutSubviews` 메소드를 오버라이드 하고, 레이아웃 코드를 여기에 구현해야한다.
- 터치 기반 이벤트를 처리하기 위해선 다음의 작업이 필요하다:
    - `addGestureRecognizer:` 메소드를 사용하여 적절한 제스처 인식기(recognizers)를 추가해야한다(attach).
    - 터치 작업을 처리하기 위해서,`touchesBegan:withEvent:`, `touchesMoved:withEvent:`, `touchesEnded:withEvent:`, `touchesCancelled:withEvent:` 메소드를 오버라이드 해라. 어떤 터치 관련 메소드를 오버라이드 하는지는 상관없이 언제나 `touchesCancelled:withEvent` 메소드는 오버라이드 해야한다.
- 출력버전을 화면에 나타나는 버전과 다르게 하고 싶다면 `drawRect:forViewPrintFormatter:` 메소드를 구현해야한다.

메소드를 재정의하는 것 외에도 뷰의 기존 프로퍼티 및 메서드로 할 수 있는 작업이 많다는 점도 기억해라.

## Initializing Your Custom View

우리가 정의하는 모든 새로운 뷰 객체는 `initWithFrame:` 초기화 메소드를 포함해야한다. 이 메소드는 생성시에 클래스를 초기화하고 뷰 객체를 알려진 상태(known state)로 만드는 역할을 한다. 프로그래밍 방식으로 뷰의 인스턴스를 만들때 이 메소드를 사용한다.

nib 파일에서 커스텀 뷰 클래스의 인스턴스를 로드하려는 경우에는, nib 로딩 코드는 새 뷰 객체를 인스턴스화하기 위해 `initWithFrame:` 메소드를 사용하지 않는다는 점을 알아야한다. 대신 `NSCoding` 프로토콜의 일부인 `initWithCoder:` 메소드를 사용한다. 뷰가 `NSCoding` 프로토콜을 채택하더라도 Interface Builder는 뷰의 커스텀 프로퍼티에 대해 알지 못하므로 해당 프로퍼티를 nib 파일로 인코딩하지 않는다. 결과적으로 자신의 `initWithCoder:` 메소드는 뷰를 알려진 상태(known state)로 만들기 위해 가능한 모든 초기화 코드를 수행해야한다. 뷰 클래스에서 `awakeFromNib` 메소드를 구현하고 해당 메소드를 사용하여 추가 초기화를 수행 할 수도 있다.

## Implementing Your Drawing Code

뷰의 커스텀 드로잉이 필요하다면, `drawRect:` 메소드를 오버라이드 해서 드로잉 작업을 수행해야한다. 커스텀 드로잉은 최후의 수단으로만 권장된다. 일반적으로 다른 뷰를 사용하여 컨텐츠를 표시 할 수 있는 경우 이를 선호한다. 

`drawRect :` 메소드의 구현은 정확히 한 가지를 수행해야한다 : 콘텐츠를 그리는 것. 이 방법은 앱의 데이터 구조를 업데이트하거나 드로잉과 관련이 없는 작업을 수행하는 곳이 아니다. 드로잉 환경을 구성하여 컨텐츠를 그리고 최대한 빨리 종료해야한다. 그리고 `drawRect :` 메소드가 자주 호출 될 수 있는 경우에는 드로잉 코드를 최적화 하여 메소드가 호출 될 때 가능한 한 적게 그릴 수 있도록 해야한다.

## Responding to Events

뷰 객체는 `UIResponder` 클래스의 인스턴스인 리스폰더 객체이므로 터치 이벤트를 수신 할 수 있다. 터치 이벤트가 발생하면 윈도우는 해당 이벤트 객체를 터치가 발생한 뷰로 전달한다. 뷰가 이벤트를 처리하지 않는경우 이를 무시하거나 리스폰더 체인으로 전달하여 다른 객체에서 처리하도록 한다.

터치 이벤트를 직접 처리하는 것 외에도 뷰는 제스처 인식기를 사용하여 탭, 스와이프, 핀치 등 일반적인 터치 관련 제스처를 감지 할 수 있다. 앱이 터치 이벤트를 추적해야하는 대신 제스처 인식기를 만들고 적절한 타켓 객체와 액션 메소드를 할당 한 다음 `addGestureRecognizer:` 메소드를 사용하여 뷰에 추가할 수 있다. 그런 다음 제스처 인식기는 해당 제스처가 발생하면 액션 메서드를 호출합니다.

뷰는 기본적으로 한 번에 한 번의 터치에만 반응한다. 뷰의 이벤트 처리 메소드에서 여러 손가락 동작을 추적하려는 경우 뷰의 `multipleTouchEnabled` 프로퍼티를 설정하여 멀티 터치 이벤트를 활성화해야한다.

## Cleaning Up After Your View

뷰 클래스가 메모리를 할당하거나, 커스텀 객체에 대한 참조를 저장하거나, 뷰가 해제 될 때 함께 해제 되어야하는 리소스를 보유하는 경우 `deinit` 메소드를 구현해야한다. 시스템은 retain count가 0에 도달하고 뷰를 할당 해제(dealloc) 할 때가 되면 `deinit` 메소드를 호출한다. 

## Reference

[Apple Developer](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/CreatingViews/CreatingViews.html#//apple_ref/doc/uid/TP40009503-CH5-SW23)
