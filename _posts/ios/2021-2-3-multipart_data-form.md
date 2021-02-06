---
layout: post
title: URLSession 이미지 업로드하기, multipart/data-form
category: iOS
tags: [multipart, HTTP]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.       
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

오늘은 URLSession을 이용해서 **Content-Type이 multipart/data-form인 경우, 이미지를 함께 업로드하는 방법**에 대해서 포스팅합니다. 처음 이 주제에 대해서 코드를 쓰느라 정말정말 많이 헤맸던 것 같습니다. 저도 차근차근 이해하면서 포스팅을 해보도록 하겠습니다. Content-Type에 대해서는 [URLSession과 setValue, Content-Type 포스팅](https://odong-tree.github.io/ios/2021/01/30/urlrequest_mimetype/)을 간단하게 읽어보시면 좋을 것 같습니다! <br>

프로젝트 구현 중에.. 만약 JSON을 넘겨주는, Content-Type이 "application/json"인 경우에는 `JSONEncoder`로 인코딩해서 바로 보내면 되겠지만, Image 파일을 포함해서 보내야 하는 **multipart/data-form** 인 경우에는 JSONEncoder로 한번에 인코딩해서는 같이 보낼 수가 없더군요. 그렇기 때문에 **URLRequest에 들어갈 Body를** multipart/data-form에 맞도록 잘 만들어주어야 합니다.




<br>
<br>

# multipart/data-form

> 주로 [MDN 원문 링크](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types)를 참고한 내용입니다. 자세한  내용은 원문 링크를 통해 확인할 수 있습니다.

multipart/form-data는 **멀티 파트 문서 형식으로써, boundary(이중 대시 '--' 로 시작되는 문자열)로 구분되어지는 다른 파트들로 구성됩니다.** 즉, 다양한 타입의 데이터를 함께 보내기위한 타입입니다. 각 파트는 그 자체로 개체이며 자신만의 HTTP 헤더를 가지는데, 파일 업로드 필드를 위한 헤더로는 Content-Disposition, 그리고 가장 일반적인 것 중 하나인 Content-Type이 있습니다(Content-Length은 경계선이 구분자로 사용되므로 무시됩니다).

```
Content-Type: multipart/form-data; boundary=aBoundaryString
(other headers associated with the multipart document as a whole)

--aBoundaryString
Content-Disposition: form-data; name="myFile"; filename="img.jpg"
Content-Type: image/jpeg

(data)
--aBoundaryString
Content-Disposition: form-data; name="myField"

(data)
--aBoundaryString
(more subparts)
--aBoundaryString--
```

이렇게 boundary로 데이터를 구분짓는 것이 눈에 띄네요. 우리가 코드로  만들어줄 Body가 이런 비슷한 모양이 될거라고 생각할 수 있습니다.

<br>

### ✐ Content-Disposition: form-data
>[MDN 원문 링크](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Content-Disposition)에서 발췌한 내용입니다. 자세한 내용은 원문에서 확인 가능합니다.

일반적인 HTTP 응답에서 Content-Disposition 헤더는 컨텐츠가 브라우저에 inline 되어야 하는 웹페이지 자체이거나 웹페이지의 일부인지, 아니면 attachment로써 다운로드 되거나 로컬에 저장될 용도록 쓰이는 것인지를 알려주는 헤더입니다.
multipart/form-data 본문에서의 Content-Disposition 일반 헤더는 multipart의 하위파트에서 활용될 수 있는데, 이 때 **이 헤더는 multipart 본문 내의 필드에 대한 정보를 제공합니다.** multipart의 하위파트는 Content-Type 헤더에 정의된 boundary 구분자에 의해 구분되며, Content-Disposition 헤더를 multipart 자체에 사용하는 것은 아무런 효과를 발휘하지 못합니다. <br>

Content-Disposition 헤더는 광의의 MIME 맥락 속에서 정의되었는데, 그 정의에서 사용되는 파라미터 중 일부인 form-data, name 그리고 filename만이 HTTP forms와 POST 요청에 적용될 수 있습니다. 여기서 name과 filename은 필수적인 파라미터는 아닙니다.

```swift
Content-Disposition: form-data
Content-Disposition: form-data; name="fieldName"
Content-Disposition: form-data; name="fieldName"; filename="filename.jpg"
```




<br>
<br>

# URLSession으로 이미지 업로드하기
multipart/data-form에 대해 살펴봤으니 이제 코드로 이미지를 업로드해보도록 하겠습니다. 코드를 작성하는 방법은 **아주아주 다양할 것입니다.** 그래서 포스팅의 편의상 Odong이라는 구조체를 업로드한다고 생각해보겠습니다. 포스팅에 사용된 코드는 직접 쓴 코드이기  때문에 하나의 예시로만 봐주시면 감사하겠습니다 :)

