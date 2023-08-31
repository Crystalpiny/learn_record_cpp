# Cs


delete []x;//释放一维数组x的空间
关于二维数组
二维数组的形参必须指定第二维的大小，例如
a[][10]是合法的形参，a[][]就不是
克服上述限制的方法就是对二维数组使用动态内存分配
动态分配内存创建一个列数为5的二维数组

```c++
char(*c)[5]
try{c=new char[n][5]}
catch(bad_alloc)
{
	//仅当new失败时才会进入
	cerr<<"Out of memory"<<endl;
	exit(1);
}
```

如果在编译的时候，数组的列数也是未知的，就不可能仅调用一次new来  就能够创建这个二维数组 (即便是 数组的行数 在编译的时候是已知的)
要创建这样的二维数组，可以把它看作是 若干行构成的结构 每一行都是能用new创建的一维数组 指向每一行的指针保存在另一个一维数组之中
创建一个二维数组的例子

```c++
template<class T>
bool make2dArray(T**& x, int numberofRows, int numberofColumns)
{
	//创建一个二维数组
	try
	{
		//创建行指针
		x = new T * [numberofRows];
		//为每一行分配空间
		for (int i = 0; i < numberofRows; i++)
			x[i] = new int[numberofColumns];
		return true;
	}
	catch (bad_alloc) { return false; }
}
```

释放这个数组的顺序 先释放for循环为每一行分配的空间， 再释放行指针 （row pointer）分配的空间 

```c++
template<class T>
void delete2dArray(T**& x, int numberofRows)
{
	//删除二维数组x
	//删除行数组空间
	for (int i = 0; i < numberofRows; i++)
		

	delete[] x;
	x = NULL;

}
```

1.5 自有数据类型
enum 枚举
 如果不用枚举

```
 {
define MON  1
#define TUE  2
#define WED  3
#define THU  4
#define FRI  5
#define SAT  6
#define SUN  7
}
```

如果是枚举
enum{MON=1,TUE,WED,THU,FRI,SAT,SUN};
signType类型的定义
enum signType{plus,minus}
currency类的声明

```c++
class currency
{
public:
	//构造函数
	currency(signType theSign = plus,
		unsigned long theDollars = 0,
		unsigned int theCents = 0);
	//析构函数
	~currency(){}
	void setValue(signType, unsigned long, unsigned int);
	void setValue(double);
	signType getSign() const { return sign; }
	unsigned long getDollars() const { return cents; }
	currency add(const currency&) const;
	void output() const;
private:
	signType sign;                 //对象的符号
	unsigned long dollars;         //美元的数量
	unsigned int cents;            //美分的数量
};
currency的构造函数
currency::currency(signType thsSign, unsigned long theDollars, unsigned int theCents)
{
	//创建一个currency类对象
	setValue(theSign, theDollars, theCents);
}
给私有数据成员赋值
void currency::setValue(signType theSign, unsigned long theDollars, unsigned int theCents)
{
	//给调用对象的数据成员赋值
	if (theCents > 99)                                       //美分太多
		throw illegalParameterValue("Cents should be < 100");
	sign = theSign;
	dollars = theDollars;
	cents = theCents;
}
void currency::setValue(double theAmout)
{
	//给调用对象的数据成员赋值
	if (theAmout < 0) { sign = minus; theAmout = -theAmout; }
	else sign = plus;
	dollars = (unsigned long)theAmout;                       //提取整数部分
	cents = (unsigned int)((theAmout + 0.001 - dollars) * 100);//提取两位小数
}
把两个currency对象的值相加
currency currency::add(const currency& x) const
{
	//把x和*this相加
	long a1, a2, a3;
	currency result;
	//把调用对象转化为符号整数
	a1 = dollars * 100 + cents;
	if (x.sign == minus)a2 = -a2;
	a3 = a1 + a2;
	//转换为currency对象的表达式
	if (a3 < 0) { result.sign = minus; a3 = -a3; }
	else result.sign = plus;
	result.dollars = a3 / 100;
	result.cents = a3 - result.dollars * 100;
	return result;
}
函数increment和output
currency& currency::increment(const currency& x)
{
	//增加x
	*this = add(x);
	return *this;
}
void currency::output() const
{
	//输出调用对象的值
	if (sign == minus)cout << '-';
	cout << '$' << dollars << '.';
	if (cents < 10)cout << '0';
	cout << cents;
}
```

可以做一些改动使得 currency类中使用最多的两个函数 add 和 increment 运行得更快 
 对于 用户隐藏 类的实现细节 的一个重大益处 是，用新的，更高效的类对象 描述 取代 以前的描述 之后 ，应用代码不需要  任何改动
 类currency的新声明

```c++
 class currency
{
public:
	//构造函数
	currency(signType theSign = plus,
		unsigned long theDollars = 0,
		unsigned int theCents = 0);
	//析构函数
	~currency(){}
	void setValue(signType, unsigned long, unsigned int);
	void setValue(double);
	signType getSign() const
	{
		if (amout < 0)return minus;
		else return plus;
	}
	unsigned long getDollars() const
	{
		if (amout < 0) return (-amout) / 100;
		else return amout / 100;
	}
	unsigned int getCents() const
	{
		if (amout < 0) return -amout - getDollars() * 100;
		else return amout - getDollars() * 100;
	}
	currency add(const currency&)const;
	currency& increment(const currency& x)
	{
		amout += x.amout;
		reuturn* this;
	}
	void output() const;
private:
	long amout;
 };
 构造函数和成员函数setValue的新代码
currency::currency(signType theSign, unsigned long theDollars, unsigned int theCents)
{
	//创建一个currency类对象
	setValue(theSign, theDollars, theCents);
}
void currency::setValue(signType theSign, unsigned long theDollars, unsigned int theCents)
{
	//给调用对象赋值
	if (theCents > 99)
		//美分值太大
		throw illegalParameterValue("Cents shoule be <100");
	amout = theDollars * 100 + theCents;
	if (theSign == minus)amout = -amout;
}
void currency::setValue(double theAmout)
{
	//给调用对象赋值
	if (theAmout < 0)
		amout = (long)((theAmout - 0.001) * 100);
	else
		amout = (long)((theAmout + 0.001) * 100);
}
 成员函数add和output的新代码
	 currency currency::add(const currency& x)const
 {
	 //把x和*this相加
	 currency y;
	 y.amout = amout + x.amout;
	 return y;
 }
 void currency::output() const
 {
	 //输出调用对象的值 
	 long theAmout = amout;
	 if (theAmout < 0)
	 {
		 coaut << '-';
		 theAmout = -theAmout;
	 }
	 long dollars = theAmout / 100;                         //美元
	 cout << '$' << dollars << '.';
	 int cents = theAmout - dollars * 100;                  //美分
	 if (cents < 10)cout << '0';
	 cout << cents;
 }
 包含操作符重载的类声明
 class currency
{
public:
	//构造函数
	currency(signType theSign = plus,
		unsigned long theDollars = 0,
		unsigned int theCents = 0);
	//析构函数
	~currency() {}
	void setValue(signType, unsigned long, unsigned int);
	void setValue(double);
	signType getSign() const
	{
		if (amout < 0)return minus;
		else return plus;
	}
	unsigned long getDollars() const
	{
		if (amout < 0) return (-amout) / 100;
		else return amout / 100;
	}
	unsigned int getCents() const
	{
		if (amout < 0) return -amout - getDollars() * 100;
		else return amout - getDollars() * 100;
	}
	currency operator+(const currency&) const;
	currency operator+=(const currency& x)
	{
		amout += x.amout;
		return *this;
	}
	void output(ostream&) const;
private:
	long amout;
};
+,output和<<的代码
currency currency::operator+(const currency&) const
{
	//把参数对象x和调用对象*this相加
	currency result;
	result.amout = amout + x.amout;
	return result;
}

void currency::output(ostream& out) const
{
	//把货币值插入流out
	long theAmout = amout;
	if (theAmout < 0)
	{
		out << '-';
		theAmout = -theAmout;
	}
	long dollars = theAmout / 100;                       //美元
	out << '$' << dollars << '.';
	int centsa = theAmout - dollars * 100;               //美分
	if (cents < 10)out << '0';
	out << cents;
}

//重载<<
ostream& operator<<(ostream& out, const currency& x)
{
	x.output(out);
	return out;
}
使用友元函数重载友元操作符<<
//重载
ostream operator<<(ostream out, const currency & x)
{
	//把货币值插入流out
	long theAmout = x.amout;
	if (theAmout < 0)
	{
		out << '-';
		theAmout = -theAmout;
	}
	long dollars = theAmout / 100;                 //美元
	out << '$' << dollars << '.';
	int cents = theAmout - dollars * 100;          //美分
	if (cents < 10) out << '0';
	out << cents;
	return out;
}
增加#ifndef   #define 和 #endif 语句
在一个类的 声明和实现细节文件  也就是 类似于 currency.h 这样的文件  
在文件头加上 
#ifndef Currency_
#define Currency_
在文件尾加上
#endif
```

包含在 这组语句之内的代码只编译一次

1.6异常类illegalParameterValue
当一个函数的实参值无意义时   要抛出的异常就是这个类型
定义一个异常类

```c++
class illegalParameterValue
{
public:
		illegalParameterValue():
		message("Illegal parameter value"){}
		illegalParameterValue(char* theMessage)
		{message=theMessage;}
		void outputMessage(){cout<<message<<endl;}
		private:
		string message;
};
```

  捕捉illegalParameterValue类型的异常

```c++
  int main()
  {
  try{cout<<abc(2,0,4)<<endl;}
  catch(illegalParameterValue e)
  {
  cout<<"The parameters to abc were 2,0 and 4"<<endl;
  cout<<"illegalParameterValue exception thrown"<<endl;
  e.outputMessage();
  return 1;
  }
  return 0;
  }
```

  1.7递归函数

计算n!的递归函数

```c++
int factorial(int n)
{
	//计算n!
	if(n<=1)return 1;
	else return n*factorial(n-1);
}
```

累加数组元素a[0:n-1]

```c++
template<class T>
T sum(T a[],int n)
{
	//返回数值数组元素a[0:n-1]的和
	T theSum=0;
	for(int i=0;i<n;i++)
		theSum+=a[i];
		return theSum;
}
```

累加数组元素a[0:n-1]的递归代码

```
template<class T>
T rSum(T a[],int n)
{
	//返回数组元素a[0:n-1]的和
	if(n>2)
	return rSum(a,n-1)+a[n-1];
	return 0;
}
```


n个元素的排列个数就是n！

使用递归函数生成排列

```c++
template<class T>
void permutations(T list[],int k,int m)
{
	//生成list[k:m]的所有排列
	if(k==m){//list[k:m]  仅有一个排列，输出它
	copy(list,list+m+1,
	ostream_iterator<T>(cout,""));
	cout<<endl;
	}
	else    //list[k:m]有多于一个的排列，递归地生成这些排列
	for(int i=k;i<=m;i++)
	{
	swap(list[k],list[i]);
	permutations(list,k+,m);
	swap(list[k],list[i]);
	}
}
```


1.8 标准模板库
  STL是一个容器，适配器，迭代器，函数对象（仿函数）  算法  的集合
STL 的一个算法accumulate 对顺序表 元素 顺序累计求和  
accumulate(start,end,initialValue)

start指向的是首元素	end指向尾元素的下一个位置 
因此要累计求和的元素范围是[start，end)
调用的语句为
accumulate(a,a+n,initialValue)

利用STL的算法accumulate对a[0:n-1]求和

```c++
template<class T>
T sum(T a[],int n)
{
	//返回数组a[0:n-1]的累计和
	T theSum=0;
	return accumulate(a,a+n,theSum);
}
```

accumulate更通用的形式如下
accumulate(start,end,initialValue,operator)

operator是一个函数吗，规定了在累计过程中的操作

利用STL的函数对象multiplies能够计算数组元素的乘积

```c++
template<class T>
T operator(T a[],int n)
{
	//返回数组a[0:n-1]的累计和
	T theProduct=1;
	return accumulate(a,a+n,theProduct,multiplies<T>());
}
```

算法copy把一个顺序表的元素从一个位置复制到另一个位置

copy(start,end,to)

其中to给出第一个元素要复制到的位置
元素从位置start,start+1,***,end-1 依次复制到位置 to,to+1,***,to+end-start

算法next_permutation的语法
next_permutation(start,end)

使用STL算法next_permutation求排列

```c++
template<class T>
void permutations(T list[],int k,int m)
{
	//生成list[k:m]的所有排列
	//假设k<=m
		//将排列逐个列出
		do
		{
		copy(list,list+m+1,
		ostream_iterator<T>(cout," "));
		cout<<endl;
		}while(next_permutation(list,list+m+1));
}
```

next_permutation更一般的形式
next_permutation(start,end,compare)
函数compare用来判定一个排列是否比另一个排列小 
且在仅有两个参数的版本中，比较操作符是由<来执行的

1.9测试与调试

计算并输出一个二次方程ax^2+bx+c的根

```c++
void outputRoots(const double& a,const double& b,const double& c)
{
	//计算和输出二次方程的根
	
	double d=b*b-4*a*c;
	if(d>0)
	{
	//两个实数根
	double sqrtd=sqrt(d);
	cout<<"There are two real roots"
		<<(-b+sqrtd)/(2*a)<<" and "
		<<(-b-sqrtd)/(2*a)
		<<endl;
	}
	else if(d==0)
	//两个根相同
	cout<<"There is only one distinct root"
		<<-b/(2*a)
		<<endl;
        else //复数共轭根
            cout<<"The roots are complex"
            	<<endl
            	<<"The real part is"
            	<<-b/(2*a)<<endl
            	<<"The imaginary part is"
            	<<sqrt(-d)/(2*a)<<endl;
}
```





​																              ***第二章 :程序性能分析***

2.2 空间复杂度

程序所需要的空间主要由以下的部分构成

(1)指令空间：指令空间是指编译之后的程序指令所需要的存储空间

（2）数据空间：常量和变量值所需要的存储空间  由两个部分构成

- 常量和 简单变量 

- 动态数组和动态类实例等动态对象

（3）环境栈空间

环境栈用来保存暂停的函数和方法在恢复运行时所需要的信息

例如说：如果函数foo调用了函数goo，那么我们至少要保存在函数goo结束时函数foo继续执行的指令地址

1.指令空间

- 指令空间的数量取决于以下因素
- 把程序转化成机器代码的编译器
- 在编译时的编译器选项
- 目标计算机

3.环境栈空间

每当一个函数被调用时，下面的数据将呗保存在环境栈中：

- 返回地址
- 正在调用的函数的所有局部变量的值以及形式参数的值（仅对递归函数而言）

递归函数所需要的栈空间通常被称为递归栈空间

它的大小依赖于局部变量和形式参数所需要的空间，依赖于递归的最大深度（即嵌套递归调用的最大层次）和编译器。

可以把一个程序所需要的空间分成两部分：

- 固定部分：它独立于实例特征，这一部分通常包括指令空间（即代码空间），简单变量空间和常量空间等。
- 可变部分：它由动态分配空间构成和递归栈空间构成，前者在某种程度上以来实例特征，而后者主要依赖实例特征。

2.3时间复杂度

时间复杂度的组成：影响空间复杂度的因素也影响时间复杂度

计算机的性能也是其中一个因素

编译器生成代码的速度

运行时间通常用*t*p（实例特征）来表示

用分析方法确定一个程序的运行时间是很复杂的，因此只能估算运行时间。

有两个比较容易控制的方法

1）.找出一个或多个关键操作，确定它们的执行时间

2）.确定程序总的步数

多项式计算

```c++
template<class T>
T polyEval(T coeff[],int n,const T& x)
{
//计算n阶多项式在点x处的值，系数为coeff[0:n]
T y=1,value=coeff[0];
for(int i=1;i<=n;i++)
{
//加上下一项
y *=x;
value+=y*coeff[i];
}
return value;
}
```

利用Horner法则的多项式计算

```c++
template<class T>
T horner(T coeff[],int n,const T& x)
{
	//计算n阶多项式在点x处的值，系数为coeff[0:n]
	T value=coeff[n];
	for (int i=1;i<=n;i++)
	{
		value=value*x+coeff[n-i];
		return value;
	}
}
```

名次计算ranking 一个元素在一个序列中的名次（rank）是所有比他小的元素个数加上在他左边出现的与它相同的元素个数

```c++
teamplate<class T>
void rank(T a[],int n,int r[])
{
    //给数组a[0:n-1]的n个元素排名次
    //结果在r[0:n-1]中返回
    for(int i=1;i<n;i++)
    {
        for(int j=0;j<i;j++)
            if(a[i]<=a[i])r[i]++;
        else r[j]++;
    }
}
```

上边这个程序可以计算出数组a 的各元素的名次，就可以移到与其名次对应的位置，按递增顺序重新排列，即a[0]<=a[1]<=***a[n-1]

下边程序中的rearrange函数使用一个附加数组u实现了这个算法

按名次排序(rank sort)

```c++
template<class T>
void rearrange(T a[],int n,int r[])
{
	//使用一个附加数组u，将元素排序、
	T *u=new T [n];                    //创建附加数组
	
	//把a中元素移到u中正确位置
	for(int i=0;i<n;i++)
		u[r[i]]=a[i];
		
		//把u中元素移动到a
		for(i=0;i<n;i++)
		a[i]=u[i];
		
		delete [] u;
}
```

选择排序(selection sort)   是另一种给数组元素排序的方式

首先找到最大的元素，把他移动到a[n-1]  然后在剩下的n-1个元素中找出最大的元素，把它移动到a[n-2]

如此进行下去

这种方式不需要借助附加数组u

```c++
template<class T>
void selectionSort(T a[],int n)
{
	//给数组a[0:n-1]的n个元素排序
	for(int size=n;size>1;size--)
	{
		int j=indexOfMax(a,size);
		swap(a[j],a[size-1]);
	}
}
```

冒泡排序：把最大元素移动到右边，在一次冒泡过程中，相邻的元素比较，如果左边的元素大于右边的元素，就交换他们。

一次冒泡排序

```c++
template<class T>
void bubble(T a[],int n)
{
	//把a[0:n-1]中最大元素移到右边
	for(int i=0;i<n-1;i++)
	if(a[i]>a[i+1])swap(a[i],a[i+1]);
}
```

函数bubble的功能在于把最大元素移到序列最右端，因此可以用来替代选择排序中的indexOfMax从而得到一个新的排序函数

```c++
template<class T>
vodi bubbleSort(T a[],int n)
{
	//对数组元素a[0:n-1] 使用冒泡排序
	for(int i=n;i>1;i--)
	bubble(a,i);
}
```

2.3.3最好，最坏和平均操作计数

在有序数组中插入元素

在有序数组中插入一个新元素，插入之后数组依然有序

在有序数组中插入一个元素

```c++
template<class T>
void insert(T a[],int& n,const T& x)
{
	//把x插入有序数组a[0:n-1]
	//假设数组a 的容量大于n
	int i;
	for(i=n-1;i>=0 && x<a[i];i--)
	a[i=1]=a[i];
	a[i+1]=x;
	n++;//数组a多了一个元素
}
```

原地重排数组元素

```c++
template<class T>
void rearrange(T a[],int n,int r[])
{
	//原地重排数组元素使之有序
	for(int i=0;i<n;i++)
	//把正确的元素移到a[i]
	while(r[i]!=i)
	{
		int t=r[i];
		swap(a[i],a[t]);
		swap(r[i],r[t]);
	}
}
```

及时终止的选择排序

```c++
template<class T>
void selectionSort(T a[],int n)
{
	//及时终止的选择排序
	bool sorted=false;
	for(int size=n;!sorted && (size>1);size--)
	{
		int indexOfMax=0;
		sorted=true;
		//查找最大元素
		for(int i=1;i<size;i++)
		if(a[indexOfMax]<=a[i])indexOfMax=i;
		else sorted=false;//无序
		swap(a[indexOfMax],a[size-1]);
	}
}
```

及时终止的冒泡排序

```c++
template<class T>
bool bubble(T a[],int n)
{
	//把数组a[0:n-1]中的最大元素移到最右端
	bool swapped=false;//目前为止未交换
	for(int i=0;i<n-1;i++)
	{
	swap(a[i],a[i+1]);
	swapped=true;//交换
	}
	return swapped;
}
template<class T>
void bubbleSort(T a[],int n)
{
	//及时终止冒泡排序
	for(int i=n;i>1 && bubble(a,i);i-1);
}
```

插入排序

```c++
template<class T>
void insert(T a[],int n,const T& x)
{
	//把x插入有序数组a[o:n-1]
	int i;
	for(i=n-1;i>=0 && x<a[i];i--)
	a[i+1]=a[i];
	a[i+1]=x;
}
template<class T>
void insertionSort(T a[],int n)
{
	//对数组a[0:n-1]实施插入排序
	for(int i=1;i<n;i++)
	{
		T t=a[i];
		insert(a,i,t);
	}
}
```

另一种插入排序

```c++
template<class T>
void insertionSort(T a[],int n)
{
	//对数组a[0:n-1]实施插入排序
	for(int i=1;i<n;i++)
	{
		//把a[i]插入a[0:i-1]
		T t=a[i];
		int j;
		for(j=i-1;i>=0 && t<a[j];j--)
		a[j+1]=a[j];
		a[j+1]=t;
	}
}
```

上边这两种插入排序的比较次数相同，最好的比较次数是n-1，最坏的比较次数是（n-1）n/2

程序步：可以大概地定义为一个语法或语义上的程序片段，该片段的执行时间独立于实例特征

计算程序步的一个例子

```c++
template<class T>
T sum(T a[],int n)
{
	//返回数值数组元素a[0:n-1]的和
	T theSum=0;
	stepCount++;                      //theSum=0是一个程序步
	for(int i=0;i<n;i++)
	{
		stepCount++;					//for循环的每一次条件的判断是一个程序步
		theSum+=a[i];
		stepCount++;					//theSum+=a[i]是一个程序步
	}
	stepCount++;						//for循坏的最后一次判断是一个程序步
	stepCount++;						//return theSum是一个程序步
	return theSum;
}
```

计算rSum函数的步数

```c++
template<class T>
T rSum(T a[],int n)
{
	//返回数组元素a[0:n-1]的和
	stepCount++;						//if语句是一个程序步
	if(n>0){
		stepCount++;					//return语句和调用语句是一个程序步
		return rSum(a,n-1)+a[n-1];
	}
	stepCount++;						//return 语句是一个程序步
	return 0;
}
```

rowsXrows矩阵a[0:rows-1] [0:rows-1] 的转置 

矩阵转置

```c++
template<class T>
void transpose(T **a,int rows)
{
	//原地完成矩阵a[0:rows-1][0:rows-1]的转置
	for(int i=0;i<rows;i++)
		for(int j=i+1;j<rows;j++)
		swap(a[i][j],a[j][i]);
}
```

​                                                                             **第三章 渐进记法**

令p(n)和q(n)是两个非负函数。

称p(n)渐近地大于q(n)  (p(n)渐近地优于q(n)),当且仅当   q(n)/p(n)的极限等于0时



称q(n)渐近地小于p(n),当且仅当p(n)渐近地大于q(n).

称p(n)渐近地等于q(n),当且仅当任何一个都不是渐近地大于另一个



渐近记法（asymptotic notation）描述的是大实例特征的时间或空间复杂度。

我们将用它来 分析步数（其实还可与i用它来分析空间复杂度和操作步数）

时间复杂度和步数是同义词

如果实例特征只含有一个变量，例如n，渐近记法就用步数中渐近最大的一项来描述复杂度

3.4复杂度分析举例

 



折半查找

```c++
template<class T>
int binarySearch(T a[],int n,const T& x)
{
	//在有序数组a中查找元素x
	//如果存在，就返回元素x的位置，否则返回-1
	int left=0;                          //数据段的左端
	int right=n-1;                       //数据段的右端
	while(left<=right)
	{
		int middle=(left+right)/2;      //数据段的中间
		if(x==a[middle])return middle;
		if(x>a[middle])left=middle+1;
		else right=middle-1;
	}
	return -1;                          //没有找到x
}
```

​                                                                                          ***性能测量***

​		性能测量，关注的是一个程序实际需要的空间和时间

时间优化代码由下面的语句产生

#program optimize(“t”,on)

对于运行空间无法明确考量是因为如下两个因素

- 指令空间和静态分配的数据空间是编译器在编译时确定的，它们的大小可以用操作系统指令来得到。
- 递归栈空间和动态分配的变量空间可以用前面的分析方法明确地估算。

定时机制，c++函数clock()

time.h头文件中定义了常数CLOCK_PER_SEC,它记录每秒流逝的 滴答  数，并转换成秒。 CLOCK_PER_SEC=1000，滴答一次等于一毫秒

导致插入排序出现最坏复杂度的程序

```c++
int main()
{
	int a[1000],step=10;
    double clocksPerMills =double(CLOCK_PER_SEC)/1000;
    			//每毫秒滴答一次
    cout<<"The worst-case time,in millseconds,are"<<endl;
    cout<<"n \t Time"<<endl;
    
    //次数n=0,10,20,***,100,200,300,***,1000
    for(int n=0;n<=1000;n+=step)
    {
        //用最坏测试数据初始化
        for(int i=0;i<n;i++)
            a[i]=n-i;
        clock_t startTime=clock();
        insertionSort(a,n);
        double elapsedMills=(clock()-startTime)/clocksPerMills;
        cout<<n<<'\t'<<elapsedMills<<endl;
        if(n==100)step=100;
    }
    return 0;
}
```

误差在10%以内的测量程序

```c++
int main()
{
	int a[1000],step10;
    double clocksPerMills=double(CLOCKS_PER_SEC)/1000;
    			//每毫秒滴答一次
    cout<<"The worst-case time ,in milliseconds,are"<<endl;
    cout<<"n \tRepetitons \t Total Ticks \tTime per Sort"<<endl;
    //次数n=0,10,20,***,100,200,300,***,1000
    for(int n=0;n<=1000;n+=step)
    {
        //为实例特征n测量运行时间
        long numberOfRepetitons=0;
        clock_t startTime=clock();
        do
        {
            numberOfRepetitions++;
            //用最坏测试数据初始化
            for(int i=0;i<n;i++)
                a[i]=n-i;
            inserttionSort(a,n);
        }while(clock()-startTime<1000);
        //重复运行，只到有足够的时间流逝
        double elapsedMills=(clock()-startTime)/clocksPerMills;
        cout<<n<<'\t'<<numberOfRepetitons<<'\t'<<elapsedMills
            <<'\t'<<elapsedMills/numberOfRepetitions
            <<endl;
        if(n==100)step=100;
    }
    return 0;
}
```

​                                                            						***线性表***---***数组描述***

C++常用的数据描述方法是 ：数组描述和链式描述

STL容器（vector和list）大致相当于线性表的数组描述方法和链式描述方法

数组描述方法将元素存储在一个数组中，用一个数学公式来确定每个元素存储的位置，即在数组中的索引。

这是最简单的一种存储方式，所有元素依次存储在一片连续的存储空间中，这就是顺序表



数据对象（data object）是一组实例或值，例如

boolean={false,true}

digit={0,1,2,3,4,5,6,7,8,9}

letter={A,B,C,---,Z,a,b,---,z}

naturalNumber={0,1,2,---}

integer={0,+-1,+-2,+-3,---}

string={a,b,---,aa,ab,ac,--}

 

数据结构（data structure）是一个数据对象，同时这个对象的实例以及构成实例的元素都存在着联系，而且这些联系都由相关的函数来规定

5.2线性表数据结构

线性表（linear list）也称有序表（ordered list），它的每一个实例都是元素的一个有序集合。

抽象数据类型 LinearList

一个线性表可以用一个抽象数据类型(abstract data type,ADT)来说明，既说明它的实例，也说明对它的操作。

```c++
抽象数据类型 LinearList
{
	实例
	有限个元素的有序集合
	操作
		empty();若表空，则返回true，否则返回false
		size();返回线性表的大小（表的元素个数）
		get(index);返回线性表中索引为index的元素
		indexOf(x);返回线性表中第一次出现的x的索引。若x不存在，则返回-1
		erase(index);删除索引为index的元素，索引大于index的元素其索引递减1
		insert(index，x)；把x插入线性表中索引为index的位置上，索引大于等于index的元素其索引加1
		output();从左到右输出表元素
}
```

抽象类LinearList

C++支持两种类，---具体类  抽象类。只有具体类才可以实例化，也就是我们只能对具体类建立实例或对象。

不过，我们可以创建抽象类的对象指针

一个线性表的抽象类

```c++
template<class T>
class linearList
{
	public:
    	virtual ~linearList() {};
    	virtual bool empty() const=0;
    //返回true，当且仅当线性表为空
    	virtual int size() const=0;
    //返回线性表的元素个数
    	virtual T& get(int theIndex)const =0;
    //返回索引为theIndex的元素
    	virtual int indexOf(const T& theElement) const=0;
    //返回元素theElment第一次出现时的索引
    	virtual void erase(int theIndex)=0;
    //删除索引为theIndex的元素
    	virtual void insert(int theIndex ,const T& theElement)=0;
    //把theElement插入线性表中索引为theIndex的位置上
    	virtual void output(ostream & out) const=0;
    //把线性表插入输出流out
    
};
```

在数组描述(array repersentation)中，用数组来存储线性表的元素。

当数组满而需要加大数组长度时，数组长度常常是要加倍的。这个过程称为数组倍增（array doubling）。

改变一个一维数组长度

```c++
template<class T>
void changeLength1D(T *a,int oldLength,int newLength)
{
	if(newLength<0)
	throw illegalParameterValue("new length must be >=0");
	
	T* temp=new T[newLength];					//新数组
	int number =min(oldLength,newLength); 		//需要复制的元素个数
	copy(a,a+number,temp);
	delete [] a;								//释放老数组的内存空间
	a=temp;
}
```

类arrayList

定义一个	C++抽象类linearList的派生类arrayList

```c++
template<class T>
class arrayList:public linearList<T>
{
    public:
    //构造函数，复制构造函数和析构函数
    arrayList(int initialCapacity=10);
    arrayList(const arrayList<T>&);
    ~arrayList(){delete [] element;}
    
    //ADT方法
    bool empty()const {return listSize==0;}
    int size()const {return listSize;}
    T& get(int theIndex)const;
    int indexOf(const T& theElement)const;
    void erase(int theIndex);
    void insert(int theIndex,const T& theElement);
    void output(ostream& out)const;
    //其他方法
    int capacity()const {return arrayLength;}
    
    protected:
    void checkIndex(int theIndex)const;
    	//若索引theIndex无效，则抛出异常
    T* element;					//存储线性表元素的一维数组
    int arrayLength;			//一维数组的容量
    int listSize;				//线性表的元素个数
};
```

方法capacity给出的是数组element当前的长度，方法checkIndex要确定一个元素在范围0~listSize-1内的索引

类arrayList的构造函数

```c++
template<class T>
arrayList<T>::arrayList(int initialCapacity)
{
	//构造函数
	if(initialCapacity<1)
	{
		ostringstream s;
		s<<"Initial capacity ="<<initialCapacity<<"Must be > 0";
		throw illegalParameterValue(s.str());
	}
    arrayLength=initialCapacity;
    element=new T[arrayLength];
    listSize=0;
}

template<class T>
   arrayList::arrayList(const arrayList<T>& theList)
{
    //复制构造函数
    arrayLength=theList.arrayLength;
    listSize=theList.listSize;
    element=new T[arrayLength];
    copy(theList.element,theList.element+listSize,element);
}
```

