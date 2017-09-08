---
layout: post
title:  "About GNU make"
date:   2017-09-08 16:47:53
categories: make
---
### makefile 包含5部分内容

1、显式规则(Explicit rule)

2、隐身规则(Implicity rule)

3、变量定义(Variable definition)

3.1 变量赋值 "=", 递归替换
  
3.2 变量赋值 ":="，只有一次替换

  ```c
foo := 1122
test := $(bar)
bar := $(foo)

foo1 = 2244
test1 = $(bar1)
bar1 = $(foo1)
all:
        echo "test============$(test)"
        echo "test1=======$(test1)"

结果：
echo "test============"
test============
echo "test1=======2244"
test1=======2244

```
4、指令(directive)

4.1 加载另一个makefile，用include指令

4.2 条件判断语句 ifeq/ifneq endif

4.3 define指令

5、注释

### makefile 规则

target ... : prerequisites ...
	recipe
	...
	...

target: makefile要生成的文件名或者中间太目标

prerequisites: 生成target需要的输入

recipe: makefile完成的动作，recipe可以包含多条命令，每一条命令需要单独一行。

每一个条命令要以\t开始

### makefile执行过程

1、可以通过 -f 给make指定makefile文件, 如make -f makefile.t

2、make 执行默认是找第一个target 规则，可以通过make target指定执行某一个特定的target

3、make 首先会加载makefile和include的makefile，做变量替换，替换好所有依赖的rule

4、完成某个target然后会找它的prerequisites，依次递归；直到找到prerequisites文件，
   或者prerequisites没空为止。
   
5、如果 prerequisites 文件比target文件老，那么不去执行recipe；
   如果 prerequisites 为空，那么去执行recipe；
   
6、完成一个target后，会重复第3步，继续执行下一个target，直到所有target都完成为止。

7、某个target的recipe完成后，不会重复执行。



