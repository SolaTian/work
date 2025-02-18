# 文件系统磁盘使用情况

## df

df 命令可以显示文件系统磁盘空间的使用情况

    df [选项] [文件或挂载点]

- -h：以人类可读格式显示
- -a：显示所有文件系统，包括虚拟文件系统
- -t：显示指定类型的文件系统
- -x：不显示指定类型的文件系统

常用的就是上面两个参数

    #df -h
    Filesystem                Size      Used Available Use% Mounted on
    rootfs                  249.3M     20.0M    229.2M   8% /
    udev                    252.9M    148.0K    252.8M   0% /dev
    /dev/mmcblk0p19         120.0M     62.5M     54.9M  53% /dav0
    /dev/mmcblk0p20         120.0M     62.5M     54.9M  53% /dav1
    /dev/mmcblk0p21          14.5M     12.2M      2.0M  86% /dav2
    /dev/mmcblk0p23          14.5M    143.0K     14.0M   1% /sysLog
    /dev/mmcblk0p26           2.9G    197.5M      2.6G   7% /dav4
    /dev/sda1                19.6G     44.5M     18.5G   0% /mnt/sda

- Filesystem：文件系统的名称，通常是设备文件（如 /dev/sda1）或挂载点对应的设备。
- Size：文件系统的总大小。
- Used：已使用的磁盘空间大小。
- Avail：可用的磁盘空间大小。
- Use%：磁盘空间的使用百分比。
- Mounted on：文件系统的挂载点，即文件系统在目录树中的挂载位置。

根据这些信息，就可以了解各个文件系统的磁盘使用情况，发现磁盘空间不足问题。


# 系统内存占用

## ps

ps 命令用于查看当前系统上运行的进程、线程信息。

    ps [选项]

- a：显示所有用户的进程（包括其他用户的进程）。
- u：显示进程的所有者（即用户名）。
- x：显示没有控制终端的进程（后台进程）。
- e：显示所有进程（包括其他用户的进程）。
- f：显示进程的树状结构，即父进程与子进程之间的关系。
- L：显示线程。
- T：显示进程的线程信息，常常和 -p 一起使用。
- p：显示指定进程ID，只查看特定进程的状态信息。

ps aux 和 ps -ef 的效果一样，只是输出格式不同。


### 显示进程信息
示例

    #ps aux 
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    root         1  0.1  0.2 169128  8908 ?        Ss   12:45   0:03 /sbin/init
    user      1234  0.0  0.1  74832  4608 pts/0    Ss+  12:46   0:00 bash
    user      5678  1.2  0.5 500000 25000 pts/1    Sl+  12:47   1:23 python3 script.py

- PID：进程ID
- TTY：终端类型，指该进程在哪个终端上运行，如果进程是后台进程或没有终端，则显示为 ?。
- TIME：进程使用的 CPU 时间总量。
- CMD：进程启动时的命令行。
- %CPU：进程使用的 CPU 百分比。
- %MEM：进程使用的物理内存百分比。
- STAT：进程/线程状态
  - S：表示休眠
  - R：表示运行中
  - Z：表示僵尸进程
  - T：停止状态
  - D：不可中断的休眠状态
  - Ss：主线程休眠（s 表示会话首进程）
  - Sl：多线程休眠（l 表示多线程）
  - Sl+：多线程休眠且在前台进程组（+ 表示前台）
  - R+：运行中的前台进程
- VSZ：进程使用的虚拟内存大小。
- RSS：进程使用的实际物理内存（驻留集大小）。


        #ps -ef
        UID        PID  PPID  C STIME TTY          TIME CMD
        root         1     0  0 12:45 ?        00:00:03 /sbin/init
        user      1234  5678  0 12:46 pts/0    00:00:00 bash
        user      5678  1234  1 12:47 pts/1    01:23:00 python3 script.py

### 显示线程消息

示例

    #ps -T -p 1234
    PID   SPID TTY      STAT   TIME COMMAND
    1234  1234 pts/1    Sl     0:00 ./my_server
    1234  1235 pts/1    Sl     0:00 ./my_server
    1234  1236 pts/1    Sl     0:00 ./my_server

    #ps -Lf -p 1234
    UID        PID  PPID   LWP  C NLWP STIME TTY      STAT   TIME CMD
    user      1234  1233  1234  0    5 14:30 pts/0    Sl+    1:23 /usr/bin/my_server
    user      1234  1233  1235  0    5 14:30 pts/0    Sl+    0:45 /usr/bin/my_server
    user      1234  1233  1236  0    5 14:30 pts/0    Sl+    0:12 /usr/bin/my_server

