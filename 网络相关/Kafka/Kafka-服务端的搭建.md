
[参考链接](https://zhuanlan.zhihu.com/p/666793036)

# 准备工作




1. VMWare 软件下载（GitHub网址，包括注册码）
2. 安装Ubuntu（镜像下载网站https://blog.csdn.net/weixin_63175492/article/details/137267410,安装教程：https://blog.csdn.net/m0_73641365/article/details/140307987）
3. 安装VMWare Tools(实现电脑和虚拟机的复制粘贴。https://blog.csdn.net/2301_76587520/article/details/141121451)
4. 下载jdk(安装网址：https://www.oracle.com/java/technologies/javase/jdk11-archive-downloads.html)
5. 下载zookeeper（安装网址：https://zookeeper.apache.org/releases.html）
6. 下载kafka（安装网址：https://kafka.apache.org/downloads）

## 安装Java（虚拟机内安装：将jdk压缩包解压缩，然后修改环境变量）

        cd /usr/local
        ls -lh jdk-11.0.20_linux-x64_bin.tar.gz
        tar -xf jdk-11.0.20_linux-x64_bin.tar.gz 
        sudo bash -c 'cat >> /etc/profile << EOF
        export JAVA_HOME=/usr/local/jdk-11.0.23
        export PATH=\$JAVA_HOME/bin:\$PATH
        export CLASSPATH=.:\$JAVA_HOME/lib:\$CLASSPATH
        EOF'

![alt text](image.png)

这是jdk的版本不匹配

在终端中输入

    uname -m

查看系统的硬件架构

![alt text](image-2.png)

系统是32位架构，而安装的jdk是64位架构的，需要重新安装jdk，之前安装的是Oracle JDK，需要改为安装OpenJDK(当然也可以安装64位的Ubuntu了hhhhh，图简单，就重新安装个32位的jdk)


直接在终端里面安装openjdk

    sudo apt update
    sudo apt install opensdk-8-jdk

好像还是不行，安装的时候报错

![alt text](image-3.png)

这个报错表示连接不到这个源，由于网络连接问题导致的下载失败。


可以尝试以下几种方式。

1. 检查网络连接
        
        ping 域名或者IP

2. 更新软件包列表

        sudo apt-get update
3. 使用提示的 `--fix-missing`选项
4. 更换软件源，编辑`/etc/apt/sources.list`文件
   
        sudo nano /etc/apt/sources.list
将`http://security.ubuntu.com/ubuntu`替换为其他可用的镜像源，例如

        deb http://mirrors.aliyun.com/ubuntu/ xenial main restricted universe multiverse
        deb http://mirrors.aliyun.com/ubuntu/ xenial-security main restricted universe multiverse
        deb http://mirrors.aliyun.com/ubuntu/ xenial-updates main restricted universe multiverse
        deb http://mirrors.aliyun.com/ubuntu/ xenial-backports main restricted universe multiverse

重新下载32位的jdk，并安装，重新配置环境变量，更新`/etc/profile`文件，执行`source /etc/profile`

![alt text](image-4.png)



## 安装配置zookeeper

将下载的zookeeper解压到/usr/local/zookeeper/目录下

    cd /usr/local/
    sudo mkdir zookeeper
    sudo mv /home/qintian/文档/apache-zookeeper-3.8.4-bin.tar.gz /usr/local/zookeeper/

    ls -lh apache-zookeeper-3.8.3-bin.tar.gz
    tar -xf apache-zookeeper-3.8.3-bin.tar.gz 
    ls -l apache-zookeeper-3.8.3-bin/
    --准备日志和数据目录
    mkdir -p apache-zookeeper-3.8.3-bin/data apache-zookeeper-3.8.3-bin/logs

![alt text](image-5.png)


安装好了之后开始配置

首先查看zookeeper的样本配置文件zoo_sample.cfg

![alt text](image-6.png)

里面介绍了一些重要的配置项及其含义

- tickTime
- initLimit
- syncLimit
- dataDir

真正在使用的时候需要将文件重命名成zoo.cfg文件，下面开始配置自己的zoo.cfg文件

    cat > apache-zookeeper-3.8.3-bin/conf/zoo.cfg <<EOF
    tickTime=2000
    initLimit=10
    syncLimit=5
    dataLogDir=/usr/local/apache-zookeeper-3.8.3-bin/logs
    dataDir=/usr/local/apache-zookeeper-3.8.3-bin/data
    clientPort=2181
    server.1=192.3.65.201:2888:3888
    server.2=192.3.65.202:2888:3888
    server.3=192.3.65.203:2888:3888
    EOF

配置完成之后为当前虚拟机配置`myid`

        sudo bash -c 'echo "1" > /usr/local/zookeeper/apache-zookeeper-3.8.4-bin/data/myid'

这个 myid 文件用来标识每个 ZooKeeper 节点的 ID。


以上执行完成之后，启动zookeeper集群。

        sudo ./zkServer.sh start

执行之后这样报错，找不到java的环境，之前在安装java的时候其实已经设置好了环境变量，并`source /etc/profile`了，现在使用`sudo -E ./zkServer.sh start`继承保留当前用户的环境变量

![alt text](image-7.png)

初次可以启用zookeeper，但是后续启动zookeeper失败，查看日志是因为java的版本太低了，导致的与zookeeper不兼容。

zookeeper3.8.4需要和jdk8以上的版本才能适配

![alt text](image-8.png)


重新下载jdk1.8，重新运行zookeeper成功，查看zookeeper状态

![alt text](image-9.png)


因为只有一个虚拟机，所以这个就易standalone的模式运行了


安装好两个zookeeper并启用
10.11.97.55是follower

![alt text](image-10.png)

10.11.97.56是Leader

![alt text](image-12.png)

## 安装配置kafka

安装配置好zookeeper之后，就需要安装配置kafka了

        cd /usr/local
        ls -lh kafka_2.13-2.7.2.tgz
        tar -zxf kafka_2.13-2.7.2.tgz 
        mkdir -p kafka_2.13-2.7.2/logs

开始配置kafka

![alt text](image-14.png)

启动kafka

![alt text](image-13.png)

启动kafka之前先执行一次stop停止kafka进程，防止直接运行会报错显示有kafka正在运行，日志`server.log`会报错。
![alt text](image-16.png)

kafka日志每隔一个小时会存一个副本

leader运行成功日志如下：

![alt text](image-15.png)

也可以用终端连接这个kafka的IP和端口，看是否通

follower运行成功日志如下：

![alt text](image-17.png)

可以看到，虽然follower能正常起来，但是和leader（10.11.97.56）之间无法正常建立连接。直接在命令行里面ping可以ping 通。

如果在启用follower时，出现以下报错，可以检查以下myid与kafka中的brokerid是否一致，如果不一致，可能会导致follower和leader之间不互通，我这边修改之后，leader和follower之间是可以互通的如下图

leader最终
![alt text](image-19.png)

follower最终

![alt text](image-18.png)


因为执行kafka的时候，用的是-daemon，表示在后台运行，所以使用ps命令是查不到kafka进程的。


## 配置topic

只需要在leader上，创建topic即可

![alt text](image-21.png)

注意就是，replication不能超过broker数量

在leader上创建topic之后，可以在follower里面直接--list查看，如下
 
![alt text](image-22.png)

## 配置SASL/PLAIN认证

![alt text](image-23.png)

回到`zookeeper`，执行`zkCli.sh`脚本

这段命令的整体流程是：

- 连接到 ZooKeeper。
- 创建一个新的节点（/digest）并存储数据 user。
- 添加认证信息（用户名 admin 和密码 abc12345），确保只有经过认证的用户可以访问相关节点。
- 设置该节点的访问控制，以限制和定义未来对该节点的访问权限。
- 退出 ZooKeeper 客户端。

这些操作通常用于设置更安全的 ZooKeeper 环境，以便管理和控制对特定数据的访问。

同样也是只需要在一个节点上执行这段命令即可，不需要在每个节点上执行。

修改好之后需要在几台服务器上分别修改配置

        cp kafka_2.13-2.7.2/config/server.properties kafka_2.13-2.7.2/config/server-sasl.properties
        # vi kafka_2.13-2.7.2/config/server-sasl.properties 
        --修改listener中PLIANTEXT为SASL_PLAINTEXT
        #listeners=PLAINTEXT://192.3.65.201:9092
        listeners=SASL_PLAINTEXT://:9092

        --增加如下内容指定认证方式
        authorizer.class.name=kafka.security.auth.SimpleAclAuthorizer
        security.inter.broker.protocol=SASL_PLAINTEXT
        sasl.mechanism.inter.broker.protocol=PLAIN
        sasl.enabled.mechanisms=PLAIN

        --指定超级管理员用户
        super.users=User:admin

之后需要在3台服务器上同时配置jaas文件，并指定用户名和密码

        cat >>  kafka_2.13-2.7.2/config/kafka-server-jaas.conf <<EOF
        KafkaServer {
                org.apache.kafka.common.security.plain.PlainLoginModule required
                username="admin"
                password="admin2023"
                user_admin="admin2023";
        };
        EOF

三台服务器均需要执行

        --停止服务器
        # kafka_2.13-2.7.2/bin/kafka-server-stop.sh 

        --修改启动脚本
        # cp kafka_2.13-2.7.2/bin/kafka-server-start.sh kafka_2.13-2.7.2/bin/kafka-server-start-sasl.sh 
        # vi kafka_2.13-2.7.2/bin/kafka-server-start-sasl.sh 
        exec $base_dir/kafka-run-class.sh $EXTRA_ARGS kafka.Kafka "$@"
        -->改为
        exec $base_dir/kafka-run-class.sh $EXTRA_ARGS -Djava.security.auth.login.config=/usr/local/kafka_2.13-2.7.2/config/kafka-server-jaas.conf kafka.Kafka "$@"

然后执行，因为kafka的版本太新，导致找不到那个类，kafka运行失败
或者换类名，或者换成旧的kafka类

