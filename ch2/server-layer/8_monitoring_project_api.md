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
  public readonly type: string;  
  
  @ApiProperty()  
  @IsNumber()  
  public readonly serialNumber: number;  
}
```

@ApiProperty() 데코레이터를 지정해주지 않은 프로퍼티는 스웨거에서 확인이 불가능합니다. 따라서 다른 개발자와 협업시 해당 API에서 사용하는 DTO가 어떤 프로퍼티들로 구성되어있는지 알려야한다면 @ApiProperty() 데코레이터를 사용해줍니다.
@IsString(), @IsNumber(), @IsEnum() 등의 데코레이터는 이름에서 유추할 수 있듯이 API 요청시 해당 프로퍼티가 데코레이터에 명시한 타입인지 검사합니다. 만약 @IsNumber() 데코레이터로 명시한 속성에 문자열 데이터가 들어온다면 해당 요청을 거부합니다. 이렇게 데코레이터를 통해 실수를 근원에 방지할 수 있으니 적극적으로 활용해줍니다.

센서 생성을 위한 DTO를 정의했으니 센서를 생성하고 데이터베이스에 저장하는 코드를 작성할겁니다. 그러나 우리는 아직 데이터베이스를 적용하는 방법을 모르기때문에 임시 데이터베이스 역할을 해줄 코드가 필요합니다. 가장 간단한 방법으로 클래스의 멤버 변수로 배열을 하나 만들어서 센서가 생성될 때마다 배열에 push하는 방법으로 진행하겠습니다. 우선 데이터베이스에 저장할 센서 엔티티를 작성합니다. CLI로 센서 리소스를 생성했다면 src/sensors/entities 디렉토리 내부에 sensor.entity.ts 파일이 생성됩니다. (...엔티티 설명) 앞으로 데이터베이스와 관련한 작업시 엔티티 객체를 사용하게됩니다. 다음과 같이 센서 엔티티를 작성합니다.

src/sensors/entities/sensor.entity.ts
```
export class Sensor {  
  id: number;  
  name: string;  
  type: string;  
  serialNumber: number;  
  createdAt: Date;  
}
```


센서 서비스 클래스에 임시 데이터베이스 변수를 만들 수도 있지만, 센서 서비스 클래스가 오직 비즈니스 로직의 책임을 지게 하기 위해 데이터를 저장하는 클래스를 새로 생성하겠습니다. 이번에는 CLI를 사용하지 않고 직접 sensors.repository.ts 파일을 생성 하겠습니다. 파일의 경로는 src/sensors/sensors.repository.ts 입니다.

다음과 같이 데이터베이스 역할을 할 센서 리파지토리 클래스를 작성합니다. 앞서 설명한 것처럼 데이터베이스 연동법을 아직 모르기때문에 메모리(변수) 데이터베이스 역할을 할 sensors배열을 생성합니다. 데이터 생성 요청마다 자동으로 id를 생성하고 시간까지 저장하는 sensorEntity 객체를 배열에 push하는 saveSensor 메소드, 데이터베이스에서 센서의 id로 조회하는 findOneById 메소드, 데이터베이스의 모든 센서들을 반환하는 findAll 메소드를 작성합니다. 그리고 @Injectable() 데코레이터를 빼먹지 않고 반드시 작성해주세요! @Injectable() 데코레이터는 잠시후 Provider에서 설명합니다.

src/sensors/sensors.repository.ts
```
import { CreateSensorDto } from './dto/create-sensor.dto';  
import { Injectable } from '@nestjs/common';  
import { Sensor } from './entities/sensor.entity';  
  
@Injectable()  
export class SensorsRepository {  
  /** 임시 데이터베이스 역할 배열 */  
  private sensors: Sensor[] = [  
    {  
      id: 0,  
      name: 'first',  
      type: 'temperature',  
      serialNumber: 1111,  
      createdAt: new Date(),  
    },  
  ];  
  
  save(createSensorDto: CreateSensorDto): Sensor {  
    /** 임시 데이터베이스에 저장할 센서 객체 */  
    const sensorEntity = {  
      id: this.sensors.length,  
      name: createSensorDto.name,  
      type: createSensorDto.type,  
      serialNumber: createSensorDto.serialNumber,  
      createdAt: new Date(),  
    };  
  
    this.sensors.push(sensorEntity);  
    return sensorEntity;  
  }  
  
  findOneById(sensorId: number): Sensor {  
    return this.sensors[sensorId];  
  }  
  
