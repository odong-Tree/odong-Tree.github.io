---
layout: post
title: Notification, Notification Center
category: iOS
tags: [Notification]
comments: true
---

>이 글은 공부 중에 작성하는 글입니다.      
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

객체간의 데이터를 주고 받는 방법에는 여러가지가 있을텐데요, 오늘은 그 중에서 Notification에 대해서 포스팅해보려 합니다. 부스트코스와 블로그, Xcode에서의 테스트를 바탕으로 작성합니다.

<br>
<br>

# Notification <br>

<img src = "/assets/post-img/ios/2020-12/notification1.jpg">             

Notification은 Notification Center를 통해 정보를 전달하기 위한 구조체입니다. 바로 뒤에서 다루어보겠지만, Notification Center는 방송국과 유사한 역할을 합니다. Notification은 '방송' 자체의 느낌이네요. <br>

어떤 ViewController에서 Notification을을 'post'하면 Observer가 등록된 다른 ViewController에서 Notification을 감지해서 적절하게 동작하는 방식입니다. 우선은 이 정도로만 이해하고 차근차근 살펴보겠습니다. <br>

Notification의 주요 프로퍼티는 **name, object, userInfo**가 있습니다.

```swift
var name: Notification.Name
// 알림을 식별하는 태그(이름), 방송의 제목 정도로 이해할 수 있습니다.

var object: Any?
// object는 post를 보내는 객체, 방송을 보내는 객체입니다.

var userInfo: [AnyHashable : Any]?​
// userInfo는 방송과 함께 같이 보낼 데이터를 말합니다.
```

여기서 Notification.Name은 Int, String과 같은 하나의 **타입**입니다.

<br>

# Notification Center
Notification Center는 앞서 말했듯이 방송국과 같습니다. 어떤 객체에서 post한  Notification을 받아서, 등록된 Observer에게 Notification 메세지를 전달하는 역할을 합니다. <br>

```swift
 class var `default`: NotificationCenter { get }​
```

Notification Center는 싱글톤으로 사용됩니다. 그래야 다른 모든 객체가 같은 메세지를 받을 수 있기 때문입니다. <br>

