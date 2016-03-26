好了，这是第一次写record啊，哈哈哈
记录一下关于看完 catvideo 外国队伍writeup的总结，加上一些实际操作。

A.	题目描述，题目中给出的是一个视频(.mp4)文件，打开看全是雪花图像如
 
仔细观察还是能发现雪花有细微的变化，但没有声音
联想以前一次比赛题目，可以利用ffmegp工具转换格式看看
好吧就是这个坑爹的工具，我装了好多次，然后发现其实是装成功的，
不能直接ffmegp –i ，我用的是./ffmegp  –i catvideo.mp4  1.jpg 
发现jpg转出来就一张，都是雪花，stegsolve后没什么东西，然后ffmegp提示转换文件名字要用类似%03d.，然后别人的writeup是转bmp的，所以，试了一下
./ffmegp  –i catvideo.mp4 %03d.bmp
bmp转了1900+张图片，一帧一张，全部总共1.23G(忘了控制大小)， 
然后第一帧和20帧XOR一下，
 
没有什么，继续往后XOR
我的跨度是一百所以试试第一帧与第150帧XOR
好吧，就出现了
 
这题难怪好多人说简单，确实，和以往有的题类似，熟悉ffmegp工具和stegsolve对这类题还是很有帮助，其实也可以写脚本




外国队伍wp
两种方法可以看看
http://err0r-451.ru/2016-bctf-forensic-catvideo-150-pts/
https://github.com/DMArens/CTF-Writeups/tree/master/2016/BCTF/Forensics150
