# Server Layer Design

[//]: # (
그림 수정 예정 
서버 계층 그림으로 변경
)

H/W 계층은 아래 그림과 같이 센서의 데이터를 수집하는 임베디드 시스템과 수집한 데이터를 서버 계층에 전달하는 게이트웨이로 구성됩니다.

![](../resources/book_iot_gateway.png)

### 서버 계층 구성 요소

#### MQTT Broker

H/W 계층의 게이트웨이와 서버 계층의 웹 애플리케이션 서버간의 통신을 중개합니다. 

#### Web Application Server

IoT 시스템의 핵심 비즈니스 로직이 들어가는 서버 프로그램입니다. MQTT 브로커와 사용자간의 통신을 중개하며 사용자의 요구사항을 처리합니다. 
예시로 온도 모니터링 시스템을 


- MQTT 브로커
    - Mosquitto
    - 임베디드 보드에서 네트워크 작업이 가능하면, IoT Gateway의 기능을 임베디드 보드에 넣어도 괜찮습니다.
    - Tech keyword: xxx
- 웹 애플리케이션 서버
    - Nest JS
    - 예제 IoT 시스템에서는 xxx 센서를 사용합니다.
    - Tech keyword: xxx

[//]: # (#### 이 예제에서는 관심사를 나누어 임베디드 보드와 Gateway가 통신하는 방법을 기술합니다.)

