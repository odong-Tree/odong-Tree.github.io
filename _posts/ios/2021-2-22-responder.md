---
layout: post
title: UIResponder, responder chain
category: iOS
tags: [UIResponder, responder chain, event]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.       
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

오늘은 UIResponder와 responder chain에 대해서 포스팅해보려고 합니다. UIResponder는 실제로 앱에 발생하는 이벤트들에 대한 처리를 담당하기 때문에 AppDelegate, UIViewController, UIView 등 UIKit과 관련된 다양한 클래스에서 상속받고 있는 클래스입니다. <br>

오늘의 포스팅은 공식문서를 토대로 작성합니다. (거의 번역문에 가깝다고 할 수 있습니다!)

<br>
<br>

# UIResponder
이벤트에 응답하고 처리하기 위한 추상 인터페이스

```swift
@interface UIResponder : NSObject
```

Responder 객체 (UIResponder의 인스턴스)는 **UIKit의 중요한 뼈대를 구성하고 있습니다.** UIApplication, UIViewController, UIView 등 UIKit의 주요 클래스들은  모두 UIResponder를 상속받고 있으며 모두 Responder입니다. 그렇기 때문에 이벤트가 발생했을 때 이를  핸들링해줄 수 있습니다. <br>

인터페이스에서 발생하는 다양한 이벤트를 처리하기 위해 다음과 같은 메서드를 override하여 이용하게 됩니다. [touchesBegan:withEvent:](https://developer.apple.com/documentation/uikit/uiresponder/1621142-touchesbegan?language=occ), [touchesMoved:withEvent:](https://developer.apple.com/documentation/uikit/uiresponder/1621107-touchesmoved?language=occ), [touchesEnded:withEvent:](https://developer.apple.com/documentation/uikit/uiresponder/1621084-touchesended?language=occ), [touchesCancelled:withEvent:](https://developer.apple.com/documentation/uikit/uiresponder/1621116-touchescancelled?language=occ) <br>

Responder는 이벤트를 처리하기도 하지만 이벤트 내용을 그대로 전달하기도 합니다.  이벤트를 받은 Responder가  이벤트를 처리하지 않고 이벤트를 넘기는 것을 **responder chain** 이라고 합니다. <br>

UIKit은 responder chain을  동적으로 관리하고 다음에 이벤트를 받을 객체를 사전에 정의해둡니다. 예를 들면 view가 이벤트를 superview로 넘기고, 계층의 root view가 이벤트를 view controller로 넘기는 것입니다. <br>

Responder는 UIEvent 객체를 처리하지만 custom된 input을 받을 수도 있습니다. system keyboard 를 예로 들 수 있는데, 사용자가 UITextField나 UITextView를 탭하면 해당 view가 first responder가 되며 keyboard와 함께 입력 view가 보여집니다. custom input과 view의 responder를 연결하기 위해서는 해당 view를 responder의 inputView 프로퍼티에 할당해주어야 합니다.

<br>
-----------------
<br>

# Responder Chain to Handle Events

### OverView
앱은 responder 객체를 통해 이벤트를 받아서 핸들링합니다. responder 객체는 UIResponder 클래스의 인스턴스이며 UIView, UIViewController, UIApplication 등 UIResponder를  상속받는 클래스들이 모두  responder입니다.  <br>
responder는 raw event data를 받고, 이를 핸들링하거나 다른 responder 객체로 넘기는  일을 합니다. 만약 앱이 어떤 이벤트를 받았을 때, UIKit은 first responder라는 가장 적절한 responder 객체로 이벤트를 자동으로 넘겨줍니다. <br>

처리되지 않은 이벤트는 동적으로 앱의 responder 객체를 설정하는 responder chain으로 responder에서 responder로 옮겨집니다. 아래의 그림은 label, text field, button, 2개의 background view를  가지고 있는 앱의 인터페이스의 responder에 대한 그림입니다. 그림에서 어떻게 이벤트가 responder에서 다음으로 넘겨지는지 responder chain에 대해서도 보여주고 있네요.

<img width="736" alt="7867831D-16F7-4078-A3AF-713ED0E228EB" src="https://user-images.githubusercontent.com/73867548/108711294-7aa2e600-7558-11eb-990e-9911cd5c6a7f.png">        

출처: [Using Responders and Responder Chain to Handle Events](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events?language=occ) <br>

만약 text field가 이벤트를 핸들링하지 않으면 UIKit은 이벤트를 text field의 부모인 UIView로 보내고 그 후에 window의 root view에 보냅니다. root view롤 부터 responder chain은 이벤트를 window로 곧장 보내기 전에 해당 view를 가지고 있는 view controller로 전환합니다. (무엇을..?) 만약 window가 이벤트를 핸들링하지 않으면 UIKit은 이벤트를 UIApplication으로 전달하고 만약 객체가 UIResponder의 인스턴스이거나 responder chain의 일부가 아닌 경우에는 앱의 delegate에게 이벤트를 전달할 수 있습니다.

<br>

### Determining an Event’s First Responder
UIKit은 이벤트의 타입에 따라 first responder를 지정합니다.

![image](https://user-images.githubusercontent.com/73867548/108711528-d66d6f00-7558-11eb-879b-c597de9c417f.jpeg)     


>**Note**
accelerometers, gyroscopes, magnetometer와 관련된 Motion 이벤트는 responder chain을 따르지 않습니다. 대신에 Core Motion는 이벤트를 지정된 객체에 곧장 전달합니다. 더 자세한 정보는 [Core Motion Framework](https://developer.apple.com/documentation/#//apple_ref/doc/uid/TP40007898-CH10-SW27)를 참고

컨트롤은 액션 메세지를 통해 그들과 연관된 target object와 직접적으로 소통합니다. 사용자가  컨트롤과 상호작용할 때 컨트롤은 target object에 액션 메세지를  보냅니다. 액션 메세지는  이벤트는 아니지만 responder chain을 이용할 수 있습니다. 컨트롤의  target object가 nil일 때, UIKit은  대상 객체에서 시작해 적절한 동작 방법을 구현하는 객체를 찾을 때까지 responderr chain을 횡단(traverses)합니다. <br>


Gesture recognizer는 view보다 먼저 touch, press 이벤트를 받습니다. 만약 view의 gesture recognizer가 연속되는 터치를 인식하지 못하면 UIKit은 터치 이벤트를  view에게  넘깁니다. 만약  view도 터치 이벤트를 핸들링하지 못한다면 UIKit은 이벤트들을 responder chain으로 넘깁니다. gesture recognizer를 이용한 이벤트 핸들리에 대해서는  [Handling UIKit Gestures](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/handling_uikit_gestures?language=occ) 를 참고하세요.

<br>
<br>


### Determining Which Responder Contained a Touch Event
UIKit은 어디서 터치 이벤트가  발생했는지를 알기 위하여 view 기반의 hit-testing을 이용합니다. 구체적으로 UIKit은 view 게층에서 view의 bounds와  touch location을 비교합니다. (어디를 터치했는지 좌표값으로  알아낸다는 의미)
UIView의 [hitTest:withEvent:](https://developer.apple.com/documentation/uikit/uiview/1622469-hittest?language=occ)  메서드는 터치 이벤트의 first rresponder가 되는 가장 깊은  subview를 찾아냅니다.

>**Note**
만약  touch location이 view의 bounds에 벗어난다면, hitTest:withEvent: 메서드는 해당 view와  모든 subview를 무시합니다. 그 결과로, view의 clipsToBounds 프로퍼티가 no일 때, bound를 벗어난 subview는 터치 기능이 있어도 반환되지 않습니다. hit-testing에 대해서는 [hitTest:withEvent:](https://developer.apple.com/documentation/uikit/uiview/1622469-hittest?language=occ)
문서를 참고하세요!


터치가 발생했을 때, UIKit은 UITouch 객체를 만들고  view와 연결합니다. touch location 호은 다른  parameters가 변할 때 UIKit은 사용하던 UITouch 객체를 새로운 정보로 업데이트합니다. 유일하게 변하지 않는 프로퍼티는  view입니다. (심지어 touch location이 view를 벗어나도, view 프로퍼티는 변하지 않습니다.) touch가 끝나면 UIKit은 UITouch 객체를 해제합니다.

<br>
<br>

###  Altering the  Responder Chain

responder 객체의 [nextResponder](https://developer.apple.com/documentation/uikit/uiresponder/1621099-nextresponder?language=occ) 프로퍼티를 오버라이딩하는 것으로 responder chain을 변경할 수 있습니다. 이때 next responder는 리턴되는 객체가 됩니다.  <br>

많은 UIKit 클래스들은 이미 nextResponder를 오버라이드하고 특정 객체를 리턴하고 있습니다.
* **UIView**       
  해당 view의 superview가 next responder가 됩니다. 만약 view controller의 root view라면 next responder는 view controller가 됩니다.
* **UIViewController**       
    만약 view controller의 view가 window의 root view라면 next  responder는 window 객체가 됩니다.       
    만약 A view controller가 B view controller에 의해 나타난 경우, next responder는 B view controller가 됩니다.
* **UIWindow**      
  next  responder는 UIApplication 객체가 됩니다.
* **UIApplication**     
  만약 app delegate가 UIResponder의 인스턴스이고 view, view controller, app 객체가 아니라면 next responder는 app delegate가 됩니다. (app delegate에서도 처리해주지 않으면 이벤트는 소멸됩니다.)




<br>
<br>

#### 참고
- [UIResponder](https://developer.apple.com/documentation/uikit/uiresponder?language=occ)
- [Using Responders and Responder Chain to Handle Events](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events?language=occ)



<br>
<br>
