# 온도 모니터링 시스템 API

우리가 단계적으로 개발하려고 했던 시스템을 기억하시나요?
우리 온도 모니터링 백엔드의 요구사항은 다음과 같습니다.

1. MQTT 토픽을 구독하여 MQTT 브로커를 통해 토픽을 전달받을 수 있습니다.
2. 전달받은 토픽에서 토픽 발행자의 id와 온도 데이터를 얻어 데이터베이스에 저장할 수 있습니다.
3. 웹 브라우저에서 토픽 발행자의 id로 온도 데이터를 요청하면 데이터베이스에 저장된 온도 데이터들을 보여줍니다.

이번 장에서는 데이터베이스에 온도 데이터들이 저장되어있다고 가정하고 (요구사항 1, 2가 구현되었다고 가정하고) 세번째 요구사항인 웹 브라우저(프론트엔드)가 서버(백엔드)에게 id로 온도 데이터를 요청하면 데이터베이스에 저장된 온도 데이터들을 전달해주는 API를 만들어보겠습니다. 

코딩을 시작하기 전에 먼저 우리가 만들고자 하는 API를 설계해봅시다. 프론트엔드 입장에선 백엔드에서 어떻게 온도 데이터를 수집하는지 알 필요가 없습니다. 물론 알 수도 없고요. 그저 백엔드에 온도 데이터가 있다고 믿고 데이터를 요청하는 행위만 할 뿐입니다. 우리는 프론트엔드에서 온도 데이터 요청에 필요한 인터페이스를 제공해야합니다. 

온도 모니터링 시스템에서 프론트엔드에게 제공할 수 있는 기능을 다음과 같이 정의해봅시다.
1. 온도 센서 등록
2. 등록된 센서의 id로 가장 최근 온도 조회
3. 등록된 센서의 id와 yyyy/MM/dd ~ yyyy/MM/dd 범위로 기간내의 온도 조회

우리는 아직 데이터베이스를 사용하는 방법을 학습하지 않았습니다. 그래서 일단 메모리(변수)에 데이터를 저장해놓고 API의 요구사항이 충족되면 메모리에 저장하는 기능을 데이터베이스로 저장하는 기능으로 바꿔볼 것입니다. 이 과정에서 계층(layer)을 나눠야 이유를 이해할 수 있을 겁니다.


## Project Setup

API 요구사항을 정의했으니 처음부터 프로젝트를 설정해봅시다.
이전의 예제들을 따라왔다면 @nestjs/cli는 이미 설치가 완료 되었을겁니다. 만약 CLI 설치를 안했다면 다음 명령어로 전역(global) 설치해줍니다.

```
npm i -g @nestjs/cli
```

프로젝트의 이름을 정하고 초기 설정을 진행합니다. 저는 iot-monitoring 으로 설정했습니다.

```
nest new iot-monitoring
```

패키지 매니저는 npm으로 선택합니다.
![[nest_monitoring_new.png]]
[그림 : Nest 모니터링 프로젝트 초기 설정]
초기 설정이 완료되면 본인의 코드 편집기를 열어 다음 명령어로 서버를 실행합니다. 이전 예제 프로젝트와 같은 3000번 포트를 사용하기 때문에 이전 예제 서버가 작동중이라면 종료하고 서버를 실행합니다.

![[nest_setup_done.png]]
[그림 : Nest 모니터링 앱 실행]
```
npm run start:dev
```

http://localhost:3000 에 접속해서 [그림 : Nest 모니터링 앱 접속 성공] 같이 Hello World! 페이지가 성공적으로 접속되는지 확인합니다.

![[nest_hello.png]]


## Resources
우리가 정의할 API는 REST API 입니다. (... REST API 설명) 터미널에 [그림 : Nest 모니터링 앱 리소스 생성] 같이 NestJS의 CLI 명령어를 입력합니다.

```
nest generate resource sensors
```

![[nest_g_sensors_1.png]]
[그림 : Nest 모니터링 앱 리소스 생성1]

[그림 : Nest 모니터링 앱 리소스 생성2] 같이 REST API를 선택하고 CRUD 엔트리 포인트를 생성하겠냐는 질문에 Y를 입력합니다. 디폴트가 Yes이므로 엔터를 눌러 진행합니다.

![[nest_g_sensors_3.png]]
[그림 : Nest 모니터링 앱 리소스 생성2] 

컨트롤러, 서비스 등의 코드들을 직접 생성해도 되지만, 이처럼 CLI를 사용하여 보일러 플레이트 코드들을 쉽게 생성할 수 있습니다.

[그림 : Nest 모니터링 앱 구조] 는 보일러 플레이트 코드가 생성된 이후 디렉토리 구조입니다.

![[tree_sensors.png]]


## Controller

