# docker相关

```
//安装docker
yum install docker
//启动docker
systemctl start docker
//设置docker开机启动
systgemctl enable docker
```
国内较快的镜像原地址

```
#Docker 官方中国区
https://registry.docker-cn.com
#网易
http://hub-mirror.c.163.com
#ustc
https://docker.mirrors.ustc.edu.cn
```
## 方法一
在拉取镜像时候指定镜像源地址

您可以使用以下命令直接从该镜像加速地址进行拉取
```
$ docker pull registry.docker-cn.com/myname/myrepo:mytag
例如:
$ docker pull registry.docker-cn.com/library/ubuntu:16.04
```
有效时间为当前命令

## 方法二
使用 –registry-mirror 配置 Docker 守护进程 
您可以配置 Docker 守护进程默认使用 Docker 官方镜像加速。这样您可以默认通过官方镜像加速拉取镜像，而无需在每次拉取时指定 registry.docker-cn.com。

您可以在 Docker 守护进程启动时传入 --registry-mirror 参数：
```
$ docker --registry-mirror=https://registry.docker-cn.com daemon
```
有效时间为当前的docker进程，重启docker服务后需要重新设置

## 方法三
最简单粗暴

为了永久性保留更改，您可以修改 /etc/docker/daemon.json 文件并添加上 registry-mirrors 键值。
```
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```
然后重新启动下docker服务

```
systemctl restart docker
```
# redis
```
//启动redis
docker run -d -p 6379:6379 --name myredis redis
docker start b70b26a71166
```

# 消息
```linux
//安装rabbitmq
docker pull registry.docker-cn.com/library/rabbitmq:3-management
//启动rabbitmq
docker run -d -p 5672:5672 -p 15672:15672 --name myrabbitmq 24cb552c7c00
```
网页输入http://192.168.1.109:15672/登录rabbitmq管理界面，账号密码为guest



```
//使用activemq
安装说明
这里使用Docker安装，查询Docker镜像：

docker search activemq
下载Docker镜像：

docker pull webcenter/activemq
创建&运行ActiveMQ容器：

docker run -d --name myactivemq -p 61617:61616 -p 8162:8161 webcenter/activemq
61616是 activemq 的容器使用端口（映射为61617），8161是 web 页面管理端口（对外映射为8162）

查看创建的容器，如果存在说明安装成功：

docker ps
查看WEB管理页面：

浏览器输入http://ip:8162 点击Manage ActiveMQ broker使用默认账号/密码：admin/admin进入查看。



配置访问密码
进入Docker容器：

docker exec -it myactivemq /bin/bash
控制台界面设置用户名和密码：

# 位于根目录 conf 目录下
vi jetty-realm.properties

# 修改密码
# username: password [,rolename ...]
admin: admin, admin
配置连接密码
编辑activemq.xml文件，放置到 shutdownHooks 下方即可。

<!-- 添加访问ActiveMQ的账号密码 -->
<plugins>
    <simpleAuthenticationPlugin>
        <users>
            <authenticationUser username="${activemq.username}" password="${activemq.password}" groups="users,admins"/>
        </users>
    </simpleAuthenticationPlugin>
</plugins>
修改conf中credentials.properties文件进行密码设置：

activemq.username=admin
activemq.password=123456
guest.password=123456
```
# 检索
```
//solr

docker安装solr
docker pull solr

启动solr镜像
docker run -d -p 8983:8983 --name mysolr solr

新建core
docker exec -it --user=solr mysolr bin/solr create_core -c ik_core

进入solr容器
docker exec -it -u root mysolr /bin/bash

安装vim(编辑容器里的文件)
apt-get update
apt-get install vim

安装rzsz(上传下载容器里的文件)
apt-get install lrzsz

进入/opt/solr/server/solr-webapp/webapp/WEB-INF/lib添加jar包
使用rz上传

进入/opt/solr/server/solr/ik_core/conf
配置managed-schema，加入IK分词
<fieldType name="text_ik" class="solr.TextField">
  <analyzer class="org.wltea.analyzer.lucene.IKAnalyzer"/>
</fieldType>

<field name="item_title" type="text_ik" indexed="true" stored="true"/>
<field name="item_sell_point" type="text_ik" indexed="true" stored="true"/>
<field name="item_price"  type="long" indexed="true" stored="true"/>
<field name="item_image" type="string" indexed="false" stored="true" />
<field name="item_category_name" type="string" indexed="true" stored="true" />

<field name="item_keywords" type="text_ik" indexed="true" stored="false" multiValued="true"/>
<copyField source="item_title" dest="item_keywords"/>
<copyField source="item_sell_point" dest="item_keywords"/>
<copyField source="item_category_name" dest="item_keywords"/>

//修改完成后exit,然后docker restart 当前镜像

// solr7建立中文分析器时遇到问题。
Unknown fieldType 'long' specified on field item_price

managed-schema配置文件中long 改为plong就可以了

IK Analyzer 和solr 7 似乎不兼容
这里选用solr自带的smartcn
配置managed-schema
 <schema>
    <!-- 配置中文分词器 -->
    <fieldType name="text_smartcn" class="solr.TextField" positionIncrementGap="100">
        <analyzer type="index">
            <tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory"/>
        </analyzer>
        <analyzer type="query">
            <tokenizer class="org.apache.lucene.analysis.cn.smart.HMMChineseTokenizerFactory"/>
        </analyzer>
    </fieldType>
</schema>



//solr开机不会自启动
运行docker restart 9daedf7ef246
```
# 分布式
启动zookeeper
```
docker run --name zk01 -p 2181:2181 --restart always -d 4ebfb9474e72
```