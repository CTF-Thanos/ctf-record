

##题目描述：
---
[http://www.shiyanbar.com/ctf/1848](http://www.shiyanbar.com/ctf/1848)


##前言
---
作为新人，本人之前是不知道**[哈希长度扩展攻击](https://www.wikiwand.com/en/Length_extension_attack)**的，当做到这道题的时候Google了一下发现这个概念，即使看了维基百科的描述也发现没有什么过多的资料解释这种攻击方式成功的缘由。但是强烈的好奇心驱使我继续Google下去，就像发现了新大陆一样一直往前走，一口气看了这篇文章。

+ [Everything you need to know about hash length extension attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks)

经过仔细的阅读我发现我看不懂这里面专业的英文单词(CET4都没过的。。),于是我想先从简单的开始，直接用MD5算法当样例，一步一步的看下去，RFC看起来我会跪的，于是我Google了一篇中文的:

+ [MD5算法原理及实现](http://noalgo.info/600.html)

跟着算法一遍看一遍想。大约花了2～3小时我才成功阅读和理解到文章中攻击手段形成的缘由。
然后我要确保我自己确实没有理解错，我继续Google了一下文章,找到了这些资源：

+ [http://netifera.com/research/flickr_api_signature_forgery.pdf](http://netifera.com/research/flickr_api_signature_forgery.pdf)
+ [科普哈希长度扩展攻击](http://www.freebuf.com/articles/web/31756.html)
+ [深入理解hash长度扩展攻击（sha1为例)](http://www.freebuf.com/articles/web/69264.html)



##正文
---

###基础
简单的讲，**哈希长度扩展攻击** 产生的原因来自于这些hash函数要分块去做多次计算，上一次的计算会作为下一次计算的输入。而分块必须得均分，所以涉及到当原始消息长度不够要补全(`padding`)的事情。因为绝大多数的消息都不可能把长度恰好弄成hash算法要求的整数倍数长度。

padding的时候，先填充一个`1`,然后后面给无数个`0`,用16进制来说，就是先给一个`0x80`,后面有无数个`0x00`.不同的hash算法有不同的块大小，而padding的0的多少需要这些hash算法决定。
举例来说：MD5算法要求:加密的原始字符串要使其位长对512(64字节)求余的果等于448(56字节)。因此，信息的位长将被扩展至`N*512+448`，N为一个非负整数，N可以是零。为什么要用448呢？因为MD5算法(SHA1也一样，我不确认是否所有hash算法都是如此)要求最后的8字节(64bit)要保存填充前信息长度。这样可以保证信息的位长=`N*512+448+64=(N+1)*512`，即长度恰好是512的整数倍.

数学定义就是:
> length(raw_msg)%64 == 56


以上文提到的[Everything you need to know about hash length extension attacks](https://blog.skullsecurity.org/2012/everything-you-need-to-know-about-hash-length-extension-attacks)提到的例子来讲：

原始信息长度为`length("secretdata") = 10`;所以我们还需要`46`字节padding的1和0，然后加上8字节的字符串长度`10字节＝>80bit=>0x50`. 最后结构就是文中提到的这个样子:


  ```
    0000  73 65 63 72 65 74 64 61 74 61 80 00 00 00 00 00  secretdata......
  	0010  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
  	0020  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
  	0030  00 00 00 00 00 00 00 00 50 00 00 00 00 00 00 00  ........P.......
  ```
  
###如何攻击呢？

由于上面我们已经补全过数据了，现在已经是完整的块了，当我们继续在后面补充数据的时候，他在分块的时候就会直接帮我们把前面的数据分块，这部分会被hash算法先计算，由于算法初始值相同，计算的顺序相同，这一块的数据计算的结果就一定是固定的，也就是在我们没有补充数据之前的数据的hash结果，那么只要当我们后面补充的数据用这个hash结果的状态继续往后面执行，计算出来的evil_hash也一定会和服务器端hash(secret+“填充数据”+“任意可控数据”)的最终结果一样。从而可以导致该攻击。

更简单的说法是：攻击者的哈希计算过程，相当于从服务器计算过程的一半紧接着进行下去。

攻击流程：

+ 服务器端：
   * hash(secret+msg) =>一系列计算 hash_value
+ 攻击者:
   * 在知道secret+msg多长的情况下可以补全为一个完整的块，从而补全的这个块计算出来的结果我们可以拿到该状态，然后加入可控的邪恶数据，再利用原来服务器的hash_value来设置当前的hash状态，从而继续执行下去。
   
这个过程中，攻击者完全不用知道secret是什么，他只用知道长度，就可以预测出你的下一个hash结果。因为他知道你的计算状态。。。


###额外需要知道的事情

+  通常来讲，攻击者不会知道服务器上原始secret的长度，所以可能需要枚举secret的长度直到成功
+  通常，这类工具存在于URL GET/POST的请求参数中。所以你需要知道还有一种攻击方式是：HTTP参数污染（HPP)
	* HTTP中当参数出现同名的时候，比如`index.php?a=1&a=2`;大多数语言都是后面的覆盖前面的。所以利用hash长度攻击的时候，直接在参数后面加入同名参数以达到构造恶意内容是可能的。
	* 题外话，PHP在处理`Cookie`的时候，同名时是取的前面的那个。但是Python的`Tornado`是取的后面的,参考:[知乎某处XSS+刷粉超详细漏洞技术分析](https://www.leavesongs.com/HTML/zhihu-xss-worm.html)。

###攻击工具

+ [https://github.com/bwall/HashPump](https://github.com/bwall/HashPump)
+ [https://github.com/iagox86/hash_extender](https://github.com/iagox86/hash_extender)


##本题答案
---

1. 修改cookie的source可以看见源代码。
2. hashpump -s 571580b26c65f306376d4f64e53cb5c7 -d adminadmin -a shellvon -k 15
3. 提交即可。



