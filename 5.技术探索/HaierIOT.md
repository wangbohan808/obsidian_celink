学习日期：2026-04-15 星期三
主题关键字：
关键待办：

---

###### C与C++中为什么需要使用线程概念

1. 如果只有一个主线程，一次只能干一件事，就会造成同步阻塞，所有逻辑只能排队执行



###### 进程与线程的区别

1. 不同进程之间，地址空间与内存完全隔离，跨进程访问需要借助：管道、socket、共享内存、信号、消息队列（内核中转）
2. 线程之间直接共享内存，并发读写会直接引发数据竞争，必须加锁
3. 所有资源归属进程，线程只是某个进程下的资源“执行者”

总结：
多线程适合高频并发、轻量任务；
多进程适合强隔离、稳定性要求高的场景



###### 定位代码中线程位置--找到代码中并发的主线



###### 函数指针与std::function的使用场景

**一个变量--描述函数的返回值、传入参数**

- std::function直接作为一个类型，类型变量的格式和传统声明统一
- 函数指针的类型与变量的描述格式有点特殊

```cpp
// 待回调的普通函数
void printMsg(int val)
{
    cout << "函数指针回调: " << val << endl;
}

// 接收函数指针做回调
void runCallback(void (*cb)(int), int data)
{
    if (cb)
        cb(data);
}

int main()
{
    // 绑定+调用
    runCallback(printMsg, 666);
    return 0;
}
```

```cpp
void printMsg(int val)
{
    cout << "std::function回调: " << val << endl;
}

// 接收 std::function
void runCallback(function<void(int)> cb, int data)
{
    if (cb)
        cb(data);
}

int main()
{
    runCallback(printMsg, 666);
    return 0;
}
```


**存储多个回调/函数数组**

```cpp
// 只能存无状态普通函数
void add(int a, int b) { cout << a+b << endl; }
void mul(int a, int b) { cout << a*b << endl; }

int main()
{
    // 函数指针数组
    void (*funcArr[2])(int, int) = {add, mul};

    funcArr[0](3,4);
    funcArr[1](3,4);
    return 0;
}
```

```c++
#include <iostream>
#include <functional>
#include <vector>
using namespace std;

void add(int a, int b) { cout << "加法: " << a + b << endl; }
void mul(int a, int b) { cout << "乘法: " << a * b << endl; }

int main()
{
    vector<function<void(int,int)>> funcList;

    // 放入普通函数 add
    funcList.emplace_back(add);
    
    // 放入 lambda（减法）
    funcList.emplace_back([](int a, int b) {
        cout << "减法: " << a - b << endl;
    });
    
    // 放入普通函数 mul
    funcList.emplace_back(mul);

    // 依次调用
    for (auto& f : funcList)
    {
        f(10, 5);
    }

    return 0;
}
```



###### Lambda = 一次性的、临时的、匿名的小函数

```cpp
void sub(int a, int b) {
    cout << a - b << endl;
}
```

```cpp
[](int a, int b) {
    cout << a - b << endl;
}
```

###### 一些C++符号

```cpp
class Thread final : public std::enable_shared_from_this<Thread>

class Thread                     // 定义一个名叫 Thread 的类
final                            // 这个类不能被继承（不能当父类）
: public std::enable_shared_from_this<Thread>  
                                 // 公开继承 标准库的 enable_shared_from_this<Thread>
```

`:` 表示继承
`::` 表示归属、谁谁的、后面属于前面这一个标准命名空间
`<>` 给模板传递类型


###### C语言与C++的枚举

```c
enum ThreadState {
    UNINITIALIZED,
    STARTING,
};
```

```cpp
enum class ThreadState {
    kUninitialized,
    kStarting,
};
```

C语言的枚举会污染全局的命名空间，C++类作用域不会全局污染：

```cpp
enum ThreadState state = UNINITIALIZED;  // 直接用
int a = UNINITIALIZED;      // 可以直接转 int

ThreadState state = ThreadState::kUninitialized; // 必须加作用域
// int a = ThreadState::kUninitialized; ❌ 错误！不能隐式转int
```



###### 类成员函数以及std::bing的使用

- std::bing相当于把原本需要传入参数的函数，直接传参，整合为一个不需要传参直接调用的函数
- 类成员函数即使表面上没有参数要传递，但实际上都会有一个this指针，指向调用这个类成员函数的对象的实例，因为只有实例化后才能传递

