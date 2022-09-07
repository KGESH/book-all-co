
# EC2에 Docker 설치하기

EC2에 직접 MQTT 브로커와 웹 애플리케이션을 설치하고 실행할 수도 있지만 관리의 편의성을 위해 도커 컨테이너 환경에서 실행하겠습니다. (... 도커 컨테이너 설명), (... 이미지들 제거  고민중) 
우선 EC2 인스턴스에 도커를 설치해야 합니다. 우리는 이전에 Ubuntu 운영체제를 선택했으므로 우분투 설치방법으로 설명합니다. 다른 운영체제나 더 자세한 방법을 알고 싶다면 다음 링크(https://docs.docker.com/engine/install/)를 참조합니다


### 오래된 버전 삭제
만약 이전 버전의 도커가 설치되어 있다면 제거합니다. (... 문제 없다고 설명 추가 예정)
```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

![[resources/ch1/5/23.png]]
### Repository 설정
저장소 이용을 위한 패키지들을 설치합니다.
```
sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```
![[resources/ch1/5/24.png]]
Docker offical GPG Key 추가
```
sudo mkdir -p /etc/apt/keyrings 
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```
![[resources/ch1/5/25.png]]
Stable repository 등록 
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
![[resources/ch1/5/26.png]]
Docker 설치
```
 sudo apt-get update
 sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
설치를 묻는 Y / N 메시지가 나오면 Y를 입력해서 설치를 진행합니다.
![[resources/ch1/5/28_1.png]]
### 설치 확인
다음과 같은 명령어로 현재 설치된 버전을 확인합니다.
```
sudo docker version
```
![[resources/ch1/5/29.png]]

### 설치된 이미지들 확인
다음과 같은 명령어로 현재 설치된 컨테이너 이미지들을 확인할 수 있습니다. 아직 우리는 아무 이미지도 설치하지 않았기 때문에 아무 이미지도 출력되지 않습니다.
```
sudo docker images
```

### sudo 명령 없이 docker 사용하기
docker 명령어 실행 시 sudo 권한이 필요합니다. 만약 sudo 명령어 없이 docker 명령어를 실행하면 권한 에러 메시지가 발생합니다. 학습 시 일일이 sudo 명령어를 입력하기 귀찮다면 다음 명령어를 실행하고 서버를 재시작하면 sudo 명령어를 붙이지 않고 학습을 진행할 수 있습니다.
```
sudo usermod -aG docker $USER
```
![[resources/ch1/5/30.png]]
AWS 콘솔에서 인스턴스를 재시작하거나 다음 명령어로 서버를 재시작 할 수 있습니다. 재시작하게 되면 터미널의 연결이 끊어지며 재연결해야 합니다.
```
sudo reboot
```

터미널에 재연결하고 sudo 없이 명령어를 실행해 봅니다. 
```
docker images
```

![[resources/ch1/5/30_1.png]]

축하합니다. 여기까지 성공적으로 완료했다면 컨테이너 환경에서 MQTT 브로커와 웹 애플리케이션을 사용할 준비가 끝났습니다.

