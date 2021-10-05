---
title: iOS - VideoToolBox
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- VideoToolBox
last_modified_at: 2021-10-05 T13:53:00+08:00
---

WWDC14 - Direct Access to Video Encoding and Decoding 를 보고 디코딩 관련 개념을 정리한 글입니다.


## Media Framework

**AVKit.**

- 미디어를 다루기 위한 high-level, view-level 인터페이스.

**AVFoundation.**

- 미디어 작업을 위한 Objective-C 인터페이스.
- Decompress direct to display
- Compress directly to file

**VideoToolbox.**

- 하드웨어 인코더/디코더에 직접 액세스 할 수 있는 프레임워크.
- Decompress directly to `CVPixelBuffers`
- Compress directly to `CMSampleBuffers.`

**Core Media and Core Video.**

- 미디어 관련한 여러 타입들을 제공해 주는 프레임워크.

**`CVPixelBuffer`:** 압축되지 않은 래스터 이미지 버퍼를 포함하며, 데이터 접근 방법을 알려준다. 치수(dimension), 높이, 너비와 pixel data를 정확하게 해석하기 위한 pixel format을 가진다.

**`CVPixelBufferPool`:**  `CVPixelBuffer`가 계속해서 할당/할당 해제되는 작업은 매우 expensive하기 때문에  `CVPixelBuffer` 풀을 관리하며 재사용할 수 있도록 해준다. `PixelBufferPool`의 동작 방식은 `CVPixelBuffer` 의 reference count가 0이 되면 가장 뒷단으로 보내서 추후에 재사용하는 방식이다.

**`CFDictionary`:**  `CVPixelBuffer`, `PixelBufferPool` 를 위한 데이터를 포함한다. (dimensions, width, height, pixel format)

**`CMTime`:**
디테일한 시간의 작업이 필요한 경우에 사용된다.

`CMTimeValue` : 64 비트 정수로 분수로 표현되는 시간의 분자 값을 정의

`CMTimeScale` : 32비트 정수로 분모 값을 정의

```
// 0.25 seconds
let quarterSecond = CMTime(value: 1, timescale: 4)

// 10 second mark in a 44.1 kHz audio file
let tenSeconds = CMTime(value: 441000, timescale: 44100)

// 3 seconds into a 30fps video
let cursor = CMTime(value: 90, timescale: 30)

```

**`CMVideoFormatDescription`:**

- 비디오 데이터에 대한 설명(description)
- 픽셀 포맷, 여러 익스텐션
- Extensions(Pixel Aspect Ratio, Color Space Information)
- H.264 data의 경우 the parameter sets(SPS, PPS) 가 포함되어 있다.

**`CMBlockBuffer`:**

- 코어 미디어에서 임의의 데이터를 래핑하는 용도
- 파이프라인의 압축된 비디오 데이터를 래핑.

**`CMSampleBuffer`**

- 데이터 샘플을 래핑
- 시간을 표현하는 `CMTime`
- `CMSampleBuffer`의 내부 데이터를 설명하는 `CMVideoFormatDescription`
- 압축된 비디오 데이터인 `CMBlockBuffer` 로 구성되어 있다.
- `CMSampelBuffer`의 내부 데이터가 압축 해제된 데이터라면 이는 `CMBlockBuffer`가 아니라 `CVPixelBuffer` 형태이다.

**`CMClock`**

- `CMClock` 시간 정보를 래핑한 Core Media wrapper.
- `CMClock`의 경우 시간이 언제나 증가한다는 특징이 있다.
- 일정한 속도로 계속해서 증가하기 때문에 컨트롤하기 어렵다.

`CMTimebase`

- `CMClock`을 기준으로 타임 매핑과 rate 제어를 할 수 있다.
- 더 통제된 뷰를 제공

## Case One

**네트워크를 통해 들어오는 데이터 스트림을  `AVSampleBufferDisplayLayer`를 사용해서 앱에서 보여주는 케이스.**

**AVSampleBufferDisplayLayer**

- 압축된 프레임 시퀀스를 input으로 받는다.
- Input은 `CMSampleBuffer` 들로 이루어져야 한다.
- 내부적으로는 디코더가 존재해서 각 프레임을 `CVPixelBuffer`로 디코딩해서 재생.

그러나 네트워크에서 내려받는 압축된 비디오 스트림은 Elementary stream이다. 따라서, 이를 `CMSampleBuffer`로 변환하는 과정이 필요하다.

### H.264

H.264에는 Elementary Stream, MPEG-4 패키징 방식이 있다. `CMSampleBuffers`를 다루는 인터페이스, CoreMedia, AVFoundation 프레임워크는 기본적으로 MPEG-4에 대응할 수 있으나 Elemantary에는 대응할 수 없다. 따라서 Elemantary stream을 MPEG-4로 변경하는 작업이 필요하다.

먼저 Elementary Stream에서 parameter set(SPS,PPS)와 NAL Unit을 구하고 이를 `CMVideoForMatDescription`으로 패키징하는 작업이 필요하다. 이 과정에선 `CMVideoFormatDescriptionCreatefromH264ParameterSets`를 사용한다.

