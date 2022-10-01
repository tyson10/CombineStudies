# Publisher & Subscribers

# Publisher

- 한 개 이상의 구독자에게 시간이 지남에 따라 일련의 값을 전송할 수 있는 유형에 대한 요구 사항을 정의하는 프로토콜임.
- 0개 이상의 값을 보낼 수 있지만, 단 한 개의 완료 이벤트만을 보낼 수 있음(Completed or Error).
- 완료 이벤트를 보낸 이후에는 값을 보낼 수 없음.
- 구독자가 한 개 이상 있어야만 값을 방출함.

---

# Subscriber

- Publisher로부터 입력을 수신할 수 있는 유형에 대한 요구 사항을 정의하는 프로토콜임.

## Subscriber with sink(_: _:)

- Publisher 로 부터 받아온 값을 클로저로 연결함.
- RxSwift 의 bind(onNext:), subscribe(onNext: …) 와 유사

### Example

```swift
example(of: "Just") {
  // 1
  let just = Just("Hello world!")
  
  // 2
  _ = just
    .sink(
      receiveCompletion: {
        print("Received completion", $0)
      },
      receiveValue: {
        print("Received value", $0)
    })
}

——— Example of: Just ———
Received value Hello world!
Received completion finished’
```

## Subscribing with assign(to:on:)

- KVO 호환 속성에 바로 연결 가능하도록 하는 메소드
- RxSwift 의 bind(to:) 와 유사

### Example

```swift
example(of: "assign(to:on:)") {
  // 1
  class SomeObject {
    var value: String = "" {
      didSet {
        print(value)
      }
    }
  }
  
  // 2
  let object = SomeObject()
  
  // 3
  let publisher = ["Hello", "world!"].publisher
  
  // 4
  _ = publisher
    .assign(to: \.value, on: object)
}

——— Example of: assign(to:on:) ———
Hello
world!
```

---

# Cancellable

- 더 이상 값을 수신 할 필요가 없는 구독자는 불필요하게 구독을 유지 할 필요가 없음.
- 구독은 AnyCancellable의 인스턴스를 "취소 토큰"으로 반환하며, AnyCancellable은 그 목적을 위해 정확히 cancel() 메서드가 필요한 Cancellable 프로토콜을 준수함.
- Subscriber 클래스들은 Cancellable 을 준수함.

---

# Understanding what’s going on

![스크린샷 2022-09-18 오후 2.15.04.png](Publisher%20&%20Subscribers%20f9a3d1e9427d4f8fa6aeb52d0c873918/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2022-09-18_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_2.15.04.png)

1. Subscriber 는 Publisher 를 구독함.
2. Publisher 가 Subscriber 에게 구독을 생성해서 줌.
3. Subscriber 가 값을 요청함.
4. Publisher 가 값을 보냄.
5. Publisher 가 completion 을 보냄.

### Example

```swift
import Combine

public func example(of description: String,
                    action: () -> Void) {
  print("\n——— Example of:", description, "———")
  action()
}

example(of: "Custom Subscriber") {
    let publisher = (1...6).publisher
    
    final class IntSubscriber: Subscriber {
        typealias Input = Int
        typealias Failure = Never
        
        func receive(subscription: Subscription) {
            subscription.request(.max(3))
        }
        
        func receive(_ input: Int) -> Subscribers.Demand {
            print("Received value", input)
            return .none
        }
        
        func receive(completion: Subscribers.Completion<Never>) {
            print("Received completion", completion)
        }
    }
    
    let subscriber = IntSubscriber()
    publisher.subscribe(subscriber)
}

——— Example of: Custom Subscriber ———
Received value 1
Received value 2
Received value 3
```