```swift
struct Odong {
    let name: String = "Odongnamu"
    var age: Int = 500
    var height: Double = 11.5
    var image: UIImage = UIImage(named: "odong.jpg")!
}
```

만약  API에서 요구되는 Content-Type이 "application/json"이었다면 간단하게 JSONEncoder로 인코딩해서 업로드하면 될 것입니다. 하지만 Content-Type이 **"multipart/data-form"** 이고 Odong의 image 파일을 함께 업로드해야 한다면 직접 Body에 들어갈 Data를 만들어주어야 합니다. 코드에 앞서 간단하게 먼저 이야기하면,
## Body = image 외의 데이터 + 이미지 데이터
이렇게 하나의 Data에 "이미지를 제외한 데이터" 뒤에 이어서 "이미지 데이터"를 넣는 것으로 Body를 만들 수 있습니다. 즉 하나의 Data를 만드는 것이지만 두 영역으로 나누어 차례대로 넣어준다고 생각할 수 있습니다. 코드를 먼저 보여드리고 하나하나 짚어가도록 하겠습니다.
<br>

```swift
// 사전 준비
let boundary = generateBoundaryString()
let boundaryPrefix = "--\(boundary)\r\n"
var parameters: [String: String] = [
    "name": "Odongnamu",
    "age": "500",
    "height": "11.5"
]
// image 데이터를 만들기위한 요소들
let imageData: Data = UIImage(named: "odong.jpg")!.jpegData(compressionQuality: 1)!
let mimeType: String = "image/jpg"
let filename: String = "odong.jpg"
let imageKey: String = "image"

func generateBoundary() -> String {
        return "Boundary-\(UUID().uuidString)"
    }

func makeBody() -> Data {
    var bodyData = Data()

    for (key, value) in parameters {
        bodyData.append(boundary.data(using: .utf8)!)
        bodyData.append("Content-Disposition: form-data; name=\"\(key)\"\r\n\r\n".data(using: .utf8)!)
        bodyData.append("\(value)\r\n".data(using: .utf8)!)
    }

    bodyData.append(boundary.data(using: .utf8)!)
    bodyData.append("Content-Disposition: form-data; name=\"\(imageKey)\"; filename=\"\(filename)\"\r\n".data(using: .utf8)!)
    bodyData.append("Content-Type: \(mimeType)\r\n\r\n".data(using: .utf8)!)
    bodyData.append(imageData)
    bodyData.append("\r\n".data(using: .utf8)!)
    bodyData.append("--".appending(boundary.appending("--")).data(using: .utf8)!)

    return bodyData
}

let url = URL(string: "업로드할 URL")!
var request = URLRequest(url: url)
request.setValue(“multipart/form-data; boundary=\(boundary)”, forHTTPHeaderField: "Content-Type")
request.httpBody = makeBody()

URLSession.shared.dataTask(with: request)
```

<br>


### ✐ UUID().uuidString

```swift
let boundary = generateBoundaryString()
let boundaryPrefix = "--\(boundary)\r\n"

func generateBoundary() -> String {
        return "Boundary-\(UUID().uuidString)"
    }
```

제 코드에는  없지만 여러 레퍼런스들을 찾아보면 Boundary에 UUID를 사용하는 것을 볼 수 있었습니다. **Boundary에 UUID를  사용하는 이유는 multipart form이 서버에 데이터를  꼭 한번에 보낼 수 없기  때문에  어떤 데이터에  대한 것인지 식별자를 추가해주는 것이라고 합니다.** 이때 꼭 UUID가 없어도 에러가 나지는 않습니다. 하지만 서버에서 어떤 데이터인지 식별하지 못하는 일이 발생할지도 모르겠네요. <br>

