---
layout: post
title: ARC, 강한 참조와 약한 참조
category: Swift
tags: [ARC, weak, strong, unowned]
comments: true
---
>이 글은 공부 중에 작성하는 글입니다.       
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

오늘은 약한 참조(weak)과 강한 참조(strong)에 대해서 알아보겠습니다.

```swift
@IBOutlet weak var image: UIImage!
```

@IBOutlet을 만들다보면 이렇게 ```weak``` 키워드를 사용하게 됩니다. weak 말고도 ```strong```이라는 선택지도 있었는데, 이 둘의 차이가 무엇인지 궁금해서 찾아보게 되었습니다. weak을 사용하는 이유는 '메모리 누수'의 문제 때문이더군요! 포스팅은 [공식문서](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)를 토대로 공부한 내용입니다 :)

<br>
<br>

# ARC (Auto Reference Counting)
먼저 ARC에 대해서 알아야할 것 같습니다. Swift는 메모리 사용을 추적하고 관리하기 위해서 ARC를 사용합니다. 즉 ARC가 메모리 관리를 해주기 때문에 우리는 따로 메모리에 대해서 생각하지 않아도 됩니다. ARC는 더 이상 참조가 필요없는 인스턴스를 메모리에서 자동으로 해제해주는 일을 합니다. 사용하지 않는 메모리를 자동으로 제거하는 것은 굉장히 효율적인 일인 것 같네요. <br>

ARC는 Reference Counting으로 메모리에서 할당, 제거를 결정하는데요, 이름처럼 참조타입에 한해서만 기능하기 때문에 클래스에서만 유효합니다. (구조체나 열거형은 값 타입이므로 ARC의 대상이 아닙니다.)

#### ✐ Reference Counting
ARC는 내부적으로 reference count을 관리합니다. 참조가 발생할 때마다 count가 하나씩 늘어나는 것이지요. reference count이 0인 인스턴스의 경우에는 메모리에 할당되지 않습니다. reference count가 0이 아닌 경우, 해당 인스턴스의 reference count가 0이 되도록 nil 값을 할당시켜 메모리에서 해제해주는 식으로 동작합니다. <br>

먼저 아래의 코드로 reference count을 이해해보겠습니다. 참고로 아래는 `강한 참조`의 예시입니다. <br>

#### ✐ 강한 참조 (strong)
```swift
// 인스턴스 생성시, 해제시 문자열을 출력하는 Person 클래스
class Person {
    let name: String
    init(name: String) {
        self.name = name
        print("\(name) is being initialized")
    }
    deinit {
        print("\(name) is being deinitialized")
    }
}

var reference1: Person?
var reference2: Person?
var reference3: Person?

reference1 = Person(name: "John Appleseed")
// Prints "John Appleseed is being initialized"
/*
 reference1에 Person 클래스를 할당하는 것으로 초기화했기 때문에 init의 print가 실행됩니다.
 reference count + 1 = 1
 */

reference2 = reference1
reference3 = reference1
/*
 reference count + 1 = 2
 reference count + 1 = 3
 현재 Person  인스턴스에는 3개의 강한 참조가 존재합니다.
 */

reference1 = nil
reference2 = nil
/*
 nil을 할당하는 것으로 참조를 해제합니다.
 reference count - 1 = 2
 reference count - 1 = 1

 하지만 아직 Person의 reference count가 0이 아니므로 메모리에서 해제되지는 않습니다.
 */

reference3 = nil
// Prints "John Appleseed is being deinitialized"
/*
 마지막 reference3도 nil로 만들어 주어야 Person 인스턴스는 메모리에서 해제되며
 deinit의 print를 호출하게 됩니다.
 */
```

<br>

### ✐ 순환 참조
두 개의 클래스 인스턴스가 서로의 인스턴스를 강하게 참조(강한 참조)하는 경우 순환 참조 (strong reference cycle)이 발생합니다. 즉, 서로가 서로를 참조(소유)하고 있기 때문에 reference count가 0이 되지 않아 메모리에서 해제되지 않는 것이지요. <br>

이러한 문제를 해결하기 위한 것이 약한 참조(weak, unowned)인데요, 먼저 순환 참조에 대해서 코드로 확인해보도록 하겠습니다.

