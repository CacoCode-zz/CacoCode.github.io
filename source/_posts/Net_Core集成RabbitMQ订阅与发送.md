---
title: .Net Core 集成 RabbitMQ 订阅与发送
date: 2020-04-10 09:52:19
tags:
  - .NET Core
  - RabbitMQ
categories:
  - .NET
---

# 什么是RabbitMQ？
- **专业理解**：
MQ全称为Message Queue，即消息队列， RabbitMQ是由erlang语言开发，基于AMQP（Advanced MessageQueue 高级消息队列协议）协议实现的消息队列，它是一种应用程序之间的通信方法，消息队列在分布式系统开发中应用非常广泛。
- **通俗理解**：
你可以把它想象成一个邮局：当你把你想要发布的邮件放在邮箱中时，你可以确定邮差先生最终将邮件发送给你的收件人。在这个比喻中，RabbitMQ是邮政信箱，邮局和邮递员。RabbitMQ和邮局的主要区别在于它不处理纸张，而是接受，存储和转发二进制数据块。

# 使用场景
- **任务异步处理**
将不需要同步处理的并且耗时长的操作由消息队列通知消息接收方进行异步处理。提高了应用程序的响应时间。

- **应用程序解耦合**
MQ相当于一个中介，生产方通过MQ与消费方交互，它将应用程序进行解耦合。

# 组成原理
<IMG src="https://timgsa.baidu.com/timg?image&amp;quality=80&amp;size=b9999_10000&amp;sec=1586436478385&amp;di=dd57da48bf811a55c1e5fe17849846ba&amp;imgtype=0&amp;src=http%3A%2F%2Fimages3.10qianwan.com%2F10qianwan%2F20190430%2Fb_0_201904300718245296.jpg"/>

Broker：消息队列服务进程，此进程包括两个部分：Exchange和Queue。

Exchange：消息队列交换机，按一定的规则将消息路由转发到某个队列，对消息进行过虑。

Queue：消息队列，存储消息的队列，消息到达队列并转发给指定的消费方。

Producer：消息生产者，即生产方客户端，生产方客户端将消息发送到MQ。

Consumer：消息消费者，即消费方客户端，接收MQ转发的消息。

# 配置安装

RabbitMQ下载地址：[https://www.rabbitmq.com/download.html](https://www.rabbitmq.com/download.html)
RabbitMQ是由Erlang编写，所以需要安装[Erlang](https://www.erlang.org/downloads) 

Windows安装后，按 window 键会出现如下

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410095021586.png)

这个时候点击**RabbitMQ Service - start** 就可以启动RabbitMQ服务，但是如果需要用到RabbitMQ 页面管理工具，就需要开启管理工具插件

点击 **RabbitMQ Command  Prompt**，输入 **rabbitmq-plugins list** 就可以查看所有插件激活状态，执行 **rabbitmq-plugins enable rabbitmq_management** 启用 **rabbitmq_management**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410095032145.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)

重启 RabbitMQ服务 ，访问 http://127.0.0.1:15672/ 就可以访问RabbitMQ管理页面，默认账号密码都是 guest
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410095040938.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410095050712.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)

# .NET Core 集成、监听与发送
作为事件监听，就需要程序启动时就连接RabbitMQ。在 .Net Core 中，微软给我们提供了后台托管服务支持，不再像framework 时代一样需要写windows 服务程序了。Core 中托管服务是一个类，具有实现 IHostedService 接口的后台任务逻辑。IHostedService 接口为主机托管的对象定义了两种方法：StartAsync 和 StopAsync，顾名思义前者会在Web主机启动是调用，后者在主机正常关闭时触发。除了IHostedService接口外，微软还给我们提供了更加强大的抽象基类BackgroundService，你也可以继承该类并在ExecuteAsync抽象方法中写自己的业务逻辑，这样就不必关注其他的工作了。

安装RabbitMQ官方SDK ，通过命令或直接安装Nuget包都行

`
Install-Package RabbitMQ.Client
`
 
配置文件中新增连接RabbitMQ服务实例的配置
 
