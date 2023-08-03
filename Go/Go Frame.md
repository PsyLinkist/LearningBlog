## Win10 + VSCODE + GoFrame
```
go get github.com/gogf/gf/cmd/gf/v2
go install github.com/gogf/gf/cmd/gf/v2
```
Ensure your Path in Environment Variable has `%GOPATH%/src/bin`;
Reboot PC;

---
# Core Components
## 数据库ORM
- 一般使用配置文件管理项目的数据库。  
    使用`g`对象管理模块的`g.DB("数据库分组名称")`来获取配置文件中相应的数据库操作对象。  
    例：
    ```yaml
    database:
    default: //数据库分组名称
        link:  "mysql:root:12345678@tcp(127.0.0.1:3306)/test"
    user:
        link:  "sqlite::@file(/var/data/db.sqlite3)"
    ```
- 也支持用函数手动配置。

### ORM链式操作
通过数据库对象的`db.Model`方法或者事务对象的`tx.Model`方法进行操作，基于指定数据表返回一个链式操作对象`*Model`，该对象可以执行各种数据库方法。

#### 模型及其创建
- `Model`即是对数据库中指定表的映射。  
- 创建方式有两种：
   - 完全映射，将整张表都映射下来。
        ```go
        g.Model("<表名>")
        // OR
        g.DB().Model("<表名>")
        ```
   - 基于原始`SQL`语句的部分映射。
        ```go
        s := "SELECT * FROM `user`"
        m, _ := g.ModelRaw(s).WhereLT("age", 18).Limit(10).OrderAsc("id").All()
        // SELECT * FROM `user` WHERE `age`<18 ORDER BY `id` ASC LIMIT 10
        // ?的使用
        s := "SELECT * FROM `user` WHERE `status` IN(?)"
        m, _ := g.ModelRaw(s, g.Slice{1,2,3}).WhereLT("age", 18).Limit(10).OrderAsc("id").All()
        // SELECT * FROM `user` WHERE `status` IN(1,2,3) AND `age`<18 ORDER BY `id` ASC LIMIT 10
        ```
##### 链式安全
本质上是映射的`Model`对象能否重复使用的问题。  
默认情况下，基于性能以及GC优化考虑，模型对象为**非链式安全**，每次修改都在`Model`上直接改动属性，以防止产生过多的临时模型对象。
- `Clone()`，克隆出新的`Model`，不会污染之前的`Model`。
- `Safe()`，在定义`Model`的时候使用，每次链式操作都能返回一个新的`Model`，并且能赋值到旧的`Model`上。这样既实现了**链式安全**，又能防止产生过多临时模型对象。  
   例：
    ```go
    // 定义一个用户模型单例
    user := g.Model("user").Safe()
    m := user.Where("status", g.Slice{1,2,3})
    if vip {
        // 查询条件通过赋值叠加
        m = m.Where("money>=?", 1000000)
    } else {
        // 查询条件通过赋值叠加
        m = m.Where("money<?",  1000000)
    }
    //  vip: SELECT * FROM user WHERE status IN(1,2,3) AND money >= 1000000
    // !vip: SELECT * FROM user WHERE status IN(1,2,3) AND money < 1000000
    r, err := m.All()
    //  vip: SELECT COUNT(1) FROM user WHERE status IN(1,2,3) AND money >= 1000000
    // !vip: SELECT COUNT(1) FROM user WHERE status IN(1,2,3) AND money < 1000000
    n, err := m.Count()
    ```

### 数据库数据转化到程序中的数据
`gf`将数据库中的数据查询出来，并转化到程序中进行使用的流程。
1. `gf gen dao`会自动生成`dao`和`model`两部分文件。
    - `dao`是抽象对象，直接交互底层数据库，只包含基础`CURD`方法。
    - `model`包括`do`和`entity`两部分：
        - `entity`作为数据库中表在程序中的映射。常用`Scan`方法将查询到的数据库数据转化为`entity`中对应的结构体。例：
            ```go
            // user *entity.User
            err := dao.User.Ctx(ctx).Where(g.Map{
                dao.User.Columns().Passport: passport,
                dao.User.Columns().Password: passwora,
            }).Scan(&user) // Scan将查询到的数据转为user结构体
            ```
        - `do`用于业务模型与实例模型转换，常用作`Where()`与`Data()`的参数。例：
            ```go
            r, err := dao.Interact.Ctx(ctx).Data(do.Interact{
                // ... Interact struct initiating
            }).InsertIgnore() // insert into "Interact" table
            ```
