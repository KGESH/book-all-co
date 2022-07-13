## 시스템 구성도

![](../../../Downloads/book_mqtt.drawio.png)

- H/W Layer
  - 온도 데이터를 읽는 임베디드 보드
  - 임베디드 보드에게 전달받은 데이터를 MQTT 브로커에게 발행할 IoT Gateway
  - 임베디드 보드에서 네트워크 작업이 가능하면, IoT Gateway의 기능을 임베디드 보드에 넣어도 괜찮다
  - 이 예제에서는 관심사를 나누어 임베디드 보드와 Gateway가 통신하는 방법을 기술한다
  - Tech stack: C, Arduino, ESP 8266 

- MQTT Broker
  - 서버와 IoT Gateway의 통신을 중개
  - 브로커는 Mosquitto를 사용한다


- Web Application Server
  - MQTT 브로커에게 전달받은 온도 데이터를 사용하는 backend
  - 브로커에게 온도 topic을 구독
  - 사용자가 온도를 요청하면 브로커에게 전달받은 최신 온도 데이터를 전달한다
  - Tech stack: TypeScript, NestJS


- User Client
  - 서버가 가진 가장 최근 온도를 확인할 수 있는 frontend
  - Tech stack: HTML, JavaScript <-- React 고민중
  
