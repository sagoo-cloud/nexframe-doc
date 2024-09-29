# 鉴权

Nexframe 实现了默认的基于JWT的鉴权中间件，可以很方便的进行鉴权操作。

参考代码：
    
 ```go
    // 创建服务
    srv := nexframe.Server()

	srv.Use(middleware.JwtMiddleware(auth.JwtConfig{
		SigningKey: secretKey,
	}))

```