copy函数中的theList.element是开始,theList.element+listSize是结束  element是被复制的第一个元素

arrayList实例化
用数组描述的线性表需要使用下面的语句来创建/实例化

```c++
//创建两个容量为100的线性表
linearList *x=(linearList)new arrayList<int>(100);
arrayList<double> y(100);

//利用容量的缺省值创建一个线性表
arrayList<char> z;

//用线性表y复制创建一个和线性表
arrayList<double> w(y);
```



arrayList的基本方法

checkIndex get indexOf

```c++
template<class T>
void arrayList<T>::checkIndex(int theIndex) const
{
    //确定索引theIndex在0和listSize-1之间
    if(theIndex<0 || theIndex>=listSize-1)
    {
        ostringstream s;
        s<<"index ="<<theIndex<<"size="<<listsize;
        throw illegalIndex(s.str());
    }
}

template<class T>
    T& arrayList<T>::get(int theIndex) const
    {
        //返回索引为theIndex的元素
        //若此元素不存在，则抛出异常
        checkIndex(theIndex);
        return element[theIndex];
    }

template<class T>
int arrayList<T>::indexOf(const T& theElement) const
{
    //返回元素theElement第一次出现时的索引
    //若该元素不存在，则返回-1
    //查找元素theElement
    int theIndex=(int)(find(element,element+listSize,theElement)-element);
    
    //确定元素theElement是否找到
    if(theIndex==listsize)
        //没有找到
        return -1;
    else return theIndex;
}
```

删除一个元素

为了从线性表中删除索引为theIndex的元素，首先要确定线性表包含这个元素，然后删除这个元素。

如果没有这个元素，就抛出类型 illegalIndex的异常

当要删除索引为theIndex的元素时，利用copy算法把索引从theIndex+1，theIndex+2，***，listSize-1的元素向左移动一个位置，然后把listSize的值-1，

删除索引为theIndex的元素

```c++
template<class T>
    void arrayList<T>::erase(int theIndex)
    {
        //删除索引为theIndex的元素
        //如果该元素不存在，则抛出illegalIndex的异常
        checkIndex(theIndex);
        
        //有效索引，移动其索引大于theIndex的元素
        copy(element+theIndex+1,element+listSize,element+theIndex);
        element[--listSize].~T();   //调用析构函数
    }
```

插入一个元素

要在线性表中索引为theIndex的位置插入一个新元素，首先把索引从theIndex到listSize-1的元素向右移动一个位置，然后将新元素插入索引为theIndex的位置，，最终listSize的值+1

向右移动元素的操作时利用STL函数copy_backward来完成的.

该函数是从最右端的元素开始移动的。

如果数组空间已满，那么数组长度会加倍

在索引theIndex的位置上插入一个元素

```c++
template<class T>
void arrayList<T>::insert(int theIndex,const T& theElement)
{
    //在索引theIndex处插入元素theElement
    if(theIndex<0 || theIndex>listSize)
    {
        //无效索引
        ostringstream s;
        s<<"index = "<<theIndex<<"size="<<listSize;
        throw illegalIndex(s.str());
        
    }
    //有效索引
    if(listSize==arrayLength)
    {
        //数组空间已满，数组长度倍增
        changeLength1D(element,arrayLength,2*arrayLength);
        arrayLength*=2;
    }
    //把元素向右移动一个位置
    copy_backward(element+theIndex,element+listSize,element+listSize+1);
    element[theIndex]=theElement;
    listSize++;
}
```

输出函数output和重载<<

把一个线性表插入输出流

```c++
template<class T>
void arrayList<T>::output(cout->out) const
{
    //把线性表插入输出流
    copy(element,element+listSize,ostream_iterator<T>(cout," "));
}

//重载<<
template<class T>
    ostream& operator<(ostream& out,const arrayList<T>& x)
{
    x.output(out);
    return out;
}
```

减少数组长度

为了能够在数组元素减少时释放一些数组空间，我们可以修改erase方法，当listSize<arrayLength/4时数组长度减到max{initialCapacity,arrayLength/2}

C++迭代器

一个 迭代器(iterator)是一个指针，指向对象的一个元素。

使用数组迭代器

```c++
int main()
{
	int x[3]={0,1,2};
    //用指针y遍历数组x
    for(int *y=x;y!=x+3;y++)
        cout<<*y<<" ";
        cout<<endl;
    return 0;
}
```



```c++
for(iterator i=start;i!=end;i++)
cut<<*i<<" ";
```



对STL的copy函数的一种可行的代码

```c++
template<class T>
void copy(iterator start,iterator end,iterator to)
{
    //从[start,end)复制到[to,to+end-start)
    while(start!=end)
    {*to=*start;
    start++;
    to++}
}
```

 arrayList的一个迭代器

定义一个类iterator，它是类arrayList的双向迭代器

这个迭代器是类arrayList的公有成员。

为类arrayList增加两个共有方法begin()    end()

他们的指向分别是首元素element[0]和尾元素的下一个位置element[listSize]

这两个方法的代码是

```c++
class iterator;
iterator begin()
{
    return iterator(element);
}
iterator end()
{
    return iterator(element+listSize);
}
```

下边的语句创建了一个迭代器实例并初始化

```c++
arrayList<T>:.iterator x=y.begin();
```

STL算法reverse可以把线性表y的元素逆置

STL的算法accumulate可以对线性表y 的元素求和

```c++
reverse(y.begin(),y.end());
int sum=accumulate(y.begin(),y.end(),0);
```

但是我们不能使用ST L的算法sort去实现基于随机访问迭代器的算法

类arrayList的一个迭代器

```c++
class iterator
{
	public:
    //用C++的typedef语句实现双向迭代器
    typedef bidirectional_iterator_tag iterator_capacity;
    typedef T value_type;
    typedef ptrdiff_t difference_type;
    typedef T* pointer;
    typedef T& reference;
    
    //构造函数
    iterator(T* thePosition=0)
    {
        position=thePosition;
    }
    //解引用操作符
    T& operator*()const {return *position;}
    T* operator->()const {return &*position;}
    
    //迭代器的值增加
    iterator& operator++()			//前加
    {++position;return *this;}
    iterator operator++(int)		//后加
    {
        iterator old=*this;
        ++position;
        return old;
    }
    
    //迭代器的值减少
    iterator& operator--()			//前减
    {--position;return *this;}
    iterator& operator--(int)		//后减
    {
        iterator old =*this;
        --position;
        return old;
    }
    
    //测试是否相等
    bool operator!=(const iterator right)const
    {return position !=right.position;}
    bool operator==(const iterator right)const
    {return position==right.position;}
    protected:
    T* position;					//指向表元素的指针
}
```

vector的描述

STL提供了一个基于数组的类vector。这个类不仅具有arrayList类的所有功能，还增加了很多方法。

数组长度是按需要动态增加的，如果实施插入操作时vector已满，那么vector的容量将按原容量的50%~100%来增加。

类vector没有一个构造函数与arrayList的构造函数等价，也没有名为get，indexOf和output的方法。

但vector和arrayList都有方法empty和size，而且等价。

虽然vector具有方法erase和insert，分别表示删除和插入。但他们需要内存地址，而不是索引。vector和arrayList还有一点不同，它们抛出的异常是不同类型。

利用vector实现的基于数组的线性表

```c++
template<class T>
class vectorList:public linearList<T>
{
    public:
    	//构造函数，复制构造函数和析构函数
    vectorList(int initialCapacity=10);
    vectorList(const vectorList<T>&);
    ~vectorList(){delete element;}
    
    //ADT方法
     bool empty()const {return element->empty();}
    int size ()const {return (int) element->size();}
    T& get(int theIndex)const;
    int indexOf(const T& theElement)const;
    void erase(int theIndex);
    void insert(int theIndex,const T& theElement);
    void output(ostream& out)const;
    
    //增加的方法
    int capacity()const {return (int)element->capacity();}
    
    //线性表的起始和结束位置的迭代器
    typedef typename vector<T>::iterator iterator;
    iterator begin(){return element->begin();}
    iterator end(){return element->end();}
    
    protected:				//增加的成员
    void checkIndex(int theIndex)const;
    vector<T>* element;			//存储线性表元素的向量
}
```

vectorList 的构造函数

```c++
template<class T>
vectorList<T>::vectorList(int initialCapacity)
{
	//构造函数
    if(initialCapacity<1)
    {
        ostringstream s;
        s<<"Initial capacity="<<initialCapacity<<"Must be > 0";
        throw illegalParameterValue(s.str());
    }
    element=new vector<T>;
    //创建容量为0的空间量\
    element->reserve(initialCapacity);
    //vector容量从0增加到initialCapacity
    
}

template<class T>
    vectorList<T>::vectorList(const vectorList<T>& theList)
    {
        //复制构造函数
        element=new vector<T>(*theList.element);
    }
```

vectorList的删除和插入

```c++
template<class T>
void vectorList<T>::erase(int theIndex)
{
    //删除索引为theIndex的元素
    //如果没有这个元素，则抛出异常
    checkIndex(theIndex);
    element->erase(begin()+theIndex);
}

template<class T>
    void vectorList<T>::insert(int theIndex,const T& theElement)
    {
        //在索引为theIndex处插入元素theElement
        if(theIndex<0 || theIndex>size())
        {
            //无效索引
            ostringstream s;
            s<<"index= "<<theIndex<<"size="<<size();
            throw illegalIndex(s.str());
        }
        element-insert(element->begin()+theIndex,theElement);
        //如果在重定向量长度时空间不足，那么可以抛出没有捕获的异常
    }
```

在一个数组中实现的多重表

数组描述的缺点是空间利用率低。

​											第六章 线性表--链式描述

在链式描述中，线性表的元素在内存中的存储位置是随机的。

每个元素都有一个明确的指针或链（指针和链是一个意思）指向线性表的下一个元素的位置（即地址）

​                                                                                                                                  

在基于数组的描述中，元素的地址是由数学公式决定的。

在链式描述中，元素的地址是随机分布的

本章节引入的数据结构概念

- 链式描述
- 链表，循环表，双向链表
- 头节点

STL的容器类list使用带有头节点的双向循环链表来描述实例。

它的方法与vector的方法具有相同的签名和操作。

因此，它的erase和insert的签名和抽象数据类型linearList的要求不同，然而和vector一样，它可以用来设计从抽象类linearList派生的具体类。

本章包括了

箱子排序（bin sort）也叫做桶排序（bucket sort）

基数排序（radix sort）

凸包（convex hull）

并查集（union-find）

其中凸包使用了双向链表

并查集问题说明了将证书作为指针来建立链表的方法

6.1单向链表

在链式描述中，数据对象实例的每一个元素都用一个单元或节点来描述。

每一个节点都明确包含另一个相关节点的位置信息，这个信息成为链（link）或指针（pointer）

结构chainNode

为了用链表描述线性表，我们要定义一个结构chainNode和一个类chain

数据成员element是节点的数据域，存储表元素

数据成员next是节点的链域，存储下一个节点的指针

链表节点的结构定义

```c++
template<class T>
struct chainNode
{
    //数据成员
    T element;
    chainNode<T> *next;
    
    //方法
    chainNode();
    chainNode(const T& element)
    {this->element=element;}
    chainNode(const T& element,chainNode<T>* next)
    {this->element=element;
    this->next=next;}
};
```

类chain

链表chain的方法header empty size

类chain用单向链表实现了线性表，其中最后一个节点的指针域为NULL，即它用单向链接的一组节点实现线性表。

链表节点的结构定义

```c++
template<class T>
class chain:public linearList<T>
{
	public:
    //构造函数，复制构造函数和析构函数
    chain(int initialCapacity=10);
    chain(const chain<T>&);
    ~chain();
    
    //抽象数据类型ADT方法
    bool empty()const {return liseSize==0;}
    int size()const {return liseSize;}
    T& get(int theIndex)const;
    int indexOf(const T& theElement)const;
    void erase(int theIndex);
    void insert(int theIndex,const T& theElement);
    void output(ostream& out)const;
    
    protected:
    void checkIndex(int theIndex)const;
    //如果索引无效，抛出异常
    chainNode<T>* firstNode;	//指向链表第一个节点的指针
    int listSize;					//线性表的元素个数
    
}；
```

构造函数和复制构造函数

为了创建一个空链表，只需要令第一个节点指针firstNode的值为NULL

链表的构造函数和复制构造函数

```c++
template<class T>
chain<T>::chain(int initialCapacity)
{
	//构造函数
    if(initialCapacity<1)
    {
        ostringstream s;
        s<<"Initial capacity="<<initialCapacity<<"Must be>0";
        throw illegalParameterValue(s.str());
    }
    firstNode=NULL;
    listSize=0;
}

template<class T>
    chain<T>::chain(const chain<T>& theList)
    {
        //复制构造函数
        listSize=theList.listSize;
        
        if(listSize==0)
        {
            //链表theList为空
            firstNode=NULL;
            return;
        }
        
        //链表theList为非空
        chainNode<T>* sourceNode=theList.firstNode;
        //要复制链表theList的节点
        firstNode=new chainNode<T>(sourceNode->element);
        		//复制链表theList的首元素
        sourceNode=sourceNode->next;
        chainNode<T>* targeNode=firstNode;
        //当前链表*this的最后一个节点
        while(sourceNode!=NULL)
        {
            //复制剩余元素
            targetNode->next=new chainNode<T>(sourceNode->element);
            targetNode=targetNode->next;
            sourceNode=sourceNode->next;;
        }
        targetNode->next=NULL;		//链表结束
    }
```

析构函数

析构函数要逐个清除链表的节点。

实现的策略是重复清楚链表的首元素节点，直到链表为空。

我们必须在清除首元素节点之前用变量nextNode保存第2个元素节点的指针。

链表的析构函数

```c++
template<class T>
chain<T>::~chain()
{
	//链表析构函数.删除链表的所有节点
	while(firstNode!=NULL)
	{
		//删除首节点
		chainNode<T>* nextNode=fistNode->next;
		delete firstNode;
		firstNode=nextNode;
	}
}
```

方法get

方法get的返回值是索引为theIndex的元素

```c++
template<class T>
T& chain<T>::get (int theIndex)const
{
    //返回索引为theIndex的元素
    //若该元素不存在，则抛出异常
    checkIndex(theIndex);
    
    //移向所需要的节点
    chainNode<T>* currentNode=firstNode;
    for(int i=0;i<theIndex;i++)
        currentNode=currentNode->next;
    return currentNode->element;
}
```

方法indexOf

返回元素theElement首次出现时的索引

```c++
template<class T>
int chain<T>::indexOf(const T& theElement)const
{
    //返回元素theElement首次出现时的索引
    //若该元素不存在，则返回-1
    
    //搜索链表寻找元素theElement
    chainNode<T>* currentNode=firstNode;
    int index=0;			//当前节点的索引
    while(currentNode!=NULL && currentNode->element!=theElement)
    {
        //移向下一个节点
        currentNode=currentNode->next;
        index++;
    }
    
    //确定是否找到所需的元素
    if(currentNode==NULL)
        return -1;
    else 
        return index;
}
```

方法erase

删除theIndex的元素，需要考虑三种情况

- theIndex<0或者theIndex>=listSize,这是删除操作无效，因为没有这个位置上的元素。这种情况可能表示链表为空
- 删除非空表的第0个元素节点
- 删除其他元素节点

删除索引为theIndex的元素

```c++
template<class T>
void chain<T>::erase(int theIndex)
{
    // 删除索引为theIndex的元素
    //若该元素不存在，则抛出异常
    checkIndex(theIndex);
    
    //索引有效，需找要删除的元素节点
    chainNode<T>* deleteNode;
    if(theIndex==0)
    {
        //删除链表的首节点
        deleteNode=firstNode;
        firstNode=firstNode->next;
    }
    else 
    {
        //用指针p指向要删除节点的前驱节点
        chainNode<T>* p=firstNode;
        for(int i=0;i<theIndex-1;i++)
            p=p->next;
        
        deleteNode=p->next;
        p->next=p->next->next;				//删除 deleteNode指向的节点
        
    }
    listSize--;
    delete deleteNode;
}
```

方法insert

插入元素theElement并使其索引为theIndex

```c++
template<class T>
void chain<T>::insert(int theIndex,const T& theElement)
{
	//在索引为theIndex的位置上插入元素theElement
    if(theIndex<0 || theIndex>listSize)
    {
        //无效索引
        ostringstream s;
        s<<"index ="<<theIndex<<"size ="<<listSize;
        throw illegalIndex(s.str());
    }
    
    if(theIndex==0)
        //在链表头插入
        firstNode=new chainNode<T>(theElement,firstNode);
    else
    {
        //寻找新元素的前驱
        chainNode<T>* p=firstNode;
        for(int i=0;i<theIndex-1;i++)
            p=p->next;
        
        //在p之后插入
        p->next=new chainNode<T>(theElement,p->next);
        
    }
    liseSize++;
}
```

输出链表

方法output

```c++
template<class T>
void chain<T>::output(ostream& out)const
{
	//把链表放入输出流
	for(chainNode<T>* currentNode=firstNode;currentNode!=NULL;currentNode=currentNode->next)
	out<<currentNode->element<<" ";
}
//重载<<
template<class T>
ostream operator<<(ostream& out,const chian<T>& x)
{x.output(out);return out;}
```

链表的成员类iterator

迭代器chain<T>::iterator

```c++
class iterator
{
	public:
	//向前迭代器所需要的typedef语句在此省略
    
   //构造函数
    iterator(chainNode<T>* theNode=NULL)
    {node=theNode;}
    
    //解引用操作符
    T& operator*()const{return node->element;}
    T* operator->()const{return &node->element;}
    
    //迭代器加法操作
    iterator& operator++()			//前加
    {node=node->next;return *this;}
    iterator operator++(int)	//后加
    {iterator old=*this;
    node=node->next;
    return old;}
    
    //相等检验
    bool operator!=(const iterator right)const
    {return node!=right.node;}
    bool operator==(const iterator right)const
    {return node==right.node;}
    protectded:
    chainNode<T>* node;
}
```

在链表chain中，从左至右的访问线性表的元素时，使用get方法和使用迭代器方法，在运行时间上的差别巨大。

迭代器方法的时间比get方法的时间少



抽象数据类型linearList的扩充

clear（清楚表的所有元素）

push_back(theElement)（将元素theElement插入表尾）

对扩展的线性表的抽象类

```c++
template<class T>
class extendedLinearList:linearList<T>
{
    public:
    virtual ~extendedLinearList(){}
    virtual void clear()=0;
    //清表
    virtual void push_back(const T& theElement)=0;
    //将元素theElement插到表尾
}
```

类extendedChain

作为extendedLinearList的链式描述

为了在链表的末端最快地插入一个元素，我们增加一个数据成员lastNode,它是指向链表尾节点的指针

extendedChain完成的工作

声明一个数据成员lastNode

提供改进erase和insert代码

定义linearList中剩余的纯虚函数，使其调用类chain的相应方法

实现方法clear和push_back





类extendedChain<T>的方法clear和push_back

```c++
template<class T>
void extendedChain<T>::clrea()
{
    //删除链表的所有节点
    while(firstNode!=NULL)
    {
        //删除节点firstNode
        chainNode<T>* nextNode=firstNode->next;
        delete firstNode;
        firstNode=nextNode;
    }
    listSize=0;
}

template<class T>
    void extendedChain<T>::push_back(const T& theElement)
    {
        //在链表尾端插入元素theElement的节点
        chainNode<T>* newNode=new chainnode<T>(theElement,NULL);
        if(firstNode==NULL)
            //链表为空
            firstNode=lastNode=newNode;
        else
        {
            //把新元素节点附加到lastNode指向的节点
            lastNode->next=newNode;
        lastNode=newNode;
        }
        listSize++;
    }
```

在一些方法测试中可能出现最坏情况的用时比平均情况的用时还少的情况

这种出现的原因可能是高速缓存的问题





循环链表和头节点

使链表的应用代码简洁和高效

- 把线性表描述成一个单向循环链表(singly linked circular list),而不是单向链表
- 在链表的前面增加一个节点，成为头节点(header node)





搜索带有头节点的循环链表

```c++
template<class T>
circularListWithHeader<T>::circularListWithHeader()
{
    //构造函数
    headerNode=new chainNode<T>();
    headerNode->next=headerNode;
    listSize=0;
}

template<class T>
    int circularListWithHeader<T>::indexOf(const T& theElement)const
    {
        //返回元素theElement首次出现的索引
        //若该元素不存在，则返回-1
        
        //将元素theElement放入头节点
        headerNode->element=theELement;
        
        // 在链表中搜索元素theElement
        chainNode<T>* currentNode=headerNode->next;
        int index=0;	//当前节点的索引
        while(currentNode->element!=theElement)
        {
            	//移动到下一个节点
            currentNode=currentNode->next;
            index++;
        }
			//确定是否找到元素theElement
        if(currentNode==headerNode)
            return -1;
        else 
            return index;
    }
```

6.3 		双向链表

双向链表(doubly linked list)便是这样一个有序的节点序列，其中每个节点都有两个指针：next previous

next指针指向右边节点(如果存在),pervious指针指向左边节点(如果存在)

定义一个双向链表类doublyLinkedList,它用两个数据成员firstNode和lastNode,分别指向链表最左边的节点和最右边的节点.

当双向链表只有一个元素节点p时，firstNode=lastNode=p

当双向链表为空时，firstNode=lastNode=NULL;、





若双向链表成为具有头节点的循环链表

非空双向循环链表中

firstNode.previous是一个指向最右端节点的指针(即firstNode.previous=lastNode),lastNode.next是指向最左端节点的指针



我们可以省略变量firstNode 或 lastNode 只用一个变量来跟踪链表





箱子排序



用一个链表来保存一个班级学生的清单。节点的数据域有:学生姓名，社会保险号码，每次作业贺考试的分数，所有作业和考试的加权总分。

箱子排序(bin sort)是一种更快的排序方法

这种排序首先把分数相同的节点放在同一个箱子里，然后把箱子链接起来就得到有序的链表

​	

箱子排序需要做的是

1. 逐个删除输入链表的节点，把删除的节点分配到相应的箱子里
2. 把每一个箱子中的链表收集并链接起来，使其成为一个有序链表。





用于箱子排序的链表元素结构、

```c++
struct studentRecord
{
	int score;
    string* name;
    int operator!=(const studentRecord& x )const
    {return (score!=x.score);}
}

ostream& operator<<(ostream& out,const studentRecord& x)
{out<<x.score<<' '<<*x.name<<endl;return out;}
```

结构studentRecore的另一种定义

```c++
struct studentRecord
{
	int score;
	string* name;
    operator int() const{return score;}
    //从studentRecord到int的类型转换
    
};

ostream& operator<<(ostream& out,const studentRecord& x)
{out<<x.score<<' '<<x.name<<endl;return out;}
```

结构studentRecord的第三种定义

```c++
struct studentRecord
{
	int score;
    string* name;
    
    int operator!=(const studentRecord& x)const
    {return (score!=x.score);}
	operator int() const{return score;}
};

ostream& operator<<(ostream& out,const studentRecord& x)
{out<<x.score<<' '<<*x.name<<endl;return out;}
```

使用链表的多个方法进行箱子排序

```c++
void binSort(chain<studentRecord>& theChain,int range)
{
    //按分数排序
    	
    	//对箱子初始化
    chain<studentRecord> *bin;
    bin=new chain<studentRecord> [range+1];
    
    //把学生记录从链表取出，然后分配到箱子里
    int numberOfElements=theChain.size();
    for(int i=1;i<=numberOfElements;i++)
    {
        studentRecord x=theChain.get(0);
        theChain.erase(0);
        bin[x.score].insert(0,x);
    }
    
    //从箱子中收集元素
    for(int j=range;i>=0;j--)
        while(!bin[j]empty())
        {
            studentRecord x=bin[j].get(0);
            bin[j].erase(0);
            theChain.insert(0,x);
        }
    
    delete[] bin;
}
```

箱子排序bin sort作为链表chain的一个成员方法

```c++
template<class T>
void chain<T>::binSort(int range)
{
    //对链表中的节点排序
    //创建并初始化箱子
    chainNode<T> **bottom,**top;
    bottom=new chainNode<T>* [range+1];
    top=new chainNode<T>* [range+1];
    for(int b=0;b<=range;b++)
        bottom[b]=NULL;
    
    //把链表的节点分配到箱子
    for(;firstNode!=NULL;firstNode=firstNode->next)
    {
        //把首节点firstNode加到箱子中
        int theBin=firstNode->element;	//元素类型转换到整型int
        if(bottom[theBin]==NULL)		//箱子为空
            bottom[theBin]=top[theBin]=firstNode;
        else
        {
            //箱子不空
            top[theBin]->next=firstNode;
            top[theBin]=firstNode;
        }
    }
    
    //把箱子中的节点收集到有序链表
    chainNode<T> *y=NULL;
    for(int theBin=0;theBin<=range;theBin++)
        if(bottom[theBin]!=NULL)
        {
            //箱子不空
            if(y=NULL)				第一排非空箱子
                firstNode=bottom[theBin];
            else
                y->next=bottom[theBin];
            y=top[theBin];
        }
    if(y!=NULL)
        y->next=NULL;
    
    delete []bottom;
    delete []top;
}
```

每个箱子都以底部节点作为首节点，顶部节点作为尾节点。

每个箱子都有两个指针，分别存储在数组bottom和top中，分别指向尾节点和头节点。

bottom[theBin]指向箱子theBin的尾节点，top[theBin]指向箱子theBin的首节点.

所有箱子开始为空表，这时bottom[theBin]=NULL.



上边的程序从第二个for循环把输入链表的节点逐个插入相应的箱子顶部。

第三个for循环从第0个箱子开始，把非空的箱子依次链接起来，形成一个有序链表。



如果一个排序方法能够保持同值元素之间的相对次序，则该方法称为   稳定排序(stable sort)





**基数排序**(radix sort)



对于箱子排序的扩展方法



扩展后的方法与binSort不同，它不直接对数进行排序，而是把数(number)按照某种基数(radix)分解为数字(digit)

,然后对数字排序

例如:

用10 把928分解为数字9,2,8,			把3725分解为3,7,2,5



用基数60把3725分解为 1，2，5    (3725)@10=(125)@60

这就是   基数排序(radix sort)



**凸包**



至少有三条直线边的平面封闭图形称为多边形(polygon).

一个多边形，如果它的任意两个点的连线都不包含该多边形以外的点，就称为凸多边形(convex polygon)



一个平面点集S的**凸包(convex hull)**是指包含S的最小凸多边形。



该多边形的顶点(即角)称为s的**极点(extreme point)**.





**并查集**

1.等价类

Book’s Page 137 bottom

在**离线等价类(offline equiralence class )**问题中，已知n和R，确定所有的等价类



由等价类的定义得知，每个元素只能属于一个等价类。

在**在线等价类(online equiralence class)**问题中，初始时有n个元素，每个元素都属于一个独立的等价类。

需要执行以下的操作：

1)combine(a,b),把包含a和b的等价类合并成一个等价类

2)find(theElement),确定元素theElement在哪一个类，目的是对给定的两个元素，确定是否属于同一个类。

它对同一类的元素，返回相同的结果，而对不同类的元素，返回不同的结果。





在线等价类问题又被称为 **并查集(union-find)**

例6-5 

机器调度问题



设计一个调度的直观方法如下：

1)按开始时间的非递增次序对任务进行排序

2)按开始时间的非递增次序考察任务。对于每个任务，确定一个空闲的时间段，这个时间段在截止时间之前，但与截止时间最接近。如果这个空闲时间段 位于任务的开始时间之前，则分配失败，否则就把这个时间段分配给该任务。





例6-6 布线问题

**网络net** 是指由电子等价的管脚所构成的最大集合

“最大”  是指不存在网络外的管教与网络内的管教电子等价

在**离线网络搜索问题(offline net finding problem)**中，已知管脚和电线，需要确定相应的网络。如果把每个管脚看成U的一个成员，把每根电线看成R的成员，那么离线网络搜索问题就可以描述为离线等价类问题。

对于**在线网络搜索问题(online net finding problem)**，起始时有一组管脚的集合，没有电线，然后要执行一下操作：

1)增加一根连接a和b的电线

2)搜索包含管脚a的网络

搜索的目的是确定两个管脚是否属于同一个网络。在线网络搜索问题实际上等同于在线等价类问题。

初始时没有电线，相当于R=空，网络搜索操作对应于等价类的find操作，添加电线(a,b)对应于combine(a,b)，它与unite(find(a),find(b))等价

**第一中并查集解决方案**

对在线等价类问题，有一种简单的解决办法。使用一个数组equivClass，且令equivClass[i]为包含元素i的等价类.

n是元素个数。

n和equivClass均为全局变量.

为了合并两个不同的类，我们从两者中任取一个类，然后把该类所有元素的值修改成另一个类元素的值。

函数unite的输入是equivClass的值(即find函数的结果),而不是元素的索引





使用数组实现的并查集算法

```c++
int *equivClass 		//等价类数组
    n;					//元素个数
void initialize(int numberOfElements)
{
    //用每个类的一个元素，初始化numberOfElements个类
    n=numberOfElements;
    equivClass=new int [n+1];
    for(int e=1;e<=n;e++)
        equivClass[e]=e;
}

void unite(int classA,int classB)
{
    //合并类classA和classB
    //假设类classA!=classB
    for(int k=1;k<=n;k++)
        if(equivClass[k]==classB)
            equivClass[k]=classA;
}

int find(int theElement)
{
    //查找具有元素theElement的类
    return equivClass[theElement];
}
```

**第二种并查集解决方案**

如果一个等价类对应一个链表，那么合并操作的时间复杂度就可以降低，因为在一个等价类中，可以沿着链表的指针找到所有的元素，而不必去检查所有的equivClass的值。

事实上，如果知道每一个等价类的大小，我们可以选择较小的类来改变equivClass的值，从而加快合并速度。

通过使用整型指针(也称模拟的指针),可以快速访问元素e的节点。

我们采用以下约定:

- ***equivNode是一个结构，具有数据成员equivClass，size 和next。***
- ***类型为equivNode的数组node[1:n]用于描述n个元素，每个元素都有一个对应的等价类链表***
- ***node[e].equivClass既是函数find(e)的返回值，也是一个整形指针，该指针指向等价类node[e].equivClass的链表的首节点***
- ***只有e是链表的首节点，才定义node[e].size,这时，node[e].size表示从node[e]开始的链表的节点个数***
- ***node[e].next给出了包含节点e的链表的下一个节点。因为节点从1至n编号，所以可以用0来表示空指针NULL***

结构equivNode

```c++
struct equivNode
{
	int equivClass,			//元素类标识符
	size,					//类的元素个数
    next;					//类中指向下一个元素的指针
};
```

使用链表和整型指针实现的并查集算法 

```c++
equivNode *node;			//节点的数组
int n;							//元素个数

void initialize(int numberOfElements)
{
    //用每个类的一个元素，初始化numberOfElements个类
    n=numberOfElements;
    node=new equivNode[n+1];
    
    for(int e=1;e<=n;e++)
    {
        node[e].equivClass=e;
        node[e].next=0;		//链表中没有下一个节点
        node[e].size=1;
    }
}

void unite(int classA,int classB)
{
    //合并类classA和classB
    //假设classA!=classB
    //classA和classB是链表首元素
    
    //使classA成为较小的类
    if(node[classA].size>node[classB].size)
        swap(classA,classB);
    
    //改变较小类的equivClass值
    int k;
    for(k=classA;node[k].next!=0;k=node[k].next)
        node[k].equivClass=classB;
    node[K].equivClass=classB;//链表的最后一个节点
    
    //在链表classB的首元素之后插入链表classA
    //修改新链表的大小
    node[classB].size+=node[classA].size;
    node[k].next=node[classB].next;
    node[classB].next=classA;
}

int find(int theElement)
{
    //查找包含元素theElement的类
    return node[theElement].equivClass;
}
```