컨트롤러에서 자동 생성된 CRUD 엔드포인트들을 확인합니다. REST 정의에 따라 Create, Read, Update, Delete 기능이 자동 생성됨을 확인할 수 있습니다.

src/sensors/sensors.controller.ts
```
import { Controller, Get, Post, Body, Patch, Param, Delete } from '@nestjs/common';  
import { SensorsService } from './sensors.service';  
import { CreateSensorDto } from './dto/create-sensor.dto';  
import { UpdateSensorDto } from './dto/update-sensor.dto';  
  
@Controller('sensors')  
export class SensorsController {  
  constructor(private readonly sensorsService: SensorsService) {}  
  
  @Post()  
  create(@Body() createSensorDto: CreateSensorDto) {  
    return this.sensorsService.create(createSensorDto);  
  }  
  
  @Get()  
  findAll() {  
    return this.sensorsService.findAll();  
  }  
  
  @Get(':id')  
  findOne(@Param('id') id: string) {  
    return this.sensorsService.findOne(+id);  
  }  
  
  @Patch(':id')  
  update(@Param('id') id: string, @Body() updateSensorDto: UpdateSensorDto) {  
    return this.sensorsService.update(+id, updateSensorDto);  
  }  
  
  @Delete(':id')  
  remove(@Param('id') id: string) {  
    return this.sensorsService.remove(+id);  
  }  
}
```

[그림 : sensors GET] 같이 http://localhost:3000/sensors 
http://localhost:3000/sensors/100 들에 접속하여 Get 요청들이 잘 작동하는지 확인해봅니다.
![[get_sensors.png]]
[그림 : sensors GET]


![[get_sensors_id.png]]

Get 메소드들은 잘 작동하는군요. 그렇다면 Post와 Delete같은 메소드들은 어떻게 테스트할까요? 이러한 API 개발에 필요한 도구들은 대표적으로 postman, insomnia, swagger 등이 있습니다. 이 책에서는 NestJS 공식 문서에서 설명하는 스웨거(swagger) ui를 사용하여 API를 명세하고 실행해 봅니다. 터미널에 다음 명령어로 스웨거 패키지들을 설치합니다.

```
npm i @nestjs/swagger swagger-ui-express
```

설치가 완료되면 다음과 같이 main.ts에 다음과 같이 스웨거 설정 코드를 작성합니다.

src/main.ts
```
import { NestFactory } from '@nestjs/core';  
import { AppModule } from './app.module';  
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';  
  
async function bootstrap() {  
  const app = await NestFactory.create(AppModule);  
  
  const swaggerConfig = new DocumentBuilder()  
    .setTitle('IoT Monitoring')  
    .setDescription('Temperatures monitoring API')  
    .setVersion('1.0')  
    .addTag('sensors')  
    .build();  
  
  const document = SwaggerModule.createDocument(app, swaggerConfig);  
  
  /** 스웨거 문서에 접근하기 위한 경로를 설정합니다. */  
  SwaggerModule.setup('api-docs', app, document);  
  
  await app.listen(3000);  
}  
  
bootstrap();
```

http://localhost:3000/api-docs 에 접속하여 API 문서가 생성되었는지 확인합니다.
  


이곳에서 우리가 작성한 API를 확인할 수 있습니다. [그림 : swagger GET 1], [그림 : swagger GET 2], [그림 : swagger GET 3] 과 같이 sensors 엔드포인트에 GET 요청을 보내봅시다.




![[swagger_get_2.png]]
[그림 : swagger GET 1]

![[swagger_get_3.png]]
[그림 : swagger GET 2]

![[swagger_get_4.png]]
[그림 : swagger GET 3]

GET 요청에 대한 응답 결과로 [그림 : swagger GET 4]과 같은 결과를 얻었습니다. 이처럼 스웨거에서 우리가 제작한 API를 테스트해볼 수 있습니다. 또한 이렇게 API를 문서화하면 요청에 어떤 파라미터들이 필요하고 어떤 응답이 돌아오는지 알 수 있으므로 다른 개발자와의 협업에 큰 도움이 됩니다.

![[swagger_get_5.png]]
[그림 : swagger GET 4]


## DTO (Data transfer object)

DTO란 Data transfer object의 약자로 계층간 데이터를 공유할때 사용하는 객체입니다. 예시로 사용자가 웹 브라우저에서 센서를 등록 한다고 생각해봅시다. 사용자는 센서 등록 페이지에서 서버에게 전달할 정보를 기입합니다.  아마도 센서 종류, 센서 이름(별칭), 시리얼 넘버 등이 있겠지요. 이러한 센서 등록에 필요한 정보를 객체에 담아서 서버에 전송합니다. DTO는 기본적으로 비즈니스 로직을 가지지 않고 순수하게 데이터만을 주고받기 위한 객체입니다. 서버는 클라이언트에게 전달받은 DTO를 사용해 센서 등록 등 서비스에 필요한 비즈니스 로직을 처리합니다. 또한 class-validator라는 라이브러리를 사용하여 API 호출에 필요한 데이터를 빠뜨리고 호출 하는 실수를 방지할 수 있습니다. 터미널에서 다음 명령어로 class-validator를 설치합니다.

