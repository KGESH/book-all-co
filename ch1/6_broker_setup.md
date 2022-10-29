# MQTT 브로커 설정

이전에 도커 설정까지 완료한 EC2 인스턴스에서 MQTT 브로커 컨테이너를 실행하고 로컬에서 EC2에 실행중인 MQTT 브로커에 토픽을 발행 & 구독하여 제대로 작동하는지 확인해 봅니다. 아직 EC2 인스턴스에 도커 설정을 마치지 않았다면 EC2 인스턴스에 도커 설정 완료 이후에 다음 내용을 진행합니다.

EC2 인스턴스에 접속합니다. 다음과 같이 EC2에서 eclipse-mosquitto 이미지를 pull합니다.
```
docker pull eclipse-mosquitto
```

docker images 명령어로 내려받은 이미지들을 확인할 수 있습니다.
```
docker images
```

접속한 계정의 디렉토리에 mosquitto/config 디렉토리를 생성합니다.
```
mkdir ~/mosquitto
mkdir ~/mosquitto/config
```

디렉토리들을 생성하고 생성된 mosquitto/config 디렉토리 내부에 다음과 같이 mosquitto.conf 설정 파일을 생성합니다. 본인이 원하는 방법으로 mosquitto.conf를 작성하면 됩니다. 이 책에서는 vi 편집기로 mosquitto.conf 파일을 작성했습니다. 
```
vi ~/mosquitto/config/mosquitto.conf
```

MQTT 통신에 사용할 1883번 포트를 허용하고 파일을 저장합니다.
~/mosquitto/config/mosquitto.conf
```
allow_anonymous true
listener 1883
```

mosquitto.conf 파일 내용이 잘 작성되었는지 출력해 봅니다.
```
cat ~/mosquitto/config/mosquitto.conf
```


## 도커 컴포즈(docker-compose)로 컨테이너들 묶기
실제 서비스를 운영하게 되면 여러 컨테이너가 서로 상호작용하는 경우가 빈번히 생깁니다. 도커 컴포즈(docker-compose)는 여러 컨테이너들을 묶어서 관리할 수 있는 도구입니다.

다음 명령어로 docker-compose를 설치할 수 있습니다. 이 책에서는 간단하게 apt로 설치합니다. 만약 최신 버전의 도커 컴포즈를 설치하고 싶다면 도커 컴포즈 공식 문서를 참조하세요.
```
sudo apt install docker-compose
```

설치가 완료되면 다음 명령어로 설치된 도커 컴포즈의 버전을 확인할 수 있습니다.
```
docker-compose -v
```


이제 컨테이너들을 묶어주기위한 설정 파일을 생성합니다. 파일 이름은 docker-compose이며 확장자는 .yml 또는 .yaml 을 사용합니다. 다음과 같이 docker-compose.yml 파일을 생성합니다.
```
vi ~/docker-compose.yml
```

도커 컴포즈 문법의 버전을 명시하고 사용할 서비스들을 정의합니다. 이 책에서 서비스의 이름은 mosquitto로 진행하겠습니다. mosquitto 서비스의 이미지는 이전에 내려받은 eclipse-mosquitto로 지정합니다. 컨테이너가 사용할 1883번 포트와 실제 EC2 인스턴스의 1883번 포트를 맵핑해줍니다. 볼륨은 이전에 생성한 mosquitto 디렉토리와 컨테이너 내부의 mosquitto 디렉토리와 맵핑해줍니다. 더 자세한 옵션 설명들은 도커 컴포즈 공식 문서를 참조하세요.
~/docker-compose.yml
```
version: "3"
services:
  mosquitto:
    image: eclipse-mosquitto
    ports:
      - "1883:1883"
    volumes:
      - ~/mosquitto:/mosquitto/

```


파일 작성 이후 도커 컴포즈 설정 파일에 정의된 서비스들을 실행합니다. -d 옵션은 컨테이너를 백그라운드에서 실행시킵니다. docker-compose.yml 파일과 같은 경로에 있다면 -f (컴포즈 파일 경로) 옵션을 생략할 수 있습니다.
```
docker-compose -f ~/docker-compose.yml up -d
```

실행 중인 컨테이너를 확인할 수 있습니다.
```
docker ps
```

컨테이너들을 종료하려면 down 명령어를 사용합니다. up과 마찬가지로 -f 옵션을 생략할 수 있습니다.
```
docker-compose -f ~/docker-compose.yml down
```

다시 docker-compose를 실행하고 로컬 PC에서 구독, 발행을 해봅니다.
```
docker-compose -f ~/docker-compose.yml up -d
```

로컬 PC의 MQTT 클라이언트로 EC2의 MQTT 브로커를 구독합니다. -h 옵션으로 지정할 호스트의 IP는 이전에 생성한 EC2 인스턴스의 공개(탄력적) IP 주소를 지정합니다. EC2 인스턴스의 공개 IP는 AWS EC2 페이지에서 확인할 수 있습니다.
```
mosquitto_sub -h 123.123.123.123 -p 1883 -t iot/+
```

마찬가지로 구독 중인 토픽에 대해 메시지를 발행합니다. 토픽을 발행하면 EC2의 MQTT 브로커를 거쳐 해당 토픽을 구독 중인 클라이언트가 메시지를 수신합니다.
```
mosquitto_pub -h 123.123.123.123 -p 1883 -t iot/sensor -m "ok"
```