### 第七章：数组和矩阵

在实际应用中，数据通常以表的形式出现，尽管用数组来描述表是最自然的方式，但为了减少程序所需的时间和空间，经常采用自定义的描述方式。

例如，当表中大部分数据为0的时候，就会用自定义的描述方式。

矩阵经常用二维数组来描述。

但是，矩阵的索引通常从1开始，但是，C++的二维数组是从0开始。

矩阵的操作有加法，乘法和转置啊，但是C++的二维数组不支持这些操作。

因此我们开发了类matrix，它与矩阵的关系更加密切。

我们还要考察具有特殊结构的矩阵，

例如：对角矩阵，三对角矩阵，三角矩阵和对称矩阵。

关于上述这些矩阵的描述方法，自定义数组和二维数组相比，不仅大大**减少了存储空间**，也**减少了大多数矩阵操作的运行时间**。



本章的最后设计了稀疏矩阵（即大部分元素为0的矩阵）的数组和链表的描述方式，并且对0元素做了特殊的处理。





7.1数组

7.1.1	抽象数据类型

​	一个数组的每一个实例都是形如(索引，值)的数对集合，其中任意两个数对的索引(index)都不相同，
有关数组的操作如下：

- **取值——对一个给定的索引，取对应数对中的值。**
- **存值——把一个新数加到数对集合中。如果已存在一个索引相同的数对，就用新数对覆盖。**

这两个操作定义了抽象数据类型array（ADT7-1)。

```c++
抽象数据类型 array
{
	实例
	形如{index,value}的数对集合，任意两个数对的索引都不同。
	操作
	get(index);返回索引为index的数对的值
	set(index,value);加入一个新数对，如果索引相同的数对已存在，则用新数对覆盖。
}
```

数组的索引又叫做 ——	下标

7.1.3	行主映射和列主映射

索引派列表(column major mapping)

对于二维数组

在行主次序中，映射函数为：

​		map(i1,i2)=i1*u2+i2;

u2是数组的列数。

三维数组的行主映射函数为：

map(i1,i2,i3)=i1*u2 *u3+i2* *u3+i3;

7.1.4	用数组的数组来描述

C++用所谓数组的数组来表示一个多维数组。一个二维数组被表示成一个一维数组，这一个一维数组的每一个元素还是一个一维数组。为表示二维数组

```
int x[3][5];
```

实际上是创建一个长度为3的一维数组x，x的每一个元素是一个长度为5的一维数组。

一个三维数组就被表示为一个一维数组，这个一维数组的每一个元素是一个二维数组。

二维数组的构建方式即是上述的方式。

7.1.5	行主描述和列主描述

另有一种是C++没有的表示方法，它创建一个一维数组，然后利用行主映射或列主映射，把多维数组映射到这个一维数组。

用这种方法把上边的整型数组

```
x[3][5]
```

映射到一个长度为15的整型数组；

int y[15];

它只需要一块连续的，能够容纳15个整数的存储空间，存储空间的大小从72字节降到60字节。

为了访问元素x\[i]\[j],必须利用二维映射函数计算出一个u，然后利用一维映射访问元素y[u].

C++数组中是行主描述法快还是列主描述法快，这取决于是先用一维映射函数定位指针，再用指针定位元素的方法快，还是用二维映射函数定位元素的方法快。

7.1.6	不规则二维数组

所谓规则的二维数组是指每行元素个数相同的二维数组。

见名知意，不规则二维数组

需要注意的是，一个二维数组是否是规则的取决于每一行的元素个数是否相同，而元素的访问方式都是相同的。

​		一个不规则二维数组的创建和使用

```c++
int main(void)
{
    int numberOfRows=5;
    //定义每一行的长度
    int length[5]={6,3,4,2,7};
    
    //声明一个二维数组变量
    //且分配所需要的行数
    int **irregularArray=new int * [numberOfRows];
    
    //分配每一行的空间
    for(int i=0;i<numberOfRows;i++)
        irregularArray[i]=new int [length[i]];
    
    //像使用规则数组一样使用不规则数组
    irregularArray[2][3]=5;
    irregularArray[4][6]=irregularArray[2][3]+2;
    irregularArray[1][1]=3;
    
    //输出选择的数组元素
    cout<<irregularArray[2][3]<<endl;
    cout<<irregularArray[4][6]<<endl;
    cout<<irregularArray[1][1]<<endl;
}
```

7.2	矩阵(matrix)

7.2.1	定义和操作

一个m x n的矩阵(matrix)是一个m行，n列的表，m和n是矩阵的维数(dimension)

矩阵通常用来组织数据

矩阵最常见的操作是矩阵转置，矩阵相加，矩阵相乘。

一个m x n的矩阵M转置之后是一个n x m的矩阵M^T

两个矩阵仅当维数相同时(即它们的行数和列数都分别相等)才可以相加

C(i,j)=A(i,j)+B(i,j)

一个m x n的矩阵A和一个q x p的矩阵B，只有当A的列数等于B的行数（n=q）时，才可以相乘A*B。

A*B=m x p矩阵C



7.2.2 类matrix

一个rows x cols的整型矩阵M可用如下的二维整数数组来描述：

其中M(i,j)对应于x[i-1]\[j-1]

这种描述形式使应用时的数组索引和矩阵索引不同:数组的索引从0开始，矩阵的索引从1开始。

另一种描述形式是把数组x定义为

```
int x[rows+1][cols+1];
```

并且对形式为[0]\[*]\和[\*]\[0]的数组元素弃之不用。

本书开发的一个矩阵描述方法是用行主次序把矩阵映射到一个一维数组中。

类matrix用一个一维数组element存储，在行主次序中，储存rows x cols 矩阵的rows*cols个元素。

下边的程序是类声明(类头),重载了函数操作符(),使得在程序中对矩阵索引的用法和在数学中的一样。

还需要重载算术操作符，使它们能够用于矩阵对象

**矩阵类matrix的声明**

```c++
template<class T>
class matrix
{
    friend ostream& operator<<(ostream&,const matrix<T>&);
    public:
    matrix(int theRows=0,int theColumns=0);
    matrix(const matrix<T>&);
    ~matrix(){delete [] element;}
    int rows()const {return theRows;}
    int columns()const {return theColumns;}
    T& operator()(int i,int j) const;
    matrix<T>& operator=(const matrix<T>&);
    matrix<T> operator+() const;		//unary+
    matrix<T> operator+(const matrix<T>&) const;
    matrix<T> operator-() const;		//unary minus
    matrix<T> operator-(const matrix<T>&) const;
    matrix<T> operator*(const matrix<T>&) const;
    matrix<T> operator+=(const matrix<T>&);
    private:
    int theRows;		//矩阵的行数
    int theColumns;		//矩阵的列数
    T *element;			//数组element
};
```

构造函数不仅生成行数和列数都大于0的矩阵，也生成0 x 0的矩阵



**矩阵类matrix的构造函数和复制构造函数**

```c++
template<class T>
matrix<T>::matrix(int theRows,int theColumns)
{//矩阵构造函数
    //检验行数和列数的有效性
    if(theRows<0) || theColumns<0)
        throw illegalParameterValue("Rows and columns must be >=0");
    if((theRows==0) || theColumns==0)
        		&& (theRows!=0 || theColumns!=0))
        throw illegalParameterValue
        ("Either both or neither rows and columns shoule be zero");
    
    //创建矩阵
    this->theRows=theRows;
    this->theColumns=theColumns;
    element=new T [theRows*theColumns];
}
template<class T>
    matrix<T>::matrix(const matrix<T>& m)
    {
        //矩阵的复制构造函数
        //创建矩阵
        theRows=m.theRows;
        theColumns=m.theColumns;
        element=new T [theRows*theColumns];
        
        //复制m的每一个元素
        copy(m.element,
            m.element+theRows*theColumns,
            element);
    }
```

矩阵类matrix对赋值操作符=的重载

```c++
template<class T>
matrix<T>& matrix<T>::operator=(const matrix<T>& m)
{
	// 赋值.(*this)=m
    if(this!=&m)
    {
        //不能自己复制自己
        delete [] element;
        theRows=m.theRows;
        theColumns=m.theColumns;
        element= new T [theRows*theColumns];
        //复制每一个元素
        copy(m.element,
            m.element+theRows*theColumns,
            element);
    }
    return *this;
}
```

为了用左右括号来表示矩阵索引，我们重载C++的函数操作符(), 它可以具有任意个数的参数，不过在矩阵应用中，我们需要两个整型参数。

**矩阵类 matrix对()操作符的重载**

```c++
template<class T>
T& matrix<T>::operator()(int i,int j) const
{
    //返回对元素element(i,j)的引用
     if(i<1 || i>theRows || j<1 || j>theColumns)
         throw matrixIndexOutOfBounds();
    return element[(i-1)*theColumns+j-1];
}
```

重载操作符+，以实现矩阵加法。因为矩阵被映射到一维数组，所以两个矩阵相加只需要一层for循环。诸如矩阵的一元增值操作(每个元素增加相同的值)和矩阵剑法的代码都与矩阵加法的代码相似.

**矩阵加法**

```c++
tempalate<class T>
    matrix<T> matrix<T>::operator+(const matrix<T>& m) const
    {
        //返回矩阵w=(*this)+m
        if(theRows!=m.theColumns || theColumns!= m.theColumns)
          throw matrixSizeMismatch();
          
        //生成结果矩阵
        matrix<T> w(theRows,theColumns);
        for(int i=0;i<theRows*theColumns;i++)
            w.element[i]=element[i]+m.element[i];
        
        return w;
    }
```

**矩阵乘法**

下方的代码中有三层嵌套for循环。

最内层的循环利用公式来计算矩阵乘积后索引为(i,j)的元素。

进入最内层循环时，element[ct]是第i行的第一个元素，m.element[cm]是第j列的第一个元素。

为了得到第i行的下一个元素，将ct增加1，因为在行主次序中同一行的元素是连续存放的。

为了得到第j列的下一个元素，将cm增加m.theColumns，因为在行主次序中同一列的两个相邻元素在位置上相差m.theColumns

当最内层循环完成时，ct指向第i行的最后一个元素，cm指向第j列的最后一个元素

对于j循环的下一次循环，起始时必须将ct指向第i行的第一个元素，cm指向m的下一列的第一个元素。

对ct的调整时在最内层循环完成后进行的。

当j循环完成时，需要将ct指向下一行的第一个元素，而将cm指向第一列的第一个元素

通过减少缓存未命中事件的次数可以提高矩阵乘法代码的效率

```c++
template<class T>
matrix<T> matrix<T>::operator*(const matrix<T>& m) const
{
    //矩阵乘法 . 返回结果矩阵w=(*this) *m
    if(theColumns!= m.theRows)
        throw matrixSizeMismatch();
    
    matrix<T> w(theRows,m.theColumns);		//结果矩阵
    
    //定义矩阵*this,m和w的游标且初始化以为(2,1)元素定位
    int ct=0,cm=0,cw=0;
    
    //对所有i和j计算w(i,j)
    for(int i=1;i<=theRows;i++)
    {
        //计算结果矩阵的第i行
        for(int j=1;j<=m.theColumns;j++)
        {
            //计算w(i,j)第一项
            T sum=element[ct] *m.element[cm];
            
            //累加其余所有项
            for(int k=2;k<=theColumns;k++)
            {
                ct++;				//*this中第i行的下一项
                cm+=element[ct] *m.element[cm];
            }
            w.element[cw++]=sum;		//存储在w(i,j)
            
            //从行的起点和下一列从新开始
            ct-=theColumns-1;
            cm=j;
        }
        
        //从下一行和第一列重新开始
        ct+=theColumns;
        cm=0;
    }
    return w;
}
```

7.3 	特殊矩阵

7.3.1定义和应用

方阵(square matrix)是行数和列数相同的矩阵

特殊的方阵有：对角矩阵，三对角矩阵，下三角矩阵，上三角矩阵，对称矩阵

栈折叠(stack folding)

7.3.2	对角矩阵

一个 rows x rows的对角矩阵D可以表示为一个二维数组element[rows]\[rows],其中element[i-1]\[j-1]表示D(i,j)

这种表示法需要rows^2个类型为T的数据空间

但是对角矩阵中最多只有rows个非0元素，因此可以用一维数组element[rows]来表示对角矩阵，其中element[i-1]表示D(i,j)

所有未在一维数组中出现的矩阵元素均为0

这种表示方法仅需rows个类型为T的数据空间，而且产生了C++类diagonalMatrix

类diagonalMatrix的声明和构造函数

```c++
template<class T>
class diagonalMatrix
{
    public:
    diagonalMatrix(int theN=10);
    ~diagonalMatrix()[delete [] element;]
        T get(int ,int ) const;
    void set(int ,int ,const T&);
    private:
    int n;				//矩阵维数
    T *element;			//存储对角矩阵的一维数组
};

template<class T>
    diagonalMatrix<T>::diagonalMatrix(int theN)
    {
        //构造函数
        //检验theN的值是否有效
        if(theN<1)
            throw illegalParameterValue("Matrix size must be >0");
        
        n=theN;
        element=new T[n];
    }
```

类diagonalMatrix的方法get

```c++
template<class T>
T diagonalMatrix<T>::get(int i,int j) const
{
    //返回矩阵中(i,j)位置上的元素
    //检验i和j的值是否有效
    if(i<1 || j<1 || i>n || j>n)
        throw matrixIndexOfBounds();
    
    if(i==j)
        return element[i-1];		//对角线上的元素
    else
        return 0;						//非对角线上的元素
}
```

类diagonalMatrix的方法set

```c++
template<class T>
    void diagonalMatrix<T>::set(int i,int j,const T& newValue)
    {
        //存储(i,j)项的新值
        //检查i和j的值是否有效
        if(i<1	||	j<1	||	i>n	||	j>n)
            throw matrixIndexOfBounds();
        
        if(i==j)
            //存储对角元素的值
            element[i-1]=newValue;
        else 
            //非对角元素的值必须是0
            if(newValue!=0)
                throw illegalParameterValue
                ("nondiagonal elements must be zero");
    }
```

7.3.3	三对角矩阵

在一个rows x rows的三对角矩阵中，非0元素排列在如下的三条对角线上：

1)主对角线----i=j

2)主对角线之下的对角线（低对角线）-----i=j+1;

3)主对角线之上的对角线（高对角线）-----i=j-1;

这三条对角线上的元素总数为3**rows-2,可以用一个容量为3**rows-2的一维数组element来描述三对角矩阵

因为只有三条对角线上的元素需要真正的存储

三对角矩阵映射到数组element有三种不同方式

数据成员和构造函数与类diagonal的相似

三对焦矩阵的方法get

```c++
template<class T>
T tridagonalMatrix<T>::get(int i,int j) const
{
    //返回矩阵中(i,j)位置上的元素
    
    //检验i和j的值是否有效
    if(i<1	||	j<1	||	i>n	||	j>n)
        throw matrixIndexOfBounds();
    
    //确定要返回的元素
    switch(i-j)
    {
        case 1:		//下对角线
            return element[i-2];
        case 0:		//主对角线
            return element[n+i-2];
        case -1:	//上对角线
            return element[2*n+i-2];
        default:	return 0;
    }
}
```

7.3.4	三角矩阵

不论是上三角还是下三角的矩阵，都可以用一个大小为n(n+1)/2的一维数组来表示。

方法lowerTriangularMatrix<T>::set

```c++
template<class T>
void lowerTriangularMatrix<T>::set(int i,int j,const T& newValue)
{
    //给(i,j)元素赋新值
    //检验i和j的值是否合法
    if(i<1	||	j<1	||	i>n	||	j>n)
        throw matrixIndexOfBounds();
    
    //(i,j)在下三角，当且仅当i>=j
    if(i>=j)
        element[i*(i-1)/2+j-1]=newValue;
    else
        if(newValue!=0)
            throw illegalParameterValue
            ("elements not in lower triangle must be zero");
}
```

7.3.5	对称矩阵

一个n	x	n对称矩阵，可以视为下三角或上三角矩阵，用三角矩阵的表示方法，用一个大小为n(n+1)/2的一维数组来表示

**7.4	稀疏矩阵**

7.4.1	基本概念

一个m	x	n的矩阵，如歌大多数元素都是0，则成为**稀疏矩阵(sqare matrix)**

一个矩阵如果不是稀疏的，就称为**稠密矩阵(dense matrix)**

在稀疏矩阵和稠密矩阵之间没有明显的界限

n x n的对角矩阵和三对角矩阵都是稀疏矩阵

7.4.2	用单个线性表描述

为了重建矩阵结构，必须记录每个非0元素的行号和列号，因此数组元素需要三个域：

row(矩阵元素所在行),col(矩阵元素所在列),value(矩阵元素的值)

为此自定义结构matrixTerm，使其有三个数据成员:整型row,col，T类型value

1.类sqareMatrix

1. reSet(newSize)把数组的元素个数改为newSize，必要时增大数组容量
2. set(theIndex,theElement)使元素theElement成为表中索引为theIndex的元素
3. clear()使表的元素个数为0



类sqareMatrix的头

```c++
template<class T>
class sqarseMatrix
{
    public:
    void transpose(sqarseMatrix<T> &b);
    void add(sqarseMatrix<T> &b,sqarseMatrix<T> &c);
    private:
    int rows,								//矩阵行数
    	cols;								//矩阵列数
    	arrayList<matrixTerm<T>>terms;	//非0项表
}；
```

重载输出操作符<<

这个代码对类arrayList的元素使用了从左至右的顺序迭代器，按行主次序提取矩阵非0元素，一行输出一个矩阵项

```c++
template<class T>
ostream& operator<<(ostream& out,sqarseMatrix<T>& x)
{
    //将x放入输出流
    
    //输出矩阵特征
    out<<"rows="<<x.rows<<"columns="
        <<x.cols<<endl;
    out<<"nonzero terms="<<x.terms.size()<<endl;
    
    //输出矩阵项，一行一个
    for(arrayList<matrixTerm<T>>::iterator i=x.terms.begin();i!=x.terms.end();i++)
        out<<"a("<<(*i).row<<','<<(*i).col<<")="<<(*i).value<<endl;
        
        return out;
}
```

重载输入操作符>>

该代码是按行主次序输入稀疏矩阵元素，建立内部表示

```c++
template<class T>
istream& operator>>(istream& in,sqarseMatrix<T>& x)
{
    //输入一个稀疏矩阵
    
    //输入矩阵特征
    int numberOfTerms;
    cout<<"Enter number of rows,columns,and #terms"
        <<endl;
    in>>x.rows>>x.cols>>numberOfTerms;
    //这里应该检验输入的合法性，留作练习
    if(x.rows<1	||	x.cols<1	||	numberOfTerms<0)
        throw matrixIndexOfBounds();
    //设置x.terms的大小，确保足够的容量
    x.terms.reSet(numberOfTerms);
    //输入项
    matrixTerm<T> mTerm;
    for(int i=0;i<numberOfTerms;i++)
    {
        cout<<"Enter row,column,and value of term"
            <<(i+1)<<endl;
        in>>mTerm.row>>mTerm.col>>mTerm.value;
        //这里应该检验输入的合法性，留作练习
        if(mTerm.row<1	||	mTerm.col<1)
            throw matrixIndexOfBounds();
        if(mTerm.value==0)
            throw illegalParameterValue
            ("mTerm.value must be not equal to zero");
        x.terms.set(i,mTerm);
    }
    
    return in;
}
```

2.矩阵转置

矩阵转置方法transpose

转置后的矩阵存储在矩阵b中

首先设置b的行数和列数。然后使线性表b.terms的元素个数等于被转置矩阵的元素个数

**尽管这时候线性表b.terms还没有元素，但还是要令它的元素个数等于它最后将具有的元素个数**

这样之后可以使用方法arrayList<T>::set把转置后的矩阵元素存储到线性表b.terms的任意相应位置

否则，就要借助插入操作，每插入一个元素，增加一次线性表的元素个数

可是，当我们转置一个稀疏矩阵时，原来的第0个元素可能成为转置后的第6个元素(比如说)，而除非线性表的元素个数是6或更多，否则我们是不能把一个元素插到第6个位置上去的。因此，我们实质上是把线性表作为一个一维数组，一开始的元素个数等于它最终等于㢝个数，然后通过方法set给任一个位置赋一个新值



接下来是创建两个数组colSize和rowNext.

colSize[i]是输入矩阵*this在第i列的非0元素个数，rowNext[i]是转置矩阵第i行首个非0元素在b中的索引。

对colSize的计算在前两个for循环中完成：使用迭代器，检查每个输入矩阵元素



对rowNext的计算在第三个for循环中完成：rowNext[i]的值是转置矩阵b在第0行至第i-1行的元素个数，即输入矩阵*this的第0列至第i-1列的元素个数

在最后一个for循环中，非0元素被复制到b中相应位置

**稀疏矩阵的转置**

```c++
template<class T>
    void sparseMatrix<T>::transpose(sqarseMatrix<T> &b)
    {
        //返回b中*this的转置
        
        //设置转置矩阵特征
        b.cols=rows;
        b.rows=cols;
        b.terms.reSet(terms.size());
        
        //初始化以实现转置
        int* colSize=new int[cols+1];
        int* rowNext=new int[cols+1];
        
        //寻找*this中每一列的项的数目
        for(int i=1;i<=cols;i++)		// 初始化
            colSize[i]=0;
        for(arrayList<matrixTerm<T>>::iterator i=terms.begin();i!=terms.end();i++)
            colSize[(*i).col]++;
        
        //寻找b中每一行的起始点
        rowNext[1]=0;
        for(int i=2;i<=cols;i++)
            rowNext[i]=rowNext[i-1]+colSize[i-1];
        
        //实施从*this到b的转置复制
        matrix<T> mTerm;
        for(arrayList<matrixTerm<T>>::iterator i=terms.begin();i!=terms.end();i++)
        {
            int j=rowNext[(*i).col]++;		//b中的位置
            mTerm.row=(*i).col;
            mTerm.col=(*i).row;
            mTerm.value=(*i).value;
            b.terms.set(j,mTerm);
        }
    }
```

与矩阵的二维数组表示相比，尽管上边的程序更加复杂，但是对于有很对个0元素的矩阵来说，它要快得多。





**3.两个矩阵相加**

两个稀疏矩阵相加

```c++
template<class T>
void sparseMatrix<T>::add(sparseMatrix<T> &b,sparseMatrix<T> &c)
{
    //计算c=(*this)+b
    
    //检验相容性
    if(rows!=b.rows	||	cols!=b.cols)
        throw matrixSizeMismatch();		//矩阵不相容
    //设置结果矩阵c的特征
    c.rows=rows;
    c.cols=cols;
    c.terms.clear();
    int cSize=0;
    
    //定义*this和b的迭代器
    arrayList<matrixTerm<T>>::ieratror it=terms.begin();
    arrayList<matrixTerm<T>>::ieratror ib=b.terms.begin();
    arrayList<matrixTerm<T>>::ieratror itEnd=terms.end();
    arrayList<matrixTerm<T>>::ieratror ibEnd=b.terms.end();
    
    //遍历*this和b， 把相关的项相加
    while(it!=itEnd && ib!=ibEnd)
    {
        //行主索引加上没一项的列数
        int tIndex=(*it).row*cols+(*it).col;
        int bIndex=(*ib).row*cols+(*ib).col;
        
        if(tIndex<bIndex)
        {
            //b项在后
            c.terms.insert(cSize++,*it);
            it++;
        }
        else {
            if(tIndex=bIndex)
            {
                //两项同在一个位置
                
                //仅当相加后部位0时加入c
                if(*it).value+(*ib).value!=0
                {
                    matrixTerm<T> mterm;
                    mTerm.row=(*it).row;
                    mTerm.col=(*it).col;
                    mTerm.value=(*it).value+(*ib).value;
                    c.terms.insert(cSize++,mTerm);
                }
                
                it++;
                ib++;
            }
            else
            {
                //一项在后
                c.terms.insert(cSize++,*ib);
                ib++;
            }
        }
    }
    
    //复制剩余项
    for(;it!=itEnd;it++)
        c.terms.insert(cSize++,*it);
    for(;ib!=ibEnd;ib++)
        c.terms.insert(cSize++,*ib);
}
```

**7.4.3	用多个线性表描述**

如果把每一行的非0元素用一个线性表存储，就得到描述稀疏矩阵的另一种方法

1.表示方法



把每行的非0元素串接在一起，构成一个链表，称作**行链表(row chain)**



每一个非阴影节点都代表稀疏矩阵的一个非0元素

在行链表中，每一个节点有两个域：element(数据域),next(指针域)

数据域element有两个子域：col(元素的列号),value(元素的值)

一行至少有一个非0元素，才会建立该行的行链表

行链表的节点按列号升序链接在一起

所有行链表(即非阴影链表)被另外一个链表(称为头节点链表)收集在一起

头节点链表的节点也有两个域：element,next

element有两个子域：row(与行链表对应的行号),rowChain(指向行链表，rowChain firtNode指向行链表的第一个非阴影节点)



头节点链表的节点按行号升序的顺序链接在一起，每个节点可以被视为一个行链表的头节点，空的头节点链表表示没有非0元素的矩阵





2.链表元素类型



结构rowElement定义了行链表的元素类型

它的数据成员有col(元素的行索引)和value(元素的值)

结构headerElement定义了头节点链表的元素类型

它的数据域有row(行索引)和rowChain(实际的链表，数据类型是extendedChain)





3.类linkedMatrix



我们都是在链表的右端插入节点(附加节点)



函数zero()，它使得链表的元素个数的值为0，但不删除链表节点(它与函数clear不同，clear会删除所有的链表节点)





linkedMatrix的数据成员与spareMatrix的数据成员几乎相同

不同之处是数据成员terms被headerChain取代，后者的类型是extendedChain



4.转置方法linkedMatrix<T>:transpose





关于转置操作，我们使用箱子从输入矩阵*this中收集的那些在结果矩阵中位于同一行的非0元素



bin[i]是与结果矩阵b的第i行非0元素所对应的链表



一个稀疏矩阵的转置

```c++
template<class T>
void linkedMatrix<T>::transpose(linkedMatrix<T> &b)
{
    //将*this的转置在矩阵b中返回
    b.headerChain.clear();			//从b中删除所有节点
    
    //创建bins以收集b的行
    extendedChian<rowElement<T> > *bin;
    bin=new extendedChain<rowElement> > [cols+1];
    
    //头节点迭代器
    extendedChain<headerElement<T>>::iterator
        ih=headerChain.begin();
    	ihEnd=headerChain.end();
    
    //把*this的项复制到this
    while(ih!=ihEnd)
    {
        // 检查所有行
        int r=ih->row;			//行链表的行数
        
        //行链表迭代器
        extendedChain<rowElement<T>>::iterator
            ir=ih->rowChain.begin();
        	irEnd=ih->rowChain.end();
        
        rowElement<T> x;
        //将*this中行r的项复制到b中的列r
        x.col=r;
        
        while(ir!=irEnd)
        {
            //把行链表的一项复制到bin
            x.value=ir->value;
            //x最终在转置矩阵的行ir->col中
            bin[ir->col].push_back(x);
            ir++;			//行中的下一项  
        }
        ih++;				//进入下一行
    }
    
    // 设置转置矩阵的维数
    b.rows=cols;
    b.cols=rows;
    
    //收集转置矩阵的头链表
    headerElement<T> h;
    //扫描this
    for(int i=1;i<=cols;i++)
        if(!bin[i].empty())
        {
            //转置矩阵的行i
            h.row=i;
            h.rowChain=bin[i];
            h.headerChain.push_back(h);
            bin[i].zero();					//免于析构
        }
    h.rowChain.zero();						//免于析构
    
    delete [] bin;
}
```

# 第八章 					栈

栈和队列很可能是应用频率最高的数据结构



栈和队列是线性表和有序表的限制版



 把线性表的插入和删除操作限制在同一端进行，就得到栈数据结构



因此，栈是一个后进先出(last-in-first-out,LIFO)的数据结构



栈是一种特殊的线性表，所以从相应的线性表类派生出栈类是很自然的事情



## 8.1	定义和应用

**定义8-1	栈(stack)是一种特殊的线性表，其插入(也称入栈或压栈) 和删除(也称出栈或弹栈)操作都在表的同一端进行。这一端叫栈顶(top),另一端称为栈底(bottom)**





计算机执行递归函数的方式：使用**递归工作栈(recursion stack)**

当一个函数被调用时，一个返回地址(即被调函数一旦执行完，接下去要执行的程序指令的地址)和被调函数的局部变量和形参的值都要存储在递归工作栈中。

当执行完一次返回时，被调函数的局部变脸和形参的值回复被调用之前的值(这些值存储在递归工作栈的顶部)，而且程序从返回地址处继续执行，这个返回地址也存储在递归工作栈的顶部。





## 8.2	抽象数据类型

栈操作方法的名称与STL中栈类的一样

```c++
抽象数据类型	stack
{
    实例
        线性表:一端为底，另一端为顶
    操作
            empty();//栈为空时返回true，否则返回false
    		  size();//返回栈中元素个数
    		  top();//返回栈顶元素
    		  pop();//删除栈顶元素
    		  push(x);//将元素x压入栈顶
}；
```

下边的程序给出了抽象数据类型stack对应的C++抽象类

```c++
template<class T>
class stack
{
    public:
    virtual ~stack(){}
    virtual bool empty() const=0;
    				//返回true，当且仅当栈为空
    virtual int size() const=0;
    				//返回栈中元素个数
    virtual T& top()=0;
    				//返回栈顶元素的引用
    virtual void pop();
    				//删除栈顶元素
    virtual void push(const T& theElement)=0;
    				//将元素theElement 压入栈顶
};
```

## 8.3	数组描述

栈时一种插入和删除操作都被限制在一端进行的线性表，因此可以使用线性表的描述方法



### 8.3.1	作为一个派生类实现

类derivedArrayStack的构造函数直接调用类arrayList的构造函数

动态创建一维数组，数组容量(长度)是参数initialCapacity的值

initialCapacity的缺省值是10

其他成员函数的代码也是直接调用基类的成员函数来实现的

**一个从类arrayList派生的数组栈类**

```c++
template<class T>
class derivedArrayStack:private arrayList<T>,public stack<T>
{
    public:
    derivedArrayStack(int initialCapacity=10):arrayList<T>(initialCapacity){}
    bool empty() const
    {return arrayList<T>::empty();}
    int size() const
    {return arrayList<T>::size();}
    T& top{
        if(arrayList<T>::empty())
            throw stackEmpty();
        return get(arrayList<T>::size()-1);
    }
    void pop()
    {
        if(arrayList<T>::empty())
            throw stackEmpty();
        erase(arrayList<T>::size()-1);
    }
    void push(const T& theElement)
    (insert(arrayList<T>::size(),theElement);)
};
```

#### 2.	对类derivedArrayStack的评价

函数top和pop都在类方法get和erase之前检查栈是否为空---------这部分是可以**考虑**删除的

