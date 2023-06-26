## Package
### Create and import your own _local_ packages
1. 在`%GOPATH%/src/<your-project-name>`下创建包：`%GOPATH%/src/<your-project-name>/<your-package-name>`。
2. 在`%GOPATH%/src/<your-project-name>/<your-package-name>`路径下,执行`go mod init github/<your-repo>/<your-package-name>`，当前路径下会创建`go.mod`。
3. 进入`%GOPATH%/src/<your-project-name>/<your-main-part>`，执行`go mod init <your-main-part>`，同样，当前路径下会创建`go.mod`。
4. 手动修改`%GOPATH%/src/<your-project-name>/<your-main-part>/go.mod`，内容如下：
```go
module helloworld

require github.com/<your-project-name>/<your-package-name> v0.0.0 //获取对应版本的package
replace github.com/<your-project-name>/<your-package-name> => ../geometry //将从远程获取package转为从本地获取package，通常用于本地开发或者测试

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