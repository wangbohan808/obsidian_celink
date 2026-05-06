# 📅2026-04-23

## 初始化命令表

```cpp
    std::map<std::string, std::function<int(const std::string&)>> cmd_obj_api_table_;
```

- `std::map`：使用关联容器，实现红黑树，存储键值对
- 键是一个字符串
- 值是一个函数，`std::function`内部包装的是函数的结构；作用是声明一种特殊的函数类型


```cpp
    cmd_obj_api_table_[HAIER_Y_CLEAN_MODE]           = std::bind(&TaskControl::Cmd_YMop, &TASK_CTL_INS, std::placeholders::_1);
```

**使用命令表的意义：**

✅ 统一接口：所有函数通过 std::function<int(const std::string&)> 统一调用
✅ 动态路由：通过键值查找自动分发到对应函数
✅ 避免冗余：不需要大量重复的if-else分支
✅ 易于维护：新增/修改命令只需要操作映射表


- `std::bind(...)` 本身不执行被绑定的函数。它做的事是：生成一个“可调用对象”（binder 对象），真正执行发生在你“调用 bind 返回对象”的那一刻，也就是对它使用 `()` 时

```cpp
auto f = std::bind(&DevStatus::StatusForceReport, this, std::placeholders::_1);
StdStringMsg msg;
f(msg);  // 这一行才会调用 this->StatusForceReport(msg)
```

哪些类型可以作为 `std::function<R(Args...)>` 的值？

只要满足一句话：能用 `()` 调用，并且调用形式能匹配 `R(Args...)`（返回值可转换成 `R`，参数可用 `Args...` 传进去），就可以。

常见可装入 `std::function` 的类型：

- 普通函数指针
    
    - `void foo(int);`
    - `std::function<void(int)> f = foo;`
- 无捕获 lambda / 有捕获 lambda
    
    - `[](int x){...}` 或 `[this](const Msg& m){...}`
    - 捕获 lambda 不能隐式转函数指针，但可以进 `std::function`
- 函数对象（仿函数）
    
    - 自定义 `struct F { void operator()(int) const; };`
- `std::bind` 的返回对象
    
    - 你用的就是这种
- 成员函数指针（注意：不能“直接”放进去就能调用，通常要配合绑定对象）
    
    - `&DevStatus::StatusForceReport` 单独拿出来不够，还缺对象实例
    - 典型做法：用 `bind` 或 lambda 把 `this` 绑上，再放进 `std::function`
- `std::mem_fn` 返回的可调用对象（同样可塞进 `std::function`）
    
    - `std::mem_fn(&DevStatus::StatusForceReport)` 生成一个可调用包装，但调用时仍需要对象参数



## 执行函数

```cpp
int HaierAdapter::HaierWriteHandle(const std::string &cmd, const std::string &cmd_value) {
    std::lock_guard<std::mutex> locker(write_mutex_);
    LOGD("HaierWriteHandle %s", cmd.c_str());
    auto iter = cmd_obj_api_table_.find(cmd);
    if (iter != cmd_obj_api_table_.end()) {
        return iter->second(cmd_value);
    } else {
        LOGD("HaierWriteHandle cmd:%s not found in cmd_obj_api_table", cmd.c_str());
    }
    return -1;
}
```

- 传入命令名称与参数，验证之后进行执行

- cmd_obj_api_table_.find(cmd) 如果在键值对列表中找到了“对应的键”，就会反馈对应的键值的地址，通过这个地址，使用first和second就可以分别对应访问键和值


## 设备写操作的回调函数

- 接收海尔云端下发的设备控制指令，执行对应函数
- 返回结果到云端


```cpp
uhsd_s32 HaierAdapter::HaierWriteCallback(uhsd_devHandle devHandle, uhsd_s32 req_sn, const uhsd_char *property_name, const uhsd_char *property_value, const uhsd_char *traceId) {
    // 实现回调逻辑
    // 1. 使用 devHandle 来标识设备
    // 2. 记录 req_sn，用于响应
    // 3. 根据 property_name 和 property_value 进行属性设置
    // 4. 使用 traceId 进行链路跟踪
    // 返回状态码，例如 0 表示成功，非 0 表示失败

    LOGD("-----------HaierWriteCallback--------------");
    std::cout << "Device Handle: " << devHandle << std::endl;
    std::cout << "Request Serial Number: " << req_sn << std::endl;
    std::cout << "Property Name: " << (property_name ? property_name : "NULL") << std::endl;
    std::cout << "Property Value: " << (property_value ? property_value : "NULL") << std::endl;
    std::cout << "Trace ID: " << (traceId ? traceId : "NULL") << std::endl;

    int result = HAIER_ADPATER.HaierWriteHandle(std::string(property_name), std::string(property_value));
    LOGD("-----------HaierWriteCallback--------------:%d", result);
    return uhsd_dev_write_resp(devHandle, req_sn, result, 0, traceId);
}
```

