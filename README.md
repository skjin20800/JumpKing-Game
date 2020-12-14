# JumpKing-Game


안녕하세요. 진경빈입니다.
이번에는 2주동안 점프킹을 제작했는데요
어떤 기능들을 구현했는지 알아보도록 하겠습니다

## First Player

### 플레이어의 기능

1. 좌, 우 이동
2. 점프
3. 중력(땅으로 떨어지는 기능)
	이렇게 크게 3가지 입니다
	
#### 우 이동 코드 핵심만 보겠습니다
```java

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
1. 키보드 리스너이용해서 메서드 호출
2. 호출된 메서드 쓰레드 생성 및 start
3. 우측으로 x값 10씩 변동 
4. 좌측도 같은 개념


#### 점프 핵심 코드
1번 쓰레드 [ 기모으기,  점프 Up ]
2번 쓰레드 [ 점프 Down ]

총 3가지로 구성 기모으기, 점프 Up은 같은 쓰레드
점프 Up이되어 최고지점 도달시 점프 Down쓰레드 호출
들어오는 좌, 우 입력이 같이 들어온다면 대각선 점프도 가능

1번 쓰레드
```java
class ThJumpUp implements Runnable {
		@Override
		public void run() {
			moveLock = true; // 점프시 좌,우이동 불가능
			isUp = true; // 업만 가능
			isRight = false; // 오른쪽이동 불가능
			isLeft = false; // 왼쪽 이동 불가능
			downJpDown = true; // 점프 실행중 true 아닐시 false
			jumpGauge = 0;
			jumpLeft = 3;
			jumpRight = 3;
			while (isUp == true) {
				setIcon(icJumpR1);
				try {
					if (jumpGauge < 101) { // 기(파워)가 100이상이면 더이상 못모움
						jumpGauge = jumpGauge + 1; // 한번에 5씩 모을수있음
					}
					Thread.sleep(5);
				} catch (Exception e) {
					e.printStackTrace();
				}
			}

			while (true) {
				try {
					isRight = false; // 우 이동불가능
					isLeft = false;// 좌 이동 불가능
					if (jumpUpDirection == 1) { // 우측 점프시 우측으로가는 조건문
						if (moveLockRight == true) {
							jumpUpDirection = -1;
							continue;
						}
						playerX = playerX + jumpRight;
						setIcon(icJumpR2);
					} else if (jumpUpDirection == -1) { // 좌측 점프시 좌측으로가는 조건문
						if (moveLockLeft == true) {
							jumpUpDirection = 1;
							continue;
						}
						playerX = playerX - jumpLeft;
						setIcon(icJumpL2);
					} else if (jumpUpDirectionStay == true) { // 우측을보며 제자리점프시 아이콘 icJumpR2아이콘사용
						setIcon(icJumpR2);
					} else if (jumpUpDirectionStay == false) { // 좌측을보며 제자리점프시 아이콘 icJumpR2아이콘사용
						setIcon(icJumpL2);
					}

					setLocation(playerX, playerY); // 내부에 repaint() 존재
					// 포물선으로 이동하는 조건문
					if (jumpGauge > 50) {// 점프 분기1 제일 밑 부분
						playerY = playerY - 5;
						jumpGauge -= 1;

					} else if (jumpGauge <= 50 && jumpGauge > 25) { // 점프 분기2 중앙 부분
						playerY = playerY - 3;
						jumpGauge -= 1;

					} else if (jumpGauge <= 25) { // 점프 분기3 제일 윗 부분
						playerY = playerY - 1;
						jumpGauge -= 1;

					}

					if (jumpGauge <= 0) {// 최대 높이 도달시 함수종료
						System.out.println("다운메서드 호출");
						jumpGauge = 100; // 점프게이지 0으로 초기화
						break;
					}
					Thread.sleep(5);
				} catch (Exception e) {
					e.printStackTrace();
				}
			}

			jumpDown(); // 다운 메서드 호출
		}
	}
	
``` 
2번 쓰레드
``` java

	class ThJumpDown implements Runnable {
		@Override
		public void run() {
			jumpGaugeDown = 100;
			isDown = true; // 다운 사용
			jumpNuckDownCount = 0; // 일정거리 이상 떨어지면 넉다운 아이콘발동
			while (isDown) {
				setLocation(playerX, playerY); // 내부에 repaint() 존재

				try {
					if (Gravity == true) {

						isRight = false;
						isLeft = false;
						if (jumpUpDirection == 1) { // 우측다운시 x값증가
							if (moveLockRight == true) {
								jumpUpDirection = -1;
								continue;
							}
							setIcon(icJumpR3);
							playerX = playerX + jumpRight;
						} else if (jumpUpDirection == -1) { // 좌측측다운시 x값 감소
							if (moveLockLeft == true) {
								jumpUpDirection = 1;
								continue;
							}
							setIcon(icJumpL3);
							playerX = playerX - jumpLeft;
						} else if (jumpUpDirectionStay == true) { // 우측보며 제자리뛰기시 이미지
							setIcon(icJumpR3);
						} else if (jumpUpDirectionStay == false) {// 좌측보며 제자리뛰기시 이미지
							setIcon(icJumpL3);
						}
						if (jumpGaugeDown > 75) {// 점프 분기1 제일 윗 부분
							playerY = playerY + 1;
							jumpNuckDownCount += +1;
							jumpGaugeDown -= 1;
						} else if (jumpGaugeDown <= 75 && jumpGaugeDown > 50) { // 점프 분기2 중앙 부분
							playerY = playerY + 3;
							jumpNuckDownCount += +3;
							jumpGaugeDown -= 1;
						} else if (jumpGaugeDown <= 50) { // 점프 분기3 제일 밑 부분
							playerY = playerY + 5;
							jumpNuckDownCount += +5;
							jumpGaugeDown -= 1;
						}
					} else if (Gravity == false) {

						isDown = false; // 다운 금지
						isUp = false; // 업 금지
						if (jumpNuckDownCount > 200) {
							setIcon(icNuckDown);
						} else if (jumpUpDirection == 1) { // 우 점프 착지 이미지
							setIcon(icJumpR4);
						} else if (jumpUpDirection == -1) { // 좌 점프 착지 이미지
							setIcon(icJumpL4);
						} else if (jumpUpDirectionStay == true) { // 우 제자리 점프 착지 이미지
							setIcon(icJumpR4);
						} else if (jumpUpDirectionStay == false) {// 좌 제자리 점프 착지 이미지
							setIcon(icJumpL4);
						}
						jumpUpDirection = 0;// 점프방향 초기화 (점프가 끝나고 그냥 위로점프시 버그해결용)
						moveLock = false; // 점프가 끝나면 다시 이동할수있게 Lock해제
						downJpDown = false; // 점프가 끝나고 다시 중력실행
						break;
					}
					Thread.sleep(5);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}
	}
```


