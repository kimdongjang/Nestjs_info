# nest_info
nestjs 정리

## Controller  
### @Req, @Res는 차후 퍼포먼스 문제로 전환할 경우를 대비하기 위해 사용하지 않는 것이 좋다.
@Request()	<-  req  
@Response(), @Res()*	<-  res  
@Next()	 <-  next  
@Session()	<-  req.session  
@Param(key?: string)	<-  req.params / req.params[key]  
@Body(key?: string)	<-  req.body / req.body[key]  
@Query(key?: string)	<-  req.query / req.query[key]  
@Headers(name?: string)	 <-  req.headers / req.headers[name]  
@Ip()	<-  req.ip  
@HttpCode(status:number)	   
  
### @Query를 사용한 예(year=2022인 데이터를 가져오기)
```js
@Controller('movies')
export class MoviesController {
  @Get('/search')
  search(@Query('year') searchingYear: number) {
    return `We are Searching made after: ${searchingYear}`;
  }

}
```
Query를 사용하면 ```/movies/search?year=2022```와 같은 형태로 접근해 year의 값을 전달할 수 있다.

  
### req 대신 @Body를 사용한 예
```js
import { Body, Controller, Delete, Get, Param, Patch, Post } from '@nestjs/common';

@Controller('movies')
export class MoviesController {
  @Post()
  create(@Body() movieData) {
    console.log(movieData); // json
    return movieData;
  }

  @Patch('/:id')
  patch(@Param('id') movieId: string, @Body() updateData) {
    return {
      updatedMovie: movieId,
      ...updateData,
    };
  }
}
```
## Provider
컨트롤러가 요청과 응답을 가공하고 처리하는 역할을 맡는다면, 사용자가 요청한 **핵심 기능**을 처리하는 역할은 프로바이더의 역할이다.  
사용자가 전달해준 데이터를 가지고 주변에 위치한 가게를 검색한다던가, 회원가입을 한다던가, 앱이 제공하고자 하는 핵심 기능을 처리하는, 즉 비즈니스 로직을 프로바이더가 수행하는데  
대표적으로 서비스(Service), 레포지토리(Repository), 팩토리(Factory), 헬퍼(Helper) 등의 여러가지 형태로 프로바이더를 구성할 수 있다.  
```js
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}
    ...

    @Delete(':id')
    remove(@Param('id') id: string) {
      return this.usersService.remove(+id);
    }
}
```
따라서 위와 같이 **의존성 주입**(Depenndency Injection, DI)을 통해 객체를 생성하고 사용할 때 관심사를 분리해 가독성과 재사용성이 높은 코드를 작성할 수 있게 한다.

