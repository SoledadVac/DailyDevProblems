> 本文的只是从开发角度所写的开发文档，并不全面。

# 获取日志操作类
```
private static final Logger logger = LoggerFactory.getLogger(AuthorizeController.class);
```
# 注意日志的输出级别
- `ERROR`:系统中发生了非常严重的问题，必须马上有人进行处理。没有系统可以忍受这个级别的问题的存在。比如：NPEs（空指针异常），数据库不可用，关键业务流程中断等等.

- `WARN`:发生这个级别问题时，处理过程可以继续，但必须对这个问题给予额外关注。这个问题又可以细分成两种情况：一种是存在严重的问题但有应急措施（比如数据库不可用，使用Cache）；第二种是潜在问题及建议（ATTENTION），比如生产环境的应用运行在Development模式下、管理控制台没有密码保护等。系统可以允许这种错误的存在，但必须及时做跟踪检查.

- `INFO`:重要的业务处理已经结束。在实际环境中，系统管理员或者高级用户要能理解INFO输出的信息并能很快的了解应用正在做什么。比如，一个和处理机票预订的系统，对每一张票要有且只有一条INFO信息描述 "[Who] booked ticket from [Where] to [Where]"。另外一种对INFO信息的定义是：记录显著改变应用状态的每一个action，比如：数据库更新、外部系统请求

- `DEBUG`:用于开发人员使用。将在TRACE章节中一起说明这个级别该输出什么信息.

- `TRACE` :非常具体的信息，只能用于开发调试使用。部署到生产环境后，这个级别的信息只能保持很短的时间。这些信息只能临时存在，并将最终被关闭。要区分DEBUG和TRACE会比较困难，对一个在开发及测试完成后将被删除的LOG输出，可能会比较适合定义为TRACE级别.
# 减少日志的打印
对于info/debug级别的日志，为了提高并发效率，减少信息计算量及输出量，采用如下方式书写：
```
        if(logger.isDebugEnabled()){
            //todo: 打印debug
        }
        if(logger.isInfoEnabled()){
            //todo:打印info
        }
```
系统越复杂，就越需要减少你的无用日志。
# ERROR打印
```
logger.error("各类参数对象tostring"+"_"+e.getMessage(),e);//保留案发现场信息和堆栈信息
throw new MyException();//视需要而定
```
# Controller
对于controller类来说，最重要的就是请求应答参数。为了方便追踪请求的调用链及对具体某一层的错误追踪，所以我们需要在日志中加入类信息，方法信息，请求路径的信息.
例如，请求参数：
```
    if(logger.isInfoEnabled()){
        logger.info("AuthorizeController_login [/auth/login] 用户登陆:"+JSONObject.toJSONString(authenticationRequest));
    }
```
回复信息：
```
    if(logger.isInfoEnabled()){
        logger.info("AuthorizeController_login [/auth/login] 用户获取token:"+JSONObject.toJSONString(authorizeTokenReponse));
    }
```
总结起来，log的基本信息就是：类名称_方法名称 [请求url] +自定义信息描述+需要输出的信息。
这里，需要输出的信息建议采用tojson方式，方便获取到这部分内容之后，放到json的格式化工具，就可以很清晰的看到对象的数据。
# Service
对于service方法，我们主要需要打印的是一些逻辑操作的中间值，比如我们开发过程中经常打断点监视的一些变量的值，多采用debug基本打印，具体格式如下：
```
       // Reload password post-security so we can generate token
       final UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        if(logger.isDebugEnabled()){
            logger.debug("AuthServiceImpl_login [/auth/login] 用户登陆，根据用户名查询到用户:"+ JSONObject.toJSONString(userDetails));
        }
```
- 在service方法中，由于我们经常会进行一些http调用，RPC调用，这种情况下，我们要着重对远程调用的参数，返回值进行info打印，对异常捕获进行error打印，并着重打印异常发生时候的方法时候的各类重要参数。因为这类调用很容易发生错误，应当尽量多打印信息。
# Dao
dao中操作比较单一，主要注意的是传入sql的参数，sql语句的情况。
另外，对于耗时的sql操作，可计算执行时间，如果时间过长，给出warn警告。

# 示例
- 根据请求url查询日志：
```
localhost:logs liuhuichao$ tail -f esbp-battery-admin.log |grep /auth/login
2017-11-15 15:24:52,711 [http-nio-8888-exec-2] INFO  c.w.e.b.a.a.c.AuthorizeController [AuthorizeController.java:48] AuthorizeController_login [/auth/login] 用户登陆:{"password":"1","username":"1"}
2017-11-15 15:24:52,734 [http-nio-8888-exec-2] DEBUG c.w.e.b.a.a.s.AuthServiceImpl [AuthServiceImpl.java:66] AuthServiceImpl_login [/auth/login] 用户登陆，根据用户名查询到用户:{"accountNonExpired":true,"accountNonLocked":true,"authorities":[{"authority":"ADMIN"}],"credentialsNonExpired":true,"enabled":true,"password":"1","username":"1"}
2017-11-15 15:24:53,041 [http-nio-8888-exec-2] INFO  c.w.e.b.a.a.c.AuthorizeController [AuthorizeController.java:56] AuthorizeController_login [/auth/login] 用户获取token:{"errorcode":8003,"errorinfo":"登陆成功","token":"eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIxIiwiY3JlYXRlZCI6MTUxMDczMDY5MjczNCwiZXhwIjoxNTExMzM1NDkyfQ.15X6UTlerUliV0qxIs06eWRkqFLzlFD3e8QV_rc7LMdGxQlvsSzsN29JmijNOc5KVQnWkoN5Q9FJ4IhDJtipvw"}
```
- 根据方法查询日志：
```
localhost:logs liuhuichao$ tail -f esbp-battery-admin.log |grep AuthServiceImpl_login
2017-11-15 15:24:52,734 [http-nio-8888-exec-2] DEBUG c.w.e.b.a.a.s.AuthServiceImpl [AuthServiceImpl.java:66] AuthServiceImpl_login [/auth/login] 用户登陆，根据用户名查询到用户:{"accountNonExpired":true,"accountNonLocked":true,"authorities":[{"authority":"ADMIN"}],"credentialsNonExpired":true,"enabled":true,"password":"1","username":"1"}
```
