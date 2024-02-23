# 函数选项模式 

这篇文档介绍了函数选项模式，这种设计模式运用在如果你需要在一个函数中接收多个参数并提供默认值的场景。这种模式在很多的框架中都有应用，比如说 gRPC 中创建客户端拨号 `gRpc.dial()`。

如下示例:

```go
type Client struct {
	opts *Options
}

type Options struct {
	Host string
	Port int32
}

type Option func(*Options)

func WithHost(host string) Option {
	return func(o *Options) {
		o.Host = host
	}
}

func WithPort(port int32) Option {
	return func(o *Options) {
		o.Port = port
	}
}

func NewClient(options ...Option) *Client {
	opts := &Options{
		Host: "127.0.0.1",
		Port: 3306,
	}
	for _, option := range options {
		option(opts)
	}
	return &Client{opts}
}
```

测试一下:

```go
func main() {
	client := NewClient(WithHost("192.168.8.1"))
	fmt.Printf("Address=%s:%d\n", client.opts.Host, client.opts.Port) // 192.168.8.1:3306
	client = NewClient(WithPort(3308))
	fmt.Printf("Address=%s:%d\n", client.opts.Host, client.opts.Port) // 127.0.0.1:3308
}
```

如同很多设计模式一样，不熟悉的时候就觉得很绕。熟悉了之后，就习惯了。这种设计模式是因为 Go 语言特性决定的，不能设定默认值。

其实换一个角度去想，也许就能理解了。复杂的是 `NewClient` 的封装，但是 `NewClient` 的调用并不复杂，只需要传入一系列 `With` 开头的方法就是了。所以当你在代码里看到这种写法的时候，不必惊讶，为的就是解决众多传值且包含部分默认值的情况。