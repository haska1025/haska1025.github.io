---
layout: post
title:  "stddef offsetof"
date:   2016-11-17 13:47:53
categories: c
---

### offsetof 说明

在c标准的stddef.h中有一个获取c/c++结构或者类成员偏移量的宏offsetof,具体定义如下：
```c
 #ifdef  _WIN64
#define offsetof(s,m)   (size_t)( (ptrdiff_t)&reinterpret_cast<const volatile char&>((((s *)0)->m)) )
#else
#define offsetof(s,m)   (size_t)&reinterpret_cast<const volatile char&>((((s *)0)->m))
#endif

#else

#ifdef  _WIN64
#define offsetof(s,m)   (size_t)( (ptrdiff_t)&(((s *)0)->m) )
#else
#define offsetof(s,m)   (size_t)&(((s *)0)->m)
#endif

```

参数说明：

s: 结构体类型

m: 是结构体数据成员名称

思想就是将地址0强制转换成类型s，然后通过求结构体数据成员地址的方式，获取偏移量。

### C程序的例子：

```c

#include <stdio.h>
struct A{
    int a1; 
    int a2; 
};
main()
{
    struct A a, *pa;
    a.a1=22;
    a.a2=33;
    pa = &a; 

    // Print the address of the data member of variable a
    printf("By variable a &a(%p) &a.a1(%p) &a.a2(%p) \n", &a, &a.a1, &a.a2);

    // Print the address of the data member of the pointer pa. The pa point to the variable a.
    printf("By pointer pa &a(%p) &pa->a1(0x%lx) &pa->a2(0x%lx)\n", &a, (size_t)&pa->a1, (size_t)&pa->a2);

    struct A *s=(struct A*)0;
    // Print the address of the data member of the pointer s. The s point to 0.
    printf("By pointer s &s->a1(%ld) &s->a2(%ld)\n", (size_t)&s->a1, (size_t)&s->a2);

    // Change s point to 100.
    s = (struct A*)100;
    printf("By pointer s &s->a1(%ld) &s->a2(%ld)\n", (size_t)&s->a1, (size_t)&s->a2);

    // First deference the pointer s, then get the address of the data memeber. 
    printf("By pointer s &(*s).a1(%ld) &(*s).a2(%ld)\n", (size_t)&(*s).a1, (size_t)&(*s).a2);
    // Dangerous? Deference the pointer s, the get value of the data member.
    // printf("By pointer s (*s).a1(%d) (*s).a2(%d)\n", (*s).a1, (*s).a2);
}

```

执行结果：

By variable a &a(0x7fff5e0bae60) &a.a1(0x7fff5e0bae60) &a.a2(0x7fff5e0bae64) 

By pointer pa &a(0x7fff5e0bae60) &pa->a1(0x7fff5e0bae60) &pa->a2(0x7fff5e0bae64)

By pointer s &s->a1(0) &s->a2(4)

By pointer s &s->a1(100) &s->a2(104)

By pointer s &(*s).a1(100) &(*s).a2(104)

从执行结果可以得出这样的结论：

1、对结构体数据成员取地址，是以结构体变量首地址为基础的。

2、获取数据成员的地址运算是指针运算，不管地址是合法的还是非法的，都可以。
因为这只是地址运算，不会操作地址所指向的 内存。

3、如果我们把地址设为0，那么对数据成员取地址，就可以获取偏移量了。

### C++例子

我们将上面的例子简单修改一下，增加一个virtual 成员函数，看看效果：

```cplusplus
#include <stdio.h>
struct A{
    virtual ~A(){}
    int a1; 
    int a2; 
};
main()
{
    struct A a, *pa;
    a.a1=22;
    a.a2=33;
    pa = &a; 

    // Print the address of the data member of variable a
    printf("By variable a &a(%p) &a.a1(%p) &a.a2(%p) \n", &a, &a.a1, &a.a2);

    // Print the address of the data member of the pointer pa. The pa point to the variable a.
    printf("By pointer pa &a(%p) &pa->a1(0x%lx) &pa->a2(0x%lx)\n", &a, (size_t)&pa->a1, (size_t)&pa->a2);

    struct A *s=(struct A*)0;
    // Print the address of the data member of the pointer s. The s point to 0.
    printf("By pointer s &s->a1(%ld) &s->a2(%ld)\n", (size_t)&s->a1, (size_t)&s->a2);

    // Change s point to 100.
    s = (struct A*)100;
    printf("By pointer s &s->a1(%ld) &s->a2(%ld)\n", (size_t)&s->a1, (size_t)&s->a2);

    // First deference the pointer s, then get the address of the data memeber. 
    printf("By pointer s &(*s).a1(%ld) &(*s).a2(%ld)\n", (size_t)&(*s).a1, (size_t)&(*s).a2);
    // Dangerous? Deference the pointer s, the get value of the data member.
    // printf("By pointer s (*s).a1(%d) (*s).a2(%d)\n", (*s).a1, (*s).a2);
}
```

执行结果：

By variable a &a(0x7fffd686f520) &a.a1(0x7fffd686f528) &a.a2(0x7fffd686f52c) 

By pointer pa &a(0x7fffd686f520) &pa->a1(0x7fffd686f528) &pa->a2(0x7fffd686f52c)

By pointer s &s->a1(8) &s->a2(12)

By pointer s &s->a1(108) &s->a2(112)

By pointer s &(*s).a1(108) &(*s).a2(112)

首个数据成员的偏移量是8，原因是因为有了vptr指针。




