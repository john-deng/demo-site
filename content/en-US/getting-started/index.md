---
date: 2018-10-10T00:11:02+01:00
title: Getting started
menu:
  docs:
    parent: "Hiboot Cloud Native Application Framework"
    weight: 1
---

## Quick start web application

This section will show you how to create and run a simple hiboot web application in 3 steps. Let’s get started!

### Get the source code

```bash

go get -u hidevops.io/hiboot-web-app-demo

cd $GOPATH/src/hidevops.io/hiboot-web-app-demo

git log

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

## - Step 1, writing the first Hiboot web application

To write Hiboot application, as we know, the executable commands must always use package main, so we need to create the main package first.

See [Effective GO](https://golang.org/doc/effective_go.html#names) to learn more about Go's naming conventions.

```bash
git reset --hard d9213107f6692bc22c3d79cd445567730b962477
```

## - Step 2, adding some starters

Here we are going to add starter [actuator](https://hidevops.io/hiboot/pkg/starter/actuator) and [logging](https://hidevops.io/hiboot/pkg/starter/logging).

```bash
git reset --hard 063c72282edff85db20d4046f5801d7f5fcf4dbb
```

## - Step 3, adding Hiboot controller

```bash
git reset --hard 2be5df8e0101b685579f1dd452059d967017148f
```

### Writing the code

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

#### Run web application

```bash
dep ensure

go run main.go
```

#### Testing the API by curl

```bash
curl http://localhost:8080/
```

Output:

```bash
My first Hiboot web application
```

## Quick start cli application

Writing Hiboot cli application is as simple as web application, you can take the advantage of dependency injection introduced by Hiboot.

```go

// import cli starter and fmt
import (
	"fmt"
	"hidevops.io/hiboot/pkg/app"
	"hidevops.io/hiboot/pkg/app/cli"
)

// define the command
type rootCommand struct {
	// embedding cli.RootCommand
	cli.RootCommand
	To string
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
	c.PersistentFlags().StringVarP(&c.To, "to", "t", "world", "e.g. --to=world or -t world")
	return c
}

// Run run the command
func (c *rootCommand) Run(args []string) error {
	fmt.Printf("Hello, %v\n", c.To)
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

### Run cli application

```bash
dep ensure

go run main.go
```

```bash
Hello, world
```

### Build the cli application and run

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
