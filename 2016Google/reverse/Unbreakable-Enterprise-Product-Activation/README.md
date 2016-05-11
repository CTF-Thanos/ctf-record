##  Unbreakable Enterprise Product Activation Record

[原题文件](./unbreakable-enterprise-product-activation.bz2)

---

### 0x01 分析
解压之后, 发现还是个 cpio 格式的压缩文件, 我直接后缀改成.tar, 再解压便得到可执行文件. 扔到 IDA 里看逻辑, 很简单, flag 就是长度51的字符串, 给出了每个字符之间的约束关系, 
简单举几个约束的例子如下:

	flag[0] == flag[6] + (flag[38] ^ flag[30]) - flag[8]
	flag[1] == (flag[42] ^ (flag[38] ^ flag[20] ^ flag[19]))

	......

	flag[49] == (flag[27] ^ flag[37])
	flag[50] == ((flag[13] + flag[8] + flag[17]) ^ (flag[24] ^ flag[15]))

这要是往常, 我就自己写个回溯噼里啪啦解, 但是这次无论是复杂度还是写法都很麻烦, 赛后学习了一下这道题的解法, 发现了 [Z3](https://github.com/Z3Prover/z3) 这个牛逼的东东. 
具体说来 Z3 是一个 SMT求解器, 说到 [SMT](https://en.wikipedia.org/wiki/Satisfiability_modulo_theories), 我们来看看[约束满足问题](https://zh.wikipedia.org/wiki/%E7%BA%A6%E6%9D%9F%E8%A1%A5%E5%81%BF%E9%97%AE%E9%A2%98), 本题就是典型的约束满足问题, 类似的还有八皇后, 数独等问题, 说白了就是给定变量, 给定变量范围及变量之间的约束逻辑关系, 求变量. 而SMT则是约束满足问题的特定形式, 另外一个有名的约束可满足问题是 [SAT](https://zh.wikipedia.org/wiki/%E5%B8%83%E5%B0%94%E5%8F%AF%E6%BB%A1%E8%B6%B3%E6%80%A7%E9%97%AE%E9%A2%98), 是第一个被证明的 [NP-完全问题](https://zh.wikipedia.org/wiki/NP%E5%AE%8C%E5%85%A8)  
一般这种问题的解法就是回溯 + 局部启发式搜索, Z3 作为 SMT 求解器, 给我们带来了巨大的便利, 可以方便的使用它提供的各种编程语言的接口来方便的解决约束满足问题, 我使用的是python api, 感觉用起来非常方便, 个人觉得比同样能解决这类问题的 [Prolog](https://zh.wikipedia.org/wiki/Prolog) 语言更友好易学, [这里](http://www.cs.tau.ac.il/~msagiv/courses/asv/z3py/guide-examples.htm) 给出一个很不错的 Z3 python api 教程. 另外与 Z3 类似的求解器其实有好多, 有一个专门的标准叫SMT-LIB, [这里](http://smtlib.cs.uiowa.edu/solvers.shtml) 列举了好些满足该标准的SMT求解器. 

### 0x02 解法 
下面给出, 使用本题使用 Z3 python api 的解法:  

	from z3 import *
	
	s = Solver()
	flag = [BitVec('x_%s' % i, 8) for i in range(51)]
	
	s.add(flag[0] == flag[6] + (flag[38] ^ flag[30]) - flag[8])
	s.add(flag[1] == (flag[42] ^ (flag[38] ^ flag[20] ^ flag[19])))
	s.add(flag[2] == flag[35] + flag[36] - flag[19] - flag[3] - flag[44])
	s.add(flag[3] == flag[19] + (flag[17] ^ (flag[41] - flag[10] - flag[10])))
	s.add(flag[4] == flag[33] - flag[21])
	s.add(flag[5] == (flag[4] ^ (flag[4] ^ flag[8] ^ flag[39])))
	s.add(flag[6] == (flag[14] ^ (flag[10] + flag[25] - flag[39])))
	s.add(flag[7] == flag[32] + (flag[15] ^ flag[1]))
	s.add(flag[9] == (flag[24] ^ flag[7]))
	s.add(flag[10] == flag[32] + (flag[49] ^ flag[17]) - flag[4])
	s.add(flag[11] == (flag[42] ^ flag[38]) - flag[17] - flag[8])
	s.add(flag[12] == flag[14] + flag[8])
	s.add(flag[13] == flag[45] + flag[20])
	s.add(flag[14] == flag[9] + (flag[20] ^ (flag[25] - flag[48])))
	s.add(flag[15] == flag[18] - flag[31])
	s.add(flag[16] == (flag[24] ^ flag[46]))
	s.add(flag[17] == ((flag[13] + flag[2] + flag[47]) ^ (flag[14] ^ flag[50])))
	s.add(flag[18] == flag[0] + flag[36] + flag[44] - flag[3])
	s.add(flag[19] == (flag[41] ^ flag[30]) - flag[25] - flag[28])
	s.add(flag[20] == (flag[25] ^ flag[44]))
	s.add(flag[21] == flag[25] + ((flag[28] + flag[22]) ^ (flag[39] ^ flag[21])))
	s.add(flag[22] == (flag[31] ^ (flag[44] - flag[4] - flag[12])) - flag[30])
	s.add(flag[23] == (flag[39] ^ (flag[32] - flag[14])))
	s.add(flag[24] == (flag[21] ^ (flag[0] ^ flag[18] ^ flag[21])))
	s.add(flag[25] == flag[18] + flag[4] + (flag[12] ^ flag[17]) - flag[11])
	s.add(flag[26] == (flag[32] ^ flag[46]) + flag[49] + flag[20])
	s.add(flag[27] == flag[36] + flag[25] + flag[39] - flag[48])
	s.add(flag[28] == (flag[14] ^ flag[15]))
	s.add(flag[29] == flag[1] + flag[35] - flag[42])
	s.add(flag[30] == flag[8] - flag[31] - flag[30] - flag[24])
	s.add(flag[31] == (flag[42] ^ (flag[15] + flag[18] - flag[29])))
	s.add(flag[32] == flag[14] + flag[5] + flag[15] - flag[44])
	s.add(flag[33] == (flag[20] ^ (flag[45] - flag[15])) - flag[32])
	s.add(flag[34] == (flag[3] ^ flag[33]) - flag[20] - flag[10])
	s.add(flag[35] == (flag[44] ^ (flag[6] - flag[43])) + flag[1] - flag[44])
	s.add(flag[36] == (flag[49] ^ (flag[31] + flag[25] - flag[28])))
	s.add(flag[37] == flag[11] + (flag[34] ^ flag[31]) - flag[34])
	s.add(flag[38] == flag[42] + (flag[27] ^ flag[36]) - flag[5])
	s.add(flag[39] == (flag[37] ^ flag[8]))
	s.add(flag[40] == (flag[44] ^ (flag[7] + flag[28])) - flag[10])
	s.add(flag[41] == (flag[20] ^ (flag[7] ^ flag[17] ^ flag[26])))
	s.add(flag[42] == flag[50] + flag[1] - flag[28])
	s.add(flag[43] == flag[46] + flag[33] - flag[15])
	s.add(flag[44] == ((flag[24] + flag[42] + flag[16]) ^ (flag[45] ^ flag[21])))
	s.add(flag[45] == flag[22] - flag[40])
	s.add(flag[46] == flag[12] - flag[46] - flag[7] - flag[35])
	s.add(flag[47] == (flag[39] ^ (flag[15] + flag[26])) - flag[12])
	s.add(flag[48] == (flag[11] ^ (flag[15] - flag[8])))
	s.add(flag[49] == (flag[27] ^ flag[37]))
	s.add(flag[50] == ((flag[13] + flag[8] + flag[17]) ^ (flag[24] ^ flag[15])))
	
	for i in range(51):
		s.add(flag[i] >= 32)
		s.add(flag[i] <= 126)
	
	s.check()
	m = s.model()
	res = []
	for i in range(51):
		res.append(chr(m[flag[i]].as_long()))
	print ''.join(res)

### 0x03 深入
在学习 Z3 的过程中, 发现其有更深层的用法, 即用于 [符号执行](https://zh.wikipedia.org/wiki/%E7%AC%A6%E5%8F%B7%E6%89%A7%E8%A1%8C), 配合污点分析等等技术, 最终实现一定程度上的自动化漏洞挖掘. 接下来我们分析下符号执行的相关原理, 并给出本题利用符号执行的解法.  