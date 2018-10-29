# Go Channel 사용법

## 비동기 작업 종료 기다리기

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



