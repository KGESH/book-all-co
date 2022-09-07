# 개발 환경 구성

## Node.js 설치
NestJS는 Node.js 기반의 웹 프레임워크 이므로 Node.js를 설치해야합니다. Node.js 공식 홈페이지에서 본인의 개발 환경에 맞게 설치합니다. 안정화가 된 LTS(Long Term Support) 버전을 추천하며 설치 방법은 공식 홈페이지를 참조하세요.
Node.js 설치가 완료되었다면 다음 명령어로 터미널에서 설치된 버전을 확인할 수 있습니다.

```
node -v
```


Node.js를 설치하면 기본적으로 npm이 함께 설치됩니다.
NestJS는 상당히 똑똑한 CLI(command line interface)를 제공합니다. NestJS 개발 환경을 구성하기 위해 우선 @nestjs/cli를 설치해야합니다. 터미널에 다음 명령어로 cli를 설치합니다.

```
npm install -g @nestjs/cli
```

또한 다음과 같이 숏컷으로 install 대신 i로 대체가 가능합니다.

```
npm i -g @nestjs/cli
```

-g 옵션은 글로벌 환경으로 설치하겠다는 옵션으로, 모든 디렉토리에서 해당 패키지를 참조 할 수 있습니다. 
cli 설치가 끝나면 프로젝트를 생성할 디렉토리에서 다음 명령어로 NestJS 프로젝트를 생성합니다.

```
nest new first-app
```

first-app이 우리가 생성할 프로젝트의 이름이 됩니다. 프로젝트 구성에 어떤 패키지 매니저를 사용할 지 묻는데, 이 책에서는 npm을 사용해 진행하겠습니다.

![[resources/ch2/5/1.png]]

 npm을 선택후 enter키를 입력하면 잠시후 NestJS 백엔드를 실행시켜볼 수 있는 보일러 플레이트(설명 밖으로 빼기)가 생성됩니다. 다음 명령어로 우리의 첫 애플리케이션을 실행해봅니다.
```
cd first-app
```

```
npm run start
```

![[resources/ch2/5/2.png]]

처음 프로젝트를 생성하면 기본 포트가 3000번으로 설정되어 있습니다. 서버가 성공적으로 실행되면 웹 브라우저에서 주소창에 localhost:3000을 입력해 서버가 정상적으로 작동하는지 확인합니다.

![[resources/ch2/5/3.png]]

축하합니다. 우리의 첫 서버 프로그램이 정상 작동 하는 것을 확인했습니다!