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



