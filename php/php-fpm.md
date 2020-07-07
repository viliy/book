## php-fpm是啥？ Fastcgi Process Manager

全称：php-Fastcgi Process Manager，感受一下就知道意思了
php-fpm php5.3.3纳入官方

1. 实现一个支持 *Fastcgi* 协议的server程序

2. 进程管理器

## cgi 通用网关接口(Common Gateway Interface)

CGI是为了保证web server传递过来的数据是标准格式的，
是 Web Server 与 Web Application 之间数据交换的一种协议。

* CGI的问题

> 当web server收到/index.php这个请求后，会启动对应的CGI程序，这里就是PHP的解析器。接下来PHP解析器会解析php.ini文件，初始化执行环境，然后处理请求，再以CGI规定的格式返回处理后的结果，退出进程。web server再把结果返回给浏览器
> CGI使外部程序与Web服务器之间交互成为可能。CGI程序运行在独立的进程中，并对每个Web请求建立一个进程，这种方法非常容易实现，但效率很差，难以扩展。面对大量请求，进程的大量建立和消亡使操作系统性能大大下降。此外，由于地址空间无法共享，也限制了资源重用。

## fastcgi

一种通信协议，但比 CGI 在效率上做了一些优化。同样，SCGI 协议与 FastCGI 类似。
FastCGI致力于减少网页服务器与CGI程序之间互动的开销，从而使服务器可以同时处理更多的网页请求

1. fastcgi 是一个协议，php-fpm实现了这个协议
2. fastcgi 协议内容：
2.1 fastcgi 启动master进程，解析配置文件，初始化环境，启动多个worker
2.2 请求进来时，master传递至其中一个空闲worker,继续接受请求
2.3 支持动态调度worker:
2.3.1 当worker不够用时，master可以根据配置预先启动几个worker等着
2.3.2 当然空闲worker太多时，也会停掉一些，提高了性能，释放资源

> 与为每个请求创建一个新的进程不同，FastCGI使用持续的进程来处理一连串的请求。这些进程由FastCGI服务器管理，而不是web服务器。 当进来一个请求时，web服务器把环境变量和这个页面请求通过一个socket比如FastCGI进程与web服务器(都位于本地）或者一个TCP connection（FastCGI进程在远端的server farm）传递给FastCGI进程。

> 工作原理
>> (1)、Web Server 启动时载入FastCGI进程管理器【PHP的FastCGI进程管理器是PHP-FPM(php-FastCGI Process Manager)】（IIS ISAPI或Apache Module);
(2)、FastCGI进程管理器自身初始化，启动多个CGI解释器进程 (在任务管理器中可见多个php-cgi.exe)并等待来自Web Server的连接。
(3)、当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi.exe。 
(4)、FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器（运行在 WebServer中）的下一个连接。 在正常的CGI模式中，php-cgi.exe在此便退出了。

> 在上述情况中，你可以想象 CGI通常有多慢。每一个Web请求PHP都必须重新解析php.ini、重新载入全部dll扩展并重初始化全部数据结构。使用FastCGI，所有这些 都只在进程启动时发生一次。一个额外的好处是，持续数据库连接(Persistent database connection)可以工作。

## PHP-CGI

关于*PHP-CGI*很多描述是 PHP （Web Application）对 Web Server 提供的 CGI
协议的接口程序。但实际上却也实现了fastcgi协议

1. php官方自带的FastCGI 进程管理器
2. php.ini修改之后，必须kill掉php-cgi再启动php.ini 才生效。不可以平滑的重启
3. 内存不能动态分配
4. php 5.3以前，用php-cgi 来实现 fastCgi web请求
5. php 5.4开始，php-fpm 取代了php-cgi ，主要原因是 不能平滑重启php ,内存不能进行动态分配

