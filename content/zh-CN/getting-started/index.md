---
date: 2018-10-10T00:11:02+01:00
title: 入门
menu:
  docs:
    parent: "Hiboot云原生应用框架"
    weight: 1
    title: "入门"
---

## 前提

我们假设你以及安装了Go语言的开发环境。当然，如果你没安装，则按照[GO语言环境搭建详解](https://hidevops.io/blog/go-env/)来安装。

## 网络应用快速入门

我们将在本章节演示如何快速入门Hiboot网络应用，即通过简单三个步骤来创建一个最简单的Hiboot应用。闲话不多说，我们开始吧！

### 获取代码

首先到github拉取演示代码，在命令行终端运行以下命令即可：

```bash
go get -u hidevops.io/hiboot-web-app-demo

cd $GOPATH/src/hidevops.io/hiboot-web-app-demo

```

查看历史记录

```bash
git log
```

结果如下：

```bash
commit 2be5df8e0101b685579f1dd452059d967017148f
Author: John Deng <john.deng@qq.com>
Date:   Sat Nov 3 07:48:46 2018 +0800

    Step 3, adding Hiboot controller

commit 063c72282edff85db20d4046f5801d7f5fcf4dbb
Author: John Deng <john.deng@qq.com>
Date:   Sat Nov 3 07:45:15 2018 +0800

    Step 2, adding some starters

commit d9213107f6692bc22c3d79cd445567730b962477
Author: John Deng <john.deng@qq.com>
Date:   Sat Nov 3 07:38:59 2018 +0800

    step 1: Writing the first Hiboot web application

```

## - 第一步, 快速开始Hiboot应用

编写Hiboot应用，和其它Go语言应用一样，必须有一个main包，里面包含一个main函数。

如果你对Go语言还不熟悉，建议阅读[Effective GO](https://golang.org/doc/effective_go.html#names)或中文版[实效Go编程](https://go-zh.org/doc/effective_go.html)

```bash
git reset --hard d9213107f6692bc22c3d79cd445567730b962477
```

代码展示如下，这个代码已经可以执行的了，只是没有任何业务逻辑。

```go

package main

import (
	"hidevops.io/hiboot/pkg/app/web"
)

func main()  {
	web.NewApplication().Run()
}
```

## - 第二步, 添加 Starter

Here we are going to add starter [actuator](https://hidevops.io/hiboot/pkg/starter/actuator) and [logging](https://hidevops.io/hiboot/pkg/starter/logging).

```bash
git reset --hard 063c72282edff85db20d4046f5801d7f5fcf4dbb
```

添加 actuator 和 logging 两个 starter

```go
package main

import (
	"hidevops.io/hiboot/pkg/app/web"
	"hidevops.io/hiboot/pkg/app"
	"hidevops.io/hiboot/pkg/starter/actuator"
	"hidevops.io/hiboot/pkg/starter/logging"
)

func main()  {
	web.NewApplication().
		SetProperty(app.ProfilesInclude, actuator.Profile, logging.Profile).
		Run()
}
```

这时候，我们来运行这段代码

```bash
go run main.go
```

输出结果如下，这时你可以看到已经映射了RESTful接口 `/health`, 也就是说，添加actuator starter后，即提供了健康检测接口。

```bash
______  ____________             _____
___  / / /__(_)__  /_______________  /_
__  /_/ /__  /__  __ \  __ \  __ \  __/   
_  __  / _  / _  /_/ / /_/ / /_/ / /_     Hiboot Application Framework
/_/ /_/  /_/  /_.___/\____/\____/\__/     https://hidevops.io/hiboot

[INFO] 2018/11/01 15:51 Starting Hiboot web application hiboot-app on localhost with PID 71924
[INFO] 2018/11/01 15:51 Working directory: /Users/johnd/.gvm/pkgsets/go1.10/hidevops/src/hidevops.io/hiboot-web-app-demo
[INFO] 2018/11/01 15:51 The following profiles are active: local, [actuator logging web]
[INFO] 2018/11/01 15:51 Initializing Hiboot Application
[INFO] 2018/11/01 15:51 Auto configure web starter
[INFO] 2018/11/01 15:51 Auto configure actuator starter
[INFO] 2018/11/01 15:51 Auto configure logging starter
[INFO] 2018/11/01 15:51 Resolving dependencies
[INFO] 2018/11/01 15:51 Injecting dependencies
[INFO] 2018/11/01 15:51 Injected dependencies
[INFO] 2018/11/01 15:51 Mapped "/health" onto actuator.healthController.Get()
[INFO] 2018/11/01 15:51 Hiboot started on port(s) http://localhost:8080
[INFO] 2018/11/01 15:51 Started hiboot-app in 0.004189 seconds
```

健康检测对于生产来说非常重要，接下来我们使用[httpie](https://httpie.org/)(当然你用curl、postman或浏览器都可以)来验证健康检测接口。

```bash
http http://localhost:8080/health
```

输出健康状态如下：

```bash
HTTP/1.1 200 OK
Content-Length: 15
Content-Type: application/json; charset=UTF-8
Date: Thu, 01 Nov 2018 07:55:33 GMT

{
    "status": "UP"
}
```

## - 第三步, 添加 Controller

在这个步骤，我们来增加一个RestController，名字就叫 `Controller`，这个结构体内嵌了`at.RestController`, 它的类型是`interface`，`at.RestController`在Hiboot中为注解作用，表示这是一个RestController。

你可以编写以下代码，也可以运行以下命令直接获取。

```bash
git reset --hard 2be5df8e0101b685579f1dd452059d967017148f
```

### 编写代码

通过以上三步，你可以看到如何快速来编写第一个Hiboot网络应用程序

```go

package main

import (
	"hidevops.io/hiboot/pkg/app/web"
	"hidevops.io/hiboot/pkg/app"
	"hidevops.io/hiboot/pkg/starter/actuator"
	"hidevops.io/hiboot/pkg/starter/logging"
	"hidevops.io/hiboot/pkg/at"
)

// Controller Rest Controller with path /
// RESTful Controller, derived from at.RestController. The context mapping of this controller is '/' by default
type Controller struct {
	// at.RestController or at.RestController must be embedded here
	at.RestController
}

// Get GET /
// Get method, the context mapping of this method is '/' by default
// the Method name Get means that the http request method is GET
func (c *Controller) Get() string {
	// response
	return "My first Hiboot web application"
}

func main()  {
	web.NewApplication(new(Controller)).
		SetProperty(app.ProfilesInclude, actuator.Profile, logging.Profile).
		Run()
}

```

### 运行代码：

```bash
go run main.go
```

你可以看到多了一个接口映射：`/` 到 `main.Controller.Get()`

```go
______  ____________             _____
___  / / /__(_)__  /_______________  /_
__  /_/ /__  /__  __ \  __ \  __ \  __/   
_  __  / _  / _  /_/ / /_/ / /_/ / /_     Hiboot Application Framework
/_/ /_/  /_/  /_.___/\____/\____/\__/     https://hidevops.io/hiboot

[INFO] 2018/11/01 16:07 Starting Hiboot web application hiboot-app on localhost with PID 72490
[INFO] 2018/11/01 16:07 Working directory: /Users/johnd/.gvm/pkgsets/go1.10/hidevops/src/hidevops.io/hiboot-web-app-demo
[INFO] 2018/11/01 16:07 The following profiles are active: local, [actuator logging web]
[INFO] 2018/11/01 16:07 Initializing Hiboot Application
[INFO] 2018/11/01 16:07 Auto configure web starter
[INFO] 2018/11/01 16:07 Auto configure actuator starter
[INFO] 2018/11/01 16:07 Auto configure logging starter
[INFO] 2018/11/01 16:07 Resolving dependencies
[INFO] 2018/11/01 16:07 Injecting dependencies
[INFO] 2018/11/01 16:07 Injected dependencies
[INFO] 2018/11/01 16:07 Mapped "/health" onto actuator.healthController.Get()
[INFO] 2018/11/01 16:07 Mapped "/" onto main.Controller.Get()
[INFO] 2018/11/01 16:07 Hiboot started on port(s) http://localhost:8080
[INFO] 2018/11/01 16:07 Started hiboot-app in 0.002689 seconds

```

### 验证接口

现在我们来验证刚完成的第一个Hiboot网络应用接口：

```bash
http http://localhost:8080/
```

输出结果如下:

```bash
My first Hiboot web application
```

## 命令行应用快速入门

编写命令行应用和网络应用一样简单，也可以使用Hiboot的依赖注入和自动配置功能。

以下代码可以在[这里](https://hidevops.io/hiboot/examples/cli/hello)找到.

```go

// import cli starter and fmt
import (
	"fmt"
	"hidevops.io/hiboot/pkg/app"
	"hidevops.io/hiboot/pkg/app/cli"
)

// define the command
type rootCommand struct {
	// embedding cli.RootCommand in each command
	cli.RootCommand

	// persistant flag to
	to string
}

func newRootCommand() *rootCommand {
	c := new(rootCommand)
	c.Use = "hello"
	c.Short = "hello command"
	c.Long = "run hello command for getting started"
	c.Example = `
hello -h : help
hello -t John : say hello to John
`
	c.PersistentFlags().StringVarP(&c.to, "to", "t", "world", "e.g. --to=world or -t world")
	return c
}

// Run run the command
func (c *rootCommand) Run(args []string) error {
	fmt.Printf("Hello, %v\n", c.to)
	return nil
}

// main function
func main() {
	// create new cli application and run it
	cli.NewApplication(newRootCommand).
		SetProperty(app.PropertyBannerDisabled, true).
		Run()
}

```

### 运行命令行应用

```bash
dep ensure

go run main.go
```

```bash
Hello, world
```

### 编译及运行

```bash
go build
```

Let's get help

```bash
./hello --help
```

```bash
run hello command for getting started

Usage:
  hello [flags]

Flags:
  -h, --help        help for hello
  -t, --to string   e.g. --to=world or -t world (default "world")

```

Greeting to Hiboot

```bash
./hello --to Hiboot
```

```bash
Hello, Hiboot
```

到目前为止，我们有了一定基础来编写Hiboot的[网络应用](/cn/web-app/)和[命令行应用](/cn/cli-app/)。