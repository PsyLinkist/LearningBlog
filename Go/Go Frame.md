## Win10 + VSCODE + GoFrame
```
go get github.com/gogf/gf/cmd/gf/v2
go install github.com/gogf/gf/cmd/gf/v2
```
Ensure your Path in Environment Variable has `%GOPATH%/src/bin`;
Reboot PC;

## RESTful
Implementing variety of handlers for variety of methods(`"GET|POST|..."`).

## Web Server
### g.Server().Run()
Starts listening in a blocking way, which can keep accept Requests until shut down of program.
- Suitable for single server situation.

### g.Server().Start() && g.Wait()
`Start()` starts listening without blocking, therefore, the program would exit straighly if we don't call `g.Wait()` to manually block the server process.
- Suitable for multi-server situation.

## Router
### 路由注册
指设置可以被匹配的地址，以支持对应的访问。例如：
```go
s.BindHandler("/user/list/{field}.html", func(r *ghttp.Request){
        r.Response.Writeln(r.Router.Uri)
    })
```
注册了一条名为"`/user/list/{field}.html`"的路由，任何符合该路由的请求(`Request`)都能被正确接收并产生响应(`Response`)。

#### 函数注册
可以是以传入函数的形式：
```go
func main() {
    s := g.Server()
    s.BindHandle(string, func(*ghttp.Request){
        //function content
    })
}

```
也可以传入包方法（Method）：
```go
var total = gtype.NewInt()

func Total(r *ghttp.Request) {
    //function content
}

func main() {
    s := g.Serve()
    s.BindHandle(string, Total)
}
```
或者对象方法：
```go
//对象
type Controller struct {
    total *gtype.Int
}
//对象的方法
func (c *Controller) Total(r *ghttp.Request) {
    //function content
}

func main() {
    s := g.Serve()
    s.BindHandle(string, c.Total)
}
```
- 简单易用

#### 对象注册
注册一个实例化的对象，常驻内存，之后每次请求都交给这一对象。  
##### 对象注册
`func (s *Server) BindObject(pattern string, object interface{}, methods ...string) error`的第三个参数可以传入多个对象的method，以隐藏未被传入的method。  
例：
```go
type Controller struct {}

func (c *Controller) Index(r *ghttp.Request) {
    r.Response.Write("index")
}

func (c *Controller) Show(r *ghttp.Request) {
    r.Response.Write("show")
}

func main() {
    s := g.Server()
    c := new(Controller) // 初始化对象
    s.BindObject("/object", c, "Show", "Index")
    s.SetPort(8199)
    s.Run()
}
```

##### 绑定路由与方法的注册
`func (s *Server) BindObjectMethod(pattern string, object interface{}, method string) error`的第三个参数只能传入一个method，绑定路由到该指定的method。  
例：
```go
type Controller struct{}

func (c *Controller) Index(r *ghttp.Request) {
    r.Response.Write("index")
}

func (c *Controller) Show(r *ghttp.Request) {
    r.Response.Write("show")
}

func main() {
    s := g.Server()
    c := new(Controller) // 初始化对象
    s.BindObjectMethod("/object", c, "Show")
    //s.BindObjectMethod("/object", c, "Show", "Index") // Error
    s.SetPort(8199)
    s.Run()
}
```

##### RESTful对象注册
`func (s *Server) BindObjectRest(pattern string, object interface{}) error`只注册对应HTTP method的方法，如`Get()`，`Post()`等，其他方法即使公开定义了，也不会注册。并且该函数还会把对应方法的HTTP请求绑定到controller的对应method上，例如`POST`请求将返回`Post()`。
例：
```go
type Controller struct{}

func (c *Controller) Get(r *ghttp.Request) {
    r.Response.Write("get")
}
func (c *Controller) Post(r *ghttp.Request) {
    r.Response.Write("post")
}
func (c *Controller) Delete(r *ghttp.Request) {
    r.Response.Write("delete")
}
// 无法访问，不会出现在路由列表中
func (c *Controller) SomeMethod(r *ghttp.Request) {
    r.Response.Write("something")
}

func main() {
	s := g.Server()
	c := new(Controller) //初始化对象
	s.BindObjectRest("/object", c)
	s.SetPort(8199)
	s.Run()
}
```

##### 构造方法Init()和析构方法Shut()
可以为对象设置这两种方法，在服务接口调用前自动回调执行`Init`，在请求结束后`Server`自动调用Shut。

#### 分组路由
- 分组注册的路由，组内所有路由都注册在同一指定路径下。
- 支持在指定域名对象上创建。
    ```go
    s := g.Server()
    group := s.Group("/api") //所有以"/api"为路由前缀的请求都将传入该路由组。
    ```
