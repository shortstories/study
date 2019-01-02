# Go Error handling

### 참고

1. [https://evilmartians.com/chronicles/errors-in-go-from-denial-to-acceptance?fbclid=IwAR2QLLZ4KrPPX7wQk5faERPOqMQNm4znUH7rvTLTw0GeZiYPnrR80TFWzbE](https://evilmartians.com/chronicles/errors-in-go-from-denial-to-acceptance?fbclid=IwAR2QLLZ4KrPPX7wQk5faERPOqMQNm4znUH7rvTLTw0GeZiYPnrR80TFWzbE)

### 일반적 사용법 \(Go way\)

```go
result, err := someFunction()
if err != nil {
  // handle err
}

//or

if _, err := someFunction(); err != nil {
  // handle err
}
```

go에서 권장하는 방법이고 가장 많이 쓰게 될 방법.

문제는 위와 같은 코드가 너무. 너무 많이 나온다는 것. 죄다 중복 코드인데;

거기에다가 에러 인스턴스에서 얻을 수 있는 데이터가 오직 에러 메세지 뿐임. 스택 트레이스를 보려면 `panic` 을 쓰던지 아니면 직접 구현해야되는 것도 짜증난다.

### Custom error 만들어서 쓰기

```go
type error interface {
  Error() string
}
```

error는 이렇게 엄청 단순한 인터페이스다. 따라서 이것만 가지고 적절하게 에러핸들링을 하는건 거의 불가능에 가까우므로 사용자가 직접 구현한 error struct를 만들어서 쓰는게 낫다. 내지는 \`github.com/juju/errors\` 같은 라이브러리를 쓰던지.

```go
type MyError struct {
    Cause MyError
    Msg string
}
func (m *MyError) Error() string {
    return fmt.Sprintf("%s: %s", m.Msg, m.Cause.Error())
}
```

그런데 이렇게 하면 또 약간 귀찮은 부분이 생긴다. 일반적으로 함수가 리턴할 때는 error 인터페이스 상태로 리턴하기 때문에 캐스팅을 해줘야되는 것. 이 땐 if else 문을 쓰는게 마음 편하다.

```go
result, err := SomeFunc()
if mErr, ok := err.(*MyError); ok {
    // mErr을 사용해서 에러 핸들링
} else {
    msg := fmt.Sprintf("unknown error: %s", err)
    mErr = &MyError{Msg: msg}
    // mErr을 사용해서 에러 핸들링
}
```

### error 모아서 처리하기

코드 중복도 줄일 겸 로직도 알아보기 편하게 할 겸 겸사겸사 모아서 처리하는 법을 찾아보게 되었다. 무엇보다 스프링 부트에서 `@AdviceController` 나 `@ExceptionHandler` 를 엄청 편하게 썼던 기억을 잊을 수가 없었다.

#### defer, panic, recover

defer, panic, recover을 사용하는 방법. 이 세개는 모두 go built-in 함수.

* `defer`: defer로 어떤 함수를 지정해놓으면 defer가 선언된 스코프가 끝나고 난 다음 실행하게 됨. 한 스코프에 여러 defer 함수를 넣을 수 있으며 여러개를 넣어놓아도 모두 실행한 다음 끝남. 보통 `Close()` 처럼 clean 로직이 필요할때 사용
* `panic`: panic을 호출하면 프로그램의 일반적인 흐름이 즉시 멈추고 panicking이라고 부르는 상태로 바뀜. 이 상태에서는 함수의 콜스택을 하나하나 거슬러 올라가면서 모든 defer 함수들을 호출하고 root까지 가면 프로그램이 강제종료됨
* `recover`: defer 함수 안에서만 사용. recover를 호출하게 되면 panicking에서 다시 원래 프로그램 흐름으로 돌아오게 됨. 이 때, panic에 패러미터로 넘겼던 interface를 반환함.

요약하면 모든 error가 발생할 때마다 panic을 호출하고, error를 모아서 처리할 곳에다가 defer 및 recover를 등록해두는 것. 이렇게 하면 어디서 언제 error가 발생하든 recover로 모두 모이게 된다.

```go
func HandleHttpRequest(request *Request) *Response {
    defer func() {
        if r := recover(); r != nil {
            if err, ok := r.(error); ok {
                // 에러 타입에 따라 적절히 에러 처리
            } else {
                panic(r)
            }
        }
    }()
    
    if request.URI != "/" {
        panic(fmt.Errorf("not found %s", request.URI))
    }
    
    return handleIndex(request)
}

func handleIndex(request *Request) *Response {
    var err error

    // 비지니스 로직
    if err != nil {    
        panic(fmt.Errorf("internal server error %s", err))
    }
}

```

#### channels, sync.WaitGroup, errgroup

defer, panic, recover 방식의 한계는 goroutine을 사용할 수가 없다는 것. 

goroutine을 쓰는이 경우엔 다른 방법을 사용해야 함. channel 또는 errgroup을 쓰는 방법이 있음.

```go
func useChannel() {
	goRoutineNum := 2
	
	errChannel := make(chan error, goRoutineNum)
	
	var wg sync.WaitGgroup
	wg.Add(goRoutineNum)

	go func() {
		defer wg.Done()

		if err := someFunc(); err != nil {
			errChannel <- err
			return
		}
	}()

	go func() {
		defer wg.Done()

		if err := someFunc(); err != nil {
			errChannel <- err
			return
		}		
	}()


	wg.Wait()
	close(errChannel)

	for err := range errChannel {
		// handle err
	}
}

func useErrGroup() {
	var g errgroup.Group

	g.Go(func() error {
		if err := someFunc(); err != nil {
			return err
		}
		return nil
	})

	g.Go(func() error {
		if err := someFunc(); err != nil {
			return err
		}
		return nil
	})

	if err := g.Wait(); err != nil {
		// handle err
	}
}
```

channel을 쓰면 각각의 에러에 대해서 적절한 처리를 하는 것이 가능. 하지만 대부분의 경우 거기까진 필요없음. 그냥 모든 goroutine에 대해서 에러가 하나 이상 발생했는지 체크하는 경우가 더 많음. 이 경우에는 errgroup을 쓰는게 좀 더 빠르고 간편함.

