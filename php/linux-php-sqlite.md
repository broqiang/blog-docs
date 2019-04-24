+++
title = "Linux 和 PHP 中 SQLite 简单应用"
author = "BroQiang"
github_url = "https://broqiang.com"
head_img = ""
created_at = 2017-04-19T02:19:36
updated_at = 2017-04-19T02:19:36
description = "记录了下 Linux 的 SQLite 的安装和在 Linux 及 PHP 中的简单应用。"
tags = ["linux", "php"]
+++

SQLite 是一个嵌入式SQL数据库引擎. 与大多数其他SQL数据库不同,SQLite没有单独的服务器进程. SQLite直接读取和写入普通磁盘文件.

具有多个表,索引,触发器和视图的完整SQL数据库包含在单个磁盘文件中.

根据 SQLite 的简介, 大概知道什么时候使用他. 一般就是数据量比较小,访问数据库也不是很频繁,又不想单独启动数据库服务,可以选择它

## Linux 下安装

```bash
# Ubuntu 下使用下面命令安装, 一般默认会安装
$ sudo apt install -y sqlite3

# CentOS 下使用下面命令安装, 一般默认会已经安装
$ sudo yum install sqlite
```

## 创建数据

```bash
# Ubuntu 下sqlite3 去创建, Ubuntu 默认安装的sqlite3
$ sqlite3 ~/www/temp/test.sq3

# CentOS 下使用sqlite
$ sqlite ~/www/temp/test.sq3

# 如果不加路径就在当前目录创建, 和新建一个文件类似
# 后缀名理论上来说随意,不过最好起个叫人一看就知道是干什么的名字
# 创建完成就会进入一个下面界面, 可以在此处执行sql命令了
# 输入.help, 可以查询基本的使用
sqlite> .help
```

## 常用命令

语法很多,此处只列出了比较常用的简单操作, 实际应用中有问题可以去 [官方文档](https://www.sqlite.org/lang.html) 查看

```bash
# -- 查询当前数据库信息
sqlite> .database

# -- 查询当前数据库下的所有表的信息
# 可选参数 ? table name , 如果加上这个参数, 会按照输入的表面去显示, 支持 LIKE 方式查找
sqlite> .tables
sqlite> .tables %persion%

# -- 创建一个表
sqlite> CREATE TABLE persions (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        name text,
        age int);

# -- 删除一个表
sqlite> drop table persions;

# -- 修改表名
sqlite> ALTER TABLE persions RENAME TO persions2;
# 查看修改结果
sqlite> .tables
# 再改回去,不要影响后面的命令
sqlite> ALTER TABLE persions2 RENAME TO persions;

# -- 查看表结构
sqlite> pragma table_info ('persions');

# -- 修改字段, SQLite 不能像主流数据库那样直接修改字段, 只能添加一个新的字段
#    如果必须要修改,可以先将表重命名一个其他的,再将数据插入即可
# 添加新字段
sqlite> ALTER TABLE persions ADD COLUMN address text;

# -- 添加数据, 规则和 MYSQL 很类似
# 方式1
sqlite> INSERT INTO persions(name,age,address) VALUES ('张三',25,'北京');
# 方式2 不推荐,如果使用这种方式, 就要字段和表的字段数量完全相同
sqlite> INSERT INTO persions VALUES (2,'李四',30,'天津');
# 方式3 一般不会用,向表中插入默认值,如果没有指定默认值,就写入 NULL
sqlite> INSERT INTO persions DEFAULT VALUES;

# -- 更新数据
sqlite> UPDATE persions SET name='李四',age=22,address='上海' WHERE id=3;

# -- 查询数据
sqlite> SELECT id,name,age,address FROM persions;

# -- 删除数据
sqlite> DELETE FROM persions WHERE id=3;

```

## PHP 中使用 SQLite

此处只是一个简单的说明, 只是为了描述清楚 Sqlite 怎样使用, 实际项目中要写规范的代码

更多用法,参考 [官方 PDO 文档](https://php.net/manual/zh/book.pdo.php)

推荐数据库操作使用 PDO 方式, 这样底层数据库更换,应用层不需要重新设计

```php

// 创建一个专门处理 SQLite 的 PDO类
class Sqlite
{
    protected $pdo;

    public function __construct($pdo = null)
    {
        // 创建 PDO_SQLITE DSN
        if ($pdo) {
            $this->pdo = $pdo;
        } else {
            try {
                $this->pdo = new PDO('sqlite:/home/bro/www/temp/test.sq3');
            } catch (PDOException $e) {
                echo '数据库连接失败: ' . $e->getMessage();
            }
        }

        // 测试的时候将错误提示打开,否则出错了不清楚怎么回事
        $this->pdo->setAttribute($this->pdo::ATTR_ERRMODE, $this->pdo::ERRMODE_EXCEPTION);
    }

    /**
     * 查询所有数据
     */
    public function fetchAll($sql)
    {

        return $this->pdo->query($sql)->fetchAll();
    }

    /**
     * 插入一条数据
     */
    public function insertOneData($sql)
    {
        return $this->pdo->prepare($sql)->execute();

    }
}

/**
 * 定义一个打印结果的函数
 */
function p($val, $title = null)
{
    if($title) {
        echo "<br><br><h3>{$title}</h3><hr>";
    }
    echo '<pre>';
    var_dump($val);
    echo '</pre><hr>';
}

$sqlite = new Sqlite; //new 一个Sqlite对象

// 查询所有数据
$sqlQueryAll = "select * from persions";

$allData = $sqlite->fetchAll($sqlQueryAll);

p($allData,'所有查询结果');

// 插入一条数据
$sqlInsertOneData = sprintf("INSERT INTO persions(name,age,address) VALUES ('%s',%d,'%s');",'张三',25,'北京');

p($sqlite->insertOneData($sqlInsertOneData) ? '插入成功' : '插入失败','插入一条数据');
```