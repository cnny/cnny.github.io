---
layout:     post
title:      "Yii2从入门到幼儿园01"
subtitle:   "Dependence Injection Container"
date:       2017-10-19 20:41:00
author:     "Yuu"
header-img: "img/20171019.jpg"
tags:
    - Git
    - PHP
    - Shell
---

### Yii2从入门到幼儿园01——Dependence Injection Container

#### 什么是依赖注入（Dependence Injection， DI）

简言之，就是将类所需要的依赖在外部实例化后再传入类中，以此降低代码的耦合性。

普通代码：

```
class test {

    public $emailer;

    public function __construct()
    {
        $this->emailer = new Emailer();
    }

    public function sendEmail($to, $content)
    {
        $this->emailer->send($to, $eontent);
    }
}

new test()->sendEmail('xxx', 'xxx');
```

依赖注入：

```
class test {

    public $emailer;

    public function __construct($emailer)
    {
        $this->emailer = $emailer;
    }

    public function sendEmail($emailer, $to, $content)
    {
        $emailer->send($to, $eontent);
    }
}

new test(new Emailer())->sendEmail('xxx', 'xxx');
```


####  什么是依赖注入容器（Dependence Injection Container， DI Container）

依赖注入有个很大的缺点，类中所有的依赖都要手动实例化然后传入类中，当代码量变大，层级变多的时候，就会产生很多问题，诸如：依赖实例化代码冗余、实例化需要按照一定顺序、多人协作容易重复实例化等等。

DI Container 便是为了解决这个问题，他将类对外部的依赖转变为对DI Container的依赖。类中所需的依赖不再需要手动传入，而是自动去DI Container中获取。

举个Yii2官方的栗子：

```
 interface UserFinderInterface
 {
     function findUser();
 }

 class UserFinder extends Object implements UserFinderInterface
 {
     public $db;

     public function __construct(Connection $db, $config = [])
     {
         $this->db = $db;
         parent::__construct($config);
     }

     public function findUser()
     {
     }
 }

 class UserLister extends Object
 {
     public $finder;

     public function __construct(UserFinderInterface $finder, $config = [])
     {
         $this->finder = $finder;
         parent::__construct($config);
     }
 }
```

若要运行上面的代码，传统写法：

```
 $db = new \yii\db\Connection(['dsn' => '...']);
 $finder = new UserFinder($db);
 $lister = new UserLister($finder);
```

可以看到，上面三次实例化必须按照顺序，先后顺序丝毫不能错，若层级、依赖变多，代码会变得非常繁杂。

DI Container:

```
 // 这就是我们上面说的 DI Container
 $container = new Container;

 // 注册依赖
 $container->set('yii\db\Connection', [
     'dsn' => '...',
 ]);
 $container->set('app\models\UserFinderInterface', [
     'class' => 'app\models\UserFinder',
 ]);

 $container->set('userLister', 'app\models\UserLister');

 // 获取实例
 $lister = $container->get('userLister');
```

可以看到，只要在最初将类中所需的依赖注册进入DI Container后，在实例化的时候，完全不需要关心类的内部是如何调用的。

当然，乍一看好像代码反而比之前多了，但是放到整个项目里便会发现方便很多，代码也清晰很多。我们只需要在一个统一的地方去注册依赖，之后实现业务的时候只需要关注实例的获取就好。

例如Yii2 Config中Component配置，便是一个依赖注入的参数配置过程。

另外，类中的依赖也并非全部都要事先注册进 DI Container，当它发现一个未注册的依赖，会利用PHP的反射机制（Reflection）实现自动注册。它的实现接下来也会说到。

#### Yii2中DI Container的实现

>代码路径 vendor/yiisoft/yii2/di/Container.php

我们可以看到，在Container中，维护了5个数组，这是DI Container实现的基础

```
// 用于保存单例Singleton对象，以对象类型为键
private $_singletons = [];

// 用于保存依赖的定义，以对象类型为键
private $_definitions = [];

// 用于保存构造函数的参数，以对象类型为键
private $_params = [];

// 用于缓存ReflectionClass对象，以类名或接口名为键
private $_reflections = [];

// 用于缓存依赖信息，以类名或接口名为键
private $_dependencies = [];
```

让我们随着DI Container执行的顺序一步步解析它。

#### 注册依赖

