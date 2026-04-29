# 📅2026-04-28

## 类构造相关

```python
class MainFrame(wx.Frame):
```
- 定义一个`MainFrame`类，继承`wx.Frame`这个父类

```python
def __init__(self, parent):
```
- 在进行类的初始化的时候，需要传入一个参数
- `python`中的`__init__`类似于`c++`中的构造函数？

```python
window = MainFrame.MainFrame(parent=None)
```
- 创建类的对象，自动调用`python`的`__init__`函数


## python中的函数调用

```python
from test_tool import test
```
- 可以导入项目文件夹下的文件，作为一个模块，直接使用模块内的函数
- 在 Python 里，“属于类的”叫方法（`def foo(self): ...`），通过 `obj.foo()` 调用

```python
test.load_config()
```


## config.yaml配置文件

**配置文件的使用流程：**

```python
if __name__ == '__main__':
    print_hi('海能机器人')
```

```python
window = MainFrame.MainFrame(parent=None)
```

```python
from test_tool import test

test.load_config()
```

### **加载配置文件：**

```python
def load_config():
    # 读配置文件

    # yaml_file_path = 'config.yaml'
    # 读取并打印YAML文件内容
    yaml_file_path = resource_path('config.yaml')   # 使用 resource_path
    config = read_yaml(yaml_file_path)
    print(type(config), config)
    load_cfg.com = config.get('user_com', "")  # config['user_com']
    load_cfg.dev = config['device_type']
    load_cfg.mcu_ver = config.get('mcu_version', "")  # config['mcu_version']
    load_cfg.test_tool = config.get('test_tool', "治具未编码")  # 测试工具编码，暂不使用（海能mes）
    load_cfg.mes = config.get('use_mes', "3")  # 使用安克mes
    load_cfg.parts_sn_head = config.get('parts_sn_head', "")  # config['parts_sn_head']
    load_cfg.project_name = config.get('project_name', "C10B ")
    print(load_cfg.parts_sn_head)

    if is_com_port(load_cfg.com) is False:
        print("配置串口端口非法：" + load_cfg.com)
        load_cfg.com = ""
    else:
        print("配置串口端口为：" + load_cfg.com)

    if int(load_cfg.mes) < 1 or int(load_cfg.mes) > 3:
        print("mes配置异常：" + str(load_cfg.mes))
        load_cfg.mes = str(load_cfg.mes)
    else:
        load_cfg.mes = '002'
```


**获得绝对路径的函数：**
```python
def resource_path(relative_path):
    """获取资源文件的绝对路径，兼容开发和打包"""
    try:
        # PyInstaller 创建的临时目录
        base_path = sys._MEIPASS
    except AttributeError:
        base_path = os.path.abspath(".")
    return os.path.join(base_path, relative_path)
```

- 输入相对路径，拼接输出绝对路径

**打开文件：**

```python
def read_yaml(file_path):
    with open(file_path, 'r', encoding='utf-8') as file:
        config = yaml.safe_load(file)
    return config
```
- 以`utf-8`格式的编码，打开文件
- 使用第三方的`yaml`库加载储存文件


**根据键获取字典中对应的值：**

YAML 本质就是一个带结构、易阅读的字典；
YAML的定位就是人类最容易读写的配置文件格式，能完美映射成：**字典（dict）+ 列表（list）**，几乎所有语言（Python/Go/Java/JS）都能轻松解析


在字典 `config` 里取值常见有两种写法：

- `config['key']`：强制取值
    
    - 键必须存在，否则立刻抛 `KeyError` 报错
    - 适合 必须配置/缺了就不能运行 的字段
- `config.get('key', 默认值)`：安全取值
    
    - 键不存在也不会报错，会返回你给的默认值
    - 适合 可选配置/缺省时用默认行为 的字段


**配置为全局变量之后，后续可以直接调用**


## 代码中的两个线程

```python
window = MainFrame.MainFrame(parent=None)
```
- 初始化ui对象的时候，会创建两个测试线程

```python
mythread.start_work_thread()
mythread.start_serial_thread()
```
- 导入的`mythread.py`文件作为模块，直接调用模块内的函数，运行线程


**创建管理线程的句柄,以及停止线程的标志位：**
```python
work_thread = None
serial_thread = None
# 创建一个事件对象，用于通知线程停止
work_stop_event = threading.Event()
serial_stop_event = threading.Event()
```


```python
def thread_test_running():
    while not work_stop_event.is_set():
        test.test_run_process()
    print("Worker thread stopped.")
```

- **线程让你把一段代码放到另一个并发执行的上下文中运行；运行一次还是循环运行，取决于你在线程入口函数里怎么写（有无 `while`、是否阻塞、是否可取消等）**

```python
def start_work_thread():
    global work_thread
    if work_thread is None or not work_thread.is_alive():
        work_stop_event.clear()
        work_thread = threading.Thread(target=thread_test_running)
        work_thread.start()
```
- 声明一次这个变量为全局的变量，否则代码会以为后续的是局部的变量
- 如果线程已经终止或者从未创建：清除状态位、将函数加载到线程、开始运行线程


```python
def stop_work_thread():
    work_stop_event.set()
    if work_thread is not None:
        work_thread.join()
```
- 设置终止进程标志位；结束函数所在的线程检测后阻塞等待，直到所检测线程彻底结束





