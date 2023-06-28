## Package
### Create and import your own _local_ packages
1. 在`%GOPATH%/src/<your-project-name>`下创建包：`%GOPATH%/src/<your-project-name>/<your-package-name>`。
2. 在`%GOPATH%/src/<your-project-name>/<your-package-name>`路径下,执行`go mod init github/<your-repo>/<your-package-name>`，当前路径下会创建`go.mod`。
3. 进入`%GOPATH%/src/<your-project-name>/<your-main-part>`，执行`go mod init <your-main-part>`，同样，当前路径下会创建`go.mod`。
4. 手动修改`%GOPATH%/src/<your-project-name>/<your-main-part>/go.mod`，内容如下：
```go
module helloworld

require github.com/<your-project-name>/<your-package-name> v0.0.0 //获取对应版本的package
replace github.com/<your-project-name>/<your-package-name> => ../<your-pacakge-name> //将从远程获取package转为从本地获取package，通常用于本地开发或者测试

go 1.20 //基于自己版本

```
5. 在你的主程序`%GOPATH%/src/<your-project-name>/<your-main-part>/main.go`中导入包：`import "github/<your-repo>/<your-package-name>"`
6. 执行`go mod tidy`

The Project's file structure would look like this:  
![FILE STRUCTURE](https://cdn.jsdelivr.net/gh/PsyLinkist/LearningBlogPics/202306261715199.png)

**Tips**: The module path `github/<your-repo>/<your-package-name>` does not necessarily have to correspond to an actual remote ropository. This form is just for convenience.

## Map
### 空映射与nil映射
使用`make()`创建空映射`var someMap := make(map[string]int)`，此时可以直接添加项：`someMap["someone"] = 21`；但如果直接创建nil映射：`var someMap map[string]int`，则会在添加新项时触发`panic`。

## Method & Function
### 如何在各种数据类型上使用方法（method）
可以通过`type`定义数据类型别名，然后对别名创建方法：  
```go
package main

import (
    "fmt"
    "strings"
)

type upperstring string //别名

func (s upperstring) Upper() string { //方法
    return strings.ToUpper(string(s))
}

func main() {
    s := upperstring("Learning Go!")
    fmt.Println(s)
    fmt.Println(s.Upper())
}

// 输出
// Learning Go!
// LEARNING GO!
```

## Interface
### 接口的意义
- 可以使用一个接口控制多种数据类型。例如创建一个函数，接口类型作为参数，则所有能实现该接口的接收者都可以作为参数传入该函数：
```go
type Tech struct {
	Name string
	Attack int
	Effect string
}

type Domain interface {
	DomainExpansion() string
}

func DomainExpansion(rcvr Domain) {
	rcvr.DomainExpansion()
}

func (t Tech)DomainExpansion {
	fmt.Println(t.Effect)
}


```

### 接口的限制

### String()
是`Go`内置的一种特殊方法，用于输出目标对象的字符串表征。使用`fmt`包输出时会自动调用。因此：
```go
package main

import "fmt"

type Person struct {
    Name, Country string
}

func (p Person) String() string {
    return fmt.Sprintf("%v is from %v", p.Name, p.Country)
}
func main() {
    rs := Person{"John Doe", "USA"}
    ab := Person{"Mark Collins", "United Kingdom"}
    fmt.Printf("%s\n%s\n", rs, ab)
}

// output
// John Doe is from USA
// Mark Collins is from United Kingdom
```

## goroutine
### How does it work?
```go
package main

import (
	"fmt"
	"net/http"
	"time"
)

func checkAPI(api string) {
	_, err := http.Get(api)
	if err != nil {
		fmt.Printf("ERROR: %s is down!\n", api)
		return
	}
	fmt.Printf("SUCCESS: %s is up and running!\n", api)
}

func main() {
	start := time.Now()

	apis := []string{
		"https://management.azure.com",
		"https://dev.azure.com",
		"https://api.github.com",
		"https://outlook.office.com/",
		"https://api.somewhereintheinternet.com/",
		"https://graph.microsoft.com",
	}

	for _, api := range apis {
		go checkAPI(api)
	}

	//time.Sleep(3 * time.Second)
	elapsed := time.Since(start)
	fmt.Printf("Done! It took %v seconds!", elapsed.Seconds())
}

//output
//Done! It took %v seconds!
```
Q：程序为什么能够在`checkAPI`没有完成的情况下结束？
A：使用`go`关键字产生并发的goroutine来执行`checkAPI`函数，这些goroutine不会阻塞主线程的执行。而主线程结束后，未完成的goroutine也会随之终止。

## Channel
### Unbuffered channel
- A capacity of zero.
- When sending an value to a nonbuffered channel(`ch <- value`), the sender will block until another goroutine receives the value from the channel(`<- ch`).
- Vice versa.
- Synchronous communication

### Buffered channel
- A capacity of greater-than-zero.
- The sender blocks only if the channel is full.
- Vice versa.
- Asynchronous communication

### Select
```
SelectStmt = "select" "{" { CommClause } "}" .
CommClause = CommCase ":" StatementList .
CommCase   = "case" ( SendStmt | RecvStmt ) | "default" .
RecvStmt   = [ ExpressionList "=" | IdentifierList ":=" ] RecvExpr .
RecvExpr   = Expression .
```
- `SendStmt | RecvStmt` will execute if executable. There is an instance:
```go
var a []int
var c, c1, c2, c3, c4 chan int
var i1, i2 int
select {
case i1 = <-c1:
	print("received ", i1, " from c1\n")
case c2 <- i2:
	print("sent ", i2, " to c2\n")
case i3, ok := (<-c3):  // same as: i3, ok := <-c3
	if ok {
		print("received ", i3, " from c3\n")
	} else {
		print("c3 is closed\n")
	}
case a[f()] = <-c4:
	// same as:
	// case t := <-c4
	//	a[f()] = t
default:
	print("no communication\n")
}
``` 

## struct
### 结构体的嵌入特性
- 结构体可以嵌入到另一个结构体内，嵌入的结构体的字段(`field`)，另一个结构体可以直接使用：
```go
type Person struct {
	ID int
	Age int
	Name string
}

type Gojo struct {
	Person
	Tech string
}

GojoSatoru := Gojo {
	Person: Person {
		ID: 0,
		Age: 28,
		Name: "Gojo Satoru",
	},
	Tech "Infinite void",
}

fmt.Println(GojoSatoru.Name) // output: "Gojo Satoru"
```