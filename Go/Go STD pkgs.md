# net
## net/http
`http` provides HTTP client and server implementations and funcs of HTTP(s) requests.
### Overview
#### Client and Transport
`type Client` have control over HTTP client headers. Just create one to modify it:  
```go
client := &http.Client{
	CheckRedirect: redirectPolicyFunc,
}

resp, err := client.Get("http://example.com")
// ...

req, err := http.NewRequest("GET", "http://example.com", nil)
// ...
req.Header.Add("If-None-Match", `W/"wyzzy"`)
resp, err := client.Do(req)
// ...
```
Transport has control over poxies, TLS configuration, keep-alives, compression, and other settings.  
```go
tr := &http.Transport{
	MaxIdleConns:       10,
	IdleConnTimeout:    30 * time.Second,
	DisableCompression: true,
}
client := &http.Client{Transport: tr}
resp, err := client.Get("https://example.com")
```
Note: Now we have knowledge about settings of _HTTP client_ and _HTTP Transport_, we should consider how to use a **config file** to store these settings in a project.  

#### Servers
`ListenAndServe` starts a HTTP server with a given address and handler. The handler in default would be `nil == DefaultServeMux`.  
`http.Handle()` and `http.HandleFunc()` add handlers to `DefaultServeMux`.  
More control over the server's behavior is available by creating a custom Server:  
```go
s := &http.Server{
	Addr:           ":8080",
	Handler:        myHandler,
	ReadTimeout:    10 * time.Second,
	WriteTimeout:   10 * time.Second,
	MaxHeaderBytes: 1 << 20,
}
log.Fatal(s.ListenAndServe())
```

### Constants
- HTTP Methods
- Status code
- MaxIdleConnsPerHost
- TimeFormat

## Q and A
Q: net.conn and net/http, what are the differences?
A: net.conn is usually for low-level networking protocol design while net/http provides higher-level of abstractions for handling HTTP communication.