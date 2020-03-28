



## 1. find 指令  查找指定文件

​	

​	· find ~ -name “target3.java”：精确查找文件

​	· find ~ -name “target*”：模糊查找文件

​	· find ~ -iname “target*”：不区分文件名大小写去查找文件

​	· ~ 为路径   不指定则从当前目录递归查找



## 2. grep 检索文件内容

 作用：<u>查找文件里符合条件的字符串</u>

![](E:\code\review\own\img\linux_grep_01.png)

![](E:\code\review\own\img\linux_grep_02.png)

### 管道操作符|

可将指令连接起来，前一个指令的输出作为后一个指令的输入

![](E:\code\review\own\img\linux_grep_03.png)

![](E:\code\review\own\img\linux_grep_04.png)



**使用管道注意的要点**

- 只处理前一个命令正确输入，不处理错误输出

- 右边命令必须能够接受标准输入流，否则传递过程中数据会被抛弃

- sed,awk,grep,cut,head,top,less,more,wc,join,sort,split等



### 常用方式

· grep ‘partial\\[true\\]’  bcs-plat-al-data.info.log

> 查找包含某个内容的文件，并将相关行展示出来

​	· grep -o ‘engine\\[[0-9a-z*\\]’

> 筛选出符合正则表达式的内容

​	· grep -v ‘grap’

> 过滤掉包含相关字符串的内容



## 3. awk 对文件内容做统计

![](E:\code\review\own\img\linux_awk_01.png)

-F：以什么符号作为分隔符进行切片

![](E:\code\review\own\img\linux_awk_02.png)



![](E:\code\review\own\img\linux_awk_03.png)



## 4. sed 指令

​	· 全名stream editor，流编辑器

​	· 适合用于对文本的行内容进行处理

> `sed` ‘/[要替换的内容]/[替换后的内容]/’，^str表示以str打头的字符串，开头s表示是对字符串进行的操作

> 默认是将变更的内容输出到终端，并不改变文件的内容

![](E:\code\review\own\img\linux_sed_01.png)

> 加入-i，则可以替换文本内容

![](E:\code\review\own\img\linux_sed_02.png)

> 句号换成分号，$表示以某某结尾

![](E:\code\review\own\img\linux_sed_03.png)

> 若加入g，则会全部替换，否则只替换一个

![](E:\code\review\own\img\linux_sed_04.png)

> 要删除空行，因为不是字符串，所以开头不加s，最后的d表示删除

![](E:\code\review\own\img\linux_sed_05.png)

> 根据Integer删除其所在行

![](E:\code\review\own\img\linux_sed_06.png)

## 5 I/O模型

### 阻塞式IO

- 使用系统调用，并一直阻塞直到内核将数据准备好，之后再由内核缓冲区复制到用户态，在等待内核准备的这段时间什么也干不了

- 下图函数调用期间，一直被阻塞，直到数据准备好且从内核复制到用户程序才返回，这种IO模型为阻塞式IO

- 阻塞式IO式最流行的IO模型 

  ![img](https://user-gold-cdn.xitu.io/2018/10/30/166c5502f8bcffc9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 非阻塞式IO

- 内核在没有准备好数据的时候会返回错误码，而调用程序不会休眠，而是不断轮询询问内核数据是否准备好

- 下图函数调用时，如果数据没有准备好，不像阻塞式IO那样一直被阻塞，而是返回一个错误码。数据准备好时，函数成功返回。

- 应用程序对这样一个非阻塞描述符循环调用成为轮询。

- 非阻塞式IO的轮询会耗费大量cpu，通常在专门提供某一功能的系统中才会使用。通过为套接字的描述符属性设置非阻塞式，可使用该功能 

  ![img](https://user-gold-cdn.xitu.io/2018/10/30/166c553d1d575e5f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### IO多路复用

- 类似与非阻塞，只不过轮询不是由用户线程去执行，而是由内核去轮询，内核监听程序监听到数据准备好后，调用内核函数复制数据到用户态

- 下图中select这个系统调用，充当代理类的角色，不断轮询注册到它这里的所有需要IO的文件描述符，有结果时，把结果告诉被代理的recvfrom函数，它本尊再亲自出马去拿数据

- IO多路复用至少有两次系统调用，如果只有一个代理对象，性能上是不如前面的IO模型的，但是由于它可以同时监听很多套接字，所以性能比前两者高 

  ![img](https://user-gold-cdn.xitu.io/2018/10/30/166c5615fdf084fd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- 多路复用包括： 

  - select：线性扫描所有监听的文件描述符，不管他们是不是活跃的。有最大数量限制（32位系统1024，64位系统2048）
  - poll：同select，不过数据结构不同，需要分配一个pollfd结构数组，维护在内核中。它没有大小限制，不过需要很多复制操作
  - epoll：用于代替poll和select，没有大小限制。使用一个文件描述符管理多个文件描述符，使用红黑树存储。同时用事件驱动代替了轮询。epoll_ctl中注册的文件描述符在事件触发的时候会通过回调机制激活该文件描述符。epoll_wait便会收到通知。最后，epoll还采用了mmap虚拟内存映射技术减少用户态和内核态数据传输的开销

### 信号驱动式IO

- 使用信号，内核在数据准备就绪时通过信号来进行通知

- 首先开启信号驱动io套接字，并使用sigaction系统调用来安装信号处理程序，内核直接返回，不会阻塞用户态

- 数据准备好时，内核会发送SIGIO信号，收到信号后开始进行io操作 

  ![img](https://user-gold-cdn.xitu.io/2018/10/30/166c569138a05186?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 异步IO

- 异步IO依赖信号处理程序来进行通知

- 不过异步IO与前面IO模型不同的是：前面的都是数据准备阶段的阻塞与非阻塞，异步IO模型通知的是IO操作已经完成，而不是数据准备完成

- 异步IO才是真正的非阻塞，主进程只负责做自己的事情，等IO操作完成(数据成功从内核缓存区复制到应用程序缓冲区)时通过回调函数对数据进行处理

- unix中异步io函数以aio_或lio_打头 

  ![img](https://user-gold-cdn.xitu.io/2018/10/30/166c56cf32b82d81?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 各种IO模型对比

- 前面四种IO模型的主要区别在第一阶段，他们第二阶段是一样的：数据从内核缓冲区复制到调用者缓冲区期间都被阻塞住！

- 前面四种IO都是同步IO：IO操作导致请求进程阻塞，直到IO操作完成

- 异步IO：IO操作不导致请求进程阻塞 

  ![img](https://user-gold-cdn.xitu.io/2018/10/30/166c578ad18a1d40?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



