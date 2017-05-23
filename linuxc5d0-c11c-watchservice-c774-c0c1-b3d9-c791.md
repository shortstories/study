# Linux에서 WatchService 이상동작

정확히는 Symbolic link를 바라보게 할 때 굉장히 이상하게 동작한다.  믿고 쓰기 좀 힘들다...

증상은 다음과 같다.

1. symbolic link가 바라보고있는 진짜 directory를 지우거나 만들었을 땐 `IOException` 이 떨어진다.
2. symbolic link 그 자체를 지우거나 지우고 새로 만들었을 땐 `WatchService.take()` 에서 계속 blocking된다. 아무런 Exception도 떨어지지 않는다... 즉, 감지할 수가 없다.
3. symbolic link를 지운 상태에서 원본과 똑같은 directory를 바라보는 symbolic link를 다시 만들면 아무 일도 없었다는 듯이 정상적으로 이벤트를 받으며 동작한다.
4. symbolic link를 지운 상태에서 다른 directory를 바라보는 symbolic link를 만들어 대신하면 아무런 이벤트도 받지 못하고 먹통이 된다.