```
npm i class-validator
```

다음과 같이 사용자가 센서 등록에 사용할 DTO를 정의해보겠습니다. CLI로 센서 리소스의 보일러 플레이트 코드를 생성했다면 다음과 src/sensors/dto 디렉토리에 dto 파일이 생성 되었을겁니다. 자동 생성된 CreateSensorDto 클래스를 다음과 같이 수정합니다.

src/sensors/create-sensor.dto.ts
```
import { IsNumber, IsString } from 'class-validator';  
import { ApiProperty } from '@nestjs/swagger';  
  
export class CreateSensorDto {  
  @ApiProperty()  // Swagger에서 API 요청에 필요한 객체의 속성들을 확인하기 위한 데코레이터
  @IsString()    // Class validator에서 검증할 데코레이터
  public readonly name: string;  
  
  @ApiProperty()  
  @IsString()  
  public readonly sensorType: string;  
  
  @ApiProperty()  
  @IsNumber()  
  public readonly serialNumber: number;  
}
```

@ApiProperty() 데코레이터를 지정해주지 않은 프로퍼티는 스웨거에서 확인이 불가능합니다. 따라서 다른 개발자와 협업시 해당 API에서 사용하는 DTO가 어떤 프로퍼티들로 구성되어있는지 알려야한다면 @ApiProperty() 데코레이터를 사용해줍니다.
@IsString(), @IsNumber(), @IsEnum() 등의 데코레이터는 이름에서 유추할 수 있듯이 API 요청시 해당 프로퍼티가 데코레이터에 명시한 타입인지 검사합니다. 만약 @IsNumber() 데코레이터로 명시한 속성에 문자열 데이터가 들어온다면 해당 요청을 거부합니다. 이렇게 데코레이터를 통해 실수를 근원에 방지할 수 있으니 적극적으로 활용해줍니다.

센서 생성을 위한 DTO를 정의했으니 센서를 생성하는 코드를 작성할겁니다. 그러나 우리는 아직 데이터베이스를 적용하는 방법을 모르기때문에 데이터베이스 역할을 해줄 코드가 필요합니다. 가장 간단한 방법으로 클래스의 멤버 변수로 배열을 하나 만들어서 센서가 생성될 때마다 배열에 push하는 방법으로 진행하겠습니다. 센서 서비스 클래스에 멤버 변수를 만들 수도 있지만, 센서 서비스 클래스가 오직 비즈니스 로직의 책임을 지게 하기 위해 데이터를 저장하는 클래스를 새로 생성하겠습니다. 이번에는 CLI를 사용하지 않고 직접 sensors.repository.ts 파일을 생성 하겠습니다. 파일의 경로는 src/sensors/sensors.repository.ts 입니다.


센서 리파지토리로 이동하여 다음과 같이 데이터베이스 역할을 할 클래스를 작성합니다. 앞서 설명한 것처럼 데이터베이스 연동법을 아직 모르기때문에 메모리에 데이터베이스 역할을 할 sensors배열을 생성합니다. 데이터 생성 요청마다 자동으로 id가 증가하고 생성 시간까지 저장하는 sensorEntity 객체를 배열에 push하는 saveSensor 메소드, 데이터베이스에서 센서의 id로 조회하는 findOneById 메소드, 데이터베이스의 모든 센서들을 반환하는 findAll 메소드를 작성합니다. 임시로 사용할 sensors 배열의 타입은 any[] 타입으로 지정하겠습니다. 그리고 @Injectable() 데코레이터를 빼먹지 않고 반드시 작성해주세요! @Injectable() 데코레이터는 잠시후 Provider에서 설명합니다.

src/sensors/sensors.repository.ts
```
import { CreateSensorDto } from './dto/create-sensor.dto';  
import { Injectable } from '@nestjs/common';  
  
@Injectable()  
export class SensorsRepository {  
  sensors: any[] = [];  
  
  save(createSensorDto: CreateSensorDto): any {  
    /** 임시 데이터베이스에 저장할 센서 객체 */  
    const sensorEntity = {  
      id: this.sensors.length,  
      name: createSensorDto.name,  
      type: createSensorDto.sensorType,  
      serialNumber: createSensorDto.serialNumber,  
      createdAt: new Date(),  
    };  
  
    this.sensors.push(sensorEntity);  
    return sensorEntity;  
  }  
  
  findOneById(sensorId: number): any {  
    return this.sensors[sensorId];  
  }  
  
  findAll(): any[] {  
    return this.sensors;  
  }  
}
```