```csharp
"MessageQueue": {
    "RabbitConnect": {
      "HostName": "127.0.0.1", // 主机名称
      "Port": 5672, // 主机端口
      "UserName": "guest", // 连接账号
      "Password": "guest" // 连接密码
    },
    "QueueBase": {
      "Exchange": "face.message.topic", // 交换机名称
      "Queue": "face.host.queue", // 队列名称
      "ExchangeType": "topic", // 交换机类型 direct、fanout、headers、topic 必须小写
      "ClientQueue": "face.client.queue" // 客户端队列名称
    }
}
```

约定好配置文件的格式后，新建 .Net Core 强类型选项类。

    
```csharp
    /// <summary>
    /// Rabbit 强类型配置
    /// </summary>
    public class RabbitConnectOption
    {
        /// <summary>
        /// Gets or sets the name of the host.
        /// </summary>
        /// <value>
        /// The name of the host.
        /// </value>
        public string HostName { get; set; }
        /// <summary>
        /// Gets or sets the port.
        /// </summary>
        /// <value>
        /// The port.
        /// </value>
        public int Port { get; set; }
        /// <summary>
        /// Gets or sets the name of the user.
        /// </summary>
        /// <value>
        /// The name of the user.
        /// </value>
        public string UserName { get; set; }
        /// <summary>
        /// Gets or sets the password.
        /// </summary>
        /// <value>
        /// The password.
        /// </value>
        public string Password { get; set; }
        /// <summary>
        /// Gets or sets the virtual host.
        /// </summary>
        /// <value>
        /// The virtual host.
        /// </value>
        public string VirtualHost { get; set; }
        /// <summary>
        /// Gets or sets the connection.
        /// </summary>
        /// <value>
        /// The connection.
        /// </value>
        public IConnection Connection { get; set; }
        /// <summary>
        /// Gets or sets the channel.
        /// </summary>
        /// <value>
        /// The channel.
        /// </value>
        public IModel Channel { get; set; }
    }

    /// <summary>
    /// 队列配置基类
    /// </summary>
    public class QueueBaseOption
    {
        /// <summary>
        /// 交换机名称
        /// </summary>
        /// <value>
        /// The exchange.
        /// </value>
        public string Exchange { get; set; }
        /// <summary>
        /// 队列名称
        /// </summary>
        /// <value>
        /// The queue.
        /// </value>
        public string Queue { get; set; }
        /// <summary>
        /// 交换机类型 direct、fanout、headers、topic 必须小写
        /// </summary>
        /// <value>
        /// The type of the exchange.
        /// </value>
        public string ExchangeType { get; set; }
        /// <summary>
        /// 路由
        /// </summary>
        /// <value>
        /// The route key.
        /// </value>
        //public string RouteKey { get; set; }

        /// <summary>
        /// 客户端队列名称
        /// </summary>
        /// <value>
        /// The client queue
        public string ClientQueue { get; set; }
    }

    /// <summary>
    /// 约定 强类型配置
    /// </summary>
    public class MessageQueueOption
    {
        /// <summary>
        /// rabbit 连接配置
        /// </summary>
        /// <value>
        /// The rabbit connect option.
        /// </value>
        public RabbitConnectOption RabbitConnect { get; set; }

        /// <summary>
        /// 批量推送消息到云信的队列配置信息
        /// </summary>
        /// <value>
        /// 
        /// </value>
        public QueueBaseOption QueueBase { get; set; }

    }
```

