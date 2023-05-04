
# 类
## 类的权限控制

友元friend，定义在类内，可以访问private变量（不推荐使用，破坏封装）



## 委托构造器

```C++
class A {
 public:
  int a_,b_,c_;
  A(int a, int b, int c): a_(a),b_(b),c_(c) {
    cout << "three args" << endl;
  }
  A(int a, int b): A(a,b,100) {
    cout << "two args" << endl;
  }
}
------------------------------------------------
A a(10, 20);
Output:
three args
two args
```

## Explicit关键字

要求类申请对象时必须为显示调用（构造）
```C++
class A {
 public:
  int a_,b_,c_;
  A(int a, int b, int c): a_(a),b_(b),c_(c) {
    cout << "three args" << endl;
  }
  A(int a): A(a,50,100) {
    cout << "one args" << endl;
  }
}
void func(A a) {
  cout << "func" << endl;
}
------------------------------------------------
func(25);   // 这是合法的，会隐式调用构造函数A(int a)
Output:
three args
one args
func
```
```C++
class A {
 public:
  int a_,b_,c_;
  A(int a, int b, int c): a_(a),b_(b),c_(c) {
    cout << "three args" << endl;
  }
  explicit A(int a): A(a,50,100) {
    cout << "one args" << endl;
  }
}
void func(A a) {
  cout << "func" << endl;
}
------------------------------------------------
func(25);    // 这是不合法的，explicit表示禁止隐式调用，必须通过显式构造
func(A(25)); //合法
```

 ## default 默认构造函数

如果一个类没有构造函数，会默认存在一个A() = default; 若有构造函数则需要手动添加。default即默认全部赋0（NOTICE：如果类内有成员为类，需要该类也存在default构造函数）
```C++
class A {
 public:
  int a_,b_,c_;
  A(int a, int b, int c): a_(a),b_(b),c_(c) {
    cout << "three args" << endl;
  }
  A() = default;
}
void func(A a) {
  cout << "func" << endl;
}
------------------------------------------------
func(25);    // 这是不合法的，explicit表示禁止隐式调用，必须通过显式构造
func(A(25)); //合法
```
**NOTICE：实践中最好永远保留一个default构造器**

## static关键字

static成员随着程序（main）的存在而存在

static静态方法内不存在this，它不依附于任一个对象，所以不可以调用类内的非static变量

static成员可以在类内声明，但必须在*<u>类外</u>*定义

```C++
class A {
  static int static_member; // declare
}
A::static_member = 10; // definition
```

# 内存管理

堆空间heap在上，由程序员分配释放（new、delete）。堆空间大

栈空间stack在下，由编译器自动分配释放存放函数参数、局部变量。栈空间小

# 虚函数

## 动态绑定 Dynamic Binding/Dispatch

interchangeably

虚函数的基类引用/指针 会调用**子类**的方法

# 浅深拷贝 Shallow / Deep Copy

浅拷贝原理：直接进行简单的复制

深拷贝要做的事

```C++
class A{
public:
  int *a;
  A(int a): a(new int(a)) {}
  ~A() { delete a; }
  // 拷贝构造函数，对于指针，重新new一块内存
  A(const A& cpa) { this->a = new int(*(cpa.a)); }
};
```

# 智能指针
## Shared Pointer
make_shared会动态分配一块内存，创建对应资源，并让shared_pointer指向它。

## 危险：使用shared pointer时不要使用delete进行手动释放！

### 创建方法

```C++
shared_ptr<int> p;
p = make_shared<int>(100);
shared_ptr<int> p1 {new int(200)};
// 或shared_ptr<int> p1 {make_shared<int>(200)};
shared_ptr<int> p2 = p1; //可以进行复制操作
```

## 原理：引用计数

## get方法：获取裸指针（不推荐）

利用ptr.get()方法可以获取裸指针，但如果共享指针全部被reset后，即便裸指针仍然存在，该共享指针仍然会被释放

## reset 方法

无参数用法: ptr.reset() 释放某个共享指针，引用计数减一

有参数用法:

```C++
shared_ptr<int> sp;
sp = make_shared<int>(10);
sp.reset(new int(20));
```

## 别名 Aliasing

