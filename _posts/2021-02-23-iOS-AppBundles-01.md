---
title: iOS Application Bundle
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- Application Bundle
last_modified_at: 2021-02-23 T14:41:00+08:00
---

## Application Bundles

앱 번들(Application bundle)은 개발자가 생성하는 가장 흔한 타입의 번들 중 하나이다. 앱 번들은 앱의 운영(operation)을 위한 모든 것들을 저장한다. 앱 번들은 개발하는 플랫폼에 따라 구체적인 구조가 다르지만, 사용하는 방식은 모두 같다. 오늘은 iOS의 앱 번들 구조를 알아보자.

## What Files Go Into a Application Bundle?

앱 번들 내에서 찾을 수 있는 파일 유형은 다음과 같다. 

- `Info.plist` file: (필수) *Information Property list* 파일은 응용 프로그램에 대한 구성 정보가 들어 있는 구조화된 파일(structured file)이다. 시스템은 이 파일의 존재 여부에 따라 애플리케이션과 관련 파일에 대한 연관 정보를 식별한다.
- Executable: (필수) 모든 응용 프로그램에는 실행 가능한 파일이 있어야 한다. 이 파일에는 애플리케이션의 메인 엔트리 포인트와 애플리케이션 타겟에 정적으로 연결된 모든 코드가 포함되어 있다.
- Resource files: 리소스는 애플리케이션의 실행 가능한 파일 외부에 있는 데이터 파일이다. 리소스는 일반적으로 이미지, 아이콘, 소리, nib 파일, 문자열 파일, 설정 파일 및 데이터 파일 등으로 구성된다. 대부분의 리소스 파일은 특정 언어나 지역에 따라 지역화(localized)되거나 모든 지역에서 공유될 수 있다.
- Other support files: Mac 앱은 private 프레임워크, 플러그인, 문서 템플릿 및 애플리케이션에 필수적인 기타 커스텀 데이터 리소스와 같은 고급 리소스(high-level)를 추가로 포함할 수 있다. iOS 애플리케이션 번들에 커스텀 데이터 리소스를 포함할 수 있지만 커스텀 프레임워크나 플러그인을 포함할 수는 없다.

앱 번들의 대부분의 리소스들은 선택적이지만, 언제나 그런 것은 아니다. 예를 들어 iOS 앱은 보통 앱의 아이콘 등을 위해 추가적인 이미지 리소스들 필요로한다. 

## Anatomy of an iOS Application Bundle

Xcode에서 제공하는 프로젝트 템플릿은 iPhone 또는 iPad 애플리케이션의 번들을 설정하는데 필요한 대부분의 작업을 수행한다. 그러나 번들 구조를 이해하면 커스텀 파일을 저장할 위치를 결정하는데 도움이 된다. iOS 앱의 번들 구조는 모바일 기기의 요구에 더 잘 맞춰져 있고 외부 디렉토리가 거의 없는 비교적 평평한 구조를 사용하여 디스크 공간을 절약하고 파일에 대한 액세스를 간소화 한다.

### The iOS Application Bundle Structure

일반적인 iOS 앱 번들에는 실행 파일과 앱에서 사용하는 모든 리소스(앱 아이콘, 기타 이미지 및 지역화된 콘텐츠)가 최상위 번들 디렉토리에 포함되어 있다. 다음 예제는 MyApp이라는 간단한 iPhone 앱의 구조를 보여준다. 하위 디렉토리에 있어야 하는 파일은 지역화해야 하는 파일뿐이지만 사용자 자신의 앱에 추가 하위 디렉토리를 생성하여 리소스 및 기타 관련 파일을 구성할 수 있다. 

```
MyApp.app
   MyApp
   MyAppIcon.png
   MySearchIcon.png
   Info.plist
   Default.png
   MainWindow.nib
   Settings.bundle
   MySettingsIcon.png
   iTunesArtwork
   en.lproj
      MyImage.png
   fr.lproj
      MyImage.png
```

