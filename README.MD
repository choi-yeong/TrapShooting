# 📝READ ME FIRST
해당 프로젝트는 PLC(LD)와 HMI를 이용해 만든 프로젝트입니다.

## 🛠환경설정
|카테고리|PLC|HMI|
|:-|:-:|:-:|
|프로그램|XG5000|XP-Builder3.90|
|버전|4.78 2024-12-24| 3.90|
|CPU 환경| XGI-CPUE| XGI(CPU Ethernet)|
|XGT Panerl| - | IPC Serise(wXP)[1024x768]|
|설치링크|[PLC설치](https://ssqfd-abhdhraad3gkgbcg.a03.azurefd.net/ssqblob/largefile/document/17421913207640/XG5000_V4.78_2025-01-17_Kr.exe "XG5000 4.78버전 한글판")|[HMI설치](https://ssqfd-abhdhraad3gkgbcg.a03.azurefd.net/ssqblob/largefile/document/17385570144790/XP-Builder_V3.90.0909.exe "XP-Builder 3.90 설치")|

[유튜브영상링크](https://youtu.be/cFVF2-EeiwI?si=3EVNnLrey5DPiLhG "시연 영상 보기")

※업데이트 상황에 따라 설치링크에 변동이 생기면 링크로 접속이 안될 수 있습니다.

## 🙋‍♂️구현한 기능
### 1. 랜덤화 펑션블록 Randomize 구현
![image](https://github.com/user-attachments/assets/a4fee6d6-cf61-469e-bfaf-7bb439823bab)
* 시스템시간을 호출하는 TOD_TO_***을 사용하여 입력값에 _RTD_TOD라는 플래그변수를 사용.

&rarr; 시스템시간은 (ms)로 표기되어 UDINT로 출력값을 받으면 아주 빠르게 1의 자리 숫자가 변함.

* MOD를 이용해 10의 자리까지 나누어 1의자리만 남겨놓음.

&rarr; 1ms 마다 0~9의 랜덤한 숫자를 출력하는 값만 남기고 이를 RESULT 변수에 담아 랜덤기능 구현.
### 2. 명중 판정 기능 구현
![image](https://github.com/user-attachments/assets/8e1f020a-3234-45f7-bb4b-dc4edb2c1ee8)
![image](https://github.com/user-attachments/assets/d364d951-fb32-4e70-b095-2e17648be2b9)

* HMI상으로 이미지가 충돌판정 감지하는 스크립트를 작성하기보다 PLC상으로 TIMER 펑션을 최대한 활용하고 싶어서 명중되는 시간을 TIMER 펑션으로 구현함.

&rarr; HMI 부품비트의 직선거리 이동속도(애니메이션)과 PLC상의 로직 타이머(판정)의 싱크를 맞추는 수고가 많이 들어가게 됨.

### 3. 탄약과 장전 기능 구현
![gif](https://github.com/choi-yeong/TrapShooting/blob/main/image/%EC%8B%9C%EC%97%B0%EC%98%81%EC%83%81%20-%20Clipchamp%EB%A1%9C%20%EC%A0%9C%EC%9E%91.gif)

* 탄약을 장전할때는 장전하는 시간 1초동안 장전버튼을 꾹 누르고 있어야한다.
* 1초동안 꾹 누르지 않고 중간에 클릭을 떼버리면 장전이 되지 않는다.
* 사격시 총알을 1발 소모하게 된다.
* 왼쪽에 등장하는 노란 반원물체는 '클레이'인데 아직 클레이가 날아다니는 모션을 구현하지 않음.

&rarr; 노란 반원의 클레이가 이동하는 애니메이션과 명중 판정의 싱크를 맞춰 정확히 사격범위에 들어왔을때 사격하게 되면 점수를 1점 얻는 형식.

### 4. 랜덤한 클레이를 랜덤한 타이밍에 던지는 기능
![image](https://github.com/user-attachments/assets/e1ee9344-2e62-40f0-b91d-183fc361ef45)
* 클레이를 일정한 속도로 던지거나 클레이가 일정한 순서대로 나오면 게임이 단순화되고 재미가 없으므로 랜덤성을 부여하기위해 Randomize 커스텀펑션블록을 적극활용.
* 사진상 파란 네모박스 쳐진 곳은 기본딜레이이며 이 딜레이를 줄일수록 클레이를 더 자주 던지게 되어 난이도가 상승하는 대신 점수를 더 많이 얻을 수 있다.
![image](https://github.com/user-attachments/assets/c0d9d2cb-c6f8-49f5-b0dc-cbd7d3ae4558)
* 사진상 파란색네모박스가 쳐진 곳은 해당 클레이가 던져지고 있는 동안 다시 던져지게 되면 클레이의 이동이 초기화되는 버그가 발생하는 현상을 방지하고자 B접점으로 클레이가 던져지고 있을 때 같은 클레이번호가 등장하지 않도록 조치하였다.
* 클레이가 던져지려면 해당 클레이가 선택되는 동안 '플라이클레이' 펑션이 상승엣지검출(양변환검출접점)되어야한다.
![image](https://github.com/user-attachments/assets/165218e4-bca8-4f68-9cf8-83fdb02ba366)

&rarr; '플라이클레이.Q'를 양변환 검출접점을 한 이유는 플라이클레이가 출력되는 1초동안 변환하는 모든 클레이들을 동작시키는 버그를 방지하고자 상승엣지검출만 하여 딱 1회 던지도록 조치하였다.

### 5. 점수 카운팅 기능
![image](https://github.com/user-attachments/assets/1564256c-5fc7-41f4-955b-aa37013f3f7e)
![image](https://github.com/user-attachments/assets/d362b8f9-51d5-41b1-b68e-07efe5102e2b)

* 게임을 완성해보고 더 테스트를 해봐야하는 기능이다.
* CTU_UINT 펑션을 사용해서 각 클레이의 '명중시간'에 해당하는 타이밍에 사격버튼을 누르면 '득점'접점이 활성화되어 카운트업 되는 기능이다.

&rarr; 예상문제점, 손이 빠르다면 한개의 클레이가 명중시간에 닿을 때 사격버튼을 빠르게 2번 눌러 1클레이2득점이 가능한 버그가 발생될것으로 예상된다.(클레이 명중시 해당 클레이가 사라지도록 조치필요.)


### 6. 기능부분 전부 완성. (클레이 비행, 사격, 명중, 탄약 기능, 재장전 기능, 사운드 등)
![gif](https://github.com/choi-yeong/TrapShooting/blob/main/image/%EA%B2%A9%EB%B0%9C%20-%20Clipchamp%EB%A1%9C%20%EC%A0%9C%EC%9E%91.gif)
![gif](https://github.com/choi-yeong/TrapShooting/blob/main/image/%EC%B4%9D%EC%95%8C%20-%20Clipchamp%EB%A1%9C%20%EC%A0%9C%EC%9E%91%20(1).gif)
![gif](https://github.com/choi-yeong/TrapShooting/blob/main/image/%EC%A0%9C%EB%AA%A9%20%EC%97%86%EB%8A%94%20%EB%8F%99%EC%98%81%EC%83%81%20-%20Clipchamp%EB%A1%9C%20%EC%A0%9C%EC%9E%91.gif)
```
// 클레이가 날아가는 애니메이션을 표현할 스크립트
//0번 클레이
if(@[X:0:QX0.1.10] == 1)  //각 클레이의 번호가 켜져있을 때 동작한다.
//클레이 번호가 켜지면 우선 클레이 램프가 켜져서 시각적으로 표현되어야한다.{
	if(@[W:0:MW0] < 100)  // 클레이의 위치를 표현한다. 클레이의 위치가 0에서 100으로 이동한다.{
			@[W:0:MW0] = @[W:0:MW0]+2;  //더하는 숫자는 이동속도를 뜻한다.
		}
}
else if(@[X:0:QX0.1.10] == 0) //클레이 번호가 꺼져있을때{
@[W:0:HW00000] = 0;  // 클레이의 위치를 0으로 초기화 한다.
}
//1번 클레이
if(@[X:0:QX0.1.11] == 1) {
	if(@[W:0:MW1] < 100) {
			@[W:0:MW1] = @[W:0:MW1]+2;
		}	
}
else if(@[X:0:QX0.1.11] == 0){
@[W:0:MW1] = 0;
}

//2번 클레이
if(@[X:0:QX0.1.12] == 1) {
	if(@[W:0:MW2] < 100){
			@[W:0:MW2] = @[W:0:MW2]+2;
		}
}
else if(@[X:0:QX0.1.12] == 0){
@[W:0:MW2] = 0;
}
//3번 클레이
if(@[X:0:QX0.1.13] == 1) {
	if(@[W:0:MW3] < 100){
			@[W:0:MW3] = @[W:0:MW3]+2;
		}	
}
else if(@[X:0:QX0.1.13] == 0){
@[W:0:MW3] = 0;
}
//4번 클레이
if(@[X:0:QX0.1.14] == 1) {
	if(@[W:0:MW4] < 100){
			@[W:0:MW4] = @[W:0:MW4]+2;
		}	
}
else if(@[X:0:QX0.1.14] == 0){
@[W:0:MW4] = 0;
}
//5번 클레이
if(@[X:0:QX0.1.15] == 1) {
	if(@[W:0:MW5] < 100){
			@[W:0:MW5] = @[W:0:MW5]+2;
		}
}
else if(@[X:0:QX0.1.15] == 0){
@[W:0:MW5] = 0;
}
//6번 클레이
if(@[X:0:QX0.1.16] == 1) {
	if(@[W:0:MW6] < 100){
			@[W:0:MW6] = @[W:0:MW6]+2;
		}	
}
else if(@[X:0:QX0.1.16] == 0){
@[W:0:MW6] = 0;
}
//7번 클레이
if(@[X:0:QX0.1.17] == 1) {
	if(@[W:0:MW7] < 100){
			@[W:0:MW7] = @[W:0:MW7]+2;
		}	
}
else if(@[X:0:QX0.1.17] == 0){
@[W:0:MW7] = 0;
}
//8번 클레이
if(@[X:0:QX0.1.18] == 1) {
	if(@[W:0:MW8] < 100)		{
			@[W:0:MW8] = @[W:0:MW8]+2;
		}
}
else if(@[X:0:QX0.1.18] == 0){
@[W:0:MW8] = 0;
}
//9번 클레이
if(@[X:0:QX0.1.19] == 1) {
	if(@[W:0:MW9] < 100)		{
			@[W:0:MW9] = @[W:0:MW9]+2;
		}
}
else if(@[X:0:QX0.1.19] == 0){
@[W:0:MW9] = 0;
}
ERROR:
{}
```

## 💣트러블 슈팅
### 1. 클레이가 한번에 여러번 날아가는 버그 발생
> 플라이클레이.Q가 켜지는 동안 여러개의 클레이0~9가 동작해서 생긴 버그로 판단
> 
> &rarr; 플라이클레이.Q를 P접점(양변환검출접점)으로 바꾸었으나 여전함.
> 
> &rarr; 같은 클레이번호가 여러번 반복하는 걸 보고 스크립트의 위치 디바이스주소가 문제인것을 깨달음.
> 
> &rarr; 스크립트에서 위치디바이스의 주소가 max에 달하면 0으로 초기화 시키는 것이 아니라 비행시간인 TRTG의 PT를 줄여 클레이가 빠르게 사라지도록 수정해서 여러번 반복하는 것을 방지 및 스크립트에서 해당디바이스주소가 ON일떄 위치디바이스가 0으로 초기화되는 코드를 삭제함.
>
> ![iamge](https://github.com/choi-yeong/TrapShooting/blob/main/image/%EC%83%88%20%ED%8F%B4%EB%8D%94/%EC%88%98%EC%A0%951.png)

> 📝결과 : 정상동작. 더 이상 클레이가 여러번 던져지지 않음.

### 2. 발포섬광(총격파티클)이 표현되지 않는 버그 발생
> 총격파티클이 1틱만 ON되어서 이미지가 보이기도 전에 OFF되는 것이라 판단
> 
> &rarr; 총격파티클 전에 TRTG로 200ms의 텀을 둬서 총격파티클이 모니터출력주파수 및 사람의 육안으로 확인할 수 있도록 수정함.
>
>![image](https://github.com/choi-yeong/TrapShooting/blob/main/image/%EC%83%88%20%ED%8F%B4%EB%8D%94/%EC%88%98%EC%A0%953.png)
>
> 📝결과 : 정상동작. 총격파티클 정상 동작. 또한, 예상문제점이던 "1개의 클레이로 2번 점수 얻기" 버그도 예방되었음.

### 3. 게임시작도 안 했는데 접시가 던져지는 걸 방지
> 클레이던지기(TOF펑션)가 게임 시작 전에는 실행되지 않도록 첫행에 게임시간.Q를 A접점으로 넣음으로 게임 중에만 접시를 던지도록 구현함
> 
> ![image](https://github.com/choi-yeong/TrapShooting/blob/main/image/%EC%83%88%20%ED%8F%B4%EB%8D%94/%EC%88%98%EC%A0%952.png)
>
> 📝결과 : 정상동작. 게임 시작 버튼을 누르기전에 접시가 던져지는 일이 사라짐.
