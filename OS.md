# 经典锁问题

## 最简单临界资源

临界资源要求同一个时刻只能一个线程访问

```C++
typedef int semaphore;
samephore mutex;
void func() {
    P(&mutex);
    // 访问临界资源
    V(&mutex);
}
```



## 生产者 & 消费者

一个生产者和一个消费者，要求buffer不满时生产者放入，buffer不空时消费者取出

```C++
semaphore mutex; // 首先buf是临界资源，需要互斥信号量
semaphore empty = N; // 其次生产者消费者需要同步，使用同步信号量
semaphore full = 0;

void producer() {
	while (1) {
        P(empty);
        P(mutex); // mutex一定要在empty/full之后，否则可能出现死锁
        
        produce();
        
        V(mutex);
        V(empty);
    }
}
void consumer() {
    while (1) {
        P(full);
        P(mutex);
        
        consume();
        
        V(mutex);
        V(full);
    }
}
```



## 读者 & 写者

一个写者，多个读者，允许多个读者同时读，但是不允许读的时候写，或写的时候读

```C++
semaphore mutex; // 用于buf临界资源
int readerCount; // 统计读者数量
semaphore countMutex; // 读者数量是临界资源，用于读者数量的锁
void reader() {
    while (1) {
        P(countMutex); // 改变readerCount前先P
        readerCount++;
        if (readerCount == 1) {
            P(mutex); // 当第一个读者在读的时候，就P数据锁mutex
        }
        V(countMutex); // 改变readerCount后释放锁
        
        read();
        
        P(countMutex); // 读完之后改变readerCount，先P
        readerCount--;
        if (readerCount == 0) { //此时还不能释放countMutex，因为还需要readerCount来做判断，也属于访问临界资源
            V(mutex); // 最后一个读者读完之后释放数据锁
        }
        V(countMutex);
        
    }
}
void writer() {
    while (1) {
        P(mutex); // 对于写者比较简单，buf就是写者的临界资源，只需要写之前P信号量即可
        
        write();
        
        V(mutex);
    }
}
```



## 哲学家进餐

5个哲学家围在圆桌前，哲学家不是思考就是吃饭。每个哲学家两边都有一根筷子，当他同时拿到两边的筷子时才能吃饭。

```C++
// 错误做法：先P左边筷子再P右边筷子
void philosopher() {
    think();
    P(leftMutex); // 当所有哲学家都先吃饭时，都P到了左边的筷子，此时右边已经没有筷子了，所以在P右边的时候会出现死锁
    P(rightMutex);
    eat();
    V(leftMutex);
    V(rightMutex);
}
```

```C++
// 正确做法：
// 1、必须同时拿起两根筷子
// 2、只有在两边都没有进餐的情况下，才可以进餐
```



# 死锁

## 必要条件

- 1、互斥：一个资源要么是可用的，要么是已经分配给一个进程
- 2、不可抢占：一个资源被分配给A进程后，其他进程不可抢占，必须由A进程主动释放
- 3、占有和等待：A进程占有一个资源后，等待另一个资源
- 4、环路：两个或者多个进程循环地等待，环路中每一个进程都在等待下一个进程占有的资源

## 死锁处理方式

- 1、预防死锁：破坏死锁的四个必要条件
- 2、避免死锁：银行家算法
- 3、检测死锁：进程和资源占用图，dfs判断环路
- 4、解除死锁：杀死进程或是释放进程占用的资源

# 进程地址空间
## 分页

程序寻址使用虚拟内存，操作系统将内存分页，运行时通过地址加法器硬件把逻辑地址转换为物理地址。

页表记录逻辑地址到物理地址的映射，页面号+页内偏移量

页面号有标记记录当前页是否在内存中，如果不在会出现缺页中断，操作系统会将页面加载到物理内存中，其他页被放到磁盘中的虚拟内存中

页面置换算法：

- 最近最久未使用 Least Recently Used 维护页面号的链表，访问页面在链表内则将该节点放到链表头，不在则载入内存，页面号放到链表头
- 最近未使用：Not Recently Used 每个页面维护Read和Modify标记，Read标记超时清零，每次优先替换Read=0且Modify=1的页面
- 先进先出：First In First Out 会导致频繁使用的页面也被替换出去
- 第二次机会：对FIFO的优化，每个页面记录Read标记，访问时置位1，当到队列头即将被替换掉时，检查Read标记，如果是0则直接替换，如果是1则置位0，且放到队尾
- 时钟 Clock：对第二次机会的优化，循环队列，记录队列尾，这样不需要频繁改变队列顺序，只需要改变队尾的记录即可



## 分段

段大小可变，为了程序内存空间互相分隔，便于管理和安全，将程序内存空间分段（代码段、数据段、BSS堆、栈）