데이터베이스의 책임을 가진 센서 리파지토리를 생성했습니다. 이제 비즈니스 로직의 책임을 가진 센서 서비스로 이동해서 CLI가 생성한 템플릿 코드를 지우고 다음과 같이 작성합니다. 센서 생성을 위한 createSensor 메소드, 저장된 모든 센서를 가져오는 getAllSensors 메소드, 센서의 id로 데이터베이스를 조회하는 getSensor 메소드까지 총 3개의 메소드를 작성합니다. getSensor 메소드에서는 데이터베이스에 저장된 센서가 없다면 BadRequestException 예외를 던져서 사용자에게 센서를 찾지 못했다고 메시지를 전달합니다.

src/sensors/sensors.service.ts
```
import { BadRequestException, Injectable } from '@nestjs/common';  
import { CreateSensorDto } from './dto/create-sensor.dto';  
import { SensorsRepository } from './sensors.repository';  
  
@Injectable()  
export class SensorsService {  
  constructor(private readonly sensorsRepository: SensorsRepository) {}  
  
  createSensor(createSensorDto: CreateSensorDto): any {  
    const saveResult = this.sensorsRepository.save(createSensorDto);  
    console.log('Created: ', saveResult);  
  
    return saveResult;  
  }  
  
  getSensor(sensorId: number): any {  
    const foundResult = this.sensorsRepository.findOneById(sensorId);  
    if (!foundResult) {  
      throw new BadRequestException('sensor not found!');  
    }  
  
    return foundResult;  
  }  
  
  getAllSensors(): any[] {  
    return this.sensorsRepository.findAll();  
  }  
}
```

현재 서비스 코드는 아주 단순합니다. 컨트롤러에게 전달 받은 센서 생성에 필요한 DTO를 리파지토리에 전달하여 '데이터베이스 저장'이라는 책임을 리파지토리에 위임합니다. 리파지토리에서 저장한 결과를 콘솔로 출력하고 그대로 컨트롤러에게 반환합니다. 앞으로 여러 요구사항들이 추가되면 서비스 코드는 더 길어질 것입니다. 서비스가 점점 커지게된다면 하나의 서비스가 너무 많은 책임을 가진게 아닌지 생각해보고 분할할 수 있다면 분할해야합니다.

센서 생성(Create), 조회(Read) 서비스를 작성했으니 이를 사용하여 다음과 같이 센서 컨트롤러를 작성합니다. 비즈니스 로직과 데이터베이스 접근이라는 관심사를 분리했기 때문에 매우 간결한 코드로 우리가 원하는 요구사항들을 구현했습니다. 컨트롤러 작성을 마치고 서버를 실행해보면 서버가 실행되지 않으며 [그림 : 센서 리파지토리 에러] 같은 오류가 발생합니다.
src/sensors/sensors.controller.ts
```
import { Controller, Post, Body, Get, Param } from '@nestjs/common';  
import { SensorsService } from './sensors.service';  
import { CreateSensorDto } from './dto/create-sensor.dto';  
  
@Controller('sensors')  
export class SensorsController {  
  constructor(private readonly sensorsService: SensorsService) {}  
  
  @Post()  
  createSensor(@Body() createSensorDto: CreateSensorDto) {  
    return this.sensorsService.createSensor(createSensorDto);  
  }  
  
  @Get()  
  getAllSensors() {  
    return this.sensorsService.getAllSensors();  
  }  
  
  @Get(':id')  
  getSensor(@Param('id') sensorId: number) {  
    return this.sensorsService.getSensor(parseInt(sensorId));
  }  
}
```

[그림 : 센서 리파지토리 에러] 같이 센서 서비스에서 센서 리파지토리를 사용할 수 없다는 에러가 발생합니다. 에러 메시지를 잘 읽어보면 센서 모듈(SensorsModule)에서 리파지토리를 사용할 수 있는지 확인하라고 알려줍니다. 또한 Nest가 제시한 해결방법을 살펴보면 센서 리파지토리가 Provider인 경우 센서 모듈의 일부인지 확인하라고 힌트를 줍니다. 여기서 나온 키워드들을 조합해보면 센서 리파지토리 클래스를 생성하고 사용하려면 센서 모듈에 등록에 등록해야한다. 라고 추측해볼 수 있습니다. 

![[sensor_di_error.png]]
[그림 : 센서 리파지토리 에러]

일단 Nest에서 제안하는 대로 센서 모듈을 살펴봅시다.

