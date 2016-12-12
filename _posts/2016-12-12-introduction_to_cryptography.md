---
layout: post
title:  "Introduction to cryptography"
date:   2016-12-12 15:47:53
categories: cryptography
---

### cryptography & encription

cryptography 和 encription都是指加密，密码，但是这两个概念使用起来有点差异。

cryptography是“研究秘密写的科学”，是指安全系统，密码系统，概念的范围更大。

encription是加密，用算法加密，是指一个动作。往往是指“将明文，通过某种算法以后，得到密文”

### 密码系统关注的三个核心问题

1. 机密性(confidentiality)

   保证数据或者信息的安全，所以需要对机密信息进行加密，形成密文(ciphertext)以后再保存或者传输，及时被泄露，窃取了。
   
   这样，及时被泄露，窃取了，窃听者也得不到明文(plaintext)，或者在短时间内得不到明文。
   
2. 身份验证(identity)

   在涉及机密信息传输的过程中，通信一方必须要能确定和他通信的另一方，就是他想要的一方，而不是一个伪装人。
   
3. 安全传输、保证完整性(integrity)

   数据传输的过程中，接收一方得到的信息有可能被篡改，所以必须要有办法验证消息是否被篡改。
   
### 加密算法

#### 对称加密(Symmetric key cryptography)

   对称加密是指加密、解密秘钥相同。常用的对称加密方法有 1）流式加密，对明文的每一个字节和秘钥进行运算，得到密文。
   2) 块式加密，对明文的某几个字节作为一个单位，和秘钥进行运算，得到密文。
   
   常用的对称加密算法：DES,3DES,AES
   
#### 公钥加密(Public key cryptography)或非对称加密(asymmetric cryptography)

   加密的秘钥(一般称作公钥)和解密的秘钥(一般称作私钥)不相同。公钥往往是对外公开的，大家都可以拿到；私钥是不公开的，需要秘密保存的。
   
   如果想给某人发消息，用其公钥加密原文，只有和此公钥配对的私钥才能解密此消息。
   
   由于公钥加密对原文的长度有限制，所以公钥算法一般用做秘钥(key-exchange)交换和数字签名(Digital signatures)。
   
   常用的公钥加密算法：RSA, Diffie-Hellman
   
### Public-key Infrastructure

   有了公钥加密算法，在不安全的通信环境下，可以安全的交换秘钥了。然而，通信两端在没有见面的情况下，怎么确认通信的另一方是靠谱的呢？
   Internet PKI就是通过CAs(Certificate Authorities)来解决这个问题的。

<pre>
   +----------+  Request certificate issuance(CSR)  +-----+          CSR          +----+
   |Subcriber |------------------------------------>| RA  |---------------------->| CA |
   |          |<------------------------------------|     |<----------------------|    |
   +----------+     Certificate signed              +-----+    Certificate signed +----+
        |                                      Validate subcriber's                  |
		|		                                      identity                       |  
		|Publish											                         |
        |certificate                  +---------------------------------+------------+
		|                             |                                 |
		V                             V                                 V
   +------------+              +------------+                  +----------------+
   | Web Server |              | CRL Server |                  | OCSP Responder |
   +------------+              +------------+                  +----------------+
          ^ |                      ^                                    ^
	1.Req | |2. Verify             |                                    |
	Cert  | | Signature            |                                    |
	      | V                      |                                    |
   +---------------+               |                                    |
   | Relying Party |---------------+------------------------------------+
   +---------------+ 3. Check for revocation
</pre>

#### Subcriber

   准备提供安全服务的个人或者组织，往往需要向CA申请一个证书(certificate).

#### Registration authority
   
   本地注册机构，对申请证书的用户，就行资料审核，确认等。是CA多分支组织架构中的一个分支结构。
 
#### Certification authority

   证书颁发机构，是值得大家信任的一个组织。提供证书发行，管理，吊销等服务。目前的CA有很多。
 
#### Relying party

   证书消费者，比如浏览器，操作系统，以及一些其他终端软件。

#### X.509

   x.509是PKI的国际标准。PKI x.509工作组叫PKIX, 目前发布的主要标准文档是RFC 5280
   
#### Certificate

   证书就是一个数字文档，包含了证书申请人的公钥，相关身份信息，已经证书签发机构的数字签名。
   
#### Certificate Chains

   证书的签发只是经过一级签发机构，在安全上，管理上都存在很多问题。所以，证书一般会经过多级证书签发。
   
   及，终端用户的证书是经过中间很多CA签发的，这样就形成了一个证书链。
   
   形成证书链的原因：
   
   1. 保证根证书的绝对安全
   
   如果用根证书直接签发用户证书，一旦根证书秘钥泄露，那么所有被签发的证书都有了安全问题。这样吊销证书，重新发布都会遇到困难。
   
   2. 交叉认证(Cross-certification)

   重新发布CA，如果重新创建自己的Root key，是比较困难的。往往是通过知名的CA来签发自己的CA。
   
   3. 多分支机构(Compartmentalization)
   
   一个CA可以在不同地方设立多个下级CA，这样可以方便、高效的展开工作。
   
   4. 委派
   
   一个CA可以委派其他机构进行正式发布。
   
 #### 吊销(Revocation)
 
   