注册依赖，也就是之前看到的`$container->set(['xx' => 'xx'])`，它的源码如下

```
    public function set($class, $definition = [], array $params = [])
    {
        $this->_definitions[$class] = $this->normalizeDefinition($class, $definition);
        $this->_params[$class] = $params;
        unset($this->_singletons[$class]);
        return $this;
    }

    protected function normalizeDefinition($class, $definition)
    {
        if (empty($definition)) {
            return ['class' => $class];
        } elseif (is_string($definition)) {
            return ['class' => $definition];
        } elseif (is_callable($definition, true) || is_object($definition)) {
            return $definition;
        } elseif (is_array($definition)) {
            if (!isset($definition['class'])) {
                if (strpos($class, '\\') !== false) {
                    $definition['class'] = $class;
                } else {
                    // throw error
                }
            }
            return $definition;
        } else {
            // throws error
        }
    }
```
`set()`方法接收三个参数
**`$class`**：我们为将要注册的依赖取的名称。
**`$definition`**：该依赖的定义。该参数包含两部分内容，键名：`class`，该依赖的真实类名。其他键名：该依赖实例化时，需要传入的构造函数参数
    *$definition 的传参类型有许多种，可以是字符串，可以是数组，也可以是对象、回调函数，最后`set()`都会利用`normalizeDefinition()`进行规范化处理。最后统一成带有`class`键名的数组，键值则是这个依赖的真实类名
**`$params`**：该依赖的构造函数所需要的参数

我们可以看到，这里的`set()`只做了两件事

1. **将用户传入的$definition存入`$_definitions`中**。
2. **将用户传入的$parmas存入`$_params`**中。

所以注册依赖的时候是不会去创建实例的，它只是将这个依赖对应的 class name 和它的所需的参数保存起来。实例化这个操作是在`get()`方法中

#### 获取实例

`get()`中逻辑比较复杂，我把它分了一下，并且在不影响逻辑的前提下调整了部分代码的先后顺序，以便解析起来更清晰些：
```
    public function get($class, $params = [], $config = [])
    {
        // 第一部分
        if (isset($this->_singletons[$class])) {
            // singleton
            return $this->_singletons[$class];
        }

        // 第二部分
        if (isset($this->_definitions[$class])) {

            $definition = $this->_definitions[$class];

            if (is_array($definition)) {

                $config = array_merge($definition, $config);
                $params = $this->mergeParams($class, $params);

                if ($class === $definition['class']) {
                    $object = $this->build($class, $params, $config);
                } else {
                    $object = $this->get($definition['class'], $params, $config);
                }
            }

            elseif (is_object($definition) || is_callable($definition, true)) {
                // 对象直接返回、回调执行后返回
            }
            else {
                // throw error
            }
        }

        // 第三部分
        else {
            return $this->build($class, $params, $config);
        }

        // 第四部分
        if (array_key_exists($class, $this->_singletons)) {
            // singleton
            $this->_singletons[$class] = $object;
        }

        return $object;
    }
```

##### 第一部分

这一部分和第四部分一起解析，暂且不谈

##### 第二部分

当你之前注册过该依赖，你会走到这一步。

如果$definition是已经实例化的对象、回调函数，那就直接返回，或者执行后返回，这没什么好说的。

如果是数组，逻辑就比较复杂了：

它先是取出之前储存在\$_definitions和\$_params中的配置，并且和本次传入的\$config、\$params合并，后者会覆盖前者。也就是说即使你在注册依赖时设置了配置，之后在获取实例时也仍然可以重新定义它:

```
    $config = array_merge($definition, $config);
    $params = $this->mergeParams($class, $params);
```

接下来这个判断主要是为了别名服务的：

```
    if ($class === $definition['class']) {
        $object = $this->build($class, $params, $config);
    } else {
        $object = $this->get($definition['class'], $params, $config);
    }
```

总之，最后会执行到`build()`这个核心方法，最终的实例便是这里创建出来的：


若要创建这个实例，首先，需要获取这个实例的所有依赖：

