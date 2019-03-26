# Go Channel 사용법

{% embed url="https://www.ardanlabs.com/blog/2017/10/the-behavior-of-channels.html" %}

{% embed url="https://medium.com/justforfunc/why-are-there-nil-channels-in-go-9877cc0b2308" %}

채널을 일종의 큐라고 생각하기보다는 말 그대로 신호를 보내는 채널 그 자체로 생각하는게 좋은 코드를 짜는데 도움이 됨. 즉, 채널이 어떻게 만들어져있는지에 신경쓰지말고 어떻게 동작하는지만 생각하라는 것.

채널을 잘 쓰기 위해서는 세 가지 요소에 대해서 고려해야 함

1. 전송 보장
2. 채널 상태
3. 데이터 여부

#### 전송 보장

누군가 메세지를 보내면 받는 사람이 확실하게 받았다고 다시 알려줄 필요가 있는 경우가 있음. 이 경우 보내는 사람은 받는 사람이 받았다는 연락을 보낼 때 까지 기다려야 함. 전송 보장은 이런 케이스를 의미함.

 go channel에서는 이러한 케이스를 buffered channel과 unbuffered channel로 구분해서 구현할 수 있음. 전송보장이 필요하다면 unbuffered channel로, 그렇지 않은 경우라면 buffered channel을 사용하면 됨. unbuffered channel을 쓰면 receiver가 channel에서 신호를 수신할 때 까지 blocking이 걸리기 때문. 반면에 buffered channel을 쓰면 send와 receive가 정확하게 동기화되지 않음.

#### 채널 상태

채널에는 nil, open, closed 세가지 상태가 존재함. 그냥 `var ch chan struct{}` 로 선언만 하면 nil, 명시적으로 `ch := make(chan struct{})` 로 생성하면 open, `close(ch)` 를 호출하면 closed 상태가 됨.

nil 상태에서는 모든 send, receive 요청이 blocking이 됨. close요청을 하면 panic이 발생함. 무한 blocking이 필요한 부분에서 사용. 주로 for + select 문에서 여러 채널을 동시에 receive 할 때 어느 한 채널이 closed 상태로 바뀌어도 busy loop가 되는 것을 방지.

open 상태에서는 buffer에 따라 blocking 여부가 달라지긴 하겠지만 일단 send, receive가 모두 허용됨

closed 상태에서는 send 요청이 들어오면 panic이 발생함. 하지만 receive는 가능하며 receive 요청을 할 경우 buffer에 남아있는 값들을 blocking 없이 바로바로 돌려줌. unbuffered거나 buffer의 값을 모두 돌려준 다음부터는 제로값을 돌려줌

#### 데이터 여부

데이터와 함께 신호를 보내는건. `ch <- "str"` 와 같은 형태를 말함. 이 케이스는 보통 goroutine에 새로운 작업 요청을 보내거나, goroutine이 작업 결과를 돌려줄 때 사용함.

데이터와 함께 신호를 보내는 것도 unbuffered, buffer 크기가 여러개인 경우, buffer 크기가 1인 경우로 나눌 수 있는데 위에서 언급한 전송 보장과 같은 맥락임. 매번 데이터가 엄격하게 동기화되어야하는 경우는 unbuffered, 약간 시간은 지연되도 좋지만 한번에 한개씩 해야 한다면 buffer 크기를 1개로. 동기화할 필요가 없다면 buffer 크기를 그 이상으로 잡으면 된다.

데이터 없이 신호를 보내는건 채널을 종료하는 `close(ch)` 를 말하는 것. 이 케이스는 보통 goroutine에게 작업을 종료하라고 신호를 보낼 때, goroutine이 결과 없이 작업을 모두 끝마쳤다고 알려올 때 사용함. 이렇게 데이터 없이 신호를 보내는 것의 장점은 receive하고 있는 모든 goroutine에게 한방에 알려줄 수 있다는 점. 만약 데이터와 함께 보내려고 한다면 receive하고 있는 goroutine의 수만큼 send를 보내야 함.

데이터 없이 신호를 보내기 위해 채널을 만든다면 보통은 `context.Context` 를 쓰는게 좋음. 내부적으로 채널과 close를 써서 구현되어있으며 timeout같은 여러 편리한 기능들이 추가되어있음. 그렇지 않고 직접 채널을 만들어 쓰겠다면 `chan struct{}` 를 쓰는게 정석임. 공간을 차지하지 않고 코드를 읽는 사람에게 이 채널은 데이터를 보내는데 쓰지 않는다는 사실을 효과적으로 알려줌. 만약에 buffered channel을 이 용도로 쓰고 있다면 다시 한번 생각해볼 필요가 있음.

## 예제 - 비동기 작업 종료 기다리기

### 고루틴이 한개인 경우

```go
func example() bool {
    done := make(chan struct{})
    go func() {
        // jobs
        close(done)
    }()

    select {
    case <-done:
    }

    return true
}
```

### 고루틴이 여러 개인 경우

```go
func example2() bool {
    wg := sync.WaitGroup{}

    wg.Add(2)

    go func() {
        defer wg.Done()
        // jobs 1
    }()

        go func() {
        defer wg.Done()
        // jobs 2
    }()

    wg.Wait()

    return true
}
```

### Timeout이 필요한 경우

```go
func exampleTimeout() bool {
    done := make(chan struct{})
    go func() {
        // jobs
        close(done)
    }()

    select {
    case <-done:
    case <-time.After(10 * time.Second)
      return false
    }

    return true
}
```

### 고루틴이 여러개인데 Timeout도 필요한 경우

```go
func example2Timeout() bool {
    wg := sync.WaitGroup{}

    wg.Add(2)

    go func() {
        defer wg.Done()
        // jobs 1
    }()

    go func() {
        defer wg.Done()
        // jobs 2
    }()

    done := make(chan struct{})
    go func() {
        wg.Wait()
        close(done)
    }()

    select {
    case <-done:
    case <-time.After(10 * time.Second):
        return false
    }

    return true
}
```

## 데이터 수신하기

### 채널 닫기로 종료

```go
func example(receiveCh <-chan string) bool {
    go func() {
    Loop:
        for {
            select {
            case data := <-receiveCh:
                // job with data
            default:
                break Loop
            }
        }
    }()

    return true
}
```

```go
func example(receiveCh <-chan string) bool {
    go func() {
        for data := range receiveCh {
            // job with data
        }
    }()

    return true
}
```

`receiveCh` 를 종료해서 수신 종료

### 종료를 알려주는 별도 채널 사용

```go
func example(receiveCh <-chan string, stopCh <-chan int) bool {
    go func() {
    Loop:
        for {
            select {
            case data := <-receiveCh:
                // job with data
            case <-stopCh:
                break Loop
            }
        }
    }()

    return true
}
```

