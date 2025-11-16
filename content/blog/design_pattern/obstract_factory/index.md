---
title: 抽象工厂设计模式
date: 2025-11-16
categories:
  - 设计模式
  - Golang
summary: 抽象工厂模式在需要创建一整组相关联、相互依赖的对象时非常有用。它通过引入“抽象工厂”这个概念，将产品的创建与使用完全分离，极大地提高了系统的灵活性和可维护性。
---

抽象工厂模式在需要创建一整组相关联、相互依赖的对象时非常有用。它通过引入“抽象工厂”这个概念，将产品的创建与使用完全分离。

通过使用不同的方法，创建不同的对象，对象之间相互隔离，极大地提高了系统的灵活性和可维护性。

## 问题背景

### 不使用抽象工厂模式的痛点

基础组件定义
```go
// 抽象产品接口
type Button interface {
    Render()
}

type Dialog interface {
    Render()
}

// 具体产品实现
type LightButton struct{}
func (l *LightButton) Render() { fmt.Println("渲染浅色主题按钮 - 浅蓝色背景") }

type DarkButton struct{}
func (d *DarkButton) Render() { fmt.Println("渲染深色主题按钮 - 深蓝色背景") }

type LightDialog struct{}
func (l *LightDialog) Render() { fmt.Println("渲染浅色主题对话框 - 白色背景") }

type DarkDialog struct{}
func (d *DarkDialog) Render() { fmt.Println("渲染深色主题对话框 - 黑色背景") }
```

 客户端代码（不使用抽象工厂的痛点体现）
 ```go
type Application struct {
    theme string
}

func (app *Application) CreateUI() {
    var button Button
    var dialog Dialog
    
    // 痛点1：客户端需要了解所有具体类（比如 light 的实现类，dark 的实现类，以及其他所有实现类）
    // 注：不使用工厂模式也有会有痛点1
    if app.theme == "light" {
        button = &LightButton{}
        dialog = &LightDialog{}

        // 痛点2：每增加一个组件，这里就要重复写一遍，重复的创建逻辑
        // 注：不使用工厂模式也有会有类似的痛点2，不同的是工厂模式这里的代码是调用方法
        // input := &LightInput{}
        // menu := &LightMenu{}

        // 痛点3：容易产生不一致的组件组合，比如应该都创建 LightXxx，这里创建了 LightXxx 和 DarkXxx
        // 住：痛点3不存在与工厂模式中，因为工厂模式不涉及多个组件的组合
        // button = &LightButton{}
        // dialog = &DarkDialog{}
    } else if app.theme == "dark" {
        button = &DarkButton{}
        dialog = &DarkDialog{}

        // 痛点2：每增加一个组件，这里就要重复写一遍，重复的创建逻辑
        // 注：不使用工厂模式也有会有痛点2
        // input := &DarkInput{}
        // menu := &DarkMenu{}
    } else {
        panic("不支持的主题: " + app.theme)
    }
    
    // 渲染UI
    button.Render()
    dialog.Render()
}
 ```

## 抽象工厂模式的基本结构

```go
// 1.抽象接口（基础组件）
type Button interface {
    Render()
}

type Dialog interface {
    Render()
}

// 2.具体实现（基础组件）
type LightButton struct{}
func (l *LightButton) Render() { fmt.Println("渲染浅色主题按钮 - 浅蓝色背景") }

type LightDialog struct{}
func (l *LightDialog) Render() { fmt.Println("渲染浅色主题对话框 - 白色背景") }

// 3.抽象工厂接口（核心所在）
type UIFactory interface {
    CreateButton() Button
    CreateDialog() Dialog
}

// 4. 定义具体工厂
type LightUIFactory struct{}

func (f *LightUIFactory) CreateButton() Button {
    return &LightButton{}
}

func (f *LightUIFactory) CreateDialog() Dialog {
    return &LightDialog{}

// 5. 客户端调用
type Application struct {
    factory UIFactory // 只依赖抽象工厂接口
}

// NewApplication 依赖注入工厂
func NewApplication(factory UIFactory) *Application {
    return &Application{factory: factory}
}

func (app *Application) CreateUI() {
    // 解决痛点1：客户端不需要知道所有实现类，省略了很多if else分支代码
    // 解决痛点2：因为不需要if else分支，省略了很多重复的代码逻辑
    // 解决痛点3：调用方不会产生错误组合，由组件提供方保证了组件的统一
    button := app.factory.CreateButton()
    dialog := app.factory.CreateDialog()
    
    button.Render()
    dialog.Render()
}

```

## 实际应用案例

{{< github repo="robinlg/onexblog" showThumbnail=false >}}

