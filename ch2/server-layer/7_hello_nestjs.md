# NestJS 둘러보기
앞서 생성한 first-app 프로젝트를 자신이 사용하는 코드 편집기로 열어서 프로젝트의 구조를 확인해봅니다.  대표적으로 Microsoft의 Visual Studio Code(vs code)와 JetBrains의 Web Storm 등이 있습니다. 자신에게 편한 개발환경을 선택하면 됩니다. 이 책에서는 Web Storm을 사용하여 진행하겠습니다.

## 첫 프로젝트 구조
처음 프로젝트 생성 시 프로젝트의 구조는 [그림 x.x : 첫 프로젝트 구조]과 같습니다.
![[tree.png]]
[그림 x.x : 첫 프로젝트 구조]

프로젝트 src 디렉토리의 main.ts 파일을 확인해봅시다.
src/main.ts
```
import { NestFactory } from '@nestjs/core';  
import { AppModule } from './app.module';  
  
async function bootstrap() {  
  const app = await NestFactory.create(AppModule);  
  await app.listen(3000);  
}  
bootstrap();
```

코드가 바로 이해되지 않아도 괜찮습니다. 우선 전체적인 흐름을 차근차근 알아봅시다. 대부분의 프로그래밍 언어와 마찬가지로 main.ts에서 NestJS 애플리케이션이 시작됩니다. NestJS 애플리케이션을 시작하기 위해 NestFactory 클래스를 사용합니다. 해당 클래스의 create 메소드는 우리가 사용할 애플리케이션 객체를 반환합니다. listen 메소드는 Nest 앱이 사용할 포트 번호를 받아 해당 포트로 들어온 요청에 응답합니다. 정리하면 main.ts 파일의 bootstrap() 함수는 NestJS 애플리케이션을 초기화 하는 함수라고 볼 수 있겠습니다.


## Controller
다음은 NestJS 공식 문서에서 설명하는 Controller(컨트롤러)에 대한 내용입니다.
컨트롤러는 들어오는 Request(요청)를 처리하고 클라이언트에 Response(응답)를 반환하는 역할입니다.
컨트롤러의 목적은 애플리케이션에 대한 특정 요청을 수신하는 것입니다. 라우팅 메커니즘을 통해 어떤 컨트롤러가 어떤 요청을 수신하는지 제어합니다.
기본적으로 컨트롤러를 만들기 위해 NestJS에서는 클래스와 데코레이터를 사용합니다. 데코레이터는 클래스를 필수 메타데이터와 연결하고 Nest가 라우팅 맵을 생성할 수 있도록 합니다(요청을 해당 컨트롤러에 연결합니다).


![[nestjs_controller.png]]
[그림 : 공식 문서 컨트롤러 설명]

공식 문서의 설명만 보면 무슨말인지 모르겠네요. 우리 프로젝트의 컨트롤러 코드를 보며 얘기해 봅시다.

src/app.controller.ts
```
import { Controller, Get } from '@nestjs/common';  
import { AppService } from './app.service';  
  
@Controller()  
export class AppController {  
  constructor(private readonly appService: AppService) {}  
  
  @Get()  
  getHello(): string {  
    return this.appService.getHello();  
  }  
}
```

cli를 사용하여 프로젝트를 생성하면 자동으로 생성되는 컨트롤러 코드입니다. 가장 먼저 우리 눈에 띄는 것은 AppController 클래스 이름 위의 @Controller()와 getHello() 메소드 이름 위의 @Get() 일 것입니다. 이처럼 @로 시작하는 문법을 Decorator(데코레이터)라고 합니다. 데코레이터는 클래스, 메서드, 프로퍼티, 매개변수등에 적용 가능합니다. 
NestJS는 서버가 요청을 처리하는데에 필요한 귀찮은 작업들을 데코레이터를 사용하여 나타냅니다. 덕분에 우리는 우리가 구현할 애플리케이션의 핵심 로직에 집중할 수 있습니다. 
AppController 클래스에 @Controller() 데코레이터와 getHello 메소드에 @Get() 데코레이터를 기술하는 것으로 우리는 AppController 클래스가 컨트롤러의 역할을 수행한다는 점과 getHello가 Get 요청에 대한 응답의 역할을 수행한다는 점을 알 수 있습니다. (...REST API 설명)

