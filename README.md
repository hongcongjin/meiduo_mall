# meiduo_mall

##### 1,使用自己定义的配置文件

​	在项目目录下新建settings包，新建dev.py(开发环境),prod.py(生产环境)	
​	在manage.py中指定自己新建的配置文件为项目配置文件

###### 2,使用Jinja2模板引擎

​	pip install Jinja2
​	需要在dev中修改配置文件
​	Jinja2比DjangoTemplates快大概10倍多
​	在utils中补充Jinja2模板引擎(确保可以使用模板引擎中的{{ url('') }} {{ static('') }}这类语句 )

###### 3,使用MySql数据库

​	pip install mysqlclient
​	在dev中配置MySql数据库

###### 4,使用redis数据库做为部分缓存数据

​	pip install django-redis

###### 5,配置工程日志

​	在dev.py中进行工程日志的相关配置
​	准备日志文件目录在与项目同级的目录下新建logs文件夹
​	(import logging

	###### 创建日志记录器
​	logger = logging.getLogger('django'))

###### 6,配置前端静态文件

​	在dev.py文件中配置两个属性即可

###### 7,查看追加项目的导包路径

​	查看导包路径:print(sys.path)
​	追加导包路径:sys.path.insert(0, os.path.join(BASE_DIR, 'apps'))
​		可以直接使用app名字实现注册

###### 8,自定义用户模型类

​	在users应用中自定义User模型类
​	并且在dev.py中配置要迁移的文件是我们自己配置的不是系统自带的auth认证模型
​	在dev.py中指定自定义用户后端认证

###### 9,模型迁移

​	python manage.py makemigrations
​	python manage.py migrate

###### 10,pipeline操作数据

​	Redis的 C - S 架构：
​		基于客户端-服务端模型以及请求/响应协议的TCP服务。
​		客户端向服务端发送一个查询请求，并监听Socket返回。
​		通常是以阻塞模式，等待服务端响应。
​		服务端处理命令，并将结果返回给客户端。
​	如果Redis服务端需要同时处理多个请求，加上网络延迟，那么服务端利用率不高，效率降低。
​	管道pipeline
​		可以一次性发送多条命令并在执行完后一次性将结果返回。
​		pipeline通过减少客户端与Redis的通信次数来实现降低往返延时时间。
​	实现的原理
​		实现的原理是队列。
​		Client可以将三个命令放到一个tcp报文一起发送。
​		Server则可以将三条命令的处理结果放到一个tcp报文返回。
​		队列是先进先出，这样就保证数据的顺序性。
​	实现代码直接在views.py文件中实现



###### 11,生产者消费者设计模式

​	我们的代码是自上而下同步执行的。发送短信是耗时的操作。如果短信被阻塞住，用户响应将会延迟。响应延迟会造成用户界面的倒计时延迟。
​	异步发送短信，发送短信和响应分开执行，将发送短信从主业务中解耦出来。
​	生产者生成消息，缓存到消息队列中，消费者读取消息队列中的消息并执行。

###### 12,RabbitMQ介绍和使用

​	消息队列是消息在传输的过程中保存消息的容器。
​	现在主流消息队列有：RabbitMQ、ActiveMQ、Kafka等等。
​	项目中使用的是redis数据库作为消息队列，在Celery_task.config.py中配置

###### 13,Celery使用

​	思考:
​		消费者取到消息之后，要消费掉（执行任务），需要我们去实现。
​		任务可能出现高并发的情况，需要补充多任务的方式执行。
​		耗时任务很多种，每种耗时任务编写的生产者和消费者代码有重复。
​		取到的消息什么时候执行，以什么样的方式执行。

	实际开发中，我们可以借助成熟的工具Celery来完成。
	有了Celery，我们在使用生产者消费者模式时，只需要关注任务本身，极大的简化了程序员的开发流程。
###### 14,Celery介绍：

