## Win10 + VSCODE + GoFrame
```
go get github.com/gogf/gf/cmd/gf/v2
go install github.com/gogf/gf/cmd/gf/v2
```
Ensure your Path in Environment Variable has `%GOPATH%/src/bin`;
Reboot PC;

## RESTful
Implementing variety of handlers for variety of methods(`"GET|POST|..."`).

## A readthrough about `gf-demo-user`
### api/user/v1:
user.go:
定义了一些HTTP请求和响应的结构体。包括个人信息、登录、护照、昵称、登录状态。

### hack:
config.yaml:
配置GoFrame CLI工具，用于生成Data Access Object (DAO)。

### internal/cmd:
cmd.go:
主程序，调用其它包的函数完成以下功能：
初始化HTTP服务器；
设置中间件:
- Middleware function sits between the web server and the route handlers to provide a way to intercept and process HTTP requests and responses.  

设置路由:
- Routers are responsible for directing incoming requests to the appropriate route handlers based on the request's URL, HTTP method, and other criteria.  

配置openAPI文档:
- For developers to describe and document RESTful APIs in standardized format.

### internal/consts:
项目内所有常量

### internal/controller/user:
user.go:
根据api中定义的对应结构体，接收request，返回reponse。本身不实现业务逻辑，通过调用service实现。
- 难怪叫接口层

### internal/dao/internal:
user.go:
GoFrame CLI tool自动生成，用于与底层数据库的交互。

### internal/dao：
user.go:
GoFrame CLI tool自动生成。
- Q：为什么要有两层？
```
|dao
  |internal
    |user.go
  |user.go
```
- A: `dao`根目录的user.go用于提供简化的接口，是`./dao/internal/user.go`的高级抽象。

### internal/logic/bizctx:
bizctx.go: