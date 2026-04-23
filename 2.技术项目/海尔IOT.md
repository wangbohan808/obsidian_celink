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