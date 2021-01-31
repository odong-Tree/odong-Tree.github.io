---
layout: post
title: URLRequest의 setValue, MIMEType
category: iOS
tags: [URLRequest, MIMEType]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.       
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  

<br>


```swift
var request = URLRequest(url: url)
request.httpMethod = "POST"
request.setValue("application/json",
                  forHTTPHeaderField: "Content-Type")
```

이번 포스팅은 URL과 관련된 코드를 쓰다가.. 이 부분에 대해서 이해하고 싶어서 작성하게 되었습니다. POST를 보내기위해 코드를 찾아보다가 발견했는데요, "application/json"과 "Content-Type"은 무엇을 의미하는 것일까요? <br>

URLRequest에 대해서 아주 간단하게 포스팅하고 `setValue`를 이해해보겠습니다.

<br>
<br>

# URLRequest

```swift
struct URLRequest
```

URLRequest는 **load할 URL** 과 **load할 때 사용되는 캐시정책(CachePolicy)** 을 캡슐화한 **구조체**입니다. 또한  HTTP 및 HTTPS 요청의 경우  URLRequest에는 GET, POST와 같은 HTTP 메서드와 HTTP 헤더가 포함됩니다. 즉, URL과 캐시정책뿐만 아니라 다양한 요청에 대한 사항들을 세팅할 수 있습니다. (HTTP메서드와 헤더에 대해서는 [HTTP와 HTTPS](https://odong-tree.github.io/cs/2021/01/18/http/)에서 언급한 바 있습니다.) <br>

URLRequest는 **요청**만을 담고 있는 타입입니다. 실제로  요청이 이루어지기 위해서는 **URLSession**을 통해 URLRequest를 다루어줄 수 있습니다.
<br>

```swift
var cachePolicy: URLRequest.CachePolicy
var httpMethod: String?
var httpBody: Data?
func addValue(String, forHTTPHeaderField: String)
func setValue(String?, forHTTPHeaderField: String)
var networkServiceType: URLRequest.NetworkServiceType
...
```

그리고 이와 같은 코드로 요청에 대한 구체적인 세팅을 해줄 수 있습니다. 더 다양한 코드들은 [URLRequest 공식문서](https://developer.apple.com/documentation/foundation/urlrequest)에서 확인할 수 있습니다. 이 중에서 오늘은 **setValue와 addValue** 메서드를 살펴보도록  하겠습니다.

<br>


>**추가로 보면 좋은  문서**
>- [Fetching Website Data into Memory](https://developer.apple.com/documentation/foundation/url_loading_system/fetching_website_data_into_memory)
>- [Uploading Data to a Website](https://developer.apple.com/documentation/foundation/url_loading_system/uploading_data_to_a_website)
>- [NSURLRequest](https://developer.apple.com/documentation/foundation/nsurlrequest)와 [NSMutableURLRequest](https://developer.apple.com/documentation/foundation/nsmutableurlrequest)
   (Objective-C의 이 클래스들이 Swift에서 구조체로 넘어온 것이 URLRequest 구조체입니다.)




<br>
<br>

# setValue, addValue

```swift
var request = URLRequest(url: url)
request.httpMethod = "POST"
request.setValue("application/json",
                  forHTTPHeaderField: "Content-Type")
```

setValue와 addValue 메서드의 모양은 거의 똑같습니다. 이 차이점에 대해서는 뒤에서 언급하도록 하고.. 다시 처음 코드로 돌아가서, "application/json"과 "Content-Type"은 무엇을 의미하는지 알아볼까요? 크롬에서 찾아보면,

![스크린샷 2021-01-30 오후 4 42 40](https://user-images.githubusercontent.com/73867548/106350711-56b60100-631a-11eb-8569-94ebf97e927d.jpg)       

이렇게 표시가 되는 것을 확인할 수 있었습니다. 그러면 각각이 무엇인지 알면 되겠네요!

<br>

### ✐ Content-Type
위의 메서드는 사진처럼 Content-Type 이라는 Response Header에 원하는 값을 넣어주는 것입니다. **Content-Type** 에는 **MIMEType** 의 값이 들어오게 됩니다. Content-Type은 서버에게 "**이러한 타입(MIMEType)으로 데이터를 보내겠다**"는 의미입니다. 이름부터가 어떤 정보를 담는지 느낌이 옵니다. <br>

그렇다면 사진 속의 "image/png"나 "application/json"은 **MIMEType**이라는 것인데.. MIMEType은 무엇일까요?

<br>

### ✐ MIMEType
MIME은 **Multipurpose Internet Mail Extensions** 의 약자로 파일 변환을 의미합니다. 지금은  웹에서 파일을 전달할 때 사용하고 있지만, 처음의 MIMEType은 Mail에서 파일을  전달하기 위해 개발되었다고 하네요.  <br>

텍스트만 주고받던 시절에는 ASCII 라는 공통 표준만 따르면 문제가 없었으나, 이후에 텍스트가 아닌 파일을  주고받는 경우가 발생하면서 이러한 파일들을 텍스트로 변환하기 위해서 **MIMEType이  등장**하게 되었습니다. <br>


>**MIME 타입**이란 클라이언트에게 전송된 문서의 다양성을 알려주기 위한 메커니즘입니다. 웹에서 파일의 확장자는 별 의미가 없습니다. 그러므로, 각 문서와 함께 올바른 MIME 타입을 전송하도록, 서버가 정확히 설정하는 것이 중요합니다. 브라우저들은 리소스를 내려받았을 때 해야 할 기본 동작이 무엇인지를 결정하기 위해 대게 MIME 타입을 사용합니다. - [원문 링크](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types)

<br>


```swift
// MIMEType
type/subtype

// example
image/png
application/json
```

MIMEType은 이와 같은 구조로 구성되어 있습니다. 딱 예시를 보면 **image 중 png타입** 이라고 쉽게 이해할 수 있습니다. <br>

첫번째로 분류되는 type에는 **video, image, audio, text, font, application, multipart** 등이 있습니다. application은 모든 종류의 이진 데이터를 나타냅니다. 그리고 그 아래의 subtype은 그리고 파일의 종류가 다양한 만큼 아주 다양합니다. 따라서 MIMEType에는 종류가 아주 많습니다. 다양한 타입에 대해서는 [Common MIME types](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Common_types) 사이트를 참고하면 좋을 것 같습니다. <br>

```swift
var request = URLRequest(url: url)
request.httpMethod = "POST"
request.setValue("application/json",
                  forHTTPHeaderField: "Content-Type")
```

그러면 다시 코드를 살펴보면, 이 코드는 **내가 서버에 json 타입의 데이터를 보내겠다** 는 의미가 되겠네요.

<br>

![스크린샷 2021-01-30 오후 5 25 18](https://user-images.githubusercontent.com/73867548/106351527-2f623280-6320-11eb-8fec-3ca1703ee496.jpg)         

그리고 추가적으로 헤더의 accept는 **서버로부터 이러한 타입의 데이터를 받고싶다.** 는 의미입니다.

<br>

### ✐ setValue와 addValue의 차이
URLRequest의  메서드에서 [setValue](https://developer.apple.com/documentation/foundation/urlrequest/2011447-setvalue)와 [addValue](https://developer.apple.com/documentation/foundation/urlrequest/2011522-addvalue)가  유사해 보입니다.

![스크린샷 2021-01-30 오후 10 43 39](https://user-images.githubusercontent.com/73867548/106357963-b200e700-634c-11eb-83c0-ca10e1d2c640.jpg)

![스크린샷 2021-01-30 오후 10 43 50](https://user-images.githubusercontent.com/73867548/106357976-c644e400-634c-11eb-8937-8351f20f0f80.jpg)

생긴 모양도, 설명도 거의 똑같네요. 무슨 차이가 있을까요?

<br>


![스크린샷 2021-01-30 오후 10 47 08](https://user-images.githubusercontent.com/73867548/106358045-1d4ab900-634d-11eb-942f-f8dca4ca446e.jpg)


**이 메서드는 header field에 점진적으로 값을 추가합니다. 만약 이전 field에 값이 set되어 있다면, 추가한 값은 구분 기호(쉼표)로 구분되어 기존 값에 추가됩니다.** <br>

addValue의 Discussion에 따르면 addValue는 값이 추가되는 메서드이고, setValue는 값을 replace하는 메서드인 것 같습니다.





<br>
<br>

#### 참고
- [https://developer.apple.com/documentation/foundation/urlrequest](https://developer.apple.com/documentation/foundation/urlrequest)
- [https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy/useprotocolcachepolicy](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy/useprotocolcachepolicy)
- [https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy](https://developer.apple.com/documentation/foundation/nsurlrequest/cachepolicy)
- [https://developer.apple.com/documentation/foundation/cachedurlresponse](https://developer.apple.com/documentation/foundation/cachedurlresponse)
- [https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types)





<br>
<br>
