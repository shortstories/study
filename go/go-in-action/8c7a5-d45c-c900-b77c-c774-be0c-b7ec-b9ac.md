# 8장 표준 라이브러리

* 하위 호환성 보장
* 새로운 버전에도 항상 동일하게 제공
* Go개발, 빌드 릴리즈 프로세스의 일부로 활용
* Go 컨트리뷰터들이 지속적으로 관리
* 새로운 버전이 릴리즈될 때 이미 테스트, 벤치마크가 수행되어있으므로 안심하고 사용 가능

## 문서화, 소스코드

* Go 웹사이트: [http://golang.org/pkg/](http://golang.org/pkg/)
* sourcegraph: [http://sourcegraph.com](http://sourcegraph.com)
* GOPATH, GOROOT에서 확인 가능

## 로깅

`log` 패키지 사용

* main 패키지의 `init()` 함수에서 세팅하는게 일반적
* 플래그로 출력될 내용 포맷 설정 가능
* Print: 일반 로그 출력
* Fatal: 치명적 로그 출력. 로그 출력 후 프로그램 종료
* Panic: 패닉 처리. 로그 출력 후 패닉을 발생시키고 복구하지 못하면 프로그램 종료
* threadsafe
* 팩토리 함수를 호출해서 사용자정의 logger도 생성 및 설정 가능

## 인코딩/디코딩

`xml`, `json` 패키지 등을 통해서 처리

* struct 필드 태그를 사용해서 처리. 만약 태그 가 없으면 필드 이름을 기준으로 처리
* `io.Reader` 구현체는 `newDecoder` 을 생성해서 디코딩 처리, 문자열은 `Unmarshal`로 디코딩 처리
* 애매하면 `map[string]interface{}`로 만들어서 처리해도 됨
  * 값을 모두 캐스팅해야되서 복잡하고 귀찮음
* 인코딩은 `Marshal` 또는 `MarshalIndent`로 처리함

## 입출력

* `io` 패키지 사용
  * `Writer` 인터페이스
  * `Reader` 인터페이스

