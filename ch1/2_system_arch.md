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
  - Tech keyword: HTML, JavaScript <-- React 고민 중
  