```swift
let url = URL(string: "업로드할 URL")!
var request = URLRequest(url: url)
request.setValue(“multipart/form-data; boundary=\(boundary)”, forHTTPHeaderField: "Content-Type")
request.httpBody = makeBody()

URLSession.shared.dataTask(with: request)
```
boundary Value는 헤더에서 가지게 하여 값을 비교할  수 있도록 해주고 있네요. 그럼 UUID가 뭔지도  간단히  살펴볼까요? <br>


#### ✐ UUID
UUID는 Universally Unique IDentifier의 약자로, **고유의 값** 을 구하기위해 많이 사용합니다. UUID는 16 옥텟 (128비트)의 수입니다. 표준 형식에서 UUID는 32개의 십육진수로 표현되며 총 36개 문자(32개 문자와 4개의 하이픈)로 된 **8-4-4-4-12** 라는 5개의 그룹을 하이픈으로 구분합니다.

```swift
print(UUID().uuidString) // D3392A30-8B61-4CF6-9F39-BBF5FF284715

// uuidString은 랜덤의 UUID를 가지는 프로퍼티입니다.
```
이런식으로 무작위로 만들어낼 수  있는 UUID 는 340,282,366,920,938,463,463,374,607,431,768,211,456개라고 하네요.  Int나 다른 랜덤값에 비해 표현되는 랜덤값이 매우 많아서 UUID가 겹칠 확률은 어떤 랜덤값보다 낮을 것 같네요. UUID에 대한 더 자세한 내용은 [UUID 위키백과](https://ko.wikipedia.org/wiki/범용_고유_식별자)에서 확인할 수 있습니다.

<br>

### ✐ parameters

```swift
var parameters: [String: String] = [
    "name": "Odongnamu",
    "age": "500",
    "height": "11.5"
]
```

이 부분은 우리가 업로드하고 싶었던 구조체의 형태를 딕셔너리 형태로 나타낸 것입니다. 함께 업로드해야하는 이미지때문에 인코딩을 해줄 수 없어서 직접 딕셔너리의 형태로 만들어 Body에 추가하는 작업이 필요합니다. 그래서 아래에서는 for 문을 돌면서 하나씩 body에 추가해주고 있습니다. <br>

딕셔너리에 들어가는 요소를 매번 수동으로 적어줄 수는 없겠지요? 직접 적절한 코드를 만들어보면 좋겠네요 :)

<br>

### ✐ \r\n
/r과 /n은 입출력 제어문자의 종류입니다. 쉽게말하면 줄바꿈의 기호로 사용되고 있습니다.

- **\r** = CR (Carriage Return) → Used as a new line character in Mac OS before X
- **\n** = LF (Line Feed) → Used as a new line character in Unix/Mac OS X
- **\r\n** = CR + LF → Used as a new line character in Windows

StackOverFlow에 나온대로는 이렇다는데.. 코드에서는 **\r이나 \n이 아니라 \r\n을 써주어야 업로드가 정상적으로 되더라구요.** 이 부분은 서버의 캐릭터셋에 따르는 것이라고 하네요. <br>

