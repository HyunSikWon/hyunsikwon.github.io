---
title: 카메라 앱을 위한 AVCaptureSession!
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- AVCaptureSession
last_modified_at: 2020-12-23 T22:17:00+08:00
---

# Overview

`AVCaptureSession`은 iOS와 macOS 환겨에서 모든 미디어 캡쳐를 위한 클래스이다. 이 객체는 입력 데이터에서 미디어 출력 까지의 과정과 캡쳐 작업을 모두 관리한다. 입출력의 연결을 구성하는 것은 capture session의 기능을 정의한다. 예를 들어, 아래 그림은 iPhone 후면 카메라 및 마이크를 사용하여 사진과 영화를 모두 캡처하고 카메라 preview를 제공하는 capture session을 보여준다.

![b9c65b62-3728-43f1-8d25-08fd42bc6bb7](https://user-images.githubusercontent.com/48352065/102999668-cd6a1c80-456c-11eb-9ce0-6a6c7566b7a8.png)

# Connect Inputs and Outputs to the Session

모든 capture sessoin은 적어도 하나의 캡쳐 입력(capture inputs)과 캡쳐 출력(capture outputs)이 필요하다. 캡쳐 입력은 iOS 기기, Mac의 카메라나 마이크와 같은 미디어 자원을 말한다. 캡쳐 출력은 캡쳐 입력으로 부터 받은 데이터를 사용하여 사진, 영상파일 같은 미디어를 생산한다.

사진이나 영상을 캡쳐하기 위해 카메라를 사용하려면, 적절한 `AVCaptureDevice`를 선택하여 이와 일치하는 `AVCaptureDeviceInput`를 생성하고 이를 session에 추가하면 된다.

```swift
captureSession.beginConfiguration()

let videoDevice = AVCaptureDevice.default(.builtInWideAngleCamera,
                                          for: .video, position: .unspecified)
guard let videoDeviceInput = try? AVCaptureDeviceInput(device: videoDevice!),
    captureSession.canAddInput(videoDeviceInput) else { return }

captureSession.addInput(videoDeviceInput)

```

그 다음엔, 선택한 장비로부터 캡쳐하기 원하는 미디어 종류를 출력에 추가하면 된다. 예를들어, 사진을 캡쳐하기 위해선 `AVCapturePhotoOutput`을 세션에 추가하면 된다:

```swift
let photoOutput = AVCapturePhotoOutput()
guard captureSession.canAddOutput(photoOutput) else { return }
captureSession.sessionPreset = .photo
captureSession.addOutput(photoOutput)
captureSession.commitConfiguration()
```

Session은 여러 입력과 출력을 가질 수 있다. 예를들어:

- 비디오와 오디오를 모두 기록하기 위해서는 입력에 카메라와 마이크 장비를 모두 세션에 추가해야 한다.
- 같은 카메라로 사진과 영상을 모두 캡쳐하기 위해서는 `AVCapturePhotoOutput`과 `AVCaptureMovieFileOutput`을 함께 session에 추가해야 한다.

> session의 입력 혹은 출력의 변경 전에 `beginConfiguration()`를 호출하고, 변경 후에 `commitConfiguration()`를 호출해라.

# Display a Camera Preview

전통적인 카메라의 뷰파인더 처럼, 사진을 찍거나 영상을 녹화하기 전에 사용자가 카메라의 입력을 볼 수 있는 것은 매우 중요하다. 우리는 이같은 preview를 `AVCaptureVideoPreviewLayer`를 capture session에 연결함으로써 제공할 수 있다. 

`AVCaptureVideoPreviewLayer`는 Core Animation layer이므로, 다른 `CALayer` 서브클래스 처럼 인터페이스에 표시하고 스타일을 지정할 수도 있다. Preview layer를 UIKit 앱에 추가하는 가장 간단한 방법은 `layerClass`가 `AVCaptureVideoPreviewLayer`인 `UIView` 서브클래스를 정의하는 것이다.

```swift
class PreviewView: UIView {
    override class var layerClass: AnyClass {
        return AVCaptureVideoPreviewLayer.self
    }
    
    /// Convenience wrapper to get layer as its statically known type.
    var videoPreviewLayer: AVCaptureVideoPreviewLayer {
        return layer as! AVCaptureVideoPreviewLayer
    }
}
```

Capture session과 함께 preview layer을 사용하기 위해서는 layer의 `session` 프로퍼티를 설정해주면 된다.

```swift
self.previewView.videoPreviewLayer.session = self.captureSession
```

# Run the Capture Session

입력, 출력, preview를 모두 구성하고 `startRunning()`를 호출하면 본격적인 작업을 진행할 수 있다. 

몇몇 캡쳐 출력은, session을 실행시키기만 하면 캡쳐가 시작된다. 예를들어 session에 `AVCaptureVideoDataOutput`가 포함된 경우, session이 실행되는 즉시 비디오 프레임을 전달 받는다. 

다른 캡쳐 출력은 먼저 session을 실행한 다음에 캡쳐 출력 클래스 자체를 사용하여 캡쳐를 시작한다. 예를들어, 세션을 실행시키면 뷰파인더처럼 preview를 사용할 수 있지만, 사진을 촬영하기 위해선 `AVCapturePhotoOutput`의 `capturePhoto(with:delegate:)`을 사용해야 한다.

## Reference
[Setting Up a Capture Session](https://developer.apple.com/documentation/avfoundation/cameras_and_media_capture/setting_up_a_capture_session)
