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
