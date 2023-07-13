## curl for testing

## cross-domain request
Refers to a web browser's attempt to make an HTTP request from one domain to another domain. It is restricted by web browswes by default for safety reason.

## Cookies
用于存储信息，由服务器端创建，在客户端保存并使用。
### 3 purposes: 
- 会话（Session）管理。比如保持登录，保持购物车内容，积分等服务器端应该记住的东西。
- 个性化。比如用户对设置的偏好。
- 跟踪。 记录与分析用户行为。

Cookie的发送场景：在每次`Request`时包在`Header`里一起发送。

### Creating cookies
服务器端负责创建`cookie`：在服务器端接收到`HTTP request`之后，对客户端发出的`HTTP response`中的`header`可以带上`Set-Cookie`字段，为当前`Request`的客户端设置`Cookie`并发送给客户端。
- `Response`指示设置两个`Cookie`：
```HTTP
HTTP/2.0 200 OK
Content-Type: text/html 
Set-Cookie: yummy_cookie=choco 
Set-Cookie: tasty_cookie=strawberry [page content] 
```
- 客户端接下来的`Request header`都将带有这两个`Cookie`:
```HTTP
GET /sample_page.html HTTP/2.0 
Host: www.example.org 
Cookie: yummy_cookie=choco; tasty_cookie=strawberry 
```

### Define the lifetime of a cookie
服务器端在设置`Cookie`时可以定义过期时间，过期后`Cookie`不再被客户端发送。例：
` Set-Cookie: id=a3fWa; Expires=Thu, 31 Oct 2021 07:28:00 GMT; `
- Note: 当你给用户放权时，不管之前有没有`Cookie`，都得重新生成一个并发送给客户端。这样可以防止之前未被放权时生成的`Cookie`被第三方滥用。

### Restrict access to cookies
在服务器端设置`Cookie`的字段中添加`Secure`和`HttpOnly`属性：
`Set-Cookie: id=a3fWa; Expires=Thu, 21 Oct 2021 07:28:00 GMT; Secure; HttpOnly `
- Secure: 只能使用`HTTPS`发送，不支持`HTTP`。但可以在本地硬盘里被访问、修改。
- HttpOnly: 不能使用`JavaScript`访问。防止上述情况。

### Domain attribute
- 服务器创建`Cookie`时如果忽略`Domain`字段，则默认这个`Cookie`用于发送到当前创建`Cookie`的服务器端（域名），在这种情况下，`Cookie`不会支持发送到相应的子域名。
- 如果显式设置`Domain`，则`Cookie`可以发送到相应子域名。

### Path attribute
`Path`属性指明`Cookie`能被发送到服务器端下的哪条路径。它包含显式指出的路径及其子路径。例如，设置`Path=/docs`，那么`Cookie`也能被发送到以下`Request`路径：
- `/docs`
- `/docs/`
- `/docs/Web/`
- `/docs/Web/HTTP`

### SameSite attribute
这个属性指明`Cookie`能不能由跨`site`的`request`发送。
- `Strict`: 只能由`Cookie`的原`site`发送。
- `Lax`: 当来自非原`site`的用户导航到原`site`时，也可以发送该`Cookie`。
- `none`: `Cookie`的`secure`属性必须设置。来自其他`site`的`request`也可以发送该`Cookie`。

#### Site
由`scheme`和`domain`组成。例如`http://github.com`和`https://github.com`是两个不同`site`。

TODO: https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#creating_cookies
Cookie predixes