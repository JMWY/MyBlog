# 单例模式

单例模式是指整个运行的系统中**最多只能有一个**类的实例。  
因为单例模式固有的特性————仅一个实例，所以可以把这部分特性独立出来，写成基类。然后把具体的接口操作可以放在其派生类中。

* [饿汉式分析与实现](#1)  
* [懒汉式分析与实现](#2)  
　　[堆上分配空间](#2.1)    
　　[栈上分配空间-不加锁的懒汉式](#2.2)   
* [各种实现的对比](#3)


<h2 id="1"></h2>

## 饿汉式的分析与实现

饿汉式在类装载时就实例化一个静态实例，所以天生是多线程的。  
以下实现一个饿汉类的基类：

```c
template<typename T>
class Singleton {
public:
    static T *Instance() { return _ins; }

protected:
    Singleton() { garbo;}
    virtual ~Singleton() { } 

private:
    class Garbo {
    public:
        ~Garbo() {
            if (Singleton<T>::_ins) {
                delete Singleton<T>::_ins;
                Singleton<T>::_ins = NULL;
            }   
        }    
    };  
    static T*  _ins;
    static Garbo garbo;
};
template<typename T> T *Singleton<T>::_ins = new T();
template<typename T> typename Singleton<T>::Garbo Singleton<T>::garbo;

```

其中，
- 实例是分配在堆上的，所以需要用户管理其空间的释放。
- 定义一个静态实例 garbo, 保证实例所占内存空间的释放。
- 构造函数中，调用下 garbo, 保证了类对象 garbo 的存在(否则，不会创建对象 garbo, 因为程序中并没有引用的话就不会创建)。


----------------------------------------------------------------
<h2 id="2"></h2>

## 2. 懒汉式分析与实现

<h3 id="2.1"></h3>

### 堆上分配空间

实现一：
```c
template<typename T>
class Singleton {
public:
    static T *Instance();

protected:
    Singleton() { garbo; }

private:
    class Garbo {
    public:
        ~Garbo() {
            if (Singleton<T>::_ins) {
                delete Singleton<T>::_ins;
                Singleton<T>::_ins = NULL;
            }
        }
    };
    static T *_ins;
    static pthread_mutex_t mutex;
    static Garbo garbo;
};
template<typename T> T* Singleton<T>::_ins = NULL;
template<typename T> pthread_mutex_t Singleton<T>::mutex = PTHREAD_MUTEX_INITIALIZER;
template<typename T> typename Singleton<T>::Garbo Singleton<T>::garbo;

template<typename T>
T* Singleton<T>::Instance()
{
    if (NULL == _ins) {
        pthread_mutex_lock(&mutex);
        if (NULL == _ins) {
            _ins = new T();
        }
        pthread_mutex_unlock(&mutex);
    }
    return _ins;
}

```

实现二：
```c
template<typename T>
class Singleton {
public:
    static T *Instance();

protected:
    Singleton() {}
    ~Singleton() { }

private:
    struct SingletonContainer {
        T *_ins;
        SingletonContainer(): _ins(NULL) {}
        ~SingletonContainer() { delete _ins; }
    };  
    static pthread_mutex_t mutex;
    static SingletonContainer SC; 
};
template<typename T> pthread_mutex_t Singleton<T>::mutex = PTHREAD_MUTEX_INITIALIZER;
template<typename T> typename Singleton<T>::SingletonContainer Singleton<T>::SC;

template<typename T>
T* Singleton<T>::Instance()
{
    if (NULL == SC._ins) {
        pthread_mutex_lock(&mutex);
        if (NULL == SC._ins) {
            SC._ins = new T();
        }
        pthread_mutex_unlock(&mutex);
    }
    return SC._ins;
}

```


<h3 id="2.2"></h3>

### 栈上分配空间

```c
#define DISALLOW_COPY_AND_ASSIGN(TypeName) \
        TypeName (const TypeName&);        \
        TypeName& operator= (const TypeName&)

template<typename T>
class Singleton{
public:
    static T* Instance(){
        static T instance;
        return &instance;
    }   
protected:
    Singleton() {}
    virtual ~Singleton() { }

private:
    DISALLOW_COPY_AND_ASSIGN(Singleton);
};

```

------------------------------------------------------------------
<h2 id="3"></h2>

## 3. 各种实现的对比

