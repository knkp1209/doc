# RabbitMQ 的基本理解、简单使用



## RabbitMQ 模型架构，图来源于《RabbitMQ 实战指南》

![image-20211107233506757](https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20211107233506757.png)

## AMQP 协议

- AMQP，即Advanced Message Queuing Protocol，一个提供统一消息服务的应用层标准高级[消息](https://baike.baidu.com/item/消息/1619218)队列协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。

## RabbitMQ

- RabbitMQ是AMQP协议的Erlang的实现, 此外RabbitMQ还支持 STOMP、MQTT等协议。

## RabbitMQ Web 管理端

- 默认账号/密码 guest/guest，端口号：15672

## 连接

- 无论是生产者还是消费者都需要和 RabbitMQ Broker 建立连接，这个连接就是一条TCP连接

- 创建连接

  ```php
  // 建立连接
  $connection = new AMQPStreamConnection(
      'localhost', // 域名或IP地址
      5672, // 端口号
      'guest', // 账号
      'guest' // 密码
  );
  
  // 关闭连接
  $connection->close();
  ```

  

## 信道

- 信道是建立在连接之上的虚拟连接，每个信道都会被指派一个唯一的ID

- 不同连接的信道相互隔离，所以不同连接的信道 ID 可以重复
- RabbitMQ 处理的每条AMQP 指令都是通过信道完成的
- 创建信道不指定信道 ID 时，信道 ID 以1开始递增

```PHP
$connection = new AMQPStreamConnection(
    'localhost',
    5672,
    'guest',
    'guest'
);

$channel = $connection->channel(); // 创建信道
$channel = $connection->channel(); // 创建信道
$channel = $connection->channel(200); // 创建信道并指定信道ID, 信道ID必须是 数字

// 关闭信道
$channel->close();
// 关闭连接
$connection->close();
```

## 交换器

- 交换器是基于信道的

- 在RabittMQ 中，生产者并不是直接将消息直接投递到队列中的，而将消息发送到交换器中, 由交换器根据路由键将消息路由到一个或者多个队列中，如果路由不到根据生产者配置的属性则选择回退给生产者或者丢弃

- 常用交换器类型：fanout、direct、topic、headers

- 创建声明一个交换器

  ```php
  // 创建声明交换器类型
  $exchangeName = 'PP';
  $channel->exchange_declare(
      // 交换器名称
      $exchangeName,
  
      // 交换器类型
      AMQPExchangeType::DIRECT,
  
      // 是否被动模式，为 false 时，交换器不存在则创建，为 true 时交换器不存在则抛异常
      false,
  
      // 是否持久化, false: rabbitmq 服务重启会消失, true: 服务重启相关信息不会丢失
      false,
  
      // 是否自动删除， 自动删除前提是至少有一个队列或者交换器与这个交换器绑定，之后所有与这个交换器绑定的队列或者交换器都解绑才会删除， 注意不是客户端连接断开时会自动删除
      true
  );
  ```
  

  
![image-20211107162608552](https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20211107162608552.png)

## 队列

- 队列是 RabbitMQ的内部对象，用于存储消息
- 创建声明一个队列

```PHP
// 创建声明一个队列
$queueName = 'queue_test';
$channel->queue_declare(
    // 队列名称
    $queueName,

    // 是否被动模式，为 false 时，交换器不存在则创建，为 true 时交换器不存在则抛异常
    false,

    // 是否持久化, false: rabbitmq 服务重启会消失, true: 服务重启相关信息不会丢失
    true,

    // 设置是否排他。为true则设置队列为排他的。如果一个队列被声明为排他队列，该队列仅对首次声明它的连接可见，并在连接断开时自动删除。这里需要注意三点：排他队列是基于连接（Connection）可见的，同一个连接的不同信道（Channel）是可以同时访问同一连接创建的排他队列；“首次”是指如果一个连接已经声明了一个排他队列，其他连接是不允许建立同名的排他队列的，这个与普通队列不同；即使该队列是持久化的，一旦连接关闭或者客户端退出，该排他队列都会被自动删除，这种队列适用于一个客户端同时发送和读取消息的应用场景。
    false,

    //autoDelete：设置是否自动删除。为true则设置队列为自动删除。自动删除的前提是：至少有一个消费者连接到这个队列，之后所有与这个队列连接的消费者都断开时，才会自动删除。不能把这个参数错误地理解为：“当连接到此队列的所有客户端断开时，这个队列自动删除”，因为生产者客户端创建这个队列，或者没有消费者客户端与这个队列连接时，都不会自动删除这个队列。
    true
);
```

![image-20211107210145475](https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20211107210145475.png)

## 路由键、绑定键、生产者发送消息

- 当路由键与绑定键相匹配（根据交换器类型选择匹配模式）那么该消息会路由（存储）到对应队列中

- 队列绑定交换器与绑定键， 生产者发送消息内容，交换器类型为 direct 时，路由键与绑定键必须相同

  ```PHP
  // 队列绑定交换器与绑定键
  $routingKey = $bindKey = 'bind_key_test';
  $channel->queue_bind(
      // 队列名称
      $queueName,
  
      // 交换器名称
      $exchangeName,
  
      // 绑定键
      $bindKey
  );
  
  // 生产者发送消息
  $msg = new AMQPMessage('消息内容', [
      'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT, // 持久化
  ]);
  $channel->basic_publish($msg, $exchangeName, $routingKey);
  ```

  

  ![image-20211107210320601](https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20211107210320601.png)

- 路由键相当于预设的目的地 绑定键相当于真实存在的目的地

- BindingKey和RoutingKey一样都是点号“.”分隔的字符串；被点号“.”分隔开的每一段独立的字符串为一个单词，BindingKey中可以存在两种特殊字符串“*”和“＃”，用于做模糊匹配，其中“＃”用于匹配一个单词，“＃”用于匹配多规格单词（可以是零个）

- \# 匹配示例，交换器类型为 topic, 该示例展示 绑定键与路由键的区别

  ```PHP
  // 创建声明一个队列
  $queueName = 'queue_topic';
  $channel->queue_declare(
      // 队列名称
      $queueName,
  
      // 是否被动模式，为 false 时，交换器不存在则创建，为 true 时交换器不存在则抛异常
      false,
  
      // 是否持久化, false: rabbitmq 服务重启会消失, true: 服务重启相关信息不会丢失
      true,
  
      // 设置是否排他。为true则设置队列为排他的。如果一个队列被声明为排他队列，该队列仅对首次声明它的连接可见，并在连接断开时自动删除。这里需要注意三点：排他队列是基于连接（Connection）可见的，同一个连接的不同信道（Channel）是可以同时访问同一连接创建的排他队列；“首次”是指如果一个连接已经声明了一个排他队列，其他连接是不允许建立同名的排他队列的，这个与普通队列不同；即使该队列是持久化的，一旦连接关闭或者客户端退出，该排他队列都会被自动删除，这种队列适用于一个客户端同时发送和读取消息的应用场景。
  //autoDelete：设置是否自动删除。为true则设置队列为自动删除。自动删除的前提是：至少有一个消费者连接到这个队列，之后所有与这个队列连接的消费者都断开时，才会自动删除。不能把这个参数错误地理解为：“当连接到此队列的所有客户端断开时，这个队列自动删除”，因为生产者客户端创建这个队列，或者没有消费者客户端与这个队列连接时，都不会自动删除这个队列。
      false
  );
  
  // 队列绑定交换器与路由键
  
  $bindKey = 'test.#';
  $channel->queue_bind(
      // 队列名称
      $queueName,
  
      // 交换器名称
      $exchangeName,
  
      // 绑定键
      $bindKey
  );
  
  
  // 生产者发送消息
  $msg = new AMQPMessage('消息内容', [
      'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT, // 持久化
  ]);
  
  $routinKey = 'test';
  $channel->basic_publish($msg, $exchangeName, $routinKey);
  
  $routinKey = 'test.aa';
  $channel->basic_publish($msg, $exchangeName, $routinKey);
  
  $routinKey = 'test.bb.cc';
  $channel->basic_publish($msg, $exchangeName, $routinKey);
  ```

![image-20211107215739679](https://knkp-doc.oss-cn-guangzhou.aliyuncs.com/doc/image-20211107215739679.png)

## 消费者

```PHP
/**
 * 消费处理回调方法
 * @param \PhpAmqpLib\Message\AMQPMessage $message
 */
function process_message($message)
{
    echo "\n--------\n";
    echo $message->body;
    echo "\n--------\n";

    $message->ack();

    // Send a message with the string "quit" to cancel the consumer.
    if ($message->body === 'quit') {
        $message->getChannel()->basic_cancel($message->getConsumerTag());
    }
}

 // 消费者标识
$consumerTag = 'aa';
// 绑定 队列、消费者标识、消费业务逻辑
$channel->basic_consume($queueName, $consumerTag, false, false, false, false, 'process_message');

/**
 * @param \PhpAmqpLib\Channel\AMQPChannel $channel
 * @param \PhpAmqpLib\Connection\AbstractConnection $connection
 */
function shutdown($channel, $connection)
{
    $channel->close();
    $connection->close();
    echo '结束' . PHP_EOL;
}

register_shutdown_function('shutdown', $channel, $connection);

// Loop as long as the channel has callbacks registered

// 开始消费
while ($channel->is_consuming()) {
    $channel->wait();
}
```



# Demo

## 生产者

```php
<?php
/**
 * Created by PhpStorm.
 * Email: 1203860880@qq.com
 * User: YangWenHang
 * Date: 2021/11/7
 * Time: 12:02 下午
 */

require_once __DIR__.'/vendor/autoload.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
use PhpAmqpLib\Exchange\AMQPExchangeType;
use PhpAmqpLib\Message\AMQPMessage;

$connection = new AMQPStreamConnection(
    'localhost',
    5672,
    'guest',
    'guest'
);

$channel = $connection->channel(); // 创建信道

// 创建声明交换器类型
$exchangeName = 'test_topic';
$channel->exchange_declare(
    // 交换器名称
    $exchangeName,

    // 交换器类型
    AMQPExchangeType::TOPIC,

    // 是否被动模式，为 false 时，交换器不存在则创建，为 true 时交换器不存在则抛异常
    false,

    // 是否持久化, false: rabbitmq 服务重启会消失, true: 服务重启相关信息不会丢失
    true,

    // 是否自动删除， 自动删除前提是至少有一个队列或者交换器与这个交换器绑定，之后所有与这个交换器绑定的队列或者交换器都解绑才会删除，
    // 注意不是客户端连接断开时会自动删除
    true
);

// 创建声明一个队列
$queueName = 'queue_topic';
$channel->queue_declare(
    // 队列名称
    $queueName,

    // 是否被动模式，为 false 时，交换器不存在则创建，为 true 时交换器不存在则抛异常
    false,

    // 是否持久化, false: rabbitmq 服务重启会消失, true: 服务重启相关信息不会丢失
    true,

    // 设置是否排他。为true则设置队列为排他的。如果一个队列被声明为排他队列，该队列仅对首次声明它的连接可见，并在连接断开时自动删除。这里需要注意三点：排他队列是基于连接（Connection）可见的，同一个连接的不同信道（Channel）是可以同时访问同一连接创建的排他队列；“首次”是指如果一个连接已经声明了一个排他队列，其他连接是不允许建立同名的排他队列的，这个与普通队列不同；即使该队列是持久化的，一旦连接关闭或者客户端退出，该排他队列都会被自动删除，这种队列适用于一个客户端同时发送和读取消息的应用场景。
    false,

    //autoDelete：设置是否自动删除。为true则设置队列为自动删除。自动删除的前提是：至少有一个消费者连接到这个队列，之后所有与这个队列连接的消费者都断开时，才会自动删除。不能把这个参数错误地理解为：“当连接到此队列的所有客户端断开时，这个队列自动删除”，因为生产者客户端创建这个队列，或者没有消费者客户端与这个队列连接时，都不会自动删除这个队列。
    false

);

// 队列绑定交换器与路由键

$bindKey = 'test.#';
$channel->queue_bind(
    // 队列名称
    $queueName,

    // 交换器名称
    $exchangeName,

    // 绑定键
    $bindKey
);


// 生产者发送消息
$msg = new AMQPMessage('消息内容', [
    'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT, // 持久化
]);

$routinKey = 'test';
$channel->basic_publish($msg, $exchangeName, $routinKey);

$routinKey = 'test.aa';
$channel->basic_publish($msg, $exchangeName, $routinKey);

$routinKey = 'test.bb.cc';
$channel->basic_publish($msg, $exchangeName, $routinKey);


// 发送一个自定义的内空作为终止消费的标识，消费业务逻辑遇到该信息则终止
$msg = new AMQPMessage('quit', [
    'delivery_mode' => AMQPMessage::DELIVERY_MODE_PERSISTENT, // 持久化
]);

$routinKey = 'test';
$channel->basic_publish($msg, $exchangeName, $routinKey);

$channel->close();
$connection->close();
```

## 消费者

```PHP
<?php
/**
 * Created by PhpStorm.
 * Email: 1203860880@qq.com
 * User: YangWenHang
 * Date: 2021/11/3
 * Time: 12:02 下午
 */

require_once __DIR__.'/vendor/autoload.php';

use PhpAmqpLib\Connection\AMQPStreamConnection;
// 以下为消费者代码

// 创建连接
$connection = new AMQPStreamConnection(
    'localhost',
    5672,
    'guest',
    'guest'
);

$channel = $connection->channel(); // 创建信道

$queueName = 'queue_topic'; // 队列名

/**
 * 消费处理回调方法
 * @param \PhpAmqpLib\Message\AMQPMessage $message
 */
function process_message($message)
{
    echo "\n--------\n";
    echo $message->body;
    echo "\n--------\n";

    $message->ack();

    // Send a message with the string "quit" to cancel the consumer.
    if ($message->body === 'quit') {
        // 终止消费
        $message->getChannel()->basic_cancel($message->getConsumerTag());
    }
}

// 消费者标识
$consumerTag = 'aa';
// 绑定 队列、消费者标识、消费业务逻辑
$channel->basic_consume($queueName, $consumerTag, false, false, false, false, 'process_message');

/**
 * @param \PhpAmqpLib\Channel\AMQPChannel $channel
 * @param \PhpAmqpLib\Connection\AbstractConnection $connection
 */
function shutdown($channel, $connection)
{
    $channel->close();
    $connection->close();
    echo '结束' . PHP_EOL;
}

register_shutdown_function('shutdown', $channel, $connection);

// Loop as long as the channel has callbacks registered

// 开始消费
while ($channel->is_consuming()) {
    $channel->wait();
}
```
