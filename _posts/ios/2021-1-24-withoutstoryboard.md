---
layout: post
title: Storyboard 없이 화면 만들기(초기 세팅하기)
category: iOS
tags: [Storyboard]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.       
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

오늘은 스토리보드없이 **only code**로만 화면을 구성할때 **초기 세팅 방법** 에 대해서 알아보도록 하겠습니다. 간단한 포스팅이 되겠네요. 실제로 구성요소들을 이용하여 **레이아웃을 짜는 방법에 대한 포스팅은 아닙니다!** (이 주제에 대해서 포스팅하게되면 링크를 남겨놓도록 하겠습니다.)

<br>
<br>

# Storyboard가 없는 환경 세팅하기


### 1. 불필요한 세팅 및 파일 제거하기
![withoutstoryboard2](https://user-images.githubusercontent.com/73867548/105609710-6e5b2a00-5dee-11eb-9b9b-24f86698c391.jpg)

- 기존의 **storyboard** 파일 (물론 지우지 않고 아래의 2가지 항목만 삭제해주어도 괜찮습니다.)
- 프로젝트 Target의  **Main Interface** 항목을 지워줍니다.
- Info.plist에서 **Stroyboard Name**와 **Main storyboard file base name**항목을 제거합니다.

<br>

### 2. SceneDelegate, AppDelegate에 코드 작성
다음으로 SceneDelegate, AppDelegate에 코드를 작성합니다. 혹시 SceneDelegate, AppDelegate에 대해 궁금하시다면 [이 포스팅](https://odong-tree.github.io/ios/2021/01/23/appdelegate_scenedelegate/)을 읽어보시면 좋을 것 같네요:) 간단하게 언급하면 **SceneDelegate는 iOS 13 이상부터 지원하기 때문에** 이 점에 주의해서 코드를 작성해주어야 합니다.

#### - SceneDelegate

```swift
@available(iOS 13.0, *)
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        // Use this method to optionally configure and attach the UIWindow `window` to the provided UIWindowScene `scene`.
        // If using a storyboard, the `window` property will automatically be initialized and attached to the scene.
        // This delegate does not imply the connecting scene or session are new (see `application:configurationForConnectingSceneSession` instead).
        guard let windowScene = (scene as? UIWindowScene) else { return }

        window = UIWindow(windowScene: windowScene)
        let rootViewController = ViewController()
        window?.rootViewController = rootViewController
        window?.makeKeyAndVisible()
    }
```

SceneDelegate 같은 경우 iOS 13.0 이상에서만 동작하므로 window 객체를 만들고 rootViewController만 설정해주면 됩니다.

```swift
window = UIWindow(windowScene: windowScene)
// window를 초기화합니다.

let rootViewController = ViewController()
window?.rootViewController = rootViewController
// 처음 시작될 rootViewController를 설정해줍니다.
// UIViewController를 상속받는 객체만 올 수 있습니다.

window?.makeKeyAndVisible()
// window를 보여주고(show), same level의 어떤 window보다 앞에 위치하게 됩니다.
```

<br>

### - AppDelegate

```swift
@main
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?

    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        // Override point for customization after application launch.
        if #available(iOS 13.0, *) {
            // empty
        } else {
            window = UIWindow()
            let rootViewController = ViewController()
            window?.rootViewController = rootViewController
            window?.makeKeyAndVisible()
        }

        return true
    }
```

AppDelegate도 SceneDelegate와 방법은 똑같지만 window 프로퍼티를 직접 선언해주어야 하고, **iOS 버전이 13.0보다 낮은 경우에만 실행되도록 분기 처리**를 해주어야 합니다.


<br>
<br>

#### 참고
- [https://developer.apple.com/documentation/uikit/uiwindow](https://developer.apple.com/documentation/uikit/uiwindow)
- [https://icksw.tistory.com/25](https://icksw.tistory.com/25)

<br>
<br>