이후엔 Elementary Stream의 NALU 헤더를 MPEG-4 패키징에 맞게 바꿔줘야 한다.(Start Code -> Length Code)

### Elementary Stream -> `CMSampleBuffer`

1. Elementary Stream 헤더의 start code를 length code로 바꾸고. 그 다음 해당하는 모든 NALU를 `CMBlockBuffer`로 래핑한다.
2. Parameter sets을 가진 `CMVideoFormatDescription`과 합친다.
3. Presentation Time을 지정한 `CMTime`을 합친다

### `AVSampleBufferDisplayLayer` and Time.

- `CMSampleBuffers`에는 관련된 타임스탬프가 있고 내부의 비디오 디코더가 연관된 타임스탬프와 함께 `CVPixelBuffers`를 만들어낸다.
- 프레임을 표시할 때 타임 스탬프는 hostTime clock(system의 기본 clock)을 기반으로 하는데, 이는 관리가 매우 어렵다. 따라서 hostTime clock을 자신만의 timebase로 대체하는 작업이 필요하다.
- hostTime clock을 기반으로 timebase를 생성하고 이를 `AVSampleBufferDisplayLayer`의 `controlTimebase`로 세팅한다.
- timebase 시간을 frame에 맞게 설정하면. 타임스탬프가 해당 시간으로 설정된 프레임이 레이어에 표시되고, 타임베이스 속도를 1로 설정하면 타임베이스가 속도에 맞게 움직이기 시작한다. hostTime clock과 동일한 속도로, 후속 프레임은 적절한 시간에 표시된다.

### Feeding `AVSampleBufferDisplayLayer`

- `CMSampleBuffer` 를 `AVSampleBufferDisplayLayer` 에 제공하는 것은 주로 2가지 케이스로 나뉜다.
- Periodic source.
- Uncontrainedsource.

### Peridodic source

첫번째 케이스는 `AVSampleBufferDisplayLayer`에 표시될 프레임을 동일한 속도로 가져오는 경우이다. 이는 라이브 스트리밍 앱 또는 화상 회의를 예로 들 수 있다.

이 케이스의 경우 매우 단순하다. 앱에 표시할 프레임은 같은 속도로 올 것이고 이를 `CMSampleBuffer`에 담아 `AVSampleBufferDisplayLayer`에 넣어주기만 하면 된다. `enqueueSampleBuffer`를 사용한다.

### Unconstrainedsource

두번째 케이스는 한 번에 `AVSampleBufferDisplayLayer`에 제공할 준비가 된 `CMSampleBuffer` 세트가 있는 경우이다. 파일에서 `CMSampleBuffers`를 읽는 경우가 이에 해당한다.

두번째 케이스의 경우는 조금 복잡하다.

한 번에 모든 `CMSampleBuffers`를 `AVSampleBufferDisplayLayer`에 밀어넣어서는 안된다. `AVSampleBufferDisplayLayerd`의 내부 버퍼가 더 많은 데이터가 필요하고 불러올 데이터가 충분할 때 데이터를 요청해야 한다.

이를 수행하는 방법은 `requestMediaDataWhenReadyOnQueue`를 사용하는 것이다. 이는 블록을 담고있으며 `AVSampleBufferDisplayLayer`는 내부 큐가 더 많은 데이터가 필요할 때마다 이 블록을 호출한다. 블록 내부에서는 데이터가 충분한지를 반복해서 물을 수도 있다. 이를 위해 `isReadyForMoreMediaData` 를 사용한다. `true`를 반환하면 더 많은 `CMSampleBuffer`가 필요하므로 계속해서 `CMSampleBuffer`를 제공해야 한다. `false`를 반환하면 이는 데이터가 충분하고 요청을 중지할 수 있음을 의미한다.

### Summary

- `AVSampleBufferDisplayLayer`을 생성한다.
- H.264 elementary stream을 `CMSampleBuffer`로 변환한다(convert).
- H.264 데이터를 담고있는 `CMSampleBuffer`는 `AVSampleBufferDisplayLayer`에 의해 압축해제 된다. -> `CVPixelBuffer`
- `AVSampleBufferDisplayLayer`와 함께 커스텀 `CMTimeBase`를 사용하여 시간을 표현한다.

## Case Two

**네트워크를 통해 들어오는 H.264 데이터 스트림을 단순히 애플리케이션에서 보여주는 것이 아니라, 실제로 해당 프레임을 디코딩하고 압축 해제된 픽셀 버퍼를 얻으려고 하는 케이스.  `AVSampleBufferDisplayLayer`를 통해 비디오 디코더에 액세스하는 대신 `VTDecompressionSession`을 통해 액세스 한다.**

`AVSampleBufferDisplayLayer`와 마찬가지로, `VTDecompressionSession` 역시 입력으로  `CMSampleBuffers` 를 받고 이를 디코딩해서 `CVPixelBuffers` 형태로 바꾼다. 디코딩된 값은 `OutputCallback` 을 구현해서 받을 수 있다.

### `VTDecompressionSession`

`VTDecompressionSession`을 생성하기 위해선 다음의 작업들이 필요하다.

