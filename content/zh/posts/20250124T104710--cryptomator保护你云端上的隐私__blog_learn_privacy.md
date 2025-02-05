---
title:      "Cryptomator保护你云端上的隐私"
date:       2025-01-24T10:47:10+08:00
tags:       ["blog", "learn", "privacy"]
identifier: "20250124T104710"
---

网盘为我们提供了随时随地的获取数据的便利性，同时也大大减轻了你我手机电脑空间不足的压力。但是一旦我们选择使用网盘，也意味着把你的私密数据交出去了。

对于公共资料来说，无非就是提供网盘服务的公司知道了你做了保存这些资料的行为而已。然而对于你私有的数据如照片，视频，笔记，备忘录，一旦你上传了，就等于你自己把所有权交出去了，你永远不知道别人怎么去使用你的数据，即便这些大公司号称会保护你的隐私。（搞不好这些数据就被LLM拿去当训练语料了。）

那是不是我们就得停止使用网盘了呢？那倒不至于因噎废食，只需要牺牲一点便利性，拿起密码学的武器来保护自己就好了。对于我们收集的公共资料来说，完全没有必要加密。保证文件的哈希值不变，反而能帮助我们快速上传（实际并没有真正的上传资料，只是增加了个标识即可）。而对于我们隐私部分的数据，那就需要进行加密了。确保密钥只有自己拥有，那么就没人能知道你放在网盘上是什么东西了。

## 为什么选择 Cryptomator

