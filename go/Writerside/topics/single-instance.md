# 单例模式 

这一篇文档描述了在 Go 中单例模式的实现。最简单的实现如下:

```go
type Client struct{}

var theClient *Client 

func NewClient() *Client {
	if theClient == nil {
		theClient = &Client{}
	}
	return theClient
}
```

但是这样的实现方法是存在并发问题的，可能会实例化多个 `Client` 对象。我们可以考虑加锁来解决：

```go
var l sync.Mutex

func NewClient() *Client {
	l.Lock()
	defer l.Unlock()
    // ...省略重复代码
}
```

这个实现功能上没有问题，但是性能上有问题，每一次调用 `NewClient` 都会上锁。那么我们可以让`theClient == nil` 的时候再加锁，如下:

```go
func NewClient() *Client {
	if theClient == nil {
		l.Lock()
		defer l.Unlock()
		theClient = &Client{}
	}
	return theClient
}
```

这样的实现包含了一个非常隐匿的问题，就是在高并发下，我们的 `Client` 结构体可能赋值到一半，其他协程执行的时候就会发现 `theClient != nil`, 这样还是会实例化多个。 于是有了下面这个更复杂的版本:

```go
type Client struct{}

var theClient *Client

var l sync.Mutex

var done int32

func NewClient() *Client {
	if atomic.LoadInt32(&done) == 1 {
		return theClient
	}
	l.Lock()
	defer l.Unlock()
	if initialed == 0 {
		theClient = &Client{}
		atomic.StoreInt32(&done, 1)
	}
	return theClient
}
```

利用了 `atomic`读写 `initialed` 变量，检查是否初始化完成。这个版本啥问题也没有，就是写起来复杂。于是有了下面这个简化版本:

```go
type Client struct{}

var theClient *Client

var once sync.Once

func NewClient() *Client {
	once.Do(func () {
		theClient = &Client{}	
	})	
	return theClient
}
```

其实，上面这个版本就是 `sync.Once`的实现，你可以进入它的源码查看。