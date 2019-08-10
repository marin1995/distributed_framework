# 1.distributed_framework
```
python分布式函数调度框架。适用场景范围超级广泛。

支持python内置Queue对象作为当前解释器下的消息队列。
支持sqlite3作为本机持久化消息队列。
支持pika包实现的使用rabbitmq作为分布式消息队列。
支持rabbitpy包实现的使用rabbitmq作为分布式消息队列。
支持amqpstorm包实现的使用rabbitmq作为分布式消息队列。
支持redis中间件作为分布式消息队列。（不支持消费确认，例如把消息取出来了，但函数还在运行中没有运行完成，
突然关闭程序或断网断电，会造成部分任务丢失【设置的并发数量越大，丢失数量越惨】，所以推荐mq）
支持mongodb中间件作为分布式消息队列。


源码实现思路100%遵守了oop的6个设计原则，很容易扩展中间件。
1、单一职责原则——SRP
2、开闭原则——OCP
3、里式替换原则——LSP
4、依赖倒置原则——DIP
5、接口隔离原则——ISP
6、迪米特原则——LOD
可以仿照源码中实现中间件的例子，只需要继承发布者、消费者基类后实现几个抽象方法即可添加新的中间件。



将函数名和队列名绑定，即可开启自动消费。

只需要一行代码就 将任何函数实现 分布式 、并发、 控频、断点接续运行、定时、指定时间不运行、
消费确认、重试指定次数、重新入队、超时杀死、计算消费次数速度、预估消费时间、
函数运行日志记录、任务过滤、任务过期丢弃等数十种功能。

和celery一样支持线程、gevent、eventlet 并发运行模式，大大简化比使用celery，很强大简单，已在多个生产项目和模块验证。
确保任何并发模式在linux和windows，一次编写处处运行。不会像celery在windwos上某些功能失效。

没有严格的目录结构，代码可以在各个文件夹层级到处移动，脚本名字可以随便改。不使用令人讨厌的cmd命令行启动。
```

# 1.1  pip安装方式
pip install function_scheduling_distributed_framework --upgrade -i https://pypi.org/simple 

# 2.具体更详细的用法可以看test_frame文件夹里面的几个示例。
 ```python
import time

from function_scheduling_distributed_framework import patch_frame_config, show_frame_config,get_consumer


# 初次接触使用，可以不安装任何中间件，使用本地持久化队列。正式墙裂推荐安装rabbitmq。
patch_frame_config(MONGO_CONNECT_URL='mongodb://myUserAdminxx:xxxx@xx.90.89.xx:27016/admin',

                   RABBITMQ_USER='silxxxx',
                   RABBITMQ_PASS='Fr3Mxxxxx',
                   RABBITMQ_HOST='1xx.90.89.xx',
                   RABBITMQ_PORT=5672,
                   RABBITMQ_VIRTUAL_HOST='test_host',

                   REDIS_HOST='1xx.90.89.xx',
                   REDIS_PASSWORD='yxxxxxxR',
                   REDIS_PORT=6543,
                   REDIS_DB=7, )

show_frame_config()

# 主要的消费函数，演示做加法，假设需要花10秒钟。
def f2(a, b):
    print(f'消费此消息 {a} + {b} ,结果是  {a + b}')
    time.sleep(10)  # 模拟做某事需要阻塞10秒种，必须用并发绕过此阻塞。


# 把消费的函数名传给consuming_function，就这么简单。
# 通过设置broker_kind，一键切换中间件为mq或redis等7种中间件或包。
# 额外参数支持超过10种控制功能，celery支持的控制方式，都全部支持。
# 这里演示使用本地持久化队列，本机多个脚本之间可以共享任务，无需安装任何中间件，降低初次使用门槛。
consumer = get_consumer('queue_test2', consuming_function=f2, broker_kind=6)  



# 推送需要消费的任务，可以变消费边推送。发布的内容字典需要和函数所能接收的参数一一对应，
# 并且函数参数需要能被json序列化，不要把自定义的类型作为消费函数的参数。
consumer.publisher_of_same_queue.clear()
[consumer.publisher_of_same_queue.publish({'a': i, 'b': 2 * i}) for i in range(100)]


# 开始从中间件循环取出任务，使用指定的函数消费中间件里面的消息。
consumer.start_consuming_message()

 ```
 
### 3.1运行中截图
![Image text](https://i.niupic.com/images/2019/08/09/_477.png)

### 3.2控频功能证明，由于截图是外网调度rabbitmq的消息有延迟，没有精确到函数每秒运行10次。

![Image text](https://i.niupic.com/images/2019/08/09/_462.png)


## 4.celery和这个框架比，存储的内容差异
### 4.1celery的
 ```
 {
  "body": "W1szLCA2XSwge30sIHsiY2FsbGJhY2tzIjogbnVsbCwgImVycmJhY2tzIjogbnVsbCwgImNoYWluIjogbnVsbCwgImNob3JkIjogbnVsbH1d",
   "content-encoding":  "utf-8",
   "content-type":  "application/json",
   "headers":  {
    "lang":  "py",
     "task":  "test_task\u554a",
     "id":  "39198371-8e6a-4994-9f6b-0335fe2e9b92",
     "shadow":  null,
     "eta":  null,
     "expires":  null,
     "group":  null,
     "retries":  0,
     "timelimit":  [
      null,
       null
    ],
     "root_id":  "39198371-8e6a-4994-9f6b-0335fe2e9b92",
     "parent_id":  null,
     "argsrepr":  "(3, 6)",
     "kwargsrepr":  "{}",
     "origin":  "gen22848@FQ9H7TVDZLJ4RBT"
  },
   "properties":  {
    "correlation_id":  "39198371-8e6a-4994-9f6b-0335fe2e9b92",
     "reply_to":  "3ef38b98-1417-3f3d-995b-89e8e15849fa",
     "delivery_mode":  2,
     "delivery_info":  {
      "exchange":  "",
       "routing_key":  "test_a"
    },
     "priority":  0,
     "body_encoding":  "base64",
     "delivery_tag":  "59e39055-2086-4be8-a801-993061fee443"
  }
}
  ```

### 4.2 此框架的消息很短，就是一个字典，内容的键值对和函数入参一一对应。
额外控制参数如重试、超时kill，由代码决定，
不需要存到中间件里面去。例如函数运行超时大小在本地代码修改后，立即生效。

不由中间件里面的配置来决定。
 ```
{"a":3,"b":6}
  ```
  

