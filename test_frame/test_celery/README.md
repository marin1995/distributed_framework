celery测试。
主要是和此框架调度同一个函数。使用同一种redis内网中间件、运行在同一个机器、都使用gevent模式运行、都不要消费结果回调、相同的并发数量所有硬件和软件环境保持一致的情况下，测试两个框架的推动和消费性能的区别。