터미널에서 다음 명령어로 NestJS 애플리케이션을 실행합니다.
NestJS는 npm run start:dev로 개발 서버를 실행하면 핫스왑(...설명 추가)을 지원합니다.
```
npm run start:dev
```

웹 브라우저에서 주소창에 http://localhost:3000 으로 접속해봅시다. 정상적으로 접속이 되었다면 "Hello, World!" 라는 문자열을 볼 수 있습니다. 
이처럼 @Get() 데코레이터가 root('/' 가 생략)로 들어오는 요청을 처리한다는 것을 확인했습니다.
이번에는 http://localhost:3000/hello url로 접속해봅시다.  [그림 : hello not found]과 같이 404 HTTP StatusCode(상태 코드)와 함께 "/hello"를 찾을 수 없다는 메시지를 확인할 수 있습니다.
(... 확장 프로그램 설명크롬 브라우저의 JSON Viewr라는 확장프로그램을 사용하면 JSON 형식의 데이터를 보기 좋게 정렬해서 보여줍니다.
이는 크롬 웹 스토어에서 검색하여 쉽게 설치할 수 있습니다.)

![[hello_notfound.png]]
[그림 : hello not found]


다시 코드로 돌아와서 @Get() 데코레이터를 @Get('hello')로 수정합니다. 
src/app.controller.ts
```
@Get('hello')  
getHello(): string {  
  return this.appService.getHello();  
}
```

변경서항을 저장하고 서버가 재시작되면 웹 브라우저 주소창에 http://localhost:3000/hello url로 접속합니다. 처음 http://localhost:3000 에 접속했을때 만난 페이지를 확인할 수 있습니다. 이번에는 다시 웹 브라우저 주소창에 http://localhost:3000 에 접속해봅시다. 아마 예상했듯이 [그림 : root not found] 처럼 "/" 를 찾을 수 없다는 메시지를 확인할 수 있습니다.
![[root_notfound.png]]
[그림 : root not found]

이처럼 NestJS 애플리케이션은 데코레이터를 사용해 라우팅(routing)을 구현합니다. 기본적으로 Get() 데코레이터에 아무 값도 주지 않으면 디폴트 값으로 '/' (루트) 경로로 라우팅하여 사용자의 요청을 처리합니다.

마찬가지로 @Controller() 데코레이터도 문자열을 전달할 수 있습니다. 다음과 같이 컨트롤러 데코레이터를 @Controller('app')로 수정하겠습니다.
src/app.controller.ts
```
import { Controller, Get } from '@nestjs/common';  
import { AppService } from './app.service';  
  
@Controller('app')  
export class AppController {  
  constructor(private readonly appService: AppService) {}  
  
  @Get('hello')  
  getHello(): string {  
    return this.appService.getHello();  
  }  
}
```

이는 라우팅 경로의 prefix를 설정한 것이며 해당 컨트롤러 클래스 내부의 요청들은 http://localhost:3000/app/hello 처럼 prefix를 포함한 경로로 접근 가능합니다.
![[app_hello.png]]
[그림 : prefix app hello]

NestJS에서 url에 접근할 때 사용되는 것은 오직 데코레이터에 전달된 라우트 경로를 나타내는 문자열입니다. 클래스나 메소드의 이름은 개발자가 마음대로 지정할 수 있습니다. 메소드의 이름을 getHello가 아닌 getHelloMessage() 라고 수정해도 우리가 원하는대로 작동합니다.

이제 라우팅이 어떻게 작동하는지 배웠으니 요청에 대한 응답으로 "Hello World!" 가 아닌 "hello IoT" 라는 문자열로 바꿔서 출력해 보겠습니다. 컨트롤러의 코드를 보면 우리의 앱 컨트롤러는 생성자로 AppService 타입의 객체를 받고 있습니다. 그런데 생성자 매개변수로 객체를 받는건 알겠는데, private과 readonly 등 키워드를 붙이는 것을 처음 보는 분도 있을듯합니다. TypeScript는 생성자 매개변수에 private(public) readonly 등의 키워드를 추가하여 프로퍼티 할당에 대한 숏컷을 제공합니다.  다음 두 클래스는 같은 기능의 클래스입니다.

