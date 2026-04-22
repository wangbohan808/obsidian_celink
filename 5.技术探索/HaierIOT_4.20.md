学习日期：2026-04-20 星期一
主题关键字：
关键待办：

---

###### 共享指针的使用方法

```cpp
#include <memory>
#include <iostream>

int main() {
    // 用 make_shared 创建 shared_ptr
    auto p = std::make_shared<int>(100);

    // 共享指针
    std::shared_ptr<int> p2 = p;

    std::cout << *p << std::endl;   // 100
    std::cout << *p2 << std::endl;  // 100

    // 最后一个指针离开作用域时，自动释放内存
}
```


**多个地方使用同一份数据，就会使用共享内存**
- 节省内存
- 保证数据同步

```cpp
#include <iostream>
#include <memory>
using namespace std;

int main() {
    shared_ptr<int> p1 = make_shared<int>(99);
    cout << "计数1：" << p1.use_count() << endl;  // 1

    {
        shared_ptr<int> p2 = p1;
        cout << "计数2：" << p1.use_count() << endl; // 2
    }
    // p2 离开作用域 → 计数-1

    cout << "计数3：" << p1.use_count() << endl;  // 1

    return 0;
}
```

- 对于指向的内存会进行引用计数
- 复制一次 → 计数 +1
- 销毁一个 → 计数 -1
- 计数到 0 → 才真正 delete 对象


在现代 C++ 里，**不直接操作裸地址**，
即使用到 “指针语义”，也是通过**智能指针、容器、new/delete、构造函数**这类**安全接口**去间接管理内存，而不是手动读写原始地址



###### RAII（资源获取即初始化）

**在 C++ 里，把所有资源（内存、文件、锁、socket…）都交给一个对象管理：**
- **构造函数获取资源**
- **析构函数自动释放资源**
- 只要对象生命周期结束，资源就一定被释放

**RAII = 让 C++ 对象帮你自动做 “收尾工作”，
你只管开，不用管关，它保证一定关。**
- 创建指针 → 自动释放内存
- 创建锁 → 自动解锁
- 打开文件 → 自动关闭
- 建立连接 → 自动断开



###### 异步转同步

理由：
异步转同步后，就可以确保结果已经返回，就可以不在回调内部，而是后续直接调用新的逻辑，即使是与回调结果强相关


```cpp
// 1. 创建 promise
auto promise_ptr = make_shared<PromiseType>();

// 2. 拿 future 等待
auto result_future = promise_ptr->get_future();

// 3. 调用异步接口，把 promise 传进去
uhsd_push_file_to_oss(..., cb, new shared_ptr(promise_ptr));

// 4. 阻塞等待结果
auto result = result_future.get();
```

```txt
【主线程】
1. 创建 promise
2. 创建 future (和promise绑定)
3. 调用 future.get()  ----------→ 【阻塞，不动了】

【异步线程】
执行任务...
执行完毕！
调用 promise.set_value()   -----→ 【发信号！】

【主线程】
收到信号！
解除阻塞！
future.get() 返回结果！
```



###### Lambda

```cpp
int a = 10;

auto func = [a]() {
    // 可以使用外面的 a
    cout << a;
};
```

`[]`用来传入参数供函数内部访问


###### 指针类型转换

目标类型 变量名 = static_cast<目标类型>(要转换的变量);



###### 调用特殊指针类的构造函数

```cpp
using CallbackWrapper = std::shared_ptr<PromiseType>;

auto* cb_param_ptr = new CallbackWrapper(promise_ptr);
```

- 类用一个宏定义表示，后面加上括号传参就是调用对应的构造函数