src/sensors/sensors.module.ts
```
import { Module } from '@nestjs/common';  
import { SensorsService } from './sensors.service';  
import { SensorsController } from './sensors.controller';  
  
@Module({  
  controllers: [SensorsController],  
  providers: [SensorsService],  
})  
export class SensorsModule {}

```

providers 배열에 센서 서비스가 등록 되어있는것을 확인할 수 있습니다. 센서 서비스 클래스위에 @Injectable() 데코레이터를 기억하시나요? 우리가 데이터 계층을 다룰때 사용하는 센서 리파지토리를 작성할때 까먹지말고 @Injectable() 데코레이터도 작성하라고 했습니다. @Injectable() 데코레이터는 해당 클래스를 Nest의 DI(dependency injection)컨테이너가 관리하는 주입(inject) 가능한 프로바이더(provider)로 만들어줍니다. 이 키워드들이 이해가 되지 않아도 괜찮습니다. 이후 provider를 설명할 때 같이 설명합니다. 다음 코드처럼 센서 리파지토리도 센서 서비스처럼 providers 배열에 등록하고 서버를 재실행 해보면 성공적으로 서버가 재실행 됩니다. 이제 우리는 @Injectable() 데코레이터가 붙은 클래스는 모듈의 프로바이더 배열에 등록하여 사용해야한다는 것을 배웠습니다.
그렇다면 provider는 대체 무엇일까요?

src/sensors/sensors.module.ts
```
import { Module } from '@nestjs/common';  
import { SensorsService } from './sensors.service';  
import { SensorsController } from './sensors.controller';  
import { SensorsRepository } from './sensors.repository';  
  
@Module({  
  controllers: [SensorsController],  
  /** 센서 리파지토리 등록 */  
  providers: [SensorsService, SensorsRepository],  
})  
export class SensorsModule {}
```


## Provider
프로젝트 진행을 잠시 멈추고 가벼운 마음으로 다음 내용들을 보겠습니다.
지루한 내용들을 공부하느라 조금 힘들 수 있겠지만 이러한 개념들을 이해하고 개발하는 것과 단순히 프레임워크에서 시키는대로 하는 것은 개인의 성장에 많은 차이가 납니다. 핵심 내용들이 거의 끝나가니 조금만 더 힘내주세요.

먼저 NestJS 공식문서에서 설명하는 provider에 대한 소개를 보겠습니다. 
프로바이더(provider)는 Nest의 기본 개념입니다. 많은 기본 Nest 클래스는 서비스, 리파지토리, 팩토리, 헬퍼 등등의 프로바이더로 취급될 수 있습니다. 프로바이더의 주요 아이디어는 **의존성을 주입**할 수 있다는 점입니다. 이 뜻은 객체가 서로 다양한 관계를 만들 수 있다는 것을 의미합니다. 그리고 객체의 인스턴스를 연결해주는 기능은 Nest 런타입 시스템에 위임될 수 있습니다.

![[nest_provider.png]]
[그림 : 공식 문서의 컨트롤러와 프로바이더의 의존 관계]


NestJS 공식문서는 여러분들이 다음과 같은 개념들을 이해하고 있다고 가정하고 설명하기 때문에 한번에 이해가 되지 않을 수 있습니다. 따라서 하나씩 알아보고 다시 공식문서의 설명을 보겠습니다.
1. 계층형 구조(Layered Architecture)
2. 의존성 주입(Dependency Injection)
3. 제어의 역전(IoC, Inversion of Control)


### 계층형 구조 (Layered Architecture)
계층형 구조 (Layered Architecture)는 n-tier architecture 라고도 불리며 개발하고자 하는 시스템의 성격상 여러 개의 계층을 구성할 수 있습니다. 핵심은 데이터베이스를 읽고 쓰는 등의 직접 상호작용하는 계층, 데이터를 가공하는 계층, 가공된 데이터를 사용자에게 제공하는 계층으로 시스템을 분리할 수 있습니다. 계층을 분할해 모듈간 의존성을 줄여 유지보수하기 좋은 구조를 유지합니다.

#### Presentation 계층
사용자와 직접 상호작용하는 계층입니다. 우리가 여태까지 NestJS에서 사용한 컨트롤러를 생각하면 이해가 쉽습니다. 컨트롤러는 사용자의 요청을 받아 해당 요청을 처리할 수 있는 서비스에게 전달하고 처리 결과를 반환받아 사용자에게 응답합니다.

