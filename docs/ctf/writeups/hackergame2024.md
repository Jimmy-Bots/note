---
counter: True
---

# Hackergame 2024

!!! abstract
    这已经是去年的赛事了，记得是ZJUCTF2024后的趁热打铁，到了2025年了才来更一下xs。不过感觉以后越来越忙应该也没什么时间大打了，善始善终吧，但是因为太久远了，所以就挑几道印象深刻的写一写

    官方题解：[Hackergame 2024](https://github.com/USTC-Hackergame/hackergame2024-writeups)

## 每日论文太多了

没想到真有人把flag藏论文中哈哈哈哈，题目很简单搜一下flag，然后移动一下元素就能看到。

[论文链接](https://dl.acm.org/doi/10.1145/3650212.3652145)

## PowerfulShell

???+ "题目逻辑"
    ```bash
    #!/bin/bash

    FORBIDDEN_CHARS="'\";,.%^*?!@#%^&()><\/abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0"

    PowerfulShell() {
        while true; do
            echo -n 'PowerfulShell@hackergame> '
            if ! read input; then
                echo "EOF detected, exiting..."
                break
            fi
            if [[ $input =~ [$FORBIDDEN_CHARS] ]]; then
                echo "Not Powerful Enough :)"
                exit
            else
                eval $input
            fi
        done
    }

    PowerfulShell
    ```

核心思路就是利用仅剩的`$-_~{}1-9:`来构造Payload，根据已知，我们易得`~`可以获取`$HOME`，即`/player`，而`$-`是`hB`，那么我们轻而易举的就能构造出很多常见的命令，比如`ls`，`cat`等等，然后就能拿到flag了。

## 强大的正则表达式

很有意思的一题，用正则表达式来实现数字是否整除，甚至CRC校验，因为不是算法出身，还是第一次了解到，学习了。

??? "题目代码"
    ```python
    import re
    import random

    # pip install libscrc
    import libscrc

    allowed_chars = "0123456789()|*"
    max_len = 1000000
    num_tests = 300

    difficulty = int(input("Enter difficulty level (1~3): "))
    if difficulty not in [1, 2, 3]:
        raise ValueError("Invalid difficulty level")

    regex_string = input("Enter your regex: ").strip()

    if len(regex_string) > max_len:
        raise ValueError("Regex string too long")

    if not all(c in allowed_chars for c in regex_string):
        raise ValueError("Invalid character in regex string")

    regex = re.compile(regex_string)

    for i in range(num_tests):
        expected_result = (i % 2 == 0)
        while True:
            t = random.randint(0, 2**64)  # random number for testing
            if difficulty == 1:
                test_string = str(t)  # decimal
                if (t % 16 == 0) == expected_result:  # mod 16
                    break
            elif difficulty == 2:
                test_string = bin(t)[2:]  # binary
                if (t % 13 == 0) == expected_result:  # mod 13
                    break
            elif difficulty == 3:
                test_string = str(t)  # decimal
                if (libscrc.gsm3(test_string.encode()) == 0) == expected_result:  # crc
                    break
            else:
                raise ValueError("Invalid difficulty level")
        regex_result = bool(regex.fullmatch(test_string))
        if regex_result == expected_result:
            print("Pass", test_string, regex_result, expected_result)
        else:
            print("Fail", test_string, regex_result, expected_result)
            raise RuntimeError("Failed")

    print(open(f"flag{difficulty}").read())
    ```

第一问非常简单，十进制数是否是16的倍数，这个规律很好找，也很好写，所以就不多赘述了。

第二问是二进制数是否是13的倍数，这个就有点难度了，因为二进制数的正则表达式不是那么好写，但是我们可以构造一个有限状态自动机（DFA）来判断。

构造方法：DFA 的状态代表余数（有 0~12 一共 13 个状态），初始状态是 0，每次读入一个 bit 更新余数（状态转移）（`s:=(s*2+b)%13`），读入完毕后如果 DFA 处于 0 状态（余数为 0），就意味着这个二进制数整除 13。

然后可以使用 [状态消除算法](https://courses.grainger.illinois.edu/cs374/sp2019/extra_notes/01_nfa_to_reg.pdf)，将 DFA 转化为正则表达式。

第三问也是类似的方法可解，这次 DFA 的状态是线性反馈移位寄存器（LFSR）的状态，寄存器有 3 位，一共是 8 种状态（000~111），DFA 初始状态是 111，每次读入一个字符更新状态，读入完毕后如果 DFA 处于 000 状态，就意味着这个字符串符合要求。

??? done "题解"
    ```python
    # pip install greenery
    # pip install regex
    from greenery import Fsm, Charclass, rxelems
    import regex as re
    import random

    m = 13
    d = 2

    digits = [Charclass(str(i)) for i in range(d)]
    other = ~Charclass("".join(str(i) for i in range(d)))
    alphabet = set(digits + [other])
    states = set(range(m + 1))  # m is the dead state
    initial_state = 0
    accepting_states = {0}
    transition_map = dict()
    for s in range(m):
        transition_map[s] = {digits[i]: (s * d + i) % m for i in range(d)}
        transition_map[s][other] = m
    transition_map[m] = {digits[i]: m for i in range(d)}
    transition_map[m][other] = m

    dfa = Fsm(
        alphabet=alphabet,
        states=states,
        initial=initial_state,
        finals=accepting_states,
        map=transition_map,
    )

    def convert_regex(regex):
        # `(...)?` -> `((...)|)`
        while '?' in regex:
            regex = re.sub(r'\((.*?)\)\?', r'(\1|)', regex)
        # Handle `{n}` quantifier
        n = 1
        while '{' in regex:
            while '{' + str(n) + '}' in regex:
                regex = re.sub(r'(\((.*?)\)|\w)\{n\}'.replace('n', str(n)), r'\1' * n, regex)
            n += 1
        # [abc] -> (a|b|c)
        while '[' in regex:
            def convert_charset(match):
                chars = match.group(1)
                return '(' + '|'.join(chars) + ')'
            regex = re.sub(r'\[([^\]]+)\]', convert_charset, regex)
        assert set(regex) <= set("0123456789|()*")
        return regex

    dfa = dfa.reduce()
    regex = rxelems.from_fsm(dfa)
    regex = regex.reduce()
    regex = convert_regex(str(regex))
    print(regex)
    ```

这里就贴一下官方题解啦:)

## 优雅的不等式

由于我们注意力惊人，注意到这篇[知乎文章](https://zhuanlan.zhihu.com/p/669285539)，其中第一种类型就是证明 $\pi$ 大于一个有理数。

我们使用如下定积分：

$\int_0^1 \frac{x^n(1-x)^n(a+bx+cx^2)}{1+x^2} dx$

根据不同的`n`，越大不等式越紧，所以只要给定一个`n`，然后求解`a,b,c`即可。使用python的sympy就能解答。

???+ done "解题代码"
    ```python
    from pwn import *
    import sympy

    N = 100
    x, a, b ,c = sympy.symbols('x a b c')

    f = (x**N*(1-x)**N*(a+b*x+c*x**2))/(1+x**2)

    inv = sympy.integrate(f, (x, 0, 1))
    inv = sympy.simplify(inv)
    # inv = sympy.collect(inv, ["log(2)", "pi"])

    l2 = inv.coeff("log(2)")
    pic = inv.coeff("pi")
    left = sympy.simplify(inv - l2 * sympy.log(2) - pic * sympy.pi)

    def get_exp(p, q):
    expr = [l2, pic - q, left + p]
    r3 = sympy.solve(expr, [a, b, c])
    r3 = zip([a, b, c], r3.values())
    return f.subs(r3)/q

    pp = remote('ip', port)
    pp.recvuntil(b'Please input your token:')
    pp.sendline(b'your_token')

    def solve():
    que = pp.recvline_contains(b"Please prove that ")
    que = que.decode().split(" ")[-1][4:]
    pp.recvuntil(b'Enter the function f(x):')
    if len(que) == 1:
        ans = b'4*((1-x**2)**(1/2)-(1-x))'
    else:
        p, q = map(int, que.split('/'))
        ans = str(get_exp(p, q)).encode()
    pp.sendline(ans)
    pp.recvuntil(b'Q.E.D.')

    for i in range(40):
    solve()
    print(f'[{i+1}/40] PASS')
    pp.interactive()
    ```

## 无法获得的秘密

这道题很有意思，不能复制粘贴交互的novnc，同时还有丢包（sad

我用的是笨办法，获取内容然后ocr，为了降低ocr难度，我用了二进制的方式，导致结果很多，还有丢包，手动校正了一下（我好菜qaq

看了题解发现可以用灰度编码的方法，剩下的就是自动化复制代码到vnc了，这个很简单，就不多说了。

其实顺着题解的思路，可以试试二维码，不过我没事（笑

## 零知识数独

这题主要学习一下零知识证明的理论和逻辑，以及相关circom工具，不过没有深入了解，仅作记录

Ref：[zk-bug-tracker](https://github.com/0xPARC/zk-bug-tracker)，[1. Missing Bit Length Check](https://github.com/0xPARC/zk-bug-tracker#dark-forest-1)，[14. Assigned but not Constrained](https://github.com/0xPARC/zk-bug-tracker?tab=readme-ov-file#14-mimc-hash-assigned-but-not-constrained)，[Circom](https://github.com/iden3/circom)。

## 先不说关于我从零开始独自在异世界转生成某大厂家的 LLM 龙猫女仆这件事可不可能这么离谱，发现 Hackergame 内容审查委员会忘记审查题目标题了ごめんね，以及「这么长都快赶上轻小说了真的不会影响用户体验吗🤣」

又是一道大模型逆向题，

> 首先我们要理解（主流的）LLM 的工作原理，简单来看单次的 inference 过程就是把 prompt 编码成一个矩阵，经过妙妙运算得到一个长度为词表大小的向量，然后再从这个向量 sampling 出一个 token。看一看代码就能发现，题目里用到的 llama-cpp-python 的默认使用了 top-p、top-k 等 sampler。我们其实不需要关心具体细节，只需要把他们理解成一个更改各个 token 出现的概率的向量函数即可。我们可以原样推理一次，把不符合语法的 token（比如当前位置之后的字符是 xxxxxp...，那么 hello 肯定不符合语法，因为 l 不会变成 x，而 hack 则符合语法，因为前四个字母都是 x）的出现概率设成 0，剩下的就是这个位置可能的 token。如此这般，每一个可能的 token 都是一个选择的分支，我们相当于进入了一个搜索树（类似于迷宫），这个树可能只有一个能达到 EOG（这里是 <|im_end|>）的叶子节点（比如第一问），也可能有很多达到 EOG 的叶子节点（比如第二问），而正确满足 hash 的答案就在某个叶子节点中。

???+ done "解题思路"
    解题代码是基于 llama.cpp 改的，diff 被导出成了 exp.patch 文件，选手想自己运行的话方法大致如下：

    clone https://github.com/ggerganov/llama.cpp/
    checkout c421ac072d46172ab18924e1e8be53680b54ed3b
    apply exp.patch
    modify examples/simple/simple.cpp L23 for censored chars
    modify examples/simple/simple.cpp L178 for prompt
    modify examples/simple/simple.cpp L202 for number of threads
    make (optional with GGML_CUDA=1)
    copy after.txt & before.sha256 to current dir
    ./llama-simple -m /path/to/qwen2.5-3b-instruct-q8_0.gguf
    echo "flag{llm_lm_lm_koshitantan_$(sha512sum output.txt | cut -d ' ' -f1 | cut -c1-16)}"