- 代码段：放逻辑代码
- 数据段：放程序中使用的数据，已经初始化的全局变量，静态变量
- BSS段：Block Started By Symbol，放程序中未初始化的全局变量
- 堆：malloc、new的内存，由程序自行管理
- 栈：函数的局部变量，函数的入参、出参，不需要程序自行管理


# 进程调度算法

- 先来先服务 First Come First Serve
- 最短任务优先 Shortest Job First
- 最短剩余时间优先 Shortest Remaining Time Next
- 时间片轮转：时间片大小需要把握，太小导致进程切换开销过大，太大导致实时性差
- 优先级调度：设置进程的优先级，为防止低优先级饿死，可以随着时间推进加大优先级
- 多级反馈队列：设置多级队列，高级队列中的时间片小，低级队列中的时间片长，上一个队列没有进程在排队时放入下一个队列中
- 实时系统：硬实时、软实时

# 磁盘调度算法

- 先来先服务
- 最短寻道时间：优先距离当前最近的磁道，有可能出现饥饿问题
- 电梯算法：每次向同一个方向巡道，直到当前方向上没有请求，换方向

# 进程 & 线程 & 协程

进程：操作系统分配资源的基本单位

线程：操作系统调度的基本单位

协程：比线程粒度更小，用户态执行，不会陷入到内核态，协程总是串行执行，优势在于协程切换开销小，且同一个线程内执行，不需要考虑加锁

# 可重入 & 线程安全

可重入函数：概念起源于单线程，可重入函数在运行过程中可以被中断，在这个函数运行结束前可以再次被调用（重入），且调用后结果正确输出。原因在于可重入函数只依赖于当前的栈空间，不依赖全局变量、静态变量

不可重入举例：malloc、free、printf

线程安全函数：概念起源于多线程

可重入与线程安全没有必然联系，可重入的函数不一定线程安全，线程安全的函数不一定可重入

# 编译

![image-20220523172231083](/home/Lancer/.config/Typora/typora-user-images/image-20220523172231083.png)

## 预处理

test.c -> test.i 处理以 # 开头的预处理命令，#define #ifdef #endif #pragma pack(n)

gcc -E main.c 只做预处理，不会编译、汇编、链接

## 编译

test.i -> test.s 翻译成汇编文件

gcc -S main.c 只会编译成为汇编代码main.s

## 汇编

test.s -> test.o 将汇编文件翻译成可重定位目标文件

## 链接

test.o + printf.o -> test.exe 将可重定位目标文件和printf.o等单独预编译好的目标文件进行合并，得到最终的可执行目标文件

## GCC

gcc -E main.c 只会进行预处理，不会生成文件，回显的形式输出在console中，也可以使用gcc -o main.i main.cpp指定生成main.i预处理文件

gcc -C（大写C） 在预处理的时候不会删除注释，一般和-E一起用与分析预处理之后的程序

gcc -S main.c 只会进行预处理和编译，生成一个main.s的汇编文件，可以cat出来查看汇编码

gcc -S -g main.c 由于是-S，只会生成汇编文件，但是会保留符号表，加了-g之后产生的汇编文件显然会大

gcc -c main.c 只会进行预处理、编译、汇编，不会进行链接，生成一个main.o可链接文件，此文件不可以执行

gcc -o <filename>  <files> 不一定会完整执行四个步骤，只是用于指定生成的文件名，可以 -o 后加-S，此时只会进行预处理、编译，只不过生成的文件用-o指定了文件名，生成文件本质还是汇编代码，不是可执行文件。由于gcc不带参数时默认直接生成可执行文件，因此gcc -o main main.cpp容易被误认为-o直接生成可执行文件

gcc -l<libname> 注意-l后面没有空格，用于链接库文件 gcc -o main -lpthread main.cpp：编译main.cpp后链接libpthread.a这个库文件生成main可执行文件（.a文件是静态链接库文件）

gcc -x <language> <filename> 指定filename的语言进行编译 gcc -o main.s -x c -S main.cpp：指定用c语言的方式预处理、编译main.cpp文件，生成main.s的汇编代码文件。类似于extern "C"，可以指定使用C语言的方式编译

gcc -Dmacro 用于定义宏，类似于C码中#define macro，一般使用场景是：C码中通过#ifdef macro和#else来区分不同分支，使用gcc编译时可以使用-Dmacro来给出macro的定义

gcc -include <filename> 类似于C码中#include<filename>的作用

gcc -M main.cpp 输出main.cpp依赖的文件关系

## 静态链接 & 动态链接

静态链接：

- 符号解析：把函数、全局变量、静态变量等符号引用和符号定义关联起来
- 符号重定位：把符号的定义和内存关联起来，修改这些符号的引用，使之直接关联到内存中
- linux下静态链接文件一般为.a，命令是libxxx.a，链接时使用gdb -lxxx参数，window下静态链接文件一般为.obj