### 사용방법
```js
import { Injectable } from '@nestjs/common';

@Injectable()
export class UsersService {
    ...

  remove(id: number) {
    return `This action removes a #${id} user`;
  }
}
```
Service를 구성했다면 ```@Injectable``` 데코레이터를 사용해 다른 nest의 컴포넌트에서도 위 서비스를 주입해 사용할 수 있도록 지정할 수 있다.


### 모듈에 프로바이더 등록
```js
@Module({
    ...
  providers: [UsersService]
})
export class UsersModule {}
```
또한 프로바이더를 등록했다는 것을 모듈에서도 명시를 해주어야 한다.


## 모듈
여러 컴포넌트를 조합해 좀 더 큰 작업을 수행하는 단위를 말한다. 유저의 정보를 관리하는 유저 모듈, 채팅기능을 담당하는 채팅 모듈 등 모듈들이 모여 채팅 서비스를 구현할 수 있게 된다.
Nest에서는 App Module이라고 하는 루트 모듈을 다른 모듈들로 구성되도록 해야 한다.  
모듈을 나누는 이유는 각 모듈마다 책임을 나누어 주어 모듈간의 응집도를 높이기 위함이다.  

### Import, Export
모듈은 가져오기와 내보내기 기능을 사용할 수 있다.
예를 들어 앱을 기동하기 위해선 AppModule에 Core Module과 로그를 기록하기 위한 Logger Module, 2개의 모듈을 Import 해야한다고 가정한다.
이때 Core Module에서 Logger Module을 Import, Export하고 AppModule에서는 Core Module만 Import하는 것으로 Logger Service를 사용할 수 있게 된다.
#### CommonModule
```js
@Module({
  imports: [LoggerModule],
  exports: [LoggerModule],
})
export class CoreModule { }
```
#### LoggerModule
```js
@Module({
  providers: [LoggerService],
  exports: [LoggerService],
})
export class LoggerModule { }
```
#### AppModule
```js
@Module({
  imports: [CoreModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```
#### AppController
```js
@Controller()
export class AppController {
  constructor(private readonly loggerService: LoggerService) { }

  @Get('/hello')
  getLoggerHello(): string {
    return this.loggerService.hello();
  }
}
```





## Middleware
![image](https://user-images.githubusercontent.com/41901043/164385899-7bfbcb4d-3a44-4126-bc8a-53ad295673ea.png)
미들웨어는 라우트 핸들러 전에 호출되는 함수로, 애플리케이션의 요청-응답 주기에 있는 미들웨어 기능이다.  

### 미들웨어의 기능
+ 모든 코드를 실행
+ 요청 및 응답 개체를 변경
+ 요청-응답 주기를 종료
+ 스택의 다음 미들웨어 함수를 호출

```js
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';

@Injectable()
export class LoggerMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    console.log('Request...');
    next();
  }
}
```

미들웨어는 ```@Injecttable()```가 있는 클래스에서 구현한다. ```NestMiddleware``` 인터페이스를 상속받아 구현해야 한다.  
```constructor```를 이용해 동일한 모듈 내에서 사용가능한 종속성(Service)을 주입할 수 있다.  

### 미들웨어 적용
```js
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './common/middleware/logger.middleware';
import { CatsModule } from './cats/cats.module';

@Module({
  imports: [CatsModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware)
      .forRoutes('cats');
  }
}
```  

```NestMoudle```을 ```app.module.ts```에 상속받아서 ```configure()```를 구현하는 것으로 모듈에 미들웨어를 적용할 수 있다.  
```forRoutes('cats')```은 ```/cats```의 경로로 들어오는 request에 대한 핸들러를 적용한다는 의미.  
```.forRoutes({ path: 'cats', method: RequestMethod.GET });``` 혹은 특정 요청 메서드 유형(Get, Post, ...)을 지정할 수도 있다.

```js
consumer
  .apply(LoggerMiddleware)
  .exclude(
    { path: 'cats', method: RequestMethod.GET },
    { path: 'cats', method: RequestMethod.POST },
    'cats/(.*)',
  )
  .forRoutes(CatsController);
```  
위와 같이 ```exclude```를 사용하여 미들웨어가 적용되는 경로를 제외할 수도 있다.  

### 미들웨어 기능 함수 사용
```js
import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`Request...`);
  next();
};
```
위와 같이 logger 미들웨어에 next라는 함수를 적용해서 실행시킬 수도 있다.


## Guard
Guard는 런타임에 존재하는 특정 조건(권한, 역할, ACL 등)에 따라 주어진 요청이 핸들러에 의해 처리될 수 있는지에 대한 여부를 결정한다. 이를 종종 **권한 부여 또는 인증**라고 한다.  

### 권한 부여
Guard가 사용되는 예로는 인증이 된 특정 사용자에게 충분한 권한이 있는 경우에만 특정한 경로를 사용할 수 있도록 하는 기능이다.
```js
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(
    context: ExecutionContext,
  ): boolean | Promise<boolean> | Observable<boolean> {
    const request = context.switchToHttp().getRequest();
    return validateRequest(request);
  }
}
```
위 처럼 Guard는 ```CanActivate```의 기능을 구현해야 한다. 이 함수는 현재 요청이 허용되는지에 대한 여부 ```boolean```, 또는 동기식, 비동기식으로 사용되는 ```Promise``` or ```Observable```으로 반환값을 넘겨줄 수 있다.  
이 반환 값을 통해 아래와 같은 작업을 제어한다.
+ true : 요청을 처리
+ false : 요청을 거부  


## 실행 컨텍스트
실행 컨텍스트는 권한 부여, 필터, 인터셉터를 빌드하는데 사용할 수 있는 정보가 담긴 ```ArgumentsHost```과 ```ExecutionContext```를 제공한다.  
```canActivate```는 ```ExcecutionContext```라고 하는 인스턴스를 가지는데, 이 인스턴스는 ```ArgumentsHost```를 상속 받는 객체다.   
```ArgumentsHost```는 요청 핸들러에 전달되는 인수에 접근하여 정보를 확인하기 위한 메서드를 제공하는 클래스로, 인수를 검색하기 위해 http, rpc, websocket과 같은 컨텍스트를 선택하여 애플리케이션 유형을 판별한다.
```js
if (host.getType() === 'http') {
  // do something that is only important in the context of regular HTTP requests (REST)
} else if (host.getType() === 'rpc') {
  // do something that is only important in the context of Microservice requests
} else if (host.getType<GqlContextType>() === 'graphql') {
  // do something that is only important in the context of GraphQL requests
}
```  

```js
const request = context.switchToHttp().getRequest();
return validateRequest(request);
```
위와 같이 context를 판별하여 해당 request가 적합한 request인지 확인하는 절차를 진행한다.


### 호스트 핸들러
핸들러에 전달되는 배열을 검색하기 위해 ```getArgs()``` 메서드를 사용한다.
```js
const [req, res, next] = host.getArgs();
// or
const request = host.getArgByIndex(0);
const response = host.getArgByIndex(1);
```




