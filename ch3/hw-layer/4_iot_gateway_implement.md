# IoT 모니터링 프로젝트 - 게이트웨이 구현
이전에 EC2에 배포한 IoT 모니터링 백엔드 애플리케이션과 상호작용하는 게이트웨이를 작성합니다. 책의 예제는 ESP8266 모듈에 업로드 하기 위한 예제이며 이전에 사용한 PubSubClient 라이브러리를 사용할 예정입니다. 또한 카페에서  C++로 작성된 리눅스에서 작동하는 게이트웨이 소스를 확인할 수 있습니다.


이전처럼 PubSubClient 예제를 불러옵니다. 예제 소스에 함수를 추가로 작성해 IoT 모니터링 프로젝트의 게이트웨이를 구현하겠습니다. 와이파이, MQTT 브로커를 설정합니다. 또한 온도 데이터 발행 토픽과 패킷을 수신할 디바이스 토픽을 설정합니다. 예제에서는 백엔드에서 시리얼 번호 1234번 센서가 존재한다 가정하고 진행합니다.
```
...
const char* ssid = "your-wifi";
const char* password = "your-password";
const char* mqtt_server = "123.123.123.123";
const char* temperatureTopic = "temperature/1234";
const char* deviceTopic = "device/1234";
...
```

백엔드에서 어떤 형식으로 메시지를 전송하는지 확인하기 위해 토픽을 구독합니다. 다음과 같이 reconnect 함수에서 디바이스 토픽을 구독을 추가합니다.
reconnect 함수
```
...
if (client.connect(clientId.c_str())) {
      Serial.println("connected");
      client.subscribe(deviceTopic);
...
```

이제 수신한 메시지를 그대로 시리얼 모니터에 출력해 보겠습니다. 다음과 같이 콜백 함수를 수정합니다.
callback 함수
```
void callback(char* topic, byte* payload, unsigned int length) {
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  
  Serial.println();
}
```

이제 코드를 ESP 모듈에 업로드합니다. 업로드 이후 시리얼 모니터를 통해 와이파이와 MQTT 브로커 접속을 확인할 수 있습니다. 
이전에 작성한 백엔드 애플리케이션의 Post('mqtt/message') 요청을 통해 토픽과 메시지를 입력받아 디바이스에 전송할 수 있습니다.
src/sensors/sensors.controller.ts
```
@Post('mqtt/message')  
async sendMessage(@Body() sendMqttDto: SendMqttDto) {  
  const { topic, command, target } = sendMqttDto;  
  const payload = Packet.generateRaw({ target, command });  
  
  return this.mqttService.emit(topic, payload);  
}
```


그림과 같이 스웨거에서 POST 요청으로 생성된 패킷이 디바이스에 JSON 형식으로 도착한 것을 확인할 수 있습니다. 수신한 페이로드(payload)의 구조를 확인했으니 이를 파싱해서 사용하면 됩니다.
#### .. 그림 추가 예정
[그림: mqtt/message POST 요청]


백엔드 애플리케이션이 보내는 메시지는 JSON 형식이므로 JSON 파싱 라이브러리를 설치합니다. 예제에서는 ArduinoJson이라는 라이브러리를 사용하겠습니다. 만약 아두이노 개발 환경 이외의 임베디드 또는 애플리케이션 개발 환경이라면 해당 환경에 맞는 JSON 파싱 라이브러리를 사용하면 됩니다. 대표적으로 cJSON, RapidJSON 등이 있습니다.
다음과 같이 ArduinoJson 라이브러리를 추가합니다. 
### ... 아두이노json 라이브러리 설치 설명 추가 

라이브러리 설치 이후 헤더파일과 파싱할 문서 변수를 생성합니다. ArduinoJson 자세한 사용 방법은 공식 문서를 참조하세요.
```
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <SoftwareSerial.h>
#include <ArduinoJson.h>

const size_t capacity = JSON_ARRAY_SIZE(10);
DynamicJsonDocument jsonBuffer(512);
StaticJsonDocument<capacity> doc;
JsonArray packets = doc.to<JsonArray>();

...
```


우선 수신한 토픽에서 패킷을 추출하는 함수를 만들겠습니다. 다음과 같이 parseMqttMessage 함수를 생성합니다. 이전에 확인한 페이로드 형식을 참고해 실제 패킷 부분인 "data"를 파싱합니다.
```
{"pattern":"device/1234","data":"{\"start\":35,\"target\":20,\"command\":10,\"checksum\":30,\"end\":13}"}
```

parseMqttMessage 함수
```
void parseMqttMessage(char* topic, byte* payload) {
  /* ArduinoJson 라이브러리 함수 */
  deserializeJson(jsonBuffer, (char*)payload);
  deserializeJson(doc, (const char *)jsonBuffer["data"]);  
}

```

구독한 토픽을 수신했을 때 해당 페이로드를 파싱하도록 콜백 함수를 수정합니다. 다음과 같이 페이로드에서 패킷을 추출해 시리얼 통신으로 출력합니다.
callback 함수
```
void callback(char* topic, byte* payload, unsigned int length) {
  /* 서버에서 받아온 패킷 추출 */
  parseMqttMessage(topic, payload);
  
  for (JsonVariant packet : packets) {
	/* 패킷을 필요한 형태로 변경하여 전송 */ 
    Serial.print(packet.as<unsigned char>(), DEC);
  }

  Serial.println();
}
```

JSON 형태의 페이로드 파싱 기능을 추가했습니다. 이제 디바이스에 업로드하고 스웨거에서 MQTT 토픽을 발행합니다. 데이터 파싱 결과를 시리얼 모니터로 확인합니다.
#### .. 페이로드 파싱 결과 사진 추가 예정

이전에 백엔드 애플리케이션을 작성할 때 컨트롤러에 설정한 온도 토픽을 기억하시나요? 우리가 설정한 온도 토픽은 다음과 같습니다.
```
temperature/시리얼번호
```






