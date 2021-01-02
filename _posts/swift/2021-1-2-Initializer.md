---
layout: post
title: Initializer (init)
category: Swift
tags: [Initializer, init, 초기화]
comments: true
---

>이 글은 공부 중에 작성하는 글입니다.
>정정이 필요한 내용은 댓글로 알려주시면 감사하겠습니다 :)

<br>

안녕하세요 오동나무입니다.  <br>

오늘은 초기화 구문, initializer에 대한 내용입니다. init 구문이라고 하면 더 눈에 익는 것 같네요 :) 주로 [공식문서](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html#ID222)를 참고하여 작성하는 내용입니다. (내용이 꽤  많습니다!)

<br>
<br>

# Initializer (초기화)
구조체와 클래스는 정의한다고 해서 바로 사용할 수 있는 것은 아닙니다. 반드시 '**초기화**'를 해주어야 하는데요, 초기화란 변수나 상수, 프로퍼티에 값을 할당하는 것을 말합니다. (열거형에도 초기화를 사용할 수 있습니다!) 즉 클래스, 구조체, 열거형 등을 사용하기 위한  준비 과정이라고 할 수 있겠네요. <br>

```swift
class someClass {
    var number: Int
    var name: String
}// error! - Class 'someClass' has no initializers
```

이렇게 number, name이라는 프로퍼티를 '선언'은 했지만, 값을 할당하는 초기화는 이루어지지 않았기 때문에 에러가 납니다. 이런 에러는 구조체에서는 볼 수 없고, 오직 클래스에서만 볼 수 있는데요, 구조체에서는 '**멤버 와이즈**' 라는 기능을 제공하기 때문입니다.  <br>

```swift
struct SomeStruct1 {
    var number: Int
    var name: String
}

let struct1 = SomeStruct1(number: 5, name: "오동나무")

// 값이 이미 있는 프로퍼티와 memberwise
struct SomeStruct2 {
    var number: Int = 10
    var name: String = "odongtree"
}

let struct2 = SomeStruct2(name: "5동나무")
struct2.number // 10
struct2.name // "5동나무"

let struct3  = SomeStruct2(number: 99, name: "아낌없이 주는 오동나무")
struct3.number // 99
struct3.name // "아낌없이 주는 오동나무"
```

여기서 주의할 점은 만약 구조체 안에 initializer를 임의로 추가해주는 경우에는 Default initializer, 즉 memberwise 기능을 사용할 수 없다는 것입니다!! <br>

구조체에는 멤버와이즈 기능이 있어서 인스턴스를 생성하는 시점에 초기화가 이루어져도 되지만, 클래스의 경우에는 프로퍼티 선언과 동시에 값이 할당하거나, 내부에 init (초기화 구문)을 반드시 작성해주어야 합니다. <br>

> ##### memberwise, 클래스에는 없고 구조체에만 있는 이유는 무엇일까요?
> 클래스에는 상속 기능이 있기 때문입니다. 자식클래스를 통해 상속받은 값에 접근하거나 값을 변경해주기 위해서는 부모클래스의 값에 먼저 접근해야합니다. 따라서 구조체의 memberwise처럼 자유롭게 접근하고 값을 변경하는 것은 불가능하지요. 에러가 발생할 것입니다. 자식클래스의 경우 상속받은 멤버에 대해서 자유롭지 못하기 때문에 memberwise를 제공하기 보다는 직접 initializer를 구현하는 것이 보다 적합한 것 같네요.

<br>

### ✐ init()
```swift
class SomeClass1 {
    var number: Int = 0
    var name: String = ""
}

let class1 = SomeClass1()
// 여기서 SomeClass1()의 빈 괄호는 init()의 축약입니다.
```

```swift
class SomeClass2 {
    var number: Int
    var name: String

    init(number: Int, name: String) {
        self.number = number
        self.name = name
    }
}

let class2 = SomeClass2(number: 5, name: "오동나무")
// 여기서도 역시 init은 축약되어 있습니다.
```

첫 번째 예시의 경우에는 선언과 동시에 초기화를 해주었기 때문에 initializer를 직접 구현하지 않아도 됩니다. 선언하는 시점에 모든 프로퍼티의 값이 정해져있거나, 내부적으로 초기화를 할 필요가 없는 경우에는 Default Initializers를 제공하기 때문입니다. 하지만 두 번째 예시의 경우 선언하는 시점에 초기화가 이루어지지 않았으므로 그 프로퍼티에 대한 초기화 구문을 작성해주어야 합니다. <br>

여기서 init 메소드 속에 self가 붙은 number, name은 클래스의 프로퍼티를 말하고, 뒤에 할당되는 number, name에는 매개변수로, 초기화시 전달인자의 값이 들어가게 됩니다.

<br>

### ✐ init구문 Overloading
init()도 메서드이기 때문에 오버로딩(Overloading)할 수 있습니다. 오버로딩은 같은 메서드명에 파라미터 구성을 달리하여 다른 메서드를 만드는 것을 말합니다. 필요한 만큼 만들어서 용도에 맞게 사용할 수 있겠네요! <br>

```swift
class SomeClass {
    var number: Int = 0
    var name: String = ""
    var height: Int = 0
    var width: Int = 0

    init(number: Int, name: String) {
        self.number = number
        self.name = name
    }

    init(_ height: Int) {
      // 언더바를 사용하면 초기화시 파라미터를 생략할 수 있습니다.
        self.height = height
    }

    init(number: Int, height: Int, width: Int) {
        self.number = number
        self.height = height
        self.width = width
    }
}
```

초기화 구문을 만들때에는 초기화가 되지 않는 멤버가 있는지를 잘 확인해야합니다. 이렇게 오버로딩을 해주는 경우에는 특히 신경을 써줄 필요가 있겠네요 :)

