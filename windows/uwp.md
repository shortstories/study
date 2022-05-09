# UWP 앱을 항상 관리자권한으로 실행하는 바로가기 만들기

이번에 업데이트된 Windows Terminal가 내 마음에 쏙 든다. 생긴거도 이쁘고 색상 스키마도 지정할 수 있고 기능도 이만하면 더할 나위 없다. 특히 텍스트를 복붙하는게 이전 cmd보다 훨씬 쉬워서 너무 좋다. 그런데 막상 쓰다보니 한가지 치명적인 문제가 있었다. 나는 터미널을 항상 관리자 권한으로 실행해야하는데 이 Windows Terminal이라는 앱이 UWP 앱이다보니 관련 설정을 하는 부분을 찾을 수가 없었던 것이다.&#x20;

일반적인 앱이라면 `.exe` 파일을 찾아가서 우클릭한 다음 항상 관리자 권한으로 실행하도록 할 수 있지만 UWP앱은 그런게 존재하지 않는다. 그래서 약간 복잡한 방법을 사용해야만 한다.

### 준비물

* 목표물인 UWP 앱
* 관리자 권한으로 실행시켜둔 Windows Power Shell
* 파일 탐색기 두개
* `C:\admin-link` 폴더 생성

### 1. UWP app의 바로가기 만들기

준비한 파일 탐색기의 주소 입력 칸에다가 `shell:AppsFolder` 를 입력한다. 그럼 지금까지 스토어 등을 통해 설치한 앱들이 나올 것이다. 여기에서 원하는 UWP 앱을 찾아 미리 준비한 `C:\admin-link` 로 드래그 앤 드랍을 해서 바로가기를 만든다. 만들어진 바로가기의 이름은 자신이 입력하기 쉬운 것으로 바꾸는게 좋다. 나의 경우 `WindowsTerminal` 으로 바꿨다.

![](../.gitbook/assets/image.png)

### 2. 바로가기 수정

```
$linkFilePath = "C:\admin-link\WindowsTerminal.lnk"
$bytes = [System.IO.File]::ReadAllBytes($linkFilePath)
$bytes[0x15] = $bytes[0x15] -bor 0x20 #set byte 21 (0x15) bit 6 (0x20) ON
[System.IO.File]::WriteAllBytes($linkFilePath, $bytes)
```

미리 열어뒀던 관리자 권한 powershell에서 위 명령어를 실행한다.

### 3. 실제로 사용할 바로가기 생성

위에서 만들어진 바로가기를 바로 사용할 수는 없다. 따라서 위 바로가기를 참조하는 또다른 바로가기를 생성해야한다. 그런데 윈도우의 특성 상 바로가기를 우클릭해서 바로가기 만들기를 하면 그대로 복사하는 것과 다름 없기 때문에 그렇게 하면 안 된다. 폴더의 빈 공간을 우클릭해서 새로 만들기 -> 바로가기 만들기를 해야 한다.

![](<../.gitbook/assets/image (2).png>)

이름은 자유롭게 지으면 된다. 나는 뒤에 (admin)을 더 붙였다. 마지막으로 새로 생성된 바로가기를 우클릭해서 '실행'  콤보박를 '최소화'로 바꿔주고 아이콘을 바꿔주면 끝이다.

![](<../.gitbook/assets/image (5).png>)

이제 이 바로가기를 통해서 Windows Terminal을 실행하면 항상 관리자 권한으로 실행되게 된다.