uhsd_s32 HaierWriteCallback(
    uhsd_devHandle devHandle,           // 设备唯一标识符，区分不同设备
    uhsd_s32 req_sn,                    // 请求序列号，用于请求-响应匹配
    const uhsd_char *property_name,     // 命令名称，如"workCommand"、"powerLevel"
    const uhsd_char *property_value,    // 命令参数，如"start"、"3"
    const uhsd_char *traceId            // 链路追踪ID，用于日志追踪
)


## 云端数据反馈函数回调的注册

```cpp
    // 设置【写】设备 通知回调，开发者只有设置该回调，才能收到设备写属性通知
    res = uhsd_dev_set_write_cb(HaierWriteCallback);
    LOGW("uhsd_dev_set_write_cb res : %d", res);
```


# 📅2026-04-25

## C++中的容器
### 容器的定义

C++ STL（标准模板库）提供的**模板类**，用于**存储同类型数据集合**，内置了**增、删、查、遍历、排序**等现成方法。

### 容器的分类

1. 序列容器（按顺序存，像列表 / 数组）

按**存放顺序**排列数据：

- `vector`：**动态数组**（最常用，变长数组）
- `list`：双向链表
- `deque`：双向队列
- `array`：固定大小数组（STL 版）

2. 关联容器（按键查找，字典结构）

**key-value 键值对**，自动排序 / 哈希，适合查找：

- `map`：有序键值对（红黑树）
- `unordered_map`：**哈希键值对**（查找超快）
- `set`：唯一元素集合
- `unordered_set`：哈希集合

👉 你代码里的 `cmd_obj_api_table_.find()`

**`cmd_obj_api_table_` 就是 `map/unordered_map` 这类「关联容器」**

3. 容器适配器（包装出来的特殊容器）

基于上面容器封装，限制功能，实现特定逻辑：

- `stack`：栈（后进先出）
- `queue`：队列（先进先出）
- `priority_queue`：优先队列

### 容器的作用

- 原生数组 `int a[]` 缺点：
    
    - 大小固定、不能动态扩容
    - 增删麻烦、无查找方法
    
- STL 容器优点：
    - **模板化**：任意类型都能装（int、string、函数、对象）
    - **自带方法**：`find()`、`size()`、`clear()`、`push_back()`
    - **自动管理内存**、动态扩容
    - 迭代器统一遍历（你刚才看的 `iter` 就是容器配套工具）

## 采集-组包-上报

```cpp
std::map<int, std::function<int()>> cmd_map_table_;

void DevStatus::InitWorkCmdTable() {
    cmd_map_table_[DUST_COLLECTION]     = std::bind(&DevStatus::Cmd_StartDustCollection, this);
    cmd_map_table_[START_SWEEP]         = std::bind(&DevStatus::Cmd_StartTotalClean, this);
    cmd_map_table_[PAUSE_SWEEP]         = std::bind(&DevStatus::Cmd_SwitchPause, this);
    cmd_map_table_[CONTINUE_SWEEP]      = std::bind(&DevStatus::Cmd_ContinueClean, this);
    cmd_map_table_[RETURN_CHARGE]       = std::bind(&DevStatus::Cmd_StartCharge, this);
    cmd_map_table_[PAUSE_RETURN_CHARGE] = std::bind(&DevStatus::Cmd_PauseCharge, this);
    cmd_map_table_[LOCALIZE_ROBOT]      = std::bind(&DevStatus::Cmd_Seek, this);
    cmd_map_table_[FORWARD]             = std::bind(&DevStatus::Cmd_DirectionForward, this);
    cmd_map_table_[BACKWARD]            = std::bind(&DevStatus::Cmd_DirectionBack, this);
    cmd_map_table_[LEFT_TURN]           = std::bind(&DevStatus::Cmd_DirectionTurnLeft, this);
    cmd_map_table_[RIGHT_TURN]          = std::bind(&DevStatus::Cmd_DirectionTurnRight, this);
    cmd_map_table_[MANUAL_MODE_STOP]    = std::bind(&DevStatus::Cmd_DirectionStop, this);
    cmd_map_table_[EXPLORE_MAP]         = std::bind(&DevStatus::Cmd_ExploreMap, this);
    cmd_map_table_[EDGE_SWEEP]          = std::bind(&DevStatus::Cmd_EdgeMode, this);
    cmd_map_table_[RESET_FACTORY]       = std::bind(&DevStatus::Cmd_ResetFactory, this);
    cmd_map_table_[CLEAR_MAP]           = std::bind(&DevStatus::Cmd_ResetMap, this);
    cmd_map_table_[IDLE_MODE]           = std::bind(&DevStatus::Cmd_StopClean, this);
}
```

