---
title: 聊聊 Interface 在 Laravel 开发中的使用
---
> 你也许听过面向接口编程，而不是面向对象编程的设计思想，如 代码到接口，而不是实现，程序到接口，使用抽象而不是具体化等。
这些所指的都是同一件事，在开发中我们的应用程序应该依赖于抽象（接口）而不是具体的（类）。
##### 为什么？
---
这是我第一次听到这个说法时的反应。为什么我要使用接口而不是类？意味着我需要创建一个接口，我还要创建一个实现该接口的类？？？这不是浪费时间吗？
+ 当然这样设计是有意义的
> 在架构师的眼中「没有什么是不会变化的」，或者说 改变 才是 不变 的。
  我们开发的业务需求随时间和不断扩张而变化，我们的代码也是如此。
  所以我们的代码必须灵活。
  代码到接口使我们的代码松散耦合且灵活。

##### 怎么做？
---
请看以下代码示例：
``` php
class Logger {

    public function log($content) 
    {
        //输出 Log 日志到文件。
        echo "Log to file";
    }
}
```
一个简单的 Logger 类将日志记录到文件，我们来在控制器中调用它。
``` php
class LogController extends Controller
{
    public function log()
    {
        $logger = new Logger();
        $logger->log('Log this');
    }
}
```
但是，如果我们想记录其他位置，如数据库、文件、云或其他呢？
我们在 Logger 类中再添加几个方法：
``` php
class Logger 
{
    public function logToDb($content) 
    {
        //输出日志到 DB。
    }

    public function logToFile($content) 
    {
        //输出 Log 日志到文件。
    }

    public function logToCloud($content) 
    {
        //输出 Log 日志到云。
    } 
}
```
然后我们还要在 LoggerController 中添加判断：
``` php
class LogController extends Controller
{
    public function log()
    {
        $logger = new Logger();

        $target = config('log.target');

        $content = 'Log this.';

        switch ($target) {
            case 'db':
                $logger->logToDb($content);
                break;
            case 'file':
                $logger->logToFile($content);
                break;
            default:
                $logger->logToCloud($content);
        }
    }
}
```
好了，我们现在可以通过配置文件把日志输出到各种终端。但我们如果还要再输出日志到 redis 呢？我们还需要再增加一个方法，并且在控制器中再加一次判断。
控制器代码很快就变得臃肿，如果还要输出日志到更多地方呢？Logger 类中每个方法如果还需要扩展呢？这对于后期维护来说并不好。
这样做同时也不符合 SOLID 原则，我们先来拆分一下 Logger 类，将职责拆分成不同的类。
``` php
//DBLogger.php
namespace App\Logs;
class DBLogger
{
    public function log($content)
    {
        //输出日志到 DB。
    }
}

//FileLogger.php
namespace App\Logs;
class FileLogger
{
    public function log($content)
    {
        //输出 Log 日志到文件。
    }
}

//CouldLogger.php
namespace App\Logs;
class CloudLogger
{
    public function log($content)
    {
        //输出 Log 日志到云。
    }
}
```
再来修改 LogController：
``` php
class LogController extends Controller
{
    public function log()
    {
        $target = config('log.target');

        switch ($target) {
            case 'db':
                (new DBLogger())->log($content);
                break;
            case 'file':
                (new FileLogger())->log($content);
                break;
            default:
                (new CouldLogger())->log($content);
        }
    }
}
```
这看上去还行，我们拆分了 Logger，如果需要添加输出日志到 redis，那就继续再加 case 吧。

但依然有一个问题就是我们的控制器「知道太多了」，它应该只去调用一个 `log()` 方法来记录，而不应该知道使用哪个 Logger 类，也不应该去实例化任何类，这样在将来有改动的时候，不论是要输出到哪里，我们都不需要再来修改 `LogController` 的代码，那应该怎么做呢？

##### Interface 出场
---
这种情况最适合使用接口来实现了，什么是接口呢？
> 接口是定义对象可以哪些执行操作的描述。
回到我们的代码，控制器只需要一个带有 `log()` 方法的 Logger 类，所以我们的接口也必须定义一个 `log()` 方法。
``` php
interface LogInterface
{
    public function log($content);
}
```
> 我一般把 Interface 接口文件放在项目的 App\Contracts 文件夹
接口只包含方法声明而不包含它的实现，这就是它被称为 `抽象` 的原因。
在我们实现接口时，实现接口的类必须提供接口中定义的 `抽象方法` 的实现细节。
再回到我们的代码，我们改写成以下：
``` php
// LogController
class LogController extends Controller
{
    public function log(LogInterface $logger)
    {
        $logger->log('log to');
    }
}

//DBLogger.php
namespace App\Logs;
use App\Contracts\LogInterface;

class DBLogger implements LogInterface
{
    public function log($content)
    {
        //输出日志到 DB。
    }
}

//FileLogger.php
namespace App\Logs;
use App\Contracts\LogInterface;

class FileLogger implements LogInterface
{
    public function log($content)
    {
        //输出 Log 日志到文件。
    }
}

//CouldLogger.php
namespace App\Logs;
use App\Contracts\LogInterface;

class CouldLogger implements LogInterface
{
    public function log($content)
    {
        //输出 Log 日志到云。
    }
}
```
现在我们的代码灵活且松耦合，无需触及现有代码，就可以随时改变 Logger 的实现来应对需求的变化：
``` php
class RedisLogger implements Logger
{
    public function log($content)
    {
        //输出 Log 日志到redis。
    }
}
```
##### 依赖注入
---
在使用 Laravel 框架时，我们可以利用它的服务容器来自动注入接口的实现。
我们先新建一个配置文件 `config/log.php`
``` php
<?php

return [
    'default' => env('LOG_TARGET', 'file'),

    'file' => [
        'class' => App\Logs\FileLogger::class,
    ],

    'db' => [
        'class' => App\Logs\DBLogger::class,
    ],

    'redis' => [
        'class' => App\Logs\RedisLogger::class,
    ]
];
```
并在 `app/Providers/AppServiceProvider.php` 添加以下代码。
``` php
public function register()
{
    $default = config('log.default');
    $logger = config("log.{$default}.class");

    $this->app->bind(
        \App\Contracts\LogInterface::class, 
        $logger
    );
}
```
我们从配置文件中读取默认 Logger，并将其绑定到 `LogInterface`。这样每当我们请求 Logger 接口时，容器都会解析它并返回默认的 Logger 实例。
默认 Logger 是在 `env()` 配置的，我们可以在不同的环境中使用不同的 Logger，例如本地环境中记录到文件、生产环境中记录到数据库。

##### 总结
---
接口允许我们创建松散耦合的代码，同时提供一定程度的抽象。它允许我们随时更改我们的实现，而无需更改它们的上下文。所以我们应该将应用程序中的所有可能会有变化的部分使用接口来实现。

在大型应用中，接口是很有帮助的。和提升的代码灵活性、可测试性相比，多敲几下键盘花费的时间就显得微不足道了。当你在不同的接口实现类之间切换如飞的时候，你的经理一定会被你的神速惊到。此外，你也能够写出更能适应变化的代码。

当然，你如果在中小型项目中，不喜欢使用接口原则那也没什么不对，记住「Code Happy」快乐撸码。不过还是建议你在闲暇时间好好评估一下这件事。
enjoy 🎉
————————————————
原文作者：MArtian
转自链接：https://learnku.com/articles/67827