具体解释 Page179



### 8.3.2	类arrayStack

通过线性表类的派生得到一个栈类时，付出的代价是性能的损失





类arrayStack

```c++
template<calss T>
class arrayStack:public stack<T>
{
    public:
    arrayStack(int initialCapacity=10);
    ~arrayStack(){delete [] stack;}
    bool empty() const {return stackTop==-1;}
    int size() const
    {return stackTop+1;}
    T& top()
    {
        if(stackTop==-1)
            throw stackEmpty();
        return stack[stackTop];
    }
    void pop()
    {
        if(stackTop==-1)
            throw stackEmpty();
        stack[stackTop--].~T();		// T的析构函数
    }
    void push(const T& theElement);
    private:
    int stackTop;					//当前栈顶
    int arrayLength;				//栈容量
    T *stack;						 //元素数组
}；
    
template<calss T>
arrayStack<T>::arrayStack(int initialCapacity)
{
    //构造函数
    if(initialCapacity<1)
    {
        ostringstream s;
        s<<"Initial capacity ="<<initialCapacity<<"Must be > 0";
        throw illegalParameterValue(s.str());
    }
    arrayLength=initialCapacity;
    stack=new T[arrayLength];
    stackTop=-1;
}

template<class T>
void arrayStack<T>::push(const T& theElement)
{
    //将元素theElement压入栈
    if(stackTop==arrayLength-1)
    {
        //空间已满，容量加倍
        changeLength(stack,arrayLength,2*arrayLength);
    }
    
    //在栈顶插入
    stack[++stackTop]=theElement;
}
```

## 8.4	链表描述

基于时间复杂度的考虑

选择**链表的左端作为栈顶**





### 8.4.1	类**derivedLinkedStack**

从chain派生，实现了抽象类stack，其代码是从derivedLinkedStack中得到的





### 8.4.2	类linkedStack



就像arrayStack，定制代码而不是从类chain派生，由此改进代码的时间性能



定制链表栈

```c++
template<class T>
class linkedStack:public stack<T>
{
    public:
    linkedStack(int initialCapacity=10)
        (stackTop=NULL;stackSize=0;)
        ~linkedStack();
    bool empty() const
    {return stackSize==0;}
    int size() const
    {return stackSize;}
    T& top()
    {
        if(stackSize==0)
            throw stackEmpty();
        return stackTop->element;
    }
    void pop();
    void push(const T& theElement)
    {
        stackTop=new chainNode<T>(theElement,stackTop);
        stackSize++;
    }
    private:
    chainNode<T>* stackTop;			//栈顶指针
    int stackSize;					  //栈中元素个数
};

template<class T>
linkedStack<T>::~linkedStack()
{
    //析构函数
    while(stackTop!=NULL)
    {
        //删除栈顶节点
        chainNode<T>* nextNode=stackTop->next;
        delete stackTop;
        stackTop=nextNode;
    }
}

template<class T>
void linkedStack<T>::pop()
{
    //删除栈顶节点
    if(stackSize==0)
        throw stackEmpty();
    
    chainNode<T>* nextNode=stackTop->next;
    delete stackTop;
    stackTop=nextNode;
    stackSize--;
}
```





## 8.5	应用





### 8.5.1	括号匹配



1.问题描述



需要做的：对一个字符串的左右括号进行匹配

例如说,字符串(a*(b+c)+d)在位置0和3有左括号，在位置7和10有右括号

位置0的左括号和位置10的右括号匹配，位置3的左括号和位置7的右括号匹配

在字符串(a+b))(中，位置5的右括号没有与之匹配的左括号，位置6的左括号没有与之匹配的右括号

我们的目的即是编写一个C++程序，其输入是一个字符串，输出是匹配的括号以及不匹配的括号

括号匹配问题等价于C++程序中的{}匹配问题

2.求解策略

从左至右扫描，把没有匹配的左括号放在栈顶的位置，每当扫描到一个右括号，就将它与栈顶的左括号(如果存在)相匹配，并将匹配的左括号从栈顶删除

3.C++实现



```c++
void printMatachedPairs(string expr)
{
    //括号匹配
arrayStack<int> s;
int length=(int) expr.size();

//扫描表达式expr寻找左括号和右括号
for(int i=0;i<length;i++)
    if(expr.at(i)=='(')
        s.push(i);
		else
            if(expr.at(i)==')')
                try
                {
                    //从栈中删除匹配的左括号
                    cout<<s.top()<<' '<<i<<endl;
                    s.pop();		// 没有栈匹配
                }
catch (stackEmpty)
{
    // 栈为空，没有匹配的左括号
    cout<<"No match for right parenthesis"
        <<" at "<<i<<endl;
}

	//栈不为空 剩余在栈中的左括号是不匹配的
	while(!s.empty())
	{
    	cout<<"No match for left parenthesis at"
   	     <<s.top()<<endl;
	    s.pop();
	}
}
```

### 8.5.2	汉诺塔

1.问题描述

汉诺塔(Towers of Hanoi)问题来自大梵天创世的一个古老传说

在创世之日，有一座钻石宝塔，其上有64个金碟，所有碟子从大到小从塔底到塔顶，旁边还有另外两座钻石宝塔

从创世之日起，婆罗门一直试图把塔1上的碟子移到塔2上去，不过要借助塔3

由于碟子非常重，每次只能移动一个碟子

大碟子要放在小碟子下边

2.求解策略

一个简洁的方法就是递归



为了把最大的碟子移到塔2的底部，必须把其余n-1个碟子移到塔3，然后把最大的碟子移到塔2，接下来是把塔3上的n-1个碟子移到塔2



3.第一种实现方法



求解汉诺塔问题的递归方法

```c++
void towerOfHanoi(int n,int x,int y,int z)
{
    //把塔x顶部的n个碟子移到塔y
    //用塔z作为中转地
    if(n>0)
    {
        towerOfHanoi(n-1,x,z,y);
        cout<<"Move top disk from tower"<<x
            <<"to top of tower"<<y<<endl;
        towerOfHanoi(n-1,z,y,x);
    }
}
```





5.第二种实现方法



把每个塔表示成一个栈

三座塔在任何时候都总共有n个碟子

如果使用链表形式的栈，只需申请n个元素空间

如果使用数组形式的栈，那么因为塔1和塔2的容量都必须是n，塔3的容量必须是n-1.因此需要的总空间为3n-1





数组形式的栈比连败哦形式的栈运行速度快



使用栈求解汉诺塔问题

```c++
//全局变量，tower[1:3]表示三个塔
arrayStack<int> tower[4];

void moveAndShow(int ,int ,int , int);

void towerOfHanoi(int n)
{
    //函数moveAndShow的预处理程序
    for(int d=n;d>0;d--)			//初始化
        tower[1].push(d);			//把碟子d加到塔1
    
    //把n个碟子从塔1移到塔3，用塔2做中转站
    moveAndShow(n,1,2,3);
}

void moveAndShow(int n,int x,int y,int z)
{
    //把塔x顶部的n个碟子移到塔y，显示移动后的布局
    //用塔z作为中转站
    if(n>0)
    {
        moveAndShow(n-1,x,z,y);
        int d=tower[x].top();			//把一个碟子
        tower[x].pop();					 //从塔x 的顶部移到
        tower[y].push(d);				 //塔y的顶部
        showState();					  //显示塔3的布局
        moveAndShow(n-1,z,y,x);
    }
}
```

### 8.5.3	列车车厢重排

1.问题描述

一列货运列车有n节车厢，每节车厢要停靠在不同的车站

假设n个车站从1到n编号，而且货运列车按照从n到1的顺序经过车站

车厢的编号与它们要停靠的车站编号相同

为了便于从列车上卸掉相应的车厢，必须按照从前至后，从1到n的顺序把车厢重新排列

这样排列之后，在每个车站只需卸掉最后一节车厢即可

车厢重排工作在一个转轨站(shunting yard)上进行，转轨站上有一个入轨道(input track),一个出轨道(output track)和k个缓冲轨道(holding track)

缓冲轨道位于入轨道和出轨道之间



2.求解策略

为了重排车厢，我们从前至后检查入轨道上的车厢

如果正在检查的车厢是满足排列要求的下一节车厢，就直接把它移到出轨道上

如果不是，就把它移到一个缓冲轨道上，知道它满足排列要求时才将它移到出轨道上

缓冲轨道时按照LIFO的方式管理的，车厢的进出都在缓冲轨道的顶部进行

在重排车厢过程中，仅允许以下移动：

- 车厢可以从入轨道的前端( 即右端)移动到一个缓冲轨道的顶部或出轨道的后端(即左端)

- 车厢可以从一个缓冲轨道的顶部移到出轨道的后端





对缓冲轨道选择的最基本要求是：编号为u的车厢应该进入的缓冲轨道，其顶部车厢编号是大于u的最小者



我们需要依据这条   **分配规则(assignment rule)**	来选择缓冲轨道





3.C++实现



用k个数组栈来表示k个缓冲轨道



列车车厢重排程序的全局变量

```
arrayStack<int> *track;				//缓冲轨道数组
int numberOfCars;
int numberOfTracks;
int samllestCar;					 //在缓冲轨道中编号最小的车厢
int itsTrack;						 //停靠着最小编号车厢的缓冲轨道
```

函数railroad计算一个车厢的移动序列，以此完成车厢的重排

车厢初始顺序为inputOrder[1:theNumberOfCars],缓冲轨道最大为theNumberOfTracks

如果移动序列不存在，railroad的返回值是false，否则是true



函数railroad从创建从创建栈track开始，track[i]代表缓冲轨道i，1<=i<=numberOfTracks



for循环开始，编号为nextCarToOutput的车厢并没有在缓冲轨道上



在for循环的第i此迭代中，入轨道的车厢unputOrder[i]被调出

若inputOrder[i]=nextCarToOutput，则该车厢直接被移到出轨道，然后nextCarToOutput的值增1

这时，缓冲轨道可能有若干节车厢可以进入出轨道，把这些车厢移到出轨道的过程有while循环完成

如果车厢inputOrder[i]不能移到出轨道，那么就没有车厢可以移到出轨道

这时，按照以及确定的轨道分配规则，把车厢inputOrder[i]移到一个缓冲轨道



函数railroad

```c++
bool railroad(int inputOrder[],int theNumberOfCars,int theNumberOfTracks)
{
	//从初始顺序开始重排车厢
    //如果重排成功，返回true，否则返回false
    
    numberOfCars=theNumberOfCars;
    numberOfTracks=theNumberOfTracks;
    
    //创建用于缓冲轨道的栈 
    track=new arrayStack<int> [numberOfTracks+1];
    
    int nextCarToOutput=1;
    smallestCar=numberOfCars+1;			//缓冲轨道中无车厢
    
    //重排车厢
    for(int i=1;i<=numberOfCars;i++)
        if(inputOrder[i]==nextCarToOutput)
        {
            //将车厢inputOrder[i]直接移到出轨道
            cout<<"Move car"<<inputOrder[i]
                <<"from input track to output track"<<endl;
            nextCarToOutput++;
            
            //从缓冲轨道移到出轨道
            while(smallestCar==nextCarToOutput)
            {
                outputFromHoldingTrack();
                nextCarToOutput++;
            }
        }
    else
        //将车厢inputOrder[i]移到一个缓冲轨道
        if(!putInHoldingTrack(inputOrder[i]))
            return false;
    
    return true;
}
```







函数outputFromHoldingTrack

```c++
void outputFromHoldingTrack
{
    //将编号最小的车厢从缓冲轨道移到出轨道
    
    //将栈itsTrack中删除编号最小的车厢
    track[itsTrack].pop();
    cout<<"Move car"<<smallestCar<<"from holding"
        <<"track"<<itsTrack<<"to output track"<<endl;
    
    //检查所有栈的栈顶，寻找编号最小的车厢和它所属的栈itsTrack
    smallestCar=numberOfCars+2;
   	for(int i=1;i<=numberOfTracks;i++)
        if(!track[i].empty() && (track[i].top()<smallestCar))
        {
            smallestCar=track[i].top();
            itsTrack=i;
        }
}
```





函数putInHolding Track

```c++
bool putInHoldingTrack(int c)
{
    //将车厢c移到一个缓冲轨道，返回false，当且仅当没有可用的缓冲轨道
    
    //为车厢c寻找最适合的缓冲轨道
    //初始化
    int bestTrack=0;					//目前最合适的缓冲轨道
    bestTop=numberOfCars+1;			  //去bestTrack中顶部的车厢
    
    //扫描缓冲轨道
    for(int i=1;i<=numberOfTracks;i++)
        if(!track[i].empty())
        {
            //缓冲轨道i不空
            int topCar=track[i].top();
            if(c<topCar && topCar<bestTop)
            {
                //缓冲轨道i的栈顶具有编号更小的车厢
                bestTop=topCar;
                bestTrack=i;
            }
        }
    else	//缓冲轨道i为空
        if(bestTrack==0) bestTrack=i;
    
    if(bestTrack==0) return false;		//没有可用的缓冲轨道
    
    //把车厢c移到轨道bestTrack
    track[bestTrack.push(c);
          cout<<"Move car"<<c<<"from input track"
          <<"to holding track"<<bestTrack<<endl;
          
          // 如果需要，更新smallestCar和itsTrack
          if(c<smallestCar)
          {
              smallestCar=c;
              itsTrack=bestTrack;
          }
          
          return true;
}
```





### 8.5.4	开关盒布线

1.问题描述

在开关盒布线问题中，给定一个矩形布线区域，其外围有若干管脚



两个管脚之间通过布设一条金属线路来连接

这条金属线路称为电线，他被限制在矩形区域内



两条电线交叉会发生电流短路，因此，电线不允许交叉

每对要连接的管脚称为一个网组

对于给定的一些网组，我们要确定，它们能否连接而又不发生交叉





布线可以没有交叉，这个开关盒称为 **可布线开关盒(routable switch box)**



我们的问题是，输入一个开关盒布线实例，然后确定它是否可以布线



2.求解策略



为了解决开关盒布线问题，注意，当一个网组互连时，连线把布线区域分隔成两个分区

分区边界上的管脚属于哪一个分区的与连线无关，而与互连网组的管脚有关



如果是可布线的，检查完所有管脚之后栈为空，若是不可布线，则栈不为空



3.C++实现



假设管脚的数目已知，每个管脚都对应一个网组编号



开关盒布线

```c++
bool checkBox(int net[],int n)
{
    //确定开关和是否可布线
    //数组net[0..n-1]管脚数组，用以形成网组
    //n是管脚个数
    
    arrayStack<int>* s=new arrayStack<int>(n);
    
    //按顺时针扫描网组
    //处理管脚i
    if(!s->empty())
        //检查栈的顶部管脚
        if(net[i]==net[s->top()])
            //管脚net[i]是可布线的，从栈中删除
            s->pop();
    	else s->push(i);
    else s->push(i);
    
    //是否有剩余的不可布线的管脚
    if(s->empty())
    {
        //没有剩余的管脚
        cout<<"Switch box is routable"<<endl;
        return true;
    }
    cout<<"Switch box is not routable"<<endl;
    return false;
}
```





### 8.5.5	离线等价类问题

1.问题描述

等价类是满足自反性，传递性的一组关系，其具有find和union两个方法。这之前我们已经说过了。
这里我们介绍的离线等价类是find的升级版本，find只找寻root，而离线等价寻找，一个等价类的所有元素。
其原理就是首先随便寻找一个等价类元素，然后利用传递性去把所有和他相关的元素找出来



这个问题的输入是元素数目n，关系对数目r以及r个关系对

目标是把n个元素划分为等价类



2.求解策略



- 输入数据，建立n个表以表示关系对。	对每个关系对(i,j)，i放在list[j],j放在list[i]，	表中元素的顺序不重要

- 寻找等价类，首先找到该等价类中第一个没有输出的元素，这个元素是该等价类的**种子**，该种子作为等价类的第一个成员输出。从这个种子开始，找出该等价类的所有其他成员。种子被加到一个表unprocessedList中。从表unprocessedList中删除一个元素i，然后处理表list[i].list[i]中所有元素和种子同属一个等价类；将list[i]中还没有作为等价类成员的元素输出，然后加入unprocessedList中。

  这是一个过程：从表unprocessedList中删除一个元素i，然后把表list[i]中还没有输出的元素输出，并且加入unprocessedList中。这个过程持续进行到unprocessedList为空。这时就得到了一个等价类，然后继续寻找下一个等价类的种子

  



3.C++实现

list使用数组来实现，unprocessedList这个线性表选择性能更佳的arrayStack



离线等价类程序

```c++
int main()
{
    int n,						//元素个数
    	r;						//关系个数
    
    cout<<"Enter number of elements"<<endl;
    cin>>n;
    if(n<2)
    {
        cout<<"Too few elements"<<endl;
        return 1;				  //因错误而终止
    }
    
    cout<<"Enter few relations"<<endl;
    cin>>r;
    if(r<1)
    {
        cout<<"Too few realtions"<<endl;
        return 1;				//因错误而终止
    }
    
    //建立空栈组成的数组，stack[0]不用
    arrayStack<int>* list=new arrayStack<int> [n+1];
    
    //输入r个关系，存储在表中
    int a,b;			//(a,b)是一个关系
    for(int i=1;i<=r;i++)
    {
        cout<<"Enter next relation/pair"<<endl;
        cin>>a>>b;
        list[a].push(b);
        list[b].push(a);
    }
    
    //初始化以输出等价类
    arrayStack<int> unprocessedList;
    bool* out=new bool[n+1];
    for(int i=1;i<=n;i++)
        out[i]=false;
    
    //输出等价类
    for(int i=1;i<=n;i++)
        if(!out[i])
        {
            //启动一个新类
            cout<<"Next class is :"<<i<<" ";
            out[i]=true;
            unprocessedList.push(i);
            //从unprocessedList 中取类的剩余元素
            while(!unprocessedList.empty())
            {
                int j=unprocessedList.top();
                unprocessedList.pop();
                
                //从list[j] 中的元素属于同一类
                while(!list[j].empty())
                {
                    int q=list[j].top();
                    list[j].pop();
                    if(!out[q])			//未输出
                    {
                        cout<<q<<" ";
                        out[q]=true;
                        unprocessedList.push(q);
                    }
                }
            }
            cout<<endl;
        }
    
    cout<<"End of list of equivalence classes "<<endl;
    
    return 0;
}
```



### 8.5.6	迷宫老鼠

1.问题描述

**迷宫**是一个矩形区域，有一个入口和一个出口。

迷宫内部包不能穿越的墙壁或障碍物，这些障碍物沿着行和列放置，与迷宫的边界平行。

迷宫的入口在左上角，出口在右下角



假定用n x m的矩阵来描述迷宫，矩阵的位置(1,1)表示入口，(n,m)表示出口，n和m分裂代表迷宫的行数和列数。迷宫的每个位置都可用其行号和列号表示。

在位置(i,j)出有一个障碍时，其值为1，否则为0



迷宫老鼠(rat in a maze)问题是要寻找一条从入口到出口的路径



**路径**是一个由位置组成的序列，每一个位置都没有障碍，而且除入口之外，路径上的每个位置都是前一个位置在东，南，西或北方向上相邻的一个位置



假设迷宫是一个方阵(即m=n)且足够小，能够整个存储在目标计算机的内存中。程序应是独立的，一个用户可以输入自己选择的迷宫来直接寻找迷宫路径

2.设计

采用自顶向下的模块化方法设计这个程序。

程序分为三个部分：输入迷宫，寻找路径和输出路径。

每个部分都用一个程序模块来实现。还有一部分用于显示欢迎信息，软件名称及作者信息，这部分用第四个模块来实现，这个模块主要是为了增强程序界面的友好性，他与我们要编写的程序没有任何直接关系。

寻找路径的模块不直接和用户打交道，因此没有提供帮助机制，也不是由菜单驱动的。

其他三个模块都与用户进行交互，因此多花一些精力来设计用户接口。

**友好的用户接口能让用户喜欢你的程序。**

首先设计欢迎模块，我们希望显示如下信息：

Welcom To

RAT IN MAZE

C Joe Bloe ,2000





输入模块的设计需要确定，是要求用户输入一个由0和1组成的矩阵好呢？还是显示一个矩阵，然后让用户使用鼠标点击来选定由障碍物的位置好呢？还要决定使用什么颜色，是否使用声响，等等



输入模块的设计要验证迷宫的入口或出口没有障碍物。若有障碍物，则不存在路径。而用户很有可能输入错误的数据。后面的讨论假定输入模块已经通过了这种验证，即入口和出口没有障碍物



如果要设计一个用户友好的接口，那么最初看上去很简单的任务(读入一个矩阵)实际上是很复杂的。



设计输出模块时需要考虑的问题基本上与设计输入模块时的一样。

3.程序开发计划

设计阶段已经指出需要4个程序模块。而实际上还需要一个模块(即主模块)来调用这4个模块，调用的次序时：欢迎模块，输入模块，寻找路径模块和输出模块。

![image-20210725133154064](D:\Typora‘s text for c++ learn\photos\image-20210725133154064-16271911176181.png)

程序的模块结构如上图所示，每个模块都可以独立编写。根模块被编写成一个main方法，欢迎模块，输入模块，寻找路径模块，和输出模块分别是一个私有方法



4.程序开发



数据结构和算法的大量问题仅仅出现在寻找路径模块的开发过程中。



迷宫老鼠程序的形式

```c++
//此处是函数welcome
//此处是函数inputMaze
//此处是函数findPath
//此处是函数outputPath
void main()
{
    welcome();
    inputMaze();
    if(findPath())
        outputPath();
    else
        cout<<"No path"<<endl;
}
```



寻找路径函数的第一个版本

```c++
bool findPath()
{
	寻找迷宫中到达出口的一条路径
	if(路径找到)
	return true;
	else return false;
}
```

在编写更详细的代码之前，首先要明白如何寻找迷宫路径.

首先把迷宫的入口作为当前位置。如果当前位置是迷宫出口，那么已经找到了一条路径，寻找工作结束。如果当前位置不是迷宫出口，则在当前位置上放置障碍物，以阻止寻找过程又绕回到这个位置。然后检查相邻位置是否又空闲(即没有障碍物)，如果有，就移动到一个空闲的相邻位置上，然后从这个位置开始寻找通往出口的路径，如果不成功，就选择另一个空闲的相邻位置，并且从它开始寻找通往出口的路径。

为了方便移动，在进入新的相邻位置之前，把当前位置保存在一个栈中。如果所有空闲的相邻位置都已经被探索过，但还未能找到路径，则表明迷宫不存在从入口到出口的路径。



首先考虑迷宫，迷宫一般被描述成一个int类型的二维数组maze。

迷宫矩阵的位置(i,j)对应数组maze的位置[i]\[j]

给迷宫围上一圈障碍物，程序就不需要再处理边界条件，大大简化代码设计，代价就是迷宫数组的空间稍稍增加了。

可以定义一个带有数据成员row和col的类position，使用它的对象来跟踪记录迷宫位置。

用数组表示栈，栈用来保存从入口到当前位置的路径。一个没有障碍物的m x m的迷宫，最长的路径可包含m^2个位置

一条路径所包含的位置最多不超过m^2

因为路径的最后一个位置不必存储到栈中，所有栈中存储的位置最多是m^2-1

我们不能保证寻找路径的程序模块能够找到最短的路径

细化后的寻找路径函数

```c++
bool findPath()
{
    //寻找一条从入口(1,1)到出口(m,m)的路径
    初始化迷宫四周的围墙
        //初始化变量，以记录我们在迷宫的当前位置
        here.row=1;
    here.col=1;
    
    maze[1][1]=1;	//防止返回入口
    
    //寻找通向出口的路径
    while(不是出口)do
    {
        寻找可以移动的下一步;
        if(下一步存在)
        {
            把下一步的位置压进路径栈;
            //走到下一步，然后在这一步加上障碍物
            here=neighbor;
            maze[here.row][here.col]=1;
        }
        else
        {
            //不能继续往下走，退回
            if(路径为空)return false;
            返回的位置在路径栈的顶部;
        }
    }
    return true;
}
```





寻找迷宫路径的代码

```c++
bool findPath()
{
    //寻找一条从入口(1,1)到达出口(size,size)的路径
    // 如果找到，返回true，否则返回false
    
    path=new arrayStack<position>;
    
    //初始化偏移量
    position offset[4];
    offset[0].row=0;offset[0].col=1;	//右
    offset[1].row=1;offset[1].col=0;	//下
    offset[2].row=0;offset[2].col=-1;	//左
    offset[3].row=-1;offset[3].col=0;	//上
    
    //初始化迷宫外围的障碍墙
    for(int i=0;i<=size+1;i++)
    {
        maze[0][i]=maze[size+1][i]=1;	//底部和顶部
        maze[i][0]=maze[i][size+1]=1;	//左和右
    }
    
    position here;
    here.row=1;
    here.col=1;
    maze[1][1]=1;	//防止回到入口
    int option=0;	//下一步
    int lastOption=3;
    
    //寻找一条路径
    while(here.row!=size || here.col!=size)
    {
        //没有到达出口
        //找到要移动的相邻的一步
        int r,c;
        while(option<=lastOption)
        {
            r=here.row+offset[option].row;
            c=here.col+offset[option].col;
            option++;	//下一个选择
        }
        //相邻的一步是否找到？
        if(option<=lastOption)
        {
            //移到maze[r][c]
            path->push(here);
            here.row=r;
            here.col=c;
            maze[r][c]=1;	//设置障碍墙，以防重复访问
            option=0;
        }
        else
        {
            //没有附近的一步可走，返回
            if(path->empty())
                return false;	//没有位置可返回
            position next=path->top();
            path->pop();
            if(next.row==here.row)
                option=2+next.col-here.col;
            else option=3+next.row-here.row;
            here=next;
        }
    }
    
    return true;	//到达出口
}
```

## 第九章	队列

队列和栈一样，是一种特殊的线性表。

队列的插入和删除操作分别在线性表的两端进行

队列是一个先进先出(FIFO)的线性表。

有一种队列是优先级队列，它的删除操作是按照元素的优先级顺序进行的。

C++标准模板库STL的队列是一种用数组描述的队列是一种用数组描述的数据结构，它是从STL的双端队列派生的。

为了执行时的效率，把队列设计成一个基类，而且分别采用数组描述和链表描述

四个应用示范程序：1.火车车厢重排问题	2。寻找两个定点之间最短线路的经典Lee算法	3.机器视觉领域的二值图像的像素识别 	4.工厂仿真程序



### 9.1	定义和应用

定义9-1	队列(queue)是一个线性表，其插入和删除操作分别在表的不同端进行。

插入元素的那一端称为**队尾(back / rear)**,删除元素的那一端称为**队首(front)**



队列是一个先进先出(FIFO)的线性表,栈是一个后进先出(LIFO)的线性表

队列的一个例子：在一个分布式系统中，计算机文件在文件服务器上复制，以提供更好的服务。一个文件的所有请求都手心进入代理人队列；代理人把每一个请求分发到负载最小的服务器，等待服务器处理。

### 9.2	抽象数据类型

​	

```c++
抽象数据类型 queue
{
    实例
        元素的有序表，一端称为队首，另一端称为队尾
    操作
        empty();	//返回true，当且仅当队列为空
    	size();		//返回队列中元素个数
    front();		//返回队列头元素
    back();			//返回队列尾元素
    pop();			//删除队列首元素
    push(x);		//把元素x加入队列
}
```



 抽象类队列

```c++
template<class T>
class queue
{
    public:
    virtual ~queue() {}
    virtual bool empty() const=0;
    //返回true，当且仅当队列为空
    virtual int size() const=0;
    //返回队列中元素个数
    virtual T& front()=0;
    //返回头元素的引用
    virtual T& back()=0;
    //返回尾元素的引用
    virtual void pop()=0;
    //删除首元素
    virtual void push(const T& theElement)=0;
    //把元素theElement加入队尾
}
```

### 9.3	数组描述

#### 9.3.1	描述

采用公式把队列的元素映射到一个数组queue中

location(i)=i

队列第i个元素存储在queue[i]中，i>=0.

arrayLength是队列queue的长度，queueFront和queueBack分别表示队首和队尾元素的位置

利用公式，queueFront=0，队列长度=queueBack+1，在队列为空时，queueBack=-1.



要在队列中插入一个元素时，先将queueBack增1，然后把新元素插到queue[queueBack]中。

要删除一个元素，必须把位置1至位置queueBack上的元素左移一个位置

改用location(i)=location(队首元素)+i

会使得删除一个队列元素的性能提高

这时queueFront=location(队首元素),queueBack=location(队尾元素),队列为空时，queueBack<queueFront



把数组组成一个环，会有更好的性能

环形数组表示队列的公式:location(i)=(location(队列首元素)+i)%arrayLength

当且仅当queueFront=queueBack时，队列为空

初始条件queueFront=queueBack=0定义了一个初始为空的队列。

直到数组queue的元素个数等于数组长度(即队列满)为止，queueFront=queueBack,与队列为空的条件完全一样，这样我们无法判定出队列是空还是满。

为了避免出现这种问题，队列不能插满。在向队列插入一个元素之前，先要判断本次操作是否会使队列变满。如果是，就先把数组长度加倍，然后再执行插入操作。

使用这种策略，队列元素个数最多是arrayLength-1

类arrayQueue

类arrayQueue用公式把队列映射到一个一维数组

数据成员queueFront,queueBack,queue；、

除插入和删除操作外，所有方法都与类arrayStack的对应方法相似



在队列中插入一个元素

```c++
template<class T>
void arrayQeueu<T>::push(const T& theElement)
{
    //把元素theElement加入队列
    
    //如果需要，则增加数组长度
    if((theBack+1)% arrayLength==theFront)
    {
        //加倍数组长度
        //此处是数组长度加倍的代码
    }
    //把元素theElement插入队列的尾部
    queueBack=(queueBack+1)% arrayLength;
    queue[queueBack]=theElement;
}
```

数组加长后，得到队列的新布局

- 创建一个新的数组newQueue，其容量翻倍

- 把第2段的元素(从queue[queueFront+1]到queue[arrayLength-1])复制到newQueue从0开始的位置

- 把第1段元素(从queue[0]到queue[queueBack])复制到newQueue从arrayLength-queueFront-1开始的位置



数组队列长度加倍

其中元素复制是用STL的复制函数完成的

```c++
//分配新的数组空间
T* newQueue=new T[2*arrayLength];

//把原数组元素复制到新数组
int start=(theFront+1)% arrayLength;
if(start<2)
//没有形成环
    copy(queue+start,queue+start+arrayLength-1,newQueue);
else 
{
    //队列形成环
    copy(queue+start,queue+arrayLength,newQueue);
    copy(queue,queue+theBack+1,newQueue+arrayLength-start)
}

// 设置新队列的首和尾元素位置
theFront=2* arrayLength-1;
theBack=arrayLength-2;	//队列长度arrayLength-1
arrayLength *=2;
delete [] queue;
queue=newQueue;
```





从队列中删除一个元素

```c++
void pop()
{
    //删除队列首元素
    if(theFront==theBack)
        throw queueEmpty();
    theFront=(theFront+1)%arrayLength;
    queue(theFront).~T();	//给T析构
    
}
```

### 9.4	链表描述

从头到尾的链接方式更适合删除操作。

可以取初值 queueFront=queueBack=NULL,而且当且仅当队列为空时，queueFront=NULL,类linkedQueue可以定义为类extendedChain的一个派生类。

链队列中的插入和删除方法

```c++
template<class T>
void linkedQueue<T>::push(const T& theElement)
{
    //把元素theElement 插到队尾
    
    //申请新元素节点
    chainNode<T>* newNode=new chainNode<T>(theElement,NULL);
    
    //把新节点插到队尾
    if(queueSize==0)
        queueFront=newNode;	//队列空
    else
        queueBack->next=newNode;	//队列不空
    queueBack=newNode;
    
    queueSize++;
}

template<class T>
void linkedQueue<T>::pop()
{
    //删除首元素
    if(queueFront==NULL)
        throw queueEmpty();
    
    chainNode<T>* nextNode=queueFront->next;
    delete queueFront;
    queueFront=nextNode;
    queueSize--;
}
```



### 9.5	应用

#### 9.5.1	列车车厢重排

1.问题描述和求解策略

这一次位于入轨道和出轨道之间的缓冲轨道按照FIFO方式运作，因此可将它们视为队列，禁止将车厢从缓冲轨道移到入轨道，或从出轨道移到入轨道。

第k条轨道Hk可直接将车厢从入轨道移到出轨道。其余k-1条轨道用来缓存不能直接进入出轨道的车厢

2.第一种实现方法

实现车厢重排算法的程序可以用队列表示k-1个缓冲轨道。



输出车厢

```c++
void outputFromHoldingTrack()
{
    //将编号最小的车厢从缓冲轨道移到出轨道
    //从栈itsTrack中删除编号最小的车厢
    track[itsTrack].pop();
    cout<<"Move car"<<smallestCar<<"from holding track"
        <<itsTrack<<"to output track"<<endl;
    
    //检查所有栈的栈顶，寻找标号最小的车厢和它所属的栈
    smallestCar=numberOfCars+2;
    for(int i=1;i<=numberOfTracks;i++)
        if(!track[i].empty() && track[i].front()<smalllestCar)
        {
            smallestCar=track[i].front();
            itsTrack=i;
        }
}
```



把车厢调入一个缓冲轨道

```c++
bool putInHoldingTrack(int c)
{
    //将车厢c移到一个缓冲轨道。返回false，当且仅当没有可用的缓冲轨道
    //为车厢c寻找最适合的缓冲轨道
    //初始化
    int bestTrack=0;	//目前最合适的缓冲轨道
    bestLast=0;		//取bestTrack中最后的车厢
    
    //扫描缓冲轨道
    for(int i=1;i<=numberOfTracks;i++)
        if(!track[i].empty())
        {
            //缓冲轨道i不空
            int lastCar=track[i].back();
            if(c>lastCar && lastCar>bestTrack)
            {
                //缓冲轨道i的尾部具有编号更大的车厢
                bestLast=lastCar;
                bestTrack=i;
            }
        }
    else	//缓冲轨道i为空
        if(bsetTrack==0)
            bestTrack=i;
    
    if(bestTrack==0)
        return false;	//没有可用的缓冲轨道
    
    //把车厢c移到轨道bestTrack
    track[bestTrack].push(c);
    cout<<"Move car"<<c<<"from input track"
        <<"to holding track"<<bestTrack<<endl;
    
    //如果需要，更新 smallestCar和itsTrack
    if(c<smallestCar)
    {
        smallestCar=c;
        itsTrack=bestTrack;
    }
    return true;
}
```





#### 9.5.2	电路布线



1.问题描述

在网格中寻找最短路径的算法，应用在电路布线问题中，

常用的方法是在布线区域设置网格，该网格把布线区域划分成n x m个方格，就像迷宫一样。

一条线路从一个方格a的中心点连接到另一个方格b的中心点，转弯处可以采用直角。

已有线路经过的方格被 封锁，成为下一条线路的障碍。

我们希望用a和b之间的最短路径来布线，以减少信号的延迟

2.求解策略

a和b之间的最短路径需要在两个过程中确定。

一是距离标记过程，另一个是路径标记过程。

具体方法在Page 217

最短路径不一定为1

3.C++实现

一个m x m的网格被描述成一个二维数组grid

其中用0表示可到达的方格，1表示被封锁的方格。

整个网格的四周是由1构成的一圈”障碍物“

数组offsets帮助我们从一个位置移动到相邻位置

用一个队列来记录本身已经编号而其相邻位置尚未编号的方格



我们用grid数组来存储距离，

因为会有冲突，所以把所有距离编号都增加2

grid[i]\[j]=1表示一个没被封锁的位置

grid[i]\[j]>1表示一个位置，它距起始位置的距离为grid[i]\[j]-2

对一个没封锁且没有到达位置grid[i]\[j]=0



寻找电路布线的路径



Grid,size,pathLength,q,start,finish,path是全局变量

代码中假设起始位置start和终点位置finish都没有障碍

代码首先检查始点和重点是否相同，若相同，则距离=0，程序终止

若不同，在网格四周设置“障碍墙”，初始化offset数组，并在起始位置上做标记2，借助队列q，并从位置start开始，首先移动距起始位置为1的可到达的位置，然后移动距起始位置为2的可到达位置，这样不断进行下去，直到到达终点或无法继续移动的时候

后一种情况表示路径不存在，在前者情况下，终点的标记就是路径距离

如果到达终点，则使用距离标记重构路径，路径上的位置(start除外)都存储在数组path中



```c++
bool findPath()
{
    //寻找从始点到终点的最短路径
    //找到时，返回true，否则返回false
    
    if((start.row==finish.row) && (start.col==finish.col))
    {
        //始点==终点
        pathLength=0;
        return true;
    }
    
    //初始化偏移量
    position offset[4];
    offset[0].row=0;offset[0].col=1;	//右
    offset[1].row=1;offset[1].col=0;	//下
    offset[2].row=0;offset[2].col=-1;	//左
    offset[3].row=-1;offset[3].col=0;	//上
    
    //初始化网格四周的障碍物
    for(int i=0;i<=size+1;i++)
    {
        grid[0][i]=grid[size+1][i]=1;	//底部和顶部
        grid[i][0]=grid[i][size+1]=1;	//左边和右边
    }
    
    position here =start;
    grid[start.row][start.col]=2;	//标记
    int numOfNbrs=4;	//一个方格的相邻位置数
    
    //对可达到的位置做标记
    arrayQueue<position> q;
    position nbr;
    do
    {
        //给相邻位置做标记
        for(int i=0;i<=numOfNbrs;i++)
        {
            //检查相邻位置
            nbr.row=here.row+offset[i].row;
            nbr.col=here.col+offset[i].col;
            if(grid[nbr.row][nbr.col]==0)
            {
                //对不可标记的nbr做标记
                grid[nbr.row][nbr.col]=grid[here.row][here.col]+1;
                if((nbr.row==finish.row)
                  && nbr.col==finish.col)) break;	//标记完成
                //把后者插入队列
                q.push(nbr);
            }
        }
        
        //是否到达终点？
        if((nbr.row==finish.row) && 
          (nbr.col==finish.col)) break;	//到达
        
        //终点不可到达，可以移到nbr吗？
        if(q.empty())
            return false;	//路径不存在
        here=q.front();	//取下一个位置
        q.pop();
    }while(true);
    
    //构造路径
    pathLength=grid[finish.row][finish.col]-2;
    path=new position [pathLength];
    
    //从终点回溯
    here=finish;
    for(int j=pathLength-1;j>=0;j--)
    {
        path[j]=here;
        //寻找祖先位置
        for(int i=0;i<numOfNbrs;i++)
        {
            nbr.row=here.row+offset[i].row;
            nbr.col=here.col+offset[i].col;
            if(grid[nbr.row][nbr.col]==j+2)	break;
        }
        here=nbr;	//移向祖先
    }
    return true;
}
```

#### 9.5.3	图元识别

1.问题描述

数字化图像时一个m x m的像素矩阵。在单色图像中，每一个像素要么为0，要么为1.

值为0的像素表示图像的背景，值为1的像素表示图元上的一个点，称其为图元像素。两个像素是相邻的，是指它们左右相邻或上下相邻。两个相邻的图元像素是同一个图元的像素。图元识别的目的就是给图元像素做标记，使得两个像素标记相同，当且仅当它们是同一个图元的像素。

2.求解策略

通过扫描像素来识别图元。扫描的方式是逐行扫描，每一行逐列扫描，当扫描到一个未标记的图元像素时，给它一个图元标记。然后把这个图元像素作为一个新图元的种子，通过识别和标记所有与该种子相邻的图元像素，来寻找新图元剩余的像素。与种子相邻的图元像素称为1-间距像素。然后，识别和标记1-间距像素相邻的所有未标记的图元像素，这些像素被称为2-间距像素。接下来识别和标记2-间距像素相邻的未标记的图元像素

这个过程已知持续到没有新的，未标记的，相邻的图元像素为止

3.C++实现

标记图元像素的程序用到电路布线中的很多技术：用0值像素在图像四周建立“围墙”；

用数组offset来寻找与给定像素相邻的像素

图元识别问题中的标记过程与电路布线中用距离(与起始方格的距离)标记方格的过程很相似

 首先再图像周围包上一圈背景像素(0值像素)，并对数组offset初始化

接下来的两个for循环通过图像来寻找下一个图元的种子

种子时一个未标记的图元像素。

对这样的像素，有pixel[r]\[c]从1变成另一个图元编号(id),赋给种子

然后借助一个队列(也可以借助一个栈)，可以识别出该图元中的其余像素

当方法labelComponents结束时，所有的图元像素都已经获得一个编号

**图元识别**

```c++
void labelComponents()
{
    //给图元编号
    
    //初始化数组offset
    position offset[4];
    offset[0].row=0;offset[0].col=1;	//右
    offset[1].row=1;offset[1].col=0;	//下
    offset[2].row=0;offset[2].col=-1;	//左
    offset[3].row=-1;offset[3].col=0;	//上
    
    //初始化0值像素围墙
    for(int i=0;i<=size+1;i++)
    {
        pixel[0][i]=pixel[size+1][i];	//底部和顶部
        pixel[i][0]=pixel[i][size+1];	//左和右
    }
    int numOfNbrs=4;	//一个像素的相邻位置数
    
    //扫描所有像素，标记图元
    arrayQueue<position> q;
    position here,nbr;
    int id=1;	//图元id
    for(int r=1;r<=sizee;i++)	//图元的行r
        for(int c=1;c<=size;c++)	//图元的列c
            if(pixel[r][c]==1)
            {
                //新图元
                pixel[r][c]=++id;	//取下一个id
                here.row=r;
                here.col=c;
                
                while(true)
                {
                    //寻找其余的图元
                    for(int i=0;i<numOfNbrs;i++)
                    {
                        //检查所有相邻位置
                        nbr.row=here.row+offset[i].row;
                        nbr.col=here.col+offset[i].col;
                        if(pixel[nbr.row][nbr.col]==1)
                        {
                            //像素时当前图元的一部分
                            pixel[nbr.row][nbr.col]=id;
                            q.push(nbr);
                        }
                    }
                    
                    //图元中任意为考察的像素
                    if(q.empty())	break;
                    here=q.front();	//一个图元像素
                    q.pop();
                }
            }		//结束if
}
```

#### 9.5.4	工厂仿真

1.问题描述

一个工厂有m台机器。工厂的每项任务都需要若干道工序才能完成。每台机器都执行一道工序，不同的机器执行不同的工序。一台机器一旦开始执行一道工序就不会中断，直到该工序完成为止



每道工序都需要工序时间(即完成该道工序所需要的时间)以及执行该工序的机器

一项任务中的若干道工序必须按照一定顺序来完成

一项任务首先被调度到执行其第一道工序的机器上；

当第一道工序完成后，该任务被转到执行第二道工序的机器上；依次进行下去，直到该任务的最后一道工序完成为止。

当一项任务到达一台机器时，可能机器正忙，因此要等待

事实上，很可能有若干项任务同时再一台机器旁等待



每台机器都可以有如下三种状态：活动状态，空闲状态和转换状态

在活动状态，机器正在执行一道工序；

在空闲状态，机器无事可做；

在转换状态，机器刚刚完成一道工序，并准备执行移到新工序

在转换状态，机器操作员可能需要清理机器，把刚刚使用的工具放好，然后稍作休息。

每台机器在转换状态所花费的时间多少由机器而定



当一台机器可以执行一项新任务时，它需要从等待执行的任务中选择一项任务。

我们假设每台机器都按照FIFO的方式来选择，因此在每台机器旁等待执行的任务构成一个队列。



一项任务的最后一道工序的完成时间称为该任务的**完成时间(finish time)**.

一项任务的**长度**=所有工序时间之和

如果一项长度为l的任务在0时刻到达工厂，在f时刻完成，那么它在机器队列中的等待时间恰好为f-l。

为了让顾客满意，我们要尽量减少任务的等待时间

如果直到任务的等待时间是多少，而且直到在哪些机器旁等待的时间最多，我们就可以改进和提高工厂的效率



2.如何仿真

在对工厂进行仿真时，我们只是记录任务在机器间的流动，并不实际地执行任何一道工序。我们用一个模拟时钟来计时，每当一道工序完成或一项新任务到达工厂时，模拟时钟就向前移动。机器上的工序完成了，新的工序又产生了。每当一道工序完或一项新任务到达工厂时，我们就称一个**事件(event)**发生了

仿真过程由一个**启动事件(start event)**来启动，当两个或两个以上的事件同时发生时，

它们的顺序任意规定。

**仿真原理**

```c++
//初始化
输入数据
    为每台机器建立任务队列
    在每一台机器的任务队列中，调度第一个任务
    
    //仿真
    while(还有未完成任务)
    {
        确定下一个事件
            if(下一个事件是一台机器的转换工序即转换状态的结束)
                从该机器的任务队列中调度下一个任务(如果还有任务的话)
                else
                {
                    //一道任务工序完成
                    将完成任务工序的机器人放入其转换工序
                        把任务移到执行其下一道工序的机器(如果还有工序的话);
                }
    }
```



3.工厂仿真的好处

- 通过仿真，可以发现工厂中的瓶颈。如果瓶颈时油漆工段，就可以补强油漆工段。

- 使用工厂仿真器，我们可以回答这样的问题：如果用一台投资更多但效果更好的机器替代一台现有的机器，平均等待事件是否可以减少？因此，仿真可以在工厂的拓展/现代化的过程中帮助决策

- 当客户想估计任务完成的准确时间时，可以通过工厂仿真器得到。

4.高级仿真器设计

在设计仿真器时，假定所有的任务都在初始时到达(即在仿真过程中不会出现新任务)，而且仿真过程一直持续到所有任务都完成时为止

仿真器由类machineShopSimulator来实现。

仿真器是一个相当复杂的程序，因此把它分解成若干个模块。

仿真器执行的任务是：输入数据，然后把任务按其第一道工序放入相应队列；

执行启动事件(即装入初始任务);

处理所有事件(即开始实际仿真)；

输出机器等待时间；

一个任务对应一个C++函数，用全局变量largeTime表示一个时间，所有工序都必须在这个时间以前完成



9-10---工厂仿真的主要程序

```c++
void main()
{
    inputData();	//获取机器和任务的数据
    startShop();	//装入初始任务
    simulate();	//执行所有任务
    outputStartstics();	//输出在每台机器上的等待时间
}
```



5.结构task

在设计程序9-10中所调用的4个函数之前，必须设计数据对象的描述方法，这些数据对象包括工序，任务，机器，事件表

前三个数据对象用结构，第四个对象用类



每个工序都由两部分构成：machine(执行该工序的机器) ,time(完成该工序所需要的时间)

9-11 结构task

```c++
struct task
{
    int machine;
    int time;
    
    task(int theMachine=0;int theTime=0)
    {
        machine=theMachine;
        time=theTime;
    }
};
```

6.结构job

每项任务都有一个工序表，每道工序按表中的顺序执行

可以把工序表描述成一个队列taskQ

为了计算一项任务的总等待时间，需要知道该任务的长度和完成时间。

完成时间通过计时确定，任务长度为各工序时间之和

为了计算任务长度，我们定义一个数据成员length

9-12	结构job

```c++
struct job
{
    arrayQueue<task> taskQ;	//任务的工序
    int length;	//被调度的工序时间之和
    int arrivalTime;	//到达当前队列的时间
    int id;	//任务标志符
    
    job(int theId=0)
    {
        id=theId;
        length=0;
        arrivalTime=0;
    }
    
    void addTask(int theMachine,int theTime)
    {
        task theTask(theMachine,theTime);
        taskQ.push(theTask);
    }
    
    int removeNextTask()
    {
        //删除任务的下一道工序，返回它的时间
        //更新长度
        
        int theTime=taskQ.front().time;
        taskQ.pop();
        length+=theTime;
        return theTime;
    }
}
```

数据成员arrivalTime用于记录一项任务进入当前机器队列的时间，然后确定该任务在这个队列中的等待时间。任务标志存储在id中，仅在输出该任务的总的等待时间时才会使用这个标志符

方法addTask把一道工序加入任务的工序队列中。

该工序在机器theMachine上执行，需要时间为theTime

该方法仅在数据输入时使用

当一个任务从一个机器队列中删除然后进入活动状态时，使用方法removeNextTask

该任务的第一道工序从工序队列(工序队列用于保存尚未被执行的工序)中删除，并把该工序时间加到任务长度中，然后返回该工序时间

当调度任务的最后一道工序时，数据成员length的值等于该任务的长度

7.结构machine

9-13	结构machine

```c++
struct machine
{
    arrayQueue<job*> jobQ;	//本机器的等待处理的任务队列
    
    int changeTime;//本机器的转换时间
    int totalWait;	//本机器的总体延时
    int numTasks;	//本机器处理的工序数量
    job* activeJob;	//本机器当前处理的任务
    
    machine()
    {
        totalWait=0;
        numTasks=0;
        activeJob=NULL;
    }
    
};
```



8. 类EventList

   ​    所有机器的完成时间都存储在一个事件表中。

   为了从一个时间转向下一个事件，我们需要在机器的完成时间中确定最小者。

   仿真器还需要一个操作，来设置一台特定机器的完成时间

   每当一个新任务被调度到一台机器上运行时就要执行操作。

   当一台机器空闲时，其完成事件被设置成一个很大的数largeTime

   9-14	类eventList

用一维数组finishTime实现事件表，finishTime[p]表示机器p的完成时间

```c++
class eventList
{
    public:
    eventList(int theNumMachines,int theLargeTime)
    {
        //为m台机器，初始化其完成时间
        if(theNumMachines<1)
            throw illegalParameterValue
            ("number of machines must be>=1");
        numMachines=theNumMachines;
        finishTime=new int [numMachines+1];
        
        //所有机器都空闲，用大的完成时间初始化
        for(int i=1;i<=numMachines;i++)
            finishTime[i]=theLargeTime;
    }
    
    int nextEventMachine()
    {
        //寻找完成时间最早的机器
        int p=1;
        int t=finishTime[1];
        for(int i=2;i<=numMachines;i++)
            if(finishTime[i]<t)
            {
                //机器i完成时间更早
                p=i;
                t=finishTime[i];
            }
        return p;
    }
    
    int nextEventTime(int theMachine)
    {return finishTime[theMachine];}
    
    void setFinishTime(int theMachine,int theTime)
    {finishTime[theMachine]=theTime;}
    private:
    int* finishTime;	//完成时间数组
    int numMachines;	//机器数量
}；
```



方法nextEventMachine的返回值是第一个完成其活动工序的机器

机器p完成其活动工序的时间通过调用方法nextEventTime(p)来确定



9. 全局变量

   程序9-15 仿真器中所使用的全局变量

   ```c++
   //全局变量
   int timeNow;	//当前时间
   int numMachines;	//机器数量
   int numJobs;	//任务数量
   eventList* eList;	//事件表的指针
   machine* mArray;	//机器数组
   int largeTime=10000;	//在这个时间之前所有机器都已完成工序
   ```

   

   

10.函数inputData

程序9-16	输入工厂数据

首先输入机器数和任务数，然后创建初始事件表List(其中每台机器的完成时间均为largeTime)和机器数组mArray

接下来输入每台机器的转换时间

然后依次输入每项任务

对于每项任务，首先输入该任务的工序数目，然后按(machine,time)的形式，输入每个工序

执行任务的第一道工序的机器被记录在变脸firstMachine中

当一个任务的所有工序都已输入完毕时，该任务被插入执行第一道工序的机器队列中

```c++
 void inputData
 {
     //输入工厂数据
     cout<<"Enter number of machines and jobs"<<endl;
     cin>>numMachines>>numJobs;
     if(numMachines<1 || numJobs<1)
         throw illegalInputData
         ("number of machines and jobs must be >=1");
     
     //生成事件和机器队列
     eList=new eventList(numMachine,largeTime);
     mArray=new machine [numMachines+1];
     
     //输入机器的转换时间
     cout<<"Enter change-over times for machines"<<endl;
     int ct;
     for(int j=1;j<=numMachines;j++)
     {
         cin>>ct;
         if(ct<0)
             throw illegalInputData("change-over time must be >=0");
         mArray[j].changeTime=ct;
     }
     
     //输入任务
     job* theJob;
     int numTasks,firstMachine,theMachine,theTaskTime;
     for(int i=1;i<=numJobs;i++)
     {
         cout<<"Enter number of tasks for job"<<i<<endl;
         cin>>numTasks;
         firstMachine=0;	//第一道工序的机器
         if(numTasks<1)
             throw illegalInputData("each job must have >1 task");
         
         //生成任务
         theJob=new job(i);
         cout<<"Enter the tasks (machine,time)"
             <<"in process order"<<endl;
         for(int j=1;j<=numTasks;j++)
         {
             //读取任务i的工序
             cin>>theMachine>>theTaskTime;
             if(theMachine<1 || theMachine>numMachines || theTaskTime<1)
                 throw illegalInputData("bad machine number or task time");
             if(j==1)
                 firstMachine=theMachine;	//处理任务的第一台机器
             theJob->addTask(theMachine,theTaskTime);
         }
         mArray[firstMachine].jobQ.push(theJob);
     }
 }
```





11.函数startShop和changeState

为了启动仿真过程，需要从每台机器的任务队列中取出第一个任务并放到该机器上执行

由于每台机器的初始状态为空闲状态，所以加载初始任务都素hi把机器从空闲状态变成活动状态

方法changeState(i)便用于机器i的这种转换

程序9-17	启动仿真

```c++
void startShop()
{
    //在每台机器上装载其第一个任务
    for(int p=1;p<=numMachine;p++)
        changeState(p);
}
```

程序9-18	修改机器状态

```c++
job* changeState(int theMachine)
{
	//机器theMachine上的工序完成了，调度下一道工序
    //返回值是在机器theMachine上刚刚完成的任务
    job* lastJob;
    if(mArray[theMachine].activeJob==NULL)
    {
        //处于空闲或转换状态
        lastJob=NULL;
        //等待，准备处理新的任务
        if(mArray[theMachine].jobQ.empty())	//没有等待执行的任务
            eList->setFinishTime(theMachine,largeTime);
        else
        {
         //从队列中提取任务，在机器上执行
            mArray[theMachine].activeJob=mArray[theMachine].jobQ.front();
            mArray[theMachine].jobQ.pop();
            mArray[theMachine].totalWait+=timeNow-mArray[theMachine].activeJob->arrivalTime;
            mArray[theMachine].numTasks++;
            int t=mArray[theMachine].activeJob->removeNextTask();
            eList->setFinishTime(theMachine,timeNow+t);
        }
    }
    else
    {
        //在机器theMachine上刚刚完成一道工序
        //设置转换时间
        lastJob=mArray[theMachine].activeJob;
        mArray[theMachine].activeJob=NULL;
        eList->setFinishTime(theMachine,timeNow+mArray[theMachine].changeTime);
    }
    return lastJob;
}
```



12.函数simulate and moveToNextMachine

程序9-19	处理所有任务

函数simulate对所有事件循环调度，直到最后一项任务完成

numJobs是尚未完成的任务数目，while循环咋numJobs=0时结束

要确定下一个事件的时间并将该事件存入变量timeNow

对于产生事件的机器nextToFinish，我们要改变它的状态

```c++
void simulate()
{
    //处理所有未处理的任务
    while(numJobs>0)
    {
        //至少有一个任务未处理
        int nextToFinish=eList->nextEventMachine();
        timeNow=eList->nextEventTime(nextToFinish);
        //改变机器nextToFinish上的任务
        //如果任务theJob调度到下一台机器
        //如果任务theJob完成，则减少任务数
        if(theJob!=NULL && !moveToNextMachine(theJob))
            numJobs--;
    }
}
```



程序9-20	把下一项任务移至下一道工序对应的机器

```c++
bool moveToNextMachine(job* theJob)
{
    //调度任务theJob到执行其下一道工序的机器
    //如果任务已经完成，则返回false
    
    if(theJob->taskQ.empty())
    {
        //没有下一道工序
        cout<<"Job"<<theJob->id<<"has compeleted at"
            <<timeNow<<"Total wait was"
            <<(timeNow-theJob->length)<<endl;
        return false;
    }
    else
    {
        //任务theJob有下一道工序
        //确定执行下一道工序的机器
        int p=theJob->taskQ.front().machine;
        //把任务插入机器p的等待任务队列
        mArray[p].jobQ.push(theJob);
        theJob->arrivalTime=timeNow;
        //如果机器p空闲，则改变它的状态
        if(eList->nextEventTime(p)==largeTime)
            //机器空闲
            changeState(p);
        
        return true;
    }
}
```

13.函数outputStatistics

程序9-21	输出每台机器的等待时间

```c++
void outputStatistics()
{
    //输出在每台机器上的等待时间
    cout<<"Finish time= "<<timeNow<<endl;
    for(int p=1;p<=numMachines;p++)
    {
        cout<<"Machine"<<p<<"completed"
            <<mArray[p].numTasks<<"tasks"<<endl;
        cout<<"The total wait time was"
            <<mArray[p].totalWait<<endl;
        cout<<endl;
    }
}
```

## 第10章	跳表和散列

为了提升有序链表的查找性能，可以在全部或部分节点上增加额外的指针。

在查找时，通过这些指针，可以跳过链表的若干个节点，不必从左到右连续查看所有节点

增加了额外的向前指针的链表叫做跳表(skip list)

它采用随机技术来决定链表的哪些节点应增加向前指针，以及增加多少个指针。

散列时用来查找，插入，删除的另一种随机方法。

如果经常需要按序输出所有元素或按许查找元素，那么跳表的执行效率将优于散列

本章中含有一个散列的应用：文本压缩和解压缩

程序是基于Lempel-Ziv-Welch算法编写的

### 10.1	字典

**字典(dictionary)**是由一些形如(k,v)的数对组成的集合，其中k是关键字，v是与关键字k对应的值

任意两个数对，其关键字都不等

有关字典的操作：

- 确定字典是否为空
- 确定字典有多少数对
- 寻找一个指定了关键字的数对
- 插入一个数对
- 删除一个指定了关键字的数对

**多重字典(dictionary with duplicate)**与上述字典类似，只是两个或更多的数对可以具有相同的关键字

在一个多重字典中进行查找和删除操作时，需要消除歧义。

就查找而言，有两种可能

1）查找任何一个具备给定关键字的数对

2)查找所有具备给定关键字的数对。就删除操作而言，我们要求给用户提供与给定关键字对应的所有数对，由用户选择删除哪些数对。此外，可以任意删除与给定关键字对应的一个数对或所有数对

