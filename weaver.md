# 构建微服务

nexframe的微服务是基于Service Weaver来实现的。

Service Weaver是Google开源的一个编程框架(programming framework) ，用于编写、部署和管理用Go开发的分布式应用程序。

使用Service Weaver，你可以像编写在本地机器上运行的传统单进程Go可执行文件一样编写应用程序。然后，将其部署到云中，该框架会将其分解为一组微服务，并将其与云提供商(主要是k8s)集成（如监控、跟踪、日志等）。简单来说，就是“以单体形式编码，以微服务形式部署”。

Google开源的Service Weaver本就是为解决微服务架构在实践中出现的诸多问题而提出的创新思路与实验，为此它提出并实现了三个核心原则：

* 在构建阶段，开发人员只需编写模块化的单体程序；
* 在首次部署和运行阶段，Service Weaver会将逻辑组件分配给物理进程，可以是本地的一个进程，也可以是多个进程，当然最主流的还是分配给运行在公有云提供商k8s的不同pod；
* 以原子方式升级变更应用，彻底杜绝应用的不同版本间的交互。

详细见【[Service Weaver文档](https://serviceweaver.dev/)】

实现weaver微服务只需要三步：

第1步：编写服务程序

跟据人业务逻辑编写服务程序，如：
```go
type Adder interface {
    Add(context.Context, int, int) (int, error)
}

type adder struct {
    weaver.Implements[Adder]
}

func (adder) Add(_ context.Context, x, y int) (int, error) {
    return x + y, nil
}

```

第2步：调用你的服务
使用常规 Go 方法调用来调用您的组件。无需 RPC 或 HTTP 请求。忘记版本控制问题。类型系统保证组件是兼容的。

```go
var adder Adder = ... // See documentation
sum, err := adder.Add(ctx, 1, 2)

```

第3步：部署服务
在本地测试您的应用程序并将其部署到云端。 Service Weaver 让您可以思考代码的用途，而不必担心它在哪里运行。
```shell
$ go test .                       # Test locally.
$ go run .                        # Run locally.
$ weaver ssh deploy weaver.toml   # Run on multiple machines.
$ weaver gke deploy weaver.toml   # Run on Google Cloud.
$ weaver kube deploy weaver.toml  # Run on Kubernetes.
```