- 如果要让`HTTP Method`请求对应方法，则可以使用以`HTTP Method`命名的方法注册路由：
    ```go
    func main() {
        s := g.Server()
        group := s.Group("/api")
        //ALL方法可以被所有HTTP method访问
        group.ALL("/all", func(r *ghttp.Request) {
            r.Response.Write("all")
        })
        group.GET("/get", func(r *ghttp.Request) {
            r.Response.Write("get")
        })
        group.POST("/post", func(r *ghttp.Request) {
            r.Response.Write("post")
        })
        s.SetPort(8199)
        s.Run()
    }
    ```


#### 规范路由  
自定义`Response`和`Request`的结构体，定义空结构体作为对象。为空结构体定义方法，在方法中以`context.Context`传递数据，传入`Request`，生成`Response`。  
- 实际上是对象注册的方式  
例：
```go
package main

import (
	"context"
	"fmt"

	"github.com/gogf/gf/v2/frame/g"
	"github.com/gogf/gf/v2/net/ghttp"
)

type HelloReq struct {
    g.Meta `path:"/hello" method:"get"`
    Name string `v:"required" dc:"Your name"`
}

type HelloRes struct {
    Reply string `dc:"Reply content"`
}

type Hello struct {}// 对象
// 对象的方法
func (Hello) Say(ctx context.Context, req *HelloReq) (res *HelloRes, err error) {
	g.Log().Debugf(ctx, `receive say: %+v`, req)
    // generate response
	res = &HelloRes{
		Reply: fmt.Sprintf(`Hi %s`, req.Name),
	}
	return
}

func main() {
    s := g.Server()
    s.Use(ghttp.MiddlewareHandlerResponse)
    s.Group("/", func(group *ghttp.RouterGroup) {
        group.Bind(
            new(Hello),// 对象注册
        )
    })
    s.Run()
}
```
### Fuzzy matching routing rules
深度优先，路由越详细，优先级越高。
- `:`命名匹配
- `*`模糊匹配
- `{}`字段匹配
Note: 字段匹配 > 命名匹配 > 模糊匹配

### Middleware
中间件用于便利地实现从请求传入到给出响应之间的其他过程。

#### 中间件的功能
- 允许跨域请求：
    ```go
    func MiddlewareCORS(r *ghttp.Request) {
        r.Response.CORSDefault()
        r.Middleware.Next()
    }
    ```
- 请求鉴权处理：
    ```go
    func MiddlewareAuth(r *ghttp.Request) {
        token := r.Get("token")
        if token.String() == "123456" {
            r.Response.Writeln("auth")
            r.Middleware.Next()
        } else {
            r.Response.WriteStatus(http.StatusForbidden)
        }
    }
    ```
- 鉴权例外处理，使用分组路由中间件：
    ```go
    s.Group("/admin", func(group *ghttp.RouterGroup) {
        //不鉴权
        group.ALL("/login", func(r *ghttp.Request) {
            r.Response.Writeln("login")
        })
        //鉴权
        group.Group("/", func(group *ghttp.RouterGroup) {
            group.Middleware(MiddlewareAuth)
            group.ALL("/dashboard", func(r *ghttp.Request) {
                r.Response.Writeln("dashboard")
            })
        })
    })
    ```
- 错误处理
- 日志处理

#### 中间件的使用
##### 分组路由中间件与全局中间件
在不同路由注册的中间件作用范围不同。  
- 全局中间件的使用：
    ```go
    func (s *Server) Use(handlers ...HandlerFunc)

    s.Use(MiddlewareSTH)
    ```
   适合需要对全部请求起作用的中间件，如日志类型，需要记录所有请求。
- 分组路由中间件的使用：
    ```go
    func (g *RouterGroup) Middleware(handlers ...HandlerFunc) *RouterGroup

    group.Middleware(MiddlewareSTH)
    ```
##### 前置中间件与后置中间件
中间件实现功能的顺序有差别。
- 前置中间件在路由服务函数调用之前调用：
    ```go
    func Middleware(r *ghttp.Request) {
        // 中间件处理逻辑
        r.Middleware.Next()
    }
    ```
    可以用于执行服务函数前鉴权、允许跨域请求。
- 后置中间件在路由服务函数调用之后调用：
    ```go
    func Middleware(r *ghttp.Request) {
        r.Middleware.Next()
        // 中间件处理逻辑
    }
    ```
   常用于错误处理的情况，在服务函数执行返回错误后利用后置中间件进行错误处理。