<br>
<br>

# 클래스의 상속과 초기화
클래스는 Value Type의 구조체와 열거형과는 다르게 **상속**이 가능하다는 특징이 있습니다. 그렇기 때문에 memberwise를 제공하지 않기도 하며, 초기화에도 더 신경써주어야 합니다. <br>

클래스의 initializer에는 **Designated initializer** 와 **Convenience initializer** 가 있습니다.
<br>

### ✐ Designated initializer
```swift
init(parameters) {
    statements
}
```

먼저 Designated initializer는 앞에 아무 키워드도 붙지 않는, 우리에게 익숙한 init입니다. Designated initializer에서는 초기화되지 않은 모든 프로퍼티들의 초기화가 이루어져야 하며 모든 클래스는 Designated initializer를 하나 이상 가지게 됩니다. 또한 적절하게 부모클래스의 init을 호출하는 것으로 클래스의 상속에 있어서 연쇄적인 init을 가능하게 해주기도 합니다.
<br>

### ✐ Convenience initializer
```swift
convenience init(parameters) {
    statements
}
```
Convenience initializer는 Designated initializer를 보조해주는 initializer입니다. 클래스의 의도를 명확하게 하거나 초기화 패턴을 단축할 때 사용합니다. 어떻게 사용하는지  볼까요? <br>

```swift
class SomeStruct1 {
    var number: Int
    var name: String

    init(number: Int, name: String) {
        self.number = number
        self.name = name
    }

    convenience init(number: Int) {
        self.init(number: number, name: "odong")
    }
}
```

이렇게 클래스 내부에서 정의된 initializer를 재사용할 수 있게 도와주는 녀석입니다. 만약 여기서 convenience 키워드를 삭제한다면 오류가 발생하게 됩니다.

<br>

>#### Initializer Delegation for Class Types
>공식문서에서는 클래스의 Initializer에 대한 3가지 규칙을 제공하고 있네요.
>1. designated initializer는 바로 직전 부모클래스의 designated initializer을 호출해야 한다.
>2. convenience initializer의 내부에는 같은 클래스 내부의 initializer만 호출할 수 있다.
>3. convenience initializer에서는 결국 designated initializer를 호출해야 한다.