#### Business(Domain) 계층
핵심 비즈니스 로직이 들어가는 계층입니다. 개발하고자 하는 시스템의 성격상 비즈니스 계층은 여러 계층으로 분할될 수 있지만 핵심 내용은 사용자의 요청을 전달받아 필요한 데이터를 가공하여 처리하고 결과를 반환합니다. 이 때 데이터베이스 등의 인프라에 접근해야한다면 직접 접근하지않고 Data Access 계층에 데이터를 처리하는 작업을 요청합니다. 예를 들어 사용자가 회원가입을 한다면 Presentation 계층에서 사용자의 이메일과 비밀번호를 전달받아 Business계층의 회원가입을 진행하는 서비스에 전달합니다. Business계층의 회원가입 서비스는 사용자의 이메일과 비밀번호를 저장하기위해 데이터베이스에 직접 접근하지 않고 Data Access 계층에 이메일과 비밀번호를 전달해 데이터의 저장을 요청하고 저장 결과를 반환받아 Presentation 계층에 회원가입 결과를 전달합니다.

#### Data Access 계층
Business 계층의 요청대로 데이터베이스 등의 인프라에 직접 접근하여 데이터를 읽고 수정하는 등의 작업을 하는 계층입니다.  
Data Access 계층은 데이터베이스에 접근해 작업을 처리하여 처리 결과를 전달하는 책임만 가집니다. 덕분에 Business 계층은 Data Access 계층의 데이터베이스가 어떻게 구성되어 있는지 알 필요가 없습니다. 오직 데이터의 조회, 수정 결과 등을 전달 받기 때문에 데이터베이스를 MySQL에서 PostgreSQL로 변경해도 Business 계층의 코드는 변경될 일이 없습니다.


### 의존성 주입 (Dependency Injection)
NestJS 공식문서에서 프로바이더의 핵심으로 의존성을 주입(Dependency injection)이라는 키워드를 사용했습니다. 대체 의존성 주입이 무엇일까요?

예를 들어 MMORPG 게임의 궁수(Archer) 캐릭터가 있다고 생각해봅시다. 궁수 직업은 활(bow)을 사용할 수 있습니다. 다음은 Archer 클래스에서 bow 인스턴스를 사용하는 코드입니다.
```
class Bow {  
  shot() {  
    console.log('Bow shot!');  
  }  
}  
  
class Archer {  
  bow: Bow;  
  
  constructor() {  
    this.bow = new Bow();  
  }  
  
  shot() {  
    this.bow.shot();  
  }  
}  
  
const archer = new Archer();  
archer.shot(); /** Bow shot! */
```

여기까지는 문제가 없습니다. 그러나 다음 업데이트때 궁수의 2차 전직으로 사냥꾼이 되어 단검(dagger)을 쓰도록 패치해달라는 요구사항이 생겼습니다. 단검은 발사(shot)하지 않고 찌르기(stab) 공격을 합니다. 큰일났네요, 이미 수백만줄의 코드 베이스에 궁수 캐릭터가 공격할 때 활을 발사하도록 작성해버렸습니다. 요구사항에 대응하려면 얼마나 걸릴지 상상이 안됩니다.

이전에 설명한 객체지향의 설계 원칙(SOLID)을 기억하시나요? 좋은 객체지향 설계는 구체적인 개념에 의존하지않고 추상적인 개념에 의존해야합니다. 궁수 클래스에서 new 연산자를 사용해 활을 생성하면 궁수가 활에게 의존하게 됩니다. 이처럼 직접, 구체적으로 클래스를 인스턴스화 하는 행위는 유지보수가 어려운 코드가 됩니다. 우리는 의존성 주입(dependency injection)을 통해 이 문제를 해결할 수 있습니다. 
다음과 같이 인터페이스를 사용해 활과 단검은 무기(Weapon)의 use() 메소드를 구현합니다. 또한 신규 캐릭터의 추가를 생각해 Character 인터페이스도 만들어 두겠습니다.
```
interface Weapon {  
  use(): void;  
}  
  
interface Character {  
  attack(): void;  
}  
  
class Bow implements Weapon {  
  use() {  
    console.log('Bow shot!');  
  }  
}  
  
class Dagger implements Weapon {  
  use() {  
    console.log('Dagger stab!');  
  }  
}  
  
class Archer implements Character {  
  weapon: Weapon;  

  /** 무기를 외부에서 '주입' 해준다. */
  constructor(weapon: Weapon) {  
    this.weapon = weapon;  
  }  
  
  attack() {  
    this.weapon.use();  
  }  
}  
  
```

이제  캐릭터들은 다음과 같이 활과 단검이 어떻게 구현 되었는지 몰라도 Weapon 인터페이스에 의존하기 때문에 공격할 수 있습니다.
```
const bow = new Bow();  
const dagger = new Dagger();  
  
const archer = new Archer(bow);  
const hunter = new Archer(dagger);  
  
archer.attack(); /** Bow shot! */  
hunter.attack(); /** Dagger stab! */
```