```swift
// deinit 시점에 문자열을 출력하는 Person 클래스와 Apartment 클래스
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { print("\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    var tenant: Person?
    deinit { print("Apartment \(unit) is being deinitialized") }
}
/*
 Person의 apartment 프로퍼티, Apartment의 tanant 프로퍼티는
 서로의 타입을 참조하고 있습니다.
 */

var john: Person?
var unit4A: Apartment?

john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")
/*
 두 변수를 만들어 각각에 Person 인스턴스와 unit4A 인스턴스를 할당해줍니다.
 john은 Person 인스턴스에 대한 강한 참조를, unit4A는 Apartment 인스턴스에 대한 강한 참조를 가지게 됩니다.
 각각의 인스턴스의 reference count에는 1씩 추가되겠네요.
 */

john!.apartment = unit4A
unit4A!.tenant = john
/*
 이제 이 변수들의 프로퍼티에 서로를 할당해줍니다.
 즉, 각각의 프로퍼티에 Person 인스턴스, Apartment 인스턴스가 할당되는 것입니다.
 또 한 번 강한 참조가 발생했으므로 각각의 인스턴스의 reference count에는 1씩 추가되겠네요.
 */

john = nil
unit4A = nil
/*
 그리고 john과 unit4A에 nil을 할당해주면..
 메모리에서 해제되면서 deinit이 호출될 줄 알았는데 그러질 않네요?!

 강한 참조를 해제시켰지만 아직 각각의 인스턴스에는
 reference count는 1이 남아있기 때문에 메모리에서 해제되지 않은 것입니다.

 만약,
 john!.apartment = nil
 unit4A!.tenant = nil

 john = nil
 unit4A = nil

 이 순서로 코드를 작성한다면 deinit이 호출되는 것을 확인할 수 있습니다.
 메모리에서 해제되는 것이지요.
 */

```

따라서 이렇게 순환참조의 경우에는 **필요없는 인스턴스가 메모리에 남아있는** 아주 비효율적인 상황이 발생할 수 있습니다. 이를 `메모리 누수`라고 하기도 합니다. <br>

이러한 메모리 누수를 해결하기 위한 방법으로 weak과 unowned이라는 `약한 참조`가 있습니다. <br>

>##### 강한 참조 (strong)
>- 해당 인스턴스의 소유권을 가짐
>- 참조 발생시, 자신이 참조하는 인스턴스의 reference count를 1씩 증가시킴
>- 선언할 때 아무것도 적어주지 않으면 strong타입          
    ex) var person = Person() 
>##### 약한 참조 (weak), unowned
>- 해당 인스턴스의 소유권을 가지지 않음
>- reference count를 증가시키지 않음              
    ex) weak var person = Person()            
    ex) unowned var person = Person()              

<br>

### ✐ weak (약한 참조)
말 그대로 '약한' 참조입니다. 강한 참조와는 다르게 약한 참조가 참조하는 동안 해당 인스턴스가 할당 해제될 수 있습니다. reference count를 증가시키지 않는 참조로도 이해할 수 있습니다. 약한 참조의 경우 참조하고 있는 인스턴스가 nil이 될 경우, 해당 인스턴스의 값도 nil로 변경됩니다.

>약한 참조의 경우 할당값이 nil로 변경될 여지가 있기 때문에 상수가 아닌  변수로만 선언해줄 수 있습니다. 또한 약한 참조가 nil이  되는 경우, 프로퍼티 옵저버는 호출되지 않습니다.

```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { print("\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    weak var tenant: Person?
    // 여기 주목
    deinit { print("Apartment \(unit) is being deinitialized") }
}

var john: Person?
var unit4A: Apartment?

john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")

john!.apartment = unit4A
unit4A!.tenant = john
/*
 unit4A!.tenant가 Person 인스턴스를 약하게 참조하고 있으므로
 이때 Person 인스턴스의 reference count는 1이 됩니다.
 반면 Apartment 인스턴스의 reference count는 2입니다.
 */

john = nil
// Prints "John Appleseed is being deinitialized"
/*
 Person 인스턴스인 john에 nil을 할당해주면 reference count는 0이 되면서 deinit이 호출됩니다.

 만약 john = nil 대신,
 unit4A!.tenant에 nil을 할당해주면
 Apartment 인스턴스의 reference count는 아직 1이 남아있으므로 deinit이 호출되지 않습니다!
 */

unit4A = nil
/*
 john = nil에 이어서  unit4A에 nil을 할당해주면
 Apartment 인스턴스의 reference count가 0이 되므로 Apartment deinit이 호출됩니다.
 */
```

