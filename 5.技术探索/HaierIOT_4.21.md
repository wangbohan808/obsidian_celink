学习日期：2026-04-21 星期二
主题关键字：
关键待办：

---

###### 宏定义的特殊使用

\#define 宏名(参数列表)    替换内容
```cpp
#define SAFE_COPY(dest, src)    \
    strncpy(...);               \
    dest[...] = '\0'
```

- 第 1 部分：`SAFE_COPY`（宏名字）
- 第 2 部分：`(dest, src)`（宏的参数）
- 第 3 部分：`strncpy(...); dest[...] = '\0'`（要替换的代码）


###### 拷贝构造与赋值

```cpp
Singleton(const Singleton&);
Singleton s1;

Singleton s2 = s1;  // ✅ 创建新对象 → 拷贝构造
```

- **s1 是一个对象（实体）**
- **但传给函数时，编译器会自动把它绑定到引用上**

```cpp
class Singleton {
public:
    // 赋值运算符重载
    Singleton& operator=(const Singleton& other);
};

// 实现（随便写点意思意思）
Singleton& Singleton::operator=(const Singleton& other) {
    return *this;
}


Singleton s1;   // 创建对象 s1
Singleton s2;   // 创建对象 s2

s2 = s1;        // ← 这一行，就是调用赋值重载
s2.operator=(s1);
```

- 原本的s1与s2都不是引用，但是从实参变为形参的时候，自动转化为引用

###### 单例模式模板类

```cpp
template<typename T>
class Singleton
{
public:
	static T& Instance()
	{
		//pthread_once(&ponce_, &Singleton::init);
		if (NULL == value_)
		{
			value_ = new T();
		}
		return *value_;
	}

private:
	Singleton();
	~Singleton();

	Singleton(const Singleton&);
	Singleton& operator=(const Singleton&);

	static void init()
	{
		value_ = new T();
		//::atexit(destroy);
	}

	static void destroy()
	{
		delete value_;
	}

private:
	//static pthread_once_t ponce_;
	static T*             value_;
};

//template<typename T>
//pthread_once_t Singleton<T>::ponce_ = PTHREAD_ONCE_INIT;

template<typename T>
T* Singleton<T>::value_ = NULL;
```

1. 静态成员变量必须在类外进行初始化；
2. `private`限制的是对于变量的访问，但是定义必须在类外进行


```cpp
typedef Singleton<AreaInfo> sAreaInfo;
#define AREA_INFO_INS sAreaInfo::Instance()
```

类模板本身不是真正的类，只有第一次被传入具体类型（如 Singleton<AreaInfo>）时，编译器才会自动生成对应的 “模板类”，并分配对应的内存布局 —— 你不需要手动定义。
