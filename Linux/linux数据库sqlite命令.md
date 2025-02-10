# Linux下执行sqlite3

在Linux系统下执行`sqlite3`命令，确保`/bin/`目录下有`sqlite3`二进制文件，且权限正确，若没有权限则赋权限。

    chmod -R 777 sqlite3

## 进入sqlite3操作界面

`sqlite3`需要到对应硬盘的目录下。

    sqlite3 /dev/sdx

`dev/sdx`表示硬盘对应的挂载文件名，可以是`sda`，也可以是`sdb`等等

进入下面的界面

    SQLite version 3.18.0 2017-03-28 18:48:43
    Enter ".help" for usage hints.
    sqlite>

## 执行查询过滤命令

这里只介绍常用的几个命令

    .help               #查看当前sqlite3支持的命令
    .tables             #查看所有数据库的表
    .exit               #退出
    .quit               #退出

查询命令

    select * from tablename limit 5;         #查询table中最前面5条的数据
    select coloum1 from tablename limit 5;    #查询table中最前面5条的coloum1列信息
    select * from tablename desc limit 5;     #查询table中最后面5条信息
    select * from tablename order by coloum1 desc limit 5; #查询table中按照coloum1排序的最后5条信息
    select * from tablename where coloum1 = 'xxxx';         #按照coloum1列进行过滤，字符串过滤用单引号。

