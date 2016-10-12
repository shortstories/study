# Windows10 Hyper-V와 Virtual Box가 충돌을 일으켰을 때
## 해결 방법
1. 우선 Virtual box를 지우고 재시도
2. 그래도 안될 시, 다음과 같은 방법으로 virtual switch 재설정
  1. 일단 Powershell 관리자 모드로 실행
  2. `Get-NetAdapter` 실행
  3. ![](캡처.PNG)
  4. InterfaceDescription에서 자신의 실제 인터페이스 확인 (여기에선 Intel(R) Ethernet Connection (3) l21... 이 된다)
  5. `New-VMSwitch` 실행. -NetAdapterName에 실제 인터페이스의 Name 입력 (여기에선 `New-VMSwitch "VM Network" -NetAdapterName "이더넷" -AllowManagementOS $True`)