该命令会列出指定进程的所有线程，每个线程会显示自己的线程ID（LWP）、状态、CPU使用情况等，下面的命令会更加详细的显示出线程信息。
- PID：进程ID。
- SPID：线程ID。每个线程有独立的 SPID（在 Linux 中等同于 LWP 或 TID）。
- TTY：进程关联的终端。pts/1 表示伪终端（如 SSH 或终端窗口）。
- STAT：进程/线程的状态
- TIME：进程/线程累计使用的 CPU 时间。
- COMMAND：启动进程的命令名称。多线程程序的线程通常显示相同的命令名。



    #ps -eLf
    UID        PID  PPID   LWP  C NLWP STIME TTY      STAT   TIME CMD
    root         1     0     1  0    2 14:30 ?        Ss     0:01 /sbin/init
    root         1     0     2  0    2 14:30 ?        Ss     0:00 /sbin/init
    user      1234  1233  1234  0    5 15:00 pts/0    Sl+    1:23 /usr/bin/myapp
    user      1234  1233  1235  0    5 15:00 pts/0    Sl+    0:45 /usr/bin/myapp
    user      5678  5677  5678  0    1 15:01 ?        S      0:00 /usr/sbin/sshd

该命令会显示所有进程及其线程信息。

- UID：进程所有者的用户 ID（如 root 或用户名）。
- PID：进程ID。同一进程的多个线程共享相同的 PID。
- PPID：父进程的 PID。
- LWP：线程的唯一标识符。主线程的 LWP 通常等于 PID，其他线程的 LWP 为独立值。
- C：CPU 占用率（百分比）。通常为瞬时值，可能与 top 数据略有差异。
- NLWP：进程包含的线程总数。例如，PID 为 1234 的进程有 5 个线程。
- STIME：进程启动时间（格式：小时:分钟）。
- TTY：关联的终端。?：无关联终端（通常是守护进程）。pts/0：伪终端（如 SSH 或终端窗口）。
- STAT：进程/线程状态码。
- TIME：累计使用的 CPU 时间（格式：分:秒）。
- CMD：启动进程的命令名称。线程通常显示与主进程相同的命令名。

## top

top 是一个实施监控工具，用于显示系统资源的使用情况，与 ps 命令不同的地方在于其显示的内容是动态更新的。

输入 top 之后，就会进入交互式的界面

    top

默认显示所有进程的信息。它会实时更新进程信息，默认每 3 秒刷新一次。

示例

    #top
    top - 14:06:23 up 70 days, 16:44,  2 users,  load average: 1.25, 1.32, 1.35
    Tasks: 206 total,   1 running, 205 sleeping,   0 stopped,   0 zombie
    Cpu(s):  5.9%us,  3.4%sy,  0.0%ni, 90.4%id,  0.0%wa,  0.0%hi,  0.2%si,  0.0%st
    Mem:  32949016k total, 14411180k used, 18537836k free,   169884k buffers
    Swap: 32764556k total,        0k used, 32764556k free,  3612636k cached

    PID   USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND                                                                
    28894 root      22   0 1501m 405m  10m S 52.2  1.3   2534:16 java                                                                   
    18249 root      18   0 3201m 1.9g  11m S 35.9  6.0 569:39.41 java                                                                          
    1     root      15   0 10368  684  572 S  0.0  0.0   1:30.85 init                                                                   
    2     root      RT  -5     0    0    0 S  0.0  0.0   0:01.01 migration/0  
    ...

系统摘要区

