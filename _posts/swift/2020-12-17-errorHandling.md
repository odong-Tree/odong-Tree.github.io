---
layout: post
title: 에러 핸들링 (Error Handling)
category: swift
tags: [error]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.
정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

앱 개발을 하면서 에러가 발생할 상황에 대해서 어떻게 대처하나요?  <br>

사용자에게 alert를 띄워줄 수도 있고, 내부적으로 확인 또는 해결할 수 있는 메서드를 작성하는 방법 등 여러 방법이 있을 수 있습니다.   <br>

에러가 발생할 수 있는 상황이 언제나 한 두가지 정도라면 좋겠지만, 대부분의 경우 다양한 에러에 대해 대처해 주어야할텐데요. 이때 일일이 에러가 발생하는 부분마다 에러를 처리하는 메서드를 작성해주어야 할까요? <br>

오늘은 **에러 핸들링(Error Handling)** 에 대해서 포스팅하도록 하겠습니다.    
포스팅은 공식문서를 토대로 학습한 내용을 작성합니다.
<br>

# Error Handling
에러 핸들링은 말 그대로 발생할 수 있는 Error 상황에 대해서 예상해두고,    
에러가 발생했을 때 핸들링(Handling) 해주는 것입니다. <br>

처음 에러 핸들링을 접했을 때에는    
'그냥 메서드를 만들거나 조건문을 사용해서 해결해주는 코드를 쓰면될텐데'라는     
의문이 들었는데요, 그렇다면    
<br>

### ✐ 에러 핸들링은 왜 필요한 것일까요?

에러 핸들링은 에러를 ‘반환’하는 것이 아니라 ‘던지는’ 것입니다.    
실행 과정에서 발생할 수 있는 다양한 오류들을 함수 외부로 전달하기 위해 사용합니다.   <br>

'**반환하는 것이 아니라 던진다**'는 말은 무엇일까요?     
<br>

#### 에러를 반환한다?

예를 들어 옵셔널 타입을 반환받는 방식으로 에러에 대처한다고 하면 오류의 종류와 상관없이 단순히 nil이라는 값을 반환하게 됩니다. 실제로 에러가 발생할 수 있는 경우는 더욱 다양할 수 있는데요, 그렇다면 그에 대한 대처도 다양해져야겠지요? 이에 대해 에러 핸들링은 오류를 좀 더 세분화하여 적절하게 핸들링하기 위해 사용할 수 있습니다. <br>

#### 오류를 던진다?
오류를 던지게 되면 반환이 아닌, 오류 객체를 만들어 다른 실행 흐름으로 옮겨가는 것입니다. 그러니까 해당 메서드에서 발생한 에러를 책임지지 않고, **"난 몰라~!"** 하면서 그의 외부 함수로 책임을 던져버리는 것입니다. 그러면 그 외부 함수에서 에러 처리를 떠맡는 상황이 되는 것이지요.    <br>

그리고 외부 함수에서 에러를 핸들링해주는 코드를 써주면 됩니다. <br>

오류를 반환한다면 함수의 반환 타입과 일치해야하지만 오류를 던지는 경우에는 반환타입과 일치하지 않아도 됩니다. 추후에 핸들링을 해주면 됩니다.

<br>

# ✐ 에러 핸들링 어떻게 할까?

>
>1. Error 프로토콜을 채택한 **열거형**으로 에러 관리하기
>2. Throwing 함수로 에러 전파(propagate)하기
>3. do-catch 문으로 에러 핸들링(handling)하기
>

*예제는 **공식문서**와 **야곰의 유튜브**에서 가져온 코드입니다 :)*

<br>

### 1. Error 프로토콜을 채택한 **열거형**으로 에러 관리하기

```swift
enum VendingMachineError: Error {
    case InvalidSelection
    case InsufficientFunds(coinsNeeded: Int)
    case OutOfStock
}
```

VendingMachine에 대한 에러를 열거형으로 작성해주었습니다.   
이때 Error 프로토콜을 채택한 것을 볼 수 있습니다.     <br>

Error는 사실 비어있는 프로토콜입니다만, 프로토콜을 채택함으로써 에러타입으로 사용하겠다는 표시 정도라고 할 수 있습니다. 이때 rawValue에 String 값을 입력하게 되면 에러 발생시 미리 입력해둔 문자열을 출력해줄 수 있습니다.  <br>

그리고 이제 throw라는 키워드를 통해 열거형으로 정리해둔 Error를 던져줄 수 있습니다.
던져진 Error를 핸들링하는 것이 에러 핸들링입니다.
<br>

### 2. Throwing 함수로 에러 전파(propagate)하기

```swift
func canThrowErrors() throws -> String

func cannotThrowErrors() -> String
```

에러를 던지는 메서드를 만들기 위해서는 throws라는 키워드가 필요합니다. 이렇게 throws 키워드가 있는 메서드를 **throwing 함수**라고 하는데요, throws 키워드는 이렇게 파라미터 다음 순서로 써줄 수 있습니다. (return 값이 없는 경우에도 throws 를 쓸 수 있습니다!) <br>

