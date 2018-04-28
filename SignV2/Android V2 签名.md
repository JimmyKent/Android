### Android V2 签名

APK signature scheme v2
Android 7.0 引入一项新的应用签名方案 APK Signature Scheme v2，它能提供更快的应用安装时间和更多针对未授权 APK 文件更改的保护。在默认情况下，Android Studio 2.2 和 Android Plugin for Gradle 2.2 会使用 APK Signature Scheme v2 和传统签名方案来签署您的应用。

虽然我们建议您对您的应用采用 APK Signature Scheme v2，但这项新方案并非强制性的。如果您的应用在使用 APK Signature Scheme v2 时不能正确开发，您可以停用这项新方案。禁用过程会导致 Android Studio 2.2 和 Android Plugin for Gradle 2.2 仅使用传统签名方案来签署您的应用。要仅用传统方案签署，打开模块级 build.gradle 文件，然后将行 v2SigningEnabled false 添加到您的版本签名配置中：

  android {
    ...
    defaultConfig { ... }
    signingConfigs {
      release {
        storeFile file("myreleasekey.keystore")
        storePassword "password"
        keyAlias "MyReleaseKey"
        keyPassword "password"
        v2SigningEnabled false
      }
    }
  }
注意：如果您使用 APK Signature Scheme v2 签署您的应用，并对应用进行了进一步更改，则应用的签名将无效。出于这个原因，请在使用 APK Signature Scheme v2 签署您的应用之前、而非之后使用 zipalign 等工具。

如需了解详细信息，请阅读相关的 Android Studio 文档，这些文档介绍了如何在 Android Studio 中签署应用以及如何使用 Android Plugin for Gradle 为签署应用配置构建文件。

测试:  

1. app没有适配7.0， targetSdkVersion设置小于24，使用V1的签名方式进行签名打包，安装在5.1和7.0的手机上  
   结果：这种情况是没有问题的

2. app适配7.0，targetSdkVersion设置25， 使用V1的签名方式进行签名打包，安装在5.1和7.0的手机上

结果：同样没有问题

3. app没有适配7.0， targetSdkVersion设置小于24，单独使用V2的签名方式进行签名打包，安装在5.1和7.0的手机上

结果：安装在7.0上是没有问题的，但安装在5.1上就会出现安装失败，找不到签名证书，如图所示：
4. app适配7.0，targetSdkVersion设置25， 单独使用V2的签名方式进行签名打包，安装在5.1和7.0的手机上

结果：安装在7.0上是没有问题的，但安装在5.1上就会出现安装失败，找不到签名证书

结论：单独使用V2签名的apk是不能在小于7.0的手机上安装的，会出现签名证书找不到的情况，为了防止出现这种情况，AS使用了可以同时选择两种签名方式
即：7.0以下使用V1的签名方式，7.0以后的就使用V2的签名方式


渠道或者开发者关于渠道包的解决方案
目前主流的渠道号方案主要有以下几种：

修改后重新打包或签名的，例如在AndroidMainfest里面添加mata-data等
修改后不需要重新签名，主要有两种：

直接把apk包看成一个zip包，然后在zip包的注释段添加对应的渠道信息
直接把apk包看成一个zip包，然后利用相关命令在META-INF内注入${channel}.txt 文件
其中下面两种不需要重新签名的方法，被各主要渠道广泛使用。

渠道包与V2签名的冲突
然而，为了提高Android系统的安全性，Google从Android N增强了签名模式，该模式在原有的签名模式上，增加校验APK的SHA256哈希值；这种新引入的签名机制，会对整个文件的每个字节都会做校验，包括 comment 区域。如果签名后对APK作了任何修改，例如上面提到的修改注释段或者注入文件，在Android 7.0以上的机型安装时都会校验失败，提示没有签名无法安装。

try this:  
可以选择将文件修改为.zip后缀，然后解压缩，删除META-INF目录，然后再次压缩为.zip并修改扩展名为.apk后再次签名

v1签名常见问题
http://blog.bihe0832.com/android_signature.html

## V1
签名过程  验签过程


签名过程:  
  -> META-INF  
    -> MENIFEST.MF  
    -> CERT.SF  
    -> CERT.RSA  

MENIFEST.MF

```
Manifest-Version: 1.0
Built-By: Generated-by-ADT
Created-By: Android Gradle 2.3.3

Name: res/mipmap-xhdpi-v4/ic_launcher_round.png
SHA-256-Digest: fLzJrZnlwezh7uIdxQJk2Udc9vtLRQviNZpPXJrdF/s=
......
```

CERT.SF

```
Signature-Version: 1.0
X-Android-APK-Signed: 2
SHA-256-Digest-Manifest: 2yWO8ySKRieK1RFDUSQPcDkq1YC4NKJd7C7bD5wuykE=
Created-By: 1.0 (Android)

Name: res/mipmap-xhdpi-v4/ic_launcher_round.png
SHA-256-Digest: fLzJrZnlwezh7uIdxQJk2Udc9vtLRQviNZpPXJrdF/s=
......
```

CERT.RSA

