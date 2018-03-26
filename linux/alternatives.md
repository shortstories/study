# Alternatives

예를 들어 java처럼 다양한 버전들을 동시에 설치하고 필요에 따라 골라 실행해야되는 일이 있다면 어떻게 해야될까? 만약에 다른 버전을 실행할 일이 아주 가끔 있다면 제일 자주 쓰는 버전만 심볼릭 링크나 PATH에 등록해서 사용하면 되기는 할 것이다. 하지만 거의 비슷한 빈도로 버전을 바꿔가며 써야된다면? 매번 절대 경로를 입력해서 실행하는 방법도 있겠지만 굉장히 번거로운 일이다. 그래서 이러한 일을 해결하기 위해서 만들어진 것이 `alternatives` 다. 심볼릭 링크를 관리하는데 도움을 준다.

일반적으로 `/usr/sbin/alternatives` 에 위치하고 있다.

## 사용법

```bash
alternatives [options] --install link name path priority [--slave link name path]... [--initscriptservice]

alternatives [options] --remove name path

alternatives [options] --set name path

alternatives [options] --auto name

alternatives [options] --display name

alternatives [options] --config name  
```

link에는 보통 `/usr/bin/` 으로 시작하는 실행 주소를 넣는다.

name은 같은 프로그램들끼리는 일치하도록 정해야한다. 예를 들어 java7, 8, 9 모두 java라는 name을 가지도록 한다.

path에는 실제로 실행할 파일의 위치를 입력한다.

priority는 auto 모드일 때 어느 것이 우선적으로 실행될지 결정한다.



일반적으로 `--install` 을 사용해서 여러 버전을 모두 등록하고 `--config` 를 사용해서 필요한 버전을 선택한다.

