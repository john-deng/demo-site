---
date: 2018-10-10T00:11:02+01:00
title: Web Application
menu:
  docs:
    parent: "Hiboot Cloud Native Application Framework"
    weight: 2
---

## Features

* *Web MVC (Model-View-Controller).
* Auto Configuration, pre-created instance with properties configs for dependency injection.
* Dependency injection with the struct tag \`inject:""\` or the constructor.
  
## Introduction to Hiboot MVC

Hiboot prefers to hide the business-independent code, so that the developers will concentrate the business logic.

unlike most of the Go web frameworks, Hiboot does not need to setup routes. Hiboot use reflection to construct the routes.

## Project structure

```bash
.
├── config
│   ├── application-gorm.yml
│   ├── application-local.yml
│   ├── application.yml
│   ├── i18n
│   │   ├── en-US.ini
│   │   └── zh-CN.ini
│   ├── keygen.sh
│   └── ssl
│       ├── app.rsa
│       └── app.rsa.pub
├── main.go
├── main_test.go
├── controller
│   ├── user.go
│   └── user_test.go
├── entity
│   ├── user.go
│   └── user_test.go
└── service
    ├── user.go
    └── user_test.go
```

Hiboot application include source code and config

## Application properties

Hiboot lets you externalize your configuration so that you can work with the same application code in different environments.

Property values can be injected directly into the struct using the \`value:""\` tag.

To provide a concrete example, suppose you develop a struct that uses a `app.name` property, as shown in the following example：

```go

type MyService struct {
	AppName string `value:"${app.name}"`
}

```

## Common application properties

Various properties can be specified inside your application.yml file. Hiboot loads properties form application.yml in config folder of working directory.

### config/application.yml

```yaml

app:
  project: examples  
  name: gorm-demo
  profiles:
    include:
    - actuator
    - locale
    - logging
    - gorm
logging:
  level: info

```

### application.yml field description

|Field|Description|Options|Example|
|---|---|---|---|
|app.project|project name|any string|examples|
|app.name|application name|any string|gorm-demo|
|app.profiles.active|application active profile|local,dev,test,staging,prod|dev|
|app.profiles.include|we use this field as starter switcher，⚠️ if the starter is imported in your source file but it's not included here, the starter will not be initialized| pakcage name of the starter|actuator, locale, logging, gorm|
|logging.level|set the logging level|debug,info,warn,error,fatal|info|

## Profile-specific properties

In addition to `application.yml` file, profile-specific properties can also be defined by using the following naming convention: `application-${profile}.yml`, active profiles can be set in `application.yml`.

Profile-specific properties are loaded from the same locations as standard `application.yml`, with profile-specific files always overriding the non-specific ones. `application-${app.profiles.active}.yml` is always be the highest priority.

### config/application-local.yml

```yaml

server:
  port: 8081
logging:
  level: debug

```

### application-local.yml field description

|Field|Description|Options|Example|
|---|---|---|---|
|server.port|service port that the application is listening on, it will overwrite the server.port in application.yml |any number|8081|
|logging.level|set logging level|debug,info,warn,error,fatal|info|

### config/application-gorm.yml

```yaml
gorm:
  type: mysql
  host: mysql-${app.profiles.active:dev}
  port: 3306
  database: ${app.name:test}
  username: demo
  password: fafUJCsVXf2Thj0d4n6UqNdX2PfI08fMyaNlrZhbJVVkghnZ+Zc/WCdITXflJpHZjYH5+LbLviy/6j9etPNwtdyAOXiqKI62itC6nDgp0Xlzu0qX8MwMIIAosUwaYpnflg23hRZnueKrq6SrEpkx4X+LWluDgHb2O5VfGGvHliE=
  charset: utf8
  parseTime: true
  loc: Asia/Shanghai
  config:
	decrypt: true
	
