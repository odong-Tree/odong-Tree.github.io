---
layout: post
title: AppDelegate와 SceneDelegate
category: iOS
tags: [AppDelegate, SceneDelegate]
comments: true
---

>이 글은 공부 중에 작성하는 글입니다.       
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

원래는 Application Life Cycle에 대해서 포스팅하려고 했으나.. 예상과는 다르게 봐야하는 내용들이 좀 있더군요. 그래서 Application Life Cycle에 앞서 AppDelegate와 SceneDelegate에 대해서 알아보도록 하겠습니다.

![scene1](/assets/post-img/ios/scene1.jpg)

<br>
<br>




# SceneDelegate
먼저 SceneDelegate를 살펴보도록 하겠습니다. SceneDelegate는 **iOS 13 이후** 로 등장한 개념입니다. 원래는 AppDelegate에서 SceneDelegate의 역할까지 함께 담당했습니다. 그렇다면 SceneDelegate는 어떤 역할을 하고 있을까요?
<br>



![scene2](/assets/post-img/ios/scene2.jpg)

![scene3](/assets/post-img/ios/scene3.jpg)

[WWDC](https://developer.apple.com/videos/play/wwdc2019/258/)에서 가져온 사진입니다. <br>

iOS 13 버전을 기점으로 원래는 AppDelegate에서 담당하던 **UI Lifecycle를 이제는 SceneDelegate가 담당** 하게 된 것입니다. 역할이 나누어졌다고 할까요? 그리고 AppDelegate는 **Session Lifecycle** 을 관리하게 되었는데요, Session에 대해서는 아래에서 한 번 더 언급하도록 하겠습니다.

<br>

### ✐ Scene
SceneDelegate가 등장하게 된 배경에는 **Scene** 이라는 새로운 개념이 있습니다. 명확한 이유는 잘 모르겠습니다만 Scene은 **MultiWindow**를 지원하기 위해서 등장했다는 이야기가 많은 것 같네요. 그리고 현재 MultiWindow 기능은 아이패드에서만 지원이 되는 기능입니다.

>##### 멀티 윈도우(MultiWindow)
> 저는 처음에 멀티 윈도우와 멀티태스킹의 개념이 헷갈리더군요. 아이패드의 한 화면에서 서로 다른 앱을 동시에 사용할 수 있습니다. 이때, 서로  다른 앱을 동시에 사용하는 것은 **멀티태스킹**, 한 화면에서 동일한 앱을 두 개  띄워서 작업하는 것을 **멀티윈도우**라고 합니다. 같은 앱을 동시에 실행할  수 있다는 것이 핵심입니다.

그렇다면 Scene과 Window는 어떤 관계일까요? iOS 13 이후부터는 **window 개념이 scene으로 대체**되어 사용되고 있습니다. 하지만 (당연히) 둘은 같은 개념이기 보다는, 엄밀히 말하면 scene이 더 넓은 개념이라고 할 수 있을 것 같네요.



<br>

#### [✐ Scenes 공식문서](https://developer.apple.com/documentation/uikit/app_and_environment/scenes)
Scene은 UIWindowScene을 사용하여 앱의 **UI 인스턴스를 관리**하는 역할을 합니다. UIWindowScene는 하나 이상의 window가 사용되는 Scene, 즉 UI 인스턴스를 관리하는 클래스로 UIScene - UIResponder를 차례로 상속받고 있습니다. <br>

하나의 Scene은 하나의 UI 인스턴스를 나타내기 위한 windows와  viewcontrollers를 포함하고 있습니다. 각 Scene은 UIKit과 앱의 상호작용을 담당하는 UIWindowSceneDelegate를 가지고 있습니다. Scene은 같은 메모리와 앱 process space를 공유하며, 여러개의 Scene이 동시에 실행될 수 있습니다. 그렇기 때문에 하나의 앱은 여러개의  Scene을 가질 수 있고 scene delegate 객체들은 동시에 동작할 수 있습니다.

 <br>

### ✐ SceneDelegate의 역할

![scene4](/assets/post-img/ios/scene4.jpg)

SceneDelegate는 **UI Lifecycle을 관리**하는 역할을 한다고 했습니다. 즉 원래 AppDelegate에서 하던 메서드들이 SceneDelegate로 넘어오면서 메서드 이름 역시 scene ~ 으로 변경되었네요. WWDC의 사진으로도 볼 수 있듯이 SceneDelegate는 **UISceneDelegate 프로토콜**을 채택하고 있습니다. 그리고  이 프로토콜은 유저가 보는 앱의 인터페이스(UI)의 life cycle을 관리합니다. <br>

프로젝트에서 기본적으로 제공되는 메서드들은 모두 UISceneDelegate 프로토콜의 메서드입니다.


```swift
func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
  // 이 메서드를 사용하여 UIWindow 'window'를 제공된 UIWindowScene 'scene'에 선택적으로 구성하고 연결합니다.
  // 만약 스토리보드를 사용한다면, window 프로퍼티는 자동으로 초기화되고 scene에 연결됩니다.
  // 해당 delegate는 연결된 scene 혹은 새로운 세션임을 의미하지는 않습니다. (이 문장은 무슨 말인지 잘 모르겠습니다)
}

func sceneDidDisconnect(_ scene: UIScene) {
  // scene이 시스템에 의해 해제되었을 때 호출됩니다.
  // scene이 백그라운드로 넘어갈 때, 또는 세션이 삭제되었을때 곧장 동작합니다.
  // 다음에 scene이 연결될 때, 재생성될 수 있는 해당 scene과 관련된 resources를 해제합니다.
  // 세션이 반드시 삭제되는 것이 아니므로 해당 scene은 다시 연결될 수 있습니다.
}

func sceneDidBecomeActive(_ scene: UIScene) {
  // scene이 inactive 상태에서 active 상태가 된 직후에 호출됩니다.
  // 이 메서드는 scene이 inactive 상태가 되었을 때 중단되었던  작업을 재시작할 때 사용합니다.
}

func sceneWillResignActive(_ scene: UIScene) {
  // scene이 active 상태에서 inactive 상태가 되기 직전에 호출됩니다.
  // 전화를 수신할 때처럼 일시적인 중단이 발생합니다.
}

func sceneWillEnterForeground(_ scene: UIScene) {
  // scene이 background에서 foreground로 이동하기 직전에 호출됩니다.
  // background로 이동할때 발생한 변화를 원상태로 돌릴 떄(undo) 사용합니다.

}

func sceneDidEnterBackground(_ scene: UIScene) {
  // scene이 foreground에서 background로 이동한 직후에 호출됩니다.
  // data를 저장하거나, 공유된 resources를 해제하거나, scene의 상태변화를 저장할 때 사용합니다. (scene을 현재 상태로 복원하기 위해서)
}
```

<br>

그런데.. 실제로 새 프로젝트를 생성해보면,

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?
```

SceneDelegate는 **UIWindowSceneDelegate**를 채택하고 있습니다. 이 UIWindowSceneDelegate는 UISceneDelegate 프로토콜을 따르고 있습니다. 따라서 **UISceneDelegate 프로토콜**을 채택하고 있다는 것과 같다고 할 수 있습니다 :) 하지만 실제로 UIWindowSceneDelegate 대신 UISceneDelegate를 채택해서 실행해보면 앱의 화면이 나오지는 않더군요.

<br>
<br>

# AppDelegate
다음으로 AppDelegate입니다. iOS 12 이전에는 주로 앱의 상태를 업데이트할 때 즉 앱의 Lifecycle을 처리하곤 했는데요, 하지만 iOS 13 이후로 SceneDelegate에게 그 역할을 양보하게 되었습니다. 그렇다면 iOS 13 이후의 AppDelegate는 어떤 일을 수행하고 있을까요? <br>

### ✐ UIApplicationDelegate
[공식문서](https://developer.apple.com/documentation/uikit/uiapplicationdelegate)의 내용을 가져왔습니다. <br>

1. 앱의 central data 구조체를 초기화합니다.
2. 앱의 scene들을 구성(configuration)합니다.
3. 메모리 부족 경고, 다운로드 완료와 같은 앱의 외부에서 발생한 알림들에 대응합니다.
4. 특정한 sences, views, viewcontrollers에 한정되지 않고 앱 자체를 target하는 이벤트에 대응합니다.
5. 애플 푸쉬 알림 서비스와 같은 앱이 시작될 때 요구되는 모든 서비스를 등록합니다.

<br>

그러면 프로젝트를 새로 만들었을 때 제공되는 코드를 살펴볼까요?

```swift
@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
      // 앱이 시작한 후 사용자에 맞도록 point를 재정의합니다.
        return true
    }

    // MARK: UISceneSession Lifecycle

    func application(_ application: UIApplication, configurationForConnecting connectingSceneSession: UISceneSession, options: UIScene.ConnectionOptions) -> UISceneConfiguration {
      // 새로운 scene 세션이 생성될때 호출됩니다.
      // 이 메서드를 사용하여 새로만들 scene의 구성(configuration)을 선택합니다.
        return UISceneConfiguration(name: "Default Configuration", sessionRole: connectingSceneSession.role)
    }

    func application(_ application: UIApplication, didDiscardSceneSessions sceneSessions: Set<UISceneSession>) {
      // scene session이 제거되었을 때 호출됩니다.
      // 만약 어떤 세션들이 앱이 동작하지 않을때 제거되었다면 이 메서드는 application:didFinishLaunchingWithOptions가 호출된 직후에 호출됩니다.
      // 이 메서드를 사용하여 제거된 scene과 관련된 모든 resources는 반환되지 않으므로 해제합니다.
    }
}
```

여기서 살펴보아야할 것이 2가지가 있습니다. **@main** 과 **SceneSession** 입니다.
<br>

### ✐ @main
먼저 **@main** 을 먼저 살펴보도록 하겠습니다. 개발을 하면서 건들지만 않으면 굳이 다루어주지 않아도 될 코드지만.. 뭔지 궁금하군요. @main은 Swift 5.3 이후에 등장한 이름입니다. 이 어노테이션은 이전에는 **@UIApplicationMain** 였습니다. 지금도 @main 대신 사용해보니 문제없이 동작하는군요. 공식문서에서는 제가 @main을 찾지 못해서.. [@UIApplicationMain](https://developer.apple.com/documentation/uikit/1622933-uiapplicationmain?language=occ)으로 대신 이해해보았습니다. <br>

```swift
int UIApplicationMain(int argc,
  char * _Nullable *argv,
  NSString *principalClassName,
  NSString *delegateClassName);
