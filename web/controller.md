# 控制器实现

在控制器层，实现具体的接口方法，在函数签名中使用api定义文件的相关的设置。
```go
package books

import (
	"context"
	"fmt"
	api "sagooiot/frontend/internal/api/v3/notice" // 引入api定义文件
)

// BookController 示例控制器

var BookController = cBookController{
}
type cBookController struct{}

func (c *cBookController) GetBookById(ctx context.Context, req *api.GetBookByIdReq) (*api.GetBookByIdRes, error) {
	fmt.Printf("GetBookById: %v\n", req)
	// 实现业务逻辑
	return &api.GetBookByIdRes{
		Types: "工具书",
		Title: "《红楼梦》",
	}, nil
}


```