编译器使用的**符号表(symbol table)**是一个用户定义的多重字典

查找操作可以按照你给定的关键字，**随机**地存取字典中的数对。有些字典的应该还有另一种存取模式--**顺序存取(sequential access)**.
在这种存取模式中，利用迭代器，按关键字的升值顺序逐个查找字典数对



### 10.2	抽象数据类型

ADT 10-1是抽象数据类型Dictionary

其中 p.first和p.second分别表示数对p的关键字和值

当字典没有与关键字p.first对应的数对时，插入操作insert(p)把数对p插入字典

当字典已经具有与关键字p.first对应的数对时，用新的值取代旧的值

这种操作方法与STL容器类hash_map的方法insert一致

ADT 10-1	字典的抽象数据类型

```c++
抽象数据类型 Dictionary
{
    实例
        关键字各不相同的一组数对
    操作
        empty();	//返回true，当且仅当字典为空
    	size();		//返回字典的数对个数
    	find(k);	//返回关键字为k的数对
    	insert(p);	//插入数对p
    	erase(k);	//删除关键字为k的数对
}
```



程序10-1	抽象类Dictionary

查找函数的返回值时指针，指向与给定的关键字相匹配的数对。

这与STL的容器类hash_map的函数find一致

STL中的类hash_multimap就是多重字典的描述方法

```c++
template<class T>
class dictionary
{
    public:
    virtual ~dictionary();
    virtual bool empty() const=0;
    //返回true，当且仅当字典为空
    virtual int size() const=0;
    //返回字典中数对的数目
    virtual pair<const K,E>* find(const K&) const=0;
    //返回匹配数对的指针
    virtual void erase(const K&)=0;
    //删除匹配的数对
    virtual void insert(const pair<const K,E>& )=0;
    //往字典中插入一个数对
}
```

### 10.3	线性表描述

字典可以保存在线性表(p0,p1,xxx)中，其中p.s是字典中按关键字递增次序排列的数对

为了适应这种表示方式，可以定于两个类sortedArrayList和sortedChain

前者用数组描述线性表，而后者用链表描述

程序10-2，10-3和10-4是类sortedChain的方法find,insert and erase,

sortedChain的节点是pairNode的实例

与chainNode的实例一样，pairNode的实例有两个域---element,next

它们的类型分别是pair<const K,E> pairNode<K,E>*

程序10-2	方法sortedChain<K,E>::find

```c++
template<class T>
pair<const K,E>* sertedChain<K,E>::find(const K& theKey) const
{
    //返回匹配的数对的指针
    //如果不存在匹配的数对，则返回NULL
    pairNode<K,E>* currentNode=firstNode;
    
    //搜索关键字为theKey的数对
    while(currentNode!=NULL && currentNode->element.first!=theKey)
        currentNode=currentNode->next;
    
    //判断是否匹配
    if(currentNode!= NULL && currentNode->element.first==theKey)
        //找到匹配数对
        return &currentNode->element;
    
    //无匹配数对
    return NULL;
}
```

程序10-3	方法sortedNode<K,E>::insert

```c++
template<class T>
void sortedChian<K,E>::insert(const pair<const K,E>& thePair)
{
    //往字典中插入thePair,覆盖已经存在的匹配的数对
    pairNode<K,E> *p=firstNode;
    *tp=NULL;	//tp trails p
    
    //移动指针tp，使thePair可以插在tp的后面
    while(p!=NULL && p->element.first<thePair.first)
    {
        tp=p;
        p=p->next;
    }
    
    //检查是否有匹配的数对
    if(p!=NULL && p->element.first==thePair.first)
    {
        //替换旧值
        p->element.second=thePair.second;
        return;
    }
    
    //无匹配的数对，为thePair建立新节点
    pairNode<K,E> *newNode=new pairNode<K,E>(thePair,p);
    
    //在tp之后插入新节点
    if(tp==NULL) firstNode=newNode;
    else tp->next=newNode;
    
    dSize++;
    return;
}
```



程序10-4	方法sortedChain<K,E>::erase

