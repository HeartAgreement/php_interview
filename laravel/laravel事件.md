1. 事件类保存在 app/Events 目录中
2. **事件只是保存的数据的容器**
3. 而这些事件的的监听器则被保存在 app/Listeners 目录下
4. event(new 事件);发送事件给监听器
```
//监听注入在 app/Providers/EventServiceProvider.php 文件中
protected $listen = [
    当这个事件被触发的时候就调用监听（listeners） 在监听中执行监听触发逻辑
    一个事件可以对应多个监听
    'App\Events\OrderShipped' => [
        'App\Listeners\SendShipmentNotification',
    ],
];
```
##一.生成自动监听命令：
```
php artisan event:generate
```
##二.手动监听：
```
//app/Providers/EventServiceProvider.php中
    public function boot()
    {
        parent::boot();
                               //监听处理 可以是监听器的名字
        Event::listen('event.name', function ($foo, $bar) {//闭包或者监听器
            //
        });
    }
```

##例子
###一个Event例子： 事件只是保存数据的容器
```
namespace App\Events;
use App\Order;
use Illuminate\Queue\SerializesModels; //序列化model
class OrderShipped
{
    use SerializesModels;  //序列化model
    public $order;
    /**
     * 创建一个事件实例。
     *
     * @param  Order  $order
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }
}
```
##一个listen例子:监听某个事件
```
namespace App\Listeners;
use App\Events\OrderShipped;
class SendShipmentNotification
{
    /**
     * 创建事件监听器。
     * @return void
     */
    public function __construct()
    {
 你的事件监听器也可以在构造函数中加入任何依赖关系的类型提示。
 所有的事件监听器都是通过 Laravel 的 服务容器 来解析的，因此所有的依赖都将会被自动注入。
    }
 
    /**
     * 处理事件
     * @param  OrderShipped  $event
     * @return void
     */
    public function handle(OrderShipped $event)
    {
        // 使用 $event->order 来访问 order ...
        停止事件传播
        你可以通过在监听器的 handle 方法中返回
         false 来阻止事件被其他的监听器获取。
    }
}
```
##三.事件监听器队列:
1. 需要继承 implements ShouldQueue
2. 其实还是要有一个队列来存储处理（）
```
php artisan event:generate 

namespace App\Listeners;
use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
class SendShipmentNotification implements ShouldQueue
{
    /**
     * 任务应该发送到的队列的连接的名称
     *
     * @var string|null
     */
    public $connection = 'sqs';

    /**
     * 任务应该发送到的队列的名称
     *
     * @var string|null
     */
    public $queue = 'listeners';
     * 创建事件监听器。
    public function __construct()
    {
    }
     * 处理事件
    public function handle(OrderShipped $event)
    {
        // 使用 $event->order 来访问 order ...
    }
}
```
3. 手动访问监听器下面队列任务的 delete 和 release 方法，
你可以添加 Illuminate\Queue\InteractsWithQueue trait 来实现。
这个 trait 会默认加载到生成的监听器中，并提供对这些方法的访问(涉及队列知识md)
```
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;
    /**
         * 处理任务失败
         *
         * @param  \App\Events\OrderShipped  $event
         * @param  \Exception  $exception
         * @return void
         */
        //事件监听器的队列任务可能会失败，
       // 而如果监听器的队列任务超过了队列中定义的最大尝试次数，则会监听器上调用 failed 方法。
       // failed 方法接受接收事件实例和导致失败的异常作为参数：
        public function failed(OrderShipped $event, $exception)
        {
            //
        }
}
```
##四、编写事件订阅者
>事件订阅者是一个可以在自身内部订阅多个事件的类，
即能够在单个类中定义多个事件处理器。
订阅者应该定义一个 subscribe 方法，这个方法接受一个事件分发器的实例。
你可以调用给定的事件分发器上的 listen 方法来注册事件监听器：
```
namespace App\Listeners;
class UserEventSubscriber
{
    /**
     * 处理用户登录事件。
     */
    public function onUserLogin($event) {}
    /**
     * 处理用户注销事件。
     */
    public function onUserLogout($event) {}
    /**
     * 为订阅者注册监听器。
     *
     * @param  Illuminate\Events\Dispatcher  $events
     */
    public function subscribe($events)
    {
        $events->listen(
            'Illuminate\Auth\Events\Login',
            'App\Listeners\UserEventSubscriber@onUserLogin'
        );
        $events->listen(
            'Illuminate\Auth\Events\Logout',
            'App\Listeners\UserEventSubscriber@onUserLogout'
        );
    }
}
```
1. 注册事件订阅者
```
//监听注入在 app/Providers/EventServiceProvider.php 文件中
    /**
     * 需要注册的订阅者类。
     * @var array
     */
    protected $subscribe = [
        'App\Listeners\UserEventSubscriber',
    ];
```