```
3082 0301 0609 2a86 4886 f70d 0107 02a0
8202 f230 8202 ee02 0101 310f 300d 0609
6086 4801 6503 0402 0105 0030 0b06 092a
```

(1) .MF文件  
apk当中的原始文件信息用摘要算法如SHA1计算得到的摘要信息并用base64编码保存，以及对应采用的摘要算法如SHA1(这个算法的特性是不管多大的文件内容都能够得到长度相同的摘要信息但是不同的文件内容信息得到的摘要信息肯定不同)  
(2) .SF文件  
.MF文件的摘要信息以及.MF文件当中每个条目在用摘要算法计算得到的摘要信息并用base64编码保存  
(3) .RSA文件  
存放证书信息，公钥信息，以及用私钥对.SF文件的加密数据即签名信息，这段数据是无法伪造的，除非有私钥，另外.RSA文件还记录了所用的签名算法等信息。  
签名过程:
使用SHA1计算所有文件,对应每个文件生成Digest, 保存在(1)中的.MF文件,然后用私钥对.MF文件中的每个条目进行签名,生成(2)的.SF文件, 生成(3)公钥信息,签名算法等. 用zip压缩, 用zipalign对齐. 

验签过程:  
Apk包在安装的时候，解压缩, 验证签名.是按照从(3)到(1)的顺序依次校验的，先用公钥还原签名信息，然后和.SF文件中的信息比对，然后用同样的摘要算法对.MF文件里面的每一个条目计算对应的摘要信息，然后比对.MF文件是否一致。 

官方的V1验签过程  
https://source.android.com/security/apksigning/v2#v1-verification

## JAR 已签名的 APK 的验证（v1 方案）
JAR 已签名的 APK 是一种标准的已签名 JAR，其中包含的条目必须与 META-INF/MANIFEST.MF 中列出的条目完全相同，并且所有条目都必须已由同一组签名者签名。其完整性按照以下方式进行验证：

1. 每个签名者均由一个包含 META-INF/<signer>.SF 和 META-INF/<signer>.(RSA|DSA|EC) 的 JAR 条目来表示。
2. <signer>.(RSA|DSA|EC) 是具有 SignedData 结构的 PKCS #7 CMS ContentInfo，其签名通过 <signer>.SF 文件进行验证。
3. <signer>.SF 文件包含 META-INF/MANIFEST.MF 的全文件摘要和 META-INF/MANIFEST.MF 各个部分的摘要。需要验证 MANIFEST.MF 的全文件摘要。如果该验证失败，则改为验证 MANIFEST.MF 各个部分的摘要。
4. 对于每个受完整性保护的 JAR 条目，META-INF/MANIFEST.MF 都包含一个具有相应名称的部分，其中包含相应条目未压缩内容的摘要。所有这些摘要都需要验证。
5. 如果 APK 包含未在 MANIFEST.MF 中列出且不属于 JAR 签名一部分的 JAR 条目，APK 验证将会失败。
因此，保护链是每个受完整性保护的 JAR 条目的 <signer>.(RSA|DSA|EC) -> <signer>.SF -> MANIFEST.MF -> 内容。
 

在这个过程中，我们发现有两点：
(1) 在校验的过程中需要解压，因为.MF文件的摘要信息是基于原始未压缩文件内容，因此在校验的时候就需要解压出原始数据，而这个解压操作无疑是耗时操作。  
(2) apk包的完整性校验不够强。这里可以看到如果我们在apk签名后，如果对apk包中没有涉及到原始文件内容的数据块做改变那么这层校验机制就会失效（如直接通过二进制改变apk包的无关数据块如核心中央目录注释字段写一些无关注释，然后用zipalign工具对齐，则apk包还是可以正常安装的，这样就绕过了v1层的校验机制）  

<mark>以上可以看出原来apk就是一个zip包, 对齐过的zip包,然后改了后缀名, 仅此而已!  </mark>

分析zip文件可以知道:在META-INF里面添加文件, 或者在central directory里面添加字段, 都不会影响旧版本Android的系统验签.那么可以在上述两个地方添加渠道号.添加完之后需要再用zipalign对齐一下.  

由此v2模式的签名机制提出来了。看看是怎么改善上述两个问题的。
问题1：耗时问题
v2签名机制不存在解压原始数据，签名校验时间显著减少，因此安装时间也相应减少。
问题2：一致性校验是否够强
v2签名机制是直接基于apk的二进制内容做的签名信息（v2签名块本身不参与加密校验），因此打包后改变apk的原来三部分的任何字节都会导致签名校验不通过。

## V2

签名过程
在V1签名完成后, 在Contents of ZIPs entries 和Central Directory 之间添加 APK Signing Block(APK 签名分块).  

![apk-before-after-signing](apk-before-after-signing.png)

该分块包含多个“ID-值”对，所采用的封装方式有助于更轻松地在 APK 中找到该分块。APK 的 v2 签名会存储为一个“ID-值”对，其中 ID 为 0x7109871a。  

![apk-sections](apk-sections.png)

为了保护 APK 内容，APK 包含以下 4 个部分：

