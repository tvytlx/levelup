# C语言


- [读《C陷阱与缺陷》](#%E8%AF%BB%E3%80%8Ac%E9%99%B7%E9%98%B1%E4%B8%8E%E7%BC%BA%E9%99%B7%E3%80%8B)
  - [词法陷阱](#%E8%AF%8D%E6%B3%95%E9%99%B7%E9%98%B1)
  - [语法陷阱](#%E8%AF%AD%E6%B3%95%E9%99%B7%E9%98%B1)
  - [语义陷阱](#%E8%AF%AD%E4%B9%89%E9%99%B7%E9%98%B1)
  - [链接](#%E9%93%BE%E6%8E%A5)
  - [库函数里的缺陷](#%E5%BA%93%E5%87%BD%E6%95%B0%E9%87%8C%E7%9A%84%E7%BC%BA%E9%99%B7)
  - [预处理器](#%E9%A2%84%E5%A4%84%E7%90%86%E5%99%A8)
  - [可移植性缺陷](#%E5%8F%AF%E7%A7%BB%E6%A4%8D%E6%80%A7%E7%BC%BA%E9%99%B7)
- [读《C专家编程》](#%E8%AF%BB%E3%80%8Ac%E4%B8%93%E5%AE%B6%E7%BC%96%E7%A8%8B%E3%80%8B)
  - [数组和指针](#%E6%95%B0%E7%BB%84%E5%92%8C%E6%8C%87%E9%92%88)
  - [像"bug"一样的语言特性](#%E5%83%8Fbug%E4%B8%80%E6%A0%B7%E7%9A%84%E8%AF%AD%E8%A8%80%E7%89%B9%E6%80%A7)
  - [链接的思考](#%E9%93%BE%E6%8E%A5%E7%9A%84%E6%80%9D%E8%80%83)
  - [运行时系统](#%E8%BF%90%E8%A1%8C%E6%97%B6%E7%B3%BB%E7%BB%9F)
  - [对内存的思考](#%E5%AF%B9%E5%86%85%E5%AD%98%E7%9A%84%E6%80%9D%E8%80%83)
  - [根据位模式构建图形](#%E6%A0%B9%E6%8D%AE%E4%BD%8D%E6%A8%A1%E5%BC%8F%E6%9E%84%E5%BB%BA%E5%9B%BE%E5%BD%A2)


## 读《C陷阱与缺陷》
### 词法陷阱
一般知道词法分析这个过程的话，词法错误就没什么了。值得注意的是，词法分析一般是贪婪策略，就是会优先匹配出更长的词法单元。

### 语法陷阱
* 理解函数声明。首先得理解声明的过程。
```c
//格式：类型 声明符号的表达式;
float *g(), (*h)();//()优先级高于*，所以g是一个函数，而h是函数指针。
```

 这样声明之后变量g,h就有意义了。
 ```c
 //(float (*)()) 表示的就是一个类型转换符
 
 // 关于typedef的用法,typedef不仅仅只能像define那样去替换
 typedef int size;
 #define size int;
 // 还可以这样用。一个简单的例子
 typedef float (*h)();
 h funcptr;
 
 // 一个复杂的例子。
 void (*signal(int ,void(*)(int)))(int);
 
 // 用typedef简化
 typedef void (*HANDLER)(int);
 HANDLER signal(int, HANDLER);
 
 // 总结：我们平常读源代码看到的奇怪的声明，应该就是利用的typedef。typedef可以用来简化定义，一般是涉及函数指针的声明才有简化声明的必要，假如是函数的话，只要声明一次。
 ```
 
 关于define和typedef的区别
 1. define是宏展开，而且作用域是全局的。typedef作用域是声明之处。
 2. 
    ```c
    typedef char* cptr;
    const char *a;// 限制的是a指向的值为只读
    const cptr  b;// 因为cptr是自己定义的，所以限制的是b本身的值为只读。
    ```

* 运算符的优先级。关系运算符比逻辑运算符高！放心写`a>b&&c<d`。

	![](http://7xrkyy.com1.z0.glb.clouddn.com/16-6-6/87346285.jpg)


* 其他的一些诸如switch穿越，else悬挂之类的比较低级，也很容易避免，就不说了。 

### 语义陷阱
* 数组与指针。没什么好讲的，翻来覆去就是考理解。
* 非数组的指针，要注意内存分配的问题。
* `int strlen(char s[]){}`和`int strlen(char *s){}`是一样的，因为编译器自动把参数的数组声明转换为相应的指针声明。
* 数组边界。采用左闭右开的不对称形式。因为c语言数组是从0开始的，因此这种方式的上界表示的正好是元素个数。eg. `int a[10]`我们可以引用a[10]的地址，`&a[10]`，且可以对这个地址进行赋值和比较，ANSI C标准规定这是合法的(允许获得数组尾端出界的第一个位置)，只不过对这个地址解引用就是非法的了。写数组相关的程序的时候，计数是个常用的概念。
* 求值顺序。C语言中只有四个运算符存在规定的求值顺序。`&& || ?: ,`。求值顺序和优先级当然是不同的概念了。eg. `if(a!=0&&b/a<1)`，这种写法就避免了当a是0的时候，分母是0的错误出现。这个应该可以用来优化代码。关于优先级，结合性自己举几个例子就理解了。
* 判断两个数相加是否溢出。
 ```c
 if(a + b < 0)   //这是不对的，因为有的机器的寄存器的状态有溢出也有负，所以结果不能断定他就是负的。
     complain();
 if((unsigned)a + (unsigned)b < INT_MAX)  //这样是正确的，ANSI C在<limit.h>里定义了INT_MAX，代表可能的最大整数值。
     complain();
     
 // 有符号数的溢出结果是不固定的。无符号数在溢出之后会从0开始重新计数。32位机器，int 范围是-2^31~2^31,unsigned int 范围是0~2^32
 ```
 
* main函数要有返回值，因为这样操作系统就知道程序的执行结果是失败还是别的情况。


### 链接
* 变量的声明与定义。`int a;`这种一般都是声明的同时定义，假如它在所有函数体外面，那么就把她叫做一个外部对象a的定义，同时为a分配了存储空间。`extern int a;`这种就是只有声明的意思，它的定义可以在本文件，也可以在别的文件，它的内存分配理所当然也是在定义的地方分配的。
* 外部变量命名冲突是不允许的。假如你写了个函数，名称和别的文件里定义的函数名称冲突了，怎么办呢。用`static`。static修饰的变量作用域是本文件，不会成为外部变量or函数。当你要用的函数是只为当前文件内的函数服务的，那么把它设置成static。
* 外部变量，相同的名字，不同的类型。这种错误，现在的编译器应该可以查出来了。
* **如果一个函数没有float,short,char 类型的参数**，在函数声明中完全可以省略参数类型的说明，看个例子。
```c
// file: a.c
int func(int n)
{
	return n;
}
// file: main.c
extern func(); // 省略了参数说明
main()
{
	func(1,2,3,4); // 传递的参数可以是任何类型，因为有相应的隐式转换
				   // 假如定义的是低级别的类型，就不能提升或者转换，所以会出错。
}
```
* 头文件，使得每个外部变量只在一个地方声明。而且头文件里每个外部变量或者函数都要用extern才对。
* 总结。编译生成obj，链接把这些需要用到的obj弄到一块，生成可执行文件。具体编译生成了什么格式的文件，需要做实验看一下。

### 库函数里的缺陷
* 返回整数的getchar()，假如你用个char类型的变量去接收它的返回值，那就有可能会发生截断。
* 文件的更新顺序。就是说fread()和fwrite()不能连在一起执行，必须要设置一下fseek才能。这也太奇葩了，不知道现在改了没。
* setbuf()，可以设置输出的缓冲区大小，这样就能让程序员在实际写操作之前控制产生的输出数据量，因为块为单位传输肯定比字节单位传输要快。这里可以自己做一下实验看看。
* errno外部变量，当库函数执行失败的时候，会设置它的值来传递错误信息，但是也没有强制要求出错就设置它为0，所以你也不好显式的利用它来检测库函数执行后的错误。ps一般不会怎么用它吧。
* signal()函数。因为触发条件可能难以预测，所以信号处理函数应该编写的越简单越好。。

### 预处理器
* 宏定义。只是展开而已，即便是带参数的宏定义，也不能把它想作函数。要注意里面的每个变量都要带一个括号，是为了防止优先级导致的错误。
* 宏不是语句。就说它展开之后，可能造成语句混乱。
* 宏不是类型定义。这是当然。
* 总结。所有的问题都来自它是直接展开的机制这个根本原因。

### 可移植性缺陷
* C语言标准变更。你要使用新特性的话，就得升级编译器。如果新标准能向下兼容那还好说，可以把旧的工程用新编译器重新编译一遍。
* 外部名称，有的编译器会截断太长的名称(printf_float()，截断就变成了printf了，所以最好避免这样命名)，有的会不区分大小写，从而导致外部命名冲突。所以这些情况可能会造成移植失败。不知道现在的这些C编译器的实现里有没有统一避免这个问题。
* 32位64位的问题，假如是整型(int ,unsigned int,long)，只要不是太大，就没有问题吧。
* (-1)>>1 和 (-1)/2的结果。
* malloc和realloc的实现机制的一些历史问题。
* 总之，程序要允许在不同的平台上，就要考虑好移植性，一般源代码的涉及要移植的部分都会有特定平台的实现。


## 读《C专家编程》
### 数组和指针
我才意识到我以前的理解是比较肤浅的，毕竟普通的教材上也不会讲这些deep C secrets。

**数组和指针显然是两种不同的类型，我们之所以会混乱，是因为数组名可以当作指针一样使用。**什么时候既可以用指针，也可以用数组名呢？

1. 用a[i]这样的形式对数组访问的时候，编译器会把a当做指向数组第一个元素的指针。
2. 在作为函数的参数的时候，无论你传进去的是数组名，还是指向数组的指针，统一都会被编译器修改为指向数组的一个元素的指针。当然这个指针其实是一个拷贝了该数组地址的变量。（c++里面的传递引用，估计就是真的传址）
3. 在其他的所有情况中，定义和声明必须匹配。你定义的是数组，声明也要用数组。`extern int *p`和`int p[10]`，这样是错误的。
4. 当数组名和指针是`&,sizeof()`的操作数的时候，你就会发现，前者和后者完全就是两个东西。

**形参为数组的时候，在函数里面是没法获取数组的长度信息的。**解决方法，要么是加一个表示长度的形参，要么在数组尾部加一个结束标记。

**函数返回值为数组。**这句话是不严格的，因为函数不能返回数组，只能返回一个指向数组的指针。
```c
// 函数定义
int (*fun()) [20]
{
    int (*p)[20]; //声明一个指向包含20个int元素的数组的指针
    p = malloc(20,sizeof(int));
    if(!p) error();
    return p;
}
// 调用
int (*r)[20];
r = fun();
(*r)[3]=1;

```
int (*p)[20]的这种声明下的p的类型是指针,而且是一个指向数组的指针，而我们平常写的int *p = 数组名，p本身是指向一个int数据的指针，在这里它指向的是数组第一个int型元素。因此两者的类型是不一样的，所以使用方法也不一样。我觉得所有类似的这些坑，都是来自类型系统，往类型上思考才是最根本的

### 像"bug"一样的语言特性
* 我以前对类型的看法是错的。我总是认为int \*a，是类型为int\*，变量名是a，所以潜意识认为所有的变量定义，都是类型名 空格 变量名 这种格式。然而实际是，先类型名，空格之后，是一个表达式，对这个表达式求值后才能知道这个变量是什么类型的。`int (*p)[20]`这种就是表示指向整型数组的指针类型，`int (*(*p)())[20]`这个表示返回值为`int [20]`数组指针的函数的函数指针类型。

 造成这种错误认知也并不全是我的过错，有来自C语言的误导，比如 `int *ap[];`按照我原来错误的理解，反而得到了正确答案，int* 表示类型，ap[]表示这是一个数组，所以这个数组里存的都是指向int的指针。然而真相却不是这样的，这里\*ap[]在运算的时候，因为[]优先级高于\*，所以ap先被视为了数组名，然后在确定数组元素类型的时候，\*和int才被加上去。

 很多东西凭感觉得出来的也不一定是对的，虽然我一直很相信自己的感觉。对那些语言设计者来说，将语言设计的没有歧义同时保持简洁优雅，是很难的吧。

 其实有了这个正确的观念后，typedef和#define的区别也就清楚了，你就可以和typedef做朋友了。

* const限定的常量，跟字面值常量是不同的。eg. `const int a = 1`和`字面值1`是不同的，你把a传到switch结构里的case 条件里就会发现不行。书里说早知道把const改成readonly了。

* 符号的重载。比如static修饰函数的时候表示这个函数只在本文件有效，当修饰函数内部的变量的时候，表示该变量在各个调用间保持延续性。等等还有一些限定符，操作符都是会根据不同的上下文而具有不同含义的。
* 总结。每个语言都有自己风格的特性，c属于内存级别的语言，它能做到在低级和高级语言之间保持平衡也是很了不起的。


### 链接的思考
编译器是由六七个小程序组合而成的，然后由一个编译驱动器来整合它们一起运行。链接阶段做了什么事情呢？

我们知道，在编译器编译过后(预处理，前端，后端，优化器，汇编器)，外部对象都存在了符号表里，而链接就是把所有的外部对象都聚集到一起，并且绑定内存地址。

假如是静态链接的话，直接去目标文件里去找到那个函数(外部对象)，并且把它跟所有的引用相关联（这个过程叫符号解析），然后给符号定义设置一个内存地址，同时修改所有的引用（这个过程叫重定位），是库函数的话，就会把库函数的二进制拷贝到可执行程序里，其他一样。

假如是动态链接的话，在编译的链接阶段，最多就只会完善一下符号表，保存下那些动态库的文件名和位置信息。而在运行阶段，首先是**加载器**(p.s.按道理说应该不存在什么专门的加载器吧，这是个很常规的过程，比如通过系统调用fork()+exec()），把可执行文件从外存加载到内存并进行执行，这时候有了进程有了自己地址空间。然后**运行时载入器**把所有共享的数据对象载入到进程的地址空间，然后**动态链接器链接程序(ld-linux.so)**会在外部函数需要真正被调用的时候，完成符号解析，重定位的工作。

因为动态链接有很多好处，比如可以将库和用户程序隔离开，方便库的升级(这种设计叫ABI，应用程序二进制接口)；减少物理内存消耗，减少换页；程序占用空间小等。书上建议只使用动态链接。

**动态链接的实际使用自己做做实验会更清楚**。比如先编译出一个自己的动态库xx.so，然后再用另外一个程序来链接它。

这一章还讲了个**Interpositioning**的问题，就是说你定义的函数和将被动态链接的库函数同名了，它将覆盖掉库函数的定义，这将导致在你的程序里所有调用该库函数的系统调用也被改成调用你定义的函数了。C编译器不会报错，因为它的设计哲学是程序员做的都是对的。书上说使用它是走钢丝的行为，你是专家倒也没事，新手要避免用它，因为Interpositioning实现的效果都可以用其它或许会复杂点的实现替代，为此书里还弄了张表告诉你那些"不要在标识符中使用的名字"。

我发现这本书很多东西都在激励你去动手做实验，以此来检查他是不是在吹牛。

### 运行时系统
编译的时候，不加`-o`参数，会生成a.out文件。来仔细观察这个文件。

先学习一下nm这个程序，它是用来查看目标文件或者执行文件里的符号信息，对于每个符号，它会显示符号的值(默认是16进制)，以及符号的类型，符号的类型很多，自己man一下就知道了，把它在这里抄一边纯属浪费时间。

发现光用nm还不足以做实验，还得结合size，readelf，ls -l。搞了一起，总算可以证明书里说的是怎么回事了。
```
1. 首先，未初始化的全局变量被记录到bss里，这是没有问题的。
2. 然后，bss实际只记录那些变量的占用的大小信息，而不会在目标文件里分配空间。证明过程：

step1:
全局范围内声明一个数组char pear[152];
$ size a.out
   text	   data	    bss	    dec	    hex	filename
   1099	    556	    184	   1839	    72f	a.out		//184字节
   
$ ls -l
-rwxrwxr-x 1 arctanx arctanx 8712 6月   8 20:53 a.out*	//8712个字节
-rw-rw-r-- 1 arctanx arctanx  178 6月   8 20:53 test.c  

step2:
将char pear[152]改为char pear[1520]，扩大了十倍。
$ size a.out 
   text	   data	    bss	    dec	    hex	filename
   1099	    556	   1552	   3207	    c87	a.out		//1552字节
$ ls -l
-rwxrwxr-x 1 arctanx arctanx 8712 6月   8 20:57 a.out*	//8712个字节
-rw-rw-r-- 1 arctanx arctanx  179 6月   8 20:57 test.c

3. data段保存初始化了的全局变量，并且会在目标文件里占用对应的空间。证明过程差不多就不说了。

```
然而，我发现自己对目标文件里的数据布局还是有点迷糊，继续探索。

通过`readelf -S a.out`可以打印段表，至于这个段表的内容是存在哪的呢？可以从打印的段表中看到，有以下内容
```
1) 以 .rec 打头的 sections 里面装载了重定位条目；
2) .symtab  section 里面装载了符号信息，一般是变量和函数
3) .strtab  section 里面装载了字符串信息
4) .shstrtab section 里面装载了段表字符串表(section head string table)
4) 其他还有为满足不同目的所设置的section，比方满足调试的目的、满足动态链接与加载的目的等等。
```

所以我认为符号表保存在.symtab和.strtab，段表保存在.shstrtab。关于更详细的内容，看[这篇博客](http://blog.chinaunix.net/uid-25147458-id-3374229.html)。

刚刚一直分析的是可重定位ELF文件，然而加上-o参数生成的可执行ELF文件的布局应该会有差别。不管了，继续看这一章后面的部分。

**操作系统对a.out干了什么？**干的东西很少，就是给进程分配一块内存，并且清零，然后把那些section从文件载入到内存。高地址是堆栈段，低地址是文本段。

**C语言运行时系统在a.out里干了啥?**堆栈段。不知道这里是不是说堆栈段就是C runtime函数创建，并且提供操作的。这个段有两个作用，为函数局部变量提供存储空间和记录函数调用的过程。

**当函数被调用时，发生了什么？**我决定用GDB来探索。

gcc编译的时候，需要加上-g选项，以便在目标文件中加上调试所需要的信息，这样GDB才能正常工作。可以看到多出来的表有:

![](http://7xrkyy.com1.z0.glb.clouddn.com/16-6-9/45792234.jpg)

实验过程 + 一些废话：
```
借用书中p126的测试代码。
(gdb) list 1,15
1	#include<stdio.h>
2	void a(int i)
3	{
4		if(i>0)
5			a(--i);
6		else
7			printf("i has reached zero");
8		return;
9	}
10	int main()
11	{
12		a(1);
13	}

// b 在line 5和line 12设置断点，并运行。
(gdb) b 5
Breakpoint 1 at 0x400537: file tracestack.c, line 5.
(gdb) b 12
Breakpoint 2 at 0x40055d: file tracestack.c, line 12.
(gdb) run
Starting program: /home/arctanx/code/practice/Expert_C_Programming/tracestack 

Breakpoint 2, main () at tracestack.c:12
12		a(1);

// 打印函数调用信息。bt --backtrace
(gdb) bt
#0  main () at tracestack.c:12
(gdb) c
Continuing.

Breakpoint 1, a (i=1) at tracestack.c:5
5			a(--i);
(gdb) bt
#0  a (i=1) at tracestack.c:5
#1  0x0000000000400567 in main () at tracestack.c:12  // 等于下面的saved rip的值

// 打印当前函数的活动区间
Stack level 0, frame at 0x7fffffffd960:		// 栈帧的地址
 rip = 0x400537 in a (tracestack.c:5); saved rip = 0x400567 // rip的值是ip指针的值，saved rip的值，调用本函数的那条汇编语句的地址
 called by frame at 0x7fffffffd970		// 上一个栈帧的起始地址。（高地址）
 source language c.
 Arglist at 0x7fffffffd950, args: i=1		// 存放函数参数的地址。
 Locals at 0x7fffffffd950, Previous frame's sp is 0x7fffffffd960  // 存放函数局部变量的地址，
 Saved registers:
  rbp at 0x7fffffffd950, rip at 0x7fffffffd958  // 压栈时要保存的相关寄存器的地址。

```
总结。递归调用会使用一个新的栈帧，这跟我自己写的虚拟机里的做法是一样的，当时是觉得这样做最自然，又简单，原来实际上也是追求的简单。栈帧里面，存了局部变量，函数参数的值，还存了一些寄存器的值(这里的寄存器应该只是逻辑上的，真的寄存器只有cpu里有= =）。GDB还是挺好用的，打印的信息很强大。

**setjmp,longjmp函数**可以操作过程活动记录，C++里的throw catch机制也是类似的。下面是我无聊写的代码，猜它会输出几个hello?

![](http://7xrkyy.com1.z0.glb.clouddn.com/16-6-9/1771283.jpg)


### 对内存的思考
这一章，先是讲了下cpu，cache，虚存，MMU。然后讲内存泄漏，总线错误，前者是指针被删了的原因，后者是地址没对齐的原因。地址对齐简单理解，就是不同的数据类型占用的内存大小不同，但是它们的地址不能任意安排，得是自己空间大小的整数倍，这样取址会更方便，因为取都是取一个字长，然后扔掉低地址位，假如没有这些空隙，各个数据都是紧挨着放的，那取地址就太麻烦了。。

其他的没什么好讲的，我觉得不值得去细看，因为操作系统的内存管理比这本书讲的好的多。

### 根据位模式构建图形
这一章不知道什么意思，我就觉得那个宏定义的位图很有意思。C不支持2进制常数，这个技巧某种程度实现了书写二进制的效果。

```c
#define X ) *2+1
#define _ )*2
#define s ((((((((((((((((0

// 表示一个16位的数，X代表1,_代表0，v的值为0x07c6。
unsigned short v = s _ _ _ _ _ X X X X X _ _ _ X X _ ;
```
