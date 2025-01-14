# 윈도우 개발환경 설정

### 윈도우 설정

1. 제어판에서 hyper-v, WSL 설정 켜기
2. WSL ubuntu 설치
3. [windows terminal](https://www.microsoft.com/store/productId/9N0DX20HK701) 설치
4. [chocolatey](https://chocolatey.org/) 설치
5. [vscode](https://code.visualstudio.com/) 설치 (`choco install vscode`)
6. [D2Coding](https://github.com/naver/d2codingfont) 폰트 설치 (`choco install d2codingfont`)
7. [~~Docker for windows~~](https://docs.docker.com/docker-for-windows/install/) ~~설치~~  [Docker 세팅](https://shortstories.gitbook.io/studybook/windows/docker-desktop-docker)
8. [Notion](https://www.notion.so/desktop) 설치
9. 키보드 shift+space, 한영키 둘 다 먹히게 만들기

### WSL설정

#### /etc/wsl.conf

```
[automount]
enabled = true
root = /
options = "metadata"
```

파일 생성 이후 admin 권한으로 실행한 PowerShell에서 `Restart-Service WSLService` 실행하고 WSL 재시작

안되면 `Restart-Service LxssManager` 언제부터인지 서비스 이름이 바뀌었다.&#x20;

또는 cmd, powershell에서 `wsl.exe --shutdown` 으로도 가능

### ubuntu 세팅

```bash
$ sudo update-alternatives --config editor
$ sudo visudo # %sudo   ALL=(ALL:ALL) NOPASSWD:ALL
$ sudo vi /etc/apt/sources.list
# :%s/archive.ubuntu.com/mirror.kakao.com/g
# :%s/security.ubuntu.com/mirror.kakao.com/g
$ sudo apt update && sudo apt upgrade
$ sudo apt install build-essential zsh
```

#### ZSH 세팅

1. [oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh) 설치
2. zsh plugin 설치
   1. [powerline10k](https://github.com/romkatv/powerlevel10k)
   2. [https://github.com/zsh-users/zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)
   3. [https://github.com/zsh-users/zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)

#### 키보드 shift space, 한영키 동시에 먹히게 만들기

{% file src="../.gitbook/assets/shift-space_default-.reg" %}

#### Golang 환경 세팅&#x20;

1. [https://go.dev/dl/](https://go.dev/dl/) 다운로드
   1. windows의 경우 그냥 msi버전 다운로드 받아서  설치하고 끝
   2. linux의 경우 압축 해제 후 $GOROOT, $GOPATH 및 $PATH 등록&#x20;

#### IDE 설정

1. JetBrain
   1. Ctrl+Alt+S -> Editor -> Code Style -> Line separator: `Unix and macOS (\n)` 설정
2. vscode
   1. Ctrl+, -> eol 검색 -> `\n` 설정

#### Git 설정

1. wsl에서 ssh key 생성 `ssh-keygen -t ed25519`
2. windows에 복사 `mkdir -p /c/Users/kwons/.ssh/ && cp ~/.ssh/* /c/Users/kwons/.ssh/`
3. oss, github에 방금 생성된 public key를  deploy key로  등록
4. windows cmd, wsl 둘다 아래 git config 명령어 실행

<pre><code>$ git config --global core.fileMode false
<strong>$ git config --global core.autocrlf input
</strong>$ git config --global core.eol lf
$ git config --global credential.helper store
$ git config --global url.ssh://git@oss.navercorp.com/.insteadOf https://oss.navercorp.com/
$ git config --global url.ssh://git@github.com/.insteadOf https://github.com/
</code></pre>