2. `Scan(&user)`转化过程：利用`reflection`检查查询到的数据`params`与结构体`pointer`的对应数据类型是否一致，如果一致，则将`params`转化为带值的`pointer`。
---
# Web App
## RESTful
Implementing variety of handlers for variety of methods(`"GET|POST|..."`).

## Web Server
### g.Server().Run()
Starts listening in a blocking way, which can keep accept Requests until shut down of program.
- Suitable for single server situation.

### g.Server().Start() && g.Wait()
`Start()` starts listening without blocking, therefore, the program would exit straighly if we don't call `g.Wait()` to manually block the server process.
- Suitable for multi-server situation.

---
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
- Note: 实际上是对象注册的方式。
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

---
## OpenAPIv3
[Official Doc](https://goframe.org/pages/viewpage.action?pageId=47703679)
It provides a standardized format for building and documenting RESTful APIs.  
`GoFrame`will automatically generate it. And `GoFrame` mostly uses it in **Format Routers**.
For example, defining `*Request` and `*Response`:  
```go
type HelloReq struct {
	g.Meta `path:"/hello" method:"get"`
	Name   string `v:"required" dc:"Itadori Yuuji"`
}
type HelloRes struct {
	Reply string `dc:"love you."`
}
```

---
## Request
依靠`ghttp.Request`获取输入的信息，并生成`ghttp.Response`。  
`GoFrame`以提交类型的方式决定参数的获取：
1. Router
2. Query
3. Form
4. Body
5. Custom //自定义参数
- 参数重名时，优先级从5到1.

### 复杂参数
`ghttp.Request`可以智能地解析请求参数。
详见[官方文档](https://goframe.org/pages/viewpage.action?pageId=1114197)。

### 参数转换
将请求处理的输入输出都转化为结构体，可以更方便地操作和维护数据。  
`GoFrame`能将请求传入的各种参数转换成标准化的结构体，主要通过`Request`对象的`Parse`方法或者`Get*Struct`方法。  
基本的转换规则：
1. `struct`中的**公开属性**需要首字母大写。
2. 传入参数则**不区分大小写**，**忽略**`-/_/空格`。
3. 匹配成功，键值赋给属性；无法匹配，则忽略键值。

### `lseRequest`对象的校验特性
[官方文档](https://goframe.org/pages/viewpage.action?pageId=1114244)
通过给请求对象的结构体绑定`v`标签，即可便捷地对传入数据进行校验。
例：
```go
type RegisterReq struct {
    Name  string `p:"username"  v:"required|length:4,30#请输入账号|账号长度为:{min}到:{max}位"`
    Pass  string `p:"password1" v:"required|length:6,30#请输入密码|密码长度不够"`
    Pass2 string `p:"password2" v:"required|length:6,30|same:password1#请确认密码|密码长度不够|两次密码不一致"`
}
```
- `|`用于分隔校验条件。

#### 校验错误的处理
可以将校验错误转换为`gvalid.Error`接口对象，更灵活地处理校验错误。  
例：
```go
...
group.ALL("/register", func(r *ghttp.Request) {
    var req *RegisterReq
    if err := r.Parse(&req); err != nil {
        if v, ok := err.(gvalid.Error); ok { // 用断言方式判断是否为校验错误
            r.Response.WriteJsonExit(RegisterRes{
                Code: 1,
                Error: v.FirstError().Error(),
            })
        }
    }
})
...
```

### `Request`参数的默认值
可以用标签设置`*Req`的默认值，如下：
```go
type GetListReq struct {
    g.Meta `path:"/" method:"get"`
    Page int `v:"min0#分页号码错误" dc:"分页号码" d:"1"` // 如果传入参数不包括`Page`，则分页号码为默认值1
}
```

### 自定义变量
通过`func (r *Request) SetParam(key string, value interface{})`设置自定义变量，普通的请求参数的获取方法如`Get/GetVar/GetMap`和特定的自定义变量获取方法`GetParam/GetParamVar`都可以获取自定义变量。  
例如，在前置中间件设置好自定义变量，在服务器处理请求过程中可以获取到设置好的自定义变量：
```go
package main

import (
	"github.com/gogf/gf/v2/frame/g"
	"github.com/gogf/gf/v2/net/ghttp"
)

// 前置中间件1
func MiddlewareBefore1(r *ghttp.Request) {
	r.SetParam("name", "GoFrame") //设置自定义变量
	r.Response.Writeln("set name")
	r.Middleware.Next()
}

// 前置中间件2
func MiddlewareBefore2(r *ghttp.Request) {
	r.SetParam("site", "https://goframe.org") //设置自定义变量
	r.Response.Writeln("set site")
	r.Middleware.Next()
}

func main() {
	s := g.Server()
	s.Group("/", func(group *ghttp.RouterGroup) {
		group.Middleware(MiddlewareBefore1, MiddlewareBefore2)
		group.ALL("/", func(r *ghttp.Request) {
			r.Response.Writefln(
				"%s: %s",
                // 获取自定义变量
			    r.GetParam("name").String(),
			    r.GetParam("site").String(),
			)
		})
	})
	s.SetPort(8199)
	s.Run()
}
```

### Context
继承自标准库的`context.Context`接口。标准库的`context`主要用于传递`Cancel signal|Deadline|Timeout|Value`这类型的数据，主要用于控制进程生命周期。

---
## Response
[官方文档](https://goframe.org/pages/viewpage.action?pageId=1114485)
`HTTP Server`响应请求，返回数据通过`ghttp.Response`实现。`Write*`相关方法采用缓冲机制输出数据，因此也可以直接对缓冲区进行数据操作。  
对`Header`的操作可以通过标准库来实现：  
`Response.Header().Set("Content-Type", "text/plain; charset=utf-8")`

### 使用缓冲区控制数据
`Response`采用缓冲区机制，数据先输入到缓冲区，等服务方法执行完毕后才输出到客户端，因此在输出到客户端之前，可以对缓冲区进行必要的数据处理。

### Redirect
`Request.Response`通过`RedirectTo|RedirectBack`实现页面之间的跳转。该功能通过对`Header`的`location`进行操作实现：
```go
func (r *Response) RedirectTo(location string, code ...int) {
	r.Header().Set("Location", location)
	if len(code) > 0 {
		r.WriteHeader(code[0])
	} else {
		r.WriteHeader(http.StatusFound)
	}
	r.Request.Exit()
}
```
- 注意：`RedirectBack`是返回进入该函数所在的路由页面之前所在的页面，例如：
    ```go
    package main

    import (
        "github.com/gogf/gf/v2/frame/g"
        "github.com/gogf/gf/v2/net/ghttp"
    )

    func main() {
        s := g.Server()
        s.BindHandler("/page", func(r *ghttp.Request) {
            r.Response.Writeln(`<a href="/back">back</a>`)
        })
        s.BindHandler("/back", func(r *ghttp.Request) {
            r.Response.RedirectBack() // 之前在/page页面，那么进入/back页面时，自动返回到上一页面，也就是/page页面。
        })
        s.SetPort(8199)
        s.Run()
    }
    ```

### Exit控制
[官方文档](https://goframe.org/pages/viewpage.action?pageId=38569768)
1. `Exit`退出当前方法。类似`return`。
2. `ExitAll`退出当前及后续的所有流程。
3. `ExitHOOK`在有多个`HOOK`方法连续执行时，退出当前及后续的所有`HOOK`方法。

### 文件下载
[官方文档](https://goframe.org/pages/viewpage.action?pageId=20086853)
```go
func (r *Response) ServeFile(path string, allowIndex ...bool)
func (r *Response) ServeFileDownload(path string, name ...string)
```

### 模板解析
`Response`使用`WriteTpl*`和`ParseTpl*`方法进行模板的解析及输出。  
- `ParseTpl*`方法对模板进行解析并`return`解析结果：
    ```go
    func (r *Response) ParseTplContent(content string, params ...gview.Params) (string, error) {
        return r.Request.GetView().ParseContent(r.Request.Context(), content, r.buildInVars(params...))
    }
    ```
- `WriteTpl*`用于输出`ParseTpl*`返回的解析结果：
    ```go
    func (r *Response) WriteTplContent(content string, params ...gview.Params) error {
        r.Header().Set("Content-Type", contentTypeHtml)
        b, err := r.ParseTplContent(content, params...) // ParseTpl*返回解析结果
        if err != nil {
            if !gmode.IsProduct() {
                r.Write("Template Parsing Error: " + err.Error())
            }
            return err
        }
        r.Write(b) // 将解析结果输出到Response缓冲区
        return nil
    }
    ```
- `Response`的解析支持一些请求相关的内置变量：
   - Config 
   - Cookie
   - Session
   - Query
   - Form
   - Request
    使用方式为`{{.<对象名>.键名}}`，例：`{{.Config.配置项}}`。可以访问当前请求的对象参数值，并写入到输出的模板中。  
    在解析过程中，如果有指定的内置变量`content`，则忽略模板文件路径。

### Stream Response
1. 设置`Header`参数。
   ```go
    r.Response.Header().Set("Content-Type", "text/event-stream")
	r.Response.Header().Set("Cache-Control", "no-cache")
	r.Response.Header().Set("Connection", "keep-alive")
   ```
2. `flush`缓冲区的数据。
   ```go
    for i := 0; i < 100; i++ {
        r.Response.Writefln("data: %d", i)
        r.Response.Flush()
        time.Sleep(time.Millisecond * 200)
        }
   ```

---
## Server Setting
### 使用配置文件对`Server`进行配置
在[Official doc](https://pkg.go.dev/github.com/gogf/gf/v2/net/ghttp#ServerConfig)查看具体配置定义。
1. 在`config.yaml`中配置服务器。
2. 多个`Server`配置时使用`g.Server("<server-name>")`读取对应的配置项并自动配置：  
    ```yaml
    server:
        address:    ":80"
        serverRoot: "/var/www/Server"
        server1:
            address:    ":8080"
            serverRoot: "/var/www/Server1"
        server2:
            address:    ":8088"
            serverRoot: "/var/www/Server2"
    ```  
模板示例：
```yaml
server:
    # 基本配置
    address:             ":80"                        # 本地监听地址。默认":80"，多个地址以","号分隔。例如："192.168.2.3:8000,10.0.3.10:8001"
    httpsAddr:           ":443"                       # TLS/HTTPS配置，同时需要配置证书和密钥。默认关闭。配置格式同上。
    httpsCertPath:       ""                           # TLS/HTTPS证书文件本地路径，建议使用绝对路径。默认关闭
    httpsKeyPath:        ""                           # TLS/HTTPS密钥文件本地路径，建议使用绝对路径。默认关闭
    readTimeout:         "60s"                        # 请求读取超时时间，一般不需要配置。默认为60秒
    writeTimeout:        "0"                          # 数据返回写入超时时间，一般不需要配置。默认不超时（0）
    idleTimeout:         "60s"                        # 仅当Keep-Alive开启时有效，请求闲置时间。默认为60秒
    maxHeaderBytes:      "10240"                      # 请求Header大小限制（Byte）。默认为10KB
    keepAlive:           true                         # 是否开启Keep-Alive功能。默认true
    serverAgent:         "GoFrame HTTP Server"        # 服务端Agent信息。默认为"GoFrame HTTP Server"

    # 接口文档
    openapiPath: "/api.json" # OpenAPI接口文档地址
    swaggerPath: "/swagger"  # 内置SwaggerUI展示地址

    # 静态服务配置
    indexFiles:          ["index.html","index.htm"]   # 自动首页静态文件检索。默认为["index.html", "index.htm"]
    indexFolder:         false                        # 当访问静态文件目录时，是否展示目录下的文件列表。默认关闭，那么请求将返回403
    serverRoot:          "/var/www"                   # 静态文件服务的目录根路径，配置时自动开启静态文件服务。默认关闭
    searchPaths:         ["/home/www","/var/lib/www"] # 提供静态文件服务时额外的文件搜索路径，当根路径找不到时则按照顺序在搜索目录查找。默认关闭
    fileServerEnabled:   false                        # 静态文件服务总开关。默认false

    # Cookie配置
    cookieMaxAge:        "365d"             # Cookie有效期。默认为365天
    cookiePath:          "/"                # Cookie有效路径。默认为"/"表示全站所有路径下有效
    cookieDomain:        ""                 # Cookie有效域名。默认为当前配置Cookie时的域名

    # Sessions配置
    sessionMaxAge:       "24h"              # Session有效期。默认为24小时
    sessionIdName:       "gfsessionid"      # SessionId的键名名称。默认为gfsessionid
    sessionCookieOutput: true               # Session特性开启时，是否将SessionId返回到Cookie中。默认true
    sessionPath:         "/tmp/gsessions"   # Session存储的文件目录路径。默认为当前系统临时目录下的gsessions目录

    # 日志基本配置
    logPath:             ""                 # 日志文件存储目录路径，建议使用绝对路径。默认为空，表示关闭
    logStdout:           true               # 日志是否输出到终端。默认为true
    errorStack:          true               # 当Server捕获到异常时是否记录堆栈信息到日志中。默认为true
    errorLogEnabled:     true               # 是否记录异常日志信息到日志中。默认为true
    errorLogPattern:     "error-{Ymd}.log"  # 异常错误日志文件格式。默认为"error-{Ymd}.log"
    accessLogEnabled:    false              # 是否记录访问日志。默认为false
    accessLogPattern:    "access-{Ymd}.log" # 访问日志文件格式。默认为"access-{Ymd}.log"

    # 日志扩展配置(参数日志组件配置)
    logger:
      path:                  "/var/log/"   # 日志文件路径。默认为空，表示关闭，仅输出到终端
      file:                  "{Y-m-d}.log" # 日志文件格式。默认为"{Y-m-d}.log"
      prefix:                ""            # 日志内容输出前缀。默认为空
      level:                 "all"         # 日志输出级别
      stdout:                true          # 日志是否同时输出到终端。默认true
      rotateSize:            0             # 按照日志文件大小对文件进行滚动切分。默认为0，表示关闭滚动切分特性
      rotateExpire:          0             # 按照日志文件时间间隔对文件滚动切分。默认为0，表示关闭滚动切分特性
      rotateBackupLimit:     0             # 按照切分的文件数量清理切分文件，当滚动切分特性开启时有效。默认为0，表示不备份，切分则删除
      rotateBackupExpire:    0             # 按照切分的文件有效期清理切分文件，当滚动切分特性开启时有效。默认为0，表示不备份，切分则删除
      rotateBackupCompress:  0             # 滚动切分文件的压缩比（0-9）。默认为0，表示不压缩
      rotateCheckInterval:   "1h"          # 滚动切分的时间检测间隔，一般不需要设置。默认为1小时

    # PProf配置
    pprofEnabled:        false              # 是否开启PProf性能调试特性。默认为false
    pprofPattern:        ""                 # 开启PProf时有效，表示PProf特性的页面访问路径，对当前Server绑定的所有域名有效。

    # 其他配置
    clientMaxBodySize:   810241024          # 客户端最大Body上传限制大小，影响文件上传大小(Byte)。默认为8*1024*1024=8MB
    formParsingMemory:   1048576            # 解析表单时的缓冲区大小(Byte)，一般不需要配置。默认为1024*1024=1MB
    nameToUriType:       0                  # 路由注册中使用对象注册时的路由生成规则。默认为0
    routeOverWrite:      false              # 当遇到重复路由注册时是否强制覆盖。默认为false，重复路由存在时将会在启动时报错退出
    dumpRouterMap:       true               # 是否在Server启动时打印所有的路由列表。默认为true
    graceful:            false              # 是否开启平滑重启特性，开启时将会在本地增加10000的本地TCP端口用于进程间通信。默认false             
    gracefulTimeout:     2                  # 父进程在平滑重启后多少秒退出，默认2秒。若请求耗时大于该值，可能会导致请求中断
```

### 使用配置方法对`Server`进行配置
- 可以通过`Server`对象的`SetConfig`或`SetConfigWithMap`方法设置。
- `Server`对象的`Set*|Enable*`等方法可以进行特定配置的设置。

---
## Cookie & Session
二者都属于`ghttp.Request`的成员对象，并对外公开。
### Cookie
继承自标准库`http.Cookie`。  
- `Cookie`对象不需要手动关闭，请求流程结束后，`HTTP Server`会自动关闭`Cookie`。
- 默认通过`Cookie`传递`SessionId`
- `Session`对象如果不再被需要，则手动调用`RemoveAll`方法。

### Session
#### Session的存储
```go
s := g.Server()
s.SetSessionStorage(storage gsession.Storage) // 修改存储方法
```
##### File
默认情况下，`ghttp.Server`的`Session`存储使用**文件+内存**的方式存储，使用`StorageFile`对象实现：
1. 在内存中对`Session`进行操作；
2. 通过存入文件来保持数据持久性；
3. 使用 `gcache`控制过期；
4. 当且仅当内存中的`Session`更新时（标记`dirty`），使用`json.Marshal`将更新后的`Session`传入文件中；
5. 当且仅当内存中不存在`Session`时，使用`json.Unmarshal`将文件中的`Session`读出，恢复数据。
- 适合`Session`读多写少的情况。

##### Memory


##### Redis-KeyValue


##### Redis-HashTable


#### Session的初始化
#### Session的注销

---
## 分页管理
### 动态分页与静态分页的区别
静态分页相比动态分页，多了一个路由，也就是说，访问页面需要使用特定的静态路由；而动态页面能通过 `Query`的形式直接访问对应页面，例如`.../page/demo?name=<page-number>`。  
究其原因，是`ghttp.Request`能自动处理`Query`。

# `focus-single` template project

## DAO (Data Access Object)
`Dao` encapsulate the interactions with specific data source, which allows the rest of the application to interact with the data source through a set of consistent methods and operations.

### How does `dao` connect to database and work in the project
We have tool configuration in the `.../hack` file, in which we set the generation of `dao`:  
![dao gen](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics/202307140951229.png)

## Q & A
### 模块化
- Q：logic中的函数是基于什么分开的，集中的Interface{}只起到抽象接口的作用吗？那么以什么作为接口分类的依据呢？是以操作的对象表吗？  
    A：在`service`中设置一个抽象接口函数，返回接口，用于在应用中调用。肯定不是以操作的对象表分离`logic`，因为里面还包含不需要数据库的功能。根据功能模块分割`logic`，集中的`interface{}`只是起到集中功能模块里功能函数的作用。

### 数据类型
- Q：数据库中的数据类型与entity生成的结构体内部数据类型不一致怎么处理？例如数据库中为`binary(2)`，而结构体则为`string`。`Scan`函数会自动转化吗？ 
    A：就是简单的用`.Type()`比较，相同则赋值。问题都提错了，生成的`dao`中的结构体是表`column`的名字，所以是`string`类型，真正的对应表中数据的结构体在生成的`model`中，使用`do`进行数据操作，`entity`则是表的结构的映射，将会用做`pointer`进行`Scan`。 
### DAO
- Q：DAO的操作方法 

    `gf gen dao`以及生成了`model`，通过`dao`直接调用就行；
    利用`Scan`方法，基于`gf gen dao`生成的`entity`中表的结构体映射，将`dao`查询到的数据转化为对应初始化结构体并使用。