```c++
template<class T>
void sortedChain<K,E>::erase(const K& theKey)
{
    //删除关键字为theKey的数对
    pairNode<K,E> *p=firstNode,
    *tp=NULL;	//tp trails p
    
    //搜索关键字为theKey的数对
    while(p!=NULL && p->element.first<theKey)
    {
        tp=p;
        p=p->next;
    }
    
    //确定是否匹配
    if(p!=NULL & p->element.first==theKey)
    {
        //找到一个匹配的数对
        //从链表中删除p
        if(tp==NULL) firstNode=p->next;	//p是第一个节点
        else tp->next=p->next;
        
        delete p;
        dSize--;
    }
}
```

### 10.4	跳表表示

#### 10.4.1	理想情况

在一个用有序链表描述的n个数对的字典中进行查找，至多需要n次关键字比较

如果在链表的中部节点加一个指针，则比较次数可以减少到n/2+1

这时，为了查找一个数对，首先与中间的数对比较，如果查找的数对关键字比较小，则仅在链表的左半部分继续查找，否则，在链表右半部分继续查找

#### 10.4.2	插入和删除

在插入时，要对新数对分配一个级(即确定它属于哪一级链表)，分配过程由后面将要介绍的随机数生成器来完成。

对删除操作，我们无法控制结构。要删除节点xx，首先要找到xx。

#### 10.4.3	级的分配

设定一个级数的上限maxLevel，最大值为
$$
[log.1/p^N]-1
$$
采用级数的上限maxLevel，还可能出现：在插入一个新数对之前由3个链表，而在插入之后就有了10个链表。

#### 10.4.4	结构skipNode

跳表结构的头节点需有足够的指针域，以满足最大链表级数的构建需要，而尾节点不需要指针域。

每个存有数对的节点都有一个存储数对的element域和个数大于自身级数的指针域

程序10-5	结构skipNode

```c++
template<class K,class E>
struct skipNode
{
    typedef pair<const K,E> pairType;
    
    pairType element;
    skipNode<K,E> **next;	//指针数组
    
    skipNode(const pairType& thePair,int size)
        :element(thePair){next =new skipNode<K,E>* [size];}
};
```

指针域由数组next表示，其中next[i]表示i级链表指针。构造函数把字典数对对存入element域，为指针数组分配空间。对一个lev级链表数对，其size值应为lev+1

#### 10.4.5	类skipList

1.类skipList的数据成员

程序10-6 类skipList的数据成员

```
float cutOff;	//用来确定层数
int levels;		//当前最大的非空链表
int dSize;		//字典的数对个数
int maxLevel;	//允许的最大链表层数
K tailKey;		//最大关键字
skipNode<K,E>* headerNode;		//头节点指针
skipNode<K,E>* tailNode;	//尾结点指针
skipNode<K,E>** last;	//last[i]表示i层的最后节点
```

2.类skipList的构造函数

参数 largeKey的值大于字典的任何数对的关键字，它存储在尾节点。

参数maxPairs的值是字典数对个数的最大数

虽然代码允许字典数对个数大于maxPairs，但如果不超过maxPairs，程序的性能会更好，因为链表级数是由maxPairs替代公式中的N之后决定的、

参数prob的值是i-1级链表数对同时i级链表数对的概率。

构造函数初始化类skipList的数据成员，也为头节点，尾节点 和 数组last分配空间

在插入和删除之前的搜索时所遇到的每级链表的最后一个数对都存储在数组last中。

跳表初始布局为空，头节点maxLevel+1个指向尾节点的指针

程序10-7	构造函数

```c++
template<class T>
skipList<K,E>skipList(K largeKey,int maxPairs,float prob)
{
    //构造函数，关键字小于largeKey,且数对个数size最多为maxPairs.0<prob<1
    cutOff=prob*RAND_MAX;
    maxLevel=(int)ceil(logf(float)maxPairs)/logf(1/prob))-1;
    levels=0;	//初始化级数
    dSize=0;
    tailKey=largeKey;
    
    //生成头节点，尾节点和数组last
    pair<K,E> tailPair;
    tailPair.first=tailKey;
    headerNode=new skipNode<K,E>(tailPair,maxLevel+1);
    tailNode=new skipNode<K,E>(tailPair,0);
    last=new skipNode<K,E> *[maxLevel+1];
    
    //链表为空时，任意级链表中的头节点都指向尾节点
    for(int i=0;i<=maxLevel;i++)
        headerNode->next[i]=tailNode;
}
```

3.方法skipList<K,E>::find

若关键字theKey的数对不存在，则返回值是NULL，否则，返回值是该数对的指针

find从最高级链表(即levels级链表，它只有一个数对)开始查找，直到0级链表。在每一级链表中，从左边尽可能逼近要查找的记录。

虽然在找到关键字等于theKey的数对时，可能在i级就终止搜索，但是用来检验是否相等的额外比较操作是不必要的，因为大部分这样的数对都只出现在0级链表。当从for循环退出时，指针正好处在要查找的数对左边。

这与0级链表的下一个数对比较，即可确定要查找的数对是否在跳表中

程序10-8	skipList<K,E>::find方法

```c++
template<class T>
pair<const K,E>* skipList<K,E>::find(const K& theKey) const
{
    //返回匹配的数对的指针
    //如果没有匹配的数对，返回NULL
    if(theKey>=tailKey)
        return NULL;	//没有可能的匹配的数对
    //为止beforeNode是关键字theKey的节点之前最右边的为止
    skipNode<K,E>* beforeNode=headerNode;
    for(int i=levels;i>=0;i--)	//从上级链表到下级链表
        //跟踪i级链表指针
        while(beforeNode->next[i]->element.first<theKey)
            beforeNode=beforeNode->next[i];
    
    //检查下一个节点的关键字是否是theKey
    if(beforeNode->next[0]->element.first==theKey)
        return &beforeNode->next[0]->element;
    
    return NULL;	//无匹配数对
}
```

4.方法skipList<K,E>::insert

在编写代码前，不仅要编写级的分配函数，为新数对指定它所属的级链表，而且编写搜索函数search，以存储在每一级链表搜索时所遇到的最后一个节点的指针。

程序10-9	级的分配方法

```c++
template<class K,class E>
int skipList<K,E>::level() const
{
    //返回一个表示链表级的随机数，这个数不大越maxLevel
    int lev=0;
    while(rand()<=cutOff)
        lev++;
    return (lev<=maxLevel)? lev:maxLevel;
}
```

程序10-10	搜素并把在每一级链表搜索时所遇到的最后一个节点指针存储起来

```c++
template<class K,class E>
skipNode<K,E>* skipList<K,E>::search(const K& theKey) const
{
    //搜索关键字theKey,把每一级链表中要查看的最后一个节点存储在数组last中
    //返回包含关键字theKey的节点
    //位置beforeNode是关键字theKey的节点之前最右边的位置
    skipNode<K,E>* beforeNode=headerNode;
    for(int i=levels;i>=0;i--)
    {
        while(beforeNode-<next[i]->element.first<theKey)
            beforeNode=beforeNode->next[i];
        last[i]=beforeNode;		//最后一级链表i的节点
        
    }
    return beforeNode->next[0];
}
```

程序10-11	跳表插入

```c++
template<class E,class K>
void skipList<K,E>::insert(const pair<const K,E>& thePair)
{
    //把数对thePair插入字典。覆盖其关键字相同的已存在的数对
    if(thePair.first>=tailKey)	//关键字太大
    {
        ostringstream s;
        s<<"Key= "<<thePair.first<<"Must be <"<<tailKey;
        throw illegalParameterValue(s.str());
    }
    
    //查看关键字为theKey的数对是否已经存在
    skipNode<K,E>* theNode=search(thePair.first);
    if(theNode->element.first==thePair.first)
    {
        //若存在，则更新该数对的值
        theNode->element.second=thePair.second;
        return;
    }
    
    //若不存在，则确定新节点所在的级链表
    int theLevel=level();	//新节点的级
    //使级theLevel为<=levels+1
    if(theLevel>levels)
    {
        theLevel=++levels;
        last[theLevel]=headerNode;
    }
    
    //在节点theNode之后插入新节点
    skipNode<K,E>* nextNode=new skipNode<K,E>(thePair,theLevel+1);
    for(int i=0;i<=theLevel;i++)
    {
        //插入i级链表
        newNode->next[i]=last[i]->next[i];
        last[i]->next[i]=newNode;
    }
    dSize++;
    return;
}
```

5.skipList<K,E>::erase

程序10-12的代码时删除关键字为theKey的数对。while循环用来修改levels的值，除非跳表为空，否则levels不会称为0

程序10-12	删除跳表的记录

```c++
template<class K,class E>
void skipList<K,E>::erase(const K& theKey)
{
    //删除关键字等于theKey的数对
    if(theKey>=tailKey)		//关键字太大
        return ;
    
    //查看是否有匹配的数对
    skipNode<K,E>* theNode=search(theKey);
    if(theNode->element.first!=theKey)	//不存在
        return ;
    //从跳表中删除节点
    for(int i=0;i<=levels && last[i]->next[i]==theNode;i++)
        last[i]->next[i]=theNode->next[i];
    
    //更新链表级
    while(levels>0 && headerNode->next[levels]==tailNode)
        levels--;
    
    delete theNode;
    dSize--;
}
```

6.其他方法

关于size和empty方法,还有迭代器方法，它们的代码与链表chain中相应的方法的代码相似。记住，每一级链表的节点(不包含头节点，因为它没有关键字)都是按照关键字递增顺序排列的。

因此，skipList的迭代器可以提供顺序访问方法。

#### 10.4.6	skipList方法的复杂度

Page 246

 

### 10.5	散列表描述

#### 10.5.1	理想散列

字典的另一种表示方法是**散列(hashing)**

它用一个**散列函数(也称哈希函数)**把字典的数对映射到一个**散列表(也称哈希表)**的具体位置

程序10-13	把一个长度为3的字符串转换为一个长整型数

```c++
long threeToLong(string s)
{
    //假设s.length()>=3
    long answer =s.at(0);
    
    //左移8位，加入下一个字符
    answer=(answer<<8)+s.at(1);
    
    //左移8位，加入下一个字符
    return (answer<<8)+s.at(2);	
}
```

#### 10.5.2	散列函数和散列表

1.桶和起始桶

当关键字的范围太大，不能用理想方法表示时，可以采用并不理想的散列表和散列函数，散列表位置的数量比关键字的个数少，散列函数把若干个不同的关键字映射到散列表的同一个位置。
散列表的每一个位置叫一个**桶(bucket)**，对关键字为k的数对，f(k)是**起始桶(home** **bucket)**;
桶的数量等于散列表的长度或大小。

2.除法散列函数

除法散列函数形式:f(k)=k%D

k是关键字，D是散列表的长度(即桶的数量),%为求模操作符。

散列表的位置索引从0到D-1

3.冲突和溢出

当两个不同的关键字所对应的起始桶相同时，就是**冲突(collision)**发生了

一个桶可以存储多个数对，因此发生冲突也没什么关系。

只要起始桶足够大，所有对应同一个起始桶的数对都可存储在一起。如果存储桶没有空间存储一个新数对，就是溢出(overflow)发生了

溢出处理方法最常见的就是：线性探查法和双重散列法

4.我需要一个好的散列函数

单就冲突而言并不可怕，可怕的是它会带来溢出，除非一个桶可以容纳无限多个数对，否则插入时的溢出就不是那么容易解决的问题了。

**均匀散列函数(uniform hash function)**可以使得映射到散列表中任何一个桶里的关键字数量大致相等，冲突和溢出的平均数量最少

在实际应用中性能表现好的均匀散列函数被称为**良好散列函数(good hash function)**

5.除法和非整型关键字

为使用除法散列函数，在计算f(k)之前，需要把关键字转换为非负整数。因为所有散列函数都把若干个关键字散列到相同的起始桶，所以没必要把关键字转换为统一的非负整数。

只要把串数据，结构和算法转换为相同的整数就可以了

程序10-14	把一个字符串转换为一个不唯一的整数

```c++
int stringToInt(string s)
{
    //把s转换为一个非负整数，这种转换依赖s的所有字符
    int length=(int) s.length();	//s中的字符个数
    int answer=0;
    if(length%2==1)
    {
        //长度为奇数
        answer=s.at(length-1);
        length--;
    }
    
    //长度是偶数
    for(int i=0;i<length;i+=2)
    {
        //同时转换两个字符
        answer+=s.at(i);
        answer+=((int)s.at(i+1))<<0;
    }
    
    return (answer<0)?-answer:answer;
}
```

C++的STL中的模板类hash<T>是实现散列的一个专业版，它把类型为T的实例转换为类型为size_t的非负整数

程序10-15	专业版hash<string>

T是STL串

这个特化版与SGI STL的hash<chat*>是一致的

```c++
template<>
class hash<string>
{
    public:
    size_t operator()(const string theKey) const
    {
        //把关键字theKey转换为一个非负整数
        unsigned long hashValue=0;
        int length=(int) theKey.length();
        for(int i=0;i<length;i++)
            hashValue=5*hashValue+theKey.at(i);
        
        return size_t(hashValue);
    }
}
```

#### 10.5.3	线性探查

1.方法

要把溢出的关键字放进散列表，最简单方法就是找到下一个可用的桶，这种解决溢出的方法叫做**线性探查(Linear Probing)**

在寻找下一个可用通时，散列表被视为环形表

实现删除的另一个策略时为每一个桶增加一个域neverUsed

在散列表初始化时，这个域被置为true，每当一个数对存入一个桶中，neverUsed域被置为false

现在搜索的结束条件2)变为：桶的neverUsed域为true

不过在删除时，只是把表的相应位置置为空，

一个新元素被插入在其对应的起始桶之后所找到的一第一个空桶中。

注意，在这种方案中，neverUsed不会重新置为true。用不了多长时间，所有(或几乎所有)桶的neverUsed域都等于false，这是搜索是否失败，只有检查所有的桶之后才可以确定。为了提高性能，当很多桶的neverUsed域等于false时，必须重新组织这个散列表。

2.线性探查的C++实现

程序10-16	hashTable的数据成员和构造函数

```c++
//散列表的数据成员
pair<const K,E>** table;	//散列表
hash<K> hash;	//把类型K映射到一个非整数
int dSize;	//字典中数对个数
int divisor;	//散列函数除数
//构造函数

template<calss K,class E>
hashTable<K,E>::hashTable(int theDivisor)
{
    divisor=theDivisor;
    dSize=0;
    
    //分配和初始化散列表数组
    table=new pair<const K,E>* [divisor];
    for(int i=0;i<divisor;i++)
        table[i]=NULL;
}
```

程序10-17	给出了hashTable的搜索函数search 

函数的返回值是一个桶的序号b，它满足如下三种情形之一

 table[b]是一个指针，指向关键字为theKey的数对

散列表没有关键字theKey的数对,table[b]=NULL,需要时，可以把关键字为theKey的数对插到b号桶

散列表没有关键字为theKey的数对，但是table[b]不是NULL，table[b]的关键字不是thekey,而且表已满

程序10-17	hashTable<K,E>::search

```c++
template<class K,class E>
int hashTable<K,E>::search(const K& theKey) const
{
    //搜索一个公开地址散列表，查找关键字为theKey的数对
    //如果匹配的数对存在，返回它的位置，否则，如果散列表不满，
    //则返回关键字为theKey的数对可以插入的位置
    
    int i=(int) hash(theKey)%divisor;	//起始桶
    int j=i;
    do
    {
        if(table[j]==NULL	||	table[j]->first==theKey)
            return j;
        j=(j+1)%divisor;	//下一个桶
        j=(j+1)%divisor;	//是否返回到起始桶？
    }while(j!=i);
    
    return j;	//表满
}
```

程序10-18	hashTable<K,E>::find

```c++
template<class K,class E>
pair<const K,E>* hashTable<K,E>::find(const K& theKey)	const
{
    //返回匹配数对的指针
    //如果匹配数对不存在，返回NULL
    //搜索散列表
    int b=search(theKey);
    
    //判断table[b]是否匹配数对
    if(table[b]==NULL || table[b]->first!=theKey)
        return NULL;	//没有找到
    
    return table[b];	//找到匹配数对
}
```

程序10-19是insert函数的实现代码

```c++
template<class K,class E>
void hashTable<K,E>::insert(const pair<const K,E>& thePair)
{
    //把数对thePair插入字典，若存在关键字相同的数对，则覆盖
    //若表满，则抛出异常
    //搜索散列表，查找匹配的数对
    int b=search(thePair.first);
    
    //检查匹配的数对是否存在
    if(table[b]==NULL)
    {
        //没有匹配的数对，而且表不满
        table[b]=new pair<const K,E>(thePair);
        dSize++;
    }
    else
    {
        //检查是否有重复的关键字数对或是否表满
        if(table[b]->second=thePair.second;)
    }
    else	//表满
        throw hashTableFull();
}
```

#### 10.5.4	链式散列

1.方法

如果散列表的每一个桶都可以容纳无限多个记录，那么溢出问题就不存在了。实现这个目标的一个方法是给散列表的每一个位置配置一个线性表。

这时，每一个数对可以存储在它的起始桶线性表中。

为了搜素关键字值为k的记录，首先计算其起始桶，k%D,然后搜索该桶所对应的链表。为插入一个记录，首先要保证散列表没有一个记录与该记录的关键字相同，为此而进行的搜索仅限于该记录的起始桶所对应的链表。

要删除一个关键字为k的记录，我们要搜索它所对应的起始桶链表，找到该纪录，然后删除。

2.链式散列表的C++实现

类hashChains用一组table[0:divisor-1]实现字典，数组元素类型是sortedChain<K,E>

程序10-20	hashChains的一些方法

```c++
template<class K,class E>
pair<const K,E>* find(const K& theKey) const
{return table[hash(theKey)%divisor].find(theKey);}

void insert(const pair<const K,E>& thePair)
{
	int homeBucket=(int) hash(thePair.first)%divisor;
	int homeSize=table[homeBucket].size();
	table[homdBucket].insert(thePair);
	if(table[homeBucket].size()>homeSize)
	dSize++;
}

void erase(const K& theKey)
	{table[hash(theKey)%divisor].erase(theKey);}
```

3.一种改进的实现方法

在每一个链表上增加一个尾节点，可以改进一些程序的性能

尾节点的关键字值最起码要比插入的所有数对的关键字都大

在实际应用中，当关键字为整数时，可以用limits.h文件中定义的INT_MAX常量来代替正无穷

有了尾节点，就可以省区在sortedChain的方法中出现的大部分对空指针的检验操作

### 10.6	一个应用——文本压缩

为了减少一个文本文件的磁盘空间，常常需要把该文本文件压缩编码后存储。

对文件进行编码的是压缩器(compressor),解码的是解压器(decompressor)

#### 10.6.1	LZW压缩

LZW压缩方法把文本字符串映射为数字编码。

首先，该文本串中所有可能出现的字母都被分配一个代码。

字符串和编码的映射关系存储在一个数对字典中，每个数对形如(key,value),其中key是字符串，value是该字符串的代码

#### 10.6.2	LZW压缩的实现

LZW压缩程序包括的函数有：打开输入和输出文件(setFiles),输出压缩文件(output),按位读取输入文件和确定输出代码(compress),主函数(main)

1.建立输入和输出流

给压缩器的输入是文本文件，从压缩器的输出是二进制文件。如果输入文件名为inputFile,则输出文件名为inputFile.zzz。

假定用户在命令行中输入要压缩的文件名为foo，若压缩程序为compress,

命令行：compress foo

就可以得到文件foo的压缩版本foo.zzz

 如果用户没有输入文件名，则提醒用户输入

函数setFiles创建输入输出流in和out,它们分别是ifstream,ostream类型的全局变量

假定函数main的原型为：

void main( int argc,char* argv[])

argc的值是命令行的参数个数，argc[i]为指向第i个参数的指针

若命令行为:compress foo

 则argc=2，argv[0]指向字符串compress,argv[1]指向foo

程序10-21	建立输入和输出流

```c++
void setFiles(int argc,char* argv[])
{
    //建立输入和输出流
    char outputFile[50],inputFile[54];
    //检查是否有文件名
    if(argc>=2)
        strcpy(intputFile,argv[1]);
    else
    {
        //没有文件名，要求提供文件名
        cout<<"Enter name of file to compress"<<endl;
        cin>>inputFile;
    }
    
    //打开二进制文件
    in.open(inputFile,ios::binary);
    if(in.fail())
    {
        cerr<<"Cannot open"<<inputFile<<endl;
        exit(1);
    }
    strcpy(outputFile,inputFile);
    strcat(outputFile,".zzz");
    out.open(outputFile,ios:binary);
}
```

std::cerr标准错误输出流
std::cout标准输出流

std::cerr与std::cout的最大不同是cerr是不带输出缓冲的，直接就可以输出到显示器上,而cout是带输出缓冲的,需要刷新缓冲区才能输出
如果不进行定向的输出操作的话，两个都可以。cerr的速度比较快，因为它没有进入缓冲区。

2.字典组织

为了简化压缩文件的解码，每个代码的位数一样，而且假定都是12位长。

声明hashChains<long,int>h(DIVISOR) 足以用来建立字典表

3.代码输出

因为每个代码是12位，每个字节是8位，所以输出的只能是代码的一部分，即一个字节。先输出代码的前8位，余下的4位留待其后输出。当要输出下一个代码时，加上前面余下的4位，共16位，可以作为2字节输出。



程序10-22是输出函数的C++代码。MASK1为255，MASK2为15，EXCESS为4，BYTE_SIZE为8.

当且仅当有余位待输出时，bitsLeftOver是true, 此时，余下的4位放在变量leftOver中

```c++
void output(long pcode)
{
    //输出8位，把剩余位保存
    int c,d;
    if(bitsLeftOver)
    {
        //前面剩余的位
        d=int (pcode&MASK1);	//右ByteSize位
        c=int ((leftOver<<EXCESS)| (pcode>>BYTE_SIZE));
        out.put(c);
        out.put(d);
        bitsLeftOver=false;
    }
    else
    {
        //前面没有剩余的位
        leftOver=pcode&MASK2;	//右EXCESS位
        c=int(pcode>>EXCESS);
        out.put(c);
        bitsLeftOver=true;
    }
}
```

4.压缩

程序10-23是LZW压缩算法的代码。首先用256个(ALPHA=256)8位ASCII字符和它们的代码对字典初始化。变量codeUsed记录目前已用的代码个数。

因为每个代码为12位，则最多可分配MAX_CODES=4096个代码

为了在字典中寻找最长的前缀，我们按前缀的长度1，2，3，，，的顺序查找，直到发现一个前缀在字典中不存在为止。这时就输出一个代码，并生成一个新代码(除非4096个代码全部用完)

```c++
void compress()
{
    //L~Z~W压缩器
    hashChains<long,int>h(DIVISOR);
    for(int i=0;i<ALPHA;i++)
        h.insert(pairType(i,i));
    int codesUsed=ALPHA;
    
    //输入和压缩
    int c=in.get();	//输入文件的第一个字符
    if(c!=EOF)
    {
        //输入文件不空
        long pcode=c;	//前缀代码
        while((c=in.get())!=EOF)
        {
            //处理字符c
            long theKey=(pcode<<BYTE_SIZE)+c;
            //检查关键字theKey的代码是否在字典中
            pairType* thePair=h.find(theKey);
            if(thePair==NULL)
            {
                //关键字theKey不在表中
                output(pcode);
                if(codesUsed<MAX_CODES)	//建立新代码
                    h.insert(pairType((pcode<<BYTE_SIZE)|c,codesUsed++));
                pcode=c;
            }
            else pcode=thePair->seconde;	//关键字theKey在表中
        }
        
        //输出最后的代码
        output(pcode);
        if(bitsLeftOver)
            out.put(leftOver<<EXCESS);
    }
    
    out.close();
    in.close();
}
```

5. 常量，全局变量和main函数

   程序10-24	常量，全局变量和main函数

   ```c++
   //常量
   const DIVISOR=4099,	//散列函数的除数
   	MAX_CODES=4096,	//2^12
   	BYTE_SIZE=8,
   	EXCESS=4,	//12-BYTE_SIZE
   	ALPHA=256,	//2^BYTE_SIZE
   	MASK1=255,	//ALPHA-1
   	MASK2=15;	//2^EXCESS-1
   typedef pair<const long,int>pairType;
   	//pair.first=key,pair.second=code
   
   //全局变量
   int leftOver;
   bool bitsLeftOver=false;	//待输出的代码位
   ifstream in;	//false表示没有余下的位
   ofstream out;	//输出文件
   
   void main(int argc,char* argv[])
   {
       setFile(argc,argv);
       compress();
   }
   ```

   

   #### 10.6.4	LZW解压缩的实现

   1.字典组织

   前缀代码存储的第一个域，后缀存储在第二个域

   声明为：

   typedef pair<int,char>pairType;

   pairType ht[MAX_CODES];

   程序10-25	计算text(code)

   ```c++
   void output(int code)
   {
       //输出与代码code对应的串
       size=-1;
       while(code>=ALPHA)
       {
           //字典中的后缀
           s[++size]=ht[code].second;
           code=ht[code].first;
       }
       s[++size]=code;		//code<ALPHA
       
       //解压的串是s[size],,,s[0]
       for(int i=size;i>=0;i--)
           out.put(s[i]);
   }
   ```

   

   

2.代码输入

由于12位代码在压缩文件中是按8位字节顺序表示的，所以要把LZW压缩器的函数output的处理过程颠倒过来。这个颠倒过程由getcode函数完成。此处唯一的新常量是MASK，值为15，它可以帮助我们提取一个字节的低4位

程序10-26	从压缩文件中提取代码

```c++
bool getCode(int& code)
{
    //把压缩文件中的下一个代码存入code
    //如果不再有代码，返回false
    int c,d;
    if((c=in.get())==EOF)
        return false;	//不再有代码
    //检查前面是否有剩余的位
    //如果有，与其连接
    if(bitsLeftOver)
        code=(leftOver<<BYTE_SIZE)|c;
    else
    {
        //没有剩余的位，需要再加4位以凑足代码
        d=in.get();	//另外8位
        code=(c<<EXCESS)|(d>>EXCESS);
        leftOver=d&MASK;	// 存储4位
    }
    bitsLeftOver=!bitsLeftOver;
    return true;
}
```

3.解压缩

程序10-27	LZW解码器

```c++
void decompress()
{
    //解压一个压缩文件
    int codesUsed=ALPHA;	//当前代码codesUsed
    
    //输入和解压缩
    int pcode,	//前面的代码
    	ccode;	//当前的代码
    if(getCode(pcode))
    {
        //文件不空
        s[0]=pcode;	//取pcode的代码
        out.put(s[0]);	//输出pcode的串
        size=0;	//s[size]是最后一个输出串的第一个字符
        
        while(getCode(ccode))
        {
            //又一个代码
             if(ccode<codesUsed)
             {
                 //确定ccode
                 output(ccode);
                 if(codesUsed<MAX_CODES)
                 {
                     //建立新代码
                     ht[codesUsed].first=pcode;
                     ht[codesUsed++].second=s[size];
                 }
             }
            else
            {
                //特殊情况，没有定义的代码
                ht[codesUsed].first=pcode;
                ht[codesUsed++].secod=s[size];
                output(ccode);
            }
            pcode=ccode;
        }
    }
    
	out.close();
    in.close();	
}
```

4.常量，全局变量和main函数

程序10-28	解压缩程序所包含的常量，全局变量和main函数

```c++
//常量
const MAX_CODES=4096,	//2^12
	BYTE_SIZE=8,
	EXCESS=4,	//12-BYTE_SIZE
	ALPHA=256,	//2^EXCESS-1
	MASK=15;
	
typedef pair<int,char> pairType;

//全局变量
pairType ht[MAX_CODES];	//字典
char s[MAX_CODES];	//用来重建文本
int size;	//重建文本的大小
int leftOver;	//待输出的代码位 
bool bitsLeftOver=false;	//false表示没有剩余位
ifstream in;	//输入文件
ofstream out;	//输出文件

void main(int argc,char* argv[])
{
    setFiles(argc,argv);
    decompress();
}
```





## 第11章	二叉树和其他树

本章研究两种基本的树：一般树(简单树)和二叉树

还有很多树：堆(heap),左高树(leftist tree),锦标赛树(tournament tree),二叉搜索树(binary tree),AVL树，红黑树(red and black tree),伸展树(splay tree)和B树

还有一些书上不包含的内容:配对堆(pairing heap),区间堆(interval heap),双端优先级队列的树结构(tree structures for double-ended priority queue),字典树(tries，也称前缀树，单词查找树，键树)，后缀树(suffix tree)



此章还涵盖了如下内容:

- 树和二叉树的术语，如高度，深度，层，根，叶子，孩子，双亲，兄弟
- 二叉树的数组和链表表示
- 4种常用的二叉树遍历方法：前序遍历，中序遍历，后序遍历，层次遍历





### 11.1	树

定义11-1	一棵树t是一个非空的有限元素的集合，其中一个元素为根(root),其余的元素(如果有的话)组成t的子树(subtree)



在书中没有孩子的元素称为**叶子(leaf)**

树的另一常用术语为**级 (level)**

一棵树的高度(height)或深度(depth)是树中级的个数

一个**元素的度(degree of an element)**是指其孩子的个数

**一棵树的度(degree of a tree)**是其元素的度的最大值

### 11.2	二叉树

定义11-2	一颗**二叉树(binary tree)**t是有限个元素的集合(可以为空)。当二叉树非空时，其中有一个元素称为**根**，余下的元素(如果有的话)被划分成两棵二叉树，分别称为t的左子树和右子树

二叉树和树的根本区别是：

- 二叉树的每个元素都恰好有两棵子树(其中一个或两个可能为空)。而树的每个元素可有任意数量的子树

- 在二叉树中，每个元素的子树都是有序的，也就是说，有左子树和右子树之分。而树的子树是无序的。

二叉树可以为空，但树不能为空



和树一样，二叉树也是根节点在顶部。二叉树左(右)子树中的元素画在根的左(右)下方。

每个元素节点和其子节点之前用一条边相连。

### 11.3	二叉树的特性

特性11-1	一棵二叉树有n个元素,n>0,它有n-1条边

特性11-2	一棵二叉树的高度为h,h>=0,它最少有h个元素，最多有2^h-1个元素

特性11-3	一棵二叉树有n个元素，n>0,它的高度最大为n，最小高度为[log~2(n+1)]

当高度为h的二叉树恰好有2^h-1个元素时，称其为**满二叉树(full binary tree)**

对高度为h的满二叉树的元素，从第一层到最后一层，在每一次中从左至右，顺序编号，从 1到2^h-1	假设从满二叉树中删除k个其编号为2^h-i元素，1<=i<=k<2^h,所得到的二叉树称为**完全二叉树(complete binary tree)**

特性11-4	设完全二叉树的一元素其编号为i,1<=i<=n,有以下关系成立:

如果i=1,则该元素为二叉树的根。若i>1,则其父节点的编号为[i/2]

如果2i>n,则该元素无左孩子。否则，其左孩子的编号为2i.

如果2i+1>n, 则该元素无右孩子。否则，其右孩子的编号为2i+1

### 11.4	二叉树的描述

#### 11.4.1	数组描述

二叉树的数组表示利用了特性11-4，把二叉树看做是缺少了部分元素的完全二叉树。

存在一种二叉树叫做**右斜二叉树(right-skewed binary tree)**

只有当缺少的元素数目较少时，这种描述方法才是有用的

#### 11.4.2	链表描述

二叉树最常用的表示方法是用指针。

