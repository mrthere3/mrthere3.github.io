---
title : rabbitmq学习笔记
categories : [python]
tags : [MQ]
cover: https://source.wjwsm.top/mulyadi-wYzjL5BwiEI-unsplash.jpg
date: 2022-10-23 10:00:00
---
#  python-RabbitMQ从入门到集群架构

## 1. python-RabbitMQ从入门到集群架构

~~~DO

# 安装好Docker，执行下面命令
docker pull rabbitmq:management
docker run -di --name Myrabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 rabbitmq:management
# 浏览器访问：
http://10.0.0.103:15672
# 输入用户名：guest 密码：guest ，进入到管理控制台
~~~



## 2.基本使用

### 2.1 基本使用

##### 生产者

~~~python

import pika
# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters('127.0.0.1'))

# 有密码
credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()
# 声明一个队列(创建一个队列)
channel.queue_declare(queue='lqz')

channel.basic_publish(exchange='',
                      routing_key='lqz', # 消息队列名称
                      body='hello world')
connection.close()
~~~

##### 消费者

~~~python

import pika

credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()

# 声明一个队列(创建一个队列)
channel.queue_declare(queue='lqz')

def callback(ch, method, properties, body):
    print("消费者接受到了任务: %r" % body)

channel.basic_consume(queue='lqz',on_message_callback=callback,auto_ack=True)

channel.start_consuming()
~~~

### 2.2消息安全之ack

##### 生产者

~~~python
import pika
# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters('127.0.0.1'))

# 有密码
credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()
# 声明一个队列(创建一个队列)
channel.queue_declare(queue='lqz')

channel.basic_publish(exchange='',
                      routing_key='lqz', # 消息队列名称
                      body='hello world')
connection.close()
~~~

##### 消费者

~~~python
import pika

credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()

# 声明一个队列(创建一个队列)
channel.queue_declare(queue='lqz')

def callback(ch, method, properties, body):
    print("消费者接受到了任务: %r" % body)
    # 通知服务端，消息取走了，如果auto_ack=False，不加下面，消息会一直存在
    # ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_consume(queue='lqz',on_message_callback=callback,auto_ack=False)

channel.start_consuming()
~~~

### 2.3  消息安全之durable持久化

##### 生产者

~~~python
import pika
# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters('127.0.0.1'))

# 有密码
credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()
# 声明一个队列(创建一个队列),durable=True支持持久化，队列必须是新的才可以
channel.queue_declare(queue='lqz1',durable=True)

channel.basic_publish(exchange='',
                      routing_key='lqz1', # 消息队列名称
                      body='111',
                      properties=pika.BasicProperties(
                          delivery_mode=2,  # make message persistent,消息也持久化
                      )
                      )
connection.close()
~~~

##### 消费者

~~~python
import pika

credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()

# 声明一个队列(创建一个队列)
channel.queue_declare(queue='lqz1')

def callback(ch, method, properties, body):
    print("消费者接受到了任务: %r" % body)
    # 通知服务端，消息取走了，如果auto_ack=False，不加下面，消息会一直存在
    # ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_consume(queue='lqz1',on_message_callback=callback,auto_ack=False)

channel.start_consuming()
~~~

### 2.4 闲置消费

##### 生产者

~~~python
import pika
# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters('127.0.0.1'))

# 有密码
credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()
# 声明一个队列(创建一个队列),durable=True支持持久化，队列必须是新的才可以
channel.queue_declare(queue='lqz123',durable=True)

channel.basic_publish(exchange='',
                      routing_key='lqz123', # 消息队列名称
                      body='111',
                      properties=pika.BasicProperties(
                          delivery_mode=2,  # make message persistent,消息也持久化
                      )
                      )
connection.close()
~~~

##### 消费者

~~~python
import pika
# 无密码
# connection = pika.BlockingConnection(pika.ConnectionParameters('127.0.0.1'))

# 有密码
credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()
# 声明一个队列(创建一个队列),durable=True支持持久化，队列必须是新的才可以
channel.queue_declare(queue='lqz123',durable=True)

channel.basic_publish(exchange='',
                      routing_key='lqz123', # 消息队列名称
                      body='111',
                      properties=pika.BasicProperties(
                          delivery_mode=2,  # make message persistent,消息也持久化
                      )
                      )
