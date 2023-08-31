# 深入剖析 std::thread
在 `g++` 中，`thread` 是基于 `pthread` 实现的。本次主要从以下三个方面分 `std::thread`：

- `std::thread`对象不可复制，只具有移动属性
- 每个线程具有唯一的标志，即线程id
- 创建子线程
## 移动属性
有很多书籍说，`std::thread` 对象的所有权只能传递不能复制。实际上，就 `std::thread` 对象，只具有移动属性，不具有复制属性。`std::thread` 的构造函数如下：

```cpp
class thread {
  private:
    id              _M_id;
  public:
    thread() noexcept = default;

    template<typename _Callable, 
              typename... _Args,
              typename = _Require<__not_same<_Callable>>>
    explicit thread(_Callable&& __f, _Args&&... __args) {
          //...
    }

    ~thread() {
      if (joinable())
        std::terminate();
    }
    // 禁止复制
    thread(const thread&) = delete;
    thread& operator=(const thread&) = delete;

    // std::thread 只具有移动属性
    thread(thread&& __t) noexcept
    { swap(__t); }

    thread& operator=(thread&& __t) noexcept {
      if (joinable())
          std::terminate();
      swap(__t);
      return *this;
    }
    //...
  }
```

可以发现，`std::thread`禁止了复制构造函数、复制赋值表达式，只留下了移动构造函数、赋值，使得`std::thread`对象只能移动，不能复制。这就是本文开篇demo中使用`emplace_back`函数添加`std::thread`对象的原因，防止触发复制构造函数。

向`threadList`中添加`std::thread`对象，有如下三种方式：

```cpp
  threadList.emplace_back(std::thread{do_some_work, idx});  // 1) ok 

  std::thread trd{do_some_work, idx};
  threadList.push_back(trd);               // 2) error
  threadList.push_back(std::move(td));     // 3) ok
  threadList.emplace_back(std::move(td));  // 4) ok
```

注意：当`push_back`接受的是右值时，底层调用的还是`emplace_back`函数，因此，`3)`和`4)`算是等价。

### std::thread::id

观察可发现，在`std::thread`对象中，只有一个成员变量`_M_id`：

```cpp
   id             _M_id;
```

这个类`id`全称是`std::thread::id`，实现如下：

```cpp
  typedef pthread_t native_handle_type;

  class id { 

    native_handle_type  _M_thread;  // _M_thread 即 pthread_t 对象，线程的唯一辨识标志
  public:
    id() noexcept : _M_thread() { }  // _M_thread 默认值是 0
    explicit id(native_handle_type __id) : _M_thread(__id) { }
  private:
    friend class thread;
    friend class hash<thread::id>;

    // 为 std::thread::id 对象重载了 == 运算
    friend bool operator==(thread::id __x, thread::id __y) noexcept;
    friend bool operator<(thread::id __x,  thread::id __y) noexcept;
    // 为 std::thread::id 对象重载了 << 操作
    template<class _CharT, class _Traits>
    friend basic_ostream<_CharT, _Traits>&
    operator<<(basic_ostream<_CharT, _Traits>& __out, thread::id __id);
  };
```

因此，这个`std::thread::id`实际上，就是封装了`pthread_t`对象，用作每个线程标志。

- 在构造`std::thread`对象的时候，如果没有设置线程入口函数，则线程`_M_id._M_thread`的值是0。

比如下面的demo中，`trd`没有设置线程入口函数，`trd`调用默认构造函数时，`trd`的`_M_id._M_thread`会被初始化为0。

```cpp
    int main(int argc, char const *argv[]) {

      std::thread trd;
      std::cout<<trd.get_id()<<std::endl;
      return 0;
    }
```

但是，打印线程标志`trd.get_id()`，输出的是却不是0。这仅仅是`std::thread::id`在重载`<<`操作符时的设定，用于提示调用者线程没有启动。

```cpp
    $ g++  thread_.cc -o thread_ && ./thread_
    thread::id of a non-executing thread
```

可以到`std::thread::id`重载的`<<`操作符的函数中一探究竟：

```cpp
    template<class _CharT, class _Traits>
    inline basic_ostream<_CharT, _Traits>& operator<<(basic_ostream<_CharT, _Traits>& __out, thread::id __id) {
      // 线程未启动 
      if (__id == thread::id())
        return __out << "thread::id of a non-executing thread";
      // 线程成功启动
      else
        return __out << __id._M_thread;
    }

    // id的相等判断 
    inline bool operator==(thread::id __x, thread::id __y) noexcept {
        return __x._M_thread == __y._M_thread;
    }
```

因此，判断一个线程是否启动，可如下检测：

```cpp
    bool thread_is_active(const std::thread::id& thread_id) { 
        return thread_id != std::thread::id();
    }
```

- 设置了线程入口函数，`_M_id._M_thread`才会有值显示。

```cpp
    int main(int argc, char const *argv[]) {

      std::thread trd{[]{std::cout<<"wok in sub-thread\n";}};

      std::cout<<trd.get_id()<<std::endl;
      trd.join();
      return 0;
    }
```

输出的是：

```cpp
    $ g++  thread_.cc -o thread_ -lpthread && ./thread_
    139794901763840
    wok in sub-thread
```

当设置了显示入口函数时，`_M_id._M_thread`才是线程的`tid`值，由`pthread_create(&tid, NULL, ...)`函数设置。

**by the way**