- 14:06:23 — 当前系统时间
- up 70 days, 16:44 — 系统已经运行了70天16小时44分钟（在这期间系统没有重启）
- 2 users — 当前有2个用户登录系统
- load average: 1.15, 1.42, 1.44 — load average后面的三个数分别是1分钟、5分钟、15分钟的负载情况。如果这个数除以逻辑CPU的数量，结果高于5的时候就表明系统在超负荷运转了。
- Tasks：显示总进程数及状态（运行、休眠、僵尸等）。
- Cpu(s)：
  - 5.9%us — 用户空间占用CPU的百分比。
  - 3.4% sy — 内核空间占用CPU的百分比。
  - 0.0% ni — 改变过优先级的进程占用CPU的百分比
  - 90.4% id — 空闲CPU百分比
  - 0.0% wa — IO等待占用CPU的百分比
  - 0.0% hi — 硬中断（Hardware IRQ）占用CPU的百分比
  - 0.2% si — 软中断（Software Interrupts）占用CPU的百分比
- Mem：内存状态
  - 32949016k total — 物理内存总量（32GB）
  - 14411180k used — 使用中的内存总量（14GB）
  - 18537836k free — 空闲内存总量（18GB）
  - 169884k buffers — 缓存的内存量 （169M）
- Swap：Swap交换分区信息
  - 32764556k total — 交换区总量（32GB）
  - 0k used — 使用的交换区总量（0K）
  - 32764556k free — 空闲交换区总量（32GB）
  - 3612636k cached — 缓冲的交换区总量（3.6GB）

使用中的内存总量（used）指的是现在系统内核控制的内存数，空闲内存总量（free）是内核还未纳入其管控范围的数量。纳入内核管理的内存不见得都在使用中，还包括过去使用过的现在可以被重复利用的内存，内核并不把这些可被重新使用的内存交还到 free 中去，因此在 Linux 上 free 内存会越来越少。

计算可用内存数，这里有个近似的计算公式：第四行的free + 第四行的buffers + 第五行的cached，按这个公式此台服务器的可用内存：18537836k +169884k +3612636k = 22GB左右。

对于内存监控，在 top 里要时刻监控第五行 Swap 交换分区的used，如果这个数值在不断的变化，说明内核在不断进行内存和 Swap 的数据交换，这是真正的内存不够用了。


进程状态监控区

- PID：进程id
- USER：进程所有者
- PR：进程优先级
- NI：nice值。负值表示高优先级，正值表示低优先级
- VIRT：进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
- RES：进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
- SHR：共享内存大小，单位kb
- S：进程状态。D=不可中断的睡眠状态 R=运行 S=睡眠 T=跟踪/停止 Z=僵尸进程
- %CPU：上次更新到现在的CPU时间占用百分比
- %MEM：进程使用的物理内存百分比
- TIME+：进程使用的CPU时间总计，单位1/100秒
- COMMAND：进程名称（命令名/命令行）

top 界面还有一些交互命令。

- P：按照 CPU 使用率进行排序
- M：按照内存使用率进行排序
- k：终止指定 PID 的进程（需输入 PID）
- H：切换进程/线程视图（显示线程时，PID 列变为线程 ID）
- q：退出 top

除了上述交互命令之外，top 还有一些选项

- -p PID：仅监控指定进程（如 top -p 1234）。
- -u USER：仅显示指定用户的进程。
- -H：默认显示线程（等同于交互模式下按 H 键）。
- -b：以批处理模式运行（适合输出到文件）。
- -n N：只显示 N 次之后就退出
- -d SEC：间隔 SEC 秒之后才刷新一次
 

在一些嵌入式的 Linux 系统中，有时 top 命令被阉割，不支持某些选项，比如不支持 -p，此时可以使用 grep 命令。例如

    top -H -n 1 | grep hik_app

    top -n 1 | grep hik_app

使用 grep 命令就可以获得和 -p 命令一样的效果。

## stat_mem

stat_mem 是一个用于监控和分析内存使用情况的工具，不是所有的 Linux 发行版都有这个内存监测工具，需要手动安装。

嵌入式Linux中，具体支持的选项，可以使用 

    stat_mem -h

结合 grep 命令可以监控指定进程的内存占用情况

    #stat_mem -m 3,1 | grep -Ei "demo|PID|dsp"
    PID         Process         Vss         Rss         Pss   Anonymous
    917            demo  1056784 kB   171516 kB   157946 kB     6232 kB
    923             dsp  3560476 kB   170576 kB   154893 kB   116992 kB
    PID         Process         Vss         Rss         Pss   Anonymous
    917            demo  1056784 kB   171516 kB   157946 kB     6232 kB
    923             dsp  3560476 kB   170576 kB   154891 kB   116992 kB


