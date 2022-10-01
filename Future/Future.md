# Future

- Future 는 단일 결과를 비동기적으로 생성한 다음 완료하는 데 사용함.
- Future는 one-shot임. 즉 promise에게 value나 error를 한번 보내고 나면 바로 finished 됨.

### Example

```swift
example(of: "Future") {
    func futureIncrement(integer: Int, afterDelay delay: TimeInterval) -> Future<Int, Never> {
        Future<Int, Never> { promise in
            DispatchQueue.global().asyncAfter(deadline: .now() + delay) {
                promise(.success(integer + 1))
            }
        }
    }
    
    let future = futureIncrement(integer: 1, afterDelay: 3)
    future
        .sink(receiveCompletion: { print($0) },
              receiveValue: { print($0) })
}

——— Example of: Future ———
2
finished
```