我Github里有项目[迷你博客](https://github.com/robinlg/onexblog)，其中就有很多包使用了这个设计模式，以下举两个例子：

### store层

store层用来跟数据库或者其他第三方维服务进行交互
```go
// 1.抽象接口（基础组件）
type ConcretePostStore interface {
	Create(ctx context.Context, obj *model.PostM) error
	Update(ctx context.Context, obj *model.PostM) error
    ....
}

type UserStore interface {
	Create(ctx context.Context, obj *model.UserM) error
	Update(ctx context.Context, obj *model.UserM) error
	...
}

// 2.具体实现（基础组件）
type concretePostStore struct {
	store *datastore
}

type userStore struct {
	*genericstore.Store[model.UserM]
}


func (s *concretePostStore) Create(ctx context.Context, obj *model.PostM) error {
	...
}

func (s *concretePostStore) Update(ctx context.Context, obj *model.PostM) error {
	...
}

func (s *Store[T]) Create(ctx context.Context, obj *T) error {
	...
}

func (s *Store[T]) Update(ctx context.Context, obj *T) error {
	...
}

// 3.抽象工厂接口（核心所在）
type IStore interface {
	...
	User() UserStore
	Post() ConcretePostStore
    ...
}

// 4. 定义具体工厂
type datastore struct {}

func (store *datastore) User() UserStore {
	return newUserStore(store)
}

func (store *datastore) Post() PostStore {
	return newPostStore(store)
}

// 5. 客户端调用
func (b *userBiz) ListWithBadPerformance(ctx context.Context, rq *apiv1.ListUserRequest) (*apiv1.ListUserResponse, error) {
	...
	count, userList, err := b.store.User().List(ctx, whr)
    ...
	for _, user := range userList {
		count, _, err := b.store.Post().List(ctx, where.T(ctx))
		...
	}
    ...
}

```

### biz层

biz层主要用来实现系统中REST资源的各类业务操作，例如用户资源的增删改查等。

```go
// 1.抽象接口（基础组件）
type UserBiz interface {
	Create(ctx context.Context, rq *apiv1.CreateUserRequest) (*apiv1.CreateUserResponse, error)
    ...
}

type PostBiz interface {
	Create(ctx context.Context, rq *apiv1.CreatePostRequest) (*apiv1.CreatePostResponse, error)
	...
}

// 2.具体实现（基础组件）
func (b *userBiz) Create(ctx context.Context, rq *apiv1.CreateUserRequest) (*apiv1.CreateUserResponse, error) {
    ...
}

func (b *postBiz) Create(ctx context.Context, rq *apiv1.CreatePostRequest) (*apiv1.CreatePostResponse, error) {
    ...
}

// 3.抽象工厂接口（核心所在）
type IBiz interface {
	UserV1() userv1.UserBiz
	PostV1() postv1.PostBiz
	// PostV2() postv2.PostBiz
    ...
}

// 4. 定义具体工厂
func (b *biz) UserV1() userv1.UserBiz {
	return userv1.New(b.store, b.authz)
}

func (b *biz) PostV1() postv1.PostBiz {
	return postv1.New(b.store)
}


// 5. 客户端调用
func (h *Handler) CreateUser(ctx context.Context, rq *apiv1.CreateUserRequest) (*apiv1.CreateUserResponse, error) {
	return h.biz.UserV1().Create(ctx, rq)
}

func (h *Handler) CreatePost(ctx context.Context, rq *apiv1.CreatePostRequest) (*apiv1.CreatePostResponse, error) {
	return h.biz.PostV1().Create(ctx, rq)
}

```

## 与工厂模式的区别

用大白话来简单描述他们两个的关系：

工厂模式：定义了一些抽象方法，由不同的实例对象实现这些抽象方法
抽象工厂模式：定义了一抽象方法，这些方法构建出`工厂模式`的实例对象。这些抽象方法由`抽象工厂模式`自己的实例对象实现。

可以看出其实抽象工厂模式就是在工厂模式之上再做了一层抽象，它解决了对象组合的问题。用上面的实际项目来举例子：

```go
// 抽象工厂接口，它组合了 User 和 Post 实例对象组合问题
// 对于整个项目来说，它需要 User 和 Post 对象组合使用，才能实现整个博客的需求
type IBiz interface {
    // UserV1() 就是上面说的抽象工厂模式的抽象方法
    // 这个方法构建出了 UserBiz 实例对象
	UserV1() userv1.UserBiz
	// PostV1() postv1.PostBiz
	// PostV2() postv2.PostBiz
    ...
}

// 而 UserV1() 的实现，由抽象工厂模式自己的实例对象实现
type biz struct {
	store store.IStore
	authz *auth.Authz
}
func NewBiz(store store.IStore, authz *auth.Authz) *biz {
	return &biz{store: store, authz: authz}
}
func (b *biz) UserV1() userv1.UserBiz {
	return userv1.New(b.store, b.authz)
}
```

```go
// 如果整个项目用一个 UserBiz 实例对象就能解决，不需要多个对象组合的话，
// 那么无需定义抽象接口 IBiz。
// 去掉 IBiz 这层抽象接口之后，剩下的代码就可以看作是工厂模式的实现

// 工厂模式接口
type UserBiz interface {
	Create(ctx context.Context, rq *apiv1.CreateUserRequest) (*apiv1.CreateUserResponse, error)
    ...
}

// 实现了 UserBiz 接口的实例对象
type userBiz struct {
	store store.IStore
	authz *authz.Authz
}
func New(store store.IStore, authz *authz.Authz) *userBiz {
	return &userBiz{store: store, authz: authz}
}

// 工厂模式接口的实现方法
func (b *userBiz) Create(ctx context.Context, rq *apiv1.CreateUserRequest) (*apiv1.CreateUserResponse, error) {
    ...
}
```