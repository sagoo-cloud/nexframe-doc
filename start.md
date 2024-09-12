# 快速开始

本章内容将通过一个示例项目的创建，来引导怎么创建与使用nexframe的项目。

## 创建项目

创建一个文件夹作为项目目录，采用开发工具打开这个目录，如使用Goland 。

在这个项目目录下，采用命令行初始化项目

```azure
go mod init github.com/yourname/yourproject

```

## 添加依赖

在项目目录下，使用命令行添加依赖

```azure

go get github.com/sagoo-cloud/nexframe

```
## 工程目录

以下是建议的工程目录，可跟据实际情况自行定义。

```shell

├── cmd
│       ├── gen # 生
├── config # 配置文件
│       ├── config.example.toml
│       ├── config.toml
│       └── lang
│           ├── en.json
│           └── zh.json
├── consts # 常量
├── frontend # 前端
│       ├── api # 前端API
│       ├── controller # 前端控制器
│       ├── frontend.go # 前端入口
│       ├── logic # 前端逻辑
│       └── service # 前端服务封装
├── main.go
├── model # 数据库模型，GORM的dao生成也在这个目录
├── pkg # 公共包
├── readme.md
├── resource # 资源文件
└── serever # 各微服务


```

## 编写代码
在项目目录下，创建一个main.go文件，并编写如下代码

```go
package main

import (
	"fmt"
	"github.com/sagoo-cloud/nexframe"
	"net/http"
)

func main() {
	server := nexframe.Server()
	// 注册控制器
	err := server.BindHandlerFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Println(w, "Hello, world!")
	})
	if err != nil {
		return
	}
	server.SetPort(":8080")
	server.Run()
}

```

## 运行项目
在项目目录下，使用命令行运行项目
```azure
go run main.go

```
## 访问项目
在浏览器中访问http://localhost:8080/，即可看到nexframe的欢迎页面。

