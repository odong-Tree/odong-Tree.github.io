---
layout: post
title: Target-Action 디자인 패턴
category: ios
tags: [target-action, UIControl]
comments: true
---

>이 글은 공부 중에 작성하는 글입니다.       
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다. <br>

오늘은 Target-Action에 대해서 알아보도록 하겠습니다. 저는 Target-Action이 생소해서 처음본다고 생각했는데 우리는 이미 이 개념을 아주 많이 사용하고 있더군요. <br>

```swift
@IBAction func someButton(_ sender: UIButton) {
    // some code
}
```

우리가 자주 사용하는 ```@IBAction```도 Target-Action입니다.

<br>

# Target-Action
Target-Action 에서 액션은 특정 이벤트가 발생했을 때 호출될 메서드를 말하고, 타겟은 액션이 호출될 객체를 말합니다. 즉, Target-Action은 어떤 **이벤트**가 발생했을 때 **타겟**이 **액션 메서드** 를 행하는 디자인 패턴입니다. 그리고  이벤트가 발생했을 때 어떤 메세지(정보)를 보낼 수 있는데 이를 액션 메세지라고 합니다. <br>

Target-Action은 ``스토리보드와 인터페이스 빌더``를 이용한 방법, 그리고 ``프로그래밍적 방법``으로 구현하는 방법이 나뉘어집니다. 먼저 **스토리보드와 인터페이스 빌더** 로 구현하는 경우에는 위에선 언급했던 @IBAction을 스토리보드와 연결해주고, 코드로 구현하는 방법입니다. 그리고 이를 스토리보드를 사용하지 않고 프로그래밍적으로 구현하는 방법이 있습니다.

<br>

### ✐ addTarget(_:action:for:)

```swift
func addTarget(_ target: Any?,
              action: Selector,
              for controlEvents: UIControl.Event)
```

```swift
let btn = UIButton(type: UIButton.ButtonType.system)

btn.addTarget(self, action: #selector(someActionMethod), for: .touchUpInside)
```