<br>

### ✐ Two-Phase Initialization
공식문서에 따르면 클래스의 초기화 과정은 2개의 과정으로 진행된다고 합니다. 두 단계를 거치는 것으로 클래스의 초기화는 안전하게, 유연하게 이루어질 수 있습니다. <br>

이 두 단계의 초기화 과정을 거치기 위해 Swift 컴파일러는 4단계의 안전 검사(Safety-Check)를 하게 됩니다.

#### Safety-Check
1. 지정 이니셜라이저는, 슈퍼클래스의 이니셜라이저에게 초기화 과정을 위임delegate하기 전에, 일단 자기 클래스에서 도입된 모든 프로퍼티가 초기화 되었음을 보장해야 한다.

2. 지정 이니셜라이저는, 상속받은 프로퍼티에 값을 할당하기 전에, 슈퍼 클래스의 이니셜라이저에게 초기화 과정을 위임해야 한다.

3. 편의 이니셜라이저는, 프로퍼티에 값을 할당하기 전에, 다른 이니셜라이저에게 초기화 과정을 위임해야 한다.

4. 초기화 1단계가 끝나기 전까지는, 이니셜라이저는 인스턴스 메서드를 호출할 수 없고, 인스턴스 프로퍼티의 값을 읽을 수도 없고, self를 참조할 수도 없다.

<br>


그리고 클래스의 초기화는 다음의 두 과정을 거쳐 진행됩니다. <br>

#### Phase 1
- designated initializer 혹은, convenience initializer 호출한다.
- 객체 인스턴스에 대한 Memory allocation을 수행합니다. 다만 이 때는 stored properties에 들어가는 데이터의 크기를 모르기 때문에 정확한 메모리 크기가 설정되어 있지 않다.
- designated initializer가 모든 stored properties가 설정되었는지 체크 후, property들에 대한 메모리 init을 수행한다.
- 이제 subClass의 작업을 마치고 superClass에서 위의 과정과 동일한 작업을 수행한다.
- 계층 구조 최상단 superClass가 모든 properties가 값이 있는지 확인 후, init 작업이 완료된다.
#### Phase 2
- Phase 1을 마치면 self에 접근할 수 있게 되고, 최상단 superClass는 property 값을 변경할 기회를 얻게 된다.
- 값 변경의 기회는 계층 구조 최상단 superClass부터 마지막 subClass의 순서로 주어진다.
- 현재 클래스에서 init을 호출한 것이 convenience initializer라면, designated initializer부터 우선적으로 self의 값을 변경할 기회를 얻게 됩니다. 이 작업을 마치면 최종적으로 convenience initializer가 값을 변경할 기회를 얻게 된다.

<br>

### ✐ initializer 상속과 오버라이딩
Swift에서 상속이 이루어지는 경우, 특정 조건이 만족되면 initializer를 자동으로 상속받고, 그렇지 않은 경우, 자식클래스는 initializer를 오버라이딩(override)해주어야 합니다. <br>

#### Initializer 자동 상속
다음 두 가지 경우에는 자식클래스가 initializer를 오버라이딩하지 않아도 자동으로 부모 클래스의 initializer들을 상속받게 됩니다.

1. 만약 자식클래스가 어떠한 designated initializer도 정의하지 않을 경우, 부모클래스의 모든 initializer를 상속받는다.
2. 만약 자식클래스에서 부모클래스의 모든 designated initializer를 구현한다면 (1번의 경우처럼 모든 initializer를 상속받거나, 혹은  모두  직접 구현한다거나), 부모클래스의 convenience initializer들을 자동으로 상속받는다.
<br>

#### 오버라이딩 (overriding)
위의 자동 상속이 가능한 경우가 아니라면, 자식클래스에서 부모클래스의 initializer를 가지게 하고 싶다면 직접 오버라이딩해야 합니다. <br>