1. 먼저 압축 해제할 소스 버퍼에 대한 설명이 필요하다. 이는 `CMVideoFormatDescription` 타입이다. Elementary Stream을 압축 해제할 경우 parameter sets을 사용해서 이를 생성하다. `CMSampleBuffer`가 이미 존재하면 `CMSampleBuffer`의 `CMVideoFormatDescription`을 사용하면 된다.
2. 출력 데이터인 pixel buffer에 대한 요구사항을 기술해야한다. 이를 위해서 `pixelBufferAttributes` 딕셔너리를 사용한다.
3. 마지막으로 `VTDecompressionOutputCallback`을 구현해야 한다.

### `CVPixelBuffer`

출력 데이터인 `CVPixelBuffer`를 위한 요구사항에 대해 알아보자. 먼저 `PixelBufferAttributes` 딕셔너리를 생성해야한다.

OpenGL ES 파이프라인에서 `CVPixelBuffer`를 사용하길 원한다고 가정해보자.
필요한 요구사항은 출력 데이터인 `CVPixelBuffer`가 OpenGL ES와 호환이 되어야한다는 것이다. 이를 위해선 `CFDictionary`를 생성하고`kCVPixelBufferOpenGLESCompatibilityKey`키와 `true` 값을 딕셔너리에 추가해주면 된다. 그러나 딕셔너리가 특정 값에만 대응되도록 너무 구체화 해서는 안된다. 이는 `VTDecompressionSession`에서 요구사항을 만족시키기 위한 오버헤드가 발생할 수 있기 때문이다.

### Output Callback

Output Callback에서는 time stamp가 내장되지 않은 디코딩된 `CVPixerBuffer`를 받고 time stamp를 표현하기 위해 외부에서 time stamp를 받기도 한다. 또한 에러가 발생하거나 어떤 이유로 인해서 프레임이 누락될 경우 관련 정보 또한 받을 수 있다. Output Callback은 에러나 누락에 상관 없이 프레임을 `VTDecompressionSession`에 제공할 때마다 호출된다.

### Feeding `VTDecompressionSession`

`CMSampleBuffer`(frame)을 `VTDecompressionSession`에 제공하기 위해선
`VTDecompressionSessionDecodeFrame`을 호출한다. `CMSampleBuffer` 파라미터와 함께 호출하며 디코딩할 순서대로 보내야 한다.

기본적으로 이 `VTDecompressionSessionDecodeFrame`는 동기적으로 작업을 수행하며 이는 Output Callback이 함수가 리턴된 후에 호출된다는 뜻이다. 비동기적인 작업을 위해선 flag 파라미터를 비동기로 지정해주면 된다.

### Async Decompressoin

비동기적인 `VTDecompressionSessionDecodeFrame`은 프레임을 디코더에 넘겨준 즉시 반환된다.

주의할 점은 디코더 자체가 한번에 처리할 수 있는 프레임의 수가 제한되어 있기 때문에 만약 디코더의 내부 파이프라인이 꽉 찼다면 자리가 생길 때까지 `VTDecompressionSessionDecodeFrame`의 호출 자체가 블록될 수 있다(decoder back pressure). 따라서 비동기 적인 작업이긴 하지만 같은 스레드에서 UI 작업을 수행해선 안된다.

만약 모든 비동기적인 프레임이 디코더에 의해 완전히 처리된 것을 보장하고 싶다면 `VTDecompressionSessionWaitForAsynchronousFrames`를 호출하면 된다. 이 함수의 호출은 모든 프레임이 `VTDecompressionSession`에서 출력되기 전까지 반환되지 않는다.

### Changing `CMVideoFormatDescription`

비디오 프레임 시퀀스를 디코딩하는 과정에서 `CMVideoFormatDescription`를 바꿔야하는 상황이 발생할 수 있다.

Elementary Stream 시퀀스의 경우로 예를 들면, 첫번째 `CMVideoFormatDescription`은 첫번째 SPS와 PPS를 통해 생성할 것이다. 이제  `VTDecompressionSession`을 생성하고 이후의 프레임들을 `CMVideoFormatDescription`를 사용해서 `CMSampleBuffer`로 만들게 된다.

이후 새로운 SPS와 PPS를 만나면 `VTDecompressionSession`의 `CMVideoFormatDescription`을 바꿔줘야 하는데 이때는 `VTDecompressionSessionCanAcceptFormatDescription`을 호출한다.

`VTDecompressionSessionCanAcceptFormatDescription`의 리턴 값이 `true` 이면 이후의 샘플들을 새로운 `CMVideoFormatDescription`과 함께 보내면 되고, `false`이면 기존의 것은 제거하고 새로운 `VTDecompressionSession`을 생성하여 이후 프레임을 보내줘야 한다.

### `VTDecompressionSession` Summary

- `VTDecompressionSession`의 생성
- Output `CVPixerBuffer`의 요구사항 정의
- `VTDecompressionSession`의 동기/비동기적인 수행
- `CMVideoFormatDescription`의 변경

## Reference
[WWDC14](https://developer.apple.com/videos/play/wwdc2014/513/)

