---
title: 函数选项设计模式
date: 2025-11-09
categories:
  - 设计模式
  - Golang
summary: Go 语言的函数选项模式（Functional Options Pattern）是一种优雅的配置模式，用于解决构造函数参数过多、可选参数、参数顺序等问题。
---

Go 语言的**函数选项模式（Functional Options Pattern）** 是一种优雅的配置模式，用于解决构造函数参数过多、可选参数、参数顺序等问题。

## 问题背景

### 传统构造函数的痛点
```go
// 传统的构造函数方式 - 参数爆炸
func NewServer(addr string, port int, timeout time.Duration, maxConns int, tls bool, certFile string, keyFile string) (*Server, error) {
    // ...
}

// 使用时的灾难
server, err := NewServer("localhost", 8080, 30*time.Second, 100, true, "cert.pem", "key.pem")
// 参数顺序容易错，可读性差
```

## 函数选项模式的基本结构

### 核心组件
```go
// 1. 选项函数类型
type Option func(*Server)

// 2. 各种配置函数
func WithTimeout(timeout time.Duration) Option {
    return func(s *Server) {
        s.timeout = timeout
    }
}

func WithMaxConns(maxConns int) Option {
    return func(s *Server) {
        s.maxConns = maxConns
    }
}

// 3. 构造函数使用可变参数
func NewServer(addr string, opts ...Option) (*Server, error) {
    s := &Server{
        addr: addr,
        // 设置默认值
        timeout:  defaultTimeout,
        maxConns: defaultMaxConns,
    }
    
    // 应用所有选项
    for _, opt := range opts {
        opt(s)
    }
    
    return s, nil
}
```

## 完整示例

### 服务器配置示例
```go
package main

import (
    "crypto/tls"
    "time"
)

// Server 结构体
type Server struct {
    addr     string
    port     int
    timeout  time.Duration
    maxConns int
    tls      *tls.Config
}

// Option 函数类型
type Option func(*Server)

// 各种配置函数
func WithPort(port int) Option {
    return func(s *Server) {
        s.port = port
    }
}

func WithTimeout(timeout time.Duration) Option {
    return func(s *Server) {
        s.timeout = timeout
    }
}

func WithMaxConns(maxConns int) Option {
    return func(s *Server) {
        s.maxConns = maxConns
    }
}

func WithTLS(certFile, keyFile string) Option {
    return func(s *Server) {
        cert, err := tls.LoadX509KeyPair(certFile, keyFile)
        if err != nil {
            panic(err) // 实际应该返回错误
        }
        s.tls = &tls.Config{Certificates: []tls.Certificate{cert}}
    }
}

// 构造函数
func NewServer(addr string, opts ...Option) *Server {
    s := &Server{
        addr:     addr,
        port:     8080,           // 默认值
        timeout:  30 * time.Second, // 默认值
        maxConns: 100,            // 默认值
    }
    
    // 应用选项
    for _, opt := range opts {
        opt(s)
    }
    
    return s
}

// 使用示例
func main() {
    // 只使用必需参数
    server1 := NewServer("localhost")
    
    // 使用一些选项
    server2 := NewServer("localhost",
        WithPort(9090),
        WithTimeout(60*time.Second),
    )
    
    // 使用所有选项（顺序无关）
    server3 := NewServer("localhost",
        WithMaxConns(500),
        WithPort(8443),
        WithTLS("cert.pem", "key.pem"),
        WithTimeout(120*time.Second),
    )
    
    _ = server1
    _ = server2  
    _ = server3
}
```

## 高级用法

由于WithXxx是函数，所以扩展出一些高级使用方法。比如在函数里增加校验逻辑、默认覆盖等操作，这样能支持更多的配置逻辑。

### 支持错误处理
```go
// 支持错误的 Option
type Option func(*Server) error

func WithTLS(certFile, keyFile string) Option {
    return func(s *Server) error {
        cert, err := tls.LoadX509KeyPair(certFile, keyFile)
        if err != nil {
            return err
        }
        s.tls = &tls.Config{Certificates: []tls.Certificate{cert}}
        return nil
    }
}

func NewServer(addr string, opts ...Option) (*Server, error) {
    s := &Server{addr: addr}
    
    for _, opt := range opts {
        if err := opt(s); err != nil {
            return nil, err
        }
    }
    
    return s, nil
}
```

### 配置验证
```go
func WithTimeout(timeout time.Duration) Option {
    return func(s *Server) error {
        if timeout < 0 {
            return errors.New("timeout must be positive")
        }
        if timeout > 24*time.Hour {
            return errors.New("timeout too large")
        }
        s.timeout = timeout
        return nil
    }
}
```

### 选项的默认值和覆盖
```go
// 默认选项
func defaultOptions() []Option {
    return []Option{
        WithPort(8080),
        WithTimeout(30 * time.Second),
        WithMaxConns(100),
    }
}

func NewServer(addr string, opts ...Option) (*Server, error) {
    s := &Server{addr: addr}
    
    // 先应用默认选项
    for _, opt := range defaultOptions() {
        if err := opt(s); err != nil {
            return nil, err
        }
    }
    
    // 再应用用户选项（覆盖默认值）
    for _, opt := range opts {
        if err := opt(s); err != nil {
            return nil, err
        }
    }
    
    return s, nil
}
```

## 实际应用案例

{{< github repo="robinlg/onexlib" showThumbnail=false >}}

