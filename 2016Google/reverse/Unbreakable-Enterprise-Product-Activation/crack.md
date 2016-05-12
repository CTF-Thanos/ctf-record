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
在学习 Z3 的过程中, 发现其有更深层的用法, 即用于 [符号执行](https://zh.wikipedia.org/wiki/%E7%AC%A6%E5%8F%B7%E6%89%A7%E8%A1%8C), 配合污点分析等等技术, 最终实现一定程度上的自动化漏洞挖掘. 先不扯那么远, 我们先来聊聊符号执行, 符号执行目前就我个人粗浅的理解是这样的:  

想象一下在你解决和本题类似的逆向题时, 在静态分析中我们完全根据指令来分析程序, 我们无法确定程序在某一个状态下的某个寄存器某个内存地址的具体的值, 这种情况如果想分析程序如何走到某个分支就需要对整个程序有一个比较完整的理解, 为了加快速度, 我们往往可以利用调试的手段来辅助, 因为调试可以让可执行程序真实的运行起来, 我们可以观察在不同的初始输入下在某一个分支路径上会有怎样具体的值, 以此来猜测可能的输入是什么, 但有时候这并不会带来什么实质帮助. 类比一下我们刚刚提到的约束满足问题, 一个是求可以让程序走进验证成功分支的输入, 一个是求可以满足约束关系的变量, 如此的相似不是么? 所以把程序的各种赋值指令都看成对变量范围的指定, 各种跳转指令的条件都看做约束关系, 我们就可以把一个可执行程序的代码转成约束关系从而利用SMT求解器来得到我们想要的输入, 这个过程如同把汇编指令转成另一种语言去执行, 而且所有的输入都被符号化, 仿佛把程序的执行从符号的意义上虚拟化, 数学成为了这个程序赖以执行的虚拟机. 从z3的角度来看, 我们可以把程序的汇编指令转成 z3 的约束语句, 把输入当成z3中类似 BitVec 声明的变量, 最后求解输入的具体. 所谓的符号执行大概如是.(我可能理解有误, 还望各位及时指出)  
那么把程序指令转成约束关系, 整个这个过程感觉可以抽取成一个框架用来分析各种二进制文件, 是否有现成好用的框架呢? 这里要推荐的是名气很大的 [angr](https://github.com/angr/angr).  

下面给出使用 angr 来解决本题的方案(由于本题太过典型, angr官网将其收录到官方example中, 以下便是angr官方的解决脚本):  

	"""
	In this challenge we are given a binary that checks an input given as a
	command line argument. If it is correct, 'Thank you - product activated!' is
	printed. If it is incorrect, 'Product activation failure %d' is printed with a
	specific error code.
	Reversing shows that the program verifies that various operations on specific 
	characters of input are equal to zero. Because of the program's linear nature
	and reliance on verbose constraints, angr is perfect for solving this challenge
	quickly. On a virtual machine, it took ~7 seconds to solve.
	Author: scienceman (@docileninja)
	Team: bitsforeveryone (USMA)
	"""
	import angr
	
	START_ADDR = 0x4005bd # first part of program that does computation
	AVOID_ADDR = 0x400850 # address of function that prints wrong
	FIND_ADDR = 0x400830 # address of function that prints correct
	INPUT_ADDR = 0x6042c0 # location in memory of user input
	INPUT_LENGTH = 0xf2 - 0xc0 + 1 # derived from the first and last character
	                               # reference in data
	
	def extract_memory(state):
	    """Convience method that returns the flag input memory."""
	    return state.se.any_str(state.memory.load(INPUT_ADDR, INPUT_LENGTH))
	
	def char(state, n):
	    """Returns a symbolic BitVector and contrains it to printable chars for a given state."""
	    vec = state.se.BVS('c{}'.format(n), 8, explicit_name=True)
	    return vec, state.se.And(vec >= ord(' '), vec <= ord('~'))
	
	def main():
	    p = angr.Project('unbreakable')
	
	    print('adding BitVectors and constraints')
	    state = p.factory.blank_state(addr=START_ADDR)
	    for i in range(INPUT_LENGTH):
	        c, cond = char(state, i)
	        # the first command line argument is copied to INPUT_ADDR in memory
	        # so we store the BitVectors for angr to manipulate
	        state.memory.store(INPUT_ADDR + i, c)
	        state.add_constraints(cond)
	
	    print('creating path and explorer')
	    path = p.factory.path(state)
	    ex = p.surveyors.Explorer(start=path, find=(FIND_ADDR,), avoid=(AVOID_ADDR,))
	
	    print('running explorer')
	    ex.run()
	
	    flag = extract_memory(ex._f.state) # ex._f is equiv. to ex.found[0]
	    print('found flag: {}'.format(flag))
	
	    return flag
	
	def test():
	    assert main() == 'CTF{0The1Quick2Brown3Fox4Jumped5Over6The7Lazy8Fox9}'
	
	if __name__ == '__main__':
	    main() 

下面我简单说讲解下次脚本:  