```
    protected function build($class, $params, $config)
    {
        list ($reflection, $dependencies) = $this->getDependencies($class);
        // DO SOMETHING
    }

    protected function getDependencies($class)
    {
        if (isset($this->_reflections[$class])) {
            return [$this->_reflections[$class], $this->_dependencies[$class]];
        }

        $dependencies = [];

        // 使用PHP5 的反射机制来获取类的有关信息，主要就是为了获取依赖信息
        $reflection = new ReflectionClass($class);

        // 通过类的构造函数的参数来了解这个类依赖于哪些单元
        $constructor = $reflection->getConstructor();
        if ($constructor !== null) {

            // 遍历构造函数中的所有参数
            foreach ($constructor->getParameters() as $param) {
                // // 构造函数如果有默认值，将默认值作为依赖
                if ($param->isDefaultValueAvailable()) {
                    $dependencies[] = $param->getDefaultValue();
                } else {
                    // 如果参数没有默认值，分析它的类型，若设置了类型，将类型名称放置到Instance实例中
                    // Instance仅仅是一个容器，在后面又会将其中的值取出来
                    // Instance是为了标识这是一个依赖
                    $c = $param->getClass();
                    $dependencies[] = Instance::of($c === null ? null : $c->getName());
                }
            }
        }

        $this->_reflections[$class] = $reflection;
        $this->_dependencies[$class] = $dependencies;

        return [$reflection, $dependencies];
    }

```

这里使用了反射，解析了该类的构造函数中的所有参数，接下来，会将这些参数中属于依赖的部分进行实例化：

```
    protected function build($class, $params, $config)
    {
        // DO SOMETHING

        // 将我们手动传入和注册依赖时配置的参数作为依赖导入
        foreach ($params as $index => $param) {
            $dependencies[$index] = $param;
        }

        // 实例化这个类的所有依赖
        $dependencies = $this->resolveDependencies($dependencies, $reflection);

        // DO SOMETHING
    }

    protected function resolveDependencies($dependencies, $reflection = null)
    {
        foreach ($dependencies as $index => $dependency) {
            // 这里将我们刚刚传入Instance中的参数类型名称又取了出来，并且递归获取了它的实例
            if ($dependency instanceof Instance) {
                if ($dependency->id !== null) {
                    $dependencies[$index] = $this->get($dependency->id);
                } elseif ($reflection !== null) {
                    // throws error
                }
            }
        }
        return $dependencies;
    }
```

这时候，我们已经有了真实的类名、以及这个类所需的所有依赖和参数，接下来只要实例化它就好了：

```
    protected function build($class, $params, $config)
    {
        // DO SOMETHINE

        // 如果$config为空，直接返回这个类的实例
        if (empty($config)) {
            return $reflection->newInstanceArgs($dependencies);
        }

        // yii\base\Configurable 这个类官方是这么解释的：
        // Configurable is the interface that should be implemented by classes who support configuring its properties through the last parameter to its constructor.
        if (!empty($dependencies) && $reflection->implementsInterface('yii\base\Configurable')) {
            $dependencies[count($dependencies) - 1] = $config;
            return $reflection->newInstanceArgs($dependencies);
        } else {
            // 将$config中的参数作为属性赋予这个实例
            $object = $reflection->newInstanceArgs($dependencies);
            foreach ($config as $name => $value) {
                $object->$name = $value;
            }
            return $object;
        }
    }
```

##### 第三部分

之前我们说过，依赖并不是一定要先进行注册操作，通过解析源码，我们会发现，注册依赖的过程仅仅是为依赖初始化了部分参数。

当你没有注册过依赖，会走到这一步，它会根据传入的$class，也就是真实的类名去执行build()，最终生成出一个实例。

##### 第四部分

在实际应用中，有许多类是需要单例模式的，例如Emailer 、Logger之类的。

而第四部分加上第一部分，都是为了维护部分依赖的单例模式

注册依赖除了`set()`还有一个方法:`setSingleton`，这个方法便是用来注册单例的：

```
    public function setSingleton($class, $definition = [], array $params = [])
    {
        $this->_definitions[$class] = $this->normalizeDefinition($class, $definition);
        $this->_params[$class] = $params;
        $this->_singletons[$class] = null;
        return $this;
    }
```

对比会发现，`set()`和`setSingleton()`的唯一区别便是以下这一句

```
    unset($this->_singletons[$class]); // set()
    $this->_singletons[$class] = null; // setSingleton():
```

所以`set()`是走不到第四部分的，也就不会将实例放入单例数组：$_singletons中。
而`setSingleton()`除了在第一次实例化的时候会走到这一步，之后便在第一部分就将之前创建的实例返回了。
以此实现部分依赖的单例模式

