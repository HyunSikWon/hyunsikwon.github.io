---
title: What is app thinning?
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- App Thinning
last_modified_at: 2021-02-19 T14:19:00+08:00
---

## What is app thinning? (iOS, tvOS, watchOS)

앱 스토어와 운영체제는 사용자의 기기 환경(운영체제 버전, 기기 종류)에 맞게 iOS, tvOS, watchOS 앱의 설치를 최적화 하며, 이를 앱 시닝(app thinning)이라고 부른다. 앱 시닝을 통해서 기기의 대부분의 특성들을 사용하고 최소한의 디스크 공간을 차지하며 Apple이 추후에 적용할 업데이트를 수용할 수 있는 앱을 생성할 수 있다. 

### Slicing (iOS, tvOS)

슬라이싱(slicing)은 서로 다른 타켓 디바이스와 운영체제 버전에 따라 앱 번들의 변형들(variants)을 생성하고 전달하는 프로세스이다. 변형(variant)에는 타겟 디바이스 및 운영 체제 버전에 필요한 실행 가능한 아키텍처와 리소스만 포함된다. 앱의 풀 버전을 계속 개발하여 App Store Connect에 업로드하면 App Store는 앱이 지원하는 기기 및 운영 체제 버전에 따라 다양한 변형을 생성하여 제공한다. App Store에서 각 변형에 적합한 이미지, GPU 리소스 및 기타 데이터를 선택할 수 있도록 에셋 카탈로그(asset catalogs)를 사용한다. 사용자가 앱을 설치하면 사용자 기기 및 운영체제 버전의 변형이 다운로드되고 설치된다.

Xcode는 개발 중에 슬라이싱을 시뮬레이션하므로 로컬에서 변형을 만들고 테스트 할 수 있다. Xcode는 디바이스 또는 시뮬레이터에서 앱을 빌드하고 실행할 때 앱을 슬라이스 한다. 아카이브를 만들 때 Xcode는 앱의 풀 버전을 포함하지만 아카이브에서 변형을 내보낼 수 있다.

![app_thinning_2x](https://user-images.githubusercontent.com/48352065/108461183-8f178200-72bd-11eb-86e5-d2b196fe7ae0.png)

### 

### Bitcode

비트코드는 컴파일된 프로그램의 중간 표현(Intermediate representation)이다. App Store Connect에 업로드된 앱은 비트코드를 포함하고 이 앱은 컴파일되어 앱 스토어에 연결된다(link). 비트코드를 포함하면 Apple은 앱의 새 버전을 App Store에 제출할 필요 없이 향후 앱 바이너리를 다시 최적화 할 수 있다. iOS 앱의 경우 비트 코드가 기본값이지만 선택 사항이고, watchOS 및 tvOS 앱의 경우 비트 코드가 반드시 필요하다. 비트 코드를 제공하는 경우 App Bundle의 모든 앱 및 프레임 워크 (프로젝트의 모든 대상)에 비트 코드가 포함되어야 한다.

### On-Demant Resources (iOS, tvOS)

주문형 리소스는 이미지 및 사운드와 같은 리소스이며 키워드로 태그를 지정하고 태그별로 그룹을 요청할 수 있다. App Store는 Apple 서버의 리소스를 호스팅하고 다운로드를 관리한다. 또한 App Store는 주문형 리소스를 슬라이스하여 앱의 변형을 더욱 최적화합니다.

주문형 리소스는 더 나은 사용자 경험을 제공한다.

- 앱 크기가 작아 앱 다운로드 속도가 빨라져 최초 실행 환경이 개선된다.
- 사용자가 앱을 탐색하는 동안 필요에 따라 백그라운드에서 주문형 리소스를 다운로드한다.
- 운영 체제는 더 이상 필요하지 않고 디스크 공간이 부족할 때 주문형 리소스를 제거합니다.

예를 들어 앱은 사용자가 해당 레벨로 이동할 것이라고 예상하는 경우에만 리소스를 레벨로 나누고 다음 레벨의 리소스를 요청할 수 있다. 또한 인앱 구매를 예로 들 수 있다. 사용자가 해당 인앱 구매를 결정할 때만 리소스를 요청하는 것이다.

![on_demand_resources_2x](https://user-images.githubusercontent.com/48352065/108461177-8d4dbe80-72bd-11eb-9e5c-d7d023ab7f31.png)

## Reference

[What is app thinning?](https://help.apple.com/xcode/mac/current/#/devbbdc5ce4f)
