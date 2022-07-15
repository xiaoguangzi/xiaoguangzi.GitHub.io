**如果你懂Linux的话最好用Linux吧，我是被逼无奈的好吧。**

## 下载confluence

<https://www.atlassian.com/software/confluence/download-archives>

上面有长期支持版本，最好选择长期支持版本，方便后续升级。也可以选择之前的版本，但是现在前面的版本都有问题，本次安装主要是为了升级，所以被迫选择了windows作为服务器。

## 下载破解工具

<https://down.whsir.com/downloads/confluence7.4pojie.zip>

因为使用markdown的问题，这里不展示里面有什么东西了，一会破解的时候需要用到其中的一个jar包，windows服务器上需要安装jdk，实际上就是双击就行了。

## 安装jdk8

没什么难度，下载以后一直回车就行了，如果有其他要求可以自定义。就是从这里开始，我开始想吐，windows运维你们还好吗？

<https://www.oracle.com/java/technologies/downloads/#java8-windows>

## 安装MySQL

我这里选择是通过外置数据库，也可以不选择这步，更加简单一些。Windows上MySQL数据库还需要装一个什么visual c++，淦，Windows玩的就是花。这里的步骤可能多谢，因为是纯文本方式，所以我尽量把动作描述的清晰些。

<https://www.microsoft.com/zh-CN/download/details.aspx?id=40784>

我选择的是MySQL 5.7.35版本，这个也可以看你们个人的需要吧。<https://downloads.mysql.com/archives/community/>

下载完成后解压到一个目录，然后打开系统设置，搜索"环境变量"，在系统变量-Path中新建，粘贴MySQL二进制文件所在路径，即为MySQL所在文件夹下的bin目录(路径一定要加上bin，不然的话会解析不到，Java配置环境也是这样，跟unix系统不同)。

Windows 安装的MySQL默认是没有配置文件的，需要自己添加，你妹……安装目录下新建my.ini文件，写入以下配置:

```shell
[mysqld]
character-set-server=utf8
collation-server=utf8_bin
default-storage-engine=INNODB
max_allowed_packet=512M
innodb_log_file_size=1024M
transaction-isolation=READ-COMMITTED
# 最后一个参数一定要加，confluence连接会报错，MySQL隔离机制必须设置成read-committed
```

在MySQL的bin目录下打开cmd，然后执行以下命令，主要是进行初始化以及安装启动:

```shell
mysqld --initialize-insecure
mysqld -install
net start mysql
# 如果要重启的话，只能使用*net stop mysql* *net start mysql*
```

接下来是登录，我们上面初始化的时候并没有生成密码，所以在登陆的时候增加 --skip-password 参数。

```shell
mysql -uroot -p --skip-password
```

```sql
alter user root@'localhost' identified by "123456";
create user root@'%' identified by "123456";
grant all on *.* to root@'%' with grant option;
# 给root用户授权权限。
create user confluence@'%' identified by "123456";
create database confluence character set utf8 collate utf8_bin;
grant all on confluence.* to confluence@'%';
# 一般不要使用root用户权限
```

## 终于该安装confluence了

解压然后开始安装，选择*custom install*，手动安装可以选择很多东西，如果选择第一个的话好像更方便些，这个我没试过。剩下的都是自定义的东西了，这个可以自己搞。访问*http://localhost:8090*，点击第二个产品安装，两个插件都要选择上。

搞完了这些以后我们现在需要进行破解软件了。C:\Program Files\Atlassian\Confluence\confluence\WEB-INF\lib，我的安装目录是这个，该目录中存在一个atlassian-extras-decoder-v2-3.4.1.jar，移动到破解工具目录并且重命名为atlassian-extras-2.4.jar。

运行confluence_keygen.jar，name可以随便写，server id 就在你的confluence的页面上，记得加上-，没有会识别不到。然后选择.path，找到刚才的atlassian-extras-decoder-2.4.jar包，点击.gen!生成密钥。这个时候不要着急关闭破解工具，现在需要将atlassian-extras-2.4.jar改回之前的名字(atlassian-extras-decoder-v2-3.4.1.jar)后放到C:\Program Files\Atlassian\Confluence\confluence\WEB-INF\lib目录下。

因为我们选择的是通过MySQL作为外置数据库，所以接一下需要下载一个MySQL的驱动，也需要放在(C:\Program Files\Atlassian\Confluence\confluence\WEB-INF\lib)目录下。

<https://cdn.mysql.com/archives/mysql-connector-java-5.1/mysql-connector-java-5.1.48.zip>

这两个文件放进去以后就可以重启confluence了，执行下面的命令(有一定时候不怎么好用，也不知道是我配置问题还是怎么回事)。

```shell
stop-service.bat
start-service.bat
```

接下来是把刚才破解的值输入就行了，基本上到这里已经结束了，如果还有其他问题的话可以在网上搜一搜，基本上都有解决方法。