* ZIP 条目的内容（从偏移量 0 处开始一直到“APK 签名分块”的起始位置）
* APK 签名分块
* ZIP 中央目录
* ZIP 中央目录结尾

第 1、3 和 4 部分的摘要采用以下计算方式，类似于两级 Merkle 树。 每个部分都会被拆分成多个大小为 1 MB（220 个字节）的连续块。每个部分的最后一个块可能会短一些。每个块的摘要均通过字节 0xa5 的连接、块的长度（采用小端字节序的 uint32 值，以字节数计）和块的内容进行计算。顶级摘要通过字节 0x5a 的连接、块数（采用小端字节序的 uint32 值）以及块的摘要的连接（按照块在 APK 中显示的顺序）进行计算。摘要以分块方式计算，以便通过并行处理来加快计算速度。

![apk-integrity-protection](apk-integrity-protection.png)

攻击者可能会试图在支持对带 v2 签名的 APK 进行验证的 Android 平台上将带 v2 签名的 APK 作为带 v1 签名的 APK 进行验证。为了防范此类攻击，带 v2 签名的 APK 如果还带 v1 签名，其 META-INF/*.SF 文件的主要部分中必须包含 X-Android-APK-Signed 属性。该属性的值是一组以英文逗号分隔的 APK 签名方案 ID（v2 方案的 ID 为 2）。在验证 v1 签名时，对于此组中验证程序首选的 APK 签名方案（例如，v2 方案），如果 APK 没有相应的签名，APK 验证程序必须要拒绝这些 APK。此项保护依赖于内容 META-INF/*.SF 文件受 v1 签名保护这一事实。请参阅 JAR 已签名的 APK 的验证部分。

攻击者可能会试图从“APK 签名方案 v2 分块”中删除安全系数较高的签名。为了防范此类攻击，对 APK 进行签名时使用的签名算法 ID 的列表会存储在通过各个签名保护的 signed data 分块中。


验证
在 Android 7.0 中，可以根据 APK 签名方案 v2（v2 方案）或 JAR 签名（v1 方案）验证 APK。更低版本的平台会忽略 v2 签名，仅验证 v1 签名。

![apk-validation-process](apk-validation-process.png)
APK 签名验证过程（新步骤以红色显示）

APK 签名方案 v2 验证
找到“APK 签名分块”并验证以下内容：

1. “APK 签名分块”的两个大小字段包含相同的值。  
	a. “ZIP 中央目录结尾”紧跟在“ZIP 中央目录”记录后面。  
	b. “ZIP 中央目录结尾”之后没有任何数据。  
2. 找到“APK 签名分块”中的第一个“APK 签名方案 v2 分块”。如果 v2 分块存在，则继续执行第 3 步。否则，回退至使用 v1 方案验证 APK。  
3. 对“APK 签名方案 v2 分块”中的每个 signer 执行以下操作：  
	a. 从 signatures 中选择安全系数最高的受支持 signature algorithm ID。安全系数排序取决于各个实现/平台版本。  
	b. 使用 public key 并对照 signed data 验证 signatures 中对应的 signature。（现在可以安全地解析 signed data 了。）  
	c.验证 digests 和 signatures 中的签名算法 ID 列表（有序列表）是否相同。（这是为了防止删除/添加签名。）  
	d.使用签名算法所用的同一种摘要算法计算 APK 内容的摘要。  
	e.验证计算出的摘要是否与 digests 中对应的 digest 相同。  
	f.验证 certificates 中第一个 certificate 的 SubjectPublicKeyInfo 是否与 public key 相同。  
4. 如果找到了至少一个 signer，并且对于每个找到的 signer，第 3 步都取得了成功，APK 验证将会成功。  

注意：如果第 3 步或第 4 步失败了，则不得使用 v1 方案验证 APK。

# TODO 基于V2签名的多渠道打包, 脚本实现,java代码读取
原理
APK Signing Block部分
Android系统只会关注ID为0x7109871a的V2签名块，并且忽略其他的ID-Value，同时V2签名只会保护APK本身，不包含签名块。

因此，基于V2签名的多渠道打包方案就应运而生：在APK签名块中添加一个ID-Value，存储渠道信息。 
整个方案包括以下几步：

找到APK的EOCD块
找到APK签名块
获取已有的ID-Value Pair
添加包含渠道信息的ID-Value
基于所有的ID-Value生成新的签名块
修改EOCD的中央目录的偏移量（上面已介绍过：修改EOCD的中央目录偏移量，不会导致数据摘要校验失败）
用新的签名块替代旧的签名块，生成带有渠道信息的APK
实际上，除了渠道信息，我们可以在APK签名块中添加任何辅助信息。

APK 签名方案 v2  
https://source.android.com/security/apksigning/v2  
Android 7.0 开发者版本, APK signature scheme v2部分  
https://developer.android.google.cn/about/versions/nougat/android-7.0.html#apk_signature_v2  
zip文件分析  
https://blog.csdn.net/a200710716/article/details/51644421
分析Android V2新签名打包机制  
http://www.10tiao.com/html/223/201704/2651232457/1.html
Android新一代多渠道打包神器
https://www.zybuluo.com/ltlovezh/note/678755