- 容器内部的数据类型，可以根据需要进行更改

```cpp
int DevStatus::Init() {
    InitWorkCmdTable();
    // 回充中状态（包含充电中、基站工作和暂停）
    property_switch_charge_converter_ = {
            {EVENT_WORK_STATUS_BACK_CHARGE, true},       
            {EVENT_WORK_STATUS_BASE_STATION, true},
            {EVENT_WORK_STATUS_BACK_CHARGE_PAUSE, false}, 
            {EVENT_WORK_STATUS_BASE_STATION_PAUSE, false},
            {EVENT_WORK_STATUS_CHARGING, false},          
            {EVENT_WORK_STATUS_FULL_CHARGING, false},
            {EVENT_WORK_STATUS_COLLECT_DUST, false},      
            {EVENT_WORK_STATUS_WASH_MOP, false}};

    staus_thread_.reset(new std::thread(std::bind(&DevStatus::DevStatusLoop, this)));
    EventWorkerInnerPtr worker_ptr(new EventWorker<const StdStringMsg &>(std::bind(&DevStatus::StatusForceReport, this, std::placeholders::_1)));
    IOT_SDK->Subscriber(TOPIC_SWEEP_DEV_STATUS, std::move(worker_ptr));
 
    return 0;
}
```


### 堆上对象与栈上对象

 1. **`new` = 创建对象 + 返回** **对象指针**

```cpp
new std::thread(...)
```
- 在 ** 堆（heap）** 上创建一个 `std::thread` 对象
- **返回值是：`std::thread*` 线程对象指针**
- 所以必须用**指针**来接收：
```cpp
std::thread* t = new std::thread(...);
```

2. **不带 `new` = 创建栈上对象 + 返回** **对象实体（值）**

```cpp
std::thread t( std::bind(...) );
```
- 在 ** 栈（stack）** 上创建对象
- **返回的是对象实体，不是指针**
- 不能用指针接收
- 生命周期到作用域结束就**自动销毁**

### 线程的相关知识点

```cpp
std::thread(std::bind(&DevStatus::DevStatusLoop, this))
```
- `std::thread` 构造函数一执行，线程立刻启动

```cpp
staus_thread_.reset
```
- 使用智能指针`status_thread_`的`reset`方法，后续可以直接使用`status_thread_`管理线程指针对象

等价为下面两种写法：
```cpp
staus_thread_ = std::make_shared<std::thread>(&DevStatus::DevStatusLoop, this);
```

```cpp
staus_thread_ = std::make_shared<std::thread>([this] { DevStatusLoop(); });
```


### 智能指针与普通指针的区别

> **智能指针**：
> 
> 是一个**包装类对象**，
> 
> 1. 拥有**自身方法**（`.` 调用：reset 等）
> 2. 重载`->`/`*`，**能像普通指针一样操作目标**
> 3. **自带自动内存回收**，安全

> **普通指针**：
> 
> 是**纯地址**，
> 
> 4. 无自身方法，无 `.`
> 5. 只能`->/*`操作目标
> 6. 不管理内存，需要手动 delete

```cpp
std::thread* ptr = new std::thread(...);

ptr->join(); // 指针只能用 ->
```

```cpp
unique_ptr<std::thread> ptr(new std::thread(...));

ptr.reset();   // 对象用 . 调用自己的方法
ptr->join();   // -> 是“解引用”，访问内部的线程
```





# 📅2026-04-26

