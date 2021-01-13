---
title: App's Life Cycle - Foreground & Background
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- App Life Cycle
last_modified_at: 2021-01-13 T21:08:00+08:00
---


# Preparing Your UI to Run in the Foreground

## Overeview

포 그라운드 전환을 사용하여 앱의 UI가 화면에 표시되도록 준비한다. 앱이 포 그라운드로 전환되는 것은 일반적으로 사용자 작업에 대한 응답이다. 예를 들어 사용자가 앱 아이콘을 탭하면 시스템이 앱을 실행하고 포 그라운드로 가져온다. 포 그라운드 전환을 사용하여 앱의 UI를 업데이트하고, 리소스를 획득하고, 사용자 요청을 처리하는 데 필요한 서비스를 시작해야 한다.

모든 상태 전환에서 UIKit은 적절한 델리게이트 객체에 알림을 보낸다.

- iOS 13 이후 - `UISceneDelegate` 객체
- iOS 12 이전 - `UIApplicationDelegate` 객체

## Update Your App’s Data Model when Entering the Foreground

시스템은 앱을 포그라운드로 전환하기 전 inactive 상태에서 앱을 시작한다. 앱의 시작 시점의 메소드를 사용하여 해당 시점에 필요한 작업을 수행한다. 백그라운드에있는 앱의 경우, UIKit은 다음 메서드 중 하나를 호출하여 앱을 inactive 상태로 이동시킨다.

- `sceneWillEnterForeground(_:)`
- `applicationWillEnterForeground(_:)`

백그라운드에서 포그라운드로 전환할때, 이 메소드를 사용하여 디스크로부터 자원을 불러오고, 네트워크로부터 데이터를 불러올 수 있다.

## Configure Your User Interface and Initial Tasks at Activation

시스템은 앱의 UI를 표시하기 직전에 앱을 active 상태로 바꾼다. 앱의 UI를 구성하고 런타입 동작을 구성하기 좋은 시점이다. 이와 같은 작업은 다음의 메소드에서 수행한다.

- `sceneDidBecomeActive(_:)`
- `applicationDidBecomeActive(_:)`

이 단계는 UI가 사용자에게 표시되기 전 마지막 시점이다. 따라서 이 메소드를 블럭할 수 있는 어떠한 작업 코드도 실행해선 안된다. 예를 들어 앱의 외부에서 데이터가 자주 바뀐다면, 앱이 포그라운드로 이동하기 전에 백그라운드 작업을 통해 새로운 데이터를 불러오도록(fetch) 해야한다. 그렇지 않으면 비동기 적으로 변경 사항을 가져 오는 동안 기존 데이터를 표시 할 준비를 해야한다.

## Start UI-Specific Tasks when Your View Appears

활성화(ativation) 메소드가 반환되면 UIKit은 사용자가 구성한 모든 window을 표시하고 관련된 뷰 컨트롤러에 해당 뷰가 곧 나타날 것임을 알린다. 뷰 컨트롤러의 `viewWillAppear(_ :)` 메소드를 사용하여 인터페이스에 대한 최종 업데이트를 수행한다. 예를 들면 :

- 애니메이션 작업을 시작할 수 있다.
- 자동재생이 활성화 되어있다면 미디어 파일을 재생할 수 있다.

다른 뷰 컨트롤러를 보여주거나, UI의 주요 사항을 변경하면 안된다. 이 시점에는 뷰 컨트롤러가 화면에 나타나므로, 인터페이스는 반드시 표시될(display) 준비가 되어있어야 한다.

# Preparing Your UI to Run in the Background

## Overview

앱은 많은 이유들로인해 백그라운드 상태로 이동한다. 사용자가 포그라운드 앱을 빠져 나오거나(exit), UIkit이 앱을 중지(suspend)하기 전에 앱은 잠시 백그라운드 상태로 이동한다. 시스템은 앱을 직접 백그라운드 상태에서 시작하거나 중지(suspend) 된 앱을 백그라운드로 이동하여 중요한 작업을 수행 할 시간을 제공 할 수도 있다.

앱은 백그라운드에서 가능한 최소한의 동작을 하거나 아무것도 하지 말아야 한다. 만약 이전에 포그라운드 상태에 있던 상태라면, 백그라운드 전환(transition)을 사용하여 작업을 종료하고 공유 자원을 반환(release)해야 한다.

모든 상태 전환에서 UIKit은 적절한 델리게이트 객체에 알림을 보낸다.

- iOS 13 이후 - `UISceneDelegate` 객체
- iOS 12 이전 - `UIApplicationDelegate` 객체

