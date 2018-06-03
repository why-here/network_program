### 配置

#### 配置 include 头文件路径

1. `#include "*.h"` 表示先再当前工程目录下查找头文件，如果没有再按标准方式查找；常用于用户自定义头文件的查找。

2. `#include <*.h>` 表示按照标准方式查找头文件，即直接到系统指定的某些目录中去找某些头文件。

3. `$ cpp -v` 可用于查找系统指定的头文件路径。

4. `gcc -l` 可以指定头文件路径。

5. 可在 `/etc/profile` （所有用户都有效）或 `~/.bashrc` （对个人有效）中添加环境变量 [Link](https://blog.csdn.net/yusiguyuan/article/details/16950547)

   ```shell
   # 在PATH中找到可执行文件程序的路径。 以 : 分割路径
   # export 命令将使系统在创建每一个新的 shell 时，定义这个变量的一个拷贝。
   export PATH =$PATH:$HOME/bin
   # gcc找到头文件的路径
   C_INCLUDE_PATH=/usr/include/:/MyLib  
   export C_INCLUDE_PATH 
   # g++找到头文件的路径 CPLUS_INCLUDE_PATH
   # 找到静态库的路径 LIBRARY_PATH
   # 找到动态链接库的路径 LD_LIBRARY_PATH
   # 头文件 gcc foo.c -I /home/xiaowp/include -o foo 
   # 动态库 gcc foo.c -L /home/xiaowp/lib -lfoo -o foo 
   # 静态库 gcc foo.c -L /home/xiaowp/lib -static -lfoo -o foo 
   ```

6. 具体配置过程

   ```bash
   $ sudo apt-get install cmake
   $ sudo apt-get install libboost-dev libboost-test-dev
   $ sudo apt-get install libboost-program-options-dev # config ld
   $ sudo apt-get install libcurl4-openssl-dev libc-ares-dev
   $ sudo apt-get install protobuf-compiler libprotobuf-dev
   $ cd muduo/
   $ ./build.sh -j2 # 编译 muduo 库和它自带的例子，生成的可执行文件和静态库文件
   $ ./build.sh install # 将 muduo 头文件和库文件安装到../build/release-install/{include,lib}
   ```

#### make 与 cmake

> 代码变成可执行文件，叫做[编译](http://www.ruanyifeng.com/blog/2014/11/compiler.html)（compile）；先编译这个，还是先编译那个（即编译的安排），叫做[构建](http://en.wikipedia.org/wiki/Software_build)（build）。
>
> [Make](http://en.wikipedia.org/wiki/Make_%28software%29)是最常用的构建工具，诞生于1977年，主要用于C语言的项目。但是实际上 ，任何只要某个文件有变化，就要重新构建的项目，都可以用Make构建。
>
> [Link](http://www.ruanyifeng.com/blog/2015/02/make.html)

> 通过编写CMakeLists.txt，然后运行cmake命令可以自动生成对应Makefile，从而控制make的编译过程。
>
> ```shell
> cmake_minimum_required(VERSION 2.8)
> add_definitions("-Wall -std=c++11") # <= 新增的编译选项
> add_executable(Main
>   main.cpp
>   mod_func1.cpp
>   mod_func2.cpp
> )
> add_library(Mod2 STATIC
>   func1.cpp
>   func2.cpp
> ) # 如果加上了STATIC，那么就是生成静态库
> add_subdirectory(mod2) # 用于添加cmake管理的目录
> target_link_libraries(Main Mod1 Mod2) # 将库文件链接到指定的可执行文件
> ```
>
> ```shell
> # 执行以下命令，生成可执行文件 Main
> $ cmake .
> $ make
> ```
>
> [Link](https://blog.csdn.net/gg_18826075157/article/details/72780431)

### 2. 一个 TCP 的简单实验

命令：

`dd: 用指定大小的块拷贝一个文件 ` 

`nc: 全称 netcat，可用于监听/连接/扫描端口等 `

`pv：全称 pipe viewer，通过管道显示数据处理进度的信息 `

`irb: 是一个交互式的Ruby界面`

测试 1 ：

```shell
atom-client: dd if=/dev/zero bs=1MB count=1000 | nc e6400 5001
e6400-server: nc -l 5001 > /dev/null
atom -> e6400 118MB/s (full 1Gb/s)
atom -> atom 580MB/s (from RAM)
```

测试 2 ：

```shell
atom-client: time nc localhost 5001 < filename
atom-server: nc -l 5001 > /dev/null
atom -> atom  115MB/s  (first time from disk) 
              1074MB/s (second time from RAM)
```

![Explain the results](tcp_test_1.jpg)

### 4. TTCP 协议

1. 客户端与服务的连接
2. 客户端发送信息，包含即将发送的分组数以及分组长度。
3. 客户端开始发送分组，包含分组长度和分组数据
4. 断开连接。

![ttcp](ttcp.jpg)

### 5. TTCP 代码

static int acceptOrDie(uint16_t prot)

1. static function: 静态函数只在同文件中可见，静态类方法只有类自身可用，其实例不可用。

2. int socket(int domain, int type, int protocol);

   domain: 通信域，(AF_UNIX, AF_LOCAL , 本地)，(AF_INET, AF_INET6, IPv4 IPv6)

   type: 类型，(SOCK_STREAM, TCP)，(SOCK_DGRAM, UDP)，(SOCK_RAW, raw)

   protocol: 协议， IPPROTO_IP, IPPROTO_TCP, IPPROTO_UDP...

   返回值：正常执行返回 sockfd，错误返回 -1，并设置 errno

3. int setsockopt(int sockfd, int level, int optname, const void * optval, socklen_t optlen);

   int getsockopt(int sockfd, int level, int optname, void *optval, socklen_t optlen);

   level: 协议层级，SOL_SOCKET, 或对应的协议号

   optname: 选项名称，

   optval, optlen: 用于存储 option 的值

   返回值：正常执行返回0，错误返回 -1，并设置 errno

4. void perror(const char *s);

   打印额外的信息 s 到 stderr ；

5. void exit(int status);  ----

   status: 返回给父进程的状态值。1--正常退出，0--非正常退出。

   exit()(或return 0)会调用终止处理程序和用户空间的标准I/O清理程序(如fclose), `_exit`和`_Exit`不调用而直接由内核接管进行清理. 可以用

6. struct sockaddr_in; 定义在<netinet/in.h>

   ```c
   /* Structure describing an Internet socket address.  */
   struct sockaddr_in
     {
       __SOCKADDR_COMMON (sin_);  /* sa_family_t sin_family;*/
       in_port_t sin_port;			/* Port number. （使用网络字节顺序） */
       struct in_addr sin_addr;		/* Internet address.  */

       /* Pad to size of `struct sockaddr'.  */
       unsigned char sin_zero[sizeof (struct sockaddr) -
                  __SOCKADDR_COMMON_SIZE -
                  sizeof (in_port_t) -
                  sizeof (struct in_addr)];
     };
   /* Internet address.  */
   typedef uint32_t in_addr_t;
   struct in_addr
     {
       in_addr_t s_addr;
     };
   ```
   sin_port 使用网络字节顺序，需要调用 htons

   sin_family 指代协议族，在socket编程中只能是AF_INET

   s_addr 按照网络字节顺序存储IP地址， 调用 inet_addr("192.168.0.1");

7. `uint16_t htons(uint16_t hostshort);` 等定义在<arpa/inet.h>

8. void bzero(void *s, size_t n); 定义在 <strings.h>

   设为 '\0'

9. reinterpret_cast<T*>(a)：用于进行没有任何关联之间的转换，比如一个字符指针转换为一个整形数。仅仅是**重新解释了给出的对象的比特模型而没有进行二进制转换**。编译器在编译期处理。

   static_cast<T*>(a)：能在内置的数据类型间转换，不允许没有关系的两个类指针之间的转换。不进行类型检查来确保转换的安全性。编译器在编译期处理。

   reinterpret_cast 从比特位层面转换，强制转换；static_cast 从类型层面转换，隐式转换。

   dynamic_cast<T*>(a)：仅能应用于指针或者引用，不支持内置数据类型，如果类型T不是a的某个基类型，该操作将返回一个空指针。在运行期，会检查这个转换是否可能。会读取RTTI信息来验证。

   const_cast<T*>(a)：去掉/添加类型中的常量属性

   static_cast 可以转，只是他不会去读取RTTI的资料来验证而已，他会假设你已经有把握了。

   [Link](https://blog.csdn.net/geeeeeeee/article/details/3427920) [Link](https://www.zhihu.com/question/46470823)

   RTTI：在程序设计中，所谓的**运行期类型信息**（Runtime type information，RTTI）指的是在程序[运行](https://zh.wikipedia.org/wiki/%E5%9F%B7%E8%A1%8C%E6%9C%9F)时保存其对象的[类型](https://zh.wikipedia.org/wiki/%E8%B3%87%E6%96%99%E5%9E%8B%E6%85%8B)信息的行为。

   typeid: 多态类的对象的类型信息保存在虚函数表的索引的-1的项中，该项是一个type_info对象的地址，该type_info对象保存着该对象对应的类型信息，每个类都对应着一个type_info对象。

   dynamic_case：对比源对象和新对象的 type_info 信息

   [Link](https://blog.csdn.net/ljianhui/article/details/46487951)

10. int bind(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

   将 sockfd 与特定的 addr 绑定，

   返回值：成功返回 0， 错误返回 -1，并设置 errno。

11. int listen(int sockfd, int backlog);

   指示 socket 用于接受连接请求，连接请求队列最长为 backlog 个，满了之后会 server 忽略请求或者 client 会收到 ECONNREFUSED。

   返回值：成功返回 0， 错误返回 -1，并设置 errno。

12. int accept(int sockfd, struct sockaddr * addr, socklen_t * addrlen);

   提取 sockfd 队列中的第一个连接请求，新建一个已连接的 socket。

   addr 中将填入对方 socket 的地址信息，addrlen 将可能被修改。

   队列中没有连接请求时，在未被设置为 nonblocking ，accept 会阻塞直到有连接请求；在设置为 nonblocking，会返回 EAGAIN 或 EWOULDBLOCK 

   返回值：成功返回新 sockfd，错误返回 -1，并设置 errno。

13. int close(int fd);

   关闭 fd，对应的锁将被移除。

   返回值：成功返回 0， 错误返回 -1，并设置 errno。

14.  结构体初始化

    `struct SessionMessage sessionMessage = {0,0};`

15.  结构体 `__attribute__ ((__packed__))`

    `__packed__`: 使用最小的内存

    `__aligned__`: 指定对齐大小，或自动对齐

    函数属性（Function Attribute）可以帮助开发者把一些特性添加到函数声明中，从而可以使编译器在错误检查方面的功能更强大。

16.  ssize_t read(int fd, void *buf, size_t count);

    尝试从 fd 中读取 count 个字节到 buf 中。

    返回值：EOF ，返回 0；成功返回读取字节数。错误返回 -1，并设置 errno。

17.  void *malloc(size_t size);

    分配 size 个字节的内存，内存不被初始化。

    返回值：正常返回指向内存的指针，错误返回 NULL，size == 0 返回 NULL 或可被 free 释放的指针。

18.  结构体变长数组。

    ​```c
    struct PayloadMessage
    {
      int32_t length;
      char data[0];
    };
    ​```
    
    使用 static_cast<PayloadMessage*> 或 (PayloadMessage *) 进行转换。

19.  ssize_t write(int fd, const void *buf, size_t count);

    尝试从 buf 中写 count 个字节到 fd 中。

    返回值：成功返回写入 fd 的字节数，不一定是 count；错误返回 -1，并设置 errno。errno 可能为 EINTR，可继续写。

20.  void free(void *ptr);

    释放 ptr 指向的内存。

21.  struct sockaddr_in resolveOrDie(const char* host, uint16_t port);

    根据 host 和 port 新建一个 socket。

22.  struct hostent *gethostbyname(const char *name); <netdb.h>

    返回值：成功返回结构体 hostent （一个静态变量）的指针，错误返回 NULL; 

23.  char *inet_ntoa(struct in_addr in); 

    将网络地址转换为 数点的格式，返回值存储在静态分配的 buffer，会被后续的调用覆盖。

24.  int connect(int sockfd, const struct sockaddr *addr, socklen_t addrlen);

    无连接的 socket 可以多次使用 connect 来更改关联节点。

    返回值：成功返回 0， 错误返回 -1，并设置 errno。

25.  KiB/KB

    1KiB = 1024 bytes

    1KB = 1000 bytes

26.  boost program_options 的 add_options 方法

    add_options 返回了对象 options_description_easy_init 这个对象重载了 operator() 方法，因此可以递归地使用后续的参数列表调用该对象的函数。

27.  `#pragma` 指令可能是最复杂的了，它的作用是设定编译器的状态或者是指示编译器完成一些特定的动作。

### 7. echo 阻塞 IO 永久阻塞

1. 客户端发送过大的数据，导致阻塞在发送状态；
2. 而服务端收到数据后，echo 发送，但是客户端没有接收，从而导致服务端发送也阻塞

### 8. TCP自连接

1. 自连接（只在本机出现）：无服务器在侦听的情况下仍然能成功连接。
2. 客户端在连接时会随机选择（有计数器记录）一个端口，向 server 发 SYN。
3. 当客户端连接的地址中的端口，与自身随机选择的端口一样时，内核会建立自连接。
4. 判断方法：判断是否与服务器使用相同的 IP 和 port 。


### 12. NTP

- Network Time Protocol：基于 UDP。一个 UDP 可以服务所有的 client。
- client 每次应该与同一个 server 同步


- 时钟误差为：${((T_1+T_4)-(T_2+T_3))}/{2}$

![协议](ntp1.jpg)



### 15. UDP vs TCP

- UDP 可能不能用满带宽。没有流量控制
- 并发模型
  - TCP ：一个 fd 对应一个 client，非线程安全（不同线程写同一个 fd 会导致信息串一起），字节流
  - UDP ：一个 fd 对应多个 client，线程安全，数据报

### 16. 时间

- UTC（Coordinated Universal Time）：原子钟，并通过不规则的加入闰秒来抵消地球自转变慢的影响
- GMT（Greenwich Mean Time）：地球自转
- t / 86400 -> day ; t % 86400 -> seconds ; 当 t < 0 时，是向 0 取整的。

### 17. Netcat

- 需要同时处理两个 fd：读写文件，发送/接收 TCP；

- 并发模型：

  - Thread-per-connection with blocking IO
  - IO-multiplexing with non-blocking IO

- “瑞士军刀”

  ```shell
  nc < /dev/zero       == chargen
  nc > /dev/null       == discard
  dd /dev/zero | nc    == ttcp
  nc < file, nc > file == scp
  ```

- 正确关闭 TCP 连接

  - 错误：send() 发送完毕后立马 close()，由于发送/接收 Buffer 中还有未处理完的数据，因此 close 会向对方发送 RST，而不是 FIN，导致对方立即关闭 TCP，丢失部分未接收的数据。
  - 正确：read() 返回 0 后才 close() 或者设计协议。
    - 发送端：send() + shutdown(WR) + read() -> 0 + close()
    - 接收端：read() -> 0 + if nothing more to send + close()

### 18. SIGPIPE

- 末端进程退出后，向输入进程发送 SIGPIPE ，使其退出；避免占用 CPU 资源
- 管道是阻塞 IO；
- client 在意外退出，server 向 client 写数据，会收到 SIGPIPE，导致其退出，进程启动时忽略 SIGPIPE 信号
- Nagle 算法（只要有已提交的数据包尚未确认，发送者会持续缓冲数据包，直到累积一定数量（MSS）的数据才提交。）会影响请求响应的延迟，建议开启 TCP_NODELAY
- 常规操作：
  - SO_REUSEADDR ( crash/kill 后可以马上重启 或者 fork)
  - 忽略 SIGPIPE
  - set TCP_NODELAY
- TTY 行缓冲

### 19. Netcat 模型

![model](netcat_model.jpg)

### 20. T-P-C

- 两个线程一个发，一个收，采用阻塞 IO，能自动节流限速。用于用户少，进程廉价的情况。

### 21. IO-Multiplexing

- 又称Reactor 模式或事件驱动，实现文件事件处理器，使用 Select/poll 加非阻塞 IO。IO 复用不是异步。 
- 阻塞 IO
  - 磁盘文件：阻塞直到读完才返回
  - socket：无数据可读时阻塞，无法发送时阻塞
- 采用阻塞 IO 可能导致一个方向上永远阻塞，如 chargen 只发不收，nc 即发送又接收，nc 会把发送缓冲填满，导致发送阻塞。

### 22. 非阻塞 IO

- 需要应用层的缓存，非阻塞写 -> 未写完缓冲 -> 注册 pollout 事件（有数据未写完，若可写继续写）-> 写完 unregister pollout 事件
- 若一直关注 pollout ，会造成 busy loop（使用电平触发）
- 为何一定要用非阻塞 IO，是否可以检测可读再读，检测可写再写？accept() 是，若 client 已经断开，可能永久阻塞。select() 可能意外设置为可读。

### 23. 非阻塞 IO 2

- 非阻塞写通常由网络库提供
- 待发送缓存不为空时，不应该调 write ，会使数据乱序；为空时，停止 pollout ，否则会 busy_loop
- 对方接收缓慢
- 电平触发：select/poll，epoll 支持电平和边缘触发，边缘触发对 write 和 accept 较好，电平触发对 read 比较好（若只读部分数据，下一次还可以读）。
- 多线程 IO 复用：One-loop-per-Thread

### 24. procmon

### 31. memcached

- 远程 hash 表，缓存数据库查询的结果


- 避免在临界区调用 syscall ，会花时间或阻塞，可将数据缓存在 临界区之外处理
- 使用不可变的数据，来简化同步 `shared_ptr<const value>`const 保证指向的内容不会被改变。
- 容易分区，各个分区各有一个 mutex，提高并发能力
- `unordered_set<shared_ptr<const Item>, Hash, Equal>`