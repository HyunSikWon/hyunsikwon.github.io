---
title: Managing Your App's Life Cycle
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- App Life Cycle
last_modified_at: 2020-12-11 T15:26:00+08:00
---

앱 생명 주기에 대한 공부를 위해 [애플 공식 문서](https://developer.apple.com/documentation/uikit/app_and_environment/managing_your_app_s_life_cycle)를 번역 및 정리한 글입니다.

### **Managing Your App's Life Cycle**

앱의 현재 상태에 따라 할 수 있는 작업과 할 수 없는 작업이 결정된다. 예를들어, 포그라운드에 있는 앱은 사용자와 직접 상호작용하기 때문에 여러 시스템 자원 보다 높은 우선순위를 갖게된다. 반대로 백그라운드에 있는 앱은 가능한 작은 작업을 하거나, 가급적 아무 작업을 하지 말아야한다. 앱의 상태 변화에 따라서 우리는 그에 맞는 동작을 조정을 해줘야 한다.

앱의 상태가 변화할 때, UIKit은 적절한 delegate 객체의 메소드를 호출하여 우리에게 알려준다.

- iOS 13 이후 부터는 scene 기반 앱의 생명 주기 이벤트에 대한 응답을 위해 `UISeceneDelegate`를 사용한다.
- iOS 12 이전 버전에서는 생명 주기 이벤트에 대한 응답을 위해 `UIApplicationDelegate`를 사용한다.

> iOS 13 이후에 scene은 서로 동시에 실행되며 **동일한 메모리와 앱 프로세스 공간을 공유**한다. 이를통해, **단일 앱에 여러 scene과 scene delegate객체가 동시에 활성화** 될 수 있다.

### **Respond to Scene-Based Life-Cycle Events**

만약 앱이 Scene을 지원한다면, **UIKit은 분리된 생명 주기 이벤트를 각각의 scene에 전달**한다. scene이란 디바이스에서 실행되는 앱 UI의 인스턴스 중 하나이다. 사용자는 여러개의 Scene을 생성할 수 있고, 그것들을 따로따로 보이거나 숨길 수 있다. 각 **Scene은 고유의 생명 주기를 갖기 때문에 각각 다른 실행 상태가 될 수 있**다. 예를들어, 하나의 scene은 포그라운드에 있고, 다른 하나는 백그라운드에 있거나 suspended(생명 주기 중 한 상태) 될 수 있다.

이 이미지는 scene의 상태 변화를 보여준다. 사용자나 시스템이 앱의 새로운 scene을 요청하면, UIKit은 scene을 생성하고 그것을 unattached 상태에 넣는다. 사용자에 의해 요청된 scene은 빠르게 포그라운드(스크린 안)로 이동하고, 시스템에 의해 요청된 scene은 보통 백그라운드로 이동하여 이벤트를 처리할 수 있다. 예를들어, 시스템은 위치 이벤트의 수행을 위해서 scene을 백그라운드에서 실행시킬 것이다. 만약 사용자가 앱을 종료한다면(완전한 종료가 아닌 앱 전환 혹은 홈 화면으로 이동) UIKit 은 연관된 scene을 백그라운드로 이동시키고 결국 suspended 상태가 된다. UIKit은 백그라운드 혹은 suspended scene을 그것의 자원을 회수하기 위해서 언제든 연결 해제시켜 scene을 unattached 상태로 되돌릴 수 있다.

![state transitions for scenes](https://user-images.githubusercontent.com/48352065/95951382-48a6a800-0e31-11eb-8c3c-d226e6ba3053.png)

scene 전환을 다음과 같은 작업을 수행하기 위해 사용하라:

- UIKit이 scene을 앱과 연결할 때, scene이 초기 UI를 구성하고 scene이 필요로하는 데이터를 불러와라.
- foreground-active 상태로 전환되었을 때, UI를 구성하고 사용자와의 상호작용을 준비하라.
- foreground-active 상태에서 벗어나면, 데이터를 저장하고 앱의 행동을 멈춰라.
- 백그라운드 상태로 들어가면, 중요한 작업을 끝내고, 가능한 많은 메모리를 반환해라. 그리고 app snapshot을 준비하라.
- scene의 연결이 해제되면 scene과 관련된 공유 자원들을 모두 정리해라.

> snapshot은 앱의 인터페이스이고 사용자가 인지할 수 있어야한다. 앱이 포그라운드로 다시 돌아가면 이 snapshot은 데이터와 뷰를 적절하게 복구되도록 도와준다.


### **Respond to App-Based Life-Cycle Event**

iOS 12 이전에는 scene을 지원하지 않았다. UIKit은 모든 생명주기 이벤트를 `UIApplicationDelegate` 객체에 전달했다.

app delegate 객체는 앱의 모든 window를 관리한다. 그 결과 앱의 상태 변화는 앱의 전체 UI에 영항을 주었다.

이 그림은 app delegate 객체와 관련된 상태 변화를 보여준다. 앱이 런치된 후에 시스템은 앱을 UI가 onscreen에 표시되어야 하는지 아닌지에 따라 inactive 혹은 백그라운드 상태에 넣는다. 앱이 포그라운드로 이동된 후 시스템은 자동적으로 앱을 active 상태로 변화시킨다. 그 후 앱이 종료될 때까지 active 상태와 백그라운드 상태를 오고간다.

![app delegate transition](https://user-images.githubusercontent.com/48352065/95951376-46444e00-0e31-11eb-9de7-6249bff3a6c7.png)

앱의 상태 변화 과정에서 다음과 같은 작업을 수행할 수 있다.:

- 앱이 실행되면 앱의 데이터 구조와 UI를 초기화 해라.
- Activation 상태에선 UI 구성을 마무리하고 사용자와의 상호작용을 준비하라.
- Deactivation 상태에선 데이터를 저장하고 앱의 동작을 종료하라.
- 백그라운드 상태에 들어서선, 중요한 작업을 종료하고, 가능한 많은 메모리를 반환하고 앱 snapshot을 준비하라.
- 종료되면 모든 작업을 즉시 종료하고 공유 자원을 반환하라.
