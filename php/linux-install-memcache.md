+++
title = "Linux 安装 Memcached 及 PHP Memcached 扩展"
author = "BroQiang"
github_url = "https://broqiang.com"
head_img = ""
created_at = 2017-06-07T02:10:41
updated_at = 2017-06-07T02:10:41
description = "记录了 Linux 下  Memcached 的安装、命令以及 PHP Memcached 扩展的使用"
tags = ["linux", "php"]
+++

## 为什么要学习 Memcahe

首先我们要知道 Memcache 是做什么的，为什么需要学习 Memcache

### Memcache 是什么？

首先我们看下 [官方](https://memcached.org/about) 的介绍

`memcached is a high-performance, distributed memory object caching system`

高性能，分布式内存对象缓存系统，用于通过减轻数据库负载来加速动态Web应用程序

我们可以看到他的定义，前面的都是修饰性的，也可以说是 Memcache 的优点，简单的说它就是一个缓存系统

就是将数据缓存到内存当中，可以通过内存来访问数据，我们如果对计算机的构造了解就应该知道，直接访问内存中的数据，和在数据库中查询数据，就完全不是一个概念了

## Memcahced 安装

[官网](https://memcached.org)

[下载地址](https://memcached.org/files/memcached-1.4.37.tar.gz)

### 安装规划

#### CPU 需求

Memcached 对CPU的使用很轻，它是多线程工作的，默认为4个工作线程，一般服务器的CPU都可以满足 Memcached

#### 内存需求

- 和 Web 应用放在一起，这种方式就是通常就是 将应用和缓存放在同一台服务器

    这时就要考虑应用的负载，比如：

    服务器是4G内存，LNMP环境消耗了2G内存，此时就建议 Memcached 分配1.5G内存

    因为如果分配过高，他就会用到 Swap ，Swap 是将内存保存到磁盘上，反而会降低缓存的效率

    这种情况是非常尴尬的，所以缓存并非越大越好，看实际的应用和硬件的配置

- 单独部署缓存服务器

    * 此种方式部署，此时就不需要担心负载问题，可以将大量的内存放在缓存服务器中，Memcached 软件本身只会消耗很少的内存

    * 不过此种方式需要注意的是服务器间的网络通讯，大多数每个实例不会超过 10Mbps

        不过最好是通过光纤通道，用hba卡将服务器在内网中连接，如果是异地部署的话，要保证足够的带宽

- 集群部署

    集群部署的好处就是可以降低缓存的丢失带来的痛苦，比如你有10个Memcached服务器，其中一个宕机了，就只会丢失10%的缓存

    但是如果你只有两台缓存服务器，一但缓存宕机，那就会丢失50%，当然，只有一台缓存服务器的时候，宕机就会更糟糕了

### 源码编译安装

```bash
# 下载安装包
wget https://memcached.org/files/memcached-1.4.37.tar.gz

# 安装依赖关系
sudo apt install libevent-dev

# 解压缩，并进入到目录
sudo tar xzvf memcached-1.4.37.tar.gz -C /usr/local/src/
cd /usr/local/src/memcached-1.4.37

# 编译安装
sudo ./configure --prefix=/usr/local/memcached
sudo make
sudo make install

```

### 配置环境变量

在 /etc/profile.d/下新建 `memcached.sh` 文件，用来专门保存Memcached的配置，写入下面内容

```bash
export MEMCACHED_HOME=/usr/local/memcached
export PATH=$PATH:$MEMCACHED_HOME/bin
```

执行 source 或 . 使配置生效

```bash
source /etc/profile.d/memcached.sh
# 或
. /etc/profile.d/memcached.sh
```

### 服务器端的配置

启动服务

需要注意几个参数，这里只列出主要的常用的参数，额外的可以通过 memcached -h 去查询

- `-m` Memcached 进程使用多少内存，单位是 M

- `-d` Memcached 以守护进程的方式执行，如果是通过初始化脚本启动的，可以不用考虑这个参数

- `-p` 监听TCP端口，默认是11211，如果需要启动多个实例，就将下一个服务的端口设置成11211以外的其他端口，如11212

- `-l` 指定监听的IP，默认是 0.0.0.0

    如果你的防火墙没有限制的情况下，Memcached 将对外暴露

    如果你是将Memcached 和web应用部署到同一台服务器，可以将监听IP设置成 127.0.0.1

    如果是一个集群，就将监听配置成私网IP，如果必须使用公网IP的时候，要设置好防火墙的权限

- `-U` UDP 端口，默认是开启，端口是 udp 11211，不过建议将其禁用，将端口设置为 0 ，就可以关闭

```bash
# 下面的配置就是Memcached 最大内存1024M，端口11211，监听地址是127.0.0.1 后台执行
memcached -m 1024 -p 11211 -l 127.0.0.1 -d
# 后台执行后要想停用，可以killall memcached ，将所有的Memcached实例全都杀死
killall memcached
# 或者 kill 指定 pid
```

### 额外的说明

- 线程 Memcached 每一个线程都可以处理并发链接，libevent 可以通过并发链接实现良好的可以伸缩性，因此每个线程都能处理许多客户端

    和否写web服务器不同，如 apache，每个活动的客户端连接就要使用一个进程或线程

    默认情况下配置4个线程，除非运行的 Memcached 非常吃力，一般不应该修改这个配置，4个足够

    当线程数超过80的时候，将使得Memcached运行会非常的慢

- 连接限制

    默认的，Memcached 最大连接数会被设置成1024个，如果超出这个数量的客户端连接，将会挂起，等待其他连接释放后才会连接

    因此配置完成后要根据实际情况去配置这个值，一般情况下这个配置足够了，当然高并发的时候就要适当的调整了

    这个一般根据web服务提供的最大连接数去配置

### 查看正在运行的配置

```bash
echo "stats settings" | nc 127.0.0.1 11211
```

### 启动脚本

我们将源码包中的 script 目录中的 启动脚本复制到 系统的启动脚本中

```bash
sudo cp /usr/local/src/memcached-1.4.37/scripts/memcached.service /lib/systemd/system/
# 我们要根据实际的软件安装情况修改几个配置
# 1. 修改下默认的配置文件的位置，将下面的配置修改
EnvironmentFile=/etc/sysconfig/memcached
# 修改成
EnvironmentFile=/usr/local/memcached/etc/memcached.conf

# 2. 将 启动文件的位置修改
ExecStart=/usr/bin/memcached -p ${PORT} -u ${USER} -m ${CACHESIZE} -c ${MAXCONN} $OPTIONS
# 修改成
ExecStart=/usr/local/memcached/bin/memcached -p ${PORT} -u ${USER} -m ${CACHESIZE} -c ${MAXCONN} $OPTIONS

```

然后我们在 我们刚刚修改的配置文件 /usr/local/memcached/etc/memcached.conf 中写入下面配置，来定义配置文件

```bash
PORT=11211

USER=root

CACHESIZE=1024

MAXCONN=1024

OPTIONS="-l 127.0.0.1 -U 0"

```

## Memcached 客户端

telnet 的连接，并测试

### 系统命令

```bash
# 连接
telnet 127.0.0.1 11211
# 查看版本
version
# 查看状态，后面可以跟 settings/items/sizes/slabs 这些参数，可以展示出不同的结果
stats

# 清空所有缓存，后面可以添加时间，表示在指定时间之后再清理缓存
flush_all
```

### 存储命令

#### 格式：

```bash
# 命令及参数
<command> <key> <flags> <exptime> <bytes> [<version>]
# 数据
<datablock>
# 返回的状态
<status>
```

#### 参数说明：

- `command` 可以使用的命令，包括 `set/add/replace/append/prepend/cas`

    * add 只有数据不存在时进行添加，如果存在就会提示NOT_STORED

    * set 不管是否存在都会添加，并且会将之前的值覆盖

    * replace 只会替换已经存在的key

    * append 在原本的key对应的值后面追加字符，并不会破坏原本的超时时间

    * prepend 在一个 key 的 value 前面追加字符，并不会破坏原本的超时时间

    * cas 即check and set，只有版本号相匹配时才能存储，否则返回EXISTS

        在后面的gets时一起说

- `key` 存储 key 的名字，长度为250个字符，不包含空格和控制字符

- `flags` 是一个16位的无符号的整数(以十进制的方式表示)。

    该标志将和需要存储的数据一起存储,并在客户端get数据时返回，

    就是Memcached提供的一个额外标记的地方，可以根据实际需要去使用

- `exptime` 过期时间，单位为秒，0为永远，表示用不过期，如果过期时间超过30天，就要用unix时间戳表示

- `bytes` byte字节数，不包含\r\n，根据长度截取存/取的字符串，可以是0，即存空串

- `version` 版本号，可选的，可不写，这个字断主要是给cas命令用，每次set key为新的value后，version都会变化

- `datablock` 储存的内容value，以\r\n结尾，可以包含\r或\n

    支持`serialize`格式的数据，这样我们通过PHP手册就可以知道PHP都能支持什么格式的数据

- `status` STORED/NOT_STORED/EXISTS/NOT_FOUND/ERROR/CLIENT_ERROR/SERVER_ERROR服务端返回的状态标志

实例：

我们先不考虑flag，等到PHP部分再详细说明用途，暂时就随便用一个整数去设置，比如16

```bash
# 设置 name 为5个字符，过期时间不限制
set name 16 0 5
# 写入名字 zhang
# 可以试验下名字超出5个或者不到5个，看看出现什么错误了

# add 命令，先尝试添加name，然后再尝试添加个不存在的 id
add name 16 0 5

# 添加 id
add id 16 0 5

# 替换id
replace id 16 0 10

```

### 读取命令

#### 读取命令格式：

```bash
get/gets key1 key2 ...keyn
```

#### 说明：

- `get` 根据 key 获取到对应的值

- `gets` 和get相同，比get会多flag、bytes和[version] 信息

#### 实例：

```bash
# 获取name和id的值
get name id
# 获取name和id的值，并有详细信息
gets name id
# 给一个已经存在的key设置新的value，但是它会先检查 version和它gets的version是否一致，如果是一致的，则会生效，否则会报错 EXISTS。
cas name 16 0 5 13
```

### 删除命令

格式：

```bash
# 这个很简单的，加上key即可，不能批量删除
delete key
```

### 计数命令

#### 用法：

```bash
incr key number
decr key number
```

说明： 用来对value为int数字型的key进行加n或者减n操作，直接看实例，如果操作的是字符串将会报错

#### 计数命令实例：

```bash
# 给刚才的id增加，先看下id现在的值，然后再看加完之后的结果，再看减少之后的结果
get id
incr id 1

# 减少，当结果小于等于0的时候一直返回0
decr id 2
```

## Memcached 内存管理机制

通过前面的讲解，已经对Memcached 有一些概念了，这里再讲解他的存储机制更容易理解一下

可以理解成这里是一些理论性的东西

### slab/chunk

大家应该都用过excel，我们可以用excel表格去解释下，经更容易理解了

memcached 的内存空间有很多个slab（对应excel就是sheet），每一页又有很多个 trunk(对应excel就是单元格)

单个slab最大是值1M

### item

memcached 会把我们写入的 value 打包成 item，并保存到trunk中，一个item保存到一个trunk中

根据上面的单个slab最大值是1M，value又会打包成item，所以我们的value最大不能超过1M

严格的说是key+value+flags不能超过1M，key是不能超过250个字节，所以计算数据能否保存的时候要将他减去

如果超过1M怎么办？

- 可以去修改Memcached的源码，将 POWER_BLOCK 常量设置更大，然后重新编译

- 替换其他的缓存系统，这个是个人比较推荐的，不建议去修改源码，因为这样稳定性不能保证，现在市面上有很多中缓存系统，要选用适合的

### slabclass

我们往chunk中塞item的时候，item总不可能会与chunk的大小完全匹配吧，chunk太小塞不下或者chunk太大浪费了怎么办？

就是我们在像单元格填充数据时的时候，格子太小，出界了，或者我们的内容很少在一个大格子里面好浪费。

所以memcached的设计是，我们会准备“几种不同格子的slab”，也就是说根据“slab分割的chunk的大小不一样”来分成“不同的种类的slab”，而 slabclass就是“slab的种类”的意思了

### trunk大小的分配

我们可以在启动服务器的时候通过-vv参数查看默认的Trunk 分配

可以通过 -n 和 -f 参数来设置默认的 trunk 大小

- `-n` 允许分配的最小空间，默认是48个字节，设置后的最小空间就是48+设置的值

- `-f` trunk 空间增长的步长，默认是1.25倍

实例：

```bash
# 查看默认的大小及数量
memcached -vv

# 自定义 -n 和 -f 参数，观察下结果
memcached -n 40 -f 2 -vv
```

### 内存分配申请

向memcached添加一个item时候，memcached首先会根据item的大小，来选择最合适的slabclass。

例如要插入 10字节的一个 item，默认情况下 slab class 1 的 chunk 大小为96字节，因此是可以放到在class1中，96-10 = 86 个字节的空间就会被浪费了(先不考虑key和flag占用的)，memcached 会去检查 slab class 1 大小的 chunk 还有没有空闲的，如果没有，将会申请新的 slab class1 空间并划分为该种类chunk。

例如我们第一次向memcached中放入一个 小于 96 字节的item时，memcached会使用slab class1，并会用去一个chunk，剩余 10921 个chunk供下次有适合大小item时使用，当我们用完这所有的chunk之后，下次再有一个在96字节以下的item添加进来时，memcached会再申请空间生成一个class1 slab（这样就存在了2个slab class1）。

### 删除策略

memcached是懒检测机制，当存储在内存中的对象过期甚至是 flush_all 时 ，它并不会做检查或删除操作，只有在get时才检查数据对象是否应该删除。

删除数据时，Memcached同样是懒删除机制，只在对应的数据对象上做删除标识并不回收内存，在下次分配时直接覆盖使用。所以，当memcached过期或者删除的时候，它所占用的内存并不会释放，除非将服务重新启动。

当Memcached 没有空间保存新的数据时会怎么处理？会自动覆盖最后一次使用时间最早的数据，用来保存最新的数据

## Memcached 集群

由于Memcached服务器与服务器之间没有任何通讯，并且不进行任何数据复制备份，所以当任何服务器节点出现故障时，会出现单点故障，如果需要实现HA，则需要通过另外的方式来解决。

通过Magent缓存代理，防止单点现象，缓存代理也可以做备份，通过客户端连接到缓存代理服务器，缓存代理服务器连接缓存连接服务器，缓存代理服务器可以连接多台Memcached机器可以将每台Memcached机器进行数据同步。

如果其中一台缓存服务器down机，系统依然可以继续工作，如果其中一台Memcached机器down掉，数据不会丢失并且可以保证数据的完整性。

由于章节和环境的关系，此处就不说明集群环境，具体可以参考：[https://code.google.com/p/memagent](https://code.google.com/p/memagent)

## PHP 操作 Memcached

我们上面这么多篇幅的内容就是为了这里做准备的

通过上面的内容，已经了解了 Memcached 到底是怎么回事了，下面我们再看PHP怎么去使用 Memcached 就会很轻松了

### PHP 的Memcached 扩展

PHP 有两个关于Memcache的扩展 [Memcache](https://php.net/manual/zh/book.memcache.php) 和 [Memcached](https://php.net/manual/zh/intro.memcached.php)，他们有什么区别呢?

- [Memcache](https://php.net/manual/zh/book.memcache.php)

	php memcache独立用php实现，是老客户端，实际应用中已发现有多个问题，而且功能少

- [Memcached](https://php.net/manual/zh/book.memcached.php)

	基于原生的libmemcached的扩展，更加完善，建议使用 memcached，后面的讲解也是按照此扩展进行

Memcached 的扩展，对应 Memcached 本身来说，就是一个客户端，和我们之前用 telnet 连接操作是相同的，只是此处把 telnet 换成了 PHP

### PHP 的Memcached 扩展安装

到 [pecl](https://pecl.php.net) 下载 Memcached 扩展包

```bash
# 安装 libmemcached 客户端 ，因为 Memcached 扩展需要 libmemcached 客户端支持
sudo apt install libmemcached-dev

# 安装依赖关系，HP 扩展需要 libmemcached 必须启用sasl，默认就是启用
# 但是要确保服务器安装了sasl包，configure 之后一定要注意看下，是否有 SASL support: yes 这个信息
sudo apt install libsasl2-dev

# 下载安装包
wget https://pecl.php.net/get/memcached-3.0.3.tgz

# 因为是PHP的扩展，所以解压到 PHP 的源码包的 ext 目录中
tar xzvf memcached-3.0.3.tgz -C /usr/local/src/php-7.1.4/ext/memcached-3.0.3
/usr/local/src/php-7.1.4/ext/memcached-3.0.3

# 通过  phpize 生成 configure 文件，phpize是专门用来准备 PHP 扩展库的编译环境的工具
sudo /usr/local/php/bin/phpize

# 编译安装
sudo ./configure --with-php-config=/usr/local/php/bin/php-config  --with-zlib-dir
sudo make
sudo make install

# 可以看到，他将 Memcached 扩展安装到了下面的位置
/usr/local/php/lib/php/extensions/no-debug-non-zts-20160303/

# 修改 php.ini 加载memcached扩展
sudo vim /usr/local/php/etc/php.ini
# 在最下面写入
extension=memcached.so

# 测试是否生效
php -i | grep memcached
```

### 代码演示

[官方文档](https://php.net/manual/zh/book.memcached.php)