connection.close()
~~~

### 2.5 发布订阅

##### 生产者

~~~python
import pika
credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()

channel.exchange_declare(exchange='m1',exchange_type='fanout')

channel.basic_publish(exchange='m1',
                      routing_key='',
                      body='lqz nb')

connection.close()
~~~

##### 订阅者(启动几次订阅者会生成几个队列)

~~~python
import pika

credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()

# exchange='m1',exchange(秘书)的名称
# exchange_type='fanout' , 秘书工作方式将消息发送给所有的队列
channel.exchange_declare(exchange='m1',exchange_type='fanout')

# 随机生成一个队列
result = channel.queue_declare(queue='',exclusive=True)
queue_name = result.method.queue
print(queue_name)
# 让exchange和queque进行绑定.
channel.queue_bind(exchange='m1',queue=queue_name)


def callback(ch, method, properties, body):
    print("消费者接受到了任务: %r" % body)

channel.basic_consume(queue=queue_name,on_message_callback=callback,auto_ack=True)

channel.start_consuming()
~~~

###  2.6 发布订阅高级之Routing(按关键字匹配)

##### 发布者

~~~python
import pika
credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()

channel.exchange_declare(exchange='m2',exchange_type='direct')

channel.basic_publish(exchange='m2',
                      routing_key='bnb', # 多个关键字，指定routing_key
                      body='lqz nb')

connection.close()
~~~

##### 订阅者1

~~~python
import pika

credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()

# exchange='m1',exchange(秘书)的名称
# exchange_type='direct' , 秘书工作方式将消息发送给不同的关键字
channel.exchange_declare(exchange='m2',exchange_type='direct')

# 随机生成一个队列
result = channel.queue_declare(queue='',exclusive=True)
queue_name = result.method.queue
print(queue_name)
# 让exchange和queque进行绑定.
channel.queue_bind(exchange='m2',queue=queue_name,routing_key='nb')
channel.queue_bind(exchange='m2',queue=queue_name,routing_key='bnb')


def callback(ch, method, properties, body):
    print("消费者接受到了任务: %r" % body)

channel.basic_consume(queue=queue_name,on_message_callback=callback,auto_ack=True)

channel.start_consuming()
~~~

##### 订阅者2

~~~python
import pika

credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()

# exchange='m1',exchange(秘书)的名称
# exchange_type='direct' , 秘书工作方式将消息发送给不同的关键字
channel.exchange_declare(exchange='m2',exchange_type='direct')

# 随机生成一个队列
result = channel.queue_declare(queue='',exclusive=True)
queue_name = result.method.queue
print(queue_name)
# 让exchange和queque进行绑定.
channel.queue_bind(exchange='m2',queue=queue_name,routing_key='nb')
channel.queue_bind(exchange='m2',queue=queue_name,routing_key='bnb')


def callback(ch, method, properties, body):
    print("消费者接受到了任务: %r" % body)

channel.basic_consume(queue=queue_name,on_message_callback=callback,auto_ack=True)

channel.start_consuming()
~~~

### 2.7 发布订阅高级之Topic(按关键字模糊匹配)

#### 发布者

~~~python
import pika
credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()

channel.exchange_declare(exchange='m3',exchange_type='topic')

channel.basic_publish(exchange='m3',
                      # routing_key='lqz.handsome', #都能收到
                      routing_key='lqz.handsome.xx', #只有lqz.#能收到
                      body='lqz nb')

connection.close()
~~~

#### 订阅者1 只能加一个单词

~~~python
import pika

credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()

# exchange='m1',exchange(秘书)的名称
# exchange_type='direct' , 秘书工作方式将消息发送给不同的关键字
channel.exchange_declare(exchange='m3',exchange_type='topic')

# 随机生成一个队列
result = channel.queue_declare(queue='',exclusive=True)
queue_name = result.method.queue
print(queue_name)
# 让exchange和queque进行绑定.
channel.queue_bind(exchange='m3',queue=queue_name,routing_key='lqz.#')



def callback(ch, method, properties, body):
    print("消费者接受到了任务: %r" % body)

channel.basic_consume(queue=queue_name,on_message_callback=callback,auto_ack=True)