>이름의 유래는, \r은 Carriage Return(CR), \n은 Line Feed(LF)라는 의미로 예전의 타자기에서 생겨난 용어라고 합니다. Carriage Return은 타자를 치던 줄의 맨 앞으로 이동하는 것이고 Line Feed는 다음줄로 넘어가는 것이라고 합니다. 지금의 컴퓨터의 자판은 엔터를 입력하면 자동으로 다음줄, 맨 처음으로 이동하지만 과거의 모든 것이 수동이었던 자판기는 직접 줄을 내려주고 줄의 맨 처음으로 이동해야했다고 하네요. <br>
더 자세한 이야기는 [이 블로그](https://m.blog.naver.com/taeil34/221325864981)에서 재밌게 설명하고 있네요!





<br>

<br>

## ✐ 만약 이미지가 여러개라면?
업로드할 이미지가 하나가 아니라 여러개라면 어떻게 코드를 쓸 수 있을까요? 위에서 작성했던

```swift
// image 데이터를 만들기위한 요소들
let imageData: Data = UIImage(named: "odong.jpg")!.jpegData(compressionQuality: 1)!
let mimeType: String = "image/jpg"
let filename: String = "odong.jpg"
let imageKey: String = "image"
```

요소들을 하나의 묶음으로 만들어서 여러 묶음을 가지고 있으면 될 것 같군요. **Alamofire** 라는 URLSession이 하는 일을 보다 간편하게 사용할 수  있는 라이브러리가 있습니다. 추후에 다루어 보겠지만 공부도할겸 URLSession을 고집해봅시다. 아래의 코드는 Alamofire의 코드에서 힌트를 얻어 URLSession을 이용해 작성한 코드입니다. <br>

```swift
// 사전 준비
let boundary = generateBoundaryString()
let boundaryPrefix = "--\(boundary)\r\n"

func generateBoundary() -> String {
        return "Boundary-\(UUID().uuidString)"
    }
    
var parameters: [String: String] = [
    "name": "Odongnamu",
    "age": "500",
    "height": "11.5"
]

let image1 = UIImage(named: "odong1.jpg")!
let image2 = UIImage(named: "odong2.jpg")!
let image3 = UIImage(named: "odong3.jpg")!

var imageList: [UIImage] = [image1, image2, image3]
var imageListToUpload: [ImageInfo] = []

struct ImageInfo {
    let imageData: Data
    let fileName: String
    let mimeType: String
    let imageKey: String
}

func makeImageListToUpload(imageList: [UIImage]) {
    let mimeType = "image/jpg"
    let imageKey = "image"
    for (index, image) in imageList.enumerated() {
        if let imageData = image.jpegData(compressionQuality: 1) {
            let image = ImageInfo(imageData: imageData, fileName: "odong\(index + 1).jpg", mimeType: mimeType, imageKey: imageKey)
            ImageListToUpload.append(image)
        }
    }
}


func makeBody(parametes: [String: String], imageList: [ImageInfo]) -> Data {
    var bodyData = Data()

    for (key, value) in parameters {
        bodyData.append(boundary.data(using: .utf8)!)
        bodyData.append("Content-Disposition: form-data; name=\"\(key)\"\r\n\r\n".data(using: .utf8)!)
        bodyData.append("\(value)\r\n".data(using: .utf8)!)
    }

    for i in imageList {
        bodyData.append(boundary.data(using: .utf8)!)
        bodyData.append("Content-Disposition: form-data; name=\"\(i.imageKey)\"; filename=\"\(i.fileName)\"\r\n".data(using: .utf8)!)
        bodyData.append("Content-Type: \(i.mimeType)\r\n\r\n".data(using: .utf8)!)
        bodyData.append(i.imageData)
        bodyData.append("\r\n".data(using: .utf8)!)
        bodyData.append("--".appending(boundary.appending("--")).data(using: .utf8)!)
    }

    return bodyData
}

makeImageListToUpload(imageList: imageList)

let url = URL(string: "업로드할 URL")!
var request = URLRequest(url: url)
request.setValue(“multipart/form-data; boundary=\(boundary)”, forHTTPHeaderField: "Content-Type")
request.httpBody = makeBody(parametes: parameters, imageList: imageListToUpload)

URLSession.shared.dataTask(with: request)
```


<br>
<br>

#### 참고
- [https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics_of_HTTP/MIME_types)
- [https://developer.mozilla.org/ko/docs/Web/HTTP/Methods/POST](https://developer.mozilla.org/ko/docs/Web/HTTP/Methods/POST)
- [https://nsios.tistory.com/39](https://nsios.tistory.com/39)
- [https://yagom.net/forums/topic/multipart통신-이미지-body-업로드/](https://yagom.net/forums/topic/multipart통신-이미지-body-업로드/)
- [https://medium.com/@johnxavier034/uploading-array-of-images-using-multipart-form-data-in-swift-5d0cf8fc3361](https://medium.com/@johnxavier034/uploading-array-of-images-using-multipart-form-data-in-swift-5d0cf8fc3361)
- [https://www.donnywals.com/uploading-images-and-forms-to-a-server-using-urlsession/](https://www.donnywals.com/uploading-images-and-forms-to-a-server-using-urlsession/)

<br>
<br>
