# 게이트웨이 구현
온도 모니터링 시스템을 예제로 게이트웨이의 구현 방법을 알아보겠습니다. 온도 모니터링 시스템의 게이트웨이 소스코드는 (xxx) 에서 확인할 수 있습니다. 
게이트웨이의 핵심 역할은 H/W 계층과 서버 계층간의 통신 중개입니다. 서버 계층의 MQTT 브로커와 통신을 구현하는 여러 MQTT Client 라이브러리가 있습니다. 
이 책에서는 Arduino IDE를 사용하여 ESP8266 모듈과 PubSubClient 라이브러리로 게이트웨이의 구현 방법을 설명하겠습니다.  만약 ESP 8266 모듈이 없다면, 카페의 아두이노 소스 또는 C로 짜여진 소스를 다운받아 실습할 수 있습니다.


## ESP 8266 MQTT 설정
(... 그림 56. 아두이노 IDE 개발환경에서 보드선택) 그림 등 ESP8266 아두이노 설정 여기에 넣기



## PubSubClient 예제 살펴보기

보드 매니저 설정 이후 아두이노 IDE에서 file -> example -> PubSubClient -> mqtt_esp8266 예제를 선택하면 기본적인 MQTT 클라이언트 예제를 확인할 수 있습니다. 이 책에서는 해당 예제를 우리 시스템에 맞게 수정합니다. 불러온 코드를 보면 하드웨어가 접속할 와이파이 이름과 비밀번호 그리고 MQTT 서버를 지정할 수 있습니다. 여기서 MQTT 서버는 이전에 EC2 인스턴스에 배포한 MQTT 브로커를 의미합니다. MQTT 브로커가 실행 중인 EC2 인스턴스의 공개 IP를 입력합니다. 다음과 같이 본인의 환경에 맞게 코드를 수정합니다. 또한 ssid를 설정할 때 해당 와이파이 주파수 대역이 2.4GHz 인지 확인해야합니다. ESP8266 모듈은 5GHz 와이파이를 사용할 수 없습니다.
mqtt_esp8266.ino
```
...
const char* ssid = "your-wifi";
const char* password = "your-password";
const char* mqtt_server = "123.123.123.123";
...
```


PubSubClient 예제를 살펴보겠습니다. 세세하게 이해할 필요는 없고 대충 어떤 방식으로 작동하는지 큰 그림만 이해하면 됩니다. 프로그램이 시작되면 setup_wifi 함수에서 설정한 와이파이에 접속을 시도합니다. 만약 와이파이 접속을 못한다면 ssid 또는 비밀번호를 확인하고 해당 와이파이 주파수가 2.4GHz가 맞는지 확인합니다.
setup 함수
```
void setup() {
  pinMode(BUILTIN_LED, OUTPUT);
  Serial.begin(115200); // baudrate 설정
  setup_wifi();         // 와이파이 설정
  client.setServer(mqtt_server, 1883);  // MQTT 브로커, 포트 번호 설정
  client.setCallback(callback);  // 토픽을 수신했을 때 실행할 콜백 함수 등록
}
```

reconnect 함수에서 MQTT 브로커에 접속을 시도합니다. 접속에 성공하면 다음과 같이 subscribe 함수를 사용해 토픽을 구독할 수 있습니다.
reconnect 함수
```
void reconnect() {
  ...
  
  client.subscribe("inTopic");  // 토픽 구독
  
  ...
}

```

MQTT 브로커에 접속 이후 publish 함수를 사용해 주기적으로 토픽과 메시지를 MQTT 브로커에 발행합니다.
loop 함수
```
void loop() {
  /**  브로커에 접속 */
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  /**  주기적으로 브로커에 메시지 발행 */
	...
	
    client.publish("outTopic", msg); // 토픽과 메시지 발행
    
    ...
  }
}
```


이전에 setup 함수에서 등록한 콜백 함수를 살펴보겠습니다. 구독한 토픽을 수신했을 때 해당 콜백 함수가 실행됩니다. 이 콜백 함수에서 수신한 토픽을 처리하면 됩니다. 예제에서는 수신한 메시지를 출력하고 사용하는 방법을 보여줍니다.
callback 함수
```
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);
  } else {
    digitalWrite(BUILTIN_LED, HIGH);
  }

}
```

만약 임베디드 시스템과 연동하는 예를 들면 다음과 같이 활용할 수 있습니다.
1. 백엔드 애플리케이션에서 하드웨어 제어 패킷을 MQTT 토픽으로 발행합니다.
2. 해당 토픽을 수신한 게이트웨이는 메시지에서 패킷을 추출합니다.
3. 추출한 패킷을 시리얼 통신으로 임베디드 시스템에 전달합니다.


이제 작성한 예제 코드를 ESP8266 모듈에 업로드하고 아두이노의 시리얼 모니터를 통해 결과를 확인할 수 있습니다. 그리고 이전에 백엔드 애플리케이션의 센서 컨트롤러에 작성한 메시지 전송 기능을 사용해 토픽과 메시지를 전송해 시리얼 모니터에 출력되는지 확인할 수 있습니다.