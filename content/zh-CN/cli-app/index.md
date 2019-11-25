---
date: 2018-10-10T00:11:02+01:00
title: 命令行应用
tags: ["cli", "web", "application"]
menu:
  docs:
    parent: "Hiboot云原生应用框架"
    weight: 3
    title: "命令行应用"
---

## 关于

Hiboot命令行应用是在[cobra](https://github.com/spf13/cobra)的基础上构建的。得益于Hiboot提供的依赖注入和自动配置功能，开发者将会更高效的开发出命令行应用。

## 功能列表

* 使用依赖注入来注入子命令
* 子命令处理方法

## 项目结构

```bash
.
├── cmd
│   ├── bar.go
│   ├── foo.go
│   ├── root.go
│   ├── root_test.go
│   └── second.go
├── config
│   ├── autoconfigure.go
│   └── autoconfigure_test.go
├── main.go
├── main_test.go
└── model
    ├── foo.go
    └── foo_test.go

```

就如Hiboot网络应用，Hiboot命令行应用的宗旨是让开发更简单、更容易。

```go

package main

import (
	"hidevops.io/hiboot/examples/cli/advanced/cmd"
	"hidevops.io/hiboot/examples/cli/advanced/config"
	"hidevops.io/hiboot/pkg/app"
	"hidevops.io/hiboot/pkg/app/cli"
)

func main() {
	// create new cli application and run it
	cli.NewApplication(cmd.NewRootCommand).
		SetProperty(app.ProfilesInclude, config.Profile).
		Run()
}

```

## 根命令

每个命令行应用都会有唯一的根命令。通过在根命令的结构体内嵌`cli.RootCommand`来标识。

命令的处理方法为`Run(args []string) error`

```go

package cmd

import (
	"hidevops.io/hiboot/pkg/app/cli"
	"hidevops.io/hiboot/pkg/log"
)

// HelloCommand is the root command
type HelloCommand struct {
	cli.RootCommand

	profile string
	timeout int
}

// HelloCommand the root command
func NewHelloCommand(second *secondCommand) *HelloCommand {
	c := new(RootCommand)
	c.Use = "hello"
	c.Short = "hello command"
	c.Long = "Run hello command"
	c.ValidArgs = []string{"baz"}
	pf := c.PersistentFlags()
	pf.StringVarP(&c.profile, "profile", "p", "dev", "e.g. --profile=test")
	pf.IntVarP(&c.timeout, "timeout", "t", 1, "e.g. --timeout=1")
	c.Add(second)
	return c
}

// Run root command handler
func (c *HelloCommand) Run(args []string) error {
	log.Infof("handle first command: profile=%v, timeout=%v", c.profile, c.timeout)
	return nil
}

```

## 子命令

所有的子命令在使用`app.Register()`注册后即可注入到任何上级命令。这里需要提醒的是注意依赖关系，不要循环依赖。

子命令通过在子命令的结构体内嵌`cli.SubCommand`来标识。

```go


package cmd

import (
	"hidevops.io/hiboot/pkg/app"
	"hidevops.io/hiboot/pkg/app/cli"
	"hidevops.io/hiboot/pkg/log"
)

type secondCommand struct {
	cli.SubCommand
}

func init() {
	app.Register(newSecondCommand)
}

func newSecondCommand(foo *fooCommand, bar *barCommand) *secondCommand {
	c := new(secondCommand)
	c.Use = "second"
	c.Short = "second command"
	c.Long = "Run second command"
	c.Add(foo, bar)
	return c
}

func (c *secondCommand) Run(args []string) error {
	log.Info("handle second command")
	return nil
}


```