```cpp
    template <class T>
    static ThreadSharePtr Create(const std::string &name, void (T::*callback)(), T *obj) {
        return Create(name, std::bind(callback, obj));
    }
```



###### 静态成员函数、模板成员函数与静态模板成员函数

 **1. 静态成员函数（static 成员函数）**

核心定义：

用`static`关键字修饰的类成员函数，**不属于某个具体对象，属于整个类**，不依赖对象即可调用，无`this`指针（无法访问类的非静态成员）。

使用示例（贴合你的线程类）：

```cpp
class Thread {
public:
    // 静态成员函数（非模板）
    static ThreadSharePtr Create(const std::string &name) {
        // 无需this指针，可直接访问类的静态成员
        return ThreadSharePtr(new Thread(name));
    }
};

// 调用：无需创建Thread对象，直接用「类名::函数名」
ThreadSharePtr thread = Thread::Create("测试线程");
```

关键特点：

- 无`this`指针，不能访问类的非静态成员（如你的`ThreadState`非静态成员变量）；
    
- 调用时无需创建类对象，直接`类名::函数名()`；
    
- 函数体固定，不支持适配不同类型（比如不能同时接收不同类的成员函数）。
    

**2. 模板成员函数（template 成员函数）**

核心定义：

用`template <class T>`声明的类成员函数，**属于具体对象**（非静态），核心作用是「适配不同类型参数」，实现代码复用（比如你的线程类要接收不同类的成员函数）。

使用示例（贴合你的线程类，修改为非静态模板）：

```cpp
class Thread {
public:
    // 模板成员函数（非静态）
    template <class T>
    ThreadSharePtr Create(const std::string &name, void (T::*callback)(), T *obj) {
        // 有this指针，可访问类的非静态成员（如this->GetStateStr()）
        return Create(name, std::bind(callback, obj));
    }

    // 非静态成员（示例）
    ThreadState GetState() const { return state; }
private:
    ThreadState state;
};

// 调用：必须先创建Thread对象，再调用，且指定模板类型T
Thread thread; // 创建对象
MyTask task;
// 显式指定T=MyTask，调用模板成员函数
ThreadSharePtr ptr = thread.Create<MyTask>("测试线程", &MyTask::run, &task);
```

关键特点：

- 有`this`指针，可访问类的非静态成员（如你的`GetStateStr()`）；
    
- 调用时必须先创建类对象（`Thread thread`），再通过对象调用；
    
- 支持适配不同类型T（比如可接收`MyTask`、`CameraDriver`等不同类的成员函数）。
    

**3. 静态模板成员函数（static + template 成员函数）**

核心定义：

同时用`static`和`template <class T>`修饰的类成员函数，**不属于具体对象，属于整个类，且支持适配不同类型**——就是你线程类中`Create`函数的类型，也是你最常用的场景。

使用示例（完全复刻你的线程类代码）：

```cpp
class Thread final : public std::enable_shared_from_this<Thread> {
public:
    // 静态模板成员函数（你的核心代码）
    template <class T>
    static ThreadSharePtr Create(const std::string &name, void (T::*callback)(), T *obj) {
        return Create(name, std::bind(callback, obj));
    }

    // 其他成员省略...
};

// 调用：无需创建Thread对象，直接类名调用，显式指定T
MyTask task;
// 关键：Thread::Create<MyTask>(...)，指定T=MyTask，适配MyTask的成员函数
ThreadSharePtr thread = Thread::Create<MyTask>("MyTask线程", &MyTask::run, &task);
```

关键特点：

- 无`this`指针，不能访问类的非静态成员；
    
- 调用时无需创建类对象，直接`类名::函数名<T>(...)`；
    
- 支持适配不同类型T（可接收任意类的成员函数），兼顾"静态调用"和"类型适配"——完美解决你的线程类"调用不同类成员函数"的需求。

###### 常量引用

```cpp
const std::string &GetName() const;
```

- 返回引用，无拷贝的开销
- 第一个const，保护类的私有成员变量安全，禁止修改返回值
- 最后的const表示该成员函数不会修改任何非静态成员变量，保护类的私有成员变量的安全



###### 引用、指针、值拷贝的区别

- 值拷贝：
- **在栈上开辟一块新内存**，调用类的**拷贝构造函数**，把实参的数据完整复制到形参空间
- 基础数据类型 (int/double) 为**浅拷贝**，自定义类 / 容器会触发**深拷贝**，开销极大

- 指针传参：
- 传参时仅拷贝**8 字节（64 位系统）地址数值**，**不会拷贝原数据**
- 