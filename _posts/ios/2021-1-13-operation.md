---
layout: post
title: OperationQueue 1편, Operation
category: iOS
tags: [Operation]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.       
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

오늘은 DispatchQueue와 유사한 OperationQueue에 대해서 포스팅합니다. 이번 포스팅은 그 중에 Operation에 대한 내용입니다. OperationQueue에 대해 간단하게 알아보고 코드로 Operation을 사용해보도록 하겠습니다 :)

<br>
<br>

# OperationQueue
**OperationQueue** 는 작업 대기열이며 **Operation** 들이 대기열을 이루는 것입니다. 즉, Operation이라는 `스레드에서 실행할 Task를 캡슐화한 객체`가 OperationQueue에 들어가는 것이지요. **OperationQueue는 Operation의 실행과 작업 스케쥴링을 관리합니다.** 그리고 내부적으로는 GCD를 사용합니다. (OperationQueue를 GCD에서 기능이 추가된 것이라고 하기도 합니다.)

<br>

### Operation

Operation은 네 가지 상태를 가집니다. Ready, Executing, Finished, isCanceled 상태가 있습니다. 먼저 Ready는 Operation을 실행할 수 있는 상태(실행 전), Executing은 실행 중인 상태, Finished는 Operation이  정상적으로 종료되거나 중간에 취소된 상태를 말합니다. 그리고 **한 번 실행된 Operation은 다시 실행할 수 없습니다.** <br>


>**Operation이 제공하는 주요 기능**
>- 작업의 의존성 설정
>- 작업 취소
>- 완료 블록
>- KVC 호환, KVO를  통해서 상태 변화 감시
>- 우선 순위 설정


<br>
<br>

# Operation 사용하기
직접 코드를 써보면서 OperationQueue에 대해서 이해해보도록 하겠습니다! OperationQueue에는 Operation이 들어갈테니, Operation을 먼저 살펴볼까요?
<br>