- PID：进程 ID，是操作系统为每个正在运行的进程分配的唯一标识符。
- Process：进程名称。
- Vss：虚拟内存大小，表示进程总共使用的虚拟内存空间，包括了进程本身的代码、数据、共享库以及被映射的文件等所占用的虚拟地址空间大小。
- Rss：驻留集大小，指的是进程当前实际占用的物理内存大小，也就是进程的页面已经被加载到物理内存中的部分。
- Pss：比例集大小，是一种更精确的内存使用度量方式，它会将共享内存按照进程实际使用的比例进行分摊，避免了在统计共享内存时的重复计算。
- Anonymous：匿名内存大小，通常指的是没有映射到文件的内存区域，例如动态分配的堆内存等。


注意 Anonymous 项，可以实时监控进程 malloc 的内存。可以观察到某个进程 malloc 的内存是否及时 free。若这个值一直变大，就有可能出现 OOM 现象。


## dmesg

dmesg 是用来查看内核日志的命令。

当系统发生崩溃时，如 OOM 或者 Undefined Signal 11 段错误，可以结合 grep 命令来进行过滤

    dmesg | grep -Ei "oom|Signal 11" > /tmp/1.log

上述命令中找出包含 oom 和 Signal 11 的行(不区分大小写)显示，并输出到 /tmp/1.log

## free 

free 用于显示系统的内存和交换分区的使用情况。

    #free
                  total        used        free      shared  buff/cache   available
    Mem:         517956      204520       33464      191232      279972       66220
    Swap:             0           0           0

- total：总内存或总交换分区大小。
- used：已使用的内存或交换分区大小。
- free：空闲的内存或交换分区大小。
- shared：共享使用的内存量（已在Linux 4.0及以后版本移除）
- buff/cache：用于缓存数据的内存量。
- available: 可用内存（包括空闲内存和可回收的缓存）。

free 还支持一些选项
- -h：以人类友好方式显示
- -m：以M字节显示
- -g：以G字节显示
- -s COUNT：指定刷新次数，用于实时监控内存使用

## /proc 目录

/proc 目录是 Linux 系统提供的一个虚拟文件系统（不占用实际存储空间），允许用户和程序获取系统信息、进程信息以及配置内核参数，为用户空间提供了访问内核数据的接口。

    #ls /proc 
    1               149             325             65              iomem
    10              150             33              7               ioports
    100             151             34              8               irq
    101             152             345             9               kallsyms
    102             153             346             900             key-users
    1024            154             35              907             keys
    103             155             351             908             kmsg
    104             156             352             909             kpagecgroup
    105             16              357             92              kpagecount
    106             17              358             927             kpageflags
    108             178             36              929             loadavg
    109             18              37              93              locks
    11              19              38              933             mdstat
    111             2               39              935             media-mem
    1118            20              40              936             meminfo
    112             20397           41              94              misc
    114             20470           42              945             modules
    1145            21              43              95              mounts
    115             21310           44              950             mtd
    117             21321           45              96              net
    118             21390           46              97              pagetypeinfo
    12              22              47              98              partitions
    123             22597           48              99              sata
    125             23              484             buddyinfo       sched_debug
    126             23534           49              bus             scsi
    127             24              5               cgroups         self
    128             24372           50              cmdline         slabinfo
    1288            24585           51              consoles        softirqs
    129             24884           52              cpinfo          stat
    13              26              53              cpuinfo         swaps
    130             27              54              crypto          sys
    1340            28              547             debug           syslink
    1356            29              548             device-tree     sysrq-trigger
    1389            3               55              devices         sysvipc
    1394            304             56              diskstats       thread-self
    1396            309             57              driver          timer_list
    1397            31              58              execdomains     tty
    14              310             59              fb              umap
    144             314             60              filesystems     uptime
    145             31431           61              fs              version
    146             32              62              i2s_info        vmallocinfo
    147             320             63              i2s_stereo_dev  vmstat
    148             324             64              interrupts      zoneinfo

包括以下信息