```
class Normal {  
  private readonly name: string;  
  
  constructor(name: string) {  
    this.name = name;  
  }  
}  
  
class ShortCut {  
  constructor(private readonly name: string) {}  
}
```

다시 앱 컨트롤러 코드로 돌아와서 코드를 봅시다. 생성자에서 AppService 객체를 받아 해당 객체의 getHello 메소드를 사용하고 있는 걸 보면 아마 저 메소드에서 "Hello, World!"와 관련된 로직이 있다는 걸 예상할 수 있습니다. 이제 서비스 코드로 넘어가봅시다.

src/app.service.ts
```
import { Injectable } from '@nestjs/common';  
  
@Injectable()  
export class AppService {  
  getHello(): string {  
    return 'Hello World!';  
  }  
}
```

생각보다 별거 없어서 놀랐나요? 서비스 계층의 코드는 웹 브라우저와 통신을 위한 거창한 코드도 없이 우리의 목적(비지니스 로직)인 "Hello, World!" 라는 문자열만 덩그러니 반환하고있습니다. 정말 "Hello World!"를 수정하면 브라우저에서 요청한 응답이 바뀌는지 확인해봅시다.


src/app.service.ts
```
import { Injectable } from '@nestjs/common';  
  
@Injectable()  
export class AppService {  
  getHello(): string {  
    return 'Hello IoT!';  
  }  
}
```

코드를 저장하고 http://localhost:3000/app/hello 로 접속해보면 [그림 : hello IoT] 같이 다른 응답을 확인할 수 있습니다.

![[hello_iot.png]]
[그림 : hello IoT]


이처럼 계층을 나누는 일은 중요합니다. 웹 브라우저(클라이언트)와 통신하는 역할(책임)은 컨트롤러에게 맡기고 서비스는 오직 자신의 비지니스 로직만 수행합니다. 이러한 구조는 프로젝트의 규모가 커질수록 유지보수와 확장하기 쉽게 만들어줍니다. 이 책에서는 다루지 않지만 객체지향의 5가지 설계원칙 SOLID를 따로 찾아보는것을 추천합니다.(... 알나지 SOLID 간략 설명)


## 라우팅
지금까지 우리는 서버에게 요청하면 정적인 데이터를 응답 받는 방법을 학습했습니다. 이제 서버에게 우리가 원하는 데이터를 요청해보겠습니다. 데이터베이스에 사용자 정보가 저장되어있다고 가정하고 데이터베이스 조회에 사용할 사용자의 id를 URL에서 동적으로 전달받을 수 있습니다. 컨트롤러에서 다음과 같이 사용자 id를 가져오고 출력해봅니다. 참고로 getUser 메소드의 리턴 문자열은 백틱(키보드 ~ 키)으로 감싸져있습니다. 헷갈리지 않게 주의하세요.
src/app.controller.ts
```
import { Controller, Get, Param } from '@nestjs/common';  
import { AppService } from './app.service';  
  
@Controller('app')  
export class AppController {  
  constructor(private readonly appService: AppService) {}  
  
  @Get('hello')  
  getHello(): string {  
    return this.appService.getHello();  
  }  
  
  @Get('user/:id')  
  getUser(@Param('id') userId: string): string {  
    console.log(`Get user ID: `, userId);  
    return `User ID: ${userId}`;  
  }  
}
```

![[route_param.png]]
[그림 : 라우트 파라미터]

라우트 파라미터를 다룰때는 항상 순서에 주의해야합니다. 예를들어 localhost:3000/app/user/hello로 요청했을때 다음과 같은 컨트롤러는 어떤 응답을 줄까요? getHello메소드의 @Get 데코레이터를 다음과 같이 수정하고 getUser 메소드가 먼저 정의되게 코드를 작성합니다.

```
import { Controller, Get, Param } from '@nestjs/common';  
  
@Controller('app')  
export class AppController {  
  /** Wrong example */  
  @Get('user/:id')  
  getUser(@Param('id') userId: string): string {  
    return `User ID: ${userId}`;  
  }  
  
  @Get('user/hello')  
  getHello(): string {  
    return `Hello user!`;  
  }  
}
```

![[route_param_wrong.png]]
[그림 : 라우트 파라미터 잘못된 예]

