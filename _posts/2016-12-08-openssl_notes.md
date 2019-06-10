---
layout: post
title:  "The openssl notes"
date:   2016-12-08 15:47:53
categories: openssl
---

### openssl version 查看版本号

### openssl dgst 消息摘要

   消息摘要，所采用的命令是通过选项的形式实现的，消息摘要的输入可以是stdin,或者文件。
   
   比如：openssl dgst -sha1 test.txt
   
   当然，也可以直接用命令，省掉dgst。例如：openssl sha1 test.txt

   openssl dgst 除了可以计算消息摘要，还可以对摘要进行签名，验证签名。例如：
   
   1  签名：openssl dgst -dss1 -sign dsakey.pem -out dsasign.bin file.txt
   
   通过dss1计算文件file.txt的消息摘要，然后用私钥dsakey.pem对消息摘要进行签名，保存在dsasign.bin
   
   2  验证签名：openssl dgst -dss1 -prverify dsakey.pem -signature dsasign.bin file.txt
   
   用私钥dsakey.pem 对签名dsasign.bin进行验证
   
### openssl enc 对称加密

   每一个对称加密可以直接使用，省掉enc. 如果加上enc，那么各个命令是以选项的形式出现。
   
   1) key是用来加密、解密数据
   
   2) pasword/passphrase(口令) 是用来生成key和初始化向量。生成Key的时候，是通过password 和 salt(8位随机数)
      
	  共同作为输入的。
	  
   例如：
   
   1) 加密 openssl des3 -salt -in test.c -out ciphertest.bin
   
      用des3加密test.c 密文保存在ciphertest.bin
	  
   2) 解密 openssl des3 -d -in ciphertest.bin -out test1.txt
   
      用des3解密ciphertest.bin 密文保存在test1.txt
   
### openssl dhparam Diffie-Hellman命令

### rsa

#### openssl genrsa 生成rsa私钥

openssl genrsa -out rsaprivatekey.pem -passout pass:test -des3 1024

生成一个私钥1024位的私钥，保存在 rsaprivatekey.pem。私钥生成输出口令是test，私钥加密算法是des3

#### openssl rsa 通过私钥生成公钥

openssl rsa -in rsaprivatekey.pem -passin pass:test -pubout -out rsapublickey.pem

和5.1相反的过程

#### openssl rsautl 用秘钥对进行加密、解密，签名、验证签名

这个一般是加密比较小的数据，比如秘钥，太大的数据要用对称加密，下面的加密就失败了.

openssl rsautl -encrypt -pubin -inkey rsapublickey.pem -in test.c -out testrsacipher.txt

result:

RSA operation error

139836004808352:error:0406D06E:rsa routines:RSA_padding_add_PKCS1_type_2:data too large for key size:rsa_pk1.c:151:

### openssl req 生成CSR

openssl req -new -x509 -days 365 -key trans.key -out trans.crt

生成一个自签名的证书

### openssl bio的使用

https://stackoverflow.com/questions/38516584/what-is-the-difference-between-bio-read-bio-write-and-ssl-read-ssl-write-when-th