## 智能指针管理模板类实例

```cpp
EventWorkerInnerPtr worker_ptr(new EventWorker<const StdStringMsg &>(std::bind(&DevStatus::StatusForceReport, this, std::placeholders::_1)));
```

`std::bind` 结果：是一个对象（内部保存了成员函数指针 + `this` + 占位符规则），将来被调用时才执行真正的成员函数



## 成员初始化列表

```cpp
class T {
  int a_;
  const int b_;
  int& c_;
  std::function<int(int)> f_;

public:
  T(int a, int b, int& c, std::function<int(int)> f)
    : a_(a)              // 用参数 a 初始化成员 a_
    , b_(b)              // 用参数 b 初始化 const 成员 b_
    , c_(c)              // 用引用 c 绑定到成员 c_
    , f_(std::move(f))   // 用 f 的右值来初始化/移动 f_
  {
      // 构造函数体
  }
};
```

- 类名+括号，首先想到构造函数


## 移动语义-右值引用

```cpp
template<typename MsgType>
class EventWorker : public EventWorkerBase
{
 private:
   typedef std::function<int32_t(const MsgType&)> SubFunc;
   SubFunc subscriber_func_;
 public:
   EventWorker(SubFunc func) : subscriber_func_(std::move(func)) {};
   int32_t Call(const MsgType& in_msg) {
        return subscriber_func_(in_msg);
   };
};
```

- 有名字、你能对它取址（大多数情况） 的东西，通常是左值（lvalue）。
- 临时对象、字面量、返回的非引用临时值 通常是纯右值（prvalue），常常表现为“没名字的临时结果”。


## 回调-订阅机制

```cpp
int DevStatus::Init() {
    InitWorkCmdTable();
    // 回充中状态（包含充电中、基站工作和暂停）
    property_switch_charge_converter_ = {
            {EVENT_WORK_STATUS_BACK_CHARGE, true},       
            {EVENT_WORK_STATUS_BASE_STATION, true},
            {EVENT_WORK_STATUS_BACK_CHARGE_PAUSE, false}, 
            {EVENT_WORK_STATUS_BASE_STATION_PAUSE, false},
            {EVENT_WORK_STATUS_CHARGING, false},          
            {EVENT_WORK_STATUS_FULL_CHARGING, false},
            {EVENT_WORK_STATUS_COLLECT_DUST, false},      
            {EVENT_WORK_STATUS_WASH_MOP, false}};

    staus_thread_.reset(new std::thread(std::bind(&DevStatus::DevStatusLoop, this)));
    EventWorkerInnerPtr worker_ptr(new EventWorker<const StdStringMsg &>(std::bind(&DevStatus::StatusForceReport, this, std::placeholders::_1)));
    IOT_SDK->Subscriber(TOPIC_SWEEP_DEV_STATUS, std::move(worker_ptr));
 
    return 0;
}
```

- 传入一个"可操作的对象"作为回调函数，后续会使用这个初始化的操作句柄给这个回调函数传入参数再实际调用

- 以 `TOPIC_SWEEP_DEV_STATUS` 作为事件/主题名（`const char*`）
- 把封装好的 `EventWorker`（基类指针 `EventWorkerBase*` 形式）注册进 SDK 的订阅表里保存起来
- 之后当该 event 有消息/事件需要处理时，SDK 会取出对应的 `EventWorker` 去调用其 `Call(...)`，从而最终触发你 `std::function` 里保存的回调

# 📅2026-04-27

## 线程告警上报



# 📅2026-04-28

## 线程循环函数

```cpp
void DevStatus::DevStatusLoop() {
    while (true) {
        bool  b_flag = false;
        request_signal_.PopSignal(b_flag, 300);
        UpdateDevAllStatus(b_flag);
    }
}
```

**其中调用了如下函数：**

