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
![gif](https://github.com/user-attachments/assets/5c415fa7-f397-4fb5-a225-8e6f2ffc13e0)

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
