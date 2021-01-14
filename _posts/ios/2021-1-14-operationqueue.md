---
layout: post
title: OperationQueue 2편
category: Swift
tags: [OperationQueue]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.       
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

이번 포스팅에서는 지난 [OperationQueue 1편, Operation](https://odong-tree.github.io/ios/2021/01/13/operation/)에 이어서, OperationQueue에 대해서 알아보고 코드로 작성해보도록 하겠습니다.

<br>
<br>

# OperationQueue
OperationQueue는 Operation으로 구성된 대기열이며 Operation의 연산과 실행을 관리합니다. 즉 Operation을 OperationQueue에 넣는 것으로 작업이 처리되고 관리되는 것입니다. <br>

Operation이 Queue에 추가되면 작업이 끝날 때까지 대기열에 남아있게 되며 직접적으로 Operation을 제거하는 것은 불가능 합니다. 대신 작업을 취소하는 방법, `Operation.cancel(), OperationQueue.cancelAllOperations()`으로 Operation을 제거할 수 있습니다. 만약 Operation을  Suspending하게 되면 **메모리 누수**가 발생할 수 있습니다.

<br>

### ✐ OperationQueue 작업 순서
Operation이 OperationQueue에 들어오면 처리되는 작업 순서가 정해집니다. (DispatchQueue처럼 FIFO가 아닙니다.) 작업 순서는 Operation에서 공부했듯, Queue에 들어오는 순서, 우선순위(Priority), 종속성(Dependency), addExecutionBlock 등을 복합적으로 고려하여 결정됩니다. 만약 모든 조건이 같은 상태라면 OperationQueue에 들어온 순서대로 수행됩니다. <br>

하지만 Operation이 ready 상태가 되는 시점, 우선순위(Priority), 종속성(Dependency), addExecutionBlock  등에 따라 **복합적으로 실행 순서가 정해진다는 점**에 대해서는 항상 주의해야합니다. (**FIFO가 아닙니다!**)

<br>

### ✐ Canceling Operation
위에서 언급한대로 Operation의 수행을 막고싶다면 `Operation.cancel(), OperationQueue.cancelAllOperations()`을 사용하여 Operation을 isCanceled 상태로 만들 수 있습니다. 이때 작업을 cancel해도  즉시 중단되는 것은 아닙니다. 예를 들어  아직 start() 메서드가 호출되지 않은 Operation을 cancel한다면, start() 메서드는 작업을 시작하지 않고 종료됩니다.
#### 라고 하는데... cancel에 대해서는 이해가 잘 안되는 부분들이 있어서 몇 가지 테스트 후 포스팅 링크를 남겨두도록 하겠습니다 :)

<br>

### ✐ KVO, KVC 준수
OperationQueue 클래스는 KVC(key-value coding), KVO(key-value observinng)을 준수합니다. 프로퍼티를 추적하고, 외부에서 접근할 수 있다는 이야기입니다. 아래와 같은 프로퍼티들을 추적할 수 있습니다.

```swift
var operations: [Operation] { get }
var operationCount: Int { get }
var maxConcurrentOperationCount: Int { get set }
var isSuspended: Bool { get set }
var name: String? { get set }
```

[[=> KVO와 KVC 포스팅 보러가기 <=]](https://odong-tree.github.io/ios/2020/12/28/kvo_kvc/)

<br>

### ✐ Thread-safe
여러 스레드로부터 단일(single) OperationQueue를 사용하는 것은 **Thread-safe**합니다. Thread-safe하다는 것은 여러 스레드에서 값에 접근할 때 값이 온전하거나 안전한 것을 말합니다. 객체 지향에서는 함수형 프로그래밍과 다르게 상태값을 가지기 때문에 동시에 접근하는 경우 문제(race condition)가 발생할 수 있습니다. <br>

단일 Operation이라는 말이 이해가 안됐는데 아래와 같이 테스트해보니 addExecutionBlock을 사용하여 같은 프로퍼티에 접근하면 race condition이 발생하더군요.

```swift
var array = [1, 2, 3, 4, 5, 6, 7, 8, 9]

let someBlock = BlockOperation {
    let num = array.removeFirst()
    print(num)
}

someBlock.addExecutionBlock {
    let num = array.removeFirst()
    print(num)
}

someBlock.addExecutionBlock {
    let num = array.removeFirst()
    print(num)
}

someBlock.addExecutionBlock {
    let num = array.removeFirst()
    print(num)
}

let queue = OperationQueue()
queue.addOperation(someBlock)

/* 출력
1
1
1
1
*/
```

##### 대신 아래의 테스트에서는 Thread-safe 했습니다.

```swift
var array = [1, 2, 3, 4, 5, 6, 7, 8, 9]

let someBlock = BlockOperation {
    let num = array.removeFirst()
    print(num)
}

let someBlock2 = BlockOperation {
    let num = array.removeFirst()
    print(num)
}

let someBlock3 = BlockOperation {
    let num = array.removeFirst()
    print(num)
}

let someBlock4 = BlockOperation {
    let num = array.removeFirst()
    print(num)
}

let queue = OperationQueue()
queue.maxConcurrentOperationCount = 1
queue.addOperations([someBlock, someBlock2, someBlock3, someBlock4], waitUntilFinished: true)

/* 출력
1
2
3
4
*/
```

<br>
<br>

# OperationQueue 사용하기
Operation 포스팅에서 살짝 맛본 OperationQueue와 위에서 예시로 들었던 코드를 보셨다면 OperationQueue가 어떻게 사용되는지 살짝 감이 오실 것 같습니다. 차근차근 살펴보도록 하겠습니다.

<br>

### ✐ name, main, current

```swift
var name: String?
class var main: OperationQueue
class var current: OperationQueue?
```

current는 현재 작업을 시작한 Operation Queue를 반환합니다. 대기열을 확인할 수없는 경우 nil이 될 수 있기 때문에 옵셔널 타입으로 정의되어 있습니다. main은 메인 스레드와 연결된 Operation Queue를 반환합니다. 메인 스레드에서 작업해야할 때 사용하는 프로퍼티입니다. <br>

그리고 OperationQueue  인스턴스는 아래와 같이 만들 수 있습니다.

```swift
let queue = OperationQueue()
```

<br>

### ✐ addOperation

```swift
func addOperation(_ op: Operation)
func addOperations(_ ops: [Operation], waitUntilFinished wait: Bool)
func addOperation(_ block: @escaping () -> Void)
```

addOperation은 OperationQueue에 Operation을 추가하는 메서드입니다. 추가되는 동시에 Operation의 작업이 수행됩니다. 작업을 시작하는 메서드라고 봐도 무방할 것 같네요.

```swift
let someOperation = BlockOperation {
    print("아낌없이 주는 오동나무")
}

let queue = OperationQueue()
queue.addOperation(someOperation)

// 아낌없이 주는 오동나무
```

addOperation에는 BlockOperation 객체를 직접 넣을 수도, 배열로서 넣을 수도, 혹은 클로저를 추가해줄 수도 있습니다.

<br>

### ✐ max, wait, cancel 관련 코드

```swift
var maxConcurrentOperationCount: Int
func waitUntilAllOperationsAreFinished()
func cancelAllOperations()
```

이름만 봐도 어떤 역할을 하는 코드인지 감이 올 것 같네요. maxConcurrentOperationCount는 한 번에 최대로 수행되는 작업의 갯수입니다. 어떠한 설정이 없는 OperationQueue에 Operation이 들어오게 되면 기본적으로 비동기적으로 작업이 진행됩니다. 하지만 이때 maxConcurrentOperationCount를 설정해주면 작업 순서에 맞게 동시에 작업될 수 있는 Operation의 수를 제한해줄 수 있습니다.

```swift
let someOperation = BlockOperation {
    sleep(2)
    print("아낌없이 주는 오동나무, 2초 경과")
}

let someOperation2 = BlockOperation {
    sleep(2)
    print("odongnamu, 2초 경과")
}



let queue = OperationQueue()
queue.maxConcurrentOperationCount = 1
queue.addOperation(someOperation)
queue.addOperation(someOperation2)

queue.waitUntilAllOperationsAreFinished()
print("Finish!")

/*
아낌없이 주는 오동나무, 2초 경과
odongnamu, 2초 경과
Finish!
*/
```

Operation에서 비슷한 개념들을 다루어 보아서 이해하는게 어렵지는 않은 것 같습니다. 하지만 cancelAllOperations() 메서드는 이해하는데에 시간이 좀 걸렸네요. 생각대로라면 음악 정지 버튼을 누르듯이 작업을 중지시키는 것으로 끝나야할 것 같은데 그렇게 동작하지 않는군요! <br>

위에서 언급했듯이 cancel 은 아직 이해가 필요한 부분들이 있어서 따로 포스팅 후에 링크를 남겨두도록 하겠습니다 :)

<br>

### ✐ qualityOfService

```swift
var qualityOfService: QualityOfService
```

Operation과 DispatchQueue에서 언급했던 QoS입니다. OperationQueue에도 같은 QualityOfService 열거형을 할당해줄 수 있습니다. 대기열 작업을 효율적으로  수행할 수 있도록  여러 우선순위 옵션을 제공하며, 역시 에너지 효율과 관련이 있겠네요.

```swift
enum QualityOfService : Int {
  case userInteractive = 33
  case userInitiated = 25
  case utility = 17
  case background = 9
  case `default` = -1
}
```

<br>

### ✐ isSuspended

```swift
var isSuspended: Bool { get set }
```

isSuspended는 대기열이 Operation 스케줄링이 진행 중인지에 대한 상태를 나타내는 Bool 값입니다. false인 경우, 대기열의 Operation을 실행시키고, true인 경우, 대기열의 Operation을 실행하지는 않지만 현재 실행 중인 Operation은 계속 실행이 됩니다. 중단된 대기열에  Operation을 추가할 수는 있지만 isSuspended가 false가 될때까지 실행되지는 않습니다.  <br>

Operation은  실행이 완료되어야만 OperationQueue에서 제거가 되는데, 대기열이 중단된 상태라면 Operation이 완료, 제거되지 못하고 계속 남아있는 상태가 됩니다. 이때에는 처음에 언급한 **메모리 누수**가 발생할 수 있겠네요.

<br>

-------------------

cancelAllOperations와 schedule  메서드에 대해서는  공부가 조금 더 필요할 것 같습니다. 포스팅이 완료되면 링크를 남겨두도록 하겠습니다 :)


<br>
<br>

#### 참고
- [https://developer.apple.com/documentation/foundation/operationqueue](https://developer.apple.com/documentation/foundation/operationqueue)
- [https://developer.apple.com/documentation/foundation/operation#1661262](https://developer.apple.com/documentation/foundation/operation#1661262)
- [https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationObjects/OperationObjects.html#//apple_ref/doc/uid/TP40008091-CH101-SW1](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/OperationObjects/OperationObjects.html#//apple_ref/doc/uid/TP40008091-CH101-SW1)
- [https://www.boostcourse.org/mo326/lecture/16898](https://www.boostcourse.org/mo326/lecture/16898)


<br>
<br>
