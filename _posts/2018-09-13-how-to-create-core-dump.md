# core dump的概念

> A core dump is the recorded state of the working memory of a computer program at a specific time, generally when the program has terminated abnormally (crached). In practice, other key pieces of program state a usually dumped at the same time, including the processor registers, which may include the program counter and stack pointer, memory management information, and other processor and operating system flags and information. The name comes from the once-standard memory technology core memory. Core dumps are often used to diagnose or debug errors in computer programs.
>
> On many operating systems, a fatal error in a program automatically triggers a core dump, and by extension the phrase "to dump core" has come to mean, in many cases, any fatal error, regardless of whether a record of the program memory is created.

# core dump文件的生成开关

## 方法一

Linux系统通过ulimit来设置应用程序是否生成core dump文件，ulimit -c参数用来设置core dump文件的最大大小，文件大小的取值范围可以从0到unlimited。当core dump文件大小设置为0时，则表示关闭生成core dump文件，否则，则表示打开。使用ulimit需要注意：

* ulimit命令只对当前tty（终端有效），若要每次都生效的话，可以把ulimit参数放到对应用户的.bash_profile里面；
* ulimit命令本身就有分软硬设置，加-H就是硬，加-S就是软；
* 默认显示的是软限制，如果运行ulimit命令修改的时候没有加上的话，就是两个参数一起改变生效；

## 方法二
 
Linux系统提供了getrlimit，setrlimit两个c函数来实现ulimit的功能，这两个函数的原型是：

```cpp
int getrlimit(int resource, struct rlimit *rlim);
int setrlimit(int resource, const struct rlimit *rlim);
```
    
* resource：resource的取值对应ulimit的各个参数，具体取值可以查看该函数的man page，其中RLIMIT_CORE表示设置core dump文件大小；
* rlim：对于每一个resource，它都有一个对应的soft limit和hard limit，这两个值就保存在一个rlimit结构中，这个结构的原型是：

```cpp
struct rlimit {
    rlim_t rlim_cur;  /* Soft limit */
    rlim_t rlim_max;  /* Hard limit (ceiling for rlim_cur) */
};
```
    
> The soft limit is the value that the kernel enforces for the corresponding resource. The hard limit acts as a ceiling(天花板；上限) for the soft limit: an unprivileged process may only set its soft limit to a value in the range from 0 up to the hard limit, and (irreversibly(不可逆转地)) lower its hard limit. A privileged process (under Linux: one with the CAP_SYS_RESOURCE capability) may make arbitrary changes to either limit value.

## 方法三

通过修改/etc/security/limits.conf文件，也可以达到永久开启/关闭生成core dump文件的目的。这个修改针对所有用户有效。
要使limits.conf文件配置生效，必须确保pam_limits.so文件被加入到启动文件中。通过查看系统文件中的/etc/pam.d/login，如果该文件中有这样一行：

```
session required pam_limits.so
```

则表示已加入，那么修改/etc/security/limits.conf才生效；否则，修改limits.conf并不会起任何作用。
**如果想对所有用户设置，也可以将设置的ulimit命令放在/etc/profile文件中。**

以下例子说明如何启用程序生成core dump文件：
### root用户
    
    1. 在终端下输入
        [root@service3 ~]$ echo "ulimit -c 1024" >> /etc/profile
    2. 重启Linux
    3. 查看是否设置成功
        [root@service3 ~]$ ulimit -c
        1024

### 非root用户
在用户的~/.bash_profile里加上：
    
    ulimit -c 1024
    
来让特定用户可以产生core dump文件，并限制core dump文件的大小为1024kb

### 通过程序控制


# core dump文件的路径
默认情况下，core dump文件生成路径为输入可执行文件运行命令的同一路径下，文件的命名默认规则为core.xxxx，xxxx代表应用程序的pid。用户可以根据需要定制core dump文件的生成路径和命名规则，修改的方式为修改/proc/sys/kernel/下的core_pattern文件。例如，如果想将core dump文件生成在/tmp/corefile下，并且将文件命名成**可执行文件名-文件生成的UNIX时间-导致core dump的信号-pid**，可以按如下命令修改：

    [addison@service3 ~]$ echo "/tmp/corefile/%e-%t-%s-%p" > /proc/sys/kernel/core_pattern

关于core dump文件命名的更多参数，可以通过在终端输入man core来查阅。/proc是内存文件，使用vim无法编辑他的内容，只能使用append的方式来对其进行修改。
> /proc这个目录是虚拟在内存中的，不在硬盘保存。
>
> /proc文件系统是一个伪文件系统，它只存在内存当中，而不占用外存空间。它以文件系统的方式为访问系统内核数据的操作提供接口。用户和应用程序可以通过proc得到系统的信息，并可以改变内核的某些参数。由于系统的信息，如进程，是动态改变的，所以用户或应用程序读取proc文件时，proc文件系统是动态从系统内核读出所需信息并提交的。
>
> /proc目录中的文件值可以进行动态的设置，若希望永久生效，可以通过修改/etc/sysctl.conf文件，并使用下面的命令sysctl -p进行确认。因此，要想修改core dump文件的路径和名称，也可以在/etc/syctl.conf文件中添加如下一行：

    kernel.core_pattern=/tmp/corefile/%e-%t-%s-%p


# 如何产生core dump文件
发生core dump一般都是在进程收到某个信号的时候。Linux上现在大概有60多个信号，可以使用kill -l命令将全部信号列举出来。 
针对特定的信号，应用程序可以写对应的信号处理函数。如果不指定，则采用默认的处理方式，默认处理是core dump的信号是：

    SIGQUIT SIGILL SIGABRT SIGFPE SIGSEGV SIGBUS SIGSYS SIGTRAP SIGXCPU SIGXFSZ SIGIOT


# core dump文件的调试
core dump文件使用gdb工具进行调试，调试的方法是：

    gdb <可执行文件名> <core dump文件>

如果要让gdb打印的信息定位到代码中的某一行，则在编译程序是需要添加-g编译选项

# 参考链接
- http://andyniu.iteye.com/blog/1965571
- http://www.jianshu.com/p/23ee9db2a620