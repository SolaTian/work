# Linux查看系统各个进程已经建立的socket句柄

输入`top`或者`ps -T`查看系统中正在运行的进程的`PID`

之后

    cd /proc/pid/

其中`pid`表示的就是通过`top`或者`ps -T`查看道德进程`PID`

之后

    ls fd

就可以查看当前系统建立的socket句柄号了
