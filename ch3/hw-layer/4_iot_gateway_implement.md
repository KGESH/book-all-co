# IoT 모니터링 프로젝트 - 게이트웨이 구현
이전에 EC2에 배포한 IoT 모니터링 백엔드 애플리케이션과 상호작용하는 게이트웨이를 작성합니다. 책의 예제는 ESP8266 모듈에 업로드 하기 위한 예제이며 이전에 사용한 PubSubClient 라이브러리를 사용할 예정입니다. 또한 카페에서  C++로 작성된 리눅스에서 작동하는 게이트웨이 소스를 확인할 수 있습니다.


이전처럼 PubSubClient 예제를 불러옵니다. 예제 소스에 함수를 추가로 작성해 IoT 모니터링 프로젝트의 게이트웨이를 구현하겠습니다. 또한 백엔드 애플리케이션이 보내는 메시지는 JSON 형식이므로 JSON 파싱 라이브러리를 설치합니다. 예제에서는 ArduinoJson이라는 라이브러리를 사용하겠습니다. 다음과 같이 ArduinoJson 라이브러리를 추가합니다.

### ... 아두이노json 라이브러리 설치 설명 추가 

라이브러리 설치 이후 헤더파일과 파싱할 문서 변수를 생성합니다.

```
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <SoftwareSerial.h>
#include <ArduinoJson.h>

const size_t capacity = JSON_ARRAY_SIZE(9);
DynamicJsonDocument jsonBuffer(512);
StaticJsonDocument<capacity> doc;
JsonArray packets = doc.to<JsonArray>();

...

```



본인의 와이파이 설정에 맞게 ssid와 password를 설정합니다. mqtt_server는 MQTT 브로커가 배포되어 있는 EC2 인스턴스의 공개 IP로 설정합니다.
```
...
const char* ssid = "your-wifi";
const char* password = "your-password";
const char* mqtt_server = "123.123.123.123";
...
```



우선 수신한 토픽에서 패킷을 추출하는 함수를 만들겠습니다. 다음과 같이 makePacket 함수를 생성합니다.
makePacket 함수
```

```
게이트웨이에서 구독한 토픽을 수신했을 때 해당 토픽을 처리하는 콜백 함수를 수정하겠습니다. 다음과 같이 토픽에서 패킷을 추출해 시리얼 통신으로 임베디드 시스템에 전달합니다.
callback 함수
```


```




이전에 백엔드 애플리케이션을 작성할 때 컨트롤러에 설정한 온도 토픽을 기억하시나요? 우리가 설정한 온도 토픽은 다음과 같습니다.
```
temperature/시리얼번호
```






