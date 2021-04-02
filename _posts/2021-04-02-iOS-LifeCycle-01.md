---
title: iOS Life Cycle
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- Life Cycle
last_modified_at: 2021-04-02 T16:25:00+08:00
---

iOS 앱에서 나타나는 여러 생명주기를 다뤄봤습니다. 

## About the App Lauch Seqeunce

앱이 처음 실행되어 초기화되기 까지의 순서는 다음과 같다.

1. 먼저 앱이 사용자 혹은 시스템에 의해 실행된다(launch).
2. `main` 함수가 UIKit의 `UIApplicationMain(_:_:_:_:)` 함수를 호출한다.
3. `UIApplicationMain(_:_:_:_:)`을 통해 `UIApplication`객체와 app delegate가 생성된다.
    - `UIApplication`은 싱글톤 객체로 `shared`를 통해 접근 가능하다.
    - `UIApplication` 객체는 이벤트 큐의 이벤트를 전달하는(routing) 역할을 한다. 컨트롤 객체 (`UIControl` 클래스의 인스턴스)에 의해 전달된 액션 메시지를 적절한 타켓 객체에 전달한다.
    - `UIApplication` 객체가 app delegate를 정의하 몇 가지 런타임 이벤트를 app delegate에 알린다.
4. UIKit이 앱의 디폴트 인터페이스를 main 스토리보드 혹은 nib 파일에서 불러온다.
5. UIKit이 app delegate의 `application(_:willFinishLaunchingWithOptions:)` 메소드를 호출한다.
6. UIKit은 추가적인 app delegate와 뷰 컨트롤러의 메소드를 호출하여 상태 복구를 수행한다.
7. UIKit이 app delegate의  `application(_:didFinishLaunchingWithOptions:)`를 호출한다.