```

#### application-gorm.yml field description

|Field|Description|Options|Example|
|---|---|---|---|
|gorm.type|database type|mysql,postgres,sqlite3,mssql|mysql|
|gorm.host|database host, can be IP address or DNS name, `mysql-${app.profiles.active}` if environment variable `APP_PROFILES_ACTIVE`  is set to `local`，then gorm.host will be `mysql-local`）|`mysql-${app.profiles.active}`|
|gorm.port|database port|any port|3306|
|gorm.database|database name|string or variable|`${app.name:test}`|
|gorm.username|username for databse connection|valid username in string|dbuser|
|gorm.password|password for database connnection, if gorm.config.decrypt is true，then it will be the password that encrypted by [crypto](https://github.com/hidevopsio/crypto) |string|fafUJ ... O5VfGGvHliE=|
|gorm.charset|character set|utf8,ascii|utf8|
|gorm.parseTime|parse time or not|true,false|true|
|gorm.loc|timezone|see [world timezone](https://www.worldtimezone.com/index_cn.php)|Asia/Shanghai|
|gorm.config.decrypt|decrypt or not|true,false|true|

## Writing the Code

To write Hiboot application, as we know, the executable commands must always use package main, so we need to create the main package first.

See [Effective GO](https://golang.org/doc/effective_go.html#names) to learn more about Go's naming conventions.

### main.go

There are two parts inside the main package：imports and the entrance of the web application。

1. In order to decouple each packages, hiboot use registeration, auto configuration and dependency injection, as  we need to import some packages solely for their side effects, for exampole: `_ "hidevops.io/hiboot-data/examples/gorm/controller"`, if you used hiboot starter you may need import them in this way as well：`_ "hidevops.io/hiboot/pkg/starter/actuator"`, `_ "hidevops.io/hiboot/pkg/starter/locale"`, `_ "hidevops.io/hiboot/pkg/starter/logging"`

2. the function main is extremely simple, that is `web.NewApplication().Run()`, the package `web` is imported from `hidevops.io/hiboot/pkg/app/web`

```go

package main

import (
	_ "hidevops.io/hiboot-data/examples/gorm/controller"
	"hidevops.io/hiboot/pkg/app/web"
	_ "hidevops.io/hiboot/pkg/starter/actuator"
	_ "hidevops.io/hiboot/pkg/starter/locale"
	_ "hidevops.io/hiboot/pkg/starter/logging"
)

func main() {
	web.NewApplication().Run()
}

```

>Alternatively, you can set property `app.ProfilesInclude` in main function. In this approach, you don't have to import starter packages solely for their side effects.

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
		SetProperty(app.ProfilesInclude,
			actuator.Profile,
			logging.Profile).
		Run()
}

```

### Controller - controller/user.go

Now lets see how does the Hiboot controller works, the controller `userController` is embedded a struct `at.RestController`, it tells Hiboot that this is a web controller。

`newUserController` is the constructor of struct `userController`, the dependency `userService service.UserService` is injecting through the argument of the constructor  during the initialization of `newUserController`。

For the simplicity purpose, Hiboot does not need to config route, the method name is the route.

Here is the list of the HTTP methods,

|Method|Description|Options|Example|
|---|---|---|---|
|Get|GET|Get or GetById which in camel case|`func (c *userController) GetById(id unit64)` |
|Post|POST|Post PostUser which in camel case|`func (c *userController) Post(request *userRequest)`|
|Put|PUT|Put or PutUser which in camel case|`func (c *userController) Post(request *userRequest)`|
|Delete|DELETE|Delete DeleteById which in camel case|`func (c *userController) DeleteById(id unit64)` |

⚠️  Note that the constructor newUserController of the controller must be registered in the func `init()`.

```go

package controller

import (
	"hidevops.io/hiboot-data/examples/gorm/entity"
	"hidevops.io/hiboot-data/examples/gorm/service"
	"hidevops.io/hiboot/pkg/app"
	"hidevops.io/hiboot/pkg/app/web"
	"hidevops.io/hiboot/pkg/model"
	"net/http"
)

// RestController
type userController struct {
	at.RestController
	userService service.UserService
}

func init() {
	app.Register(newUserController)
}

// newUserController inject userService automatically
func newUserController(userService service.UserService) *userController {
	return &userController{
		userService: userService,
	}
}

// Post POST /user
func (c *userController) Post(request *entity.User) (model.Response, error) {
	err := c.userService.AddUser(request)
	response := new(model.BaseResponse)
	response.SetData(request)
	return response, err
}

// GetById GET /id/{id}
func (c *userController) GetById(id uint64) (response model.Response, err error) {
	user, err := c.userService.GetUser(id)
	response = new(model.BaseResponse)
	if err != nil {
		response.SetCode(http.StatusNotFound)
	} else {
		response.SetData(user)
	}
	return
}

// GetById GET /id/{id}
func (c *userController) GetAll() (response model.Response, err error) {
	users, err := c.userService.GetAll()
	response = new(model.BaseResponse)
	response.SetData(users)
	return
}

// DeleteById DELETE /id/{id}
func (c *userController) DeleteById(id uint64) (response model.Response, err error) {
	err = c.userService.DeleteUser(id)
	response = new(model.BaseResponse)
	return
}

```