每个元素用一个节点表示，节点有两个指针域,leftChild,rightChild.除两个指针域外，每个节点还有一个element域。

程序11-1	链表二叉树的节点结构

```c++
template<class T>
struct binaryTreeNode
{
    T element;
    binaryTreeNode<T> *leftChild,	//左子树
    binaryTreeNode<T> *rightChild;	//右子树
    
    binaryTreeNode() {leftChild=rightChild=NULL;}
    binaryTreeNode(const T& theElement)
    {
        element(theElement)
            leftChild=rightChild=NULL;
    }
    binaryTreeNode(const T& theElement,
                  binaryTreeNode *theLeftChild,
                  binaryTreeNode *theRightChild)
    {
        element(theElement)
            leftChild=theLeftChild;
        rightChild=theRightChild;
    }
};
```

### 11.5	二叉树常用操作

- 确定高度

- 确定元素数目

- 复制
- 显示或打印二叉树
- 确定两棵二叉树是否一样
- 删除整棵树

这些操作可以通过有步骤的遍历二叉树来完成。在二叉树的**遍历(traversal)**中，每个元素仅被访问一次。访问一个元素，意味着可以对该元素实施任何操作。

### 11.6	二叉树遍历

4种遍历二叉树的方法

- 前序遍历

- 中序遍历

- 后序遍历

- 层次遍历

程序11-2	前序遍历

```c++
template<class T>
void preOrder(binaryTreeNode<T> *t)
{
    //前序遍历二叉树 *t
    if(t!=NULL)
    {
        visit(t);	//访问树根
        preOrder(t->leftChild);	///前序遍历左子树
        preOredr(t->rightChild);	//前序遍历右子树
    }
}
```

程序11-3	中序遍历

```c++
template<class T>
void inOrder(binaryTreeNode<T> *t)
{
    //中序遍历二叉树*t
    if(t!=NULL)
    {
        inOrder(t->leftChild);	//中序遍历左子树
        visit(t);	访问树根
        inOrder(t->rightChild);		//中序遍历右子树
    }
}
```

程序11-4	后序遍历

```c++
template<class T>
void postOrder(binaryTreeNode<T> *t)
{
	//后序遍历二叉树*t
    if(t!=NULL)
    {
        postOrder(t->leftChild);	//后序遍历左子树
        postOrder(t->rightChild);	//后序遍历右子树
        visit(t);	//访问树根
    }
}
```

这三种遍历的区别在于对每个节点的访问时间不同。

在前序遍历种，先访问一个节点，再访问该节点的左右子树；

在中序遍历种先访问一个节点的左子树，然后访问该节点，最后访问右子树。

在后序遍历种，先访问一个节点的左右子树，再访问该节点

程序11-5	visit函数

```c++
template<class T>
void visit(binaryTreeNode<T> *x)
{
    //访问节点*x,仅输出element域
    cout<<x->element<<' ';
}
```

**中缀(infix)**是我们通常的书写形式。

程序11-6	输出完全括号化的中缀表达式

```c++
template<class T>
void infix(binaryTreeNode<T> &t)
{
    //输出中缀表达式
    if(t!=NULL)
    {
        cout<<'(';
        infix(t->leftChild);	//左操作数
        cout<<t->element;	//操作符
        infix(t->rightChild);	//右操作数
        cout<<')';
    }
}
```

在层次遍历种，从顶层到底层，在同一层中，从左到右，依次访问树的元素。因为层次遍历需要队列而不是栈，所以编写递归程序很困难。

程序11-7	层次遍历

```c++
template<class T>
void levelOrder(binaryTreeNode<T> *t)
{
    //层次遍历二叉树 *t
    arrayQueue<binaryTreeNode<T>* q;
    while(t!=NULL)
    {
        visit(t);	//访问t
        
        //将t的孩子插入队列
        if(t->leftChild!=NULL)
            q.push(t->leftChild);
        if(t->rightChild!=NULL)
            q.push(t->rightChild);
        
        //提取下一个要访问的节点
        try(t=q.front();)
            catch(queueEmpty)	{return;}
        q.pop();
    }
}
```

在上述程序中，仅当树为非空时，才进入while循环。进入while循环之后，首先访问根节点，再把其子节点(如果有)加到队列中，然后访问队首元素。若队列为空，则front()抛出一个类型为queueEmpty的异常；若队列不空，则front()的返回值指向队首元素指针，下一次循环将访问这个元素。

### 11.7	抽象数据类型Binarytree

ADT11-1	二叉树的抽象数据类型

```c++
抽象数据类型 binaryTree
{
    实例
        元素集合；如果非空，则集合划分一为一个根，一颗左子树和一颗右子树；每一棵子树也是二叉树
    操作
        empty();若树为空，则返回true，否则返回false
        size();返回二叉树的节点/元素个数
        preOrder(visit);前序遍历二叉树;visit是访问函数
        inOrder(visit);中序遍历二叉树
        postOrder(visit);后序遍历二叉树
        levelOrder(visit);层次遍历二叉树
}
```

二叉树遍历方法的参数类型是：

```
void (*) (T*)
```

程序11-8	二叉树抽象类

```c++
template<class T>
class binaryTree
{
    public:
    virtual ~binaryTree() {}
    virtual bool empty() const=0;
    virtual int size() const=0;
    virtual void preOrder(void (*) (T *))=0;
    virtual void inOrder(void (*) (T *))=0;
    virtual void postOrder(void (*) (T *))=0;
    virtual void levelOrder(void (*) (T *))=0;
};
```

### 11.8	类linkedbinarytree

类linkedBinaryTree是抽象类binaryTree的派生类，节点的类型是binaryTreeNode

root是指向二叉树节点的指针，treeSize是二叉树节点个数

程序11-9	类linkedBinaryTree的数据成员和访问方法

```c++
template<class E>
class linkedBinaryTree:public binaryTree<binaryTreeNode<E> >
{
    public:
    linkedBinaryTree(){root=NULL; treeSize=0;}
    ~linkedBinaryTree(){erase();};
    bool empty() const {return treeSize==0;}
    int size() const {return treeSize;}
    void preOrder(void(*theVisit)(binaryTreeNode<E>*))
    {visit=theVisit;preOrder(root);}
    void inOrder(void(*theVisit)(binaryTreeNode<E>*))
    {visit=theVisit;inOrder(root);}
    void postOrder(void(*theVisit)(binaryTreeNode<E>*))
    {visit=theVisit;postOrder(root);}
    void levelOrder(void(*)(binaryTreeNode<E>*));
    void erase
    {
        postOrder(dispose);
        root=NULL;
        treeSize=0;
    }
    private:
    binaryTreeNode<E> *root;	//指向根的指针
    int treeSize;	//树的节点个数
    static void (*visit)(binaryTreeNode<E>*);
    static void preOrder(binaryTreeNode<E> *t);
    static void inOrder(binaryTreeNode<E> *t);
    static void postOrder(binaryTreeNode<E> *t);
    static void dispose(binaryTreeNode<E> *t) {delete t;}
};
```

程序11-10	类linkedBinaryTree的私有前序遍历方法

```c++
template<class E>
void linkedBinaryTree<E>::preOrder(binaryTreeNode<E> *t)
{
    //前序遍历
    if(t!=NULL)
    {
        linkedBinaryTree<E>::visit(t);
        preOrder(t->leftChild);
        preOrder(t->rightChild);
    }
}
```

程序11-11	确定二叉树高度

```c++
int height() const {return height(root);}

template<class E>
int linkedBinaryTree<E>::height(binaryTreeNode<E> *t)
{
    //返回根为*t的树的高度
    if(t==NULL)
        return 0;	//空树
    int h1=height(t->leftChild);	//左树高
    int hr=height(t->rightChild);	//右树高
    if(h1>hr)
        return ++h1;
    else 
        return ++hr;
}
```

### 11.9	应用

#### 11.9.1	设置信号放大器

1.问题描述

可以用术语**信号(signal)**来指称输送的资源(汽油，天然气，电力等)

为了保证信号衰减不超过容忍值(tolerance),应在网络中至关重要的位置上放置信号放大器(signal booster).

信号放大器可以增加信号的压强或电压使它与源点相同，可以增强信号，使信号与噪声之比与源点的相同



2.求解策略

设degradFromParent(i)表示节点i与其父节点间的衰减量

放置放大器和计算degradeToLeaf的伪代码

```c++
degradeToLeaf(i)=0;
for(each child j of i)
	if(degradeToLeaf(j)+degradeFromParent(j))>容忍值)
	{
        在j处放一个放大器
            degradeToLeaf(i)=max{degradeToLeaf(i),degradeFromParent(j)};
    }
else
    degradeToLeaf(i)=max{degradeToLeaf(i),degradeToLeaf(j)+degradeFromParent(j)};
```

3.C++实现

当分布树没有节点超过两个孩子时，可用程序11-9的类linkedBinaryTree表示为二叉树，二叉树节点的数据域element的类型时程序11-12的结构booster。

在结构booster中，域boosterHere用来区分具有放大器的节点和没有放大器的节点

程序11-12	结构booster

```c++
struct booster
{
    int degradeToLeaf,	//到达叶子时的衰减量
    	degradeFromParent;	//从父节点出发的衰减量
    bool boosterHere;	//当且仅当放置了放大器时，值为真
    
    void output(ostream& out) const
    {out<<boosterHere<<' '<<degradeToLeaf<<' '
    	<<degradeFromParent<<' ';}
};

//重载操作符<<
ostream operator<<(ostream& out,booster x)
{x.output(out);return out;}
```

程序11-13	在二叉树中放置放大器并计算degradeToLeaf值

```c++
void placeBoosters(binaryTreeNode<booster> *x)
{
    //计算*x的衰减量。若小于容忍值，则在x的子节点放置一个放大器
    
    x->element.degradeToLeaf=0;	//初始化x处的衰减
    
    ///计算x的左子树的衰减量。若大于容忍值，则在x的左孩子处放置一个放大器
    binaryTreeNode<booster> *y=x->leftChild;
    if(y!=NULL)
    {
        //x有一颗非空左子树
        int degradetion=y->element.degradeToLeaf+
            			y->element.degradeFromParent;
        if(degradetion>tolerance)
        {
            //在y处放置一个放大器
            y->element.boosterHere=true;
            x->element.degradeToLeaf=y->element.degradeFromeParent;
        }
        else 	//不需要在y处放置放大器
            x->element.degradeToLeaf=degradetion;
    }
    
    y=x->rightChild;
    if(y!=NULL)
    {
        //x有一棵非空右子树
        int degradetion=y->element.degradeToLeaf+
            			y->element.degradeFromParent;
        if(degradetion>tolerance)
        {
            //在y处放置一个放大器
            y->element.boosterHere=true;
            degradetion=y->elemen.degradeFromParent;
        }
        if(x->element.degradeToLeaf<degradetion)
            x->element.degradeToLeaf=degradetion;
    }
}
```

#### 11.9.2	并查集

1.问题描述

2.集合的树形描述

4.C++实现

程序11-14	基于树结构的并查集问题解决方案

```c++
void initialize(int numberOfElements)
{
    //初始化numberOfElements棵树，每棵树一个元素
    parent=new int[numberOfElements+1];
    for(int e=1;e<=numberOfElements;e++)
        parent[e]=0;
}

int find(int theElement)
{
    //返回元素theElement所在树的根
    while(parent[theElement]!=0)
        theElement=parent[theElement];	//向上移动一层
    return theElement;
}

void unite(int rootA,int rootB)
{
    //合并两棵其根节点不同的树(rootA,rootB)
    parent[rootB]=rootA;
}
```

6.合并函数的性能改进

在对根为i和根为j的树进行合并操作时，利用重量规则或高度规则，可以提高并查集算法的性能

定义11-3	重量规则：若根为i的树的节点数少于根为j的树的节点数，则将j作为i的父节点，否则，将i作为j的父节点

定义11-4	高度规则：若根为i的树的盖度小于根为j的树的高度，则将j作为i的父节点，否则，将i作为j的父节点

程序11-15	实现重量规则时使用的结构

```c++
struct unionFindNode
{
    int parent;	//若为真，则表示树的重量，否则是父节点的指针
    bool root;	//当且仅当是根时，值为真
    
    unionFindNode()
    {parent=1;root=true;}
};
```

程序11-16	用重量规则进行合并

```c++
void initialize(int numerOfElements)
{
    //初始化numberOfElements棵树，每棵树包含一个元素
    node=new unionFindNode[numberOfElements+1];
}

int find(int theElement)
{
    //返回元素所在树的根
    while(!node[theElement].root)
        theElement=node[theElement].parent;	//向上移动一层
    return theElement;
}

void unite(int rootA,int rootB)
{
    //使用重量规则，合并其根不同的树(rootA, rootB)
    if(node[rootA].parent<node[rootB].parent)
    {
        //树rootA称为树rootB的子树
        node[rootB].parent+=node[rootA].parent;
        node[rootA].root=false;
        node[rootA].parent=rootB;
    }
    else
    {
        //树rootB称为树rootA的子树
        node[rootA].parent+=node[rootB].parent;
        node[rootB].root=false;
        node[rootB].parent=rootA;
    }
}
```

引理11-1	重量规则引理	假设从单元素集合除法，用重量规则进行合并操作.若以此方式构建一颗具有p个节点的树t，则t的高度最多为[log2~p]+1

7.查找函数的性能改进

为了进一步改进查找函数在最坏情况下的性能，我们缩短从元素e到根的查找路径。

这个方法利用了**路径压缩(path compression)**过程，这个过程的实现至少有3种不同的途径——路径紧缩(path compaction),路径分割(path splitting)和路径对折(path halving)

在路径紧缩种，从待查节点到根节点的路径上，所有节点的parent指针都被改为指向根节点。

虽然路径紧缩增加了单个查找操作的时间，但它减少了此后查找操作的时间。

程序11-17	利用路径紧缩来查找一个元素

```c++
int find(int theElement)
{
    //返回元素theElement所在树的根
    //紧缩从元素theElement到根的路径
    
    //theRoot最终晒树的根
    int theRoot=theElement;
    while(!node[theRoot].root)
        theRoot=node[theRoot].parent;
    
    //紧缩从theElement到theRoot的路径
    int currentNode=theElement;	//从theElement开始
    while(currentNode!=theRoot)
    {
        int parentNode=node[currentNode].parent;
        node[currentNode].parent=theRoot;	//移到第2层
        currentNode=parentNode;	//移到原来的父节点
    }
    
    return theRoot;
}
```

在路径分割种，从e节点到根节点的路径上，除根节点和其子节点之外，每个节点的parent指针都被改为指向各自的祖父。

在路径分割时，只考虑从e到根节点的一条路径就够了。

在路径对折种，从e节点到根节点的路径上，除根节点和其子节点之外，每隔一个节点，其parent指针都被改为指向各自的祖父。

在路径对折中，指针改变的个数仅为路径分割中的一半。同样，在路径对折中只考虑从e到根的一条路径就够了。

## 第12章	优先级队列

与之前的FIFO结构的队列不同，在优先级队列中，元素出队列的顺序由元素的优先级决定。可以按照优先级的递增顺序，也可以按照优先级的递减顺序，但不是元素进入队列的顺序。

堆时实现优先级队列效率很高的数据结构。堆是一课完全二叉树。

在链表结构中，在高度和重量上的左高树也适合于表示优先级队列。

C++的STL类priority-queue是用堆实现的优先级队列

### 12.1	定义和应用

**优先级队列(priority queue)**是0个或多个元素的集合，每个元素都有一个优先权或值，对优先级队列执行的操作有：1.查找一个元素；2.插入一个新元素；3.删除一个元素

这些操作对应的函数分别是top,push,pop

在**最小优先级队列(min priority queue)**中，查找和删除的元素都是优先级最小的元素

在**最大优先级队列(max priority queue)**中，查找和删除的元素都是优先级最大的元素

优先级队列的元素可以有相同的优先级，对这样的元素，查找与删除可以按任意顺序处理

### 12.2	抽象数据类型

ADT12-1	最大优先级队列的抽象数据类型说明

```c++
抽象数据类型 maxPriorityQueue
{
    实例
        有限个元素集合，每一个元素都有优先级
        
    操作
        empty();返回值为true,当且仅当队列为空
        size();返回队列的元素个数
        top();返回优先级最大的元素
        pop();删除优先级最大的元素
        push(x);插入元素x
}
```

程序12-1	抽象类maxPriorityQueue

```c++
template<class T>
class maxPriorityQueue
{
    public:
    virtual ~maxPriorityQueue(){}
    virtual bool empty() const =0;
    		//返回true，当且仅当队列为空
    virtual int size() const =0;
    		//返回队列的元素个数
    virtual const T& top() =0;
    		//返回优先级最大的元素的引用
    virtual void pop()=0;
    		//删除队首元素
    virtual void push(const T&& theElement)=0;
    		//插入元素theElement
}
```

### 12.3	线性表

描述最大优先级队列最简单的方法是无序线性表。

### 12.4	堆

#### 12.4.1	定义

定义12-1	一棵大根树(小根树)是这样的一棵树，其中每个节点的值都大于(小于)或等于其子节点(如果有子节点的话)的值

定义12-2	一个大根堆(小根堆)既是大根树(小根树)也是完全二叉树

#### 12.4.2	大根堆的插入

把新元素插入新节点，然后沿着从新节点到根节点的路径，执行一趟起泡操作，将新元素与其父节点的元素比较交换，直到后者大于或等于前者为止。

#### 12.4.3	大根堆的删除

在大根堆中删除一个元素，就是删除根节点的元素

#### 12.4.5	类maxHeap

类maxHeap用来实现一个最大优先级队列。类的数据成员是heap(一个类型为T的一维数组)，arrayLength(数组heap的容量),heapSize(堆的元素个数).

top方法在堆为空时，抛出一个异常queueEmpty，在堆不为空时，返回一个值heap[1].

 程序12-2	大根堆的插入

```c++
template<class T>
void maxHeap<T>::push(const T& theElement)
{
    //把元素 theElement 加入堆
    
    //必要时增加数组长度
    if(heapSize==arrayLength-1)
    {
        //数组长度加倍
        changeLength1D(heap,arrayLength,2*arrayLength);
        arrayLength*=2;
    }
    //为元素 theElement寻找插入位置
    //currentNode从新叶子向上移动
    int currentNode=++heapSize;
    while(currentNode!=1 && heap[currentNode/2]<theElement)
    {
        //不能把元素 theElement 插入在 heap[currentNode]
        heap[currentNode]=heap[currentNode/2];	//把元素向下移动
        currentNode/=2;	//currentNode 移向双亲
    }
    
    heap[currentNode]=theElement;
}
```

程序12-3	从大根堆删除最大元素

```c++
template<class T>
void maxHeap<T>::pop()
{
    //如果堆为空，抛出异常
    if(heapSize==0)		//堆为空
        throw queueEmpty();
    
    //删除最大元素
    heap[1].~T();
    
    //删除最后一个元素，然后重新建堆
    T lastElement=heap[heapSize--];
    
    //从根开始，为最后一个元素寻找位置
    int currentNode=1,
    child=2;	//currentNode的孩子
    while(child<=heapSize)
    {
        //heap[child]应该是currentNode的更大的孩子
        if(child<heapSize && heap[child]<heap[child+1])
            child++;
        
        //可以把lastElement放在heap[currentNode]吗？
        if(lastElement>=heap[child])
            break;	//可以
        
        //不可以
        heap[currentNode]=heap[child];	//把孩子child向上移动
        currentNode=child;
        child*=2;
    }
    heap[currentNode]=lastElement;
}
```

方法initialize在数组theHeap中建堆。函数一开始，令heap指向数组theHeap，heapSize等于数组中元素个数theSize。在for循环中，我们从最后一个具有孩子的节点开始到根节点进行扫描，并用root表示正在处理的节点。对于每一个root值，嵌入的while循环把以root为根的子树调整为大根堆。for循环和程序12-3的pop函数相似

程序12-4	初始化一个非空大根堆

```c++
template<class T>
void maxHeap<T>::initialize(T *theHeap,int theSize)
{
    //在数组theHeap[1:theSize]中建大根堆
    delete [] heap;
    heap=theHeap;
    heapSize=theSize;
    
    //堆化
    for(int root=heapSize/2;root>=1;root--)
    {
        T rootElement=heap[root];
        
        //为元素rootElement寻找位置
        int cild=2*root;	//孩子child的双亲是元素rootElement的位置
        while(child<=heapSize)
        {
            //heap[child]应该是兄弟中较大者
            if(child<heapSize && heap[child]<heap[child+1])
                child++;
            
            //可以把元素rootElement放在heap[child/2]吗
            if(rootElement>=heap[child])
                break;	//可以
            
            //不可以
            heap[child/2]=heap[child];	//把孩子向上移
            child*=2;
        }
        heap[child/2]=rootElement;
    }
}
```

 

#### 12.4.6	堆和STL

STL的类priority_queue利用了基于向量的堆来实现大根堆，它允许用户自己制定优先级的比较函数，因此，这个类也可以用于实现小根堆

### 12.5	左高树

#### 12.5.1	高度优先与宽度优先的最大及最小左高树

 12.4的堆结构是一种**隐式数据结构(implicit date structure)**

用完全二叉树表示的堆在数组中是隐式存储的(即没有明确的指针或其他数据能够用来重塑这种结构)

由于没有存储结构信息，这种表示方法的空间利用率很高，它实际上没有浪费空间。而且它的时间效率也很高。

它并不适合于所有优先级队列的应用，尤其是当两个优先级队列或多个长度不同的队列需要合并的时候，这时我们就需要其他数据结构了。

考察一棵二叉树，它有一类特殊的节点叫做**外部节点(external node)**,它代替树中的空子树。其余节点叫做**内部节点(internal node)**.增加了外部节点的二叉树被称为**扩充二叉树(extended binary tree)**

定义12-3	一棵二叉树称为**高度优先左高树(height-biased leftist tree,HBLT)**,当且仅当其任何一个内部节点的左孩子的s值都大于或等于右孩子的s值。

定理12-1	令x为HBLT的一个内部节点，则

1. 以x为根的子树的节点数目至少为x^s(x)-1
2. 若以x为根的子树右m个节点，那么s(x)最多为log~2(m+1)
3. 从x到一外部节点的最右路径(即从x开始沿右孩子移动的路径)的长度为s(x)

定义12-4 若一棵HBLT同时还是大根树，则称为最大HBLT(max HBLT).若一棵HBLT同时还是小根树,则称为最小HBLT(min HBLT)

定义12-5	一棵二叉树称为重量优先左高树(weight-biased leftist tree,WBLT),当且仅当其任何一个内部节点的左孩子的w值都大于或等于右孩子的w值。若一棵WBLT同时还是大根树，则称为最大WBLT(max WBLT).若一棵WBLT同时还是小根树，则成为最小WBLT(min WBLT)

#### 12.5.2	最大HBLT的插入

最大HBLT的插入操作可利用最大HBLT的合并操作来实现

#### 12.5.3	最大HBLT的删除

最大元素在根中。若根被删除，则分别以左右孩子为根的子树是两棵最大HBLT。将这两棵最大HBLT合并，便是删除后的结果。

#### 12.5.4	两棵最大HBLT的合并

算法从两棵HBLT的根开始仅沿右孩子移动

合并策略最好用递归来实现。

#### 12.5.5	初始化

初始化过程是将n个元素逐个插入最初为空的最大HBLT。

为得到具有线性时间的初始化算法，我们首先创建n个仅含一个元素的最大HBLT，这n棵树组成一个FIFO队列，然后从队列中依次成对删除HBLT，然后将其合并后再插入队列末尾，直到队列只有一棵HBLT为止

#### 12.5.6	类maxHblt

在一棵最大HBLT中，节点的数据类型是binaryTreeNode<pair<int,T>>,其中binaryTreeNode如程序11-1所示；pair的第一个成员是节点的s值，第二个成员是优先级队列元素。

我们用类maxHblt实现一棵最大HBLT，它拓展了类linkedBinaryTree

基本方法empty,size,top的代码与maxHeap类似

程序12-5	合并两棵左高树

```c++
template<class T>
void maxHbly<T>::meld(binaryTreeNode<pair<int,T>> * &x)
						binaryTreeNode<pair<int,T>>* &y
{
    //合并分别以*x和*y为根的两棵左高树
    //合并后的左高树以x为根，返回x的指针
    if(y==NULL)	//y为空
        return ;
    if(x==NULL)	//x为空
    {x=y;return;}
    
    //x和y都不空，必要时交换x和y
    if(x->element.second<y->element.second)
        swap(x,y);
    
    //x->element.second>=y->element.second
    
    meld(x->rightChild,y);
    //如果需要，交换x的子树，然后设置x->element.second的值
    if(x->leftChild==NULL)
    {
        //左子树为空，交换子树
        x->leftChild=x->rightChild;
        x->rightChild=NULL;
        x->element.first=1;
    }
    else
    {
        //只有左子树的s值较小时才交换
        if(x->leftChild->element.first<x->rightChild->element.first)
            swap(x->leftChild,x->rightChild);
        //更新x的s值
        x->element.first=x->rightChild->element.first+1;
    }
}
```

为了将元素theElement插入一棵最大HBLT，程序12-6首先创建一棵只含元素theElement的最大HBLT，然后通过私有成员函数meld将此树与要插入的最大HBLT合并

在程序12-6的pop代码中，若最大HBLT为空，则抛出一个queueEmpty异常；若不为空，则删除根，然后调用私有方法meld将其左右子树合并

程序12-6	公有方法meld,push,pop

```c++
template<class T>
void maxHblt<T>::meld(maxHblt<T>& theHblt)
{
    //把左高树*this和theHblt合并
    meld(root,theHblt.root);
    treeSize+=theHblt.treeSize;
    theHblt.root=NULL;
    theHblt.treeSize=0;
}

template<class T>
void maxHblt<T>::push(const T& theElement)
{
    //把元素theElement插入左高树
    //建立只有一个节点的左高树
    binaryTreeNode<<pair<int, T> > *q=
        new binaryTreeNode<<pair<int, T> >(pair<int, T>(1,theElement));
    
    //将左高树q和原树合并
    meld(root,q);
    treeSize++;
}

template<class T>
void maxHblt<T>::pop()
{
    //删除最大元素
    if(root=NULL)
        throw queueEmpty();
    
    //树不空
    binaryTreeNode<<pair<int, T> > *left=root->leftChild,
    								*right=root->rightChild;
    
    delete root;
    root=left;
    meld(root,right);
    treeSi
}
```

程序12-7	最大HBLT的初始化

代码用一个基于数组的FIFO队列保存在初始化过程中产生的最大HBLT。

在第一个for循环中，产生了n个只含一个元素的最大HBLT，并将此插入初始为空的队列中。

在第二个for循环中，每次从队列中删除两个最大HBLT进行合并，然后把结果加入队列。

当for循环结束时，队列仅有一棵最大HBLT

```c++
template<class T>
void maxHblt<T>::initialize(T* theElements,int theSize
{
    //用数组theElements[1:theSize]建立左高树
    arrayQueue<binaryTreeNode<pair<int, T> *> q(theSize);
    erase();	//使*this为空
    
    //初始化树的队列
    for(int i=1;i<theSize;i++)
        //建立只有一个节点的树
        q.push(new binaryTreeNode<pair<int, T> >
              (pair<int, T>(1,theElements[i])));
    
    //从队列中重复取出两棵树合并
    for(i=1;i<=theSize-1;i++)
    {
        //从队列中删除两棵树合并
        binaryTreeNode<pair<int, T> > *b=q.front();
        q.pop();
        binaryTreeNode<pair<int, T> > *c=q.front();
        q.pop();
        meld(b,c);
        //把合并后的树插入队列
        q.push(b);
    }
    
    if(theSize>0)
        root=q.front();
    treeSize=theSize;
}
```

### 12.6	应用

#### 12.6.4	堆排序

堆可以用来实现n个元素的排序。先用n个待排序的元素来初始化一个大根堆，然后从堆中逐个提取(即删除)元素。结果，这些元素按非递增顺序排列

这种排序策略称为堆**排序(heap sort)**,

程序12-8	用堆排序给数组a[1:n]排序

使用了大根堆的方法deactiveArray,将maxHeap<T>::heap置为NULL。

```c++
template<class T>
void heapSort(T a[],int n)
{
    //使用堆排序方法给数组a[1:n]排序
    //在数组上建立大根堆
    maxHeap<T> heap(1);
    heap.initialize(a,n);
    
    //逐个从大根堆中提取元素
    for(int i=n-1;i>=1;i--)
    {
        T x=heap.top();
        heap.pop();
        a[i+1]=x;
    }
    
    //从堆的析构函数中保存数组a
    heap.deactivateArray();
}
```

#### 12.6.2	机器调度

一个工厂具有m台一模一样的机器。我们有n个任务需要处理。

设作业i的处理时间为t~i,这个时间就包括把作业放入机器和从机器上取下的时间。

所谓**调度(schedule)**是指按作业在机器上的运行睡觉分配作业，使得：

- 一台机器在同一时间内只能处理一个作业
- 一个作业不能同时在两台机器上处理
- 一个作业i的处理时间是t~i个时间单位

假设每台机器在0时刻都是可用的，**完成时间(finish time)**或**调度长度(length of a schedule)**是指完成所有作业的时间。

调度问题是一类臭名昭著的**NP-复杂问题(NP表示nondeterministic polynornial)**中的一个

NP-复杂及NP-完全问题是指尚未找到具有多项式时间复杂性算法的问题

NP-完全问题是一类判定问题，也就是说，对这类问题的每一个实例，答案为是或否。

机器调度问题不是一个判定问题，因此对每一个问题实例，都是按照某种方案把作业分配给机器外，还给定了时间TMin，要求确定是否存在一种调度，它的完成时间为TMin或更少。

NP-复杂问题的优化问题经常用**近似算法(approximation aigorithm)**解决，虽然近似算法不能保证得到最优解，但是能保证得到近似最优解

在机器调度问题中，可以采用一个简单调度策略，称为 **最长处理时间(longest processing time,LPT)**,它的调度长度是最优调度长度的 4/3-1/(3m)

在LPT算法中，作业按处理时间的递减顺序排序。当一个作业需要分配时，总是分配给最先变为空闲的机器。没有固定的分配方案。



程序12-9	基于a[1:n]的m台机器的LPT调度

```c++
void makeSchedule(jobNode,a [],int n,int m)
{
    //输出一个基于n个作业a[1:n]的m台机器的LPT调度
    if(n<=m)
    {
        cout<<"Schedule each job on a different machine."<<endl;
        return;
    }
    
    heapSort(a,n);	//按递增顺序排序
    
    //初始化m台机器，建立小根堆
    minHeap<machineNode> machineHeap(m);
    for(int i=1;i<=m;i++)
        machineHeap.push(machineNode(i,0));
    
    //生成调度计划
    for(int i=n;i>=1;i--)
    {
        //把作业i安排在第一台空闲的机器
        machineNode x=machineHeap.top();
        machineHeap.pop();
        cout<<"Schedule job"<<a[i].id
            <<"on machine"<<x.id<<"from"<<x.avail
            <<"to"<<{x.avail+a[i].time}<<endl;
        x.avail+=a[i].time;	//这台机器新的空闲空间
        machineHeap.push(x);
    }
}
```

#### 12.6.3	霍夫曼编码

**霍夫曼编码(Huffman code**)是一种文本压缩算法，这种算法的依据是不同符号在一段文本中相对出现的频率。

