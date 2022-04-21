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
  
#### @Query를 사용한 예(year=2022인 데이터를 가져오기)
```js
@Controller('movies')
export class MoviesController {
  @Get('/search')
  search(@Query('year') searchingYear: number) {
    return `We are Searching made after: ${searchingYear}`;
  }

}
```

  
#### req 대신 @Body를 사용한 예
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

## Middleware
![image](https://user-images.githubusercontent.com/41901043/164385899-7bfbcb4d-3a44-4126-bc8a-53ad295673ea.png)
미들웨어는 라우트 핸들러 전에 호출되는 함수로, 애플리케이션의 요청-응답 주기에 있는 미들웨어 기능이다.  

#### 미들웨어의 기능
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

#### 미들웨어 적용
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

#### 미들웨어 기능 함수 사용
```js
import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
  console.log(`Request...`);
  next();
};
```
위와 같이 logger 미들웨어에 next라는 함수를 적용해서 실행시킬 수도 있다.
