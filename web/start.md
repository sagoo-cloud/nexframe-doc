# 开始web服务

```go

    // 创建服务
	srv := nexframe.Server()
	
	// 注册控制器
	srv.RegisterController("/",
	    &books.BookController,
	)
	// 启动服务
	server.SetPort(":8080")
	server.Run()
	
	
```