一个符号出现的次数称为**频率(frequency)**

WEP是二叉树的**加权外部路径长度(weighted external path length)**

霍夫曼树(Huffman tree):一组给定的频率，其WEP最小

用霍夫曼编码对一个字符串(或一段文本)进行编码：

1. 确定字符串的符号和它们出现的频率
2. 建立霍夫曼树，其中外部节点用字符串中的符号表示，外部节点的权用相应符号的频率表示
3. 沿着从根部到尾部节点的路径遍历，取得每个符号的代码
4. 用代码替代字符串中的符号





程序12-10	构造一棵霍夫曼树

```c++
template<class T>
linkedBinaryTree<int>* huffmanTree(T weight[],int n)
{
    //用权weight[1:n]生成霍夫曼树，n>=1
    //创建一组单节点树
    huffmanNode<T> *hNode=new huffmanNode<T> [n+1];
    linkedBinaryTree<int> emptyTree;
    for(int i=1;i<=n;i++)
    {
        hNode[i].weight=weight[i];
        hNode[i].tree=new linkdeBinaryTree<int>;
        hNode[i].tree->makeTree(i,emptyTree,emptyTree);
    }
    
    //使一组单节点树构成小根堆
    minHeap<huffmannode<T> > heap[1];
    heap.initialize(hNode,n);
    
    //不断从小根堆中提取两个树合并，直到剩下一棵树
    huffmanNode<T> w,x,y;
    linkedBinaryTree<int> *z;
    for(i=1;i<n;i++)
    {
        //从小根堆中取出两棵最轻的树
        x=heap.top();heap.pop();
        y=heap.top();heap.pop();
        
        //合并成一棵树
        z=new linkedBinaryTree<int>;
        z->makeTree(0,*x.tree,*y.tree);
        w.weight=x.weight+y.weight;
        w.tree=x;
        heap.push(w);
        delete x.tree;
        delete y.tree;
    }
    
    return heap.top().tree;
}
```

## 第13章	竞赛树

竞赛树(tournament tree)

竞赛树是完全二叉树

存储效率最高

竞赛树的基本操作是替换最大(或最小)元素

当我们需要按指定的方式断开连接时，比如选择最先插入的元素，或选择左端元素(假定每个元素都有一个从左到右的名次)，这时，竞赛树就成为我们要选择的数据结构

### 13.1	赢者树和应用

假定有n个选手参加一次网球比赛。比赛规则是“突然死亡法(sudden-death mode):一名选手只要输掉一场球，就被淘汰。一对一选手比赛，最终只剩下一个选手保持不败，这个“幸存者”就是比赛赢者”

**赢者树(winner tree)**,**输者树(loser tree)**

竞赛树也称为**选择树(selection tree)**

定义13-1	赢者树 ：有n个选手的一棵赢者树是一棵完全二叉树，它有n个外部节点和n-1个内部节点，每个内部节点记录的是在该节点比赛的赢者

为了确定一场比赛的赢者树，我们假设每个选手都有一个分数，而且有一个规则用来比较两个选手的分数以确定赢者

在**最小赢者树(min winner tree)**中，分数小的选手获胜

在**最大赢者树(max winner tree)**中，分数大的选手获胜

插入排序，堆排序都是**内部排序法(internal sorting method)**

这些方法要求待排序的元素全部放入计算机内存

引入外部排序法：

1. 生成一些初始归并段(run),每一个初始归并段都是有序集
2. 将这些初始归并段合并为一个归并段



### 13.2	抽象数据类型winnertree

在定于抽象数据类型WinnerTree时，我们假设选手的个数是固定的。

也就是说，如果初始化时选手个数为n，那么初始化之后不能再增减选手

ADT	13-1	赢者树的抽象数据类型说明

```c++
抽象数据类型 WinnerTree
{
    实例
        完全二叉树，每一个节点指向比赛胜者；外部节点表示参赛者
    操作
        initiallize(a);为数组a的参赛者初始化胜者树
        winner();返回锦标赛胜者
        rePlay(i);在参赛者i改变之后重赛
}
```

程序13-1	抽象类WinnerTree

```c++
template<class T>
class winnerTree
{
    public:
    virtual ~winnerTree()	{}
    virtual void initialize(T *thePlayer,int theNumberOfPlayers)=0;
    //用数组thePlayer[1:numberOfPlayers]生成赢者树
    virtual int winner() const=0;
    //返回赢者的索引
    virtual void rePlay(int thePlayer)=0;
    //在参赛者thePlayer的分数变化后重赛
}
```

确定胜者的方式：重载操作符   ‘<=’

### 13.3	赢者树的实现

#### 13.3.2	赢者树的初始化

为了初始化一棵赢者树，我们从右孩子选手开始，进行他参加的比赛，而且逐层往上，只要是从右孩子上升到比赛节点，就可以进行在该节点的比赛。
为此，要从左往右得考察右孩子选手

#### 13.3.3	重新组织比赛

当选手thePlayer的值改变时，在从外部节点player[thePlayer]到根tree[1]的路径上，一部分或全部比赛都需要重赛。

简单起见，我们将该路径上的全部比赛进行重赛

#### 13.3.4	类completeWinnerTree

实现赢者树的数据结构时类completeWinnerTree

n是选手的个数

类的构造函数利用存储在数组中的选手进行赢者树的初始化

### 13.4	输者树

当一个赢者发生变化时，使用输者树可以简化重赛的过程。

但是，其他选手发生改变时，就不是那么回事了

### 13.5	应用

#### 13.5.1	用最先适配法求解箱子装载问题

1.问题描述

在箱子装载问题中，箱子的数量不限，每个箱子的容量为binCapacity,待装箱的物品有n个。物品i需要占用的箱子容量为objectSize[i],0<=objectSize[i]<=binCapacity

所谓**可行装载(feasible packing)**，是指所有物品都装入箱子而不溢出

所谓**最优装载(optimal packing)**,是指使用箱子最少的可行装载

2.近似算法

箱子装载问题和机器调度问题一样，是NP-复杂问题，因此常用近似算法求解

求解所得的箱子数不是最少，但接近最少

有4种流行的近似算法：

1. 最先适配法(Fist Fit,FF).物品按1,2,···,n的顺序装入箱子。假设箱子从左至右排列。每一物品i放入可装载它的最左面的箱子
2. 最优适配发(Best Fit, BF).令bin[j].unusedCapacity为箱子j的可用容量。初始时，所有箱子的可用容量为binCapacity。物品i放入可用容量unusedCapacity最小但不小于objectSize[i]的箱子
3. 最先适配递减法(Fist Fit Decreasing, FFD).此方法与FF类似，区别在于，所有物品首先按所需容量的递减次序排列，即对于1<=i<n,有objectSize[i]>=objectSize[i+1]
4. 最优适配递减法(Besf Fit Decreasing, BFD).此法与BF相似，区别在于，所有物品首先按所需容量的递减的次序排列，即对于1<=i<n,有objectSize[i]>=objectSize[i+1]

定理13-1	设I为箱子装载问题的任一实例，b(I)为最优装载所用的箱子数.FF和BF所用的箱子数不会超过(17/10)b(I)+2, 而FFD和BFD所用的像字数不会超过(11/9)b(I)+4

3.最先适配法和赢者树

赢者树可用来实现FF和FFD算法

因为最多用到n个箱子，所以从n个空箱子开始

初始化时，对于所有n个箱子,bin[j].unusedCapacity=binCapacity

接下来，用bin[j]作为选手，对最大赢者树进行初始化

4.最先适配法的C++实现

首先我们给类completeWinnerTree增加一个公有方法：

```
int winner(int i) const
	{return (i<numberOfPlayer)?tree[i]:0;}
```

这个方法的返回值时在内部节点i的比赛中获胜者

程序13-2的函数firstFitPack实现最先适配策略。

程序13-2	函数firstFitPack

```c++
void firstFitPack(int *objectSize,int numberOfObjects,int binCapacity)
{
    //输出箱子容量为binCapacity的最先适配装载
    //objectSize[1:numberOfObjects]是物品大小
    
    int n=numberOfObjects;	//物品数量
    
    //初始化n个箱子和赢者树
    binType *bin=new binType [n+1];	//箱子
    for(int i=1;i<=n;i++)
        bin[i].unusedCapacity=binCapacity;
    completeWinnerTree<binType> winTree(bin, n);
    
    //将物品装到箱子里
    for(int i=1;i<=n;i++)
    {
        //把物品i装入一个箱子
        //找到第一个有足够容量的箱子
        int child=2;	//从根的左孩子开始搜索
        while(child<n)
        {
            int winner=winTree.winner(child);
            if(bin[winner].unusedCapacity<objectSize[i])
                child++;	//第一个箱子在右子树
            child *=2;	//移到左孩子
        }
        
        int binToUse;	//设置为要使用的箱子
        child /=2;	//撤销向最后的左孩子的移动
        if(child<n)
        {
            //在一个树节点
            binToUse=winTree.winner(child);
            //若binToUse是右孩子，则要检查箱子binToUse-1
            //计时binToUse是左孩子，检查箱子binToUse-1也不会有问题
            if(binToUse>1 && bin[binToUse-1].unusedCapacity>=objectSize[i])
                binToUse--;
        }
        else	//当n是奇数
            binToUse=winTree.winner(child/2);
        
        cout<<"Pack object"<<i<<" in bin"
            <<binToUse<<endl;
        bin[binToUse].unusedCapacity-=objectSize[i];
        winTree.rePlay(binToUse);
    }
}
```

赢者树是用数组表示的完全二叉树，它能够按照数组下标乘2或加1的方式从下向上移动

这种向下移动方式没有实现类的一个目标——信息隐藏

我们希望类的实现细节对用户是不可见的，这样的话，我们就可以在不改变类的公有成员的情况下修改类的具体实现细节，而且修改后的结果不会影响应用代码的正确性。

利用信息隐藏的好处，我们可以在类completeWinnerTree中增加方法，使得用户可以从一个内部节点移动到它的左孩子和右孩子，然后在函数firstFitPack中应用这些方法

#### 13.5.2	用相邻适配发求解箱子装载问题

1.何谓相邻适配法

相邻适配法是一种箱子装载策略：为装载一件物品，首先在非空的箱子中循环搜索能够装载该物品的箱子，如果找不到这样的箱子，就启用一个空箱子。开始时，没有非空的箱子，因此，为装载物品1，启用箱子1.

假设箱子1~箱子b已经装有物品，我们把这些箱子想象成一个环：当i!=b时，i的下一个箱子为i+1；当i=b时，i的下一个箱子为箱子1.

要装载当前的一件物品，假设上一件物品已经装入箱子。

我们从箱子开始循环查找非空的箱子，如果找到合适的箱子，就把物品放入这个箱子，如果回到箱子j还没有找到合适的箱子，则启用一空箱子，并把物品放入这个空箱子

相邻适配法略于另一种同名的动态存储分配策略类似，如果和箱子装载联系起来，这时另一种相邻适配策略：每次装载一个物品，若一个物品不能装入当前箱子，则将当前箱子关闭并启用一个新的箱子

2.相邻适配法和赢者树

可用最大赢者树来高效地实现相邻适配策略

若上一件物品被放入箱子lastBinUsed中且当前已使用了b个箱子，则按如下两个步骤来搜索下一个可用的箱子：

1）在编号大于lastBinUsed的箱子中，找到第一个合适的箱子j。当箱子总数为n时，这样的j总存在。若该箱子非空(即j<=b),则将物品放入该箱子

2)若步骤1未找到一个非空的箱子，则在树中搜索适合该物品的最左边的箱子，然后将物品放入该箱子

为了实现步骤1，我们从箱子j=lastBinUsed+1开始。若lastBinUsed=n,则所有n个物品都已装箱，并且用了n个箱子，每件物品一个箱子。因此j<=n.

图13-8	步骤1的伪码

从箱子j开始查找合适的箱子的过程。沿着从箱子j到根的路径进行遍历，依次查看右子树，直到找到第一颗包含这种箱子的右子树为止，这时，在该子树中，能够容纳物品的最左面的箱子就是索要查找的箱子

```
//寻找离lastBinUsed右面最近的适合物品i的箱子
j=lastBinUsed+1;
if(bin[j].unusedCapacity>=objectSize[i])
	return j;
if(bin[j+1].unusedCapacity>=objectSize[i])
	return j+1;
	
	p=bin[i]的双亲
	if(p==n-1)
	{
		//特殊情况
		令q等于tree[p]右端的外部节点
		if(bin[q].unusedCapacity>=objectSize[i])
		return q;
	}
	
	//向根方向移动，寻找第一个右子树，它有一个容量足够的箱子。p的右面的子树是p+1
	p/=2;	//移动到父节点
	while(bin[tree[p+1]].unusedCapacity<objectSize[i])
	p/=2;
	
	return 在子树p+1中适合物品i的第一个箱子
```

## 第14章	搜索树

本章开发的树形结构适合于字典描述

使用索引二叉搜索树，你可以按关键字和按名次进行字典操作

### 14.1	定义

#### 14.1.1	二叉搜索树

如果给dictionary增加以下操作，那么散列不再具有较好的平均性能：

1. 按关键字的升序输出字典元素
2. 按升序找到第k个元素
3. 删除第k个元素



定义14-1	**二叉搜索树(binary search tree)**是一棵二叉树，可能为空；一棵非空的二叉搜索树满足以下特征:

1. 每个元素有一个关键字，并且任意两个域㢝的关键字都不同；因此，所有的关键字都是唯一的；
2. 在根节点的左子树中，元素的关键字(如果有的话)都小于根节点的关键字
3. 在根节点的右子树中，元素的关键字(如果有的话)都大于根节点的关键字
4. 根节点的左，右子树也都是二叉搜索树





**有重复值的二叉搜索树(binary search tree with duplicates)**:元素的关键字可以相同，小于和大于相应变为 小于等于 大于等于

#### 14.1.2	索引二叉搜索树

**索引二叉搜索树(indexed binary search tree)**在每个节点中添加一个leftSize域

这个域的值是该节点左子树的元素个数

### 14.2	抽象数据类型

ADT	14-1	二叉搜索树的抽象数据类型描述

```
抽象数据类型 bsTree
{
	实例
		二叉树，每一个节点都有一个数对，其中一个成员是关键字，另一个成员是数值；所有关键字都不相同；
		任何一个节点的左子树的关键字小于该节点的关键字；右子树的关键字大于该节点的关键字
	操作
		find(k);返回关键字为k的数对
		inseert(p);插入数对p
		erase(k);删除关键字为k的数对
		asecend();按关键字升序输出所有数对
}
```

ADT	14-2	索引二叉搜索树的抽象数据类型描述

```
抽象数据类型 IndexedBSTree
{
	实例
		与bsTree相同，只是每一个节点还有一个leftSize域
	操作
		find(k);返回关键字为k的数对
		get(index);返回第index个数对
		inseert(p);插入数对p
		erase(k);删除关键字为k的数对
		asecend();按关键字升序输出所有数对
}
```

程序14-1	C++抽象类bsTree

```c++
template<class K,class E>
    class bsTree:public dictionary<K,E>
    {
        public:
        virtual void ascend()=0;
        				//按关键字升序输出
    };
```



程序14-2	C++抽象类indexedBSTree

```c++
template<class K,class E>
    class indexedBSTree:public bsTree<K,E>
    {
        public:
        virtual pair<const K,E>* get(int) const=0;
        						//根据给定的索引，返回其数对的指针
        virtual void delete(int)=0;	
        						//根据给定的索引，删除其数对
    }
```

### 14.3	二叉搜索树的操作和实现

#### 14.3.1	类binarySearchTree













#### 14.3.4	删除

程序14-6	二叉搜索树的删除

```c++
template<class T>
void binarySearchTree<K,E>::erase(const K& theKey)
{
    //删除其关键字等于theKey的数对
    
    //查找关键字为theKey的节点
    binaryTreeNode<pair<const K,E> > *p=root,
    *pp=NULL;
    while(p!=NULL && p->element.first !=theKey)
    {
        //p移到它的一个孩子节点
        pp=p;
        if(theKey<p->element.first)
            p=p->leftChild;
        else
            p=p->rightChild;
    }
    
    if(p==NULL)
        return ;	//不存在与关键字theKey匹配的数对
    
    //重新组织树结构
    //当p有两个孩子时的处理
    if(p->leftChild!= NULL && p->rightChild!=NULL)
    {
        //两个孩子
        //转化为空或只有一个孩子
        //在p的左子树中寻找最大元素
        binaryTreeNode<pair<const K,E> > *s=p->leftChild,
        *ps=p;	//s的双亲
        while(s->rightChild!=NULL)
        {
            //移到最大的元素
            ps=s;
            s=s->rightChild;
        }
        
        //将最大元素s移到p，但不是简单的移动
        //p-<element=s->element,因为key是常量
        binaryTreeNode<pair<const K,E> > *q=new binaryTreeNode<pair<const K,E> >(s->element,p->leftChild,p->rightChild);
        if(pp==NULL)
            root=q;
        else if(p==pp->leftChild)
            pp->leftChild=q;
        else
            pp->rightChild=q;
        if(ps==p) pp=q;
        else pp=ps;
        delete p;
        p=s;
    }
    
    //p最多有一个孩子
    //把孩子指针放在c
    binaryTreeNode<pair<const K,E> *c;
    if(p->leftChild!=NULL)
        c=p->leftChild;
    else
		c=p->rightChild;
    
    //删除p
    if(p==root)
        root=c;
    else
    {
        //p是pp的左孩子还是右孩子?
        if(p==pp->leftChild)
            pp->leftChild=c;
        else
            pp->rightChild=c;
    }
    treeSize--;
    delete p;
}
```

### 14.4	带有相同关键字元素的二叉搜索树

当一棵二叉搜索树可以具有两个或多个关键字相同的元素时，相应的类称为dBinarySearchTree.

要实现这个类，只需修改程序14-5	的函数binarySearchTree<K,E>::insert,即把while循环语句

if(thePair.first<p->element.first)

改为

if(thePair.first<=p->element.first)

程序14-7	修改程序14-5的while循环以允许相同关键字

```c++
while(p!=NULL)
{
	//检查元素p->element
    pp=p;
    //p移到它的一个孩子节点
    if(thePair.first<p->element.first)
        p=p->leftChild;
    else
        p=p->rightChild;
}
```

### 14.5	索引二叉搜索树

类linkedBinarySearchTree可以定义为类linkeedBinaryTree的派生类

对于类linkedBinarySearchTree，一个节点的数值域是三元的:leftSize,key,value

我们实施带有索引的搜索方法与搜索偶对(一个偶对被定义为关键字域key和值域value)的方法是类似的

### 14.6	应用

#### 14.6.1	直方图

1.何为直方图

在直方图问题中，输入由n个关键字所构成的集合，然后输出一个列表，它包含不同关键字及其每个关键字在集合中出现的次数(频率)

直方图一般是用来确定数据分布的

2.简单直方图程序

当关键字的值是从0到r范围内的整数，且r的值足够小时，直方图可以在线性时间里，用一个很简单那的过程来计算。

程序14-8	简单的直方图程序

```c++
void main(void )
{
    //非负整数值的直方图
    int n,		//元素个数
    	r;		//0至r之间的值
    cout<<"Enter number of elements and range"
        <<endl;
    cin>>n>>r;
    
    //生成直方图数组h
    int *h=new int[r+1];
    
    //将数组h初始化为0
    for(ijnt i=0;i<=n;i++)
        h[i]=0;
    
    
    //输入数据，然后计算直方图
  	for(int i=1;i<=n;i++)
    {
        //假设输入的值在0至人之间
        int key;	//输入值
        cout<<"Enter element"<<i<<endl;
        cin>>key;
        h[key]++;
    }
    
    //输出直方图
    cout<<"Distinct elements and frequencies are0"
        <<endl;
    for(int i=0;i<=r;i++)
        if (h[i ]!=0)
            cout<<i<<" "<<h[i]<<endl;
} 
```





3.直方图与二叉搜索树

当关键字类型不是整型(如关键字是实数)而且关键字范围很大时，上边的程序就不合适了

4.类binarySearchTreeWithVisit

为了求解实施直方图问题的二叉搜索树方法，我们首先定义类binarySearchTreeWitVisit，它是类binarySearchTree的拓展，其中增加了以下公有成员函数：

```
void insert(const pair<const K,E>& thePair,void (*visit) (E&))
```

该函数实现插入元素thePair的操作，前提是，树中不存在关键字等于thePair.first的元素

若存在一个关键字等于thePair.first的元素p，则调用函数visit(p.second)

5.重新考虑直方图与二叉搜索树

程序14-9	

```c++
int main(void )
{
    //使用搜索树的直方图
    int n;	//元素个数
    cout<<"Enter number of elements"<<endl;
    cin>>n;
    
    //输入元素，然后插入树
    binarySearchTreeWithVisit<int ,int> theTree;
    {
        pair<int ,int> thePair;	//输入元素
        cout<<"Enter element"<<i<<endl;
        cin>>athePair.first;	//关键字	
        thePair.second=1;		//频率
        //将thePair插入树，除非存在与之匹配的元素
        //在后一种情况下，count值增1
        theTree.insert(thePair,add1);
    }
    
    //输出不同的关键字和它们的频率
    cout<<"Distince elements and frequencies are"
        <<endl;
    theTree.ascend();
}
```

#### 14.6.2	箱子装载问题的最优匹配法

1.使用带有重复关键字的二叉搜索树

在实现最优匹配法时， 搜索树的每个元素代表一个正在使用且剩余容量不为0的箱子

2.c++实现

需要扩充类的定义，增加公有成员函数findGE(thKey)

返回值为剩余容量既大于等于thKey又是最小的箱子

程序 14-10	查找大于等于theKey的最小关键字

```c++
template<class K,class E>
pair<const K ,E>* dBinarySearchTreeWithGE<K,E>::findGE(const K& theKey) const
{
    //返回一个元素的指针，这个元素的关键字是不小于theKey的最小关键字
     //如果这样的元素不存在，返回NULL
    binaryTreeNode<pair<const K,E> > *currentNode=root;
    pair<const K,E> *bestElement=NULL;	//目前找到的元素，其关键字是不小于
    //theKey的最小关键字
    
    //对树搜索
    while(currentNode!=NULL)
        //currentNode->element是一个候选者吗
        if(currentNode->element.first>=theKey)
        {
            //是,currentNode->element是比bestElement更好的候选者
            bestElement=&currentNode->element;
            //左子树中唯一较小的关键字
            currentNode=currentNode->leftChild;
        }
    else
        //不是，currentNode->element.first 大小
        //检查右子树
        currentNode=currentNode->rightChild;
    
    return bestElement;
}
```

程序14-11	箱子装载问题的最优匹配法

```c++
void bestFitPack(int *objectSize,int numberOfObjects,int binCapacity)
{
    //输出容量为binCapacity的最优箱子匹配
    //objectSize[1:numberOfObjetcts]是物品大小
    int n=numberOfObjects;
    int binsUsed=0;
    dBinarySearchTreeWithGE<int ,int> theTree;	//箱子容量树
    pair<int ,int > theBin;
    
    //将物品逐个装箱
    for(int i=1;i<=n;i++)
    {
        //将物品i装箱
        //寻找最匹配的箱子
        pair<const int ,int> *bestBin=theTree.findGE(objectSize[i]);
        if(bestBin==NULL)
        {
            //没有足够大的箱子，启用一个新箱子
            theBin.first=binCapacity;
            theBin.second=++binsUsed;
        }
        else
        {
            //从树theTree中删除最匹配的箱子
            theBin=*bestBin;
            theTree.erase(bestBin->first);
        }
        
        cout<<"Pack object"<<i<<in Bin
			<<theBin.second<<endl;
        
        //将箱子插到树中，除非箱子已满
        theBin.first==objectSize[i];
        if(theBin.first>0)
            theTree.insert(theBin);
    }
}
```

#### 14.6.3	交叉分布

1.通道布线与交叉

在交叉分布问题中，从一个布线通道开始，在通道的顶部和底部各有n个针脚

2.分布交叉

使得上半部分和下半部分的交叉数目大抵相同

3.使用线性表实现的分布交叉

程序14-12	使用线性表的交叉分布程序

```c++
void main(void)
{
    //定义要解决的问题实例
    //在通道底部的连接点theC[1:10]
    int theC[]={0,8,7,4,2,5,1,9,3,10,6};
    //交叉数量，k[1:10]
    int k[]={0,7,6,3,1,2,0,2,0,1,0};
    int n=10;	//在通道每一边的针脚数量 
    int theK=22;	//交叉总数量
    
    //生成数据结构
    arrayList<int> theList(n);
    int *theA=new int[n+1];	//顶端的排列
    int *theB=new int[n+1];	//底端的排列
    int *theC=new int[n+1];	//中间的排列
    
    int crossingsNeeded=theK/2;	//需要在上半部分保留的交叉数
    
    //从右到左扫描线路
    int currentWire=n;
    while(crossingsNeeded>0)
    {
        //在上半部需要更多的交叉
        if(k[currentWire]<crossingsNeeded)
        {
            //使用来自currentWire的所有交叉
            theList.insert(k[currentWire],currentWire);
            crossingsNeeded-=k[currentWire];
        }
        else
        {
            //仅使用来自currentWire的crossingsNeeded
            theList.insert(crossingsNeeded,currentWire);
            crossingsNeeded=0;
        }
        currentWire--;
    }
    
    //确定中间的线路排序
    //第一个线路currentWire次数相同
    for(int i=1;i<=currentWire;i++)
        theX[i]=theList.get(i-currentWire-1);
    
    //计算上半部的排列
    for(int i=1;i<=n;i++)
        theA[theX[i]]=i;
	/
    计算下半部的排列
        for(int i=1;i<=n;i++)
            theB[i]=theC[theX[i]];
    
    cout<<"A is ";
    for(int i=1;i<=n;i++)
        cout<<theA[i]<<" ";
    cout<<endl;
    
    cout<<"B is";
    for(int i=1;i<=n;i++)
        cout<<theB[i]<<" ";
    cout<<endl;
}
```

## 第15章	平衡搜索树

AVL和红-黑树->平衡树结构

B-树 ->树结构， 它的度大于2

AVL和红-黑 树适合内部存储应用

B- 树适合外部存储的应用(例如，存储在磁盘上的大型词典)

分裂树是一种数据结构

STL类map  / multimap使用的是红-黑 树结构，以保证查找，插入和删除操作具有对数级的时间性能

如果我们是按关键字实施字典操作，而且操作时间不能超过指定范围，这时我们提倡使用平衡搜索树 

AVL和红-黑 树都使用“旋转”来保持平衡

AVL树对每个插入操作最多需要一次旋转

### 15.1	AVL树

#### 15.1.1	定义

平衡树(balanced tree)：最坏情况下的高度为O(log~n)的树

AVL树就是一种平衡树

定义15-1	一棵空的二叉树是AVL树；如果T是一棵非空的二叉树，TL和TR分别是其左子树和右子树，那么当T满足了以下条件，T是一棵AVL树:

1）TL和TR是AVL树

2）| hL-hR|<=1,hL和hR是TL和TR的高





一棵AVL搜索树既是二叉搜索树，也是AVL树。



一棵索引AVL搜索树既是索引二叉搜索树，也是AVL树。



如果用AVL搜索树来描述字典，并在对数级时间内完成每一种字典操作，那么，我们必须确定AVL树的以下特征：

1）一棵n个元素的AVL树，其高度是O(log~n)

2)对于每一个n，n>=0,都存在一棵AVL树

3)对一棵n元素的AVL搜索树，在O(高度)=O(log~n)的时间内可以实现查找

4）将一个新元素插入一棵n元素的AVL搜索树中，可以得到一棵n+1个元素的AVL树，而且插入用时为O(log~n)

5)一个元素从一棵n元素的AVL搜索树中删除，可以得到一棵n-1个元素的AVL树，而且删除用时为O(log~n)

#### 15.1.2	AVL树的高度

对一棵高度为h的AVL树，令N~h是其最少的节点数

在最坏情况下：根的一棵子树的高度是h-1，另一棵子树的高度是h-2，而且两棵子树都是AVL树

#### 15.1.3	AVL树的描述

AVL树一般用链表描述

为简化插入和删除操作，我们为每个节点增加一个平衡因子bf

节点x的平衡因子bf(x)定义为：

x的左子树高度-x的右子树高度

#### 15.1.4	AVL搜索树的搜索

程序14-4

#### 15.1.5	AVL搜索树的插入

节点A的不平衡情况有两类：L型不平衡(新插入节点在A的左子树中)和R型不平衡

A的不平衡类型的细分是：LL(新插入节点在A节点的左子树的左子树中)，LR(新插入节点在A节点的左子树的右子树中)，RR RL

矫正LL和RR型不平衡所做的转换称为**单旋转(single rotation)**

矫正LR 和RL型不平衡所做的转换称为**双旋转(double rotation****)**

#### 15.1.6	AVL搜索树的删除



### 15.2	红-黑 树



#### 15.2.1	基本概念

 **红-黑 树(red-black tree)**是这样的一棵二叉搜索树：树中每一个节点的颜色或者是黑色或者是红色

红-黑树的其他特征可以用相应的扩充二叉树来说明

在一棵规则二叉树中，每一个空指针用一个外部节点来替代，由此得到一棵扩展二叉树

其他的性质：

RB1:根节点和所有外部节点都是黑色

RB2：在根至外部节点路径上，没有连续两个节点是红色

RB3：在所有根至外部节点的路径上，黑色节点的数目都相同

红-黑 树还有另一种等价，它取决于父子节点间的指针颜色。从父节点指向黑色孩子的指针是黑色的，从父节点指向红色孩子的指针是红色的，另外还有：

RB1‘：从内部节点指向外部节点的指针是黑色的

RB2’：在根至外部节点路径上，没有两个连续的红色指针
RB3‘：在所有根至外部节点路径上，黑色指针的数目都相同

如果直到指针的颜色，就能推断节点的颜色，反之亦然

令 红=黑 树的一个节点的**阶(rank)**,是从该节点到一外部节点的路径上黑色指针的数量

因此，外部节点的阶是0

定理15-1	设从根到外部节点的路径长度(length)是该路径上的指针数量

如果P和Q是红-黑 树中的两条从根至外部节点的路径，那么length(P)<=2length(Q)

定理15-2	令h是一棵红-黑 树的高度(不包括外部节点)，n是树的内部节点数量，而r是根节点的阶，则

1) h<=2r
2) n>=2^r-1
3) h<=2log~2  (n+1)





#### 15.2.2	红-黑 树的描述

在红-黑 树的定义中包括了外部节点，但在执行过程中，我们对外部节点仍然用空指针来描述，而不是用物理节点来描述

#### 15.2.3	红-黑 树的搜索

可以使用普通二叉搜索树的搜索代码(程序14-4)来搜索红-黑 树

#### 15.2.4	红-黑 树的插入

红-黑 树的插入操作使用的是程序14-5所示的普通二叉树的插入算法

对插入的新元素，需要上色

如果擦汗如前的树是空的，那么新节点就是根节点，颜色应为黑色(参看特征RB1)

假设插入前的树是非空的，如果新节点的颜色是黑色，那么在从根到外部节点的路径上，将有一个特殊的黑色节点作为新节点的孩子

如果新节点是红色，那么可能出现两个连续的红色节点

把新节点赋为黑色肯定不符合RB3，而把新节点赋为红色则可能违反，也可能符合RB2，因此，应将新节点赋为红色