<br>

### ✐ unowned
unowned는 약한 참조처럼 인스턴스를 강하게 참조하지 않습니다. 하지만 약한 참조와 다르게, unowned 참조는 다른 인스턴스가 더 길거나 같은 생명주기를 가질 때 사용합니다. <br>

공식문서에 따르면 unowned에는 옵셔널이나 nil을 할당할 수 없습니다. 그러므로 할당이 해제되지 않을 인스턴스를 참조하는 경우에만 사용하고, 만약 해당 인스턴스의 할당이 해제된 후 unowned 인스턴스에 접근하면 런타임 에러가 발생한다고 합니다. (하지만 unowned optional을 사용하면 옵셔널과 nil을 할당할 수 있답니다. 이 내용은 아래에서 다룹니다.)

```swift
class Customer {
    let name: String
    var card: CreditCard?
    // 고객은 카드가 있을 수도, 없을 수도 있습니다.
    init(name: String) {
        self.name = name
    }
    deinit { print("\(name) is being deinitialized") }
}

class CreditCard {
    let number: UInt64
    unowned let customer: Customer
    // 카드는 주인이 반드시 있습니다.
    init(number: UInt64, customer: Customer) {
        self.number = number
        self.customer = customer
    }
    deinit { print("Card #\(number) is being deinitialized") }
}

var john: Customer?

john = Customer(name: "John Appleseed")
john!.card = CreditCard(number: 1234_5678_9012_3456, customer: john!)
/*
 이때 Customer 인스턴스의 reference count는 1,
 CreditCard 인스턴스의 reference count는 1이 됩니다.
 하지만 CreditCard의 customer 프로퍼티는 Customer을 unowned 참조하고 있는 인스턴스가 됩니다.
 */

john = nil
// Prints "John Appleseed is being deinitialized"
// Prints "Card #1234567890123456 is being deinitialized"
/*
 그리고 john에 nil을 할당해주면 Customer과 CreditCard 인스턴스 모두 할당이 해제되어 deinit이 호출됩니다.
 그냥 보아서는 weak 약한 참조와 같은 결과로 보입니다만,
 프로퍼티를 let으로 선언했다는 점, 옵셔널이 아닌 타입을 타입으로 갖는다는 점에서 차이가 있네요.
 만약 unowned를 약한 참조(weak)로 만들어주려면
 weak var customer: Customer?
 이렇게 작성해주어야 합니다.
 */
```

### ✐ unowned optional
위에서 unowned에는 optional, nil을 할당할 수 없다고 했는데요, unowned optional이라는 개념이 바로 아래에 추가되었더군요. 그러니까 unowned에도 옵셔널과 nil을 할당할 수 있다는 이야기입니다!! 공식 문서는 왜 이렇게 헷갈리게 적어뒀을까요 ㅜ 이걸로 한참을 고민했네요.. 굳이 이 설명의 구조를 이해한다면.. 'Int에는 nil을 할당할 수 없지만 Int?에는 nil을 할당할 수 있다.' 이런 맥락인 것 같아요. <br>

```swift
class Department {
    var name: String
    var courses: [Course]
    init(name: String) {
        self.name = name
        self.courses = []
    }
}

class Course {
    var name: String
    unowned var department: Department
    unowned var nextCourse: Course?
    init(name: String, in department: Department) {
        self.name = name
        self.department = department
        self.nextCourse = nil
    }
}

let department = Department(name: "Horticulture")

let intro = Course(name: "Survey of Plants", in: department)
let intermediate = Course(name: "Growing Common Herbs", in: department)
let advanced = Course(name: "Caring for Tropical Plants", in: department)

intro.nextCourse = intermediate
intermediate.nextCourse = advanced
department.courses = [intro, intermediate, advanced]
```
<br>

