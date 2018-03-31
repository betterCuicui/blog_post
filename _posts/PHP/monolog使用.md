---
title: monolog使用
date: 2017-06-14 11:45:23
tags: [php,composer,monolog]
category: [PHP]
---
monolog是php的一款非常强大的日志神器，可以自定义把日志发送到你的文件、socket、邮件、数据库甚至云端。并且可以同时发送。
<!--more-->
## 安装monolog
```
{
  "require": {
    "monolog/monolog": "1.0.*"
  }
}
```
使用composer安装
## 使用
```
$log_warning = new Monolog\Logger('majiang_warning');//自定义通道名字
$log_warning->pushHandler(new Monolog\Handler\StreamHandler(LOG.'/A.log', Monolog\Logger::WARNING));//通过把日志发送给handller进行打印。StreamHandler打印到文件，这里可以设计多个打印信息，比如也可以同时发送邮件打印。并且是先调用后者
$log_warning->pushHandler(new FirePHPHandler());
 
$log_warning->addWarning('Foo');
$log_warning->addWarning('Foo'，array('fdsafd','rewqrewq'));//允许传递两个参数，第二个参数是数组
 
$log_error = new Monolog\Logger('majiang_warning');
$log_error->pushHandler(new Monolog\Handler\StreamHandler(LOG.'/B.log',Monolog\Logger::ERROR));
 
$log_error->addError('Bar');
```
 
## 日志等级
*   **DEBUG** (100): .
*   **INFO** (200): .
*   **NOTICE** (250): .
*   **WARNING** (300): .
*   **ERROR** (400): .
*   **CRITICAL** (500): .
*   **ALERT** (550): .
*   **EMERGENCY** (600): .
 
## 各种handler
- StreamHandler：把记录写进PHP流，主要用于日志文件。
- SyslogHandler：把记录写进syslog。
- ErrorLogHandler：把记录写进PHP错误日志。
- NativeMailerHandler：使用PHP的mail()函数发送日志记录。
- SocketHandler：通过socket写日志。
- AmqpHandler：把记录写进兼容amqp协议的服务。
- BrowserConsoleHandler：把日志记录写到浏览器的控制台。由于是使用浏览器的console对象，需要看浏览器是否支持。
- RedisHandler：把记录写进Redis。
- MongoDBHandler：把记录写进Mongo。
- ElasticSearchHandler：把记录写到ElasticSearch服务。
- BufferHandler：允许我们把日志记录缓存起来一次性进行处理。
## 各种formatter
- LineFormatter：把日志记录格式化成一行字符串。
- HtmlFormatter：把日志记录格式化成HTML表格，主要用于邮件。
- JsonFormatter：把日志记录编码成JSON格式。
- LogstashFormatter：把日志记录格式化成logstash的事件JSON格式。
- ElasticaFormatter：把日志记录格式化成ElasticSearch使用的数据格式。
 
## 自定义日志格式
```
use Monolog\Formatter\LineFormatter;
$dateFormat  = "Y n j, g:i a";
// the default output format is [%datetime%] %channel%.%level_name%: %message% %context% %extra%\n"
$output = "%datetime% > %level_name% > %message% %context% %extra%\n";
$formatter = new LineFormatter($output, $dateFormat);
$handler = new Monolog\Handler\StreamHandler(LOG.'/B.log',Monolog\Logger::ERROR);
$handler->setFormatter($formatter);
$log_error = new Monolog\Logger('majiang');
$log_error->pushHandler($handler);
$log_error->addError('Bar');
```
这样打印出来的日志是：
```
2017 6 11, 11:14 pm > ERROR > Bar [] []
```
 
## 使用processors
在日志格式中，我们可以看到是
```
%datetime% > %level_name% > %message% %context% %extra%\n"
```
- datetime系统自带时间
- level_name：日志类型，比如error
- message。日志消息
- context：日志内容
- extra？？？：Processors 可以是任何可调用的方法（回调）。它们接受`$record`作为参数，然后返回它（`$record`）,返回之前，即是我们添加**额外信息**的操作，在这里，这个操作是改变`$record`的`extra`key的值。eg：
```
$logger->pushProcessor(function ($record) {
    $record['extra']['dummy'] = 'Hello world!';
    return $record;
});
```