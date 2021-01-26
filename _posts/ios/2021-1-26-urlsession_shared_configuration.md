---
layout: post
title: URLSession, shared와 configuration
category: iOS
tags: [URLSession, shared, configuration]
comments: true
---

>이 글은 공부 중에 작성하는 글입니다.
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

오늘은 URLSession의 Type에 대해서 포스팅합니다. 공식문서를 토대로  작성합니다.

<br>
<br>

# URLSession의 타입
우리가 API로 부터 데이터를 받아오거나 업로드하기 위해서는 가장 먼저 해야할 일은 아마 URLSession을 만드는 일일 것입니다. (혹은 URLSession.shared를 사용하거나) shared와  configuration을 합치면 4가지의 URLSession을 만들 수 있는데요, 이  타입들은 어떤 차이가 있고 어떤 기준에서 선택해야할까요?

```swift
URLSession.shared
let session1 = URLSession(configuration: .default)
let session2 = URLSession(configuration: .ephemeral)
let session3 = URLSession(configuration: .background(withIdentifier: String))
```

<br>

# URLSession.shared
기본적인 요청에 대해서, URLSession은 `shard`라는 **singleton** 을 제공합니다. shared는 기본적으로 제공되는 싱글톤이기 때문에 새로운 프로퍼티를 만들지 않고 사용할 수 있습니다. 하지만 가장 큰 차이점이라면 **delegate와 configuration**을 제공하지 않는다는 것입니다. 기본적으로 간결하게 사용이 가능하지만 보다 디테일한 작업에는 configuration 타입이 더 유리합니다.

### Shared Session의 한계
shared Session은 **delegate와 configuration**을 제공하지 않으므로 아래와 같은 한계가 있습니다.

- 데이터를 주고받는 과정에서 단계별로 데이터를 다루어줄 수 없습니다.
- 기본 연결 동작을 사용자에 맞도록 설정할 수 없습니다.
- 인증(authentication)을 하는 능력이 제한됩니다.
- background에서 다운로드, 업로드 작업을 처리할 수 없습니다.

이는 크게 URLSessionConfiguration와 구분되는 지점이라고 생각이 되는데요, shared는 디테일한 세팅이 필요하지 않은 경우에 쓰기 적합해보입니다. <br>

