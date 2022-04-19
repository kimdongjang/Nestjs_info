# nest_info
nestjs 정리

## Controller  
### @Req, @Res는 차후 퍼포먼스 문제로 위해 전환할 경우를 대비하기 위해 사용하지 않는 것이 좋다.
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
