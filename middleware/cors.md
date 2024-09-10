# 跨域设置

## 默认设置

在框架中可以使用默认的跨域中间件。允许所有域名访问的。使用方式为：

```go
	srv := nexframe.Server()

    srv.WithMiddleware(srv.CORSDefault) //添加默认的跨域

```
## 个性化设置

如果想要个性化设置，你需要定义一个CORSOptions结构体实例，根据你的应用需求配置CORS选项。这个结构体包含了所有CORS相关的配置项，例如允许的域名、允许的方法、允许的头信息等。

```go
corsOptions := CORSOptions{
    AllowDomain:      []string{"http://example.com", "http://sub.example.com"},
    AllowOrigin:      "*", // 允许所有域名
    AllowCredentials: "true", // 允许携带凭证（cookies）
    ExposeHeaders:    "Content-Length, X-Content-Type-Options", // 暴露给客户端的头信息
    MaxAge:           86400, // 预检请求的缓存时间（秒）
    AllowMethods:     "POST, GET, OPTIONS, PUT, DELETE", // 允许的HTTP方法
    AllowHeaders:     "Origin, X-Requested-With, Content-Type, Accept, Authorization", // 允许的头信息
}

```
应用CORS中间件
在创建web服务之后，使用WithMiddleware方法将CORSMiddleware应用到所有路由上。你需要将corsOptions作为参数传递给CORSMiddleware。

```go

	srv.WithMiddleware(srv.CORSMiddleware(nf.CORSOptions{
		AllowDomain:      []string{"http://example.com", "http://sub.example.com"},
		AllowOrigin:      "*",
		AllowCredentials: "true",
		ExposeHeaders:    "Content-Length, X-Content-Type-Options",
		MaxAge:           86400,
		AllowMethods:     "POST, GET, OPTIONS, PUT, DELETE",
		AllowHeaders:     "Origin, X-Requested-With, Content-Type, Accept, Authorization",
	}))

```

## 注意事项

* AllowDomain: 如果你希望只允许特定的域名进行跨域请求，可以在这里设置。如果设置为空数组或不设置，将使用AllowOrigin的值。
* AllowOrigin: 如果设置为"*"，则允许所有域名进行跨域请求。如果需要限制，可以设置为特定的域名。
* AllowCredentials: 设置为"true"时，允许请求携带凭证（如cookies）。
* ExposeHeaders: 设置你希望在响应中暴露给客户端的特殊头信息。
* MaxAge: 设置预检请求的缓存时间，单位为秒。
* AllowMethods和AllowHeaders: 分别设置允许的HTTP方法和头信息。