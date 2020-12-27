---
layout: post
title: iOS, 특정 화면만 방향 고정시키기
category: ios
tags: [view]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.     
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

특정 화면을 세로로, 혹은 가로로만 볼 수 있도록 고정시키려면 어떻게 해야할까요? 오늘은 화면 회전(Interface Orientation), 특정 화면을 고정시키는 방법에 대해서 짧고 굵게 포스팅합니다.

<br>
<br>

# 특정 화면의 방향 고정시키기
특정 화면의 방향을 고정시키는 방법으로는 2가지를 소개합니다. AppDelegate에서 메서드를 추가하는 방법이 있고, 해당 ViewController 자체에서 제어하는 프로퍼티가 있습니다. 먼저 기본적인 화면 회전에 대한 세팅을 어디서 하는지 살펴볼까요?

<br>

### ✐ 화면 회전 <br>

<img src  = "/assets/post-img/ios/2020-12/orientation1.jpg" width = "90%">             

화면 전환에 대한 설정은 ```Project Targets```와 ```Info.plist```에서 설정할 수 있습니다. Device Orientation에 원하는 세팅을 체크해주면 Info.plist에도 자동으로 적용됩니다.

<br>

### ✐ AppDelegate
먼저 AppDelegate의 [application(_:supportedInterfaceOrientationsFor:)](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623107-application) 메서드로 제어하는 방법입니다. <br>

이 메서드를 구현하는 것으로 App의 화면 방향을 제어할 수 있지만 만약 이 메서드를 구현하지 않을 경우 기본적으로 Targets에서, Info.plist에서 세팅을 해준 값을 따릅니다. <br>

만약 모든 화면의 방향의 조건이 예외없이 일정하다면 AppDelegate에 메서드를 구현하지 않고도 기본적인 세팅만으로 구현해줄 수 있습니다. <br>


```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
    var supportOnlyPortrait = true

       func application(_ application: UIApplication, supportedInterfaceOrientationsFor window: UIWindow?) -> UIInterfaceOrientationMask {
           if (supportOnlyPortrait == false){
            return UIInterfaceOrientationMask.allButUpsideDown
           }
           return UIInterfaceOrientationMask.portrait
       }

    ...
}
```

먼저 AppDelegate에 위의 코드를 추가하고

```swift
class ViewController: UIViewController {
    let appDelegate = UIApplication.shared.delegate as! AppDelegate

    override func viewWillAppear(_ animated: Bool) {
        appDelegate.supportOnlyPortrait = true
    }

    override func viewWillDisappear(_ animated: Bool) {
        appDelegate.supportOnlyPortrait = false
    }

    ...
}
```

그리고 세로로만 구현할 ViewController에 위의 코드를 추가해줍니다. 해당 뷰만 세로로 고정해야하기 때문에 viewWillDisappear의 시점에 설정을 해제해줍니다.

<br>

> **UIInterfaceOrientationMask (struct)**
> - portrait
> - landscapeLeft
> - landscapeRight
> - portraitUpsideDown
> - landscape
> - all
> - allButUpsideDown

<br>

이때 기본적으로 설정했던 Info.plist와 AppDelegate에서 설정해준 방향이 다르다면 어떻게 될까요? 예를 들어 Info.plist에서는 **[.portrait]** 만 설정을 해줬는데 AppDelegate에서는 **[.portrait, .landscape]** 를 설정하면 어떻게 될까요? <br>

이러한 경우에는 AppDelegate에서 설정한 값이 우선적으로 처리됩니다. Info.plist는 추후 코드로 별다른 설정이 없을 경우 적용되는 default값과 같은 느낌이네요!

<br>

이렇게 AppDelegate로도 제어를 해줄 수 있지만, **전역 변수**가 발생한다는 점에서 위험하다는 생각이 들기도 합니다. 전역 변수가 아니라 해당 ViewController 내부에서 제어를 해줄 수 있는 방법은 없을까요?

<br>
<br>

### ✐ ViewController 내부에서 제어하기
ViewController 내부에서 제어하는 방법으로는 [supportedInterfaceOrientations](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621435-supportedinterfaceorientations) 프로퍼티를 override해주는 방법이 있습니다. <br>

```swift
class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
    }

    override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
        return [.portrait, .landscapeLeft]
    }

    ...
}
```

이렇게 화면 고정을 원하는 ViewController 내부에 프로퍼티를 재정의해주기만 하면 됩니다!! 완전 간단한 방법이네요. 전역 변수를 만들어야하는 리스크도 없고요. 만약 화면 방향을 하나가 아닌 다수의 방향으로 설정해주고 싶다면 위의 코드처럼 배열을 사용하여 코드를 작성해줄 수 있습니다. <br>

이 경우 역시, Info.plist의 기본 세팅보다 우선적으로 적용됩니다 :)

<br>

#### ✐ 상위의 ViewController를 가지는 경우
하지만 해당 ViewController가 **상위의 ViewController를 가지는 경우** 주의해야합니다. 예를 들어서 NavigationController 안에 들어가있는 ViewController의 경우를 말합니다. <br>

이때에는 supportedInterfaceOrientations 프로퍼티를 재정의 해주더라도 상위의 ViewController의 세팅을 따르기 때문인데요, 그렇다면 어떻게 코드를 작성해줄 수 있을까요? NavigationController 안의  ViewController를 예로 듭니다. <br>

여기서부터는 [라자냐의 블로그](https://velog.io/@wonhee010/특정-ViewController에서-화면-회전-처리)에서 코드를 참고하여 작성합니다!
<br>

```swift
class ContainerViewController: UINavigationController {
    override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
        return self.topViewController?.supportedInterfaceOrientations ?? [.all]
        }
}
```

우선 UINavigationController를 상속받는 클래스를 만들어 NavigationController의 class로 설정해줍니다. 그리고 위에서 처럼 화면을 고정할 ViewController에 코드를 작성해줍니다. <br>

```swift
class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
    }

    override var supportedInterfaceOrientations: UIInterfaceOrientationMask {
        return [.portrait, .landscapeLeft]
    }

    ...
}
```

이렇게 하면 화면 회전에 대한 설정이 적용되는 것을 확인할 수 있습니다! <br>

이외에도 ```shouldAutorotate```, ```attemptRotationToDeviceOrientation``` 등 화면 회전에 대한 다양한 설정과 다양한 프로퍼티, 메서드가 있었는데요, 자세하게는 해당 공식문서를 읽어보는 것도 좋을 것 같네요 :) 아래에 링크 첨부합니다!




<br>
<br>
<br>

#### 참고
- [https://velog.io/@wonhee010/특정-ViewController에서-화면-회전-처리](https://velog.io/@wonhee010/특정-ViewController에서-화면-회전-처리)
- [https://developer.apple.com/documentation/uikit/uiviewcontroller/1621435-supportedinterfaceorientations](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621435-supportedinterfaceorientations)
- [https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623107-application](https://developer.apple.com/documentation/uikit/uiapplicationdelegate/1623107-application)
- [https://developer.apple.com/documentation/uikit/uiinterfaceorientationmask](https://developer.apple.com/documentation/uikit/uiinterfaceorientationmask)
- [https://developer.apple.com/documentation/uikit/uiviewcontroller/1621400-attemptrotationtodeviceorientati](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621400-attemptrotationtodeviceorientati)
- [https://developer.apple.com/documentation/uikit/uiviewcontroller/1621419-shouldautorotate](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621419-shouldautorotate)