```C++
struct Bar{int i=123;};
struct Foo {Bar bar;};
int main() {
    shared_ptr<Foo> f = make_shared<Foo>();
    cout << f.use_count() << endl; // print 1
    shared_ptr<Bar> b(f, &(f->bar));
    cout << f.use_count() << endl; // print 2
    
    cout << b->i << endl; // print 123;
}
```



### 悬空指针 dangling pointer

```C++
void func()
{
    // dangling pointer 悬空指针：内存已被释放
    A* a = new A(42);
    {
        shared_ptr<A> ptr(a);
    }
    cout << a->a << endl;
}
```

# 使用自定义的析构

关闭文件/结束网络连接时常用

```C++
void DestructAClass(A* a) {
    delete a->a;
    cout << "In self-made mode" << endl;
}

void func()
{
    A* a = new A(42);
    shared_ptr<A> ptr(new A(42), DestructAClass);
}
```
```C++
void close_file(FILE* fp) {
    if(fp == nullptr) return;
    fclose(fp);
    cout << "File closed" << endl;
}

void func()
{
    FILE* fp = fopen("data.txt", "w");
    shared_ptr<FILE> sfp(fp, close_file);
}
```
## 独享指针 Unique Pointer

unique pointer独占某个资源的管理权，当unique pointer销毁或reset时，绑定的资源也会自动释放

不能有两个unique pointer指向同一份资源，也不支持复制

```C++
// 在这种情况下也可能发生内存泄漏，因为如果p->fun()抛出异常，delete p便不会被执行，但使用unique_ptr可以避免这种情况
void func() {
    Object *p = new Object;
    p->fun();
    delete p;
}
```

释放：up.reset( [new Ball{}] ) // 若有参数，则为指向其他实例

获取裸指针：up.get() （当某些函数只能接受裸指针时有用）

## 解绑 release()

release返回资源的裸指针，同时把unique_ptr变成nullptr

```C++
auto ball = make_unique<Ball>();
Ball * ball = up.release( ); 
```

如果进行以下操作，也会释放资源

```C++
up = nullptr;	// 会释放资源
```

## unique_ptr不能复制

以下代码会error

```C++
unique_ptr<int> up1 = make_unique<int>(100);
unique_ptr<int> up2(up1);	// error
up2 = up1;					// error
```

## 控制权转移

```C++
unique_ptr<int> up1 = make_unique<int>(100);
unique_ptr<int> up2(up1.release());
unique_ptr<int> up2 = std::move(up1);
```

## 函数间的参数传递

unique_ptr由于禁止复制操作，因此传参时需要注意

```C++
void pass_up(unique_ptr<int> up) {
    cout << "In pass_up:" << *up << endl;
}
int main()
{
    auto up = make_unique<int>(123);
    pass_up(up);
}
```

### 方法一：传递引用

```C++
void pass_up(int& up_value) {
    cout << "In pass_up:" << up_value << endl;
}
int main()
{
    auto up = make_unique<int>(123);
    pass_up(*up);
}
```

### 方法二：传递裸指针

```C++
void pass_up2(int* p) {
    cout << "In pass_up2:" << *p << endl;
}
int main()
{
    auto up = make_unique<int>(123);
    pass_up2(up.get());
}
```

### 方法三：传递unique_ptr的引用

如果函数需要改变**unique_ptr本身(如果只需要改变指向的内容，传递裸指针即可)**，则需要传递引用

```C++
void pass_up3(unique_ptr<int>& up) {
    cout << "In pass_up3:" << *up << endl;
    up.reset();
}
int main()
{
    auto up = make_unique<int>(123);
    pass_up3(up);
    if(up == nullptr) cout << "up is reset." << endl;
}
```

### 方法四：std::move

std::move可以转移unique_ptr对资源的控制权

```C++
void pass_up4(unique_ptr<int> up) {
    cout << "In pass_up4:" << *up << endl;
}
int main()
{
    auto up = make_unique<int>(123);
    pass_up4(move(up));
    if(up == nullptr) cout << "up is reset." << endl;
}
```

## 函数可以返回unique_ptr

此时编译器知道将会临时销毁，故会进行优化

```C++
unique_ptr<int> return_uptr(int value) {
    unique_ptr<int> up = make_unique<int>(value);
    return up;		// 实际上为return move(up);
}
int main()
{
    unique_ptr<int> up = return_uptr(321);
    cout << "up: " << *up << endl;
}
```