定义RabbitMQ 监听服务抽象类 RabbitListenerHostService ， 继承 IHostedService

    
```csharp
/// <summary>
    /// RabbitMQ 监听服务
    /// </summary>
    /// <seealso cref="Microsoft.Extensions.Hosting.IHostedService" />
    public abstract class RabbitListenerHostService : IHostedService
    {
        /// <summary>
        /// 
        /// </summary>
        protected IConnection _connection;
        /// <summary>
        /// The channel
        /// </summary>
        protected IModel _channel;
        /// <summary>
        /// The rabbit connect options
        /// </summary>
        private readonly RabbitConnectOption _rabbitConnectOptions;
        /// <summary>
        /// The logger
        /// </summary>
        protected readonly ILogger _logger;
        /// <summary>
        /// Initializes a new instance of the <see cref="RabbitListenerHostService" /> class.
        /// </summary>
        /// <param name="messageQueueOption"></param>
        /// <param name="logger">The logger.</param>
        public RabbitListenerHostService(IOptions<MessageQueueOption> messageQueueOption, ILogger logger)
        {
            _rabbitConnectOptions = messageQueueOption.Value?.RabbitConnect;
            _logger = logger;
        }
        /// <summary>
        /// Triggered when the application host is ready to start the service.
        /// </summary>
        /// <param name="cancellationToken">Indicates that the start process has been aborted.</param>
        /// <returns></returns>
        public Task StartAsync(CancellationToken cancellationToken)
        {
            try
            {
                if (_rabbitConnectOptions == null) return Task.CompletedTask;
                var factory = new ConnectionFactory()
                {
                    HostName = _rabbitConnectOptions.HostName,
                    Port = _rabbitConnectOptions.Port,
                    UserName = _rabbitConnectOptions.UserName,
                    Password = _rabbitConnectOptions.Password,
                };
                _connection = factory.CreateConnection();
                _channel = _connection.CreateModel();
                _rabbitConnectOptions.Connection = _connection;
                _rabbitConnectOptions.Channel = _channel;
                Process();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Rabbit连接出现异常");
            }
            return Task.CompletedTask;
        }

        /// <summary>
        /// Triggered when the application host is performing a graceful shutdown.
        /// </summary>
        /// <param name="cancellationToken">Indicates that the shutdown process should no longer be graceful.</param>
        /// <returns></returns>
        public Task StopAsync(CancellationToken cancellationToken)
        {
            if (_connection != null)
                this._connection.Close();
            return Task.CompletedTask;
        }

        /// <summary>
        /// Processes this instance.
        /// </summary>
        protected abstract void Process();
    }
```

定义一个真正处理消息的监听服务 MessageSubscribeHostService 继承 RabbitListenerHostService，实现其中的抽象方法 Process()。

    
```csharp
    /// <summary>
    /// 
    /// </summary>
    /// <seealso cref="Magicodes.Admin.Application.Core.RabbitMQ.RabbitListenerHostService" />
    public class MessageSubscribeHostService : RabbitListenerHostService
    {
        /// <summary>
        /// The services
        /// </summary>
        private readonly IServiceProvider _services;
        /// <summary>
        /// The batch advance option
        /// </summary>
        private readonly QueueBaseOption _queueBaseOption;
        /// <summary>
        /// Initializes a new instance of the <see cref="MessageSubscribeHostService" /> class.
        /// </summary>
        public MessageSubscribeHostService(IServiceProvider services, IOptions<MessageQueueOption> messageQueueOption,
            ILogger<MessageSubscribeHostService> logger) : base(messageQueueOption, logger)
        {
            _services = services;
            _queueBaseOption = messageQueueOption.Value?.QueueBase;
        }
        /// <summary>
        /// Processes this instance.
        /// </summary>
        protected override void Process()
        {
            _logger.LogInformation("调用ExecuteAsync");
            using (var scope = _services.CreateScope())
            {
                try
                {
                    //我们在消费端 从新进行一次 队列和交换机的绑定 ，防止 因为消费端在生产端 之前运行的 问题。
                    _channel.ExchangeDeclare(_queueBaseOption.Exchange, _queueBaseOption.ExchangeType, true);
                    _channel.QueueDeclare(_queueBaseOption.ClientQueue, true, false, false, null);
                    _channel.QueueBind(_queueBaseOption.ClientQueue, _queueBaseOption.Exchange, "face.log", null);
                    _logger.LogInformation("开始监听队列：" + _queueBaseOption.ClientQueue); // 监听客户端列队
                    _channel.BasicQos(0, 1, false);//设置一个消费者在同一时间只处理一个消息，这个rabbitmq 就会将消息公平分发
                    var consumer = new EventingBasicConsumer(_channel);
                    consumer.Received += (ch, ea) =>
                    {
                        try
                        {
                            var content = Encoding.UTF8.GetString(ea.Body);
                            _logger.LogInformation($"{_queueBaseOption.ClientQueue}[face.log]获取到消息：{content}");
                            // TODO: 对接业务代码
                        }
                        catch (Exception ex)
                        {
                            _logger.LogError(ex, "");
                        }
                        finally
                        {
                            _channel.BasicAck(ea.DeliveryTag, false);
                        }
                    };

                    _channel.BasicConsume(_queueBaseOption.ClientQueue, false, consumer);
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "");
                }
            }
        }
    }
```

