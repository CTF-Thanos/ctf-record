### 题目
 题目是给的一个[mp4](catvideo-497570b7e2811eb52dd75bac9839f19d7bca5ef4.mp4)文件,该文件打开全部都是雪花，无声音，无法看清楚。

该题目由S.K做出来。

联想以前一次比赛题目，可以利用ffmegp工具转换格式看看
好吧就是这个坑爹的工具，我装了好多次，然后发现其实是装成功的，
不能直接ffmegp –i ，我用的是`./ffmegp  –i catvideo.mp4  1.jpg`
发现jpg转出来就一张，都是雪花，stegsolve后没什么东西，然后ffmegp提示转换文件名字要用类似%03d.，然后别人的writeup是转bmp的，所以，试了一下
`./ffmegp  –i catvideo.mp4 %03d.bmp`
bmp转了1900+张图片，一帧一张，全部总共1.23G(忘了控制大小)

![img_lst](img_lst.png)

没有什么，继续往后XOR(使用的Stegsolve)

![steg_xor](steg_xor.png)

我的跨度是一百所以试试第一帧与第150帧XOR
好吧，就出现了结果,看见了flag.


这题难怪好多人说简单，确实，和以往有的题类似，熟悉ffmegp工具和stegsolve对这类题还是很有帮助，其实也可以写脚本来做。


比赛期间,S.K同学告诉我转化格式，试过用ffmpeg转化为gif/jpg,但是都无疾而终。仍然是雪花图片。
我在stackoverflow上找到一个转化方式：
`ffmpeg -i catvideo-497570b7e2811eb52dd75bac9839f19d7bca5ef4.mp4 -vf fps=1 out%d.bmp`  加上fps=1这个的好处是转化出来的图片文件数量不多。

赛后看了一些网上的题解，才知道通常这类隐藏数据的方式都是使用某个key和原始数据进行XOR,我开始尝试了Python的PIL库

``` python
from PIL import Image,ImageChops
import glob
im1 = Image.open('out1.bmp')
cnt = 1
for f in glob.glob("out*.bmp"):
	ImageChops.logical_xor(Image.open(f), im1).show()
	if cnt % 10 == 0:
	    raw_input("press key to continue")
	cnt += 1
```

结果函数不可以用在RGB模式上，网上对于`logical_xor`的文档也少的可怜.遍历了我知道的各种格式，发现只支持灰度图，但是XOR出来虽然知道有东西，也看不清楚。
后来在网上找到了这么一个地址:[https://www.cs.hmc.edu/~jlevin/ImageCompare.py](https://www.cs.hmc.edu/~jlevin/ImageCompare.py)

用了其中的XOR搞定。

最后得到的图片大概这样子：

![flag.bmp](flag.bmp)

###扩展阅读

+ [http://err0r-451.ru/2016-bctf-forensic-catvideo-150-pts/](http://err0r-451.ru/2016-bctf-forensic-catvideo-150-pts/)
+ [https://github.com/DMArens/CTF-Writeups/tree/master/2016/BCTF/Forensics150](https://github.com/DMArens/CTF-Writeups/tree/master/2016/BCTF/Forensics150)
