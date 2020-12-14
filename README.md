# JumpKing-Game

안녕하세요. 진경빈입니다.
이번에는 2주동안 점프킹을 제작했는데요

어떤 기능들을 구현했는지 알아보도록 하겠습니다.

####플레이어
### 플레이의 기능
1. 좌, 우 이동
2. 점프
3. 중력(땅으로 떨어지는 기능)

###이렇게 크게 3가지 입니다
#### 좌, 우 이동 소스코드입니다.
```

	class RightMove implements Runnable { // Leftmove()랑 거의비슷
		@Override
		public void run() {
			// 좌, 우 이동변경시 부드럽게 하기위해 다시한번 설정
			isLeft = false; // 왼쪽이동 false
			isRight = true; // 오른쪽이동 true

			while (isRight == true && isMoveLockRight() == false) {
				try {
					playerX = playerX + 10; // 쓰레드 1회당 이동시 x값 10씩증가
					setLocation(playerX, playerY); // 내부에 repaint() 존재

					// 일정시간마다 이미지 변경 좌,우,점프업,점프다운 모두 동일
					Thread.sleep(10);
					setIcon(icPlayerRR);
					Thread.sleep(10);
					setIcon(icPlayerRS);
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		}
	}
```


