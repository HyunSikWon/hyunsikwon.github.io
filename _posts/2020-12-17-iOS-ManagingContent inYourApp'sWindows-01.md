---
title: iOS의 기본적인 UI 구성 방식을 알아봅시다. 
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- UIKit
last_modified_at: 2020-12-17 T21:40:00+08:00
---


## Overview

iOS 앱의 각각의 scene은 하나의 window 객체와 하나 이상의 view 객체를 포함한다. Window는 view를 위한 최상위 container이며, 발생하는 event를 view에게 전달하는 역할을 한다. View는 사용자에게 보여지는 실제 컨텐츠이다. Window는 오직 scene의 전체 UI를 tear down 할 때만 사라지는데 반해, view는 새로운 정보나 컨텐츠를 표시하기위해 주기적으로 바뀐다. 

![b0a380a9-d692-4a37-b349-d7d24137c144](https://user-images.githubusercontent.com/48352065/102489393-bc269900-40b0-11eb-847c-6112322419c6.png)


View를 효율적으로 관리하고 window에 쉽게 제거/추가 하기 위해서, UIKit은 view controller를 제공한다. View controller는 view(set of views)를 관리면서, 이 view들이 최신 정보를 유지하도록 한다. 각각의 window는 초기 view(set of views)를 명시하기 위해 사용하는 root view controller를 갖는다. View(set of views)를 바꾸고 싶을 땐, UIKit에 다른 view controller를 present/dismiss 하도록 요구한다. UIKit은 view 집합 간 전환을 다루고, view controller 객체들을 통해서 앱의 전체 인터페이스를 관리한다. 이와 같이 view controller는 UI를 구현하는데 매우 중요한 역할을 한다.

## Define View Controllers for Each Unique Page of Content

앱의 UI를 디자인 할 때, 컨텐츠에 따라 여러 페이지로 나누고, 각각의 페이지가 서로 다른 방식으로 정보를 표시하도록 설계해라. 예를들어, 한 페이지는 테이블의 행에 데이터를 표시하고, 다른 페이지는 이미지를 그리드 형식으로 표현할 수 있다. 실제로 많은 앱들이 서로 비슷한 유형의 페이지와 형태를 갖고 서로 다른 정보를 표시할 뿐이므로, 페이지에 표시되는 구체적인 데이터는 중요하지 않지만 각 페이지의 구조와 형태는 중요한 사항이다. 

![9cfab753-f093-434b-b7a5-157b19694af8](https://user-images.githubusercontent.com/48352065/102489380-ba5cd580-40b0-11eb-95f3-ec82eb881147.png)


각각의 페이지는 고유의 view controller를 갖으며 이 view controller는 페이지를 표시하고 view를 관리한다. 모든 view controller에는 다음과 같은 작업들을 정의한다.

- 데이터와 함께 view를 정의하고, 데이터의 변화에 따라 view를 갱신하는 작업.
- 데이터 모델 객체에 데이터 변화를 알리는 작업.
- 크기, 위치, 환경에 맞는 view의 형태를 조정하는 작업.
- 다른 페이지의 컨텐츠로의 전환을 위한 작업.

## Choose the Navigation Model for Your Content

간단한 앱은 데이터를 표시하는데 화면 하나면 충분하지만, 대부분의 앱은 여러 화면을 통해 데이터를 표시한다. 이렇게 여러 view controller 사이의 이동을 쉽게 하기 위해, UIKit은 view controller를 위한 컨테이너를 제공한다.

Container view는 다른 view controller(child view controllers) 들을 관리하는 특별한 타입의 view controller 이다.  컨테이너는 한번에 하나의 view controller(child) 만 표시하거나 동시에 두 개의 view controller(child) 를 동시에 표현하며, view controller 간 전환에 따른 효과나 애니메이션 작업을 수행한다.

기본 UIKit 컨테이너는 다음과 같다.

- `UINavigationController`는 navigation interface에서 child view controllers의 스택을 관리한다. view controller를 스택에 push/pop 하면서 화면간 이동을 수행한다.
- `UISplitViewController`는 split-view interface에서 두 개의 child view controller를 분리된 화면에 나란히 표시한다. 공간의 제약이 있는 경우 시스템은 이를 navigation interface에서 표시한다.
- `UITabBarController`는 tab-bar interface에서 하나 이상의 버튼을 담은 행을 표시한다. 이 버튼을 선택하면 버튼과 관련된 child view controller를 표시한다.
- `UIPageViewController`는 paged interface에서 순서가 있는 연속된 child view controller를 관리하면서 한번에 하나 혹은 두개를 표시한다. 사용자는 이 view controller 사이를 오직 swipping 혹은 tapping을 통해서만 이동할 수 있다.

![47837cd1-5bff-4077-81c3-5ac977a904b5](https://user-images.githubusercontent.com/48352065/102489421-c6489780-40b0-11eb-8164-ba200cb00bc7.png)


## Assign a Root View Controller to Each Window

Root view controller는 window의 초기 네비게이션 모델을 정의한다. scene 기반 앱의 경우, root view controller는 scene의 스토리보트 파일의 초기 view controller가 된다. 만약 코드로 작성할 경우, window가 보여지기 전에, window의 `rootViewController` 프로퍼티에 값을 할당하면 된다.

![f1e1901f-5ce0-4d40-8060-1513bc729bf5](https://user-images.githubusercontent.com/48352065/102489415-c5176a80-40b0-11eb-9b4f-23cccd81b98b.png)

만약 root view controller가 컨테이너일 경우, 컨테이너의 child view controller를 반드시 구성해야 한다. 컨테이너의 경우 자신은 최소한의 UI 만 갖을 뿐 나머지 컨텐츠는 child view controller을 통해 표현해야 한다. 예를들어 `UISplitViewController`는 좌,우를 구분하는 구분선 만 표현한다.  

## Reference

[Managing Content in Your App’s Windows](https://developer.apple.com/documentation/uikit/view_controllers/managing_content_in_your_app_s_windows)
