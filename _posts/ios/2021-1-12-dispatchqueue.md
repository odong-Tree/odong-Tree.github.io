---
layout: post
title: GCD, DispatchQueue 이해하기
category: iOS
tags: [GCD, DispatchQueue]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.       
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

오늘은 GCD, DispatchQueue에 대해서 알아보도록 하겠습니다. GCD에 대해서 공부하기 전에 미리 알아두면 좋은 개념들을 **간단**하게 살펴봅시다. (GCD에 대한 개념 정리는 [부스트코스](https://www.boostcourse.org/mo326/lecture/16916)를 참고합니다.)

# 미리 알아두기
#### ✐ Thread (쓰레드)
스레드는 프로세스 내에서 실행되는 흐름의 단위를 말합니다. **프로그램의 작업을 처리하는** 녀석들을 말합니다. 경우에 따라서 스레드는 하나일 수도, 여러 개(멀티 스레드)가 있을 수도 있습니다. 하나의 스레드가 모든 작업을 처리하는 것보다, 여러 개의 스레드에 분산 시켜 작업을 처리하는 것이 효율적입니다.
<br>

#### ✐ 동시성 프로그래밍
동시성 프로그래밍과 병렬성 프로그래밍은 유사해서 헷갈리는 개념입니다. 먼저 동시성 프로그래밍은 실제로는 동시에 작업을 처리하지 않지만, **동시에 작업이 처리되는 것처럼 보이는 것** 입니다. **싱글 코어에서 멀티 스레드(Multi thread)를 동작시키는 것** 으로, **실제로는 아주 작은 단위로 스레드를 번갈아가며 작업을 처리** 하는 것이기 때문에 우리에게는 마치 동시에 작업이 처리되고 있는 것처럼 보이게 됩니다. 즉 **논리적인 개념**으로 동시인 것이지요.
<br>

#### ✐ 병렬성 프로그래밍
반면 병렬성 프로그래밍은 **실제로 동시에 여러 작업을 처리하는 것** 을 말합니다. 동시성 프로그래밍과 달리 **물리적인 개념** 으로, **멀티 코어에서 멀티 스레드를 동작시키는 방식** 입니다.
<br>

#### ✐ 동시성과 병렬성
 **동시성과  병렬성은 반대되는 개념이 아닙니다.** 한 상황에 동시성과 병렬성이 공존할 수도 있습니다. 병렬성 프로그래밍은 코어가 멀티 코어 이상일 경우에만 가능하지만, 이 둘의 개념을 꼭 CPU의 갯수로 구분할 수는 없습니다. 만약 여러 개의 일을 여러 CPU가 처리하고 있는 경우, 만약 하나의 CPU가 하나의 작업만 처리하고 있다면 이는 병렬성 프로그래밍이라고 볼 수 없습니다. <br>

 - 병렬성: 하나의 작업을 쪼개서 여러 개의 CPU가 동시에 처리한다.
 - 동시성: 하나의 작업을 하나의 CPU가 처리하고 있다.


<br>
<br>

# GCD (Grand Central Dispatch)
**GCD** 란 멀티코어와 멀티 프로세싱 환경에서 최적화된 프로그래밍을 할 수 있도록 애플이 개발한 기술을 말합니다. 스레드 풀의 관리를 프로그래머가 아닌 운영체제에서 관리하기 때문에 프로그래머가 작업을 비동기적으로 쉽게 사용할 수 있게 됩니다. 프로그래머가 실행할 작업을 생성하고 **Dispatch Queue** 에 추가하면 GCD는 작업에 맞는 스레드를 자동으로 생성해서 실행하고 작업이 종료되면 해당 스레드를 제거합니다. 즉, 프로그래머가 직접 스레드를 어떻게 멀티 코어 프로세서에 분산 시킬 것인가에 대해 알아서 처리해주는 기능입니다.

<br>

# DispatchQueue
DispatchQueue는 GCD 기술의 일부입니다. 작업을 연속적으로 혹은 동시에 진행하지만 그 내부는 **FIFO(First-In, First-Out)구조**로 대기열이 관리됩니다. DispatchQueue에는 Serial DispatchQueue와 Concurrent DispatchQueue가 있습니다.

>DispatchQueue와 OperationQueue가 자주 비교됩니다. OperationQueue에 대해서는 따로 포스팅을 할 예정입니다. 간단하게 이야기하면 둘은 서로 비슷하지만 GCD에서 약간의 기능이 추가된 것이 OperationQueue라고 할 수 있습니다.

### ✐ Serial, Concurrent
- **Serial**       
대기열(Queue)에 등록한 순서대로 작업을 실행합니다. 하나의 작업을 실행하고 실행이 끝날 때까지 대기열(Queue)에 있는 다른 작업을 미루고 있다가 이전 작업이 끝나면 실행합니다. 즉 하나의 스레드에서 작업이 이루어 지는 것입니다.

```swift
DispatchQueue(attributes: .serial)
DispatchQueue.main

// main은 전역적으로 사용되는 Serial DispatchQueue 입니다.
```

- **Concurrent**       
실행 중인 작업이 끝나기를 기다리지 않고 대기열(Queue)에 있는 작업을 동시에 별도의 스레드를 사용하여 실행합니다. 즉, 새로운 스레드를 만들어 작업을 처리하는 병렬처리 방식입니다.

```swift
DispatchQueue(attributes: .concurrent)
DispatchQueue.global()
```

<br>

### ✐ 동기, 비동기
각 DispatchQueue에는 동기와 비동기 방식이 있습니다.

- **동기**:  DispatchQueue에서 실행을 위해 클로저를 대기열(Queue)에 추가하고 해당 클로저가 완료될 때까지 대기합니다.

- **비동기**: DispatchQueue에서 비동기 실행을 위해 클로저를 추가하고 즉시 실행합니다.

이렇게 보면 무슨 말인지 모르겠네요. 코드와 같이 설명해보겠습니다.

```swift
// 동기, sync
DispatchQueue.main.sync {}
DispatchQueue.global().sync {}

// 비동기, async
DispatchQueue.main.async {}
DispatchQueue.global().async {}
```
- **DispatchQueue.main.sync**         
  논리로 따지면 일반적으로 메인스레드에서 메서드가 호출되는 것과 같습니다. 하지만 실제로 코드를 사용하려고 하면 main.sync는 **deadlock**이 발생됩니다.

- **DispatchQueue.global().sync**        
concurrent 스레드에서 동기적으로 작업을 처리하게 됩니다. 작업은 여러 스레드로 분산되었지만 순서대로 처리됩니다.

- **DispatchQueue.main.async**         
비동기적 실행이지만 곧장 메서드가 시작되지는 않습니다. DispatchQueue가 아닌, 메인스레드의 작업이 모두 종료된 후에 순차적으로 작업이 실행됩니다. 그 이유는 비동기적 처리이지만 하나의 스레드에서 작업 순서가 정해지기 때문인 것 같습니다.

- **DispatchQueue.global().async**        
스레드를 새로 만들어 동시적으로 작업을 수행합니다.

<br>
<br>

# ✐ 확실하게 이해하기
#### 1. serial과 main은 같은 것인가?
- Main Queue는 전역적으로 사용 가능한 Serial Queue이다.
- Global은 전역적으로 사용 가능한 Concurrent Queue이다.

#### 2. main과 global
- 공통점: 전역적으로 접근이 가능하다.
- 차이점: main은 serial, global은 concurrent이다. 또 main은 프로퍼티, global은 메서드이다. global에서는 우선 순위를 수 있다.

#### 3. serial, sync
- serial은 큐 내부에서 순서대로 일이 처리가 되는 것
- sync는 큐 외부의 코드를 중단하고, 큐 내부의 코드가 우선적으로 전부 수행되기까지 기다리게하는 것

#### 4. main.async
- main큐는 단일스레드에서 업무를 처리하기 때문에 비동기적으로 일을 처리하더라도 본인의 업무 순서를 기다리는 방식으로 결국 구현된다.
>**main.async의  백그라운드 작업**                     
main.async의 백그라운드 작업 UI작업을 메인스레드에서 하고, main.async로 새로운 스레드에서 네트워크 등의 작업을 수행한다. 사용자의 인터페이스 작업이 가능한 시점은 viewDidLoad가 끝난 후다. 그래서 viewDidLoad에 네트워크 작업하면 뷰가 만들어지는 시점이 늦어지게 된다. viewDidAppear에서 네트워크 등의 백그라운드 작업을 해주어야 하는데, 이때 main.async로 메인 스레드에서 비동기 작업을 하는 것으로 사용자와의 상호작용이 가능한 상태에서 백그라운드 작업을 하게된다. (만약 viewDidAppear에서 DispatchQueue가 아닌 일반적인 코드를 써주면, 코드가 끝날때까지 사용자와의 상호작용은 할 수 없다.)


#### 5. main.sync의 deadlock
- Main.sync를 직접적으로 호출하면 deadlock(교착상태)에 빠지게 된다. 새로 수행하고자 하는 작업은 자신의 큐를 block하면서 들어가고자하는 큐의 업무수행완료를 기다리게되고, 기존에 수행되던 큐는 block에 걸려서 자신의 업무를 수행하지 못한다. 서로가 서로의 업무수행완료를 기다리게 된다.

1. main.sync는 main 큐에서 진행중인 작업을 Block-wait 상태로 만들어버린다.
2. main.sync는 동기적으로 진행중인 작업이 끝나기를 기다렸다가, 끝나면 작업을 수행한다.
3. 진행중인 작업을 멈춰버렸기 때문에, 끝나기를 기다리면 영원히 기다리게 된다. 새로운 작업을 시작할 수가 없는 상황이된다.

#### 그 외 주의할 점
DispatchQueue를 통해 스레드에 작업을 할당하는 경우, 작업이 처리되는 순서에 대해서는 논리적인 기대를 할 수 없다는 점에 유의해야합니다. 예를들어  `global().async` 로 비동기적으로 작업을 처리하는 경우, 먼저 호출된 DispatchQueue라고 먼저 작업이 수행되는  것은 아닙니다.

---------------------


[GCD, Qos와 DispatchGroup 포스팅 보러가기](https://odong-tree.github.io/ios/2021/01/12/dispatchqueue2/)

<br>
<br>

#### 참고
- [https://www.boostcourse.org/mo326/lecture/16916](https://www.boostcourse.org/mo326/lecture/16916)
- [https://jinshine.github.io/2018/07/09/iOS/GCD(Grand%20Central%20Dispatch)/](https://jinshine.github.io/2018/07/09/iOS/GCD(Grand%20Central%20Dispatch)/)

<br>
<br>