[addTarget(_:action:for:)](https://developer.apple.com/documentation/uikit/uicontrol/1618259-addtarget)은 **이벤트**가 발생했을 때 **타겟**이 **액션 메서드**입니다. 매개변수를 살펴봅시다. <br>

- **target:** action 이 호출될 객체. 호출할 대상 메서드가 있는 인스턴스. 같은 뷰 컨트롤러 내에 액션 메서드가 작성되어 있으면 self를 인자값으로 전달하고, 다른 뷰 컨트롤러에 속한 메서드를 호출해야 한다면 해당 뷰 컨트롤러의 인스턴스를 인자값으로 넣어줍니다.
- **action:** 호출할 메서드를 지정하는 매개변수. for에서  지정한  조건(이벤트)이 실행될 때, action에 등록한 메서드를 실행시킵니다. 매개변수의 타입은 Selector이며 실사용에서는 #selector(액션메서드) 형태로 사용할 수 있습니다.
- **for:** 액션 메서드의 실행 조건을 지정하는 매개변수입니다. UIControl.Event에 대해서는 아래에서 자세하게 다루어도록 하겠습니다. <br>

>addTarget으로 등록된 TargetAction은 [removeTarget(_:action:for:)](https://developer.apple.com/documentation/uikit/uicontrol/1618248-removetarget)으로 제거해줄 수도 있습니다.

<br>

바로 예제 코드를 써보면 이해가 쉽습니다. 코드는 **꼼꼼한 재은씨의 Swift - 실전편**을 참고합니다. <br>

```swift

import UIKit

class NewViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        // 버튼 객체를 생성하고, 속성을 설정한다.
        let btn = UIButton(type: UIButton.ButtonType.system)
        btn.frame = CGRect(x: 50, y: 100, width: 150, height: 30)
        btn.setTitle("Test Button", for: UIControl.State.normal)

        // 버튼을 수평 중앙 정렬한다.
        btn.center  = CGPoint(x: self.view.frame.size.width / 2, y: 100)

        // 루트 뷰에 버튼을 추가한다.
        self.view.addSubview(btn)

        // 버튼의 이벤트와 메서드 btnOnClick을 연결한다.
        btn.addTarget(self, action: #selector(btnOnClick(_: )), for: .touchUpInside)
    }

    @objc func btnOnClick(_ sender: Any) {
        // 호출한 객체가 버튼이라면
        if let btn = sender as? UIButton {
            btn.setTitle("Clicked", for: UIControl.State.normal)
        }
    }
}
```

<br>

### ✐ Selector, #selector
'#selector'는 특이하게 생겨서 눈에 띕니다. 함수의 매개변수로 다른 함수를 사용하기 위해 사용하는 키워드입니다. Selector는 원래 Objective-C에서 클래스 메서드의 이름을 가리키는데 사용되는 참조 타입인데요, 원래는 @selector(메서드이름)으로 사용하다가 Swift로 넘어오면서 String 타입으로 "메서드이름"으로 표기하다가 Swift 2.2 이후로  #selector(메서드이름) 형태를 띄게 되었다고 하네요. <br>

Selector 타입으로 전달할 메서드를 작성하기 위해서는 반드시 **@objc** 어트리뷰트가 필요합니다. 이는 Objective-C와의 호환성을 위한 것으로, 스위프트에서 정의한 메서드를 Objective-C에서도 인식할 수 있게 해줍니다.


<br>

### ✐ Action Method
매개변수 action에 들어가는 Selector에는 **액션 메서드**가 들어가게 되는데요, 액션 메서드에는 @objc / @IBAction 등과 같은 특정한 양식이 필요합니다.

```swift
// 프로그래밍 방식
@objc func doSomething(_ sender: Any) {

}

// 인터페이스 빌더
@IBAction func doSomething(_ sender: Any) {

}
```

@IBAction은 스토리보드와 연결하기 위한 키워드로 알고 있었는데, 스토리보드와 연결하지 않고도 @objc를 대신해서 사용할 수도 있더군요.

```swift
btn.addTarget(self, action: #selector(btnOnClick(_: )), for: .touchUpInside)

@IBAction func btnOnClick(_ sender: Any) {
if let btn = sender as? UIButton {
    btn.setTitle("Clicked", for: UIControl.State.normal)
}
```

##### sender
액션메서드는 반드시 sender라는 하나의 매개변수를 가져야하며, Any, AnyObject 또는 객체의 타입으로 선언되어야 합니다. 이는 호출한 객체의 정보를 인자값으로 전달하기 위함입니다. sender의 속성값에 쉽게 접근할 수 있다는 이야기입니다. <br>

sender의 타입을 Any, AnyObject로도 해줄 수 있지만 UIButton처럼 특정할 수도 있습니다. 하지만 이때에는 이 메서드를 발생시킬 수 있는 객체가 UIButton으로 한정된다는 점에 유의해야 합니다. 하나의 액션 메서드를  여러 객체가 호출해야 한다면 Any 혹은 AnyObject로 선언하는 것이 유리합니다.

<br>

### ✐ UIControl.Event
addTarget 메서드의 마지막 매개변수 for에 들어오는 타입입니다. 이는 action이 일어나기 위한 **실행 조건**이기 때문에 역시 매우 중요합니다. 이에 대해서는 [부스트코스](https://www.boostcourse.org/mo326/lecture/16854)에 매우 잘 설명이 되어 있어서 내용을 그대로 가져오는 부분입니다. (추가로 현재를 기준으로, 부스트코스에서 사용된 `UIControlEvents.Events` 코드는 `UIControl.Event.Events`로 변경되어서 수정하여 포스팅합니다.)<br>

#### 컨트롤 이벤트와 액션과의 관계
UIKit에는 `UIButton`, `UISwitch`, `UIStepper` 등 `UIControl`을 상속받은 다양한 컨트롤 클래스가 있습니다. 그런 컨트롤 객체에 발생한 다양한 이벤트 종류를 특정 액션 메서드에 연결할 수 있습니다. 즉, 컨트롤 객체에서 특정 이벤트가 발생하면, 미리 지정해 둔 타겟의 액션을 호출하게 됩니다. <br>

#### 컨트롤 이벤트의 종류
컨트롤 이벤트는 `UIControl.Event`라는 타입으로 정의되어 있습니다. 아래는 컨트롤 객체에 발생할 수 있는 이벤트의 종류입니다.
- touchDown
  - 컨트롤을 터치했을 때 발생하는 이벤트
  - UIControl.Event.touchDown
- touchDownRepeat
  - 컨트롤을 연속 터치 할 때 발생하는 이벤트
  - UIControl.Event.touchDownRepeat
- touchDragInside
  - 컨트롤 범위 내에서 터치한 영역을 드래그 할 때 발생하는 이벤트
  - UIControl.Event.touchDragInside
- touchDragOutside
  - 터치 영역이 컨트롤의 바깥쪽에서 드래그 할 때 발생하는 이벤트
  - UIControl.Event.touchDragOutside
- touchDragEnter
  - 터치 영역이 컨트롤의 일정 영역 바깥쪽으로 나갔다가 다시 들어왔을 때 발생하는 이벤트
  - UIControl.Event.touchDragEnter
- touchDragExit
  - 터치 영역이 컨트롤의 일정 영역 바깥쪽으로 나갔을 때 발생하는 이벤트
  - UIControl.Event.touchDragExit
- touchUpInside
  - 컨트롤 영역 안쪽에서 터치 후 뗐을때 발생하는 이벤트
  - UIControl.Event.touchUpInside
- touchUpOutside
  - 컨트롤 영역 안쪽에서 터치 후 컨트롤 밖에서 뗐을때 이벤트
  - UIControl.Event.touchUpOutside
- touchCancel
  - 터치를 취소하는 이벤트 (touchUp 이벤트가 발생되지 않음)
  - UIControl.Event.touchCancel
- valueChanged
  - 터치를 드래그 및 다른 방법으로 조작하여 값이 변경되었을때 발생하는 이벤트
  - UIControl.Event.valueChanged
- primaryActionTriggered
  - 버튼이 눌릴때 발생하는 이벤트 (iOS보다는 tvOS에서 사용)
  - UIControl.Event.primaryActionTriggered
- editingDidBegin
  - `UITextField`에서 편집이 시작될 때 호출되는 이벤트
  - UIControl.Event.editingDidBegin
- editingChanged
  - `UITextField`에서 값이 바뀔 때마다 호출되는 이벤트
  - UIControl.Event.editingChanged
- editingDidEnd
  - `UITextField`에서 외부객체와의 상호작용으로 인해 편집이 종료되었을 때 발생하는 이벤트
  - UIControl.Event.editingDidEnd
- editingDidEndOnExit
  - `UITextField`의 편집상태에서 키보드의 `return` 키를 터치했을 때 발생하는 이벤트
  - UIControl.Event.editingDidEndOnExit
- allTouchEvents
  - 모든 터치 이벤트
  - UIControl.Event.allTouchEvents
- allEditingEvents
  - `UITextField`에서 편집작업의 이벤트
  - UIControl.Event.allEditingEvents
- applicationReserved
  - 각각의 애플리케이션에서 프로그래머가 임의로 지정할 수 있는 이벤트 값의 범위
  - UIControl.Event.applicationReserved
- systemReserved
  - 프레임워크 내에서 사용하는 예약된 이벤트 값의 범위
  - UIControl.Event.systemReserved
- allEvents
  - 시스템 이벤트를 포함한 모든 이벤트
  - UIControl.Event.allEvents

<br>

# Target-Action, 왜 사용할까?

Target-Action은 이벤트가 발생했을 때 메서드를 실행시킬 객체(Target)을 특정할 수 있다는 점에서 유리합니다. 아래는 부스트코스에서 소개된 예시입니다.<br>

>직접 타겟이 될 객체에 액션에 해당하는 메서드를 호출하면 될 텐데 굳이 Target과 Action을 지정하고 디자인 패턴으로 활용하는 이유가 궁금하죠? 왜 그럴까요? <br>
>만약 특정 이벤트가 발생했을 때 abc라는 이름의 메서드를 호출해야 하는 상황이라고 생각해 봅시다. 그런데 이 abc라는 (액션)메서드는 A라는 클래스에도 정의되어 있고, B라는 클래스에도 정의되어 있는 경우가 있습니다. 이렇게 같은 메서드가 여러 클래스에 정의되어 있는 경우도 있고, 그런 클래스의 인스턴스가 여러개인 상황도 있습니다. 이런 여러 가지 상황에서 우리가 원하는 객체를 Target으로 지정하면 abc라는 액션을 실행할 객체를 상황에 따라서 선택할 수 있습니다. - 부스트코스







<br>
<br>

#### 참고
- [https://www.boostcourse.org/mo326/lecture/16854](https://www.boostcourse.org/mo326/lecture/16854)
- [https://developer.apple.com/documentation/uikit/uicontrol/1618259-addtarget](https://developer.apple.com/documentation/uikit/uicontrol/1618259-addtarget)
- [https://daheenallwhite.github.io/ios/2019/07/24/Target-Action/](https://daheenallwhite.github.io/ios/2019/07/24/Target-Action/)
- [https://developer.apple.com/documentation/uikit/uicontrol](https://developer.apple.com/documentation/uikit/uicontrol)

<br>
<br>
