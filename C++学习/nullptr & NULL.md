## 什么是 NULL？
`NULL` 是一个宏，它被定义为 `0`（也就是 `int` 型的零），或者 `0L`（`long` 型的零），或者其他**零指针常数**；但是一般来说我们还是把它当作整形来看待。零指针常数之所以得名，是因为他们能够被转化为空指针。在 C 中，`NULL` 也可以被理解为 `(void*)0`，因为 `void` 类型的指针都可以被转化为**其他任何指针类型**。详情请看文章 [C++干货系列——起底万能指针void*](https://zhuanlan.zhihu.com/p/163676489)。

虽然 `NULL` 很明显是为了指针被制造出来的，但是如果你把它当作参数传到函数中去，它就会“原形毕露”——编译器会把它当作一个 `int` 类型，或一个 `long` 类型，而不是一个指针类型。比如说下边的例子：
```cpp
 class Girlfriend {/* ... */};
 ​
 void kissGirlfriend(Girlfriend* gf);
 void kissGirlfriend(int gfId);
 ​
 int main() {
   // 参数匹配后，其实调用的是第二个函数，你的第零号女朋友莫名其妙被亲了一下
   kissGirlfriend(NULL);
 }
```
从上边代码来看，我们很明显是想传一个空指针进去，让程序调用第一个重载函数，但很可惜的是，这种情况下我们唯一能确定的事情就是我们的预想肯定不会发生——可能有两个结果:

- 如果`NULL`被定义为`0`（`int`零），那么编译器会自然而然调用第二个重载函数，毕竟这是一个完美参数匹配。
- 如果`NULL`被定义为`0L`或者其他形式的整型零，编译器会报错，告诉你有一个指向不明的函数调用，——因为`0L`可以被等同地转化为空指针或`int`，由此就产生了二义性。

让我们看看这个问题怎么解决。第一个尝试：我们可以试着让一个 `enum` 来代替那个函数的 `int`，然后去消除二义性。同时让我们给参数一个名字，让代码看起来指向性也更明确一些：
```cpp
class Girlfriend {/* ... */};
 ​
 enum GirlfriendId {/* ... */};
 void kissGirlfriend(Girlfriend* gf);
 void kissGirlfriend(GirlfriendId gfId);
 ​
 int main() {
   auto noGf = NULL;
   kissGirlfriend(noGF); //还是ERROR
 }
```
在这里，`noGf` 不是指针，是一个正儿八经的整形变量，另一方面，从整型零到空指针的转化只能发生在零指针常数上。所以编译器很无奈的告诉我们，他不知道该把 `noGf` 转化为一个 `GirlfriendId` 还是 `Girlfriend*`。

**`NULL`的弊端**

上边两个例子都向我们展示了问题所在：`NULL` 终究只是一个宏。它是一个整型，它不是指针，所以一旦涉及类型转换就会有风险。因而随之带来的问题是，我们没有办法在不显示声明指针类型的情况下定义一个空指针。
## nullptr
自从 C++11以来，有一个小特性完美的解决了这个问题—— `nullptr`。作为一个字面常量和一个零指针常数，它可以被隐式转换为**任何指针类型**。我们只需稍作修改上边的例子，你就会得到：

```cpp
 void kissGirlfriend(GirlFriend* gf);
 void kissGirlfriend(int gfId);
 ​
 int main() {
   kissGirlfriend(nullptr);  //调用第一个重载函数
 }
```

这时编译器就会乖乖听话了：因为`nullptr`是没办法转换成`int`的，它会隐式转化为一个空的`Girlfriend*`从而正确的函数会被调用。下边这个例子对`nullptr`的类型进行说明：

```cpp
 class Girlfriend {/* ... */};
 ​
 enum GirlfriendId {/* ... */};
 void kissGirlfriend(Girlfriend* gf);
 void kissGirlfriend(GirlfriendId gfId);
 ​
 int main() {
   auto noGf = nullptr;
   kissGirlfriend(noGF); //OK
 }
```

`nullptr`有它自己的类型，`std::nullptr_t`，他可以隐式转换为任何指针类型。所以上例中`noGf`现在有了std::`nullptr_t`类型并且可以转换为`Girlfriend*`，而不是`GirlfriendId`。所以，无论在哪里，都请使用`nullptr`，而不是`NULL`、`0`或者其他零指针常数。
## **`nullptr` 和智能指针**

智能指针并不是真正的指针，他们是**类**。所以上边所有的隐式转化对诸如`shared_ptr<T>`等智能指针都不会奏效。但很幸运的是，对于`nullptr`自己的类型，智能指针类都重载了针对这种类型的**构造器**，所以下边的代码其实是合法的：

```cpp
 shared_ptr<Girlfriend> gfPtr = nullptr;    //隐式构造shared_ptr
 unique_ptr<Boyfriend> bfPtr = nullptr;    //隐式构造unique_ptr
```

需要注意的是，除了从`auto_ptr`到`unique_ptr`的转换，这是唯一一种智能指针的隐式构造函数。因此，当一个函数的参数类型是智能指针类型时，理论上你也可以直接传一个`nullptr`进去，而不必画蛇添足地额外用`nullptr`创建一个智能指针出来：

```cpp
 void doSomething(unique_ptr<Object> objectPtr);
 ​
 int main() {
   doSomething(nullptr); //OK
 }
```
### **向后兼容**

更妙的是，C++11标准规定了任何**零指针常量都，**如`0、NULL`等，都可以隐式转化为`nullptr`，这让我们能够有效地兼容使用`NULL`的老代码。也就是说，如果你将`nullptr`和`nullptr_t`引入老代码，代码将会很好地兼容以前的逻辑。尤其是当我们向03版及之前的代码中引入智能指针时，我们更能体会到这种兼容特性的妙处：

```cpp
 //C++03 version:
 void doSomething(Object* objectPtr) {
   //...
   delete objectPtr;
 }
 ​
 int main() {
   doSomething(NULL);
 }
```

我们通过把原始的指针用`unique_ptr`来替换时，如果某处忘记了把`NULL`替换成`nullptr`怎么办？别担心，这样不会引起错误：

```cpp
 //引入 unique_ptr
 void doSomething(unique_ptr<Object> objectPtr) {
   //...
 }
 ​
 int main() {
   doSomething(NULL);  // OK
 }
```

原因就在于，`NULL`是一个零指针常数，可以隐式转换为`nullptr`，而`nullptr`又可以调用`unique_ptr<T>`的隐式构造器，通过两次隐式转换，我们就可以完美完成上边的调用。