### ✐ BlockOperation
BlockOperation은 Operation을 상속받는 클래스입니다. Operation은 직접적인 초기화는 없고 자식클래스인 BlockOperation을 만드는 것으로 형체를 가지게 됩니다. [공식문서](https://developer.apple.com/documentation/foundation/operation)에서도 Opertaion은  추상 클래스이기 때문에 작업 수행을 위해 직접 사용되지는  않고, 다른 객체에 상속시킨 후에 사용한다고 하네요 :)

```swift
let someOperation1 = BlockOperation {
    // some code
}

let someOperation2 = BlockOperation {
    // some code
}
```

위에서 Operation은 캡슐화된 객체라고 했습니다. 코드 블록 안에는 실행될 코드가 들어갑니다.
<br>

### ✐ start(), cancel(), name
꼭 OperationQueue에 Operation을 넣지않아도, 독립적으로 실행해줄 수 있습니다. 그리고 name 프로퍼티로 Operation의 이름을 String 타입으로 할당해줄 수 있습니다.

```swift
let someOperation1 = BlockOperation {
    for i in 1...10 {
        print("\(i) -------> 👍")
    }
}

someOperation1.name = "오동나무"
someOperation1.start()
// someOperation1.cancel()
```

<br>

### ✐ addExecutionBlock(_: )
addExecutionBlock 메서드는 BlockOperation의 코드블록에 코드를 추가하는 메서드입니다.

```swift
let someOperation1 = BlockOperation {
    for i in 1...10 {
        print("\(i) -------> 👍")
    }
}

someOperation1.addExecutionBlock {
    print("아낌없이 주는 오동나무")
}

someOperation1.start()
```

뭐가 먼저 실행될까요? 어떤 순서로 출력이 되는지 가늠이 되지 않아서 여러개의 블록을 추가해서 실험해보았습니다.

```swift
let someOperation1 = BlockOperation {
    print("Start!")
}

someOperation1.addExecutionBlock {
    sleep(2)
    print("아낌없이")
}

someOperation1.addExecutionBlock {
    sleep(2)
    print("주는")
}

someOperation1.addExecutionBlock {
    sleep(2)
    print("오동나무")
}

someOperation1.addExecutionBlock {
    sleep(2)
    print("odongnamu")
}

someOperation1.start()

/*
Start!
주는
오동나무
odongnamu
아낌없이
*/
```
연속적으로 실행되는 것인지 확인하기 위해서 각 코드블록에 2초씩 지연시켜보았는데요, 만약 연속적으로 코드가 실행되는 것이라면 적어도 8초 이상의 시간이 걸려야겠죠? 하지만! 대충 2~4초 정도에 코드가 모두 수행되는 것을 확인했습니다. **비동기적으로 처리되었습니다.** <br>

addExecutionBlock으로 추가한 코드는 `DispatchQueue.global().async`처럼 비동기적으로 동작합니다. **출력 순서 또한 비동기적이기 때문에 예측할 수 없었고 매번 순서가 달랐습니다.**

<br>

### ✐ completionBlock: (() -> Void)?
completionBlock은 Operation의 프로퍼티로 Operation의 **수행이 끝났을 때 실행** 되는 코드 블록을 담는 프로퍼티입니다.

```swift
let someOperation1 = BlockOperation {
    print("Start!")
}

someOperation1.addExecutionBlock {
    sleep(2)
    print("아낌없이")
}

someOperation1.addExecutionBlock {
    sleep(2)
    print("주는")
}

someOperation1.addExecutionBlock {
    sleep(2)
    print("오동나무")
}

someOperation1.addExecutionBlock {
    sleep(2)
    print("odongnamu")
}

someOperation1.completionBlock = {
    print("Finish!")
}

someOperation1.start()

/*
Start!
아낌없이
오동나무
주는
odongnamu
Finish!
*/
```

이렇게 사용하니 DispatchGroup의 사용과 유사한 것 같네요. 또한 DispatchGroup의 wait()와 유사하게 `waitUntilFinished()` 메서드로  Operation의 종료를 추적할 수도 있습니다.

```swift
let someOperation1 = BlockOperation {
    print("Start!")
}

someOperation1.addExecutionBlock {
    sleep(2)
    print("아낌없이")
}

...

someOperation1.waitUntilFinished()
print("Finish!")

someOperation1.start()
```

<br>

### ✐ addDependency(operation), removeDependency(operation)
의존, 종속 정도로 해석되는 것 같습니다. 저는  종속이라는 의미가 더 적합해보여서 '종속을 추가한다'는  메서드로 이해해보았습니다. 어떤 A라는 Operation에 종속된 B(Operation)는 A가 처리되지 않으면 실행되지 않는 성격을 가지게 됩니다. 종속, 또는 의존하게 되는 것이지요.

```swift
let someOperation1 = BlockOperation {
    print("Start!")
}

let blockOperation1 = BlockOperation {
    sleep(1)
    print ("아낌없이 주는")
}

let blockOperation2 = BlockOperation {
    print("오동나무")
}

blockOperation2.addDependency(blockOperation1)
// blockOperation2는 blockOperation1에 종속됩니다. (의존합니다.)

someOperation1.start()
blockOperation1.start()
blockOperation2.start()
```

**addDependency를 호출한 Operation이 종속됩니다!!**       
이 부분이 좀 헷갈리긴 하는 것 같아요 ㅜ.

```swift
someOperation1.start()
//blockOperation1.start()
blockOperation2.start() // -> error!
```

2가 1에 종속된 상태에서, 만약 blockOperation1을 먼저 실행하지 않고 blockOperation2를 실행하게 되면 에러가 발생합니다. 또한 서로를 종속한다면 아무것도 실행되지 않습니다. `removeDependency` 메서드를 사용하면 종속성을 해제해줄 수 있습니다. 또한 `dependencies: [Operation]` 배열로서 여러 Operation에 대한 종속성을 가질 수도 있습니다. <br>

addDependency의 성격은 OperationQueue 안에 들어가도 종속성은 유지되기 때문에 유용한 기능이 될 것 같네요 :)

<br>

### ✐ Getting the Operation Status
Operation은 Bool 타입으로 상태를 확인할 수 있습니다.
- isCancelled: Bool
- isExecuting: Bool
- isFinished: Bool
- isConcurrent: Bool
- isAsynchronous: Bool
- isReady: Bool

<br>

### ✐ queuePriority
Operation에도 우선 순위를 할당해줄 수 있습니다. `queuePriority` 프로퍼티를 통해 실행되는 우선 순위를 할당할 수 있으며 아무런 할당을 해주지않으면 normal 수준이 됩니다. (Queue 에 Operation이 들어가야 의미가 있기때문에 아직 OperationQueue를 공부하지 않으신 분이라면 느낌만 보세요! 사실 이해하기 어렵지는 않습니다! ) <br>

```swift
enum QueuePriority: Int {
  case veryLow = -8
  case low = -4
  case normal = 0
  case high =  4
  case veryHigh = 8
}
```