다음은 MyApp의 내용을 설명한다. 

- MyApp: (필수) 앱의 코드를 포함하는 실행 파일이다. 이 파일의 이름은 앱의 이름에서 .app 확장자를 뺀 것과 같다.
- 앱 아이콘((MyAppIcon.png, MySearchIcon.png, and MySettingsIcon.png): (필수/권장) 앱 아이콘은 앱을 나타내는 특정 시점에 사용된다. 예를 들어, 홈 스크린, 검색 결과 및 설정 앱에서 다양한 크기의 앱 아이콘이 표시된다.
- Info.plist: (필수) 이 파일에는 번들 ID, 버전 넘버 및 디스플레이 이름과 같은 앱에 대한 구성(configuration) 정보가 포함되어 있다.
- Launch images (Default.png): (권장) 앱의 초기 인터페이스를 특정 방향으로 표시하는 하나 이상의 이미지이다. 시스템은 앱이 윈도우와 UI를 로드할 때까지 Launch 이미지 중 하나를 임시 배경으로 사용한다. 앱에서 Launch 이미지를 제공하지 않으면 애플리케이션이 시작되는 동안 검은 배경이 표시된다.
- MainWindow.nib: (권장) 앱의 main nib 파일은 앱의 런치 타입에 로드할 기본 인터페이스 객체를 포함한다. 일반적으로 이 nib 파일에는 앱의 기본 윈도우 객체와 App delegate 객체의 인스턴스가 포함된다. 그런 다음 다른 인터페이스 객체는 추가 nib 파일에서 로드되거나 앱에 의해 프로그래밍 방식으로 생성된다.
- Settings.bundle: Setting 번들은 설정 앱에 추가할 앱의 기본 설정이 포함된 특별한 유형의 플러그인이다. 이 번들에는 기본 설정을 구성하고 표시할 프로퍼티 리스트와 기타 리소스 파일이 포함되어 있다.
- 커스텀 리소스 파일: 지역화되지 않은 리소스는 최상위 디렉토리에 배치되고 지역화된 리소스는 앱 번들의 언어별 하위 디렉토리에 배치된다. 리소스는 nib 파일, 이미지, 사운드 파일, 설정 파일, 문자열 파일 및 앱에 필요한 기타 커스텀 데이터 파일로 구성된다.

> iOS 앱 번들에는 "Resources"라는 이름의 사용자 지정 폴더가 포함될 수 없다.

iOS 앱은 국제화되어야 하며 지원하는 각 언어에 대한 *language.lproj* 폴더가 있어야 한다. 국제화를 위해 앱의 지역화 버전의 커스텀 리소스를 제공하거나 언어별 프로젝트 디렉토리에 동일한 이름의 파일을 배치하여 Launch 이미지를 로컬화할 수도 있다. 그러나 지역화 버전을 제공하는 경우에도 항상 해당 파일의 기본 버전을 애플리케이션 번들의 최상위 수준에 포함해야 한다. 기본 버전은 특정 위치를 지정하여 사용할 수 없는 경우에 사용된다. 

### The Information Property List File

모든 iOS 앱에는 앱의 구성 정보가 포함된 Information Property list(Info.plist) 파일이 있어야 한다. 새 iOS 앱 프로젝트를 만들 때 Xcode는 자동으로 이 파일을 생성하고 일부 키 프로퍼티 값을 설정한다. 다음은 명시적으로 설정해야 하는 몇 가지 추가 키이다.

Info.plist 파일의 필수 키

- `CFBundleDisplayName` (Bundle display name): Bundle display name이란 앱 아이콘 아래에 표시되는 이름을 뜻한다. 지원하는 모든 언어에 지역화되어야 한다.
- `CFBundleIdentifier` (Bundle identifier): Bundle identifier 문자열은 시스템이 앱을 식별하는데 사용된다.
- `CFBundleVersion` (Bundle version): Bundle version 문자열은 번들의 빌드 버전 번호를 지정한다. 이 값은 지속적으로 증가하게 되어있으며 지역화할 수 없다.
- `CFBundleIconFiles`: 앱의 다양한 아이콘에 사용되는 이미지의 파일 이름이 포함된 문자열 배열이다.
- `LSRequiresIPhoneOS` (Application requires iOS environment): 번들이 iOS에서만 실행가능한지를 나타내는 BOOL 값이다.
- `UIRequiredDeviceCapabilities`: iTunes와 App Store가 앱을 실행하기 위해 필요로 하는 디바이스 기능을 알려주는 키이다. iTunes와 Mobile App Store는 이 목록을 사용하여 고객이 해당 기능을 지원하지 않는 기기에 애플리케이션을 설치하지 못하도록 한다.

Info.plist 파일에 흔하게 포함되는 키

- `NSMainNibFile` (Main nib file base name)
- `UIStatusBarStyle`
- `UIStatusBarHidden`
- `UIInterfaceOrientation`
- `UIPrerenderedIcon`
- `UIRequiresPersistentWiFi`
- `UILaunchImageFile`

### Application Icon and Launch Images

앱 아이콘과 런치 이미지는 모든 앱에 있어야 하는 표준 그래픽이다. 모든 앱은 기기의 홈 화면과 앱 스토어에 표시할 아이콘을 지정해야 한다. 또한 여러 가지 상황에서 사용할 수 있는 아이콘들을 지정할 수 있다. 예를 들어 앱은 검색 결과를 표시할 때 사용할 아이콘의 작은 버전을 제공할 수 있으며 시작 이미지는 앱이 시작될때 사용자에게 시각적 피드백을 제공한다.

아이콘 및 런치 이미지를 나타내는 데 사용되는 이미지 파일은 모두 번들의 루트 레벨에 있어야 한다. 시스템에서 이러한 이미지를 식별하는 방법은 여러가지가 있지만 앱 아이콘을 지정하는 권장 방법은 `CFBundleIconFiles` 키를 사용하는 것이다. 

### Resources in an iOS Application

iOS 앱에서 로컬이 아닌 리소스는 애플리케이션 실행 파일, Info.plist 파일과 함께 번들 디렉토리의 탑 레벨에 위치한다. 대부분의 iOS 앱은 루트 레벨에 앱 아이콘, 런치 이미지 및 하나 이상의 nib 파일을 포함하고 있다. 대부분의 비로컬 리소스는 이 최상위 디렉토리에 배치해야 하지만 하위 디렉토리를 생성하여 리소스 파일을 구성할 수도 있다. 지역화된 리소스는 하나 이상의 언어별 하위 디렉토리에 배치되어야 한다.

다음은 지역화된 리소스와 지역화되지 않은 리소스를 포함하는 가상의 앱을 보여준다. 지역화되지 않은 리소스는 Hand.png, MainWindow.nib, MyAppViewController.nib와 WaterSounds 디렉토리 컨텐츠들이고 en.lproj 및 jp.lproj 디렉토리에 있는 항목들은 지역화된 리소스에 해당된다.

```
MyApp.app/
   Info.plist
   MyApp
   Default.png
   Icon.png
   Hand.png
   MainWindow.nib
   MyAppViewController.nib
   WaterSounds/
      Water1.aiff
      Water2.aiff
   en.lproj/
      CustomView.nib
      bird.png
      Bye.txt
      Localizable.strings
   jp.lproj/
      CustomView.nib
      bird.png
      Bye.txt
      Localizable.strings
```

## Reference

[Apple document](https://developer.apple.com/library/archive/documentation/CoreFoundation/Conceptual/CFBundles/BundleTypes/BundleTypes.html#//apple_ref/doc/uid/10000123i-CH101-SW15)