주요 메서드로는 **post, addObserver, removeObserver**가 있습니다. 참고로 [멍구님의 블로그](https://0urtrees.tistory.com/26)에서 예제를 따라해보았는데, 이해가 쉬웠습니다. 감사합니다 :)

<br>

### ✐ post
post는 Notification Center에 Notification을 보내는 것입니다. 예를 들어, 버튼을 눌렀을 때 모든 Observer에게 알리고 싶다면,

```swift
// 기본형
func post(name aName: NSNotification.Name,
   object anObject: Any?,
 userInfo aUserInfo: [AnyHashable : Any]? = nil)


// 버튼으로 Notification으로 알리기
@IBAction func postButton(_ sender: Any) {
    NotificationCenter.default.post(name: Notification.Name(rawValue: "PostButton"),
    object: nil,
    userInfo: nil)
}
```

 이렇게 작성해줄 수 있습니다.

 <br>

### ✐ addObserver
 addObserver는 post된 Notification을 감지하고 싶은 객체에게 Observer를 추가하는 메서드입니다. 이번에는 위에서 post한 PostButton을 감지하는 옵저버를 추가해보겠습니다.

```swift
// 기본형
func addObserver(_ observer: Any,
                   selector aSelector: Selector,
                   name aName: NSNotification.Name?,
                   object anObject: Any?)
```

```swift
override func viewDidLoad() {
    super.viewDidLoad()

    NotificationCenter.default.addObserver(self,
      selector: #selector(someMethod),
      name: Notification.Name(rawValue: "PostButton"),
      object: nil)
}

@objc func someMethod(_ noti: Notification) {
    // some code
}
```

기본형을 뜯어보면, observer가 Notification.Name과 일치하는 Notification을 감지하면 selector를 실행시킬 것이라는 코드입니다. 이때 object를 설정해주면, 특정 객체에서 보내는 Notification만 감지할 수 있습니다. <br>

즉, 아래의 코드를 보면, self가 PostButton이라는 Notification.Name을 감지하면 someMethod를 실행시킬 것이다. 라는 코드가 되겠네요!
<br>

>조금 다르게 생긴 addObserver도 있습니다. <br>
>**addObserver(forName:object:queue:using:)**
 노티피케이션을 노티피케이션 대기열(Queue)과 대기열(Queue)에 추가할 블록(스위프트의 클로저), 노티피케이션 이름을 노티피케이션 센터의 메서드를 가리키는 장소(디스패치 테이블, Dispatch Table)에 이름을 추가합니다. 여기서 object에 특정 객체를 명시하면 명시한 객체가 발송한 노티피케이션일 때에만 해당 이름의 노티피케이션을 수신합니다.

```swift
func addObserver(forName name: NSNotification.Name?,
                 object obj: Any?,
                 queue: OperationQueue?,
                 using block: @escaping (Notification) -> Void)
                 -> NSObjectProtocol {
                   // some code
                 }
```



<br>

### ✐ removeObserver
이 메서드는 말 그대로 등록된 Observer를 제거하는 메서드입니다.

```swift
// self에 등록된 옵저버 전체 제거
notiCenter.removeObserver(self)

// 옵저버 하나 제거(등록된 특정 노티를 제거하기 위해 사용)
notiCenter.removeObserver(self, name: NSNotification.Name.UIKeyboardDidShow, object: nil)
```

#### 화면이 사라질 때 Observer를 제거해야 할까요?
[공식문서](https://developer.apple.com/documentation/foundation/notificationcenter/1413994-removeobserver)에 따르면 iOS 9.0 이후의 버전부터는 addObserver후에 화면이 사라질 때, 의무적으로 removeObserver를 해주지 않아도 자동으로 옵저버가 제거된다고 하네요.

<br>

### ✐ userInfo
이번에는 userInfo로 뷰컨트롤러 간의 데이터를 주고 받아보도록 하겠습니다. 역시 [멍구님의 블로그](https://0urtrees.tistory.com/26)의 예제를 바탕으로 테스트해보았습니다. <br>

탭바로 3개의 ViewController를 만들고, 첫번째 화면에서 버튼을 눌렀을 때 첫 화면의 프로퍼티를 받아와서, 두번째 화면의 Label에 받아온 값을 띄워주는 상황입니다.

```swift

// 첫번째 ViewController
class ViewController: UIViewController {

    var num = "123"

    override func viewDidLoad() {
        super.viewDidLoad()
    }

    @IBAction func postButton(_ sender: Any) {
        let userInfo: [AnyHashable : Any] = ["num" : num]
        NotificationCenter.default.post(name: Notification.Name(rawValue: "PostButton"), object: nil, userInfo: userInfo)
    }
}
```

```swift
// 두번째 ViewController
class FirstObserverViewController: UIViewController {

    var test: String = ""

    @IBOutlet weak var firstLabel: UILabel!

    override func viewDidLoad() {
        super.viewDidLoad()

        NotificationCenter.default.addObserver(self,
          selector: #selector(showLabel),
          name: Notification.Name(rawValue: "PostButton"),
          object: nil)
    }

    @objc func showLabel(_ noti: Notification) {
        firstLabel.text = noti.userInfo?["num"] as? String
    }
}
```

<br>

### ✐ NSNotification? Notification?
Notification을 사용하다보면, NSNotification을 사용하기도 하고, Notification을 사용하기도 합니다. 무엇을 사용해도 동작에는 아무런 차이가 없습니다. [NSNotification 공식 문서](https://developer.apple.com/documentation/foundation/nsnotification)를 보면 Notification과 NSNotification은 연결되어 있다고 합니다. 실제로 둘 중에 아무거나 사용을해도 아무 문제가 없습니다. <br>

둘은 어떤 차이가 있을까요? 우선 NSNotification은  class, Notification은 struct 입니다. NSNotification은 Swift의 구조체로된 Notification이 나오기 이전에 사용하던 개념입니다. 뭔가 Objective-C의 냄새가 나기도 하네요. NSNotification은 새로운 구조체 Notification이 등장하면서부터는 사용할 필요가 없어졌는데요, 하지만 이 둘은 브릿징으로 연결되어 있어서 섞어써도 기능상으로는 상관이 없습니다.

<br>
<br>

#### 참고
- [https://www.boostcourse.org/mo326/lecture/16919](https://www.boostcourse.org/mo326/lecture/16919)
- [https://0urtrees.tistory.com/26](https://0urtrees.tistory.com/26)
- [https://jinshine.github.io/2018/07/05/iOS/NotificationCenter/](https://jinshine.github.io/2018/07/05/iOS/NotificationCenter/)
- [https://developer.apple.com/documentation/foundation/notification](https://developer.apple.com/documentation/foundation/notification)
- [https://stackoverflow.com/questions/40710751/swift-3-nsnotification-vs-notification](https://stackoverflow.com/questions/40710751/swift-3-nsnotification-vs-notification)