### ✐ 그러면 도대체 weak과 unowned는 무슨 차이가 있을까요?
위의 예제들에서 weak대신 unowned을 사용해도, unowned대신 weak을 사용해도 같은 결과가 나오는데요, 그렇다면 도대체 이 둘은 어떤 차이가 있는걸까요? <br>

#### - unowned은 상수(let)으로 선언할 수 있다.
unowned은 weak과 다르게 상수로 선언할 수 있습니다. 또한 옵셔널(?)이 필수인 weak과 달리 옵셔널이 아닌 타입으로 선언할 수 있습니다. 하지만 그렇기 때문에 nil이 발생하면 런타임 에러가 발생하지요. 하지만 이  부분은 중요한 차이라고 할 수 없어보이는데요, 왜냐면 이제는 unowned에도 옵셔널을 할당할 수 있기 때문이지요. 그러면 weak과 unowned optional은 똑같은 것일까요?? <br>

#### - Lifetime
weak과 unowned optional은 굉장히 유사해 보입니다. 생긴  모양도, 기능도 비슷하기 때문이지요. weak을 사용하면 될텐데, 굳이 Swift에 unowned optional을 추가한 이유는 무엇일까요? weak과 unowned optional은 클래스의 생명 주기(Lifetime)와 관련하여 차이를 보입니다. ([stackoverflow](https://stackoverflow.com/questions/54852878/optional-unowned-reference-versus-weak-in-swift-5-0)를 참고했습니다.) <br>

위에서 설명했듯이 unowned은 참조할 다른 인스턴스의 생명주기가 더 길거나, 같을 때 사용합니다. 즉, 참조할 인스턴스의 생명주기가 더 길다는 것이 자명한 상황이라면 unowned optional이 weak보다 유리합니다. <br>

unowned optional이 없을 때에는 그냥 weak으로 선언하는 것이 유일한 방법이었는데요, 하지만 이 경우에는 불필요한 오버헤드를 수반한다고 합니다. <br>

>오버헤드(overhead)는 어떤 처리를 하기 위해 들어가는 간접적인 처리 시간 · 메모리 등을 말한다. - 위키피디아

<br>
<br>

# 다시 @IBOutlet으로
ARC 내용이 많아서 돌고 돌아왔네요. 다시 처음으로 돌아가서 @IBOutlet을 연결해줄 때 왜 strong(강한 참조)이 아닌 weak(약한 참조)을 사용하는 것일까요? <br>

우리가 라이브러리에서 가져오는 UIKit들은 모두 Class입니다. 그러니까 Button, Label 등을 할당하는 것은 클래스를 참조하는 것이지요. 예를 들어 UIButton 클래스를 강한 참조하게 된다면 해당 프로퍼티가 인스턴스를 소유하므로 ViewController가 dealocate 되더라도 메모리에서는 UIButton이 해제되지 않게 됩니다. 따라서 ViewController에 UIKit의 클래스들을 가져올 때에는 약한 참조를 사용하는 것입니다. <br>

```swift
@IBOutlet var storngButton: UIButton!
@IBOutlet weak var weakButton: UIButton!

func removeButton() {
    storngButton.removeFromSuperview() // 메모리에서 해제 안됨
    weakButton.removeFromSuperview() // 메모리에서 해제
}
```




<br>
<br>

#### 참고
- [https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html](https://docs.swift.org/swift-book/LanguageGuide/AutomaticReferenceCounting.html)
- [https://shark-sea.kr/entry/iOS-ARC-strong-weak-unowned?category=785515](https://shark-sea.kr/entry/iOS-ARC-strong-weak-unowned?category=785515)
- [https://stackoverflow.com/questions/54852878/optional-unowned-reference-versus-weak-in-swift-5-0](https://stackoverflow.com/questions/54852878/optional-unowned-reference-versus-weak-in-swift-5-0)
- [https://daheenallwhite.github.io/speaking/swift/2019/06/28/ARC-Memory-Management-Swift/](https://daheenallwhite.github.io/speaking/swift/2019/06/28/ARC-Memory-Management-Swift/)
<br>
<br>