메서드의 오버라이딩과 같이 override 수식어를 붙여서 구현합니다. 그리고 그 안에서 부모클래스의 지정 이니셜라이저를 불러준다. 이때의 부모클래스의 designated initializer가 자동으로 만들어진 Default iinitializer인 경우에도 예외는 아닙니다. <br>

>**부모클래스의 Convenience Initializer를 오버라이딩?**
자식클래스가 부모클래스의 convenience initializer를 오버라이딩한다고 하더라도, convenience initializer는 해당 클래스의 initializer에 대한 기능이기 때문에 사실상 부모클래스의 convenience initializer는 오버라이딩을 할 수 없으며, 의미가 없습니다.

```swift
class Vehicle {
    var numberOfWheels = 0
    var description: String {
        return "\(numberOfWheels) wheel(s)"
    }
}

let vehicle = Vehicle()
print("Vehicle: \(vehicle.description)")
// Vehicle: 0 wheel(s)

class Bicycle: Vehicle {
    override init() {
        super.init()
        numberOfWheels = 2
    }
}

let bicycle = Bicycle()
print("Bicycle: \(bicycle.description)")
// Bicycle: 2 wheel(s)
```

<br>

### ✐ Required Initializers
initializer 앞에 required 키워드를 붙여주면 필수적으로 구현해야하는 initializer가 됩니다. 따라서 override 키워드는 생략할 수 있습니다. <br>

```swift
class SomeClass {
    required init() {
        // initializer implementation goes here
    }
}

class SomeSubclass: SomeClass {
    required init() {
        // subclass implementation of the required initializer goes here
    }
}
```

<br>
<br>

# Failable Initializers
Failable Initializers는 초기화가 실패할 경우에 대한 초기화입니다. **init?, init!** 의  모양으로 옵셔널과 매우 닮아 있습니다. 그 개념 역시 유사합니다. <br>

초기화의 목적은 값 리턴이 아니기 때문에 초기화가 실패했을 때 return nil을 해야하긴 하지만, 초기화가 성공했을 때도 무언가를 리턴할 필요는 없습니다. <br>

```swift
struct Animal {
    let species: String
    init?(species: String) {
        if species.isEmpty { return nil }
        self.species = species
    }
}

let someCreature = Animal(species: "Giraffe")
if let giraffe = someCreature {
    print("An animal was initialized with a species of \(giraffe.species)")
}
// Prints "An animal was initialized with a species of Giraffe"

let anonymousCreature = Animal(species: "")
if anonymousCreature == nil {
    print("The anonymous creature could not be initialized")
}
// Prints "The anonymous creature could not be initialized"
```
<br>

>**Failable Initializers의 오버라이딩**
Failable Initializer도 마찬가지로 오버라이딩 할 수 있습니다. 그리고 init? 을 init으로 nonfailable initializer로 오버라이딩 할 수도 있습니다. 하지만 이 경우에는 옵셔널 해제에 대해서 고려해야 합니다. 내부에서 if-else 구문을 사용한다거나 강제해제하는 식으로 오버라이딩 할 수 있습니다.




<br>
<br>

#### 참고
- [https://docs.swift.org/swift-book/LanguageGuide/Initialization.html](https://docs.swift.org/swift-book/LanguageGuide/Initialization.html)
- [https://www.hackingwithswift.com/quick-start/understanding-swift/why-dont-swift-classes-have-a-memberwise-initializer](https://www.hackingwithswift.com/quick-start/understanding-swift/why-dont-swift-classes-have-a-memberwise-initializer)
- [https://hcn1519.github.io/articles/2019-02/swift-init-class-deep](https://hcn1519.github.io/articles/2019-02/swift-init-class-deep)
- [https://wlaxhrl.tistory.com/48](https://wlaxhrl.tistory.com/48)

<br>
<br>