또한 캐릭터의 직업도 Character 인터페이스에 의존하게 만들었으니 새로운 직업이 추가되어도 다음과 같은 코드를 수정할 필요가 없습니다.
```
function BossRaid(characters: Character[]) {  
  characters.forEach((character) => character.attack());  
}  
  
const partyMember: Character[] = [archer, hunter];  

BossRaid(partyMember); /** Bow shot!, Dagger stab! */
```


구체적 개념에 의존하지말고 추상적 개념에 의존하라는 말이 조금 와닿으셨나요? 그런데 만약 궁수의 2차 전직이 단검 뿐만아니라 총도 쓰게 해달라고 요구사항이 변경되면 어떻게 해야할까요? (더이상 궁수가 아니게 되어버리지만) 일단 구현하려면 수백만줄의 코드 베이스에서 다음과 같이 단검과 총을 주입해주도록 수정 해야합니다.
```
const dagger = new Dagger();  
const gun = new Gun();

const hunter = new Archer(dagger, gun);  
hunter.attack(); /** Dagger stab & Gun shot! */
```

의존성 주입을 통해 궁수가 가진 무기가 무엇인지 몰라도 공격할수 있게 되었지만, 여전히 요구사항의 변경에 많은 코드를 수정해야합니다.
요구사항의 변경이 생길때마다 무기 클래스를 정의하고 일일이 new 연산으로 무기 인스턴스를 생성해서 전달하는게 최선일까요? 프로그램 규모가 커지고 복잡해질수록 요구사항들에 대응하기가 어려워집니다. 이 경우 제어의 역전을 사용합니다.



### 제어의 역전 (IoC, Inversion of Control)
제어의 역전은 선 요약하면, "나 대신 프레임워크가 제어한다" 라고 할 수 있습니다.
제어의 역전이라는 개념을 이해하기 위해서는 우선 의존성(dependency)이라는 개념을 알아야합니다. 우리는 이전 의존성 주입 챕터에서 활과 단검을 통해 의존성이 무엇인지 배웠습니다. 이제 제어의 역전을 통해 프로그래머가 수백만줄의 코드베이스에서 일일이 무기를 생성하지 않아도 되는 기적을 보겠습니다.


다음은 제어의 역전을 이용해 궁수(총과 칼을 쏨) 캐릭터를 구현합니다. NestJS는 제어의 역전을 추상화하여 동작 확인이 어렵기 때문에 TypeDI를 사용해 예제를 구현합니다. 직접 실행은 하지 않아도 됩니다. 그냥 이런 느낌이구나~ 하고 이전 코드들과 다른점을 느끼고 넘어가면 됩니다. 만약 실행해보고 싶다면 TypeDI 공식 문서를 참조해 설치후 실행해봐도 괜찮습니다.

```
import 'reflect-metadata';  
import { Container, Service } from 'typedi';  
  
interface Weapon {  
  use(): void;  
}  
  
interface Character {  
  attack(): void;  
}  
  
@Service()  
class Bow implements Weapon {  
  use() {  
    console.log('Bow shot!');  
  }  
}  
  
@Service()  
class Dagger implements Weapon {  
  use() {  
    console.log('Dagger stab!');  
  }  
}  
  
/** 요구사항이 변경되어 추가된 무기 */  
@Service()  
class Gun implements Weapon {  
  use() {  
    console.log('Gun shot!');  
  }  
}  
  
@Service()  
class Archer implements Character {  
  constructor(private readonly weapon: Bow) {}  
  
  attack() {  
    this.weapon.use();  
  }  
}  
  
  
@Service()  
class Hunter implements Character {  
  /** 요구사항이 변경되면 생성자에서 수정, 추가하면 된다. */  
  constructor(private readonly dagger: Dagger, private readonly gun: Gun) {}  
  
  attack() {  
    this.dagger.use();  
    this.gun.use();  
  }  
}  

const archer = Container.get(Archer);  
const hunter = Container.get(Hunter);  
  
archer.attack(); /** Bow shot! */  
hunter.attack(); /** Dagger stab & Gun shot! */
```

코드 어디를 봐도 new 연산자가 없습니다. 덕분에 여러 무기를 사용하도록 요구사항이 변경되어도 수백만 라인의 코드베이스에서 무기를 생성하는 코드를 추가하지 않고 사냥꾼 클래스의 생성자 파라미터에 무기만 등록해주면 됩니다. 이제 컨테이너(Container)에서 궁수와 사냥꾼 인스턴스를 가져와 공격 메소드를 호출하면 우리가 기대한대로 캐릭터가 주입 받은 무기로 공격합니다. 어떻게 이런 일이 가능할까요?

@Service() 데코레이터가 붙은 클래스는 TypeDI의 의존성  관리 시스템에 의해 **제어**됩니다. 이처럼 제어의 역전 (IoC, Inversion of Control)이라는 용어처럼 프로그래머가 직접 new 연산자로 인스턴스를 주입하는것이 아닌 **프레임 워크가** 의존 관계에 필요한 클래스를 생성해 주입해 줍니다.