- 进程信息：每个进程对应一个以 PID 命名的目录，包含进程的详细信息。
  - maps：显示内存映射
  - status：进程状态，如内存使用情况。
  - task：子任务目录，记录线程信息。
  - oom_score：进程被杀的可能性评分。

        #ls /proc/1
        auxv             fd               net              setgroups
        cgroup           fdinfo           ns               smaps
        clear_refs       gid_map          oom_adj          stat
        cmdline          limits           oom_score        statm
        comm             map_files        oom_score_adj    status
        coredump_filter  maps             pagemap          syscall
        cpuset           mem              personality      task
        cwd              mountinfo        projid_map       timerslack_ns
        environ          mounts           root             uid_map
        exe              mountstats       sched            wchan
- 系统信息
  - /proc/version：显示内核版本和编译信息。
  - /proc/cpuinfo：详细CPU信息，如型号和功能。
  - /proc/meminfo：内存使用情况。
  - /proc/loadavg：系统负载平均值。
- 设备和文件系统信息
  - /proc/partitions：磁盘分区信息。
  - /proc/diskstats：块设备I/O统计数据。
- 网络信息：
  - /proc/net/dev：网络设备状态。
  - /proc/net/tcp：TCP连接状态。

通过前面的 top 或者 ps 命令获取到某个进程的 PID，就可以利用 ls /proc/PID/fd 查看当前进程建立的socket 句柄。

### /proc/meminfo

使用 cat /proc/meminfo 查看内存信息

    # cat /proc/meminfo
    MemTotal:         517956 kB  #所有可用RAM大小（即物理内存减去一些预留位和内核的二进制代码大小）,(HighTotal + LowTotal),系统从加电开始到引导完成，BIOS等要保留一些内存，内核要保留一些内存，最后剩下可供系统支配的内存就是MemTotal。这个值在系统运行期间一般是固定不变的。
    MemFree:           32896 kB  #LowFree 与 HighFree的总和，被系统留着未使用的内存,MemFree是说的系统层面
    MemAvailable:      65628 kB  #应用程序可用内存数。系统中有些内存虽然已被使用但是可以回收的，比如cache/buffer、slab都有一部分可以回收，所以MemFree不能代表全部可用的内存，这部分可回收的内存加上MemFree才是系统可用的内存，即：MemAvailable≈MemFree+Buffers+Cached，它是内核使用特定的算法计算出来的，是一个估计,MemAvailable是说的应用程序层面
    Buffers:           17256 kB  #用来给文件做缓冲大小
    Cached:           249016 kB  #被高速缓冲存储器（cache memory）用的内存的大小（等于 diskcache minus SwapCache ）
    SwapCached:            0 kB  #被高速缓冲存储器（cache memory）用的交换空间的大小，
    已经被交换出来的内存，但仍然被存放在swapfile中。用来在需要的时候很快的被替换而不需要再次打开I/O端口
    Active:           195820 kB
    Inactive:         220860 kB
    Active(anon):     159904 kB
    Inactive(anon):   185820 kB
    Active(file):      35916 kB
    Inactive(file):    35040 kB
    Unevictable:        4092 kB
    Mlocked:               0 kB
    SwapTotal:             0 kB
    SwapFree:              0 kB
    Dirty:                 0 kB
    Writeback:             0 kB
    AnonPages:        154616 kB
    Mapped:           212552 kB
    Shmem:            191232 kB
    Slab:              38292 kB
    SReclaimable:      13660 kB
    SUnreclaim:        24632 kB
    KernelStack:        6784 kB
    PageTables:         2428 kB
    NFS_Unstable:          0 kB
    Bounce:                0 kB
    WritebackTmp:          0 kB
    CommitLimit:      492056 kB
    Committed_AS:     718372 kB
    VmallocTotal:   263061440 kB
    VmallocUsed:           0 kB
    VmallocChunk:          0 kB
    ...（嵌入式Linux往往会阉割一些选项）


MemFree 表示的是当前系统中完全未使用的、空闲的物理内存数量。这些内存没有被任何进程或内核使用，可以立即用于新的内存分配请求。MemFree 通常可以用于 malloc，但系统可能会优先使用缓存内存以提高效率。

MemAvailable 表示系统中可以立即分配给应用程序使用的内存总量，包括当前空闲的内存以及可以被回收的缓存内存


