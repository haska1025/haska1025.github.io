---
layout: post
title:  "graphic image"
date:   2018-2-28 16:47:53
categories: graphic&image
---
### YUV

http://www.discoverybiz.net/enu0/faq/faq_YUV_YCbCr_YPbPr.html

http://blog.csdn.net/suiyunonghen/article/details/3861896

```c
Y = Kry · R + Kgy · G + Kby · B 
Cb = B – Y 
Cr = R – Y 
Kry + Kgy + Kby = 1 

Y = Kry · R + Kgy · G + Kby · B 
Cb = Kru · R + Kgu · G + Kbu · B 
Cr = Krv · R + Kgv · G + Kbv · B
Where

Kru = - Kry 
Kgu = - Kgy 
Kbu = 1 - Kby 
Krv = 1 – Kry 
Kgv = -Kgy 
Kbv = -Kby
Formulae for inverse conversion from YCbCr to RGB are the following:

R = Y + Cr 
G = Y – (Kby / Kgy) · Cb – (Kry / Kgy) · Cr 
B = Y + Cb
Where

Kyg = 1 
Kug = - Kby / Kgy 
Kvg = - Kry / Kgy

Reference standard:              |  Kry    | Kby
---------------------------------+---------+---------
ITU601 / ITU-T 709 1250/50/2:1   |  0.299  | 0.114
---------------------------------+---------+---------
ITU709 / ITU-T 709 1250/60/2:1   | 0.2126  | 0.0722

```