我Github里有一个仓库是作为多个项目的[公共依赖库](https://github.com/robinlg/onexlib)，其中就有很多包使用了这个设计模式，以下举两个例子：

### 构造GORM的查询条件

where包通过配置where.Options结构体，来配置GORM的查询条件
```go
type Options struct {
	Offset int `json:"offset"`
	Limit int `json:"limit"`
	Filters map[any]any
	Clauses []clause.Expression
}

func WithOffset(offset int64) Option {
	return func(whr *Options) {
		if offset < 0 {
			offset = 0
		}
		whr.Offset = int(offset)
	}
}

func WithLimit(limit int64) Option {
	return func(whr *Options) {
		if limit <= 0 {
			limit = defaultLimit
		}
		whr.Limit = int(limit)
	}
}

func WithPage(page int, pageSize int) Option {
	return func(whr *Options) {
		if page == 0 {
			page = 1
		}
		if pageSize == 0 {
			pageSize = defaultLimit
		}

		whr.Offset = (page - 1) * pageSize
		whr.Limit = pageSize
	}
}

func WithFilter(filter map[any]any) Option {
	return func(whr *Options) {
		whr.Filters = filter
	}
}

func WithClauses(conds ...clause.Expression) Option {
	return func(whr *Options) {
		whr.Clauses = append(whr.Clauses, conds...)
	}
}

func NewWhere(opts ...Option) *Options {
	whr := &Options{
		Offset:  0,
		Limit:   defaultLimit,
		Filters: map[any]any{},
		Clauses: make([]clause.Expression, 0),
	}

	for _, opt := range opts {
		opt(whr) // Apply each Option to the opts.
	}

	return whr
}
```
```go
// 使用
where := NewWhere(
    WithLimit(10),
    WithOffset(0),
    WithFilter(map[any]any{
        "name": "abc",
    }),
)
```

### 创建随机code

id包通过配置id.CodeOptions结构体，来创建随机code

```go
type CodeOptions struct {
	chars []rune
	n1    int
	n2    int
	l     int
	salt  uint64
}

func WithCodeChars(arr []rune) func(*CodeOptions) {
	return func(options *CodeOptions) {
		if len(arr) > 0 {
			getCodeOptionsOrSetDefault(options).chars = arr
		}
	}
}

func WithCodeN1(n int) func(*CodeOptions) {
	return func(options *CodeOptions) {
		getCodeOptionsOrSetDefault(options).n1 = n
	}
}

func WithCodeN2(n int) func(*CodeOptions) {
	return func(options *CodeOptions) {
		getCodeOptionsOrSetDefault(options).n2 = n
	}
}

func WithCodeL(l int) func(*CodeOptions) {
	return func(options *CodeOptions) {
		if l > 0 {
			getCodeOptionsOrSetDefault(options).l = l
		}
	}
}

func WithCodeSalt(salt uint64) func(*CodeOptions) {
	return func(options *CodeOptions) {
		if salt > 0 {
			getCodeOptionsOrSetDefault(options).salt = salt
		}
	}
}

func getCodeOptionsOrSetDefault(options *CodeOptions) *CodeOptions {
	if options == nil {
		return &CodeOptions{
			// base string set, remove 0,1,I,O,U,Z
			chars: []rune{
				'2', '3', '4', '5', '6',
				'7', '8', '9', 'A', 'B',
				'C', 'D', 'E', 'F', 'G',
				'H', 'J', 'K', 'L', 'M',
				'N', 'P', 'Q', 'R', 'S',
				'T', 'V', 'W', 'X', 'Y',
			},
			// n1 / len(chars)=30 cop rime
			n1: 17,
			// n2 / l cop rime
			n2: 5,
			// code length
			l: 8,
			// random number
			salt: 123567369,
		}
	}
	return options
}
```
```go
// 使用
options := []func(*CodeOptions){
    WithCodeChars([]rune{'0', '1', '2', '3', '4', '5', '6', '7', '8', '9'}),
    WithCodeN1(9),
    WithCodeN2(3),
    WithCodeL(5),
    WithCodeSalt(56789),
}

func NewCode(id uint64, options ...func(*CodeOptions)) string {
    ops := getCodeOptionsOrSetDefault(nil)
	for _, f := range options {
		f(ops)
	}
    ...
}
```

## 与其他模式的对比

###  vs 配置结构体模式
```go
// 配置结构体模式
type Config struct {
    Addr     string
    Port     int
    Timeout  time.Duration
    MaxConns int
}

func NewServer(cfg Config) *Server {
    // ...
}

// 使用
server := NewServer(Config{
    Addr:     "localhost",
    Port:     8080,
    Timeout:  30 * time.Second,
    MaxConns: 100,
})
```

**对比**：
- **函数选项模式**：更灵活，支持复杂的验证逻辑
- **配置结构体模式**：更简单，适合简单场景

## 总结

**函数选项模式的优点**：
1. **可读性强**：命名的参数比位置参数更清晰
2. **灵活性高**：可选参数、任意顺序、默认值
3. **易于扩展**：添加新选项不影响现有代码
4. **类型安全**：编译时检查

**适用场景**：
- 构造函数参数较多
- 有很多可选参数
- 需要复杂的配置逻辑
- API 需要良好的可扩展性

**核心思想**：**使用函数闭包来修改配置对象，通过可变参数提供灵活的配置方式**。

这种模式在 Go 的标准库和流行开源项目中广泛应用，是 Go 语言中非常地道的配置管理方式。