### entity/user.go

User entity is the business model.

```go

type User struct {
	model.RequestBody
	Id       uint64 `json:"id"`
	Name     string `json:"name" validate:"required"`
	Username string `json:"username" validate:"required"`
	Password string `json:"password" validate:"required"`
	Email    string `json:"email" validate:"required,email"`
	Age      uint   `json:"age" validate:"gte=0,lte=130"`
	Gender   uint   `json:"gender" validate:"gte=0,lte=2"`
}

// Here we specify the table name as user instead of users by default.
func (u *User) TableName() string {
	return "user"
}


```

### Model -  service/user.go

Service implements the business logic. We declared an interface `UserService`，which includes method `AddUser`, `GetUser`, `GetAll`, and `DeleteUser`, userServiceImpl is the implementation of UserService。

By importing `hidevops.io/hiboot-data/starter/gorm`，the instance of `repository gorm.Repository` will be injectable, it will be injected to `userServiceImpl` through the constructor newUserService.

⚠️  Note that the constructor newUserService must be registered in the func `init()`.

```go

package service

import (
	"errors"
	"hidevops.io/hiboot-data/examples/gorm/entity"
	"hidevops.io/hiboot-data/starter/gorm"
	"hidevops.io/hiboot/pkg/app"
	"hidevops.io/hiboot/pkg/utils/idgen"
)

type UserService interface {
	AddUser(user *entity.User) (err error)
	GetUser(id uint64) (user *entity.User, err error)
	GetAll() (user *[]entity.User, err error)
	DeleteUser(id uint64) (err error)
}

type userServiceImpl struct {
	// add UserService, it means that the instance of UserServiceImpl can be found by UserService
	UserService
	repository gorm.Repository
}

func init() {
	// register UserServiceImpl
	app.Register(newUserService)
}

// will inject BoltRepository that configured in hidevops.io/hiboot/pkg/starter/data/bolt
func newUserService(repository gorm.Repository) UserService {
	repository.AutoMigrate(&entity.User{})
	return &userServiceImpl{
		repository: repository,
	}
}

func (s *userServiceImpl) AddUser(user *entity.User) (err error) {
	if user == nil {
		return errors.New("user is not allowed nil")
	}
	if user.Id == 0 {
		user.Id, _ = idgen.Next()
	}
	err = s.repository.Create(user).Error()
	return
}

func (s *userServiceImpl) GetUser(id uint64) (user *entity.User, err error) {
	user = &entity.User{}
	err = s.repository.Where("id = ?", id).First(user).Error()
	return
}

func (s *userServiceImpl) GetAll() (users *[]entity.User, err error) {
	users = &[]entity.User{}
	err = s.repository.Find(users).Error()
	return
}

func (s *userServiceImpl) DeleteUser(id uint64) (err error) {
	err = s.repository.Where("id = ?", id).Delete(entity.User{}).Error()
	return
}

```

### Run the sample code

```bash

go run main.go

```

Output:

```bash
______  ____________             _____
___  / / /__(_)__  /_______________  /_
__  /_/ /__  /__  __ \  __ \  __ \  __/
_  __  / _  / _  /_/ / /_/ / /_/ / /_     Hiboot Application Framework
/_/ /_/  /_/  /_.___/\____/\____/\__/     https://hidevops.io/hiboot
[INFO] 2018/10/23 23:37 Starting Hiboot web application gorm-demo on localhost with PID 28423
[INFO] 2018/10/23 23:37 Working directory: /Users/johnd/.gvm/pkgsets/go1.10/hidevops/src/hidevops.io/hiboot-data/examples/gorm
[INFO] 2018/10/23 23:37 The following profiles are active: local, [actuator locale logging gorm]
[INFO] 2018/10/23 23:37 Auto configure gorm starter
[INFO] 2018/10/23 23:37 Auto configure locale starter
[INFO] 2018/10/23 23:37 Auto configure logging starter
[INFO] 2018/10/23 23:37 The dependency graph resolved successfully
[INFO] 2018/10/23 23:37 connected to dataSource demo@mysql-local:3306/gorm_demo
[DBUG] 2018/10/23 23:36 GET: /health -> github.com/hidevops	io/hiboot-data/vendor/hidevops.io/hiboot/pkg/starter/actuator/controller/healthController.Get() and 2 more
[DBUG] 2018/10/23 23:36 DELETE: /user/id/{id} -> hidevops.io/hiboot-data/examples/gorm/controller/userController.DeleteById() and 2 more
[DBUG] 2018/10/23 23:36 GET: /user/id/{id} -> hidevops.io/hiboot-data/examples/gorm/controller/userController.GetById() and 2 more
[DBUG] 2018/10/23 23:36 GET: /user/all -> hidevops.io/hiboot-data/examples/gorm/controller/userController.GetAll() and 2 more
[DBUG] 2018/10/23 23:36 POST: /user -> hidevops.io/hiboot-data/examples/gorm/controller/userController.Post() and 2 more
Now listening on: http://localhost:8080
Application started. Press CMD+C to shut down.
```

