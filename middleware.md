# 中间件设置

nexframe 路由中间件通常通过一个闭包来定义，我们可以在闭包函数中处理传入的请求和响应实例或增加额外业务逻辑，然后调用传入的处理器继续后续请求处理（可能是下一个中间件或者最终的路由处理器）。比如，我们可以这样定义一个日志中间件：


```go
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        // Do stuff here
        log.Println(r.RequestURI)
        // Call the next handler, which can be another middleware in the chain, or the final handler.
        next.ServeHTTP(w, r)
    })
}
```

在nexframe中，我们可以在路由组中添加中间件，也可以在路由中添加中间件，也可以在全局中添加中间件，中间件的执行顺序是按照添加的顺序执行的。

```go
	srv := nexframe.Server()
	
	// 添加跨域，日志中间件
	srv.WithMiddleware(middleware.CORSMiddleware)
	srv.WithMiddleware(middleware.LoggingMiddleware)

```

也可以连续添加，如：

```go
	srv.WithMiddleware(
		middleware.Metric,
		middleware.PanicRecover,
	)

```