channel.start_consuming()
~~~

####  订阅者2

~~~python
import pika

credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()

# exchange='m1',exchange(秘书)的名称
# exchange_type='topic' , 模糊匹配
channel.exchange_declare(exchange='m3',exchange_type='topic')

# 随机生成一个队列
result = channel.queue_declare(queue='',exclusive=True)
queue_name = result.method.queue
print(queue_name)
# 让exchange和queque进行绑定.
channel.queue_bind(exchange='m3',queue=queue_name,routing_key='lqz.*')


def callback(ch, method, properties, body):
    queue_name = result.method.queue # 发送的routing_key是什么
    print("消费者接受到了任务: %r" % body)

channel.basic_consume(queue=queue_name,on_message_callback=callback,auto_ack=True)

channel.start_consuming()
~~~

### 2.8 基于rabbitmq实现rpc

#### 服务端

~~~python
import pika
credentials = pika.PlainCredentials("admin","admin")
connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166',credentials=credentials))
channel = connection.channel()

# 起翰监听任务队列
channel.queue_declare(queue='rpc_queue')

def on_request(ch, method, props, body):
    n = int(body)
    response = n + 100
    # props.reply_to  要放结果的队列.
    # props.correlation_id  任务
    ch.basic_publish(exchange='',
                     routing_key=props.reply_to,
                     properties=pika.BasicProperties(correlation_id= props.correlation_id),
                     body=str(response))
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_qos(prefetch_count=1)
channel.basic_consume( queue='rpc_queue',on_message_callback=on_request,)
channel.start_consuming()
~~~

#### 客户端

~~~python
import pika
import uuid

class FibonacciRpcClient(object):
    def __init__(self):
        credentials = pika.PlainCredentials("admin", "admin")
        self.connection = pika.BlockingConnection(pika.ConnectionParameters('101.133.225.166', credentials=credentials))
        self.channel = self.connection.channel()

        # 随机生成一个消息队列(用于接收结果)
        result = self.channel.queue_declare(queue='',exclusive=True)
        self.callback_queue = result.method.queue

        # 监听消息队列中是否有值返回,如果有值则执行 on_response 函数(一旦有结果,则执行on_response)
        self.channel.basic_consume(queue=self.callback_queue,on_message_callback=self.on_response, auto_ack=True)

    def on_response(self, ch, method, props, body):
        if self.corr_id == props.correlation_id:
            self.response = body

    def call(self, n):
        self.response = None
        self.corr_id = str(uuid.uuid4())

        # 客户端 给 服务端 发送一个任务:  任务id = corr_id / 任务内容 = '30' / 用于接收结果的队列名称
        self.channel.basic_publish(exchange='',
                                   routing_key='rpc_queue', # 服务端接收任务的队列名称
                                   properties=pika.BasicProperties(
                                         reply_to = self.callback_queue, # 用于接收结果的队列
                                         correlation_id = self.corr_id, # 任务ID
                                         ),
                                   body=str(n))
        while self.response is None:
            self.connection.process_data_events()

        return self.response

fibonacci_rpc = FibonacciRpcClient()

response = fibonacci_rpc.call(50)
print('返回结果:',response)
~~~

## 3 集群搭建

### 3.1 普通集群(副本集群)

默认情况下:RabbitMQ代理操作所需的所有数据/状态都将跨所有节点复制。这方面的一个例外是消息队列，默认情况下，消息队列位于一个节点上，尽管它们可以从所有节点看到和访问
普通集群主节点必须可用，可以同步交换机和队列，但是队列中的消息只能看到，不能同步

~~~she
# 1 集群规划
mq1: 10.0.0.100  mq1  master 主节点
mq2: 10.0.0.101  mq2  repl1  副本节点
mq3: 10.0.0.103  mq3  repl2  副本节点

# 2 克隆三台机器主机名和ip映射（建立集群时仅支持按主机名创建，所以我们加好映射）
vim /etc/hosts
# 分别在三台机器上加入（直接通过主机名就可以ping通）
10.0.0.100 mq1
10.0.0.101 mq2
10.0.0.103 mq3