### Make a request

Let's make a request on the RESTful API GET /user/all

```bash
http GET localhost:8080/user/all
```

```bash

HTTP/1.1 200 OK
Content-Length: 307
Content-Type: application/json; charset=UTF-8
Date: Tue, 23 Oct 2018 15:38:41 GMT
Set-Cookie: app.language=; Path=/; Expires=Tue, 23 Oct 2018 17:38:41 GMT; Max-Age=7200; HttpOnly
{
    "code": 200,
    "data": [
        {
            "age": 18,
            "email": "john.doe@gmail.com",
            "gender": 0,
            "id": 209536579658580081,
            "name": "John Doe",
            "password": "poi321",
            "username": "johnd"
        },
        {
            "age": 25,
            "email": "mike.phil@gmail.com",
            "gender": 0,
            "id": 209536656246571121,
            "name": "Mike Phil",
            "password": "iutg039",
            "username": "mikep"
        }
    ],
    "message": "Success"
}

```

## Unit test

As I always say, Hiboot is built for production ready, we care about the quality so much, here is the realtime code coverage [![codecov](https://codecov.io/gh/hidevopsio/hiboot/branch/master/graph/badge.svg)](https://codecov.io/gh/hidevopsio/hiboot). So how do we do the unit testing?

First, let's see the unit test for main.go

### mian_test.go

As shown below, the unit test for func main，we have `go main()` inside the test case `TestRunMain` just for the sake of main func testing。

```go

package main

import (
	"testing"
	"time"
)

func TestRunMain(t *testing.T) {
	go main()
	time.Sleep(200 * time.Millisecond)
}

```

### Mock up the testing - controller/user_test.go

We want to test the `userController`, but the userController depends on `userSerivce` which will connect to the real database through gorm repository. In order to do so, we must mock the `userService`.

We use [Mockery](https://github.com/vektra/mockery) to generate the some of the test code, [Mockery](https://github.com/vektra/mockery) is a tool that automatically generates mock implementations of interfaces `UserService`.

First, install mockery,

```bash

go get github.com/vektra/mockery/.../

```

Then, generate mocks for the interface

```bash
# go to the directory where the UserService is.
cd $GOPATH/src/hidevops.io/hiboot-data/examples/gorm/service

# generate mocks for the interface UserService
mockery -name UserService

```

After that, you will see mocks/UserService is generated under $GOPATH/src/hidevops.io/hiboot-data/examples/gorm/service.

```go

// Code generated by mockery v1.0.0. DO NOT EDIT.

package mocks

import entity "hidevops.io/hiboot-data/examples/gorm/entity"
import mock "github.com/stretchr/testify/mock"

// UserService is an autogenerated mock type for the UserService type
type UserService struct {
	mock.Mock
}

// AddUser provides a mock function with given fields: user
func (_m *UserService) AddUser(user *entity.User) error {
	ret := _m.Called(user)

	var r0 error
	if rf, ok := ret.Get(0).(func(*entity.User) error); ok {
		r0 = rf(user)
	} else {
		r0 = ret.Error(0)
	}

	return r0
}

// DeleteUser provides a mock function with given fields: id
func (_m *UserService) DeleteUser(id uint64) error {
	ret := _m.Called(id)

	var r0 error
	if rf, ok := ret.Get(0).(func(uint64) error); ok {
		r0 = rf(id)
	} else {
		r0 = ret.Error(0)
	}

	return r0
}

// GetAll provides a mock function with given fields:
func (_m *UserService) GetAll() (*[]entity.User, error) {
	ret := _m.Called()

	var r0 *[]entity.User
	if rf, ok := ret.Get(0).(func() *[]entity.User); ok {
		r0 = rf()
	} else {
		if ret.Get(0) != nil {
			r0 = ret.Get(0).(*[]entity.User)
		}
	}

	var r1 error
	if rf, ok := ret.Get(1).(func() error); ok {
		r1 = rf()
	} else {
		r1 = ret.Error(1)
	}

	return r0, r1
}

// GetUser provides a mock function with given fields: id
func (_m *UserService) GetUser(id uint64) (*entity.User, error) {
	ret := _m.Called(id)

	var r0 *entity.User
	if rf, ok := ret.Get(0).(func(uint64) *entity.User); ok {
		r0 = rf(id)
	} else {
		if ret.Get(0) != nil {
			r0 = ret.Get(0).(*entity.User)
		}
	}

	var r1 error
	if rf, ok := ret.Get(1).(func(uint64) error); ok {
		r1 = rf(id)
	} else {
		r1 = ret.Error(1)
	}

	return r0, r1
}


```

### Writing the unit test cases

Below are the unit test cases, we tested POST, GET, DELETE method to user controller with the mock interface mocks.UserService.

```go

package controller

import (
	"hidevops.io/hiboot-data/examples/gorm/entity"
	"hidevops.io/hiboot/pkg/app/web"
	"hidevops.io/hiboot/pkg/log"
	"hidevops.io/hiboot/pkg/utils/idgen"
	"github.com/stretchr/testify/assert"
	"net/http"
	"testing"
	"hidevops.io/hiboot-data/examples/gorm/service/mocks"
	"errors"
)

func init() {
	log.SetLevel(log.DebugLevel)
}

func TestCrdRequest(t *testing.T) {

	mockUserService := new(mocks.UserService)
	userController := newUserController(mockUserService)
	testApp := web.RunTestApplication(t, userController)

	id, err := idgen.Next()
	assert.Equal(t, nil, err)

	testUser := &entity.User{
		Id:       id,
		Name:     "Bill Gates",
		Username: "billg",
		Password: "3948tdaD",
		Email:    "bill.gates@microsoft.com",
		Age:      60,
		Gender:   1,
	}

	// first, call mocks.UserService.AddUser
	mockUserService.On("AddUser", testUser).Return(nil)
	// then run the test that will call UserService.AddUser
	t.Run("should add user with POST request", func(t *testing.T) {
		// First, let's Post User
		testApp.Post("/user").
			WithJSON(testUser).
			Expect().Status(http.StatusOK)
	})

	mockUserService.On("GetUser", id).Return(testUser, nil)
	t.Run("should get user with GET request", func(t *testing.T) {
		// Then Get User
		// e.g. GET /user/id/123456
		testApp.Get("/user/id/{id}").
			WithPath("id", id).
			Expect().Status(http.StatusOK)
	})

	mockUserService.On("GetAll").Return(&[]entity.User{*testUser}, nil)
	t.Run("should get user with GET request", func(t *testing.T) {
		// Then Get User
		// e.g. GET /user/id/123456
		testApp.Get("/user/all").
			Expect().Status(http.StatusOK)
	})

	// assert that the expectations were met
	mockUserService.AssertExpectations(t)

	unknownId, err := idgen.Next()
	assert.Equal(t, nil, err)
	mockUserService.On("GetUser", unknownId).Return((*entity.User)(nil), errors.New("not found"))

	t.Run("should return 404 if trying to find a record that does not exist", func(t *testing.T) {
		// Then Get User
		testApp.Get("/user/id/{id}").
			WithPath("id", unknownId).
			Expect().Status(http.StatusNotFound)
	})

	// assert that the expectations were met
	mockUserService.AssertExpectations(t)

	mockUserService.On("DeleteUser", id).Return(nil)
	t.Run("should delete the record with DELETE request", func(t *testing.T) {
		// Finally Delete User
		testApp.Delete("/user/id/{id}").
			WithPath("id", id).
			Expect().Status(http.StatusOK)
	})
}

```

### Finally, Let run above unit tests, the test result is positive.

![unit-test](/images/web-app/unit-test.png)