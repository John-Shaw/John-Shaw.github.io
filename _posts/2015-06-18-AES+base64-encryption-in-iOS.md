---

layout: post
title: iOS中的AES+base64加密方案
categories:
- 格物
tags:
- iOS
- Encryption
- PassCapsule
disqus: y

---

###前言
---------------------------
最近在忙一个密码管理的 iOS APP([PassCapsule](https://github.com/John-Shaw/PassCapsule))。本来这是实验室老师要我们参加比赛（信息安全竞赛），我临时想的一个项目，没准备好好做，但想想我这三年虽然代码写的不少，但几乎没有严肃的写一个面向用户的正式程序，再加上自己马上要去找实习，项目经验少，博客又几乎不写，实在愧对码农这种荣耀的勤劳的无产阶级劳动者的称号。所以今天开始准备写博客记录，记录这个项目的点点滴滴。同时 More Objective Objective-C 的坑，还是等我功力深厚些再补。（是跳票吗？是跳票吧？是跳票？(╯°□°）╯︵ ┻━┻

1. 拒绝重复造轮子
--------------------------
iOS上本身是有AES等对称加密算法的(`CommonCrypto/CommonCryptor.h`)，而且使用也并不复杂，大多数情况下够用了。但是说实话OO程序员都不太喜欢 C，而且也并不优雅。偶然看到了 [RNCryptor](https://github.com/RNCryptor/RNCryptor)，一个对AES加解密的Objective-C封装，也支持其他语言如 C++,C#,Java,PHP,Python,JavaScript 和 Ruby。我看了一下源代码，发现他其实就是封装了 CommonCrypto，其默认的加密算法相关设置如下(摘自`RNCryptor.h`)

```objc
static const RNCryptorSettings kRNCryptorAES256Settings = {
    .algorithm = kCCAlgorithmAES128,
    .blockSize = kCCBlockSizeAES128,
    .IVSize = kCCBlockSizeAES128,
    .options = kCCOptionPKCS7Padding,
    .HMACAlgorithm = kCCHmacAlgSHA256,
    .HMACLength = CC_SHA256_DIGEST_LENGTH,

    .keySettings = {
        .keySize = kCCKeySizeAES256,
        .saltSize = 8,
        .PBKDFAlgorithm = kCCPBKDF2,
        .PRF = kCCPRFHmacAlgSHA1,
        .rounds = 10000
    },

    .HMACKeySettings = {
        .keySize = kCCKeySizeAES256,
        .saltSize = 8,
        .PBKDFAlgorithm = kCCPBKDF2,
        .PRF = kCCPRFHmacAlgSHA1,
        .rounds = 10000
    }
};
```
我对加密算法和信息安全领域不没有很深的研究，但通过资料查阅也大致了解了这些参数的意义。RNCryptor 默认使用 AES 256位加密算法，ECB 模式，PKCS7Padding 填充算法。当然你也可以使用其他参数设置（RNCryptor 的文档我并没有细看），例如使用 CBC 模式进行加密。

当然我们暂时不考虑这么多，先来看看怎么用。RNCryptor 的使用非常简单：

```objc
   NSString *plainText = @"I feel luck.";
   NSString *keyString = @"I am a key."
   NSData *data = [plainText dataUsingEncoding:NSUTF8StringEncoding];
   
   NSError *error;
   NSData *encryptedData = [RNEncryptor encryptData:data
                                       withSettings:kRNCryptorAES256Settings
                                           password: keyString
                                              error:&error];
```
是的，我上面blahblah一堆话，但其实几步就可以满足我们的需求。


2. 将加密后的NSData用base64编码
-------------------------
加密好的`encryptedData `是二进制的`NSData`，不方便存储和网络传输，考虑到我的`PassCapsule`是使用XML进行存储，并且有网络传输的需要，所以我们将加密好的数据进行 base64 编码。base64 算法非常简单，并且 iOS SDK 中已集成，简单使用如下：

```objc
	NSData *encryptedData = blahblah; 
	NSString *base64String = [encryptedData base64EncodedStringWithOptions:0];
```


3. 解密数据
-------------------------
一般用户注册账号的密码，服务端只需存储加密后的数据，在对比验证即可，不需要知道密码明文。`PassCapsule` 作为密码管理软件，在加密用户各种密码的同时也必须要能解密，将密码明文提供给用户。所以我们同样需要解密，在使用 `RNCryptor` 后解密也相当简单。

```objc
	//先从base64的字符串中反编码出二进制加密数据
	NSString *base64String = blahblah; 
	NSData *encryptedData = [[NSData alloc] initWithBase64EncodedString: base64String options:NSDataBase64DecodingIgnoreUnknownCharacters];
	
	//AES 解密数据
	NSString *keyString = blahblah;
	NSData *encryptedData = blahblah; 
    NSData *decryptData = [RNDecryptor decryptData:encryptedData
                                 	  withPassword:keyString
                                        	 error:nil];
                                        	 
    NSString * decryptString = [[NSString alloc] initWithData:decryptData encoding:NSUTF8StringEncoding];
```
我们的keyString，即加密密匙要小心保存，最好是用随机字符串或加入`salt`哈希，然后用 iOS 的 KeyChain 保存。当然 Keychain 也不是绝对安全了，越狱后就可以导出其中内容。但是一般来说，黑客通过软件手段是几乎无法攻克 Keychain 的。

另外一种更安全的解决方案时，每次用户输入主密码后，将其加密放入 Keychain，用这段数据作为所有其他数据的加密密匙，在程序退出 (`- (void)applicationWillTerminate:(UIApplication *)application`) 前再将 Keychain 清空。这只是我暂时的设想，有待以后在实战中验证。


小结
-----------------------------------
终于开始正式写博客了，虽然今天只是简单介绍了iOS下的 AES + base64 加密方案，而且由于使用了 `RNCryptor`，实际代码量很少。但目测还是会遇到很多坑，以后会详细记载更多的解决方案。