throwing 함수는 함수 내부에서 발생으로 던져진 에러를 자신을 호출한 scope으로 전파(propagate)합니다. throwing 함수만이 이렇게 에러를 전파할 수 있습니다. 만약 throwing 함수가 아닌 함수에서 에러가 발생한다면 함수의 내부에서 직접 에러를 처리해주어야 합니다. <br>

이제 에러를 어떻게 던지는지 아래의 코드로 확인해보겠습니다.

```swift
enum VendingMachineError: Error {
    case invalidSelection
    case insufficientFunds(coinsNeeded: Int)
    case outOfStock
}

struct Item {
    var price: Int
    var count: Int
}

class VendingMachine {
    var inventory = [
        "Candy Bar": Item(price: 12, count: 7),
        "Chips": Item(price: 10, count: 4),
        "Pretzels": Item(price: 7, count: 11)
    ]

    var coinsDeposited = 0

    func vend(itemNamed name: String) throws {
        guard let item = inventory[name] else {
            throw VendingMachineError.invalidSelection

        }

        guard item.count > 0 else {
            throw VendingMachineError.outOfStock
        }

        guard item.price <= coinsDeposited else {
            throw VendingMachineError.insufficientFunds(coinsNeeded: item.price - coinsDeposited)
        }

        coinsDeposited -= item.price

        var newItem = item
        newItem.count -= 1
        inventory[name] = newItem

        print("Dispensing \(name)")
    }
}
```

vend라는 메서드에서 throw 키워드를 통해 에러 프토콜의 에러 케이스를 적절하게 던져주고 있습니다. throw와 return의 퍼포먼스는 비슷합니다. throw는 오류를 던지는 동시에 메서드를  return 시킵니다.
<br>

#### 에러를 던진다.. 에러를 던지면 어떻게 될까요?
함수가 에러를 던지게 되면 프로그램의 플로우가 바뀌게 됩니다. 에러가 던져진 순간 해당 메서드는  return되고, 그 외부 메서드로 플로우가 이동하게 되겠지요. **따라서 에러가 던져질 수 있는 코드 구간을 식별할 수 있도록 하는 것은 매우 중요합니다!!** <br>

에러를 던질 수 있는 메서드 앞에 try (try?, try!) 키워드를 써서 이를 식별해줄 수 있습니다.
<br>

```swift
let favoriteSnacks = [
    "Alice": "Chips",
    "Bob": "Licorice",
    "Eve": "Pretzels",
]
func buyFavoriteSnack(person: String, vendingMachine: VendingMachine) throws {
    let snackName = favoriteSnacks[person] ?? "Candy Bar"
    try vendingMachine.vend(itemNamed: snackName)
}
```

```swift
struct PurchasedSnack {
    let name: String
    init(name: String, vendingMachine: VendingMachine) throws {
        try vendingMachine.vend(itemNamed: name)
        self.name = name
    }
}
```
<br>

### 3. do-catch 문으로 에러 핸들링(handling)하기

```swift
do {
    try <throw 함수>
} catch (pattern1) {
    statement
} catch (pattern2) where condition {
    statement
} catch (pattern3), (pattern4) where condition {
    statements
} catch {
    statements
}
```

do-catch 문으로 에러를 핸들링할 수 있습니다. do 구문 안에서 에러가 던져지면, 그 에러를 핸들링할 수 있는 catch 문을 찾아 에러를 핸들링해줍니다. (왠지  if-else?  switch? 와 비슷하게 생겼습니다.) <br>

catch에 원하는 패턴(열거형에서 에러 종류)를 작성하고, statements에서 원하는 코드블록을 실행시킬 수 있습니다. do-catch문 역시 if-else if-else문처럼 순서대로 조건을 확인합니다.  <br>

```swift
var vendingMachine = VendingMachine()
vendingMachine.coinsDeposited = 8
do {
    try buyFavoriteSnack(person: "Alice", vendingMachine: vendingMachine)
    print("Success! Yum.")
} catch VendingMachineError.invalidSelection {
    print("Invalid Selection.")
//} catch VendingMachineError.outOfStock {
//    print("Out of Stock.")
} catch VendingMachineError.insufficientFunds(let coinsNeeded) {
    print("Insufficient funds. Please insert an additional \(coinsNeeded) coins.")
} catch {
    print("Unexpected error: \(error).")
}
// Prints "Insufficient funds. Please insert an additional 2 coins."
```

buyFavoriteSnack 메서드는 do 구문 안에서 try라는 키워드를 사용하면서 호출되고 있습니다. 그 이유는 이 메서드가 에러를 throw 할 수 있는 throwing 메서드이기 때문입니다. 이 경우, buyFavoriteSnack가 에러를 던지게 되면 알맞은 catch 구문을 찾아가서 코드블록을 실행시키게 됩니다. <br>

다른 방식으로도 코드를 쓸 수 있는데요,


```swift
do {
    try machine.receiveMoney(300)
} catch {

    switch error {
    case VendingMachineError.invalidInput:
        print("입력이 잘못되었습니다.")
    case VendinMachineError.insufficientFunds(let moneyNeeded):
        print("\(moneyNeeded)원이 부족합니다.")
    case VendingMachineError.outOfStock:
        print("수량이 부족합니다.")
    default:
        print("알 수 없는 오류 \(error)")
    }
}
```

