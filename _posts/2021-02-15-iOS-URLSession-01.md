---
title: URLSession 알아보기!
layout: single
comments: true
share: true
categories: 
- iOS
tag:
- Networking
last_modified_at: 2021-02-15 T14:42:00+08:00
---

## Overview

`URLSession` 클래스는 URL로 지정된 엔드포인트에서 데이터를 다운받거나 데이터를 업로드하기 위한 API를 제공한다. 이 API를 통해서 백그라운드 다운로드 작업을 수행할 수도 있다. `URLSessionDelegate`, `URLSessionTaskDelegate`를 통해서 인증(authentication)을 지원하고 이벤트를 받을 수도 있다.

앱에서 하나 이상의 `URLSession` 인스턴스를 생성할 수 있고, 각각은 관련 데이터 전송 작업 그룹을 조정한다(coordinates). 예를 들어 웹 브라우저를 생성한다면, 하나의 탭 혹은 윈도우당 하나의 세션을 생성하거나, 하나의 세션은 대화식(interactive) 방식으로 다른 하나는 백그라운드 다운로드 작업으로 생성할 것이다. 이후 앱은 각 세션 내에 특정 URL에 대한 요청을 나타내는 일련의 작업을 추가한다.

## Types of URL Sessions

주어진 URL 세션 내의 작업은 단일 호스트에 대한 최대 동시 연결 수, 연결이 셀룰러 네트워크를 사용할 수 있는지 여부 등과 같은 연결 동작을 정의하는 공통 세션 구성 객체를 공유한다.

`URLSession`에는 기본 요청에 대한 싱글톤 `shared` 세션이 있다. 세션을 커스터마이징 할 수는 없지만 요구 사항이 매우 제한적인 경우 좋은 출발점 역할을 한다. 다른 종류의 세션의 경우 다음 세 가지 구성(configurations) 중 하나를 통해 `URLSession`을 만든다.

- Defalut session: 기본 세션은 공유 세션과 매우 유사하게 작동하지만 원하는 방식으로 커스터마이징할 수 있다. 또한 데이터를 점진적으로 가져 오기 위해 기본 세션에 델리게이트를 지정할 수도 있다.
- Ephemeral session: 임시 세션은 공유 세션과 유사하지만 캐시, 쿠키 또는 사용자 인증 정보를 디스크에 쓰지 않는다.
- Background session: 백그라운드 세션을 사용하면 앱이 실행되지 않는 동안 백그라운드에서 콘텐츠 업로드 및 다운로드를 수행 할 수 있다.

## Types of URL Session Tasks

세션 내에 선택적으로 데이터를 서버에 업로드 한 다음 서버에서 데이터를 디스크의 파일 또는 메모리의 하나 이상의 `NSData` 객체로 가져오는는 작업(task)을 생성한다. `URLSession` API는 네 가지 유형의 작업을 제공한다.

- 데이터 작업(`dataTask`)은 `NSData` 객체를 사용하여 데이터를 보내고 받습니다. 데이터 작업은 서버에 대한 짧은 대화형 요청을 위한 작업이다.
- 업로드 작업(`uploadTask`)은 데이터 작업과 유사하지만 데이터를 보내고 앱이 실행되지 않는 동안 백그라운드 업로드를 지원한다.
- 다운로드 작업(`downloadTask`)은 파일 형식으로 데이터를 가져오고 앱이 실행되지 않는 동안 백그라운드 다운로드 및 업로드를 지원합니다.
- WebSocket 작업(`webSocketTest`)은 RFC 6455에 정의 된 WebSocket 프로토콜을 사용하여 TCP 및 TLS를 통해 메시지를 교환한다.

## Using a Session Delegate

세션의 작업은 공통 델리게이트 객체 또한 공유한다. 다음과 같은 경우를 포함하여 다양한 이벤트가 발생할 때 정보를 제공하고 얻기 위해 델리게이트를 구현한다.

- 인증에 실패한 경우.
- 서버에서 데이터 도착한 경우.
- 데이터를 캐싱할 수 있게 되는 경우.

델리게이트가 제공하는 기능이 필요하지 않은 경우, 세션을 만들 때 `nil`을 전달하여 이 API를 사용할 수 있다.

> 세션 객체는 델리게이트에 앱이 종료되거나 명시적으로 세션을 무효화(invalidates)하기 전까지 강한 참조를 유지한다. 만약 명시적으로 세션을 무효화하지 않는다면, 메모리 누수가 발생한다.

## Asynchronicity and URL Sessions

대부분의 네트워킹 API와 마찬가지로 `URLSession` API는 비동기적이다. 호출하는 메소드에 따라 다음 두 가지 방법 중 하나로 앱에 데이터를 반환한다.

- 전송이 성공적으로 완료되거나 오류가 있는 경우 완료 핸들러 블록을 호출한다.
- 데이터가 도착하고 전송이 완료되면 세션의 델리게이트에서 메서드를 호출한다.

이와 같은 정보를 델리게이트에 전달하는 것 외에도, `URLSession`은 쿼리 할 수 있는 상태 및 진행률 프로퍼티를 제공한다.

## App Transport Security (ATS)

iOS 9.0 및 macOS 10.11 이상은 `URLSession`으로 이루어진 모든 HTTP 연결에 대해 앱 전송 보안(ATS)을 사용한다. ATS는 HTTP 연결이 HTTPS를 사용하는 것을 요구한다.

## Foundation Copying Behavior

세션 및 작업 객체는 다음과 같이 `NSCopying` 프로토콜을 준수한다(conform).

- 앱이 세션 또는 작업 객체를 복사하면 동일한 개체를 다시 가져온다.
- 앱이 구성(configuration) 객체를 복사하면 독립적으로 수정할 수 있는 새 복사본이 생성된다.

## Thread Safety

URL 세션 API는 thread-safe 하다. 모든 스레드 컨텍스트에서 세션과 작업을 자유롭게 만들 수 있다. 델리게이트 메소드가 제공된 컴플리션 핸들러를 호출하면 작업이 올바른 델리게이트 큐에 자동으로 예약된다(scheduled).

## Reference

[Apple Developer](https://developer.apple.com/documentation/foundation/urlsession)
