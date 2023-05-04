
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

## Unique Pointer

