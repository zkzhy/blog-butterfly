---
title: php-protocol
date: 2025-04-01 18:00:12
tags:
---

## php_filter_chain_generator工具

开源地址：[synacktiv/php_filter_chain_generator](https://github.com/synacktiv/php_filter_chain_generator)

这款工具可以使filter流构造时，利用不同协议的解析差使得转化字符的不同差异写入预计的shell。

抛出一道题:

```
file_put_contents($file, file_get_contents($file));
```

从代码本意上来说,两边$file是相等都一样吗，也就是说对于一个相同的文件写入一个相同的内容。但是这个时候我们可以思考php的一个特性,php://filter流制造两边的解析差。

制造漏洞的过程就是使得file get contents成为shell通过file put contents写入预计的文件中。

利用工具，构造生成的chain。

```
python phpchain.py --chain '<?php @eval($_POST[1]);?>' 
```

可以得到php://filter链，而最后的php://input可以有两种选择。

1.任意但不包含index.php,此时file get contents会产生报错，即使报错无法get预计的文件，但是file put content作为写入文件流函数可以新建出webshell。

2.特定的index.php(作为一个有内容且内容我们不可控制的文件)，则需要一点修饰。由于 convert.iconv.UTF8.CSISO2022KR 这个编码不支持中文或者其他字符，在一长串的 iconv 前面加上一个 base64-encode 使其全部变成 ASCII 字符即可，使得程序可以正常运行植入shell。——由于经过一系列复杂的字符转换，原本的内容已经成为乱码。

## data://流

使得文件函数返回对应的文本,

共同点：两者都可以使得有关文件函数返回值被输入者控制。

```
copy($a,backup/$a)
```

考虑data://流制造差异化。

data://text/plain,<?php @eval($_POST[1]);?>../../../index.php即可实现后部分目录穿越以及前部分执行<>内部预定的代码内容植入shell。