이렇게 switch 문으로 에러를 핸들링해줄 수도 있고,      
새로운 메서드를 만드는 것으로 핸들링을 해줄 수도 있습니다. <br>

catch에 아무 패턴도 적어주지 않는다면 어떤 에러라도 매칭이 가능하며 그 에러는 error라는 이름의 로컬 상수로 바인딩 됩니다. if-else 구문의 else, switch 구문의 default같은 역할을 하는군요. <br>

하지만 귀찮도록 모든 에러에 대해서 저렇게 적어줘야할까요?
아래와 같이 에러를 그냥 묶어서 쓰기도하고, 에러가 던져져도 그냥 두도록 작성할 수도 있습니다.

```swift
do {
    result = try machine.vend(numberOfItems: 4)
} catch {
    print(error)
} // insufficientFunds(100)


do {
    result = try machine.vend(numberOfItems: 4)
}
```

<br>

그렇다면 던져질 수 있는 모든 에러에 대해서 catch 구문으로 핸들링해야 할까요? <br>

꼭 그런 것은 아닙니다. 만약 catch에서 모든 에러를 핸들링하지 않는다면, catch 구문이 다루지 않는 에러가 던져진다면 그 에러는 메서드가 호출된 scope에 전파됩니다. <br>

하지만 주의할 점은 do-catch에서 던져진 에러를 다루지 않더라도 scope 내부에서는 에러 핸들링을 반드시 해주어야 한다는 것입니다.

```swift
func nourish(with item: String) throws {
    do {
        try vendingMachine.vend(itemNamed: item)
    } catch is VendingMachineError {
        print("Couldn't buy that from the vending machine.")
    }
}

do {
    try nourish(with: "Beet-Flavored Chips")
} catch {
    print("Unexpected non-vending-machine-related error: \(error)")
}
// Prints "Couldn't buy that from the vending machine."
```

<br>

### + 에러를 옵셔널 값으로 변환하기

throw 함수를 호출하는 키워드 try는 옵셔널로 try? 혹은 try! 로 써줄 수 있습니다.

```swift
func someThrowingFunction() throws -> Int {
    // ...
}

let x = try? someThrowingFunction()

let y: Int?
do {
    y = try someThrowingFunction()
} catch {
    y = nil
}

let z = try! someThrowingFunction()
```

try?의 경우, someThrowingFunction이 에러를 던지게 되면 x와 y의 값은 nil이 됩니다. 하지만 에러를 던지지 않고 정상동작이 될때에 return 값은 optional 값이 되어버립니다. <br>

try!는  오류를 던지도록 설계된 throw function이지만 필요에 의해 오류를 던지지 않게 하고 싶을 때  사용합니다. 하지만 이때 만약 오류가 던져지게 된다면 그대로 런타임 오류로 이어집니다.옵셔널 강제 해제와 같은 맥락을 하네요.

<br>

#### try?, try! 왜 쓸까?
저는 try?, try! 키워드를 왜 사용하는지에 대해 의문이 들었습니다. 결국 에러를 던지지 않고, 반환하는 식으로 메서드를 호출하는 것인데 그러면 throw 함수를 만든 의미가 있을까? throw 함수로 만들었지만, 에러를 던지고 싶지않을 때 사용하는 키워드인가? 라는 의문이었습니다. <br>

**개인적인 생각입니다.** 코드를 좀 더 써보면서 그 쓰임새를 알아가겠지만, 지금 생각으로는 throw 함수로 만들어놓은 메서드를 테스트하는 과정에서 자주 사용될 수 있을거라 생각했습니다. 또한 의문  그대로, throw 함수이지만, 에러를 던지고 싶지 않은 상황이 또 있지 않을까하는 생각입니다. 그게 언제일까요?

<br>



# defer

```swift
func processFile(filename: String) throws {
    if exists(filename) {
     let file = open(filename)
    defer {
     close(file)
    }
    while let line = try file.readline() {
     // Work with the file.
     }
    // close(file) is called here, at the end of the scope.
    }
}
```

defer문은 현재 코드에서 벗어나기 직전에 실행시키고 싶은 코드가 있을 때 사용하는 키워드로 보통 cleanup을 위해 사용됩니다. defer문은 현재 scope이 끝나기 전까지 실행을 미루었다가 끝나기 직전에 실행이 되는데요, 이때 중간에 에러가 던져져서 return 되거나 break 되는 상황이 발생하더라도 defer의 코드블록은 메서드에서 벗어나기 직전에 실행됩니다. <br>

참고로 defer문은 나중에 쓰여진 defer문이 먼저 실행이 되는, 역순으로 실행된다는 특징이 있습니다. <br>

<br>

#### 참고
- [https://docs.swift.org/swift-book/LanguageGuide/ErrorHandling.html](https://docs.swift.org/swift-book/LanguageGuide/ErrorHandling.html)
- [https://www.youtube.com/watch?v=l0fGtNnNsJg](https://www.youtube.com/watch?v=l0fGtNnNsJg)

<br>