이제 센서 리파지토리를 생성하고 사용하려 했을때 발생한 문제를 추측할 수 있습니다. 센서 서비스에서 리파지토리를 사용하기 위해 Nest의 DI 컨테이너에서 센서 리파지토리가 주입되기를 원했지만, DI 컨테이너가 의존성을 관리하는 프로바이더(provider) 목록에 센서 리파지토리가 등록되지 않아서 생긴 문제입니다. Nest의 DI 시스템은 @Module 데코레이터에 등록한 providers 배열에서 필요한 클래스를 찾아 주입해줍니다. Nest의 세계에서 **제어**당할 클래스는 프로바이더 목록에 등록해야함을 잊지마세요!


### 모듈(Module) 

Nest 애플리케이션은 [그림 : Nest 공식 문서 모듈 설명] 처럼 루트 모듈(application module)에서 여러 모듈들을 레고 블럭처럼 조립하여 애플리케이션을 구성합니다. 

![[nest_module.png]]
[그림 : Nest 공식 문서 모듈 설명]

Nest CLI를 사용하여 Sensors 모듈을 생성했다면 다음과 같이 app.module.ts 파일의 imports 배열에 자동으로 센서 모듈이 등록되어 있는 것을 확인할 수 있습니다. 만약 CLI를 사용하지 않고 클래스를 생성했다면 모듈 파일에 등록해 주는 작업이 필요합니다.

src/app.module.ts 
```
import { Module } from '@nestjs/common';  
import { AppController } from './app.controller';  
import { AppService } from './app.service';  
import { SensorsModule } from './sensors/sensors.module';  
  
@Module({  
  imports: [SensorsModule],  
  controllers: [AppController],  
  providers: [AppService],  
})  
export class AppModule {}
```

@Module() 데코레이터는 다음과 같은 프로퍼티들을 가질 수 있습니다.
- controllers : 현재 모듈에서 사용할 컨트롤러 컨트롤러를 등록합니다.
- providers : 현재 모듈에서 주입받아 사용, 공유할 프로바이더를 등록합니다.
- imports: 현재 모듈에서 사용할 다른 모듈을 등록합니다.
- exports : 현재 모듈에 등록된 프로바이더들을 다른 모듈에서 import 하여 사용할 수 있도록 공개합니다. 예를 들어 센서 모듈에서 새로운 프로바이더를 생성하고 다른 모듈의 프로바이더에서 사용하려면 providers 배열에 새로운 프로바이더를 등록 후, exports 배열에 새로운 프로바이더를 등록합니다. 더 자세한 사항은 공식 문서의 프로바이더 항목을 참조하세요.



이렇게 등록한 루트 모듈(AppModule)은 다음과 같이 main.ts 파일에서 애플리케이션을 생성할 때 사용됩니다.

src/main.ts
```
const app = await NestFactory.create(AppModule);  
```


다음과 같이 프로바이더에 등록한 SensorsRepository가 주입되었는지 확인하는 코드를 작성합니다.

src/main.ts
```
import { NestFactory } from '@nestjs/core';  
import { AppModule } from './app.module';  
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';  
import { SensorsRepository } from './sensors/sensors.repository';  
  
async function bootstrap() {  
  const app = await NestFactory.create(AppModule);  
  
  /** 프로바이더가 주입 되었는지 확인 */  
  const injectedRepository = app.get<SensorsRepository>(SensorsRepository);  
  console.log('Injected: ', injectedRepository);  
  
  const swaggerConfig = new DocumentBuilder()  
    .setTitle('IoT Monitoring')  
    .setDescription('Temperatures monitoring API')  
    .setVersion('1.0')  
    .addTag('sensors')  
    .build();  
  
  const document = SwaggerModule.createDocument(app, swaggerConfig);  
  
  /** 스웨거 문서에 접근하기 위한 경로를 설정합니다. */  
  SwaggerModule.setup('api-docs', app, document);  
  
  await app.listen(3000);  
}  
  
bootstrap();
```

다음과 같이 등록한 프로바이더가 주입된 것을 확인할 수 있습니다.
![[injected.png]]

이제 스웨거로 우리가 생성한 임시 데이터베이스가 작동하는지 테스트 해봅니다. 다음과 같이 /sensors POST 요청으로 데이터의 생성 결과를 확인할 수 있습니다.

![[sensor_create_1.png]]
[그림 : 센서 생성 요청]


![[sensor_create_2.png]]
[그림 : 센서 생성 응답]

센서 생성 이후 /sensors GET 요청으로 임시 데이터베이스에 저장된 센서를 조회할 수 있습니다.

![[sensors_get.png]]

[그림 : 센서 목록 요청]