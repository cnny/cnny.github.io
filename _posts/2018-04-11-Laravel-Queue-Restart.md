---
layout:     post
title:      "Laravel Queue:restart 是如何实现队列的平滑启动的"
subtitle:   ""
date:       2018-04-11 17:11:00
author:     "Cann"
header-img: "img/2018-04-11.jpg"
tags:
    - PHP
    - Laravel
---

##### 当我们使用superviosrctl reload强制重启队列进程时，如果队列中有正在执行的任务，就会造成任务丢失，而laravel的队列提供了queue:restart命令用于解决这个问题。接下来我们来分析一下它是如何实现这一功能的。

`queue:restart`命令定义于`Illuminate\Queue\Console\RestartCommand`中：

```Php
    /**
     * Execute the console command.
     *
     * @return void
     */
    public function handle()
    {
        $this->laravel['cache']->forever('illuminate:queue:restart', $this->currentTime());

        $this->info('Broadcasting queue restart signal.');
    }
```

当执行restart命令后，它仅做了一件事，就是将当前时间戳存储在redis中。

再来看`Illuminate\Queue\Worker`文件，该文件中的`daemon`方法便是Laravel队列的核心代码。

```php
    public function daemon($connectionName, $queue, WorkerOptions $options)
    {
        // do something

        $lastRestart = $this->getTimestampOfLastQueueRestart();

        while (true) {

            // do something(在这里它会去find job => exec job balabalabalabala)

            $this->stopIfNecessary($options, $lastRestart);
        }
    }
```

它首先会调用`getTimestampOfLastQueueRestart()`方法，获取上次重启时间：

```Php
    protected function getTimestampOfLastQueueRestart()
    {
        if ($this->cache) {
            return $this->cache->get('illuminate:queue:restart');
        }
    }
```

然后进入`while()`死循环，不断去redis里pop任务、执行任务。

在while语句的底部，**<u>也就是当前任务执行完后</u>**，调用`stopIfNecessary()`方法：

```Php
    protected function stopIfNecessary(WorkerOptions $options, $lastRestart)
    {
        if ($this->shouldQuit) {
            $this->kill();
        }

        if ($this->memoryExceeded($options->memory)) {
            $this->stop(12);
        } elseif ($this->queueShouldRestart($lastRestart)) {
            $this->stop();
        }
    }
```

在`queueShouldRestart()`中，它会再去redis取出重启时间，然后和守护进程启动之初获取到的重启时间比对，如果两者不相等，表示守护经常运行后，开发者执行过`queue:restart`命令，队列需要重启。

```Php
    protected function queueShouldRestart($lastRestart)
    {
        return $this->getTimestampOfLastQueueRestart() != $lastRestart;
    }
```

判断队列需要重启后，它会运行`stop()`方法，在这里，它直接使用`exit`结束while循环，退出守护进程。

```php
    public function stop($status = 0)
    {
        $this->events->dispatch(new Events\WorkerStopping);

        exit($status);
    }
```

最后，再依赖supervisor等进程管理工具重新启动进程。

#### 简而言之，当我们执行`queue:restart`时，会生成一个标识，Laravel Queue每执行完一个任务，都会去验证当前标识和进程启动之前的标识是否一致，若不一致，强制结束进程，再依赖supervisor重启进程，以此保证队列重启时，没有正在运行中的任务