```cpp
int32_t PopSignal(T& signal_out, int32_t timeout = -1) {
    // 查看是否有存货
    queue_mutex_.lock();
    if (event_queue_.size() > 0) {
        signal_out = event_queue_.front();
        event_queue_.pop();
        is_set_flag_ = false;
        queue_mutex_.unlock();
        return 1;
    }
    queue_mutex_.unlock();

    // 无存货，等新货
    bool signal_status = false;
    std::unique_lock<std::mutex> lck(cond_mutex_);
    if (timeout == -1) {
        // 无超时时间，永久等待
        condition_var_.wait(lck);
    } else {
        if(!is_set_flag_.load()) {
            // 等待新的消息,直到超时
            // 注: 只有当lanbada 表达式中的返回值为true时,wait才会提前退出;
            // 陷阱：当在进入等待状态前，通知发生，会导致通知丢失了，即条件变量仅在接收方处于等待状态时才发送通知。
            // std::condition_variable存在两个缺陷：虚假唤醒 和 唤醒丢失
            // 为解决以上两个问题，采取如下方案：每间隔100ms或收到信号时 查看下是否收到了信号，若收到了则退出，若没收到则继续wait_for，直到timeout
            int32_t once_timeout     = 0;
            int32_t once_timeout_max = 100;
            while (timeout > 0) {
                if (timeout > once_timeout_max) {
                    timeout      = timeout - once_timeout_max;
                    once_timeout = once_timeout_max;
                } else {
                    once_timeout = timeout;
                    timeout      = 0;
                }

                signal_status = condition_var_.wait_for(lck, std::chrono::milliseconds(once_timeout),
                                                        [&] { return is_set_flag_ == true; });
                
                if (signal_status) {
                    if (is_set_flag_.load() || event_queue_.size() > 0) {
                        break;
                    }
                }
                
                // debug log by zzx
                // printf("222 PopSignal ignore wait_for. is_set_flag:%d event_queue:%d signal_status:%d once_timeout:%d\n", is_set_flag_.load(), event_queue_.size(), signal_status, once_timeout);
            }

        } else {
            signal_status = true;
        }
    }

    // 来货了
    if (signal_status == true) {
        queue_mutex_.lock();
        if (event_queue_.size() > 0) {
            signal_out = event_queue_.front();
            event_queue_.pop();
            queue_mutex_.unlock();
            is_set_flag_ = false;
            return 1;
        }
        queue_mutex_.unlock();
    }

    // debug log by zzx
    // printf("333 PopSignal ignore wait_for. is_set_flag:%d event_queue:%d signal_status:%d timeout:%d\n", is_set_flag_.load(), event_queue_.size(), signal_status, timeout);
    is_set_flag_ = false;
    return -1;
}
```

- 队列中如果有数据，就直接将数据给出去，并解锁


- 调用函数时传入的参数，决定后续进入的分支
- 调用 PopSignal 时，传入的 timeout 不是 -1，就会进 else
- `-1`是永久等待，其余的是超时等待


- `!is_set_flag_.load()` 如果是false就继续进入等待
- 如果是`true`就进入另外一个分支

- 弹出队列中储存的元素，而队列中储存的元素正是bool值


### 锁防止数据的竞争

下面三种函数，不同的线程之间可能会调用，造成这组队列的数据之间的竞争；因为访问这组队列数据的方法只有以下三种方法，因而下面三种方法加上同一把锁就好了

所有会动这个队列（或在其上做强一致读写的）路径，都用的是同一把 `queue_mutex_`：

|接口|对 `queue_mutex_` 的使用|
|---|---|
|`PushSignal`|`lock` → 可能 `pop`/`push` → `unlock`|
|`Size`|`lock` → 读 `size()` → `unlock`|
|`PopSignal`|开头快路径（59–67）、以及被唤醒后再取货（115–123）|

因此：多个线程无论调用 Push、Pop 还是 Size，只要碰到队列，都争用同一把 `queue_mutex_`，从而避免对 `event_queue_` 的并发读写造成数据竞争。  
这就是你说的“锁防止线程间数据竞争”在这里的具体体现：共享数据结构 = `event_queue_`，互斥量 = `queue_mutex_`



### 锁的封装

```cpp
std::unique_lock<std::mutex> lck(cond_mutex_);
```

- **`std::mutex`**
    
    C++ 标准**互斥锁**，作用：**同一时间只允许一个线程加锁**，防止多线程同时修改数据。
    
- **`std::unique_lock`**
    
    C++ 标准**锁管理工具**（不是锁本身！是锁的**管理员 / 包装器**）。
    
    它负责帮你自动上锁、解锁，不用手动写 `lock()`/`unlock()`。
    
- **`lck`**
    
    你定义的**锁管理对象**名字。
    
- **`(cond_mutex_)`**
    
    把 `cond_mutex_` 这个互斥锁交给 `unique_lock` 管理

