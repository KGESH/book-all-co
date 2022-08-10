# MQTT 실습

로컬 환경에서 MQTT 브로커와 클라이언트를 설치해 MQTT 통신을 실습합니다. Widnows 10, Linux, Mac 설치 방법이 각각 다르니 자세한 내용은 링크(https://mosquitto.org/download)를 참조합니다.

## Mac 설치 방법
Mac의 패키지 매니저인 home brew를 통해 설치할 수 있습니다.
```
brew install mosquitto
```

설치가 완료되면 다음 명령어로 MQTT 브로커를 실행할 수 있습니다.
```
brew services start mosquitto
```

다음 명령어로 MQTT 브로커가 실행 중인지 확인할 수 있습니다.
```
brew services list
```


## Linux(Ubuntu) 설치 방법
다음 명령어로 리포지토리에 있는 mosquitto 패키지를 설치할 수 있습니다.
```
sudo apt install mosquitto
```

설치가 완료되면 다음 명령어로 MQTT 브로커가 실행 중인지 확인할 수 있습니다.
```
sudo systemctl status mosquitto.service
```


## Windows 설치 방법
다음 링크(https://mosquitto.org/download)에서 본인 시스템(64bit, 32bit)에 맞는 설치 파일을 다운로드합니다.
# ... 추가 작성 예정




실습을 위해 mosquitto 클라이언트를 사용해 토픽을 구독 & 발행 합니다.

### Subscribe (구독)
```
mosquitto_sub -h localhost -p 1883 -t company/floor2/toilet
```
![[resources/ch1/3/1.png]]

### Publish (발행)
```
mosquitto_pub -h localhost -p 1883 -t company/floor2/toilet -m "hello"
```
![[resources/ch1/3/2.png]]

mosquitto client 옵션
- -h: MQTT 브로커의 IP 주소입니다. 우리는 로컬 환경에서 브로커와 클라이언트를 실행할 것이므로 로컬호스트로 설정합니다.
- -p: MQTT 브로커의 포트 번호입니다. MQTT는 기본적으로 보안 설정을 하지 않은 1883번 포트를 사용합니다. SSL을 통한 MQTT 통신을 원한다면 일반적으로 8883번 포트를 사용합니다.
- -t: 토픽입니다. '/' 로 토픽 레벨을 구분하며, 와일드카드(+, *) 를 포함할 수 있습니다.
- -m: 발행할 메시지입니다. 모든 구독자들은 브로커를 통해 발행자의 메시지를 수신합니다.
더 많은 정보를 확인하려면 다음 링크를 참조하세요.(https://mosquitto.org/documentation)

축하합니다. 다음과 같이 성공적으로 구독자가 브로커를 통해 발행자의 메시지를 수신했습니다. 앞서 설명했듯이 하나의 토픽을 여러 클라이언트들이 구독할 수 있습니다. 여러 개의 구독자 클라이언트를 실행하고 토픽을 구독해 보세요. 하나의 발행자가 토픽을 발행하면 모든 구독자들이 메시지를 수신합니다.

### 결과 이미지(...수정 예정)
(... MQTT Client 이미지)
(... 주의사항 기재 - 루트 토픽 '/'로 시작하지 않기)
![[resources/ch1/3/3.png]]