```

뭔가 Swift 보다는 Objective-C의 냄새가 강하게 나는 코드네요. **UIApplicationMain** 메서드는 application 객체를 주요한 클래스에서 인스턴스화하고, 주어진 클래스에서  delegate를 인스턴스화 및 세팅합니다. 말이 좀 복잡한데 **앱의 큰 뼈대를 인스턴스화** 한다는 것으로 이해가 되네요.  <br>

그러니까 앱이 실행될 때 그 무엇보다도 먼저 실행이되는 메서드인 것 같습니다. application 객체와 delegate를 만들면, event loop와 run loop를 설정하여 이벤트 처리를 시작합니다. 만약 Info.plist가 main nib file을 지정하고 있다면 해당
nib 파일을 로드합니다. return 타입으로 정의된 메서드지만 절대 return 하지 않는 메서드라고 하네요. 아주아주 중요하고 특별해 보이는 군요 :-) <br>

설명이 부족했다면 추후에 포스팅할 App의 Lifecycle에서 한 번 더 다루어보도록 하겠습니다.

<br>

### ✐ SceneSession
이 글의 초반부에서 iOS 13 이후, AppDelegate는 Session Lifecycle을 관리한다고 언급했습니다. 위의 코드를 보면 아래의 두 메서드는 모두 Session Lifecycle에 대한 메서드네요. SceneSession에 대한 내용도 [공식문서](https://developer.apple.com/documentation/uikit/uiscenesession)를 참고하여 작성합니다. <br>

```swift
class UISceneSession : NSObject
```

**UISceneSession은 고유한 런타임을 갖는 scene들을 관리하는 객체입니다.** 사용자가 새로운 scene을 앱에 추가할때, 혹은 하나를 프로그래밍적으로 요청할때 시스템은 그 scene을 추적하기 위해 하나의 **세션(session)을 만듭니다.** 세션은 고유한 식별자와 scene의 구체적인 configuration을 가지고 있습니다. UIKit은 scene의 자체적인 lifetime 동안 세션 정보를 가지고 있다가, 사용자가 앱의 scene을 닫으면 세션을 파괴합니다. <br>

세션은 직접적으로 만들 수 없고, UIKit이 사용자의 동작에 반응하여 생성하게 됩니다. 하지만 UIApplication에서 **requestSceneSessionActivation(_:userActivity:options:errorHandler:)** 메서드를 호출하는 것으로 임의로 scene과 세션을 만들어줄 수도 있다고 합니다. (확실히 AppDelegate 의 역할이 맞네요.) 그리고 세션 초기화 설정은 Info.plist에서  해줄 수 있습니다.

<br>

--------------
<br>

포스팅이 조금 길어졌네요. 추후에 App의 Lifecycle에 대해서도 다루어보도록  하겠습니다.



<br>
<br>

#### 참고
- [Scenes](https://developer.apple.com/documentation/uikit/app_and_environment/scenes)
- [UIWindowScene](https://developer.apple.com/documentation/uikit/uiwindowscene)
- [UISceneDelegate](https://developer.apple.com/documentation/uikit/uiscenedelegate)
- [UIWindowSceneDelegate](https://developer.apple.com/documentation/uikit/uiwindowscenedelegate)
- [UIApplicationDelegate](https://developer.apple.com/documentation/uikit/uiapplicationdelegate)
- [UIApplicationMain](https://developer.apple.com/documentation/uikit/1622933-uiapplicationmain?language=occ)
- [UISceneSession](https://developer.apple.com/documentation/uikit/uiscenesession)
- [https://velog.io/@dev-lena/iOS-AppDelegate와-SceneDelegate](https://velog.io/@dev-lena/iOS-AppDelegate와-SceneDelegate)
- [https://usinuniverse.bitbucket.io/blog/scenedelegatepart1.html](https://usinuniverse.bitbucket.io/blog/scenedelegatepart1.html)