# 3 修改3个节点的主机名（分别在三台机器上）
hostname mq1
hostname mq2
hostname mq3
# 4 分别在三台机器上安装相同版本的rabbitmq-server，并启动（启动就会生成cookie）
启动查看管理web页面能看到，但是不是集群状态
查看集群状态也是孤立的
rabbitmqctl cluster_status
现在我们全停掉（3台机器都停掉）


# 5 三个机器安装rabbitmq,并同步cookie文件,在mq1上执行:(启动一遍，cookie才会有)
cat /var/lib/rabbitmq/.erlang.cookie
scp /var/lib/rabbitmq/.erlang.cookie root@mq2:/var/lib/rabbitmq/
scp /var/lib/rabbitmq/.erlang.cookie root@mq3:/var/lib/rabbitmq/

# 6 查看cookie是否一致:
mq1: cat /var/lib/rabbitmq/.erlang.cookie 
mq2: cat /var/lib/rabbitmq/.erlang.cookie 
mq3: cat /var/lib/rabbitmq/.erlang.cookie 

# 7 后台启动rabbitmq所有节点执行如下命令,启动成功访问管理界面（三台机器都启动，老版本web页面看不到，新版本能看到）:
rabbitmq-server -detached 

# 8 在mq2和mq3上分别执行加入集群命令:
1.关闭       rabbitmqctl stop_app
2.加入集群    rabbitmqctl join_cluster rabbit@mq1  # 仅支持写主机名
3.启动服务    rabbitmqctl start_app

# 9 查看集群状态,任意节点执行:
rabbitmqctl cluster_status

# 10 如果出现如下显示,集群搭建成功:
Cluster status of node rabbit@mq1 ...
Basics
Cluster name: rabbit@mq1
Disk Nodes
rabbit@mq1
rabbit@mq2
rabbit@mq3
Running Nodes
rabbit@mq1
rabbit@mq2
rabbit@mq3
# 11 登录管理界面,展示如下状态:


注意：如果启动不了，删除/var/lib/rabbitmq/mnesia下的所有问题重新操作
~~~

测试命令

~~~shell
# 1 现在我们挂掉从节点是可以的
# 在mq2上执行
rabbitmqctl stop_app
# 再启动（集群不受影响），再发布消息也是可以的，只要主节点活着就可以
rabbitmqctl start_app
# 5 挂掉主节点，整个服务就不能用了
# 在mq1上执行
rabbitmqctl stop_app
# 集群不能用了（两个从节点不能对外提供服务），如果消息是不持久化的消息，主节点挂了，消息就没了
# 发布消息可以发布到主，从节点；消费消息也可以从主从节点消费
rabbitmqctl start_app
~~~

### 3.2 镜像集群

~~~shell
# 0.策略说明
rabbitmqctl set_policy [-p <vhost>] [--priority <priority>] [--apply-to <apply-to>] <name> <pattern>  <definition>
-p Vhost： 可选参数，针对指定vhost下的queue进行设置
Name:     policy的名称
Pattern: queue的匹配模式(正则表达式)
Definition：镜像定义，包括三个部分ha-mode, ha-params, ha-sync-mode
                ha-mode:指明镜像队列的模式，有效值为 all/exactly/nodes
                      all：表示在集群中所有的节点上进行镜像
                      exactly：表示在指定个数的节点上进行镜像，节点的个数由ha-params指定
                      nodes：表示在指定的节点上进行镜像，节点名称通过ha-params指定
             ha-params：ha-mode模式需要用到的参数
             ha-sync-mode：进行队列中消息的同步方式，有效值为automatic(自动)和manual（手动）
             priority：可选参数，policy的优先级


# 1.查看当前策略（可以在任意节点上执行）
rabbitmqctl list_policies

# 2.添加策略
rabbitmqctl set_policy ha-all '^lqz' '{"ha-mode":"all","ha-sync-mode":"automatic"}' 
# 说明:策略正则表达式为 “^” 表示所有匹配所有队列名称  ^lqz:匹配lqz开头队列
rabbitmqctl set_policy ha-all '^' '{"ha-mode":"all","ha-sync-mode":"automatic"}' 
# 对所有队列都做同步

# 3.删除策略
rabbitmqctl clear_policy ha-all

# 4.测试集群
停掉主节点，其他节点依然可以提供服务
~~~