shared session은 기본적으로 shared [URLCache](https://developer.apple.com/documentation/foundation/urlcache), [HTTPCookieStorage](https://developer.apple.com/documentation/foundation/httpcookiestorage) 및 [URLCredentialStorage](https://developer.apple.com/documentation/foundation/urlcredentialstorage) 개체를 사용하고 shared custom networking protocol list를 사용합니다. (이는 configuration의 default 타입의 기반이 되기도 한다고 하네요.) <br>

일반적으로 shared session에서 작업할 때에는 cache, cookie storage, or credential storage를 커스텀하지 않고, 만약  이러한 요소들의 **커스터마이징이 필요한 경우라면** configuration **default** session을 사용해야합니다.

<br>

# URLSessionConfiguration

```swift
class URLSessionConfiguration : NSObject
```

**configuration**을 번역하면 '환경설정' 혹은 '구성'이라고 이해가 됩니다. (단어가 딱딱해서 조금 어색하네요.) shared에서 언급했듯, 보다 디테일한 세팅이 필요한 경우 사용할 수 있습니다. configuration은 맨 위의 코드에서처럼 URLSession이 초기화될 때 init 메서드에서 설정할 수 있습니다. <br>

configuration은 URLSession이 데이터를 다룰 때, 어떻게 동작할지, 어떤 정책(policy)를 따를지를 결정하는 요소입니다. 구체적으로는 시간 제한 값(timeout values), 캐싱(caching) 정책, 연결시 요구사항, type의 정보를 구성합니다. 즉  **URLSession의 성격** 을 결정하는 것이지요. **주의할 점은 한 번 초기화가 이루어지면 configuration을 변경할 수 없다는  점입니다.** 따라서 초기화할 때 적절한 configuration을 잘 설정해주어야 겠네요. 만약 configuration에 대해 변경이 필요하다면 새로운 URLSession을 만들어서 사용해야 합니다. <br>

URLSessionConfiguration의 종류에는  default, ephemeral, background가 있습니다.

<br>

### ✐ default

```swift
class var `default`: URLSessionConfiguration
```

default session은 (customizing을 많이 하지 않는다면) **shared session과 매우 유사** 합니다. 하지만 **delegate**를 사용하여 데이터를 단계별로(점진적으로) 얻을 수 있다는 차이점이 있습니다. 위에서 다루었던 hared session의 한계를 custom의 가능성, delegate의 사용 측면에서 보완하고 있네요.

<br>

### ✐ ephemeral

```swift
class var ephemeral: URLSessionConfiguration
```

ephemeral session은 또 **default와 유사** 합니다. 하지만 [caches, cookies](https://odong-tree.github.io/cs/2021/01/20/session_cache_cookie/), credentials 또는 세션 관련 데이터를 **사용자의 disk에 저장하지 않는다** 는 차이가  있습니다. 사용자의 disk 대신 **세션 관련 데이터는 RAM에 저장됩니다.**  ephemeral session은 사용자의 disk에 데이터를 저장하는 경우는 URL을 file로 저장할 때 뿐입니다. (default session을 커스텀해서  ephemeral session을 만들 수도 있습니다.) <br>

ephemeral session을 사용하는 가장 큰 이유는 **Privacy(개인 정보 보호)**입니다. 언급했듯이 data를 disk에 저장하지 않기때문에 보안상으로 보다 안전하겠지요? 그래서 ephemeral session은 웹이나 다른 유사한 상황 등에서 비공개 브라우징 모드(private browsing mode)에 적합합니다. <br>

ephemeral session은 cached data를 disk에 저장하지 않으므로 cache의 크기는 사용가능한 RAM에 의해 제한됩니다. 그러므로 이전에 가져온 resource가 cache에 없을 확률이 높겠네요. 또한 RAM은 휘발성 메모리니까 앱을 다시 시작하면 데이터는 사라지게 될 것입니다. 그렇기 때문에 이러한 성격은 앱에 따라서 성능상 불리할 수도 있을 것 같군요. (앱이 일시중지될 때 메모리의 캐시가 자동으로 제거되지는 않지만 앱이 종료되거나 시스템에 메모리  부족이 발생하면 제거될 수 있습니다.)

<br>

### ✐ background

```swift
class func background(withIdentifier identifier: String) -> URLSessionConfiguration
```

background는 특이하게 **return 메서드** 형태로 되어있네요. 이름처럼 이 메서드는 **background에서 데이터 파일을 전송하는데에** 적합한 configuration을 만들어줍니다. iOS에서 background session은 앱 자체가 일시 중지되거나 종료된 경우에도 데이터 전송을 유지할 수 있습니다. <br>

background는 왜 메서드로 되어있을까요? 그 이유는 **identifier** 를 가지기 위해서입니다. 만약 iOS 앱이 **시스템에 의해** 종료되고 다시 시작된다면 앱이 **identifier** 을 사용하여 **새로운 configuration session을 생성하고 종료된 시점의 데이터 전송 상태를 알 수 있습니다.** (이는  정상적으로 앱이 종료되었을 때에만 유효합니다!) 만약 사용자가 **멀티태스킹** 화면에서 앱을 종료하면 시스템은 세션의 **모든 백그라운드의 전송을 취소**합니다. 또한 시스템은 유저가 강제 종료한 앱을 자동으로 다시 시작하지는 않습니다. 이때 전송을 재개하고 싶다면 유저가 직접 앱을 다시 시작해주어야 합니다. <br>

background session에서 **isDiscretionary** 프로퍼티를 사용하여 최적의 성능을 위해 **시스템의 재량에 따라 전송을 예약하도록** background session을 구성 할 수 있습니다. **많은 양의 데이터를 전송할 때는이 속성 값을 true로 설정하는 것이 좋습니다.** <br>

-------------
<br>

추가적으로 [Downloding Files in the background](https://developer.apple.com/documentation/foundation/url_loading_system/downloading_files_in_the_background) 문서에서 background session에 대해서 더 알아볼 수 있을 것 같네요! 사용하게 되는 날 이 문서에 대해 포스팅을 해보도록  하겠습니다 :)






<br>
<br>

#### 참고
- [https://developer.apple.com/documentation/foundation/urlsession/1409000-shared](https://developer.apple.com/documentation/foundation/urlsession/1409000-shared)
- [https://developer.apple.com/documentation/foundation/urlsessionconfiguration#1660412](https://developer.apple.com/documentation/foundation/urlsessionconfiguration#1660412)
- [https://developer.apple.com/documentation/foundation/urlsession](https://developer.apple.com/documentation/foundation/urlsession)
- [https://developer.apple.com/documentation/foundation/urlsessionconfiguration/1410529-ephemeral](https://developer.apple.com/documentation/foundation/urlsessionconfiguration/1410529-ephemeral)
- [https://developer.apple.com/documentation/foundation/urlsessionconfiguration/1407496-background](https://developer.apple.com/documentation/foundation/urlsessionconfiguration/1407496-background)


<br>
<br>
