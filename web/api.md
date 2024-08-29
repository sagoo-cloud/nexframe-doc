# API定义

在nexframe中可以基于OpenAPIv3的规范定义api。在控制器将自动绑定数据。

接口的元数据信息可以通过为输入结构体 embedded 方式嵌入 g.Meta 结构，并通过 g.Meta 的属性标签方式来实现。

示例如下：

```go

package api

import "github.com/sagoo-cloud/nexframe/nf/g"

type GetBookByIdReq struct {
	g.Meta  `path:"/book/info" method:"GET" summary:"获取书的信息" tags:"书管理"`
	Id      string `json:"id" description:"id，图书ID" v:"required#id不能为空"`
	KeyWord string `json:"keyWord" description:"关键字"`
}

type GetBookByIdRes struct {
	Types     string `json:"types" description:"图书类型"`
	CreatedAt string `json:"createdAt" description:"创建时间"`
	Title     string `json:"title" description:"图书标题"`
}

```