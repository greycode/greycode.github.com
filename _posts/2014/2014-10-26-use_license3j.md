---
layout: post
title: 使用License3j实现简单的License验证
categories:
- Java开发
tags:
- [java,License3j,license,pgp,密钥,非对称加密]
---
<link rel="stylesheet" href="/media/highlight/styles/github.css">
<script src="/media/highlight/highlight.pack.js"></script>
<script>
$(document).ready(function() {
  $('pre code').each(function(i, block) {
    hljs.highlightBlock(block);
  });
});
</script>

在项目中可能会遇到要提供License的情况，虽然商业的解决方案有很多而且足够强大，但有时候我们认为不需要投入太多只是希望借License机制提供基本的限制或提醒功能，使用简单开源的方案就可以。

License3j是一个免费开源可用于商业项目的License库，你可以借助它在项目中实现基本的License验证功能。
我们可能会提供如下的License文件
<pre><code class="text">
user=greycode
company=Grey`s Code
email=anywalker@163.com
edition=enterprise
release-version=1.0.0
valid-until=2015.12.31
</code></pre>
如果把上面的文件直接在项目中使用Java来读取验证的话很容易被客户篡改，最好是提供如下不可读的方式
<pre><code class="text">
-----BEGIN PGP MESSAGE-----
Version: BCPG v1.50

owJ4nJvAy8zAxbgv/llYiaPva8bTB/iSxHMyk1PzilPdMnNS/RJzU3U90/Pyi1JT
QnymNynr6ipApRXSgPK8XMrBpXkK/sklCkZmCoYmViaGVgYmCs7BIQpGBoYmvFxl
iTmZKbqleSWZObZAEVM9QyM9Y0NeruT83ILEvEpb96LUyoRiBef8FKBZpcWpRbbp
QJFkMDc1NxGoC6iqPDEnO7XIwdDMWA+oj5erKDUnNbE4Vbcstag4Mz/P1lDPQM8A
qD4lswTETc0rSS0qKMosBprRyajKwsDIxcDPygRyP4dMXmppcX5aCQMXpwDM12Z/
2P8ZRuxn2CvensAgZF2VrHMwlfmKWdPT8FLlKoPvDtNYCpbomHK7cTmGFJxdLrLW
e+kR7qzDR3bl+UoafWg/Vrxzsvm7l1knL7Nw3Lxq+pHj+iTxgKCMxZlPhU/yykhJ
c1g0Pfnbphv9z2UW93yts739z4JyE/3EK9hT3tTreVvzvV00szh94vPYyJgGtZqS
bWE/zGoOxb55Urtenfkh+56oqo40zn1Xbz16ViHw6xdj94fgTb1fWk0/Fuy5/CBH
KOXMvJ+OWSE823k371v+6VHU5WIht9RlOsphq33us4p6PEvmzb4usJS9Z8uRdTml
8Zcyjp7vTLQQkRVc4G+7/P6rF9d2z9q+8jFb9O4UVgA6F865
=WLRy
-----END PGP MESSAGE-----

</code></pre>

上面的文字经过加密后已经不可能被人为任意篡改了，这样在Java程序中使用某个专用于解密的文件解密并读取出相应的配置然后验证该Lisence的有效性即可，而加密解密部分的功能，也正是License3j提供的。

如果你了解非对称加密/解密，可能已经知道这其中的原理。我们使用一对私钥公钥，私钥用于加密License原文，然后我们自己保留私钥，将公钥和加密后的内容发布，而公钥用于解密，加密后的内容只用于读取不可修改，这也算是很简单的一种实现了。

------
## 如何使用
License3j使用的是PGP密钥，但本身不提供密钥对生成功能，你可以借助其他工具，如果你安装过Git，那么Git的安装目录下已经有一个可以直接使用的GPG程序，可以执行如下来生成一对密钥：
<pre><code class="text">
D:\Program Files (x86)\Git\bin>gpg --gen-key
gpg (GnuPG) 1.4.9; Copyright (C) 2008 Free Software Foundation, Inc.
  This is free software: you are free to change and redistribute it.
  There is NO WARRANTY, to the extent permitted by law.

  Please select what kind of key you want:
     (1) DSA and Elgamal (default)
     (2) DSA (sign only)
     (5) RSA (sign only)
  Your selection? 5
</code></pre>
选择加密的算法，选哪个都可以，具体的算法细节区别请自行搜索，需要注意的是RSA算法选项是5 不是 3！！。
<pre><code class="text">
  RSA keys may be between 1024 and 4096 bits long.
  What keysize do you want? (2048) 4096
</code></pre>
选择key的大小，在1024到4096字节之间，默认2048。
<pre><code class="text">
  Requested keysize is 4096 bits
  Please specify how long the key should be valid.
           0 = key does not expire
        <n>  = key expires in n days
        <n>w = key expires in n weeks
        <n>m = key expires in n months
        <n>y = key expires in n years
  Key is valid for? (0) 0
  Key does not expire at all
  Is this correct? (y/N) Y
</code></pre>
选择key过期的时间，0就是永不过期，1m就是1个月后密钥过期。
<pre><code class="text">
  You need a user ID to identify your key; the software constructs the user ID
  from the Real Name, Comment and Email Address in this form:
      "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

  Real name: greycode
  Email address:
  Comment:
  You selected this USER-ID:
      "greycode"

  Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit?
</code></pre>
接下来输入用户名等信息，这里除名字外可以跳过，注意记住上面的`USER-ID`，这个之后会用到。
<pre><code class="text">
  You need a Passphrase to protect your secret key.
  Enter passphrase:
</code></pre>
这里输入密码，然后会提示正在生产密钥，这个时候随便动动鼠标。
<pre><code class="text">
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
.+++++
....+++++
gpg: key C06EB015 marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   4  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 4u
pub   2048R/C06EB015 2014-10-26
      Key fingerprint = 51AF 8A9B B990 F0EC 7DD7  0DED 1117 E6E2 C06E B01
uid                  greycode

Note that this key cannot be used for encryption.  You may want to use
the command "--edit-key" to generate a subkey for this purpose.

D:\Program Files (x86)\Git\bin>
</code></pre>
这样我们的密钥对就生成好了，如果你不知道密钥对文件的位置，可以命令行中输入`gpg -h`:
<pre><code class="text">
...
Home: C:/Users/walker/AppData/Roaming/gnupg
Supported algorithms:
Pubkey: RSA, RSA-E, RSA-S, ELG-E, DSA
Cipher: 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH
Hash: MD5, SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB
...
</code></pre>
看到Home那里了吧，在上面的目录中找到`secring.gpg`与`pubring.gpg`文件。
如果觉得上面的gpg命令行太麻烦，可以使用GUI的工具[Gpg4win]( http://www.gpg4win.org/download.html)（怎么一开始不说啊，害我跟到现在！！）。
接下来使用`secring.gpg`进行加密。`pom.xml`中加入：
<pre><code class="xml">
 &lt;dependency>
	 &lt;groupIdd&gt;com.verhas &lt;/groupIdd&gt;
	 &lt;artifactIdd&gt;license3j &lt;/artifactIdd&gt;
	 &lt;versiond&gt;1.0.5 &lt;/versiond&gt;
 &lt;/dependencyd&gt;
</code></pre>
这里使用1.0.5，可以去Maven Repository中查找最新版本。
Java代码如下：
<pre><code class="java">
package org.greycode.demo;

import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStream;

import com.verhas.licensor.License;

public class LicenseDemo {

	public static void main(String[] args) {
		try {
			File licenseFile = new File("demo.license");
			
			if (!licenseFile.exists()) {
			// license 文件生成
				OutputStream os = new FileOutputStream("demo.license");
				os.write(new License()
				    // license 的原文
				    .setLicense(new File("license-plain.txt"))
				    // 私钥与之前生成密钥时产生的USER-ID
				    .loadKey("secring.gpg","keyUserId")
				    // 生成密钥时输入的密码
				    .encodeLicense("keyPassword").getBytes("utf-8"));
				os.close();
			} else {
			// licence 文件验证
			    License license = new License();

			    if (license
				    .loadKeyRing("pubring.gpg", null)
				    .setLicenseEncodedFromFile("demo.license").isVerified()) {
			    	System.out.println(license.getFeature("edition"));
				System.out.println(license.getFeature("valid-until"));
			    }
			
			}
			
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}
</code></pre>
如上通过在Java代码中获取licence的过期日期、版本号等属性进行验证，或者采取其他手段如自动连接验证服务器、启动定时任务等等。