![76e68c08-6b09-4bac-8a00-44df7a097a43](https://user-images.githubusercontent.com/48352065/113392530-4381e900-93d0-11eb-9d79-f1432806c6cb.png)

![66747981-eedb6d00-eec0-11e9-91ab-438ab601c719](https://user-images.githubusercontent.com/48352065/113392627-614f4e00-93d0-11eb-9601-074080d1663d.png)

## App Life Cycle

[지난 글](https://hyunsikwon.github.io/ios/iOS-AppLifeCycle-01/)

[지난 글2](https://hyunsikwon.github.io/ios/iOS-AppLifeCycle-02/)

## View Life Cycle

`UIViewController` 클래스/서브클래스의 객체는 뷰 계층을 관리하는 메소드 집합을 가진다. iOS는 이 메소드들을 뷰 컨트롤러의 상태가 변할때 자동으로 호출한다. 뷰 컨트롤러 서브클래스를 생성하면, `UIViewController`에 정의된 라이프사이클 메소드를 상속받아 원하는 동작을 각 메소드에 구현할 수 있다. 

<img width="462" alt="WWVC_vclife_2x" src="https://user-images.githubusercontent.com/48352065/113392539-454bac80-93d0-11eb-9a5c-12818dd661dc.png">

iOS는 다음과 같은 `UIViewController` 메소드를 호출한다:

- `viewDidLoad()`: 뷰 컨트롤러의 컨텐트 뷰(뷰 계층의 최상위 뷰)가 생성되고 스토리보드에서 로드되면 이 메소드가 호출된다. 이 메소드가 호출된 시점에 뷰 컨트롤러의 아웃렛들은 유효한 값을 갖는다. 이 메소드를 뷰 컨트롤러에서 요구하는 추가적인 세팅을 위해 사용한다.

    보통 iOS는 `viewDidLoad()`를 컨텐트 뷰가 처음 생성될 때 한번만 호출한다. 그러나, 컨텐트 뷰가 반드시 처음 컨트롤러가 초기화될때 생성되어야 하는 것은 아니다. 컨텐트 뷰는 시스템이나 다른 코드에서 컨트롤러의  `view` 프로퍼티에 접근할 때 생성될 수 있다(lazily).

- `viewWillAppear()`:  뷰 컨트롤러의 컨텐트 뷰가 앱의 뷰 계층에 추가되기 직전에 호출된다. 이 메소드는 컨텐트 뷰가 스크린에 표시되기 전에 필요한 작업을 위해 사용한다. 그러나, 시스템이 이 메소드를 호출한다고 해서 컨텐트 뷰가 반드시 화면에 보여질 것이라는(visible) 것은 아니다. 뷰는 다른 뷰에의해 가려지거나 숨겨질 수 있다. 이 메소드는 단지 컨텐트 뷰가 앱의 뷰 계층에 추가될 것임을 알려줄 뿐이다.
- `viewDidAppear()`: 뷰 컨트롤러의 컨텐트 뷰가 앱의 뷰 계층에 추가된 후에 호출된다. 이 메소드는 뷰가 스크린에 표시된 직후에 필요한 작업(fetching data or showing an animation)을 위해 사용한다. 그러나, 시스템이 이 메소드를 호출한다고 해서 컨텐트 뷰가 반드시 화면에 보여지는(visible) 것은 아니다. 뷰는 다른 뷰에의해 가려지거나 숨겨질 수 있다. 이 메소드는 단지 컨텐트 뷰가 앱의 뷰 계층에 추가되었음을 알려줄 뿐이다.
- `viewWillDisappear()`: 뷰 컨트롤러의 컨텐트 뷰가 앱의 뷰 계층에서 제거되기 직전에 호출된다. 이 메소드는 변경사항을 적용하는 것 처럼 작업을 정리하는데 사용할 수 있다. 컨텐트 뷰가 숨겨지기 전이나 투명해지기 전에 이 메소드가 호출되는 것은 아니다. 이 메소드는 오직 컨텐트 뷰가 앱의 뷰 계층에서 제거되기 직전에만 호출된다.
- `viewDidDisappear()`: 뷰 컨트롤러의 컨텐트 뷰가 앱의 뷰 계층에서 제거된 직후에 호출된다. 이 메소드는 추가적인 마무리(teardown) 작업을 위해 사용된다. 컨텐트 뷰가 숨겨졌거나 투명해졌다고 해서 이 메소드가 호출되는 것은 아니다. 이 메소드는 오직 컨텐트 뷰가 앱의 뷰 계층에서 제거되기 직후에만 호출된다.

## View Drawing Cycle

`UIView` 클래스는 컨텐츠를 표시하기 위해서 on-demand 드로잉 모델을 사용한다. 뷰가 처음 화면에 나타나면, 시스템은 뷰에게 컨텐츠를 그리도록 요청하고 컨텐츠의 스냅샷을 갭쳐한다. 이 스냅샷은 뷰의 시각적 표현을 위해 사용되고 뷰와 관련된 대부분의 작업에서 재사용된다. 뷰의 컨텐츠가 변경되면 시스템에게 뷰가 변경되었음을 알리고, 뷰의 드로잉 프로세스를 반복한다. 만약 뷰의 컨텐츠가 변경되지 않는다면, 뷰의 드로잉코드는 다시 호출될 일이 없다. 

뷰의 컨텐츠를 변경하고 싶다고해서 변경사항을 직접 그리면 안된다. 대신, `setNeedsDisplay` 혹은`setNeedsDisplay(rect:)` 메소드를 호출해야 한다. 이 메소드는 시스템에게 뷰가 변경되어 추후에 다시 그려져야한다는 것을 알린다. 그럼 시스템은 현재 런루프가 종료되기 까지 기다린 후, 드로잉 작업을 시작한다. 런 루프가 종료되기를 기다리는 시점에서 뷰를 추가/제거하거나, 뷰를 숨기거나, 뷰의 크기를 재조정 등의 작업을 수행할 수 있다. 이 변경 사항들은 동시에 반영된다. 

**NOTE:** `setNeedsDisplay` 혹은`setNeedsDisplay(rect:)` 메소드는 뷰의 외형과 컨텐츠의 변경이 있는 경우에만 사용해야 한다. 단순한 geometry 변경에는 사용하면 안된다. 이 경우에는 뷰의 `contentMode` 프로퍼티 값을 기반으로 컨텐츠를 재조정한다. 변경사항을 다시 그리는(redraw) 것을 피하는 것으로 성능을 늘릴 수 있다. 

뷰의 업데이트를 자동으로 유발하는(trigger) 동작들도 존재한다:

- 뷰를 부분적으로 가리는 다른 뷰를 이동하거나 제거하는 경우.
- 이전에 숨겨진 뷰를 `hidden` 프로퍼티를 통해 표시하는 경우.
- 스크롤하여 뷰를 벗어났다 돌아온 경우.
- 뷰에서 명시적으로 `setNeedsDisplay` 혹은 `setNeedsDisplayInRect:` 메소드를 호출한 경우.

런루프가 종료된 후 진행되는 뷰의 실제 드로잉 작업은 뷰와 뷰의 구성에 따라 다르다. 시스템 뷰는 보통 프라이빗 드로잉 메소드를 사용하여 자동으로 컨텐츠를 렌더링하고, 커스텀 뷰(`UIView` 서브클래스)는 보통 `drawRect:` 메소드를 오버라이드해서 뷰의 컨텐츠를 그리는데 사용한다.

## Reference

[https://developer.apple.com/documentation/uikit/uiapplication](https://developer.apple.com/documentation/uikit/uiapplication)

[https://developer.apple.com/library/archive/documentation/2DDrawing/Conceptual/DrawingPrintingiOS/GraphicsDrawingOverview/GraphicsDrawingOverview.html](https://developer.apple.com/library/archive/documentation/2DDrawing/Conceptual/DrawingPrintingiOS/GraphicsDrawingOverview/GraphicsDrawingOverview.html)

[https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/WindowsandViews/WindowsandViews.html#//apple_ref/doc/uid/TP40009503-CH2-SW9](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/WindowsandViews/WindowsandViews.html#//apple_ref/doc/uid/TP40009503-CH2-SW9)

[https://developer.apple.com/library/archive/referencelibrary/GettingStarted/DevelopiOSAppsSwift/WorkWithViewControllers.html](https://developer.apple.com/library/archive/referencelibrary/GettingStarted/DevelopiOSAppsSwift/WorkWithViewControllers.html)

[https://gaki2745.github.io/ios/2019/10/14/iOS-Basic-13/](https://gaki2745.github.io/ios/2019/10/14/iOS-Basic-13/)
