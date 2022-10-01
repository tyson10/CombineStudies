# Subject

- Non-Combine 코드가 Combine 의 Subscriber 에게 값 전달을 할 수 있게 하는 연결자임.(Publisher 를 상속 받음)
- RxSwift 의 Subject 와도 유사.

---

# PassthroughSubject

- send(_:)를 통해 stream에 값을 주입할 수 있는 publisher
- RxSwift 의 PublishSubject 와 유사

### Example

```swift
example(of: "PassthroughSubject") {
    enum MyError: Error {
        case test
    }
    
    final class StringSubscriber: Subscriber {
        typealias Input = String
        typealias Failure = MyError
        
        func receive(subscription: Subscription) {
            subscription.request(.max(2))
        }
        
        func receive(_ input: String) -> Subscribers.Demand {
            print("Received value", input)
            return input == "World" ? .max(1) : .none
        }
        
        func receive(completion: Subscribers.Completion<MyError>) {
            print("Received completion", completion)
        }
    }
    
    let subscriber = StringSubscriber()
    
    // 5
    let subject = PassthroughSubject<String, MyError>()
    
    // 6
    subject.subscribe(subscriber)
    
    // 7
    let subscription = subject
        .sink(
            receiveCompletion: { completion in
                print("Received completion (sink)", completion)
            },
            receiveValue: { value in
                print("Received value (sink)", value)
            }
        )
    
    subject.send("Hello")
    subject.send("World")
    subject.send("Still there?")
    subject.send(completion: .finished)
    subject.send("How about another one?")
}

——— Example of: PassthroughSubject ———
Received value Hello
Received value (sink) Hello
Received value World
Received value (sink) World
Received value Still there?
Received value (sink) Still there?
Received completion finished
Received completion (sink) finished
```

---

# CurrentValueSubject

- PassthroughSubject 와 달리 초기값, 최근 값에 대한 buffer를 가짐.
- 완료 이벤트 발행 못함.
- RxRelay 의 BehaviorRelay 와 유사.

### Example

```swift
example(of: "CurrentValueSubject") {
    // 1
    let subject = CurrentValueSubject<Int, Never>(0)
    
    // 2
    subject
        .sink(receiveValue: { print($0) })
    
    subject.send(1)
    subject.send(2)
    print(subject.value)
}

——— Example of: CurrentValueSubject ———
0
1
2
2
```