定义消息推送服务类 MessagePublishService

    
```csharp
public class MessagePublishService : ITransientDependency
    {
        /// <summary>
        /// The channel
        /// </summary>
        protected IModel _channel;
        /// <summary>
        /// The batch advance option
        /// </summary>
        private readonly QueueBaseOption _queueBaseOption;

        public MessagePublishService(IOptions<MessageQueueOption> messageQueueOption)
        {
            _queueBaseOption = messageQueueOption.Value?.QueueBase;
            _channel = messageQueueOption.Value?.RabbitConnect.Channel;
        }

        /// <summary>
        /// 发送消息
        /// </summary>
        public void SendMsg<T>(T msg , string routeKey)
        {
            _channel.ExchangeDeclare(_queueBaseOption.Exchange, _queueBaseOption.ExchangeType, true);
            _channel.QueueDeclare(_queueBaseOption.Queue, true, false, false, null);
            _channel.QueueBind(_queueBaseOption.Queue, _queueBaseOption.Exchange, routeKey, null);
            var basicProperties = _channel.CreateBasicProperties();
            //1：非持久化 2：可持久化
            basicProperties.DeliveryMode = 2;
            var json = JsonConvert.SerializeObject(msg);
            var payload = Encoding.UTF8.GetBytes(json);
            var address = new PublicationAddress(ExchangeType.Direct, _queueBaseOption.Exchange, routeKey);
            _channel.BasicPublish(address, basicProperties, payload);
        }
    }
```
新建 IServiceCollection 的扩展类，方便在Startup 中注入服务。

    
```csharp
    /// <summary>
    /// 依赖注入扩展类
    /// </summary>
    public static class RabbitRegistrar
    {
        /// <summary>
        /// Adds the register queue sub.
        /// </summary>
        /// <param name="services">The services.</param>
        /// <param name="option">The option.</param>
        /// <returns></returns>
        public static IServiceCollection AddMessageQueueOption(this IServiceCollection services, Action<MessageQueueOption> option)
        {
            services.Configure(option);
            return services;
        }
        /// <summary>
        /// Adds the register queue sub.
        /// </summary>
        /// <param name="services">The services.</param>
        /// <param name="configuration">The configuration.</param>
        /// <returns></returns>
        public static IServiceCollection AddMessageQueueOption(this IServiceCollection services, IConfiguration configuration)
        {
            services.Configure<MessageQueueOption>(configuration);
            return services;
        }
        /// <summary>
        /// 服务自注册，实现自管理
        /// </summary>
        /// <param name="services">The services.</param>
        /// <returns></returns>
        public static IServiceCollection AddMessageSubscribeHostService(this IServiceCollection services)
        {
            services.AddHostedService<MessageSubscribeHostService>();
            return services;
        }
    }
```

Startup ConfigureServices 中注入服务

```csharp
RabbitRegistrar.AddMessageQueueOption(services, _appConfiguration.GetSection("MessageQueue"));
RabbitRegistrar.AddMessageSubscribeHostService(services);
```

启动项目，如果能在RabbitMQ管理页面看到连接存在，则配置成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410095111678.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410095118613.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200410095126423.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2R6MTgyMjgwMjc4NQ==,size_16,color_FFFFFF,t_70)