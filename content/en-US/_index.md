---
date: 2018-10-08T21:07:13+01:00
title: Hiboot Cloud Native Application Framework
type: index
tags: ["hiboot", "framework"]
weight: 1
---

{{<badges>}}

## About 

Hiboot is a high performance web and cli application framework with dependency injection

Hiboot is not trying to reinvent everything, it integrates the popular libraries but make them simpler, easier to use. It borrowed some of the Spring features like dependency injection, aspect oriented programming, and auto configuration. You can integrate any other libraries easily by auto configuration with dependency injection support.

## Overview

* Web MVC (Model-View-Controller).
* Auto Configuration, pre-create instance with properties configs for dependency injection.
* Dependency injection with struct tag name \`inject:""\` or Constructor func.

## Features


* **Apps**
    * cli - command line application
    * web - web application

* **Starters**
    * actuator - health check
    * locale - locale starter
    * logging - customized logging settings
    * jaeger - open tracing starter jaeger
    * jwt - jwt starter
    * grpc - grpc application starter
    * websocket - websocket two-way interactive communication

* **Tags**
    * inject - inject generic instance into object
    * default - inject default value into struct object 
    * value - inject string value or references / variables into struct string field

* **Utils** 
    * cmap - concurrent map
    * copier - copy between struct
    * crypto - aes, base64, md5, and rsa encryption / decryption
    * gotest - go test util
    * idgen - twitter snowflake id generator
    * io - file io util
    * mapstruct - convert map to struct
    * replacer - replacing stuct field value with references or environment variables
    * sort - sort slice elements
    * str - string util enhancement util
    * validator - struct field validation