​	一个简单、灵活且可靠、处理大量消息的分布式系统，可以在一台或者多台机器上运行。
​	单个 Celery 进程每分钟可处理数以百万计的任务。
​	通过消息进行通信，使用消息队列（broker）在客户端和消费者之间进行协调。

	celery+django
	在windows环境利用celery实现简单的任务队列
	测试使用环境：
	　　1、Python==3.6.1
	　　2、redis==3.0.501
	　　3、celery==4.1.1
	　　4、eventlet==0.23.0
	#启动celery异步命令
	#celery -A celery_tasks.main worker -l info -P eventlet
	#celery -A celery_tasks.main worker -l info -P eventlet  -c 1000    同时开启1000个协程，提高并发量
###### 15,qq登录开发

​	申请成为qq互联应用开发者
​	定义qq登录模型类(并迁移)
​	安装qq登录的工具QQLoginTool(python第三方库)
​	OAuth2.0认证获取openid(修改本机的host)
​	判断openid是否绑定了用户
​		为了能够在后续的绑定用户操作中前端可以使用openid，在这里将openid签名后响应给前端。
​		openid属于用户的隐私信息，所以需要将openid签名处理，避免暴露。
​		补充itsdangerous的使用
​			安装：pip install itsdangerous
​			TimedJSONWebSignatureSerializer的使用
​			使用TimedJSONWebSignatureSerializer可以生成带有有效期的token
​	openid绑定用户的实现

###### 16,django发送邮件的配置(使用163邮箱)

​	在dev.py中进行配置
​		EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend' # 指定邮件后端
​		EMAIL_HOST = 'smtp.163.com' # 发邮件主机
​		EMAIL_PORT = 25 # 发邮件端口
​		EMAIL_HOST_USER = 'hmmeiduo@163.com' # 授权的邮箱
​		EMAIL_HOST_PASSWORD = 'hmmeiduo123' # 邮箱授权时获得的密码，非注册登录密码
​		EMAIL_FROM = '美多商城<hmmeiduo@163.com>' # 发件人抬头	
​	发送邮件是一个耗时操作,但是不能阻塞商城响应，所以需要异步响应，我们继续使用Celery实现异步任务。(在email中写逻辑代码，在celery_tasks.main.py中进行配置)

###### 17,文件存储方案FastDFS

​	用c语言编写的一款开源的轻量级分布式文件系统。
​	功能包括：文件存储、文件访问（文件上传、文件下载）、文件同步等，解决了大容量存储和负载均衡的问题。特别适合以文件为载体的在线服务，如相册网站、视频网站等等。
​	为互联网量身定制，充分考虑了冗余备份、负载均衡、线性扩容等机制，并注重高可用、高性能等指标。
​	可以帮助我们搭建一套高性能的文件服务器集群，并提供文件上传、下载等服务。
​	FastDFS架构 包括Client、Tracker server和Storage server。
​		Client请求Tracker进行文件上传、下载，Tracker再调度Storage完成文件上传和下载。
​	Client： 客户端，业务请求的发起方，通过专有接口，使用TCP/IP协议与Tracker或Storage进行数据交互。FastDFS提供了upload、download、delete等接口供客户端使用。
​	Tracker server：跟踪服务器，主要做调度工作，起负载均衡的作用。在内存中记录集群中所有存储组和存储服务器的状态信息，是客户端和数据服务器交互的枢纽。
​	Storage server：存储服务器（存储节点或数据服务器），文件和文件属性都保存到存储服务器上。Storage server直接利用OS的文件系统调用管理文件。
​	Storage群中的横向可以扩容，纵向可以备份。

###### 18,FastDFS文件索引

​	FastDFS上传和下载流程 可以看出都涉及到一个数据叫文件索引（file_id）。
​		文件索引（file_id）是客户端上传文件后Storage返回给客户端的一个字符串，是以后访问该文件的索引信息。
​	文件索引（file_id）信息包括：组名、虚拟磁盘路径、数据两级目录、文件名等信息。
​		组名：文件上传后所在的 Storage 组名称。
​		虚拟磁盘路径：Storage 配置的虚拟路径，与磁盘选项store_path*对应。如果配置了store_path0则是M00，如果配置了store_path1则是M01，以此类推。
​		数据两级目录：Storage 服务器在每个虚拟磁盘路径下创建的两级目录，用于存储数据文件。
​		文件名：由存储服务器根据特定信息生成，文件名包含:源存储服务器IP地址、文件创建时间戳、文件大小、随机数和文件拓展名等信息。
​	