  findAll(): Sensor[] {  
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
import { Sensor } from './entities/sensor.entity';  
  
@Injectable()  
export class SensorsService {  
  constructor(private readonly sensorsRepository: SensorsRepository) {}  
  
  createSensor(createSensorDto: CreateSensorDto): Sensor {  
    const saveResult = this.sensorsRepository.save(createSensorDto);  
    console.log('Created: ', saveResult);  
  
    return saveResult;  
  }  
  
  getSensor(sensorId: number): Sensor {  
    const foundResult = this.sensorsRepository.findOneById(sensorId);  
    if (!foundResult) {  
      throw new BadRequestException('sensor not found!');  
    }  
  
    return foundResult;  
  }  
  
  getAllSensors(): Sensor[] {  
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

[그림 : 센서 목록 응답]


온도 센서를 임시 데이터베이스에 생성했으니 이제 온도 데이터를 조회하는 API를 생성합니다. 이전과 마찬가지로 아직 데이터베이스 연동하는 방법을 배우지 않았으니 임시 데이터베이스 역할을 할 변수를 사용하겠습니다. CLI에서 다음 명령어로 센서 디렉토리 내부에 온도 모듈과 서비스를 생성합니다.
```
nest g mo sensors/temperatures
```

```
nest g s sensors/temperatures
```

CLI로 모듈을 생성하면 Nest는 해당 모듈을 사용할 수 있도록 자동으로 구성요소들을 등록해줍니다. CLI를 통해 모듈과 서비스를 생성했다면 temperatures.module.ts 는 다음과 같이 작성됩니다. providers 배열에 온도 서비스가 자동으로 등록된 것을 확인할 수 있습니다.

src/sensors/temperatures.module.ts
```
import { Module } from '@nestjs/common';  
import { TemperaturesService } from './temperatures.service';  
  
@Module({  
  providers: [TemperaturesService]  
})  
export class TemperaturesModule {}
```

[그림 : Nest 모니터링 앱 구조_2] 는 온도 모듈 생성 이후 디렉토리 구조입니다.



![[tree_2.png]]
[그림 : Nest 모니터링 앱 구조_2]


이전에 센서 리파지토리를 생성한 것처럼 이번에도 온도를 저장하는 책임을 가진 온도 리파지토리를 생성하겠습니다. 우선 데이터베이스에 저장될 온도 엔티티를 작성합니다. 다음과 같이 src/sensors/temperatures/entities 디렉토리를 생성하고 내부에 temperature.entity.ts 파일을 생성합니다.

src/sensors/temperatures/temperature.entity.ts
```
export class Temperature {  
  id: number;  
  sensorId: number;  
  temperature: number;  
  createdAt: Date;  
}
```

온도 엔티티 생성 이후 src/sensors/temperatures 디렉토리 내부에 temperatures.repository.ts 파일을 생성하고 다음과 같이 온도 리파지토리 클래스를 작성합니다. 이전과 마찬가지로 꼭 @Injectable() 데코레이터를 작성해주세요!

src/sensors/temperatures/temperatures.repository.ts
```
import { Injectable } from '@nestjs/common';  
import { Temperature } from './entities/temperature.entity';  
  
@Injectable()  
export class TemperaturesRepository {  
  /** 임시 데이터베이스 역할 배열 */  
  private temperatures: Temperature[] = [  
    { id: 0, sensorId: 0, temperature: 21.5, createdAt: new Date('2022/01/01') },  
    { id: 1, sensorId: 0, temperature: 22, createdAt: new Date('2022/01/02') },  
    { id: 2, sensorId: 0, temperature: 23, createdAt: new Date('2022/01/03') },  
    { id: 3, sensorId: 0, temperature: 24, createdAt: new Date('2022/01/04') },  
  ];  
  
  findBySensorId(sensorId: number): Temperature[] {  
    return this.temperatures.filter((column) => column.sensorId === sensorId);  
  }  
}
```

센서의 id로 데이터베이스에 저장되어 있는 온도 데이터들을 가져오는 기능을 추가했습니다. findBySensorId 메서드는 센서의 id를 받아 해당 센서가 저장한 모든 온도 데이터들을 조회하는 기능을 가집니다. 이후 데이터베이스를 적용해도 매개변수 타입과 리턴 타입이 변하지 않으므로 서비스 계층의 코드를 수정할 일이 없습니다.



다음과 같이 온도 서비스에 센서 id로 조회한 데이터들 중 가장 최근 데이터를  찾는 기능을 추가합니다. 지금은 데이터베이스를 적용하지 않기 때문에 간단한 helper 메서드를 만들어 요구사항을 구현합니다. 데이터베이스 적용 이후 임시 메서드는 제거하고 해당 책임은 데이터베이스에 위임합니다.

src/sensors/temperatures/temperatures.service.ts
```
import { BadRequestException, Injectable } from '@nestjs/common';  
import { TemperaturesRepository } from './temperatures.repository';  
import { Temperature } from './entities/temperature.entity';  
  
@Injectable()  
export class TemperaturesService {  
  constructor(  
    private readonly temperaturesRepository: TemperaturesRepository,  
  ) {}  
  
  getLatestTemperature(sensorId: number): Temperature {  
    const temperatures = this.temperaturesRepository.findBySensorId(sensorId);  
    if (this.isEmpty(temperatures)) {  
      throw new BadRequestException('Temperatures not found!');  
    }  
  
    return temperatures.reduce(this.getLatest);  
  }  
  
  private isEmpty(temperatures: Temperature[]): boolean {  
    return temperatures.length === 0;  
  }  
  
  /** 데이터베이스 사용전 임시 helper 메서드 */  
  private getLatest(previous: Temperature, current: Temperature): Temperature {  
    return previous.createdAt > current.createdAt ? previous : current;  
  }  
}
```


온도 데이터를 조회하는 서비스를 작성했으니 다음과 같이 CLI로 온도 컨트롤러를 생성합니다.

```
nest g co sensors/temperatures
```


다음과 같이 온도 모듈 클래스에 자동으로 등록된 컨트롤러를 확인하고 이전에 생성한 온도 리파지토리도 프로바이더에 등록합니다.

src/sensors/temperatures/temperatures.module.ts
```
import { Module } from '@nestjs/common';  
import { TemperaturesService } from './temperatures.service';  
import { TemperaturesController } from './temperatures.controller';  
import { TemperaturesRepository } from './temperatures.repository';  
  
@Module({  
  controllers: [TemperaturesController],  
  providers: [TemperaturesService, TemperaturesRepository],  
})  
export class TemperaturesModule {}
```


온도 컨트롤러를 다음과 같이 작성합니다.

src/sensors/temperatures/temperatures.controller.ts
```
import { Controller, Get, Param } from '@nestjs/common';  
import { TemperaturesService } from './temperatures.service';  
  
/** 센서의 id를 식별하기 위한 파라미터 */  
@Controller('sensors/:id/temperatures')  
export class TemperaturesController {  
  constructor(private readonly temperaturesService: TemperaturesService) {}  
  
  @Get('latest')  
  getLatestTemperature(  
    @Param('id') sensorId: string /** 컨트롤러의 id 파라미터를 가져온다 */,  
  ) {  
    return this.temperaturesService.getLatestTemperature(parseInt(sensorId));  
  }  
}
```

[그림: 센서 최신 온도 조회] 과 같이 0번 센서가 저장한 최신 온도 데이터를 조회합니다. 조회 결과로 [그림: 센서 최신 온도 조회 결과]와 같이 온도 리파지토리 클래스에 임시로 넣은 데이터들 중 최신 데이터를 확인할 수 있습니다.

![[sensors_temperatures_get.png]]
[그림: 센서 온도 조회]


![[sensors_temperatures_get_result.png]]
[그림: 센서 온도 조회 결과]



### 인프라 관리 (Infrastructure)
잠시 프로젝트 진행을 멈추고 인프라 환경에 대하여 한번 생각해 볼 필요가 있습니다. 만약 개발(development) 환경과 실제 고객의 정보를 저장하는 배포(production)환경 모두 같은 데이터베이스를 사용하면 어떻게 될까요? 개발 환경에서는 데이터베이스의 구조가 변경될 일이 자주 발생합니다. 이때 개발자가 실수로 개발에 사용되는 데이터베이스가 아닌, 실제 고객의 정보가 담긴 배포 환경의 데이터베이스에 접근해 고객의 데이터를 전부 날려버리는 일이 발생할 수 있습니다. 이러한 일을 사전에 차단하기 위해 개발 환경의 데이터베이스와 운영 환경의 데이터베이스를 분리해야 합니다. 이러한 환경에 따른 분리 방법으로 Nest에서 환경변수를 관리하는 모듈을 제공합니다.


#### 환경변수 다루기 (Configuration)
자바스크립트 생태계에서 환경변수를 다루는 방법으로 dotenv (.env)가 주로 사용됩니다. 예를 들어 개발 환경과 배포 환경의 데이터베이스 접속 URL을 다르게 하고 싶다면 여러 변수들을 정의해둔 .env 파일을  NODE_ENV 환경변수에 따라 다르게 불러옵니다.

다음과 같이 dotenv 패키지를 설치하고 2개의 .env 파일을 프로젝트 최상단에 작성합니다.
```
npm i dotenv
```

#### ..ENV 파일 Git 주의사항 추가 예정
.env.development
```
DATABASE_HOST=localhost
DATABASE_PORT=5432
```

.env.production
```
DATABASE_HOST=my-production-database-host
DATABASE_PORT=5432
```

package.json
```
...
"start:dev": "NODE_ENV=development nest start --watch",
"build": "nest build",
"start:prod": "NODE_ENV=production node dist/main",
...
```

만약 Windows 운영체제라면 다음과 같이 cross-env 패키지를 설치해서 환경변수를 주입합니다. cross-env는 운영체제에 관계없이 환경변수 주입을 도와주는 도구입니다.
```
npm i -D cross-env
```

package.json
```
...
"start:dev": "cross-env NODE_ENV=development nest start --watch",
"build": "nest build",
"start:prod": "cross-env NODE_ENV=production node dist/main",
...
```

다음과 같이 main.ts에서 env 파일을 불러옵니다. 파일을 불러오는 기능들은 해당 함수를 호출하는 파일의 경로를 기준으로 파일을 탐색하는 걸 잊지 마세요. 예를 들어 프로젝트를 빌드 하게 되면 main.ts 파일의 경로는 src/main.ts가 아니라 dist/main.ts가 됩니다. main.ts 기준으로 env 파일은 한 단계 상위 디렉토리에 존재하기 때문에 파일을 불러올 때 올바른 경로를 가리켜야 합니다.

src/main.ts
```
...

import * as path from 'path';  
import * as dotenv from 'dotenv';  
  
/** config() 함수를 실행하는 파일 위치 기준  
 *  실제 env 파일의 경로를 지정합니다  */  
dotenv.config({  
  path: path.join(__dirname, '../', `.env.${process.env.NODE_ENV}`),  
});  
  
const DATABASE_HOST = process.env.DATABASE_HOST;  
const DATABASE_PORT = +process.env.DATABASE_PORT;  
// -> parseInt(process.env.DATABASE_PORT)
console.log(`${DATABASE_HOST}:${DATABASE_PORT}`);

...
```


#### 환경변수 모듈 (Config Module)
이전까지 환경변수를 다루는 방법을 알아봤습니다. Nest에서 환경변수를 다룰때 직접 dotenv를 사용하기보다 Nest에서 제공하는 ConfigModule을 사용합니다. 다음과 같이 @nestjs/config 패키지를 설치합니다.

```
npm i @nestjs/config
```

패키지 설치 이후 다음과 같이 ConfigModule에서 env 파일에 정의된 환경변수를 불러옵니다.
src/app.module.ts
```
import { Module } from '@nestjs/common';  
import { AppController } from './app.controller';  
import { AppService } from './app.service';  
import { SensorsModule } from './sensors/sensors.module';  
import { ConfigModule } from '@nestjs/config';  
import * as path from 'path';  
  
@Module({  
  imports: [  
    ConfigModule.forRoot({  
      envFilePath: path.join(__dirname, '../', `.env.${process.env.NODE_ENV}`),  
      isGlobal: true, // Config 모듈을 전역으로 등록합니다.  
    }),  
    SensorsModule,  
  ],  
  controllers: [AppController],  
  providers: [AppService],  
})  
export class AppModule {}
```
ConfigModule을 등록할 때 isGlobal 옵션을 true로 설정하면 등록한 모듈을 전역에서 사용합니다. isGlobal의 기본값은 false입니다. 필요한 모듈마다 별도로 ConfigModule을 import할 수도 있습니다. ConfigModule로 env 파일을 불러오고 이 값을 가져오는 ConfigService 프로바이더를 주입해서 환경변수를 사용합니다. 


이전에 dotenv 관련 코드를 제거하고 main.ts를 다음과 같이 작성 후, env 파일이 로딩 되었는지 확인합니다.
src/main.ts
```
import { NestFactory } from '@nestjs/core';  
import { AppModule } from './app.module';  
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';  
import { ConfigService } from '@nestjs/config';  
  
async function bootstrap() {  
  const app = await NestFactory.create(AppModule);  
  
  /** 환경변수 확인 */  
  const configService = app.get<ConfigService>(ConfigService);  
  const DATABASE_HOST = configService.get<string>('DATABASE_HOST');  
  const DATABASE_PORT = +configService.get<string>('DATABASE_PORT');  
  console.log(`${DATABASE_HOST}:${DATABASE_PORT}`);  
  
  const swaggerConfig = new DocumentBuilder()  
    .setTitle('IoT Monitoring')  
    .setDescription('Temperatures monitoring API')  
    .setVersion('1.0')  
    .addTag('sensors')  
    .build();  
  
  const document = SwaggerModule.createDocument(app, swaggerConfig);  
  SwaggerModule.setup('api-docs', app, document);  
  
  await app.listen(3000);  
}  
  
bootstrap();
```

[그림: 환경변수 주입 결과]


### 데이터베이스 (Database)
지금까지 우리는 데이터베이스 없이 임시로 메모리(변수)에 데이터를 다뤘습니다. 이제 데이터베이스를 다룰 시간입니다. 핵심 비즈니스 로직을 담당하는 계층과 데이터베이스에 접근하는 계층을 나누었기 때문에 임시 데이터베이스를 실제 데이터베이스로 바꾸는 인프라에 관한 코드만 추가하면 됩니다. 그렇다면 실제 데이터베이스에 어떻게 접근해야 할까요?


#### ORM (Object Relational Mapping)
ORM이란 데이터베이스의 관계를 객체로 바꾸어 개발자가 객체지향적인 코드를 더 쉽게 작성하게 도와주는 도구입니다. 객체지향 프로그래밍 언어는 클래스를 사용하고, 데이터베이스는 테이블을 사용하기 때문에, 객체 모델과 관계 모델 간의 불일치가 발생합니다. ORM은 이 문제를 해결하고 개발자가 비즈니스 로직에 더 집중할 수 있게 도와줍니다. 개발자는 ORM에서 제공하는 인터페이스를 통해 SQL 쿼리 작성만이 아닌 메서드를 호출하는 방식으로 데이터베이스를 다룰 수 있습니다.
이 책에서는 PostgreSQL 데이터베이스와 TypeORM을 사용합니다.


#### 개발환경 데이터베이스 설정
PC에 직접 데이터베이스를 설치해도 되지만, 간편한 설정과 격리된 환경을 위해 도커를 사용해 데이터베이스를 설치하겠습니다. 도커 설치는 챕터 1장이나 공식 문서를 참조하세요.

다음과 같이 터미널에서 도커로 PostgreSQL을 실행합니다.
```
docker run -p 5432:5432 --name postgres -e POSTGRES_PASSWORD=mypassword -d postgres
```

데이터베이스가 실행되고 있는지 확인합니다.
```
docker ps
```

postgres 컨테이너 생성 이후 컨테이너 내부로 들어가서 데이터베이스를 생성해야 합니다. 다음 명령어로 컨테이너 내부로 들어갑니다.

```
docker exec -it postgres /bin/bash
```

다음 명령어로 데이터베이스를 생성합니다 데이터베이스의 이름은 .env.development 파일과 동일한 example로 하겠습니다. 비밀번호는 도커 컨테이너를 생성할 때 -e 옵션의 환경변수로 넘겨준 값입니다. 예제의 경우 기본 사용자인 postgres와 비밀번호로 mypassword를 사용했습니다. 이 값들은 이후 사용할 .env.development 파일과 동일해야 합니다.
```
root@618d145602a6:/# psql -U postgres

postgres=# CREATE DATABASE example ENCODING 'UTF-8';
```


#### TypeORM 설정
도커로 데이터베이스 설정을 완료 했다면, 이제 TypeORM을 사용해 백엔드 애플리케이션과 데이터베이스를 연결할 차례입니다.

다음과 같이 터미널에서 패키지들을 설치합니다.
```
npm i pg typeorm @nestjs/typeorm
```

### ...버전 경고 문구 추가 예정

다음과 같이 TypeORM 모듈을 등록합니다. 
src/app.module.ts
```
... 
import { TypeOrmModule } from '@nestjs/typeorm';  
  
@Module({  
  imports: [  
	  ...
    TypeOrmModule.forRoot({  
      type: 'postgres',  
      host: 'localhost',  
      port: 5432,  
      username: 'postgres',  
      password: 'mypassword',  
      database: 'example',  
      entities: [],  
      autoLoadEntities: true,  
      synchronize: true,  
    }),  
  ],  
  ...
})  
export class AppModule {}
```

### ...Synchronize 경고 추가 예정

TypeORM 모듈을 등록하는 과정에서 데이터베이스에 관한 민감한 정보가 포함되어 있습니다. 만약 관리하는 프로젝트가 오픈소스 프로젝트 등 공개되어 있다면 민감한 정보는 코드에 포함되지 않아야 합니다. 다음과 같이 ConfigModule에서 불러온 env 파일의 환경변수들을 사용합니다. 비 동기적으로 ConfigModule이 env 파일을 불러오기 때문에 TypeORM.forRootAsync 메서드를 사용합니다. 이전에 사용한 TypeORM.forRoot 메서드을 사용하는 것과 조금 다른 모습을 볼 수 있습니다.

src/app.module.ts
```
import { Module } from '@nestjs/common';  
import { AppController } from './app.controller';  
import { AppService } from './app.service';  
import { SensorsModule } from './sensors/sensors.module';  
import { ConfigModule, ConfigService } from '@nestjs/config';  
import * as path from 'path';  
import { TypeOrmModule } from '@nestjs/typeorm';  
  
@Module({  
  imports: [  
    ConfigModule.forRoot({  
      envFilePath: path.join(__dirname, '../', `.env.${process.env.NODE_ENV}`),  
      isGlobal: true
    }),  
    TypeOrmModule.forRootAsync({  
      imports: [ConfigModule],  
      inject: [ConfigService],  
      useFactory: (configService: ConfigService) => {  
        return {  
          type: 'postgres',  
          host: configService.get<string>('DATABASE_HOST'),  
          port: +configService.get<string>('DATABASE_PORT'),  
          username: configService.get<string>('DATABASE_USERNAME'),  
          password: configService.get<string>('DATABASE_PASSWORD'),  
          database: configService.get<string>('DATABASE_NAME'),  
          entities: [],  
          autoLoadEntities: true,  
          synchronize:  
            configService.get<string>('DATABASE_SYNCHRONIZE') === 'true'
        };  
      },  
    }),  
    SensorsModule,  
  ],  
  controllers: [AppController],  
  providers: [AppService],  
})  
export class AppModule {}
```

우선 TypeORM.forRootAsync 메서드 안에서 ConfigModule을 등록해야 합니다. 모듈 등록 이후 사용할 프로바이더를 주입(Inject)을 해줘야 합니다. 우리는 ConfigService를 사용해야 하니 inject 배열에 ConfigService를 등록합니다. 이제 주입받은 환경변수를 통해 팩토리 메서드로 TypeORM이 사용할 설정값들을 생성합니다. 

#### .. auto load entity 설명 추가

forRootAsync 메서드의 옵션을 작성했으니 이제 옵션에 주입될 환경변수를 추가해야 합니다. 다음과 같이 env 파일들을 수정합니다.

.env.development
```
DATABASE_HOST=localhost  
DATABASE_PORT=5432  
DATABASE_USERNAME=postgres  
DATABASE_PASSWORD=mypassword  
DATABASE_NAME=example  
DATABASE_SYNCHRONIZE=true
```

.env.production
```
DATABASE_HOST=my-production-database-host  
DATABASE_PORT=5432  
DATABASE_USERNAME=postgres  
DATABASE_PASSWORD=mypassword  
DATABASE_NAME=example  
DATABASE_SYNCHRONIZE=false
```


#### 온도 엔티티 작성
이전에 우리가 사용한 센서 엔티티 코드를 봅시다. 
src/sensors/entities/sensor.entity.ts
```
export class Sensor {  
  id: number;  
  name: string;  
  type: string;  
  serialNumber: number;  
  createdAt: Date;  
}
```

임시로 사용하기엔 손색이 없지만 센서의 타입이 string 타입인 점이 조금 아쉽습니다. string은 실수하기 좋은 타입이기 때문에 센서 타입에 관한 enum을 생성해 타입을 지정해 주겠습니다.

다음과 같이 새로운 파일을 생성하고 enum을 작성합니다.
src/sensors/entities/sensor.type.ts
```
export enum SensorType {  
  TEMPERATURE = 'temperature',  
}
```

센서의 타입을 재정의 했으니 DTO도 바뀌어야 합니다. 다음과 같이 센서 DTO도 수정해줍니다.
src/sensors/dto/create-sensor.dto.ts
```
import { IsEnum, IsNumber, IsString } from 'class-validator';  
import { ApiProperty } from '@nestjs/swagger';  
import { SensorType } from '../entities/sensor.type';  
  
export class CreateSensorDto {  
  @ApiProperty() // Swagger에서 API 요청에 필요한 객체의 속성들을 확인하기 위한 데코레이터  
  @IsString() // Class validator에서 검증할 데코레이터  
  public readonly name: string;  
  
  @ApiProperty({ enum: SensorType })  
  @IsEnum(SensorType)  
  public readonly type: SensorType;  
  
  @ApiProperty()  
  @IsNumber()  
  public readonly serialNumber: number;  
}
```

변수에 저장될 때는 객체 그대로여도 상관없었지만, 데이터베이스를 연결하게 되면 객체를 데이터베이스의 엔티티로 변환하는 작업이 필요합니다. 이는 ORM을 통해 쉽게 해결할 수 있습니다.
다음과 같이 센서 엔티티 클래스를 수정합니다. OneToMany 관계를 작성할 때 에러가 나겠지만 괜찮습니다. 아직 Temperature 엔티티를 작성하지 않아서 생긴 문제입니다. Temperature 엔티티 작성이 완료되면 에러가 사라집니다.
src/sensors/entities/sensor.entity.ts
```
import { Entity, Column, CreateDateColumn, PrimaryGeneratedColumn, OneToMany } from 'typeorm';  
import { SensorType } from './sensor.type';  
import { Temperature } from '../temperatures/entities/temperature.entity';  
  
@Entity()  
export class Sensor {  
  @PrimaryGeneratedColumn()  
  id: number;  
  
  @Column()  
  name: string;  
  
  @Column({ type: 'enum', enum: SensorType })  
  type: SensorType;  
  
  @Column({ unique: true })  
  serialNumber: number;  
  
  @OneToMany((type) => Temperature, (temperature) => temperature.sensor)  
  temperatures: Temperature[];  
  
  @CreateDateColumn()  
  createdAt: Date;  
}
```
TypeORM의 데코레이터들은 공식 문서를 통해 자세한 확인이 가능합니다.


센서 엔티티 작성을 마치면 온도 엔티티를 작성할 차례입니다. 하나의 센서가 여러개의 온도 데이터를 가지기 때문에 OneToMany, ManyToOne 관계를 설정해주고 센서의 ID 컬럼을 외래키(Foreign key)로 설정해줍니다.
다음과 같이 온도 엔티티를 작성합니다.

src/sensors/temperatures/entities/temperature.entity.ts
```
import {  
  Column,  
  CreateDateColumn,  
  Entity,  
  JoinColumn,  
  ManyToOne,  
  PrimaryGeneratedColumn,  
} from 'typeorm';  
import { Sensor } from '../../entities/sensor.entity';  
  
@Entity()  
export class Temperature {  
  @PrimaryGeneratedColumn()  
  id: number;  
  
  @Column()  
  sensorId: number;  
  
  @Column({ type: 'float' })  
  temperature: number;  
  
  @JoinColumn({ name: 'sensorId', referencedColumnName: 'id' })  
  @ManyToOne((type) => Sensor, (sensor) => sensor.temperatures)  
  sensor: Sensor;  
  
  @CreateDateColumn()  
  createdAt: Date;  
}
```
@JoinColumn 데코레이터의 옵션으로 준 name은 실제 데이터베이스 테이블의 컬럼 이름입니다. 엔티티 클래스의 프로퍼티 이름이 아니므로 주의하세요. referencedColumnName은 참조할 key의 이름입니다. 센서 엔티티를 참조하기 위해 센서 엔티티의 ID를 온도 엔티티의 sensorId 컬럼에 추가하는 것입니다.

센서와 온도 엔티티를 수정했으니 임시 데이터베이스를 다루는 리파지토리 클래스들을 수정할 차례입니다. 우선 센서 리파지토리를 다음과 같이 수정합니다.
src/sensors/sensors.repository.ts
```
import { CreateSensorDto } from './dto/create-sensor.dto';  
import { Injectable } from '@nestjs/common';  
import { Sensor } from './entities/sensor.entity';  
import { DataSource } from 'typeorm';  
  
@Injectable()  
export class SensorsRepository {  
  /** 데이터베이스를 주입 받음 */  
  constructor(private readonly dataSource: DataSource) {}  
  
  async save(createSensorDto: CreateSensorDto): Promise<Sensor> {  
    const repository = this.dataSource.getRepository(Sensor);  
  
    /** 데이터베이스에 저장할 센서 엔티티 생성 */  
    const sensor = repository.create(createSensorDto);  
  
    /** 데이터베이스에 센서 엔티티 저장 시도 */  
    return repository.save(sensor);  
  }  
  
  async findOneById(sensorId: number): Promise<Sensor> {  
    return this.dataSource.getRepository(Sensor).findOneBy({ id: sensorId });  
  }  
  
  async findAll(): Promise<Sensor[]> {  
    return this.dataSource.getRepository(Sensor).find();  
  }  
}
```


TypeORM 모듈에 데이터베이스에서 사용할 센서 엔티티를 등록해야 합니다. 다음과 같이 센서 모듈에 TypeORM 모듈을 등록합니다. 센서 엔티티를 등록(import)하고 등록한 TypeORM 모듈을 내보내고(export)있습니다. 이러면 다른 모듈에서 센서 모듈 등록 이후 센서 엔티티를 사용할 수 있습니다.
src/sensors/sensors.module.ts
```
import { Module } from '@nestjs/common';  
import { SensorsService } from './sensors.service';  
import { SensorsController } from './sensors.controller';  
import { SensorsRepository } from './sensors.repository';  
import { TypeOrmModule } from '@nestjs/typeorm';  
import { Sensor } from './entities/sensor.entity';  
  
@Module({  
  imports: [TypeOrmModule.forFeature([Sensor])],  
  controllers: [SensorsController],  
  providers: [SensorsService, SensorsRepository],  
  exports: [TypeOrmModule],  
})  
export class SensorsModule {}
```

센서 서비스 메서드들의 반환 타입을 다음과 같이 수정합니다. 데이터베이스에 비동기적으로 접근하기 때문에 async 메서드를 사용하며 Promise를 반환합니다. 데이터를 콘솔에 찍어보고 싶다면 await 키워드를 잊지 말고 추가해 주세요.  데이터베이스와 직접 관련된 코드를 서비스와 분리했기 때문에 핵심 로직을 수정하지 않아도 됩니다. 
#### .. Promise 설명 추가 예정
src/sensors/sensors.service.ts
```
import { BadRequestException, Injectable } from '@nestjs/common';  
import { CreateSensorDto } from './dto/create-sensor.dto';  
import { SensorsRepository } from './sensors.repository';  
import { Sensor } from './entities/sensor.entity';  
  
@Injectable()  
export class SensorsService {  
  constructor(private readonly sensorsRepository: SensorsRepository) {}  
  
  async createSensor(createSensorDto: CreateSensorDto): Promise<Sensor> {  
    const saveResult = await this.sensorsRepository.save(createSensorDto);  
    console.log('Created: ', saveResult);  
  
    return saveResult;  
  }  
  
  async getSensor(sensorId: number): Promise<Sensor> {  
    const foundResult = await this.sensorsRepository.findOneById(sensorId);  
    if (!foundResult) {  
      throw new BadRequestException('sensor not found!');  
    }  
  
    return foundResult;  
  }  
  
  getAllSensors(): Promise<Sensor[]> {  
    return this.sensorsRepository.findAll();  
  }  
}
```


이제 온도 리파지토리의 코드를 수정하겠습니다. 다음과 같이 온도 리파지토리를 수정합니다. 센서 리파지토리에서는 일반적인 findOneBy 메서드를 사용했지만 다음과 같이 findOneOrFail 메서드를 사용할 수도 있습니다. OrFail이 붙은 메서드는 대상을 찾지 못하면 예외를 던집니다. 상황에 맞는 메서드를 선택해 사용하면 됩니다.
src/sensors/temperatures/temperatures.repository.ts
```
import { BadRequestException, Injectable } from '@nestjs/common';  
import { Temperature } from './entities/temperature.entity';  
import { DataSource } from 'typeorm';  
  
@Injectable()  
export class TemperaturesRepository {  
  /** 데이터베이스를 주입 받음 */  
  constructor(private readonly dataSource: DataSource) {}  
  
  async findBySensorId(sensorId: number): Promise<Temperature> {  
    try {  
      return await this.dataSource  
        .getRepository(Temperature)  
        .findOneOrFail({ where: { sensorId }, order: { id: 'DESC' } });  
      // .findOneOrFail({ where: { sensorId }, order: { createdAt: 'DESC' } });  
    } catch (e) {  
      throw new BadRequestException(e.message);  
    }  
  }  
  
  async findTemperaturesByDates(  
    sensorId: number,  
    begin: Date,  
    end: Date,  
  ): Promise<Temperature[] | null> {  
    return this.dataSource  
      .getRepository(Temperature)  
      .createQueryBuilder()  
      .select()  
      .where('sensorId = :sensorId', { sensorId })  
      .andWhere('createdAt BETWEEN :begin AND :end', { begin, end })  
      .execute();  
  }  
}
```


센서 엔티티를 등록한 것과 마찬가지로 온도 엔티티도 온도 모듈에 등록합니다. 추후에 센서 모듈의 엔티티를 사용할 예정이기 때문에 센서 모듈을 등록(import)합니다.
src/sensors/temperatures/temperatures.module.ts
```
import { Module } from '@nestjs/common';  
import { TemperaturesService } from './temperatures.service';  
import { TemperaturesController } from './temperatures.controller';  
import { TemperaturesRepository } from './temperatures.repository';  
import { TypeOrmModule } from '@nestjs/typeorm';  
import { Temperature } from './entities/temperature.entity';  
import { SensorsModule } from '../sensors.module';  
  
@Module({  
  imports: [SensorsModule, TypeOrmModule.forFeature([Temperature])],  
  controllers: [TemperaturesController],  
  providers: [TemperaturesService, TemperaturesRepository],  
})  
export class TemperaturesModule {}
```

루트 모듈에 온도 모듈 등록을 잊지 마세요.
src/app.module.ts
```
...

import { TemperaturesModule } from './sensors/temperatures/temperatures.module';
  
@Module({  
  imports: [  
  
    ...
    
    SensorsModule,
    TemperaturesModule,  
  ],  
  
...

```


온도 서비스에서 이전에 작성했던 임시 데이터베이스의 helper 메서드들을 제거하고 async 메서드로 수정합니다. 온도 저장 기능은 센서가 보낸 온도를 수신할 때 제작하겠습니다.
src/sensors/temperatures/temperatures.service.ts
```
import { Injectable } from '@nestjs/common';  
import { TemperaturesRepository } from './temperatures.repository';  
import { Temperature } from './entities/temperature.entity';  
  
@Injectable()  
export class TemperaturesService {  
  constructor(private readonly temperaturesRepository: TemperaturesRepository) {}  
  
  async getLatestTemperature(sensorId: number): Promise<Temperature> {  
    return this.temperaturesRepository.findBySensorId(sensorId);  
  }  
}
```

지금까지 많은 코드를 작성했습니다. 이제 데이터베이스가 잘 동작하는지 검증해 볼 시간입니다. 다음과  같이 Swagger에서 API들을 테스트해 봅니다. 우선 데이터베이스에 센서를 생성해 보겠습니다.
![[sensor_typeorm.png]]

다음과 같이 성공적으로 센서가 생성된 것을 볼 수 있습니다.
![[sensor_res_typeorm.png]]

방금 생성한 센서를 조회해 봅니다.
![[sensor_find_typeorm.png]]

다음과 같이 데이터베이스에서 방금 생성한 센서를 확인할 수 있습니다.
![[sensor_find_res_typeorm.png]]

아직 온도 저장 기능을 추가하지 않았지만 최신 온도 조회 API도 테스트 해봅니다.
![[temperature_latest_find_typeorm.png]]

다음과 같이 리파지토리의 findOneOrFail 메서드에서 조회에 실패했을 때 우리가 던진 예외를 확인할 수 있습니다.
![[temperature_late_find_exception_typeorm.png]]


축하합니다. 드디어 데이터베이스를 연동하는 긴 여정을 끝냈습니다. 이제 센서가 보낸 MQTT 메시지에서 온도를 받아 데이터베이스에 저장하는 모듈을 만들면 백엔드 기능의 요구사항은 모두 충족하게 됩니다.


### NestJS에서 MQTT 통신하기
현재 우리 백엔드 애플리케이션은 사용자의 요청(Request)에 따른 응답(Response)을 제공합니다. 

```
npm i mqtt @nestjs/microservices
```

.env 파일들에 MQTT 브로커의 url을 추가합니다. 배포 환경 url의 경우 챕터1에서 AWS EC2 인스턴스에 MQTT 브로커를 설치했기 때문에 AWS EC2 인스턴스의 탄력적 IP 주소로 설정합니다. 만약 로컬 환경에 MQTT 브로커의 설치가 어렵다면 이전에 생성한 AWS EC2의 MQTT 브로커를 사용해 학습을 진행해도 괜찮습니다.
.env.development
```
...

MQTT_BROKER_URL=mqtt:localhost:1883  
```

.env.production
```
...

MQTT_BROKER_URL=mqtt:aws-ec2-public-ip:1883  
```

CLI에서 MQTT 통신에 사용할 모듈과 서비스를 생성합니다.
```
nest g mo mqtt
nest g s mqtt
```

다음과 같이 MQTT 클라이언트를 주입받을 토큰을 생성합니다.
src/mqtt/mqtt.constant.ts
```
export const MQTT_CLIENT = 'MQTT_CLIENT';
```

다음과 같이 MQTT 브로커에 메시지를 '전송'하는 클라이언트를 등록합니다. name에 이전에 생성한 토큰을 등록하면 이후 클래스의 프로퍼티에 직접 주입할 수 있습니다. MQTT 클라이언트 등록 이후 다른 모듈에서 MQTT 서비스를 사용하기 위해 다시 내보내줍니다(exports). MQTT 서비스를 사용하는 모듈에서 MQTT 모듈을 등록(imports)하면 MQTT 서비스를 주입받아 사용할 수 있습니다. 주의 해야할 점은 메시지의 '수신'이 아니라 '전송'하는 클라이언트입니다. 메시지의 수신 등록은 이후 main.ts에서 마이크로서비스를 연결할 때 진행합니다.
src/mqtt/mqtt.module.ts
```
import { Module } from '@nestjs/common';  
import { ClientsModule, Transport } from '@nestjs/microservices';  
import { ConfigModule, ConfigService } from '@nestjs/config';  
import { MQTT_CLIENT } from './mqtt.constant';  
import { MqttService } from './mqtt.service';  
  
@Module({  
  imports: [  
    ClientsModule.registerAsync([  
      {  
        name: MQTT_CLIENT,  
        imports: [ConfigModule],  
        inject: [ConfigService],  
        useFactory: (configService: ConfigService) => {  
          return {  
            transport: Transport.MQTT,  
            options: {  
              url: configService.get<string>('MQTT_BROKER_URL'),  
            },  
          };  
        },  
      },  
    ]),  
  ],  
  providers: [MqttService],  
  exports: [MqttService], // 다시 export 해준다
})  
export class MqttModule {}
```

MQTT 서비스에서는 메시지를 전송할 MQTT 클라이언트를 주입받아 사용합니다. 이전에 name에 등록한 토큰으로 해당 프로바이더를 주입할 수 있습니다.
src/mqtt/mqtt.service.ts
```
import { Inject, Injectable } from '@nestjs/common';  
import { MQTT_CLIENT } from './mqtt.constant';  
import { ClientProxy } from '@nestjs/microservices';  
import { Observable } from 'rxjs';  
  
@Injectable()  
export class MqttService {  
  constructor(@Inject(MQTT_CLIENT) private readonly mqttClient: ClientProxy) {}  
  
  send(topic: string, payload: unknown): Observable<unknown> {  
    return this.mqttClient.send(topic, payload);  
  }  
}
```

MQTT 메시지 전송 모듈을 만들었으니 사용해 보겠습니다. 다음과 같이 예시로 사용할 프로토콜과 패킷을 정의합니다. 패킷 클래스는 패킷의 생성과 직렬화를 담당합니다. 작성한 serialize() 메서드를 사용해 패킷을 객체에서 문자열로 직렬화합니다.
src/mqtt/packet.schema.ts
```
/** 예시 프로토콜 */  
export enum Protocol {  
  START = 0x23,  
  READ = 0xc1,  
  WRITE = 0xd1,  
  END = 0x0d,  
}  
  
interface PacketSchema {  
  start: number;  
  command: number;  
  target: number;  
  checksum: number;  
  end: number;  
}  
  
type BuildSchema = Pick<PacketSchema, 'command' | 'target'>;  
  
export class Packet {  
  private constructor(private readonly packet: PacketSchema) {}  
  
  static generateRaw(packetDto: BuildSchema): string {  
    const checksum = this.getChecksum(packetDto);  
  
    return new Packet({  
      start: Protocol.START,  
      ...packetDto,  
      checksum,  
      end: Protocol.END,  
    }).serialize();  
  }  
  
  private static getChecksum(packetDto: BuildSchema): number {  
    return Object.values(packetDto).reduce(  
      (previous, current) => previous + current,  
    );  
  }  
  
  private serialize(): string {  
    return JSON.stringify(this.packet);  
  }  
}
```

센서 모듈에서 MQTT 모듈을 사용하기 위해 다음과 같이 MQTT 모듈을 등록합니다.
src/sensors/sensors.module.ts
```
...

import { MqttModule } from '../mqtt/mqtt.module';  
  
@Module({  
  imports: [MqttModule, TypeOrmModule.forFeature([Sensor])],  
  
  ...

```

POST 요청으로 MQTT 메시지 전송을 위해 다음과 같이 DTO를 작성합니다. 예시에서는 간단하게 MQTT 토픽과 명령어와 대상 하드웨어 정도로 작성했습니다. 추후 본인의 프로젝트 진행 상황에 맞게 패킷 클래스와 DTO를 수정하면 됩니다.
src/sensors/dto/send-mqtt.dto.ts
```
import { IsNumber, IsString } from 'class-validator';  
import { ApiProperty } from '@nestjs/swagger';  
  
export class SendMqttDto {  
  @IsString()  
  @ApiProperty()  
  topic: string;  
  
  @IsNumber()  
  @ApiProperty()  
  command: number;  
  
  @IsNumber()  
  @ApiProperty()  
  target: number;  
}
```

다음과 같이 센서 컨트롤러에 MQTT 서비스를 주입받아 메시지 전송이 가능합니다.
src/sensors/sensors.controller.ts
```
...

import { MqttService } from '../mqtt/mqtt.service';  
import { Packet } from '../mqtt/packet.schema';  
import { SendMqttDto } from './dto/send-mqtt.dto';  
  
@Controller('sensors')  
export class SensorsController {  
  constructor(  
    private readonly sensorsService: SensorsService,  
    private readonly mqttService: MqttService,  
  ) {}  
  
  @Post('mqtt/message')  
  sendMessage(@Body() sendMqttDto: SendMqttDto) {  
    const { topic, command, target } = sendMqttDto;  
    const payload = Packet.generateRaw({ target, command });  
  
    return this.mqttService.send(topic, payload);  
  }  

  ...
}
```

#### .. MQTT 전송 결과 추가 예정

이번에는 MQTT 메시지를 수신하는 방법을 알아보겠습니다. 다음과 같이 main.ts에서 기존 앱에 MQTT 메시지를 수신하는 마이크로서비스를 연결합니다. 분리된 마이크로서비스를 생성하고 마이크로서비스 간의 통신을 할 수도 있지만 분량이 너무 방대해지므로 이 책에서는 다루지 않습니다. 만약 마이크로서비스에 관심이 있다면 공식 문서를 참조하세요. 이 책에서는 기존 앱에 마이크로서비스를 연결하는 하이브리드 애플리케이션 방식으로 MQTT 메시지 수신 예제를 진행합니다.
src/main.ts
```
...

import { MicroserviceOptions, Transport } from '@nestjs/microservices';  
  
async function bootstrap() {  
  const app = await NestFactory.create(AppModule);  
  const configService = app.get<ConfigService>(ConfigService);  
  
  app.connectMicroservice<MicroserviceOptions>({  
    transport: Transport.MQTT,  
    options: {  
      url: configService.get<string>('MQTT_BROKER_URL'),  
    },  
  });  
  
  ...
  
  await app.startAllMicroservices();  
  await app.listen(3000);  
}  
  
bootstrap();
```

MQTT 메시지를 수신할 마이크로서비스 연결 이후, 이전에 작성했던 온도 서비스를 수정합니다. 이전에 작성한 온도 리파지토리의 이름을 temperaturesQueryRepository로 변경하고 새로운 방식으로 리파지토리를 사용해 보겠습니다. 리파지토리 클래스를 직접 작성하지 않고 다음과 같이 주입받아 사용할 수도 있습니다.
src/sensors/temperatures/temperatures.service.ts
```
import { Injectable } from '@nestjs/common';  
import { TemperaturesRepository } from './temperatures.repository';  
import { Temperature } from './entities/temperature.entity';  
import { InjectRepository } from '@nestjs/typeorm';  
import { Repository } from 'typeorm';  
import { Sensor } from '../entities/sensor.entity';  
  
@Injectable()  
export class TemperaturesService {  
  constructor(  
    @InjectRepository(Sensor)  
    private readonly sensorRepository: Repository<Sensor>,  
    @InjectRepository(Temperature)  
    private readonly temperaturesRepository: Repository<Temperature>,  
    private readonly temperaturesQueryRepository: TemperaturesRepository,  
  ) {}  
  
  async saveTemperature(  
    serialNumber: number,  
    temperature: number,  
  ): Promise<Temperature> {  
    const sensor = await this.sensorRepository.findOneByOrFail({  
      serialNumber,  
    });  
  
    const temperatureEntity = this.temperaturesRepository.create({  
      sensor,  
      temperature,  
    });  
  
    return this.temperaturesRepository.save(temperatureEntity);  
  }  
  
  async getLatestTemperature(sensorId: number): Promise<Temperature> {  
    return this.temperaturesQueryRepository.findBySensorId(sensorId);  
  }  
}
```


온도를 저장하는 메서드를 추가했으니 MQTT 메시지를 수신하는 컨트롤러를 생성합니다. MQTT 토픽(topic)에서 센서의 시리얼 번호를 추출하여 온도를 저장합니다.
src/sensors/temperatures/temperatures-mqtt.controller.ts
```
import { Controller } from '@nestjs/common';  
import { Ctx, MessagePattern, MqttContext, Payload } from '@nestjs/microservices';  
import { TemperaturesService } from './temperatures.service';  
  
@Controller()  
export class TemperaturesMqttController {  
  constructor(private readonly temperaturesService: TemperaturesService) {}  
  
  // MQTT topic 'temperature/serial-number'  
  @MessagePattern('temperature/+')  
  async receiveTemperature(  
    @Payload() temperature: number,  
    @Ctx() context: MqttContext,  
  ) {  
    const sensorSerialNumber = parseInt(context.getTopic().split('/')[1]);  
    await this.temperaturesService.saveTemperature(  
      sensorSerialNumber,  
      temperature,  
    );  
  }  
}
```


#### .. MQTT 터미널 결과 사진 추가 예정





## 웹 클라이언트
이제 우리의 API가 실제 웹 브라우저에서 작동하는지 확인할 차례입니다. 웹 브라우저에서 작동하는 프론트엔드(frontend) 개발에 React, Vue 등을 사용할 수도 있지만 이 책의 영역을 넘어가기 때문에 프론트엔드 스택에 대해서는 다루지 않습니다. 예제에서는 간단하게 HTML과 JavaScript를 사용해 사용자 계층을 구현하겠습니다.

index.html
```
<!DOCTYPE html>  
<html lang="en">  
<head>  
  <meta charset="UTF-8">  
  <title>Temperature Example</title>  
</head>  
<body>  
  <h1>온도 모니터링 프론트엔드</h1>  
  <h2>최근 온도</h2>  
  <p id="temperature">unknown</p>  
<script src="index.js"></script>  
</body>  
</html>
```


다음과 같이 웹 브라우저에서 주기적으로 센서의 최근 온도를 요청하는 코드를 작성합니다.
index.js
```
const API_URL = 'http://localhost:3000';  
  
const temperatureText = document.getElementById('temperature');  
const fetchLatestTemperature = () => {  
  const latestTemperaturesApiUrl = `${API_URL}/sensors/8/temperatures/latest`;  
  return fetch(latestTemperaturesApiUrl)  
    .then((res) => res.json())  
    .then((res) => (temperatureText.innerText = res.temperature))  
    .catch((e) => {  
      console.log(e);  
    });  
};  
  
setInterval(fetchLatestTemperature, 2000);
```



#### .. CORS 설명

#### AWS에 백엔드 배포하기
이제 백엔드 애플리케이션을 AWS EC2에 배포할 차례입니다. 이 책에서는 간단하게 로컬 PC에서 이미지 빌드, 도커 허브(Docker hub)에 push, 배포할 EC2에 직접 접속해서 이미지를 pull하는 방식으로 진행하겠습니다. Github Actions, AWS ECR, AWS CodePipeline 등을 통해 파이프라인을 구축할 수도 있지만 책의 범위를 벗어나므로 테스트, 배포 자동화에 관심이 있다면 CI / CD 파이프라인에 대하여 따로 알아보기를 추천드립니다.

우선 도커 허브(Docker hub)의 계정을 생성해야합니다. 다음과 같이 도커 허브 계정을 생성합니다.

#### ... 도커 허브 계정 생성, 이미지 저장소 설명 추가 예정

생성한 public 이미지 저장소는 추후 백엔드 이미지가 저장될 공간입니다.
다음과 같이 프로젝트 루트 디렉토리에 도커파일(Dockerfile)을 작성합니다. src 디렉토리 내부가 아닙니다. 주의하세요.
Dockerfile
```
# Step 1 build  
  
FROM node:14 AS builder  
  
WORKDIR /app  
  
COPY . .  
  
RUN npm install  
  
RUN npm run build  
  
  
# Step 2 use build image  
FROM node:14-alpine  
  
WORKDIR /app  
  
COPY --from=builder /app ./  
  
EXPOSE 3000  
  
CMD ["npm", "run", "start:prod"]

```


도커파일 작성 이후 다음과 같이 package.json에 이미지 빌드와 배포 명령어를 작성합니다. 이전에 생성한 도커 허브 public repository에 push 명령어를 확인할 수 있습니다.  다음과 같이 tagname만 latest로 수정합니다. 또한 태그를 지정하지 않으면 자동으로 latest 태그가 적용됩니다. docker build 명령어의 "."을 잊지 말고 작성해 주세요. 도커파일 경로를 지정해야 합니다. 만약 M1 Mac을 사용 중이라면 빌드 시 --platform linux/amd64 옵션을 지정해 주어야 합니다.
package.json
```
...
"scripts": {  
	...
  "start:dev": "cross-env NODE_ENV=development nest start --watch",
  "start:prod": "cross-env NODE_ENV=production node dist/main",
  "start:docker-build": "docker build -t username/reponame:latest --platform linux/amd64 .",
  "start:docker-deploy": "docker push username/reponame",
	...
},

...
```
Windows 사용자가 아니라면 스크립트의 cross-env 옵션을 제거해도 됩니다.

스크립트 작성 이후 다음과 같이 백엔드 애플리케이션을 도커 이미지로 빌드합니다.
```
npm run start:docker-build
```

이미지 빌드가 완료되면 다음과 같이 도커 허브에 이미지를 배포합니다.
```
npm run start:docker-deploy
```


다음과 같이 도커 허브에서 push한 이미지를 확인할 수 있습니다. 이제 도커 허브에 저장된 이미지를 EC2 인스턴스에서 내려받아(pull) 실행하기만 하면 됩니다. 이 책에서는 EC2 인스턴스에 직접 접속해서 도커허브의 public 이미지를 내려받겠습니다. private repository나 AWS ECR을 이용하는 방법은 이 책에서 다루지 않습니다. 
#### ... 도커 허브 push 결과 이미지 추가 예정


### EC2에서 도커 이미지 불러오기
도커 허브에 백엔드 애플리케이션 이미지를 저장했으니 실제 애플리케이션이 작동할 서버에서 해당 이미지를 가져와(pull) 실행해야 합니다. 이전에 EC2에 도커를 설치할 때처럼 다시 EC2 인스턴스에 접속합니다.

EC2 인스턴스에 접속 후 다음과 같이 배포한 백엔드 애플리케이션을 pull 합니다.
```
docker pull username/reponame
```

다음 명령어로 이미지를 가져왔는지 확인할 수 있습니다.
```
docker images
```

이전에 도커 컴포즈로 MQTT 브로커를 실행하기 위해 생성한 docker-compose.yml 파일을 다음과 같이 수정합니다. 서비스의 이름은 backend로 지정하겠습니다. 백엔드 애플리케이션에서 mosquitto 서비스에 접근하기 위해 links 옵션으로 mosquitto 서비스를 지정합니다. links 옵션은 해당 컨테이너를 연결해 주는 옵션입니다. 더 자세한 내용은 도커 컴포즈 공식 문서를 참조하세요.
```
version: "3"  
services:  
  mosquitto:  
    image: eclipse-mosquitto  
    ports:  
      - "1883:1883"  
    volumes:  
      - ~/mosquitto:/mosquitto/  

  backend:  
    image: baram987/iot-monitoring  
    ports:  
      - "3000:3000"  
    links:  
      - mosquitto
```

도커 컴포즈 파일을 수정하고 컨테이너들을 실행합니다.
```
docker-compose -f ~/docker-compose.yml up -d
```

다음과 같이 실행 중인 컨테이너들을 확인할 수 있습니다.
```
docker ps
```



## .. 추가 작성 예정 