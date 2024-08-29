# 调用微服务

在web服务中调用微服务的模式，web服务本身也可以是一个微服务。这样就可以与其它微服务之间进行调用。

如果想在web应用的controller中调用微服务的方法，需要在web服务启动前进行服务注册，采用`AddWeaverService`方法。

如：

```go

type server struct {
	weaver.Implements[weaver.Main]
	ServiceDict   weaver.Ref[dictSrv.T]
	
	address weaver.Listener
}

func RunServe(ctx context.Context, s *server) error {

    srv := nexframe.Server()
	
    // 自动注册 ServiceWeaver 服务
    err := srv.AddWeaverService(s)
    if err != nil {
         return fmt.Errorf("failed to add ServiceWeaver services: %v", err)
    }

    // 注册控制器
    srv.RegisterController("/",
        &books.BookController,
    )

    // 启动服务
    server.SetPort(":8080")
    server.Run()
}
```

在 controller 中调用微服务的方法：

需要实现一个基础的service，如下：
```go
package controller

import (
	"github.com/ServiceWeaver/weaver"
	"github.com/sagoo-cloud/nexframe/nf"
	"sagooiot/ruleEngine"
	"sagooiot/server/configSrv"
	"sagooiot/server/dictSrv"
)

type BaseWeaverService struct {
	F             *nf.APIFramework // 将被自动注入，可以通过它访问框架提供的功能，也可以不填写。
	ServiceDict   weaver.Ref[dictSrv.T]
}

func (c *BaseWeaverService) Initialize(f *nf.APIFramework) error {
	if err := f.MustGetWeaverService("ServiceConfig", &c.ServiceConfig); err != nil {
		return err
	}
	if err := f.MustGetWeaverService("ServiceDict", &c.ServiceDict); err != nil {
		return err
	}
	if err := f.MustGetWeaverService("ServiceRule", &c.ServiceRule); err != nil {
		return err
	}
	return nil
}

```

然后在自己实现的controller中，继承BaseWeaverService，就可以在controller中调用微服务的方法了。

```go
// DictController 示例控制器
type DictController struct {
	controller.BaseWeaverService
}

```

在俱体的接口实现在就可以按下面的方式进行服务调用了。` c.ServiceDict.Get()`
```go
	// 直接使用 ServiceConfig
	configData, err := c.ServiceDict.Get().GetSystemConfigData(ctx, "sys.uploadFile.fileType")
	
```