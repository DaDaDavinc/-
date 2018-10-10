<!-- TOC -->

- [extern](#extern)
- [static](#static)
- [volatile](#volatile)
- [const](#const)
- [new & malloc](#new--malloc)
- [多态性与虚函数表](#多态性与虚函数表)
- [指针运算](#指针运算)
- [指针和引用](#指针和引用)
- [指针和数组](#指针和数组)
- [智能指针](#智能指针)
- [C++四种类型转换](#c四种类型转换)
- [内存对齐](#内存对齐)
- [inline 内联函数](#inline-内联函数)
- [C++内存管理](#c内存管理)
- [C++面向对象三大特征](#c面向对象三大特征)
- [STL](#stl)
    - [STL内存池实现](#stl内存池实现)
    - [红黑树](#红黑树)
- [常见的内存错误及策略](#常见的内存错误及策略)
- [指针、数组、字符串](#指针数组字符串)
    - [常量字符串](#常量字符串)
    - [计算内存容量](#计算内存容量)
    - [指针参数传递内存](#指针参数传递内存)
    - [杜绝野指针](#杜绝野指针)
- [有了malloc()/free()为何还要new/delete](#有了mallocfree为何还要newdelete)
- [new/delete使用要点](#newdelete使用要点)
- [哪些库函数属于高危函数，为什么？](#哪些库函数属于高危函数为什么)
    - [字符串操作系列](#字符串操作系列)
    - [标准输入输出流系列](#标准输入输出流系列)

<!-- /TOC -->
### extern 
可以用来修饰变量和函数，标定变量或函数在别的文件中定义了，提示编译器遇到时到其他模块中查找，一定程度上可以代替头文件的作用，区别是extern全局变量只需要在定义处占用一份内存，而头文件需要在每个包含它的文件中占有内存。
* 用来链接指定，`extern "C" void fun()` ,指定编译器按照 `C` 的规则去编译函数
* 声明函数或全局变量的作用范围，是声明而不是定义 
* 告知编译器变量存储在静态存储区而不是栈区
```C++
char str[6];     //定义
extern char a[]; //声明
//错误示范
//指针变量不同于字符数组，会造成内存非法访问 
extern char *a; 
```

### static
**面向过程**
* 静态全局变量

限定变量在一个编译单元内，一个编译单元指一个源文件和它的头文件，因为编译过程会连涉及多个源文件，声明静态变量可以限制作用范围，实现数据隐藏，防止冲突
* 静态局部变量

变量作用域被限定在函数内或模块内，声明时分配内存，函数运行结束后释放内存
* 静态函数

函数作用域限定在模块内或本文件内

**面向对象**  
static在C++中，可以实现所有对象维护同一个实例，实现了资源共享。
* 静态数据成员

对所有类的对象是共有的(public)，因此只需分配一块内存，可以节省空间。另，更新静态数据成员，对所有的对象都会更新，可以提高效率。

* 类成员函数

**when**

需要一个数据对象为整个类而非某个对象服务,同时又力求不破坏类的封装性,即要求此成员隐藏在类的内部，对外不可见。

**static内部机制**

因为函数被调用时，静态数据成员就要存在了。所以static数据成员必须在函数外部定义和初始化，而不能在函数内分配内存和初始化。
排除函数内部，静态数据成员的内存分配有三个可能的空间：
* 作为类外部接口的头文件，头文件中定义类
* 类定义的内部实现，在成员函数中声明（注意是声明）
* main函数外部的全局静态变量

因为static需要实际分配内存，所以不能在类内部定义；也不能在头文件中的类外部定义，会造成重复定义。

**注意**

* 类的静态成员函数属于类，不属于对象，故没有`this` 指针，只能访问类的静态成员函数和静态数据成员，无法访问对象的。
* 静态成员函数不能定义为`virtual`函数
* 必须初始化

### volatile
 
**有什么作用**

* 易变性

告诉编译器该变量是易变的，遇到时要从内存去访问，而不能从缓存、寄存器等读取. (因为访问寄存器的速度要比访问内存快，所以系统会优先访问寄存器。但由于其易变性，有可能在内存中已经发生改变，但寄存器未同步)

* 顺序性

两个包含volatile属性的变量，在编译后不能乱序（运行时可能乱序，这点需要由它的机制决定，例如内存屏障memory-barriers）

* 一个参数可以同时是const volatile吗？

可以。举个例子：`只读状态存储器`。其本身是volatile属性，是const告诉程序不能去改变它。

### const

**const的作用**

* 修饰变量，表示常变量
* 修饰指针，`const int * `(const int)型指针，指针指向的地址不变
* 修饰指针指向的对象 `int *cosnt` （int型指针，const对象）
* 修饰引用作为形参 `fun(const &a)`
* 修饰成员变量，必须在构造函数中初始化
* 修饰成员函数，说明该成员函数不能修改静态成员，但不是很可靠
### new & malloc
new|malloc
--|--
C++|C
按数据类型分配内存|按大小分配
调用默认构造函数|无法调用
new是操作符可以reload|malloc是库函数则不可以
delete销毁，调用析构函数|free
new失败抛出bad_malloc异常|malloc返回NULL
采用try...catch..|判断返回值是否NULL
new[] delete[] | sizeof(int)* n

new和new[]区别：new[]一次性分配所有内存，多次调用构造函数

### 多态性与虚函数表
静态多态|动态多态
--|--
重载、模板|虚函数、继承(派生)
编译时确定|运行时确定

**动态多态实现的条件**

* 虚函数(虚表、虚指针)
* 一个基类的指针或引用指向派生类的对象(继承)

C++内部为每一个类维持一个`虚函数表`， 该类的每个对象首地址都保存着该虚函数表的指针, 同一个类不同对象指向同一个表。

**虚函数作用**
* 用于实现多态
* 在设计上有抽象和封装的作用，比如抽象工厂模式
* 动态绑定就是结合虚函数表实现的

**纯虚函数如何定义**
```C++
virtual myclass();        //虚函数
virtual ~myclass() = 0;   //纯虚析构函数，=0
```
* 为什么对于存在虚函数的类，其析构函数要定义为虚函数

在销毁对象时，如果析构函数为非虚函数，有可能只销毁部分数据；如果要调用对象本身的析构函数，就必须在基类中将析构函数定义为虚函数。

* 析构函数能抛出异常吗？

肯定不能。析构函数作用是销毁对象，释放内存。而对象在运行期间出现了异常时，C++异常处理模型会调用析构函数清除由于异常而失效的对象，并释放对象原来分配的资源。从这个意义上看。析构函数本身就是异常处理的一部分。

* 如果析构函数也会抛出异常

则异常点之后的程序不会继续执行；如果恰好异常点后面是关键的动作比如释放资源，那么会造成`内存泄露`。 另外，旧的异常未解决，又出现了新的异常，可能导致`程序崩溃`

* 举个栗子说明虚函数的作用(多态)
```C++
//基类
class Base {
    int m_tag;
public:
    Base(int tag):m_tag(tag) {}  //:构造函数 m_tag=tag;
    
    void print() {
        cout << "Base::print() called" << endl;
    }
    //虚函数
    virtual void vprint() {
        cout << "Base::vPrint() called" << endl;
    }
    virtual void printTag() {
        cout << "Base::m_tag of this instance is: " << m_tag << endl;
    }
}
//派生类
class Derived : public Base {
public:
    Derived(int tag) : Basa(tag) {}
    //重定义非虚函数print()
    void print() {   
        cout << "Derived1::print() called" << endl;
    }
    //重定义虚函数vprint()
    virtual void vPrint() {
        cout << "Derived::vPrint() called" << endl;
    }
}

//调用类
main() {
    Derived* foo = new Derived(1);  
    Base* bar = foo;  //父类指针指向子类对象

    foo->print();   //Derived1::print() called
    foo->vprint();  //Derived::vPrint() called

    bar->print();   //Base::print() called，还是指向父类
    bar->vprint();  //Derived::vPrint() called
}
```
可以看出，如果在父类中函数没有定义为virtual, 那么当父类指针指向子类对象时，实际指向父类本身的成员函数；而定义为虚函数时，在子类对象中的成员函数将会在虚表中再分配一块内存。(简单的说，如果不是虚函数，那么父类指针就相当于this指针，不能指向子类对象)
* 虚函数表  
因为有虚函数，编译器会创建虚表来放他们的地址，所以也有虚指针指向它们

虚表1|虚表2
--|--
Base::vprint()的地址|Derived::vprint()
Base::printTag()地址|-------------------
* 由于派生类Derived中，虚函数vprint()是重写的，printTag()是继承的，所以父类指针指向子类对象时，vprint()地址是虚表2的地址，printTag()地址是虚表1的地址。
### 指针运算
* 自增、自减

**p++/++p、p--/--p、p+=i、p-=i**
```C
#include <stdio.h>

int main() {
 int arrInt[] = {100, 200, 300, 400, 500};
 int *p = arrInt;
 // *、++ 优先级相同
 printf("%d\n", *p++);     // 100, p++
 printf("*p = %d\n", *p);  // 200
 printf("%d\n", *++p);     // ++p, 300
 printf("*p = %d\n", *p);  // 300
 printf("%d\n", ++*p);     // 301
 printf("*p = %d\n", *p);  // 301
 return 0;
}
```
* 两个指针可以相减(不能相加)

**前提**:两个指针指向同一个数组array[]

![](http://c.biancheng.net/cpp/uploads/allimg/140512/1-140512222R0459.png)
```C++
int arrInt[5] = {1, 2, 3, 4, 5};
int *pl = arrInt[1], 
    *p2 = arrInt[4];
// 指针相减 p1 < p2
int *p3 = p2 - p1;     // p2 - p1 = arrInt[4-1] = arrInt[3] = 4
std::cout << "*p3 = " << arrInt[3] << std::endl;
```

### 指针和引用
指针|引用
--|--
保存对象的地址|相当于对象的别名
需要解地址间接访问|直接访问
可以改变所指地址而改变对象|从一而终
不需要初始化|必须初始化

### 指针和数组
指针|数组
--|--
int *const p;|int p[];
int (*const p)[];|int p[][];
数组名做参数时(即使用其数组名)会退化为指针，sizeof()除外,即
```C++
char string[10] = "123456789";
std::cout << sizeof(string) << endl;  //10
```

### 智能指针
* 智能指针

是一个类，类的构造函数中传入一个普通指针，析构函数中释放传入的指针。智能指针的类都是栈上的对象，所以当函数（或程序）结束时会`自动被释放`，即能够对内存进行自动管理
* 普通指针

没有自动内存回收机制，当多个指针指向同一个对象时，如果其中一个指针delete了对象，就会造成`指针悬垂`。另外，忘记释放指针内存，会导致`内存泄露`
* 常用的智能指针

`auto_ptr`  
不支持复制（拷贝构造）和赋值（operator=），但编译不报错；因为不能复制，所以无法放入容器中  
`unique_ptr`  
也不支持复制和赋值，但比auto_ptr好，直接赋值会编译出错。  
**`share_ptr`**  
基于引用计数counter，可以随意赋值，内存引用计数为0时内存才会被释放    
`weak_ptr`  
弱引用，只引用，不计数。因为引用计数有一个问题：互相引用形成环，比如A><B，这样导致两个指针指向对象的内存，无法释放，所以有了weak_ptr，同时产生新问题：若share_ptr，weak_ptr指向同一块内存，引用计数为0时，内存就会被释放。故weak_ptr指向的内存不一定是有效的，所以需要先判断
```C++
if (weak_pt != nullptr)
```
* 智能指针的实现
```C++
template <typedef T>
class SmartPointer {
    T* _ptr;
    size_t _reference_count;  //new申请size_t类型大小的内存
    void releaseCount() {     //清零计数器，释放内存
        if (_ptr) {
            (*_reference_count)--;
            if ((*_reference_count) == 0){
                delete _ptr;
                delete _reference_count;
            }
        }
    }
public:
    //构造函数
    SmartPointer(T* p = 0) : _ptr(p), _reference_count(new size_t) {
        if ( p ) 
            *_reference_count = 1;
        else 
            *_reference_count = 0;
    } 
    //拷贝构造函数
    SmartPointer(const SmartPointer& src) {
        if (this != &src) {
            _ptr = src._ptr;  //拷贝
            _reference_count = src._reference_count;
            ++(*_reference_count);  //拷贝一次，计数加一
        }
    }
    //重载赋值操作符 = 
    SmartPointer& operator= (const SmartPointer& src) {
        //源对象指针跟当前指针指向同一个内存，不重复赋值
        if (_ptr == src._ptr) 
            return *this;

        releaseCount();   //清零计数器
        _ptr = src._ptr;
        _reference_count = src._reference_count;
        ++(*_reference_count);
    }
    //重载操作符 *
    T& operator*() {
        if (_ptr)
            return *_ptr;
    }
    //重载操作符 ->
    T* operator->() {
        if (_ptr)
            return _ptr;
    }
    //析构函数
    ~SmartPointer() {
        if (--(*_reference_count) == 0) {
            delete _ptr;
            delete _reference_count;
        }
    }
}

int main(int argc, char* argv[]) {
    SmartPointer<char> cp1(new char('a'));  //定义一个share_ptr并初始化
    SmartPointer<char> cp2(cp1);            //拷贝构造
    SmartPointer<char> cp3;
    cp3 = cp2;     //赋值
    cp3 = cp1;     
    cp3 = cp3;
    SmartPointer<char> cp4(new char('b'));
    cp3 = cp4;
}
```
通过智能指针的实现细节，可以发现：
* 构造函数中_reference_counter初始化为1
* 拷贝构造函数中_reference_counter加1
* 赋值运算符中，A=B，则A对象减少，B对象增加，所以左边的对象_reference_counter--, 右边的对象 _reference_counter++
* 析构函数中引用计数_reference_counter--
* share_ptr<T> / weak_ptr<T>区别：share_ptr循环引用，会导致内存泄露；后者不影响引用计数，不会导致内存泄露

### C++四种类型转换
key word|statement
--|--
static_cast|各种隐式转换，非const转const，void*转指针等，风险低
const_cast|const转非const，volatile转非volatile
reinterpret_cast|最灵活的转换，针对二进制位的转换，如将int转为指针，但是高度危险
dynamic_cast|借助RTTI，用于类型安全的向下转换，用于含有虚函数的类(动态多态)

### 内存对齐
* 数据成员对齐规则 
1. 结构(struct)或联合(union)，第一个数据成员放在offset为0的地方
2. 变量存储的起始位置总是该变量大小的整数倍
3. 每个数据成员的对齐按照#pragma pack(n)指定的和该变量自身长度中，较小的那个进行
* 结构体或联合对齐规则
1. 按照#pragma pack(n)指定的和联合中最大的变量之间，较小的那个进行

小端模式|大端模式
--|--
Intel 80x86 结构|  PowerPC / KEIL C51 / ARM
高位对高地址，低位对低地址|高位低地址，低位高地址
```C++
//高地址
　　buf[3] (0x78) //-- 低位
　　buf[2] (0x56)
　　buf[1] (0x34)
　　buf[0] (0x12) //-- 高位
//低地址
```
`0x12345678` (化为二进制共32位4个字节)存放方式
内存地址|小端|大端
--|--|--
0x4000|0x78|0x12
0x4001|0x56|0x34
0x4002|0x34|0x56
0x4003|0x12|0x78
* 如何确定union内存情况？
```C++
union People {
    int age;          //4
    char name;        //1
};                    //分配8字节内存
/*  假设“小端”
 *  int age = 1; //0x0001
 *  地址： 0x03    0x02   0x01   0x00     //高地址-->低地址
 *  age： [ 0x0 ][ 0x0 ][ 0x0 ][ 0x1 ]   //高位-->低位
 *  name:                      [ 0x1 ]   //小端模式，且从低地址开始，所以对应低位1
 */
int main() {
    union People Leason;
    Leason.age = 1;   //0x0001
    if (Leason.name == 1)                     
        //char字符一定是放在低地址，此时低位对低地址，小端
        std::cout << "Small edition" << endl;  
    else 
        std::cout << "Big edition" << endl;    //否则大端
}
```
### inline 内联函数
* 有什么优点？  
提高程序的执行效率,消除了编译时将函数具体展开的额外开销。
* 怎么用？
放在函数**定义**处，若放在声明处不起作用
```C++ 
inline int Max(int a, int b)
{
    return a>b? a:b;
}
cout << Max(a, b) << endl;
//编译时展开为
//cout << a>b?a:b << endl;
```
* 内联函数和宏的区别
1. 宏在**预编译**时就会进行宏替换
2. 内联函数在**编译**阶段，调用处进行替换，所以编译文件会变大，因此适用于简单函数，对于复杂函数，默认使用宏
3. inline函数比宏安全，因内联函数会进行参数检查，宏定义只是简单的文本替换
4. 使用宏定义时需要注意给所有单元添加()

### C++内存管理
* C++内存分为哪几块？
  
内存分块|存放类型 
--|--
自由存储区|malloc、free等
全局区/静态区|全局变量、静态变量，编译时自动初始化
常量区|const常变量、字符串
堆 heap|new申请、delete释放（区别于C）(向上生长)
栈 stack|函数内的局部变量、实参(向下生长)

![](https://blog-10039692.file.myqcloud.com/1503627445050_5296_1503627445162.png)
* 在C语言中，内存分配有三种方式

分配方式|内容
--|:--:
栈分配|由`编译器`在程序`运行时`从栈上分配，函数栈退出时`自动释放`;运算在处理器的指令集中，所以`效率很高`，但能分配的`内容有限`
堆分配|由`程序员`调用内存分配函数`动态分配`申请内存，要`自己释放`申请的内存，使用非常`灵活`；但是由于是通过调用函数来分配，`效率低`
全局/静态区域分配|由编译器自动分配与释放，内存在`编译时`已经分配好，这块内存整个运行期间都存在，如全局变量和static变量
![](http://c.biancheng.net/cpp/uploads/allimg/150304/1-15030412404Q30.png)

* 栈和堆的区别

不同之处|详情
:--:|:--:
管理方式|栈由编译器自动分配和释放；堆由程序员申请和释放
空间大小|一般32位系统下，堆最大能达到4G, 几乎没什么限制；栈一般都有空间大小的，如VC6默认1M
能否产生碎片|堆频繁的操作new/delete，必然造成内存的不连续，产生内存碎片；栈是先进后出的队列，中间的内存块不可能从中间弹出，所以不会造成内存碎片
生长方向|栈向下生长，堆向上生长
分配方式|堆是动态分配的；栈可以静态分配和动态分配
分配效率|栈是系统提供的数据结构，硬件底层为栈提供了专门的寄存器存放栈的地址，因此栈的分配效率很高；堆由库函数分配，效率低
结构不同|堆是一棵完全二叉树，栈是先进后出的结构
* "堆栈"指的是栈，不是堆
* 举个malloc用法的例子
```C++
void getMemory(char **p, int length) {
    *p = (char*)malloc(length);
}

int main(){
    char *str = NULL;
    //注意函数传值，要改变实参的值，必须传入引用或指针
    getMemory(&str, 12);       
    //此时 &str 为实参下一步copy是改变实参的值
    strcpy(str, "hello world"); //加上'\0'功12位
    printf(str);
    free(str);  //释放内存，防止出现野指针
}
```
* 32Linux内核虚拟地址空间划分

![](https://images0.cnblogs.com/blog/593253/201409/281903230762976.png)

从进程的角度来看，每个进程有4G字节的`虚拟空间`， 较低的3G字节是自己的用户空间，最高的`1G`字节则为与所有进程以及内核共享的系统空间.

linux采用`虚拟内存管理技术`，每一个进程都有一个3G大小的独立的进程地址空间，这个地址空间就是用户空间。每个进程的用户空间都是完全独立、互不相干的。进程访问内核空间的方式：`系统调用和中断`。

x86将1G内核空间划分为三部分:ZONE_DMA、ZONE_NORMAL和 ZONE_HIGHMEM。`ZONE_HIGHMEM`即为高端内存。为什么高端？因为当内核想访问高于896MB物理地址内存时，从0xF8000000 ~ 0xFFFFFFFF地址空间范围内找一段相应大小空闲的逻辑地址空间，借用一会。借用这段逻辑地址空间，建立映射到想访问的那段物理内存（即填充内核PTE页面表），临时借用，用后归还。

![](https://images0.cnblogs.com/blog/593253/201409/281913227797020.png)

* 进程内存的分配、回收  
创建进程fork()、程序载入execve()、映射文件mmap()、动态内存分配malloc()/brk()等进程相关操作都需要分配内存给进程。不过这时进程申请和获得的还不是实际内存，而是`虚拟内存`，准确的说是“内存区域”。

使用`do_mmap()`创建新的线性地址空间(扩展)，或`VMA`（虚拟内存），使用`do_ummap()`释放。
* 用户态 内核态

内核态|用户态
--|--
系统调用、内核代码|外部程序，用户代码
中断|用户程序中断，象征性内核态
* 用户态切换到内核态三种方式
1. 异常
2. 系统调用
3. 外围设备中断  
`实际上都是执行了中断`
* 定位内存泄露
1. windows下通过CRT的库函数进行检测
2. 预防：在可能泄露的调用前后生成块的快照，比较前后的状态，定位内存泄露的地方
3. Linux通过valgrind进行检测



### C++面向对象三大特征
特征|作用|目的
--|--|--
封装|隐藏细节、模块化|代码重用
继承|扩展功能，父类、子类|代码重用
多态|抽象类、具体类、虚函数|接口重用

允许将子类对象赋值给父类指针的只能使虚函数，即**多态**
* 模板特化
模板特化分为`全特化和偏特化`，模板特化的`目的`就是对于某一种变量类型具有不同的实现，因此需要特化版本。例如，在STL里迭代器为了适应原生指针就将原生指针进行特化。(特化是为了使用特定环境)
* `函数重载`注意函数名相同，函参个数、类型或顺序必须不同，不能只是返回值类型不同。

### STL
#### STL内存池实现  

一级分配器|二级分配器  
--|--  
malloc(C)|内存池(C++)
>128K | ~128K (8K/16K/^/128K)共16组
`二级分配器`设计的非常巧妙，分别给8k，16k,..., 128k等比较小的内存片都维持一个`空闲链表`，每个链表的头节点由一个数组来维护。需要分配内存时从合适大小的链表中取一块下来。释放该块内存时，将内存节点归还给链表。

如果要分配的内存大于128K则直接调用一级分配器。

为了节省维持链表的开销，采用了一个`union`结构体，分配器使用union里的next指针来指向下一个节点，而用户则使用union的`空指针`来表示该节点的地址。

#### 红黑树
* 集合和映射是基于什么实现？  
(或者：STL中的set底层用的是什么数据结构？)
基于红黑树` RB-Tree`
* 红黑树是怎么定义的？
```C++
enum Color {
    int 0; //red
    int 1; //black
}
struct RBtreeNode {
    struct RBtreeNode *left, *right, *parent; 
    int key;       //键值
    int data;      //数据
    Color color;   //颜色
}
```
* 红黑树有哪些特点？

只有满足以下全部性质的树，我们才称之为红黑树：
1. 每个结点要么是红的，要么是黑的。
2. 根结点是黑的。
3. 每个叶结点（叶结点即指树尾端NIL指针或NULL结点）是黑的。
4. 如果一个结点是红的，那么它的俩个儿子都是黑的
5. 根节点到每个叶结点尾端NIL指针的所有路径中的黑色结点数相同

![RB-tree](https://www.intuit.ru/EDI/01_05_17_1/1493590891-23956/tutorial/909/objects/32/files/32_04.png)
* 红黑树各种操作(如删除、插入、查找)等的时间复杂度是多少？  
最坏情况下，基本的动态几何操作时间复杂度为`O(logn)` 

* RB-tree相比于二叉搜索树`BST`和`AVL` 平衡二叉树有什么优点？   
1. RB-tree & AVL
   
(1) 红黑树牺牲了严格的高度平衡性，只要求部分平衡;降低了对旋转的要求, 从而提高了性能   
(2) 插入、删除、查找等操作的时间复杂度为`O(logN)`    
(3) 任何不平衡都会在`三次旋转`之内达到平衡

2. RB-tree & BST
(1) 由于RB-tree最长路径不超过2倍最短路径，故能保证查找效果，`最坏时间复杂度`是O(logN)，而BST没有这方面限制，所以最坏时间复杂度为O(N)

3. 什么情况下用RB-tree?
在数据分布本身比较理想的情况下，使用AVL;数据分布比较乱时，使用RB-tree处理速度快。

* 红黑树相 & 哈希表，在选择使用的时候有什么依据？  
权衡几个因素: `查找速度, 数据量, 内存使用，可扩展性。`

(1) 总体来说，hash table查找速度比map快，而且速度基本和数据量大小无关，属于`常数级别`；而map查找速度属于`O(logN)`级别(不一定大于常数，而且hash表还需要hash函数消耗时间,构造速度较慢)      
(2) 动态规则的防火墙中，红黑树更具有伸缩性，Linux使用红黑树管理VMA；若数据是静态的，使用哈希表(散列表)

* 如何扩展红黑树来获得比某个结点小的元素有多少个？  
相当于求结点的顺序统计量，任意结点的顺序统计量都可以在O(logN)内得到。
为每个节点添加一个`size域`，表示以结点x为根的任意子树的大小
```C++
size[x] = size[left[x]] + size[right[x]] + 1  //子树的大小 
```
这时红黑树就成了一棵顺序统计树.
利用`size域`可以做两件事：   
1. 找到树中第i小的节点
```C++
OS-SELECT(x, i)  
r = size[left[x]] + 1;  //先查找左子树
if i == r  
     return x  
elseif i < r  
     return OS-SELECT(left[x], i)  
else return OS-SELECT(right[x],  i)  //左子树查找不到，再查找右子树
```
`size[left[x]]`表示在对以`x`为根的子树进行`中序遍历`时排在x之前的个数，递归调用的深度不会超过O(lgn);  

2. 确定某个节点之前有多少节点（即题目问题）
```C++
OS-RANK(T,x)  
r = x.left.size + 1;  //子树x的秩
y = x;  
while y != T.root  
         if y == y.p.right  
                 r = r + y.p.left.size +1  
         y = y.p  
return r  
```
### 常见的内存错误及策略
* 内存未成功分配(NULL)，却使用了它
 
解决方法：在使用内存前检查指针是否为NULL(C++)
1. 如果指针p是函参，那么在函数入口处检查`assert(p != NULL);` 
2. 指针指向动态分配的内存，`if (p != NULL)`  or  `if (p == NULL)`

* 内存虽然成功分配，但未初始化 `int *p = NULL;`
* 内存成功分配且初始化，但操作越出内存边界，通常发生在for循环，下标"多1"或者"少1"
* 忘记释放内存，造成内存泄漏

`(C++) new/delete`  `(C)malloc()/free()` 成对出现
* 内存已经释放，却继续使用它；释放内存后，要立即指针指向NULL，避免出现野指针
* 函数return语句出错

return语句不能返回指向"栈内存"的指针或引用，(栈内存即由编译器自动分配的内存，而不是用过malloc/new申请的内存)，因为函数结束时会自动销毁这些内存。

### 指针、数组、字符串
#### 常量字符串
```C
char a[] = "hello";   // sizeof(a) == 6
a[0] = 'X';           // 数组a的内容是6个字符，其内容为 hello，栈区，可以修改
cout << a << endl;    // Xello

char *p = "world";    // 指针p指向常量字符串，静态存储区，不能修改
p[0] = 'X';           // 编译器不能发现该错误
cout << p << endl;    // 但企图修改常量字符串的内容导致运行报错
```
* 字符串内容的复制与比较

复制时不能使用p = a, 因为这样只是把a的地址赋给了p，要复制内容得用strcmp()
```C
// 数组
char a[] = "hello";
char b[10];
strcpy(b, a);            // 字符串复制，不能使用赋值 b = a，否则编译出错
if (strcmp(b, a) == 0)   // 字符串比较，不能使用 b == a;
{/*...*/}

// 指针
int len = strlen(a);    
char *p = (char *)malloc(sizeof(char)(len+1));  
strcpy(p, a);            // 不能使用 p = a;
if (strcmp(p, a) == 0)   // 不能使用 p == a;
{/*...*/}
free(p);
p = NULL;
```
#### 计算内存容量

sizeof可以得到数组的长度即容量，C/C++没有办法知道指针指向的内存大小，只能在申请时记住它。
```C++
char a[] = "hello world";        // sizeof(a) == 12
char *p = a;                     // sizeof(p) == 4
std::cout << sizeof(a) << endl;  // 12
std::cout << sizeof(p) << endl;  // 4
```
数组作为函参传递时，自动`退化`为同类型的指针
```C++
void fun(char a[100]) {
    std::cout << sizeof(a) << endl;  // 4，而不是100
}
```
#### 指针参数传递内存

如果函参是个指针，不要指针利用指针来动态申请内存，容易出错
```C
void GetMemory(char *p, int nums) {
    p = (char *)malloc(sizeof(char) * nums);  
}

void test1(void) {
    char *str = NULL;
　  GetMemory(str, 100);    // 申请的内存传给了指针str的副本_str，而str仍然为 NULL
　  strcpy(str, "hello");   // 复制给不存在的内存，运行错误
}
```
编译器总是要为函数的每个参数制作`临时副本`，指针参数p的副本是 _p，编译器使` _p = p`。如果函数体内的程序修改了_p的内容，就导致参数p的内容作相应的修改，这就是指针可以用作输出参数的原因。
1. 在本例中，_p申请了新的内存，只是把_p所指的内存地址改变了，但是p丝毫未变。所以函数GetMemory并不能输出任何东西。
2. 而且，每执行一次GetMemory就会泄露一块内存，因为没有用free()释放内存。

* 如果一定要用指针作函参来传递内存，使用双指针或引用
```C
void GetMemory(char **p, int nums) {
    *p = (char *)malloc(sizeof(char) * nums);
} 

void test2(void) {
    char *str = NULL;
    GetMemory(&str, 100);     // 必须使用引用 &
    strcpy(str, "hello");
    printf(str);              // hello
    std::cout << str << endl; // hello
    free(str);
}
```
* 利用函数返回值来代替引用，代替双指针
```C++
char *GetMemory(int nums) {
    char *p = (char *)malloc(sizeof(char) * nums);    // 堆内存
    return p;
}

void test3(void) {
    char *str = NULL;
    str = GetMemory(100);
    strcpy(str, "hello");        // 不需要引用
    std::cout << str << endl;    // hello
    free(str);
}
```
* 利用函数返回指针的方法虽好，但是注意不能返回"栈内存"指针
```C++
char *GetMemory(void) {
    char p[] = "hello world";    // 栈内存
    return p;                    // 编译时发出警告
}

void test4(void) {
    char *str = NULL;
    str = GetMemory();        // str内容是垃圾，因为GetMemory()系统自动分配栈，函数结束时自动销毁
    std::cout << str << endl; //运行没有报错，但是输出的内容是垃圾内容
}
```
* 更改以上数组为指针，有什么问题？
```C++
char *GetMemory(void) {
    char *p = "hello world";     // 常量字符串，位于静态存储区，只读
    return p;
}

void test5(void) {
    char *str = NULL;
    str = GetMemory();        
    std::cout << str << endl; // 编译运行不会报错，但是str指向的永远是一块只读内存
}
```
#### 杜绝野指针
野指针不是NULL指针，而是指向垃圾内存的指针。一般不会错用NULL指针，因为可以用if语句判断；而野指针很难通过if语句判断，很危险。野指针的产生一般有两种原因：
1. 指针没有初始化。 任何指针创建时没有不会自动成为NULL指针，而是指向一块随机的内存。所以创建时应当初始化。
```C
char *p = NULL;
char *str = (char *)malloc(sizeof(char));
```
2. 内存被销毁后没有将指针置为NULL，编译器误以为是合法的指针
3. 指针操作越界，超过了变量的作用于范围。
```C++
class A
{
　public:
　 　void Func(void) { cout << “Func of class A” << endl; }
};

void Test(void)
{
    A *p;
    {              // 函数模块
        A a;
        p = &a;    // 注意 a 的生命期，a是局部变量，在函数结束时会被自动销毁
    }
    p->Func();     // p是野指针，因为p此时指向的内容已经被销毁了
}
```
**但是**上面的函数仍可以正常运行，明明编译器会自动回收啊？？这是因为操作系统`管理内存的单位`不是字节，而是`页`，通常一页为4K字节，所以当函数执行完毕时，系统可能没有马上发现并清除这块内存，因此p指向的内存仍然可读且没有报错。
### 有了malloc()/free()为何还要new/delete
* 相同点

malloc()/free()是C/C++标准库函数，new/delete是C++运算符，都可以用于申请动态内存和释放内存。

* 不同点

 对于`非内部类型`的对象，光靠malloc()/free()无法满足`动态对象`的需求：动态对象在创建时要自动执行构造函数，对象销毁时要自动执行析构函数。由于malloc()/free()时库函数，不在编译器控制范围内，无法将构造函数或析构函数的任务强加于其上；因此C++需要一个能够完成动态内存分配和初始化任务的运算符new，以及一个能够完成清理与释放内存工作的运算符delete。

* 既然new/delete运算符功能已经覆盖了malloc()/free()，为什么还需要它们呢？这是因为C++程序经常调用C函数，而C函数只能通过malloc()/free()库函数来管理动态内存，所以malloc()/free()不能淘汰。

* free()为什么不想malloc那样复杂？

这是因为free()操作对象的类型和长度都是已知的，语句free(str)能够正确释放内存。如果指针是NULL，那么free(NULL)无论操作多少次都不会出错；如果指针不是NULL，执行两次就会出错。

### new/delete使用要点
```C++
int *p1 = new int[length];     // 不需要指定类型
int *p2 = (int *)malloc(sizeof(int) * length);
```
因为new内置了sizeof、类型转换和类型安全检查功能。对于非内部类型的数据对象，new自动完成初始化工作
* 如果对象有多个构造函数，那么new语句也有多个写法
```C++
class Obj {
public:
    Obj(void);    // 无参无返回值
    Obj(int x);   // 一个参数
};

void test(void) {
    Obj *obj1 = new Obj;      // 默认使用无参的构造函数
    Obj *obj2 = new Obj(1);   // 匹配使用带参的构造函数，且初始化为 x = 1
    Obj *objs = new Obj[100]; // 创建了100个对象，不能写成Obj *objs = new Obj[100](1); 
    //...
    // 销毁单个对象
    delete obj1;
    delete obj2;
    // 销毁多个对象
    delete []objs;            // delete objs 会造成程序崩溃和内存泄漏
}
```
### 哪些库函数属于高危函数，为什么？
#### 字符串操作系列
strcpy() / strcmp()等，容易因为内存不足而出现缓冲溢出等问题。可以使用memcpy() / memcmp()等内存操作函数来替换
#### 标准输入输出流系列
sprintf() / scanf().

sprintf() 容易因为输出信息过长导致缓冲溢出，可以用sprintf_s()代替，n表示长度

scanf()   同理可能因为输入信息过长导致缓冲溢出，可以使用scanf_s()代替，前者需要变量地址( &value )，需要( %8s )指明字符串长度。




