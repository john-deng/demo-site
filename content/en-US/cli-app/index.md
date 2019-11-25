---
date: 2018-10-10T00:11:02+01:00
title: Cli Applications
tags: ["cli", "application"]
menu:
  docs:
    parent: "Hiboot Cloud Native Application Framework"
    weight: 3
    title: "Cli Applications"
---

## About Hiboot cli application

Hiboot cli application is built based on top of [cobra](https://github.com/spf13/cobra) with Hiboot dependency injection and auto configuration.

## Features

* Dependency Injection
* Sub command handler

## Cli application structure

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

Just like web application, Hiboot cli application is designed to be simpler and easier.

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

## Root Command

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

## Sub Command

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