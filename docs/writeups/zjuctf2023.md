---
encryption_info_message: "请输入密码查看本篇文章（可以联系我）"
placeholder: '在此输入密码'
summary: '🔒 ZJUCTF 2023 Writeup'
password: 'zjuctf@888'
counter: True
---

# ZJUCTF 2023 Writeup

!!! abstract
    这次比赛，感觉 misc 题难度适中，有些题赛后发现并不难，只是自己当时懒得折腾了；至于 web 题，以为这次有所长进，结果很多 Java 题，没怎么做过，手生 qaq ；reverse 和 pwn 本身也不太会，挑了几道简单的做；crypto 一如既往的不会，只靠非预期解拿了些分。虽然排名比去年高，但是自我还是不太满意，还要继续学习！

> 碎碎念：本来其实不想打的，只是想随便做几道玩玩，因为实验室和课程事情很多，但是谁知道还是有点上瘾了，打了几天，沉没成本有点大（x，于是索性就打完了。刚开始以为7天，结果今年增加到9天，很累，周末没了！

> *2024.10.30 今天整理2024的writeup时候发现居然上次没更新上去，特地补上，懒得改了，直接raw贴qwq*

***2023.10.15***

*时间原因，都没贴脚本（不然太长了，大部分脚本应该都在*

##### *无关，请跳过* 

~~*今年怎么奖品这么少？ACTF2020二十几名还有书，ZJUCTF2022十几名还有手环，今年啥也没有QAQ！AAA怎么会穷呢（ ps应该是我太菜了*~~

*上面这个话不知道谁说的，反正不是我说的*

## 普通的魔法史

根据语法手动推理就行，写个脚本可能也行，哪个快就哪个吧，反正不是批量处理

## 23w40a_or_ctf

附件发现是服务器形式的地图

网上找个这个版本服务端部署，然后打开就行了

进去把任务都做了，就能拿到flag

不能用指令的有那个二维码(replace替换一下相关方块)，羊毛16进制，月饼的data_value

ps:后来发现非预期解，查找常加载区块，其实可以定位到命令方块那个区块，可以一下子获得一部分碎片

## RURU

发现压缩包，有秘密，hex看一下，发现伪加密，然后写个脚本循环解压，倒数第二个压缩包告知8位密码，猜测是文件名？不管了，反正爆破很快。
ps：其实视力好可以跳过循环解压，直接从hex中扣出最后一个压缩包（

## Genshin Impact Format

替换一下color table就行了，另外可能就是长宽之类的问题，没细究，因为系统自带预览就能看到flag了

## Proof-of-Work

略

## Quine Relay

研究了一下，其实和写quine差不多

参考了github上一个c/c++/java的relay

## Minesweeper Master

用了现成的轮子

## easy reverse

具体有点忘了，印象里怎么记得把密文放进去跑了一遍flag就出来了？哦对，swap table要改一下

## hash predictor 1

先写一个输出程序，留好md5数据存放的位置，然后用md5_fastcall碰撞，最后把md5写入相应位置就好

非预期：读取自身算md5后输出

## hash predictor 2

非预期：因为有回显，且有权限，直接可以读到flag

## welcome

通过修改stack上的返回地址，执行backdoor函数

## baby sql

利用mysql 8的table进行盲注

## easy sql

flask ssti制造回显

session伪造突破长度限制

udf提权，获取root权限

## babyphp

php反序列化，略

## easy calc

屏蔽了原型链，但是可以通过this没有屏蔽，就可以调用相关方法了

## Baby（for real）js

ejs ssti，略

## rev signin

根据逆向的encrypt代码，然后省力直接8位8位的爆破就行了

## ZJU dorm

go逆向，先用脚本恢复符号，然后动调，记得绕过debugger检测就好了

## IF ELSE

发现只有一个字符正确后才会跳转到正确的函数，直接动调一路跟踪即可

## 500mazes

一开始以为要找bug通关，后来发现这个maze有解，那直接写一个动态规划算法求解，然后把ida里的maze和方向数据清洗出来就好了

## ⚡ 遥 遥 领 先 ⚡

hap应该和apk差不多，解压缩打开，发现不是java，是ets，直接定位到相关加密算法，然后写个脚本解密就好了
