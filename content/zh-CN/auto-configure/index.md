---
title: "自动配置"
date: 2018-10-22T16:22:23+08:00
tags: ["auto-configuration", "starter", "go"]
menu:
  docs:
    parent: "Hiboot云原生应用框架"
    weight: 5
    title: "自动配置"
---

## 自动配置 Starter

Hiboot auto-configuration attempts to automatically configure your Hiboot application based on the pkg dependencies that you have added.
For example, if bolt is imported in you main.go, and you have not manually configured any database connection,
then Hiboot auto-configures an database bolt for any service to inject.

You need to opt-in to auto-configuration by embedding app.Configuration in your configuration and
calling the app.Register() function inside the init() function of your configuration pkg.

For more details, see https://godoc.org/hidevops.io/hiboot/pkg/starter

## 创建你自己的 Starter

A full Hiboot starter for a library may contain the following structs:
	autoconfigure - object that handle the auto-configuration code.
	properties - object that contains properties which will be injected configurable default values or user specified values
If you work in a company that develops shared go packages, or if you work on an open-source or commercial project,
you might want to develop your own auto-configured starter. starter can be implemented in external packages and
can be imported by any go applications.

Understanding Auto-configured Starter

Under the hood, auto-configuration is implemented with standard struct. Additional embedded field app.Configuration.
AutoConfiguration used to constrain when the auto-configuration should apply. Usually, auto-configuration struct use
`after:"fooConfiguration"` or `missing:"fooConfiguration"` tags. This ensures that auto-configuration applies only
when relevant configuration are found and when you have not declared your own configuration.

## 代码示例

This example shows the guide to make customized auto configuration
for more details, see https://hidevops.io/hiboot-data/blob/master/starter/bolt/autoconfigure.go

```go
package bolt

import (
	"hidevops.io/hiboot/pkg/app"
	"os"
)

// properties
type properties struct {
	Database string      `json:"database" default:"hiboot.db"`
	Mode     os.FileMode `json:"mode" default:"0600"`
	Timeout  int64       `json:"timeout" default:"2"`
}

// declare boltConfiguration
type boltConfiguration struct {
	app.Configuration
	// the properties member name must be Bolt if the mapstructure is bolt,
	// so that the reference can be parsed
	BoltProperties properties `mapstructure:"bolt"`
}

// BoltRepository
type BoltRepository struct {
}

func init() {
	// register newBoltConfiguration as AutoConfiguration
	app.Register(newBoltConfiguration)
}

// boltConfiguration constructor
func newBoltConfiguration() *boltConfiguration {
	return &boltConfiguration{}
}

func (c *boltConfiguration) BoltRepository() *BoltRepository {

	repo := &BoltRepository{}

	// config repo according to c.BoltProperties

	return repo
}

```