> php-cgi与php-fpm一样，是一个fastcgi进程管理器，php-cgi的问题在于 
> 1、php-cgi变更php.ini配置后需重启php-cgi才能让新的php-ini生效，不可以平滑重启 
> 2、直接杀死php-cgi进程,php就不能运行了。(PHP-FPM和Spawn-FCGI就没有这个问题,守护进程会平滑从新生成新的子进程。） 针对php-cgi的不足，php-fpm应运而生。


## PHP-FPM

上述都一而再再而三的说明PHP-FPM的牛逼之处了，这里就不累赘了。但引用一个博主的反问：
*性能！性能？可能么，难道FastCGI比多线程CGI解释器更快？*

### 配置文件 php-fpm.conf

* 用于编辑Unix套接字与用户和组
* 监听的IP和端口号，允许连入的客户端IP
* 提供同时处理请求总数
* 了解phpfpm使用状态
* 定义资源使用

### 全局配置

```

[global]
pid = /run/php-fpm/php-fpm.pid
error_log = /var/log/php-fpm/error.log
;log_level = notice
;emergency_restart_threshold = 0
;设置时间间隔来决定初始化时间，对于工作在加速共享内存意外时很有用
;emergency_restart_interval = 0

; 子进程等待master进程对信号的回应
;process_control_timeout = 0
daemonize = no

```

### 进程池

```


;   'ip.add.re.ss:port'
;   'port'
;   '/path/to/unix/socket'
listen = 192.168.101.29:9000

; -1 代表不限制
;listen.backlog = -1
; 地址之间用“逗号”隔开
listen.allowed_clients = 192.168.101.191,192.168.101.29

; pm 进程管理器如何控制子进程的数目
; static 静态算法 子进程数目控制为固定数目（pm.max_children）
; dynamic动态算法
pm = dynamic

;同一时刻的最大进程数
pm.max_children = 50

;启动时的进程数
pm.start_servers = 5

; 最小子进程数
pm.min_spare_servers = 5

; 最大子进程数
pm.max_spare_servers = 35

;派生出子进程之前，每个进程应该处理的请求数目
;pm.max_requests = 500

; php-fpm status详解，需要先配置将status转发到phpfpm
pool – fpm池子名称，大多数为www
process manager – 进程管理方式,值：static, dynamic or ondemand. dynamic
start time – 启动日期,如果reload了php-fpm，时间会更新
start since – 运行时长
accepted conn – 当前池子接受的请求数
listen queue – 请求等待队列，如果这个值不为0，那么要增加FPM的进程数量
max listen queue – 请求等待队列最高的数量
listen queue len – socket等待队列长度
idle processes – 空闲进程数量
active processes – 活跃进程数量
total processes – 总进程数量
max active processes – 最大的活跃进程数量（FPM启动开始算）
max children reached - 大道进程最大数量限制的次数，如果这个数量不为0，那说明你的最大进程数量太小了，请改大一点。
slow requests – 启用了php-fpm slow-log，缓慢请求的数量

; 请求脚本可以执行的最大时间
request_terminate_timeout = 0

; 蛮请求设置
;request_slowlog_timeout = 0
slowlog = /var/log/php-fpm/www-slow.log

; 打开文件描述符限制
; 默认是继承系统设置
;rlimit_files = 1024

```

* pm 进程管理器

pm 进程管理器通过静态算法（static）、动态算法（控制子进程数目）

一般web网站推荐使用 *static*

> 优点是不用动态的判断负载情况，提升性能；
> 缺点是多占用些系统内存资源。

* pm.max_chindren 子进程数量限制

> 如果代码是 CPU 计算密集型的，pm.max_chindren 不能超过 CPU 的内核数。
> 如果不是，那么将 pm.max_chindren 的值大于 CPU 的内核数，是非常明智的。

一般情况下 web网站都为IO密集型的，所以*pm.max_chindren*设置数量，建议

> 内存/40m = pm.max_chindren数量
> 即1G内存即可设置： 1000/ 40 = 25 个进程左右

另外比较权威的计算方式是：

* 在 N + 20% 和 M / m 之间。

```

N 是 CPU 内核数量。
M 是 PHP 能利用的内存数量。
m 是每个 PHP 进程平均使用的内存数量。
适用于 dynamic 方式。

```

> static方式： M / (m * 1.2)。如假设每个 PHP 进程使用的内存在 20M - 30M 左右（具体消耗可能有差异，如果要上传文件，处理图像，或者允许的是内存集中型应用，得到的消耗会高些）。而为整个 php-fpm 环境配置了 512 M 内存，那么可以设置为 17-25 中间值。




[参考:关于CGI 和 PHP-FPM需要弄清的](http://www.cleey.com/blog/single/id/848.html)
[参考:cgi、fastcgi、php-cgi、php-fpm剖析](https://www.jianshu.com/p/acccd29b2a9b?utm_campaign=maleskine&utm_content=note&utm_medium=seo_notes&utm_source=recommendation)
[参考:php-fpm 子进程的数量，是越大越好吗？](https://xujimmy.com/2017/10/11/fpm-conf.html)