```swift
let someOperation1 = BlockOperation {
    print("Start!")
}

let blockOperation1 = BlockOperation {
//    sleep(1)
    print ("아낌없이 주는")
}

let blockOperation2 = BlockOperation {
    print("오동나무")
}

blockOperation2.queuePriority = .high
blockOperation1.queuePriority = .normal

let queue = OperationQueue()
queue.addOperation(blockOperation1)
queue.addOperation(someOperation1)
queue.addOperation(blockOperation2)

/*
아낌없이 주는
Start!
오동나무
*/
```

보통 우선 순위라고 하면 blockOperation2가 높기때문에 먼저 실행될 거라 예상되지만 blockOperation1이  먼저 실행이 됩니다. 어떻게 된 일일까요? <br>

우선 순위에서 유의해야할 점은 queue 내부에서 우선순위대로 Operation이 실행되는 것이 아니라는 점입니다. 여기서 다루어지는 우선순위는 Ready 상태에 있는  Operation을 대상으로  하는 우선 순위입니다. 따라서 아래의 테스트가 더 정확하겠네요.

```swift
blockOperation2.queuePriority = .high
blockOperation1.queuePriority = .normal
blockOperation2.addDependency(someOperation1)
blockOperation1.addDependency(someOperation1)

let queue = OperationQueue()
queue.addOperation(blockOperation1)
queue.addOperation(someOperation1)
queue.addOperation(blockOperation2)

/*
Start!
오동나무
아낌없이 주는
*/
```

blockOperation1과  blockOperation2가 someOperation1에 종속되어 있기 때문에 someOperation1이 끝나면 둘은 동시에 Ready 상태가 됩니다. 따라서 이때에는 우선 순위에 따라 blockOperation2가 먼저 실행이 되는 것입니다.

>**QueuePriority는 Int 타입?**
QueuePriority는 Int 타입의 rawValue를 가지고 있습니다. 이때 만약 rawValue에 해당하지 않는 Int 값을 입력해주면 자동으로 가장 가까운 값을 찾아서 우선 순위가 결정됩니다.

<br>

### ✐ qualityOfService
qualityOfService 역시 queuePriority와 같이 우선 순위를 설정하는 프로퍼티입니다. QoS에 대해서는 [DispatchQoS](https://odong-tree.github.io/ios/2021/01/12/dispatchqueue2/)에서도  잠깐 언급한 적이 있는데, 실행 순서와도 관련이 있지만 에너지 효율을 결정하는 우선순위이기도 합니다. (queuePriority는 iOS 2.0에서, qualityOfService는 iOS 8.0에서 나온 개념으로 더 최신 기술이네요!)<br>

우선 순위가 높으면 queue에서 먼저 실행되기도 하면서 처리 시간이 짧아지도록 하기 위해 보다 많은 에너지를 투자하는 원리입니다. 사용 방법은 QueuePriority와 비슷합니다.

```swift
enum QualityOfService : Int {
  case userInteractive = 33
  case userInitiated = 25
  case utility = 17
  case background = 9
  case `default` = -1
}
```

```swift
blockOperation2.qualityOfService = .userInteractive
blockOperation1.qualityOfService = .default
blockOperation2.addDependency(someOperation1)
blockOperation1.addDependency(someOperation1)

let queue = OperationQueue()
queue.addOperation(blockOperation1)
queue.addOperation(someOperation1)
queue.addOperation(blockOperation2)
```

<br>

### ✐ main()
마지막으로 main 메서드는 Operation를 상속받는 블록을 직접 만들어줄 때 override 해야하는 메서드입니다. 즉, BlockOperation 타입을 사용하지 않고 직접 커스텀해주는 것입니다. **main메서드에 구현한 코드블럭이 Operation이 수행하는 코드가 됩니다.**

```swift
class newOperation: Operation {
    override func main() {
        // some code
    }
}
```
<br>

---------------

[OperationQueue 2편 포스팅 보러가기](https://odong-tree.github.io/swift/2021/01/14/operationqueue/)



<br>
<br>

#### 참고
- [https://developer.apple.com/documentation/foundation/blockoperation](https://developer.apple.com/documentation/foundation/blockoperation)
- [https://developer.apple.com/documentation/foundation/operation](https://developer.apple.com/documentation/foundation/operation)
- [https://developer.apple.com/documentation/foundation/operationqueue](https://developer.apple.com/documentation/foundation/operationqueue)
- [https://blog.naver.com/PostView.nhn?blogId=horajjan&logNo=220888295104&redirect=Dlog&widgetTypeCall=true](https://blog.naver.com/PostView.nhn?blogId=horajjan&logNo=220888295104&redirect=Dlog&widgetTypeCall=true)

<br>
<br>