那有什么好的加密工具呢？我这里选择了使用[Cryptomator](https://cryptomator.org/)， 主要的一个原因是它是一款开源软件，跟我选择[Bitwarden]()的理由是一样的。在Cryptomator的官网里有专门一篇文章[1]来讲述为什么开源能保证提高安全性。

里面用了一个很形象的例子来描述开源的安全性：想象一下你在一个陌生的国家里，晚上到了小镇阴暗角落的一家酒吧，以下情形哪一种听起来更安全：
* 酒保当着你的面调出来的酒
* 陌生人给你提供的一杯不知道怎么来的酒

![](https://cryptomator.org/img/open-source/bartender-vs-stranger@2x.png)

## 如何使用

Cryptomator 提供了详细的[doc](https://docs.cryptomator.org/en/latest/desktop/getting-started/)来介绍如何使用这个工具。
1. 创建Vault来存放你的私密文件。
2. 每个Vault都需要提供一个密码，Cryptomator会对你的Vault进行加密。
3. 在打开Vault的时候，Cryptomator会创建出Vault对应的一个解密后的文件夹。
4. 我们可以在解密后的文件夹里面正常存放私密数据，而在Vault里面，永远都是加密后的数据。

而我们同步到网盘的数据，都是Vault里面已经被加密后的数据。

## 原理

想了解背后的原理，可以看看文档的[Security](https://docs.cryptomator.org/en/latest/security/security-target/)一节。代码的话主要可以看[cryptolib](https://github.com/cryptomator/cryptolib), 里面涉及到了关键的加密解密内容。

### 安全架构

**虚拟文件系统**

当我们解锁Vault时，Cryptomator会创建一个对应的文件夹来存放解密文件。不同的系统会使用不同的前端。如果找不到对应的虚拟文件系统，则会fallback到基于HTTP协议的WebDAV。

不同系统的默认虚拟文件系统为：WinFsp (on Windows) and macFUSE (on macOS) and FUSE (Linux)

**Vault结构**

每个Vault都会有一个`vault.cryptomator`文件，文件内容是一个JWT（json web token），里面包含了一些vault的基本信息，以及要使用哪个masterkey

打开一个Vault的流程如下

1. 解码`vault.cryptomator`文件，不对签名进行验证（原因是这时候我们还没有私钥）
2. 获取kid, 基于kid获取`masterkey`
3. 用拿到的masterkey进行签名验证
4. 确保format和cipherCombo是支持的

**Masterkey**

每个Vault有一个256位的密钥和MAC masterkey， 这两个key都是通过CSPRNG(Cryptographically Secure Pseudo-Random Number Generator)生成的.

这两个key都被加密之后存放起来，可以有以下两种方式进行使用

* Cryptomator Hub
* Masterkey File

使用[AES的加密方式](https://datatracker.ietf.org/doc/html/rfc3394)，根据用户的口令和随机生成的盐进行加密，并保存在`masterkey.cryptomator`文件里

加密密钥

![encrypted](https://docs.cryptomator.org/en/latest/_images/key-derivation%402x.png)

解密密钥

![decrpted](https://docs.cryptomator.org/en/latest/_images/masterkey-decryption%402x.png)

### Vault密码学

**文件头加密**

file header保存了特定的元数据，主要是nonce和content key，后续会用于对文件内容的加密。每个文件都会生成对应的一个file header

总共有68 Bytes：

* 12 bytes的nonce
* 40 bytes 用于AES-GCM
  * 8 bytes 留待后用
  * 32 bytes AES密钥.
* 16 bytes tag of the encrypted payload.

```
headerNonce := createRandomBytes(12)
contentKey := createRandomBytes(32)
cleartextPayload := 0xFFFFFFFFFFFFFFFF . contentKey
ciphertextPayload, tag := aesGcm(cleartextPayload, encryptionMasterKey, headerNonce)
```

加密文件头

![AES-GCM 加密](https://docs.cryptomator.org/en/latest/_images/file-header-encryption%402x.png)


**文件内容加密**

明文被split成chunks, 每个chunk都有自己的nonce（看了下AES-GCM，提到IV重复会导致安全漏洞，这可能就是为什么每个chunk都要有自己的nonce的原因）

```
cleartextChunks[] := split(cleartext, 32KiB)
for (int i = 0; i < length(cleartextChunks); i++) {
    chunkNonce := createRandomBytes(12)
    aad := [bigEndian(i), headerNonce]
    ciphertextPayload, tag := aesGcm(cleartextChunks[i], contentKey, chunkNonce, aad)
    ciphertextChunks[i] := chunkNonce . ciphertextPayload . tag
}
ciphertextFileContent := join(ciphertextChunks[])
```

加密文件

![加密文件](https://docs.cryptomator.org/en/latest/_images/file-content-encryption%402x.png)

**目录id**

[这里介绍目录如何被加密](https://docs.cryptomator.org/en/latest/security/vault/#directory-ids)，其中用sha1哈希值拆成路径，所以最终所有的目录都拥有一样的层级结构

```
dirIdHash := base32(sha1(aesSiv(dirId, null, encryptionMasterKey, macMasterKey)))
dirPath := vaultRoot + '/d/' + substr(dirIdHash, 0, 2) + '/' + substr(dirIdHash, 2, 30)
```

**文件名加密**

文件名先用utf-8进行编码成[Normalization Form C](https://unicode.org/reports/tr15/#Norm*Forms)以获得唯一二进制表示

```
ciphertextName := base64url(aesSiv(cleartextName, parentDirId, encryptionMasterKey, macMasterKey)) + '.c9r'
```

加密文件名

![加密文件名](https://docs.cryptomator.org/en/latest/_images/filename-encryption%402x.png)


## 风险

由于我们对文件进行了加密，而且密钥并没有备份到线上，一旦出现密钥相关文件损坏，则意味着数据的丢失。网上有人提到百度网盘会删除相关的密钥文件导致文件无法解密，虽然不确定真伪，但是还是小心使得万年船，我们还是要做好最坏的打算。所以可以从以下方面进行着手。

* 预防网盘上的文件损坏：对此，我们的一种防范机制就是把Vault同步到多个网盘，这样即使某一个网盘上的文件损坏，我们在别处还是可以找到对应的完整文件
* 预防本地文件损坏：对于本地文件损坏，则所有网盘文件都同步到了损坏的文件。为了应对这种情况的出现，那么还是需要我们对本地文件也要进行备份。这时候我们有两个选择，一个是备份加密后的文件，一个是备份原始文件。

想了解大家平时都遇到什么样的问题，可以去[讨论区](https://community.cryptomator.org/)逛逛。看完大家的问题你就知道备份是多么重要了。

## Reference

1. [How You Can Further Increase Your Data Security Through Open Source](https://cryptomator.org/open-source/)
2. [Vault Cryptography](https://docs.cryptomator.org/en/latest/security/vault/#)