在创建`std::thread`对象`trd`时，如果设置了线程入口函数，那么就必须使用`trd.join()`或者`trd.detach()`来表达子线程与主线程的运行关系，否则在`std::thread`对象析构时，整个程序会被`std::terminate()`中止。

没有设置线程入口函数，`trd.joinable()`返回值就是`false`，因此不会触发`std::terminate()`。

```cpp
  ~thread() {
    if (joinable())
      std::terminate();
   }
```

### 创建子线程

当构造`std::thread`对象时，设置了线程入口函数，会在相匹配的构造函数里调用`pthread_create`函数创建子线程。先看整体实现：

```cpp
   // std::thread 构造函数
    template<typename _Callable, 
             typename... _Args,
             typename = _Require<__not_same<_Callable>>>
    explicit thread(_Callable&& __f, _Args&&... __args)
    {
        static_assert( __is_invocable<typename decay<_Callable>::type, 
                                      typename decay<_Args>::type...>::value,
                      "std::thread arguments must be invocable after conversion to rvalues");

        // Create a reference to pthread_create, not just the gthr weak symbol.
        auto __depend = reinterpret_cast<void(*)()>(&pthread_create);
        // 启动线程
        _M_start_thread(_S_make_state(__make_invoker(std::forward<_Callable>(__f), 
                                                     std::forward<_Args>(__args)...)),
                        __depend);
    }
```

再细看构造函数执行流程：

1. 在编译期判断构造`std::thread`对象时设置的线程入口函数`__f`及其参数`__args`能否调用。

比如，下面的demo中，线程入口函数`thread_func`有个`int`类型的参数`arg`，如果传入的参数`__args`无法隐式转换为`int`类型，或者没有设置`__args`，都会触发`std::thread`构造函数中的静态断言`static_assert`，报错：**error: static assertion failed: std::thread arguments must be invocable after conversion to rvalues** 。

```cpp
    void thread_func(int arg) {   }

    int main(int argc, char const *argv[]) {
        std::thread trd_1{thread_func, "str"};  // arg类型不对
        std::thread trd_2{thread_func};	 	   // 缺少 arg

        // ...
        return 0;
    }
```

2. 将线程入口函数__f及其参数__args进一步封装起来。

这里是使用`__make_invoker`完成的：

```cpp
  __make_invoker(std::forward<_Callable>(__f), std::forward<_Args>(__args)...);
```

`__make_invoker`的作用是返回一个`_Invoker`对象，`_Invoker`是个仿函数，通过`_Invoker()`就可以以指定的参数`__args`直接执行线程入口函数`__f`。类似于`std::bind`：

```cpp
     void print_num(int i) {
         std::cout << i << '\n';
     }

     int main(int argc, const char* argv[]) {
       // wrapper
       auto invoker =  std::bind(print_num, -9);
       // 直接调用 invoker() 就可以以指定参数 -9 调用 print_num
       invoker();
     }
```

3. 启动子线程

在调用`_M_start_thread`函数启动子线程前，执行过程：创建 `_State_ptr`的对象，来封装`_Invoker`对象，再传递给`_M_start_thread`函数。这个过程，由`_S_make_state`函数完成，`_S_make_state`最终返回`_State_ptr`对象。

```cpp
       // 基类
       struct _State {
           virtual ~_State();          // 虚析构函数
           virtual void _M_run() = 0;  // 线程运行函数
       };
       using _State_ptr = unique_ptr<_State>;	// 父类指针
   
       // 子类
       template<typename _Callable>
       struct _State_impl : public _State {
           _Callable		_M_func;	// 线程入口函数
   		
           _State_impl(_Callable&& __f) : _M_func(std::forward<_Callable>(__f))
           { }
   
           void _M_run() { _M_func(); } // 执行线程入口函数
       };
   
       // 传入_Invoker对象，返回 _State_ptr 对象
       template<typename _Callable>
       static _State_ptr _S_make_state(_Callable&& __f)  {
           using _Impl = _State_impl<_Callable>;
           // 使用子类对象来初始化父类
           return _State_ptr{new _Impl{std::forward<_Callable>(__f)}};
       }
```

`_S_make_state`函数，将线程入口函数`__f`及其参数`__args`封装到`_State_ptr`对象`_State_ptr_obj`中， 这样最后可以通过`_State_ptr_obj->_M_run()`来调用`__f`。

下面到了 `_M_start_thread` 函数了：

```cpp
      void thread::_M_start_thread(_State_ptr state, void (*)())
      {
          const int err = __gthread_create(&_M_id._M_thread,
                                            &execute_native_thread_routine, // 线程执行函数
                                            state.get());
          if (err)
            __throw_system_error(err);
          state.release();
      }
    
      // 内部调用的是 pthread_create 函数
      static inline int __gthread_create(pthread_t *__threadid, void *(*__func) (void*), void *__args)
      {
          return pthread_create(__threadid, NULL, __func, __args);
      }
      // 内部执行线程入口函数
      static void* execute_native_thread_routine(void* __p)
      {
          thread::_State_ptr __t{static_cast<thread::_State*>(__p)};
          __t->_M_run();		// 运行线程入口函数
          return nullptr;
      }
```

因此，在执行完`_M_start_thread`函数后，才具有`_M_start_thread !=0`。

好，到此为此已实现了本文开篇提出的两个目标，下一篇将继续讲解线程间共享数据的访问