动态链接：

静态链接时，如果库文件有变动，那程序需要重新链接，且对于常用的库函数，如果每一个程序都包含，就会导致程序臃肿

动态链接在程序加载到内存中运行的时候才会链接，多个程序可以共享库中的代码

linux中动态链接库一般后缀.so，windows .dll

# 文件系统

inode：一个文件对应一个inode，记录文件的信息，以及文件占用的block，只有ext2文件系统有inode，FAT系统没有

block：一个文件可能占用多个block，每个block只能存储一个文件，ext2文件系统，inode中记录文件所有block；FAT文件系统，没有inode，每个block链表形式链到多个block

![image-20220523172427946](/home/Lancer/.config/Typora/typora-user-images/image-20220523172427946.png)

# 硬链接 & 软连接

硬链接：ln <originFileName> <dstFileName>，链接文件里面存储inode信息，只要链接数量不是0，文件就不会真正删除，不能为目录创建硬链接

软连接：ln -s <originFileName> <dstFileName>，链接里面存储源文件的目录绝对路径，删除源文件之后链接文件失效，类似于windows的快捷方式，可以为目录创建软连接

ll第二列代表链接的引用个数，-i参数查看inode，硬链接和源文件的inode相同，引用次数都是2，软连接不同，引用次数1

![image-20220523172442572](/home/Lancer/.config/Typora/typora-user-images/image-20220523172442572.png)

# RAM & ROM

## RAM （Random Access Memory）随机存储器，掉电易失

- SRAM 静态RAM，成本高，触发器实现，一般用于CPU内的cache
- DRAM 动态RAM，使用电容实现，由于电容存在漏电的问题，需要一定时间间隔扫描补电，一般用于PC内存条

## ROM （Read Only Memory）只读存储器，掉电不易失

- Mask ROM 掩膜ROM，最初的ROM，只是可读的，出厂后不可写
- PROM programmeable ROM，可编程ROM，芯片设计完成后出厂写入，只可写一次
- EPROM eraseable programmeable ROM，可擦除且可编程的ROM，早期使用紫外线照射的方式擦除
- E2PROM  electronic eraseable programmeable ROM ，电可擦除可编程ROM，不需要紫外线照射，电可擦写
- flash，固态硬盘，u盘，sd卡
- 磁盘，光盘



# linux 进程拉起过程

# VxWorks
​	RTOS实时操作系统  https://blog.csdn.net/gkxg001/article/details/84494938
​	优先级抢占：任务创建时有优先级，如果有任务优先级高于当前正在执行的任务的优先级，系统会保存当前任务上下文（寄存器，PC，堆栈），切换到高优先级任务的上下文执行
​	轮转调度：对于同样优先级的任务，系统对各个ready的任务分配一定的时间间隔，进行轮转执行
​	

# shell

# makefile & CMakeLists
​	patsubst
​	

# service文件

# systemd

# rpm

# aarch64/arm64

# socket进程通信
​	CS模式，通过TCP协议进行通信
​	client设置目的ip和端口，进行socket绑定
​	server监听端口的过程会阻塞，一般新建一个线程进行监听
​	server
​		listen
​		bind
​	client
​		
IO复用
​	https://cloud.tencent.com/developer/article/1805838
​	由于server中一个socket需要阻塞地监听消息，需要新建线程进行操作，导致在多个客户端的情况，server需要新建多个线程，开销较大
​	IO复用让server能够至需要启动一个线程，就可以使用socket监听多个client的请求
​	三种方式 select poll epoll
​	epoll主流，性能好，原因在于：
​		1、select 和 poll需要频繁地从用户空间和内核空间拷贝文件描述符的内存，而epoll的文件描述符本身就在内核当中，在用户空间使用时通过mmap()映射
​		2、select和poll使用轮序机制，用户需要遍历就绪的文件描述符，复杂度线性增长
​		3、select 通过位域来标记活跃的文件描述符，默认最大1024个，poll使用链表，虽然可扩容，但是复杂度也是线性增长；
​			而epoll使用回调的方式触发，用户可以直接知道哪个文件描述符是就绪的状态
​		

	epoll支持水平触发和边缘触发，select和poll只支持水平触发
		水平触发：
			socket接收缓冲区不空，一直触发；
			socket发送缓冲区不满，一直触发
		边缘触发：
			socket接收缓冲区从空到有数据的时候触发
			socket发送缓冲区从满到不满的时候触发


# 看门狗
​	

# daemon

# posix
​	portable operation system interface
​	可移植操作系统接口，是一个ieee公布的标准，目的是跨unix平台使用，比如write，fork

# selinux

# gdb符号表

# 