user/hello 라우팅 요청이 작동되길 기대했지만 user/:id 라우팅 요청이 작동한 것을 확인할 수 있습니다. 이처럼 라우트 파라미터를 사용할 때는 컨트롤러의 메소드 정의 순서에 주의해야합니다. 먼저 정의된 @Get('user/:id') 데코레이터가 user/hello URL에서 hello를 id로 인식해서 생긴 문제입니다. 다음과 같이 메소드 순서를 바꿔 실행해보겠습니다.

```
import { Controller, Get, Param } from '@nestjs/common';  
  
@Controller('app')  
export class AppController {  
  /** Correct example */  
  @Get('user/hello')  
  getHello(): string {  
    return `Hello user!`;  
  }  
  
  @Get('user/:id')  
  getUser(@Param('id') userId: string): string {  
    return `User ID: ${userId}`;  
  }  
}
```

![[route_param_correct.png]]
[그림 : 라우트 파라미터 옳은 예]


이제 우리가 원하는 대로 작동하는 것을 확인할 수 있습니다. 


## 요청 객체 (Request object)
앞서 URL에 사용자의 id를 담아 전달한것처럼 클라이언트는 서버에게 요청(request)을 보낼 때 여러 정보를 함께 전송합니다. 컨트롤러에서는 이러한 클라이언트의 요청을 객체로 변환해 핸들링할 수 있는 기능을 제공합니다. 다음과 같이 @Req() 데코레이터를 사용해 객체가 어떤 모습인지 출력해볼 수 있습니다.

```
import { Controller, Get, Param, Req } from '@nestjs/common';  
import { Request } from 'express';  
  
@Controller()  
export class AppController {  

  @Get(':id')  
  getRequest(@Req() req: Request, @Param('id') userId: string): string {  
    console.log(req);  
    console.log('------------------------------');  
    console.log(req.params);  
  
    return `UserId: ${userId}`;  
  }  
}
```

요청 객체를 출력해보면 HTTP 요청에 관한 많은 정보가 출력되며 우리가 URL에서 가져온 파라미터도 포함되어 있다는 것을 확인할 수 있습니다. 요청 객체는 파라미터 외에도 쿼리 스트링, 헤더, 본문(body) 등 많은 정보를 가집니다. 더 자세한 사항은 Express 공식 문서를 참조하세요.

요청 객체를 직접 다루기보다 @Param(), @Query(), @Body() 데코레이터를 사용해서 데이터를 핸들링하는 것을 추천합니다.


## 응답 (Response)
우리는 여태까지 컨트롤러에서 데이터를 리턴하는 방식으로 요청에 대한 응답을 처리했습니다. 컨트롤러에서 string, number, boolean 등의 원시 타입(primitive type)을 리턴하면 직렬화(serialize) 없이 값을 보내고, 객체를 리턴한다면 자동으로 객체를 직렬화하여 JSON 형식으로 변환하여 응답합니다. NesJS 공식 문서에서는 이 방법을 권장하지만 다음과 같이 응답 객체를 직접 핸들링할 수도 있습니다.

```
import { Controller, Get, Res } from '@nestjs/common';  
import { Response } from 'express';  
  
@Controller()  
export class AppController {  

  @Get()  
  getHello(@Res() res: Response) {  
    return res.status(200).send('hello!');  
  }  
}
```


주의사항: @Res() 데코레이터를 사용하면 반드시 응답객체를 리턴해야합니다. 예를들어 다음 코드와 같이 @Res() 데코레이터를 선언하고 응답 객체가 아닌 다른 값을 리턴하면 응답을 전달받지 못합니다.

```
import { Controller, Get, Res } from '@nestjs/common';  
import { Response } from 'express';  
  
@Controller()  
export class AppController {  
  @Get()  
  getHello(@Res() res: Response) {  
    return 'hello world!'; /** Wrong */  
    // return res.status(200).send('hello!');  /** Correct */  
  }  
}
```

지금까지 사용자와 백엔드와의 인터페이스 역할을 하는 컨트롤러를 알아보았습니다. 와일드카드, Header 데코레이터, HttpCode 데코레이터등 유용한 기능들이 많이 있으니 더 자세한 내용은 공식 문서를 참조하세요. https://docs.nestjs.com/controllers