## Quiet Your App upon Deactivation

사용자가 포그라운드 앱에서 빠져나오면, 시스템은 백그라운드로 이동 직전에 해당 앱을 비활성화(deactivates) 한다. 또한, 시스템은 알람과 같이 앱이 일시적으로 중단될 필요가 있는 상황에서도 앱을 비활성화 한다. 비활성화하는 동안에 시스템은 다음의 메소드 중 하나를 호출한다.

- `sceneWillResignActive(_:)`
- `applicationWillResignActive(_:)`

비활성화를 사용하여 사용자의 데이터를 보존하고 모든 주요 작업을 일시 중지하여 앱을 정적인(quite) 상태로 만든다. 

- 사용자 데이터를 디스크에 저장하고 열린 파일들을 닫는다.
- Dispatch, operation queue를 중단한다.
- 실행을 위한 새로운 작업들을 예약하지(schedule) 않는다.

## Release Resources upon Entering the Background

앱이 백그라운드로 가면, 앱이 소유한 공유 자원과 메모리를 반환한다. 포그라운드에서 백그라운드로의 전환 작업에서 메모리를 반환하는(freeing up) 것은 특히 중요하다. 포그라운드는 메모리와 다른 시스템 자원에 대한 높은 우선 순위를 가지기 때문에 이러한 자원을 사용할 수 있도록 필요에 따라 백그라운드 앱을 종료하기 때문이다. 앱이 포그라운드에 있지 않더라도 가능한 한 적은 자원을 사용하는지 확인해야 한다.

백그라운드로 들어가면 UIKit은 앱의 다음 메서드 중 하나를 호출한다.

- `sceneDidEnterBackground(_:)`
- `applicationDidEnterBackground(_:)`

백그라운드로 전환하는 동안 앱에 맞게 다음 작업을 수행해야한다.

- 파일에서 직접 읽은 이미지 혹은 미디어를 제거한다(discard).
- 디스크로부터 다시 생성할 수 있거나 다시 불러올 수 있는 대용량 메모리 객체를 제거해야 한다.
- 카메라 및 기타 공유 하드웨어 자원에 대한 접근을 해제해야 한다.
- 비밀번호 같은 민감 정보를 UI에서 숨겨야 한다.

앱의 에셋 카탈로그에서 로드한 이미지를 제거할 필요는 없다. 비슷하게 `NSDiscardableContent` 프로토콜을 채택한 객체나 `NSCache` 객체를 사용하여 관리하는 객체를 제거할 필요도 없다. 이러한 객체는 시스템이 자동으로 관리한다.

백그라운드로 전환할 떄 앱이 어떠한 공유 시스템 자원을 보유(holding)하지 않게 해야한다. 만약 백그라운드 전환 후에도 카메라 같은 자원에 계속 접근한다면 시스템은 자원을 위해(free up) 앱을 종료시킬 것이다.

## Prepare Your UI for the App Snapshot

앱이 백그라운드 상태에 도달하고, 델리게이트 메소드가 반환되면, UIKit은 앱의 현재 UI의 스냅샷을 생성한다. 시스템은 앱의 스위처에 결과 이미지를 표시한다. 또한 앱을 포그라운드로 되돌릴 때 일시적으로 이 이미지를 표시한다.

앱의 UI는 비밀번호, 카드 번호 같은 민감한 사용자 정보를 포함되어서는 안된다. 만약 인터페이스가 민감 정보를 포함한 경우, 백그라운드로 들어갈 때 뷰에서 이 정보들을 지워야 한다. 또한 앱의 컨텐츠를 가리는(obscure) 알람, 시스템 뷰 컨트롤러 등을 없애야 한다. 스냅샷은 앱의 인터페이스를 표현하고 사용자가 반드시 인지할 수 있는 상태여야 한다.

## Respond to Important Events in the Background

앱은 일반적으로 백그라운드에 들어간 후에는 추가 실행 시간을 받지 않는다. 그러나 UIKit은 당므과 같은 시간에 민감한 기능을 지원하는 앱에는 실행 시간을 부여한다.

- 위치에 민감한 서비스
- APN 서비스를 위한 지원하는 기능
- 외부 액세서리와 커뮤니케이션 하는 기능
- 블루투스 LE 액세서리와 커뮤니케이션 하거나, 장비를 블루투스 LE 액세서리로 전환하는 기능

## Reference

[Preparing Your UI to Run in the Foreground](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_foreground)

[Preparing Your UI to Run in the Background](https://developer.apple.com/documentation/uikit/app_and_environment/scenes/preparing_your_ui_to_run_in_the_background)
