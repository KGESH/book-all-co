# IoT 시스템 디자인
이 책에서는 사용자가 웹을 통해 하드웨어와 통신할 수 있는 간단한 IoT 시스템을 구축해 봅니다. 예시 IoT 시스템에서는 그림과 같은 3개의 계층을 나누어 설명합니다.

![[resources/ch1/1/1.png]]

만약 온도 모니터링 시스템을 구축한다면 시스템의 큰 흐름은 다음과 같습니다.

1. H/W 계층에서 주기적으로 온도를 읽고 서버로 전송합니다.
2. 서버는 온도를 수신할 때마다 최신 데이터로 업데이트합니다. 
3. 사용자 계층에서 사용자가 서버에게 온도 데이터를 요청합니다.
4. 서버는 가지고 있는 최신 온도 데이터를 사용자에게 전달합니다.

위의 큰 흐름을 기억해두고, 추후 세부적인 내용들을 확인해 봅니다. 


# 시스템 구성도

## H/W Layer
### IoT Gateway
- 서버와 임베디드 시스템간의 통신을 중개합니다. 이때 사용하는 프로토콜을 MQTT 프로토콜이라고 합니다.
- Tech keyword: C, Arduino, ESP 8266, MQTT

### Embedded System
- 센서의 데이터를 읽는 임베디드 보드입니다. 게이트웨이에 데이터를 송신, 수신합니다.
- Tech keyword: xxx


## Server Layer
### MQTT Broker
- 서버와 IoT Gateway의 통신을 중개합니다. 이 책은 오픈소스 MQTT 브로커인 Mosquitto를 사용합니다. (https://mosquitto.org/)
- Tech keyword: Mosquitto, MQTT

### Web Application Server
- IoT 시스템의 핵심 비즈니스 로직이 들어가는 서버 프로그램입니다. MQTT 브로커와 사용자 계층의 사용자와 통신을 중개하며 사용자의 요구사항을 처리합니다. 
- Tech keyword: TypeScript, NestJS, MQTT


## User Layer
### User Client
  - 서버에게 데이터를 요청하는 웹 프론트엔드 입니다.
  - Tech keyword: HTML, JavaScript <-- (...React 고민 중)
  
