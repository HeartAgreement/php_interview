#队列
1. 调用队列   队列名字::dispatch($podcast);//默认队列
2. 调用队列   队列名字::dispatch($podcast)->onQueue('emails');;//指定队列 看 [tips1说明文字](#tips1)
3.  队列名字::dispatch($podcast)->delay(Carbon::now()->addMinutes(10));延迟10分钟
> Laravel 队列监控面板composer包[Horizon](https://laravel-china.org/docs/laravel/5.5/horizon/1345)
1. 队列配置文件存放在 config/queue.php
>在使用列表里的队列服务前，必须安装以下依赖扩展包：
 1. Amazon SQS: aws/aws-sdk-php ~3.0
 2. Beanstalkd: pda/pheanstalk ~3.0
 3. Redis: predis/predis ~1.0
##database


```
php artisan queue:table
php artisan migrate
```
##创建任务
```
php artisan make:job SendReminderEmail
```
###例子
```
<?php
namespace App\Jobs;
use App\Podcast;
use App\AudioProcessor;
use Illuminate\Bus\Queueable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    protected $podcast;
    /**
     * 创建一个新的任务实例。
     * @param  Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }
    /**
     * 运行任务。
     * @param  AudioProcessor  $processor
     * @return void
     */
    public function handle(AudioProcessor $processor)
    {
        // Process uploaded podcast...
    }
}
```
1.tips
>我们在任务类的构造器中直接传递了一个 Eloquent 模型。
因为我们在任务类里引用了 SerializesModels 这个 trait，
使得 Eloquent 模型在处理任务时可以被优雅地序列化和反序列化。
如果你的队列任务类在构造器中接收了一个 Eloquent 模型，
那么只有可识别出该模型的属性会被序列化到队列里。
当任务被实际运行时，队列系统便会自动从数据库中重新取回完整的模型。
这整个过程对你的应用程序来说是完全透明的，
这样可以避免在序列化完整的 Eloquent 模式实例时所带来的一些问题。

2.tips
>像图片内容这种二进制数据，
在放入队列任务之前必须使用 base64_encode 方法转换一下。
否则，当这项任务放置到队列中时，可能无法正确序列化为 JSON。
##工作链
```
//工作链允许你指定应该按顺序运行的队列列表。
//如果一个任务失败了，则其余任务将不会运行
//你可以在分发任务的时候使用 withChain 方法来执行具有工作链的队列任务。
ProcessPodcast::withChain([
    new OptimizePodcast,
    new ReleasePodcast
])->dispatch();
```
##自定义队列 & 连接
```
//指定链接到那个驱动中去
ProcessPodcast::dispatch($podcast)->onConnection('Redis');
ProcessPodcast::dispatch($podcast)->onConnection('Redis')
->onQueue('processing');//指定队列
```
##最大尝试次数 / 超时
```
php artisan queue:work --tries=3
php artisan queue:work --timeout=30
//或  job
class ProcessPodcast implements ShouldQueue
{   
    public $timeout = 120;
    public $tries = 5;
}
```
#重点 命令
##运行队列命令 / 重启队列
```
php artisan queue:work
```
>要让 queue:work 进程永久在后台运行，你应该使用进程监控工具，
比如 Supervisor 来保证队列处理器没有停止运行
一定要记得，队列处理器是长时间运行的进程，
并在内存里保存着已经启动的应用状态。
这样的结果就是，处理器运行后如果你修改代码那这些改变是不会应用到处理器中的。
所以在你重新部署过程中，一定要 重启队列处理器 。
```
//重启队列
php artisan queue:restart
//队列处理器在执行完当前任务后结束进程
//因为队列处理器在执行 queue:restart 命令时对结束进程，(先开一个再关一个)
//你应该运行一个进程管理器，比如 Supervisor 来自动重新启动队列处理器。
//队列使用 缓存 来存储重新启动信号，所以在使用此功能之前，
//你应该确保应用程序的缓存驱动程序已正确配置。
```
##处理单一任务
```
//--once 选项来指定仅对队列中的单一任务进行处理：
php artisan queue:work --once
```
##指定连接 or 队列
```
//你可以指定队列处理器所使用的连接。
//你在 config/queue.php 配置文件里定义了多个连接，
//而你传递给 work 命令的连接名字要至少跟它们其中一个是一致的：
php artisan queue:work redis
```

##自定义队列处理器   <span value='tips1' id = "tips1"></span>
```
//你可以自定义队列处理器，
//方式是处理给定连接的特定队列。
//举例来说，如果你所有的邮件都是在 redis 连接中的 emails 队列中处理的，
//你就能通过以下命令启动一个只处理那个特定队列的队列处理器了：
php artisan queue:work redis --queue=emails
```




 