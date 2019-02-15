---
layout: post
title: SUCTF 2018 Write-up
date: 2018-05-28 15:28:53.000000000 +08:00
tags: writeup
mathjax: true
---

1. 还是先写最近的……否则真的容易忘
2. 想交个wp从来都没资格


## Crypto
这次SUCTF算是好好的复习了一下数论（明明本来就很简单）。
然而为什么这么多数学公式……排个版真难

### SandGame
这是一道Misc题，但由于涉及了数论知识，故写在Crypto里。
#### 题目
game.py

```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-

import flag

flag = flag.flag
sands = int(flag[5:-1].encode("hex"), 16)

holes = [257, 263, 269, 271, 277, 281, 283, 293, 307, 311, 313, 317, 331, 337, 347, 349, 353, 359, 367, 373]

with open("sand.txt", "w") as f:
    for i in range(len(holes)):
        sand = sands % holes[i]
        f.write(str(sand)+"\n")
```

sand.txt
```
222
203
33
135
203
62
227
82
239
82
11
220
74
92
8
308
195
165
87
4
```

#### 同余方程组
这段代码加上sand.txt给出来的解，实际上构成了下面的同余方程组：

$$
\begin{aligned}
    n \equiv 222 \pmod{257}\\
    n \equiv 203 \pmod{263}\\
    n \equiv 33 \pmod{269}\\
    n \equiv 135 \pmod{271}\\
    n \equiv 203 \pmod{277}\\
    n \equiv 62 \pmod{281}\\
    n \equiv 227 \pmod{283}\\
    n \equiv 82 \pmod{293}\\
    n \equiv 239 \pmod{307}\\
    n \equiv 82 \pmod{311}\\
    n \equiv 11 \pmod{313}\\
    n \equiv 220 \pmod{317}\\
    n \equiv 74 \pmod{331}\\
    n \equiv 92 \pmod{337}\\
    n \equiv 8 \pmod{347}\\
    n \equiv 308 \pmod{349}\\
    n \equiv 195 \pmod{353}\\
    n \equiv 165 \pmod{359}\\
    n \equiv 87 \pmod{367}\\
    n \equiv 4 \pmod{373}
\end{aligned}
$$

类似这样的方程组，在数论中称作一元线性**同余方程组**。

#### 中国剩余定理
著名的[**中国剩余定理**](https://zh.wikipedia.org/wiki/%E4%B8%AD%E5%9B%BD%E5%89%A9%E4%BD%99%E5%AE%9A%E7%90%86)描述了一元线性同余方程组有解的判定条件及其解法：

* 一元线性同余方程组

$$
\begin{aligned}
\(\mathbf{S}) : \quad \left\{ \begin{matrix} x \equiv a_1 \pmod {m_1} \\ x \equiv a_2 \pmod {m_2} \\ \vdots \qquad\qquad\qquad \\ x \equiv a_n \pmod {m_n} \end{matrix} \right.
\end{aligned}
$$

有解，当且仅当其所有模数\\(m_1, m_2, \ldots, m_n\\)互质（这里的S式符合这个条件）；

* 记\\(M=\prod_{i=1}^{n} m_i\\)，\\(M_i = M / m_i\\)，\\(t_i\\)为\\(M_i\\)的模逆元，则S的解符合以下公式：

$$
\begin{aligned}
x \equiv \sum_{i_1}^{n} a_it_iM_i \pmod{M}
\end{aligned}
$$

#### 模逆元和扩展欧几里得算法
讲到这里，就不得不提一下[模逆元](https://zh.wikipedia.org/wiki/%E6%A8%A1%E5%8F%8D%E5%85%83%E7%B4%A0)的概念：整数a在模N意义下的模逆元是指满足以下公式的整数b：

$$
\begin{aligned}
a^{-1} \equiv b \pmod{N}
\end{aligned}
$$

模逆元仅在a和N互质的情况下存在。换言之，在模N的意义下，**对a的除法可以通过和a的模逆元b的乘法来达成**（RSA一题的主要原理）。

求模逆元使用[扩展欧几里得算法](https://zh.wikipedia.org/wiki/%E6%89%A9%E5%B1%95%E6%AC%A7%E5%87%A0%E9%87%8C%E5%BE%97%E7%AE%97%E6%B3%95)（欧几里得算法就是辗转相除法）来实现。

```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-

def extended_eucild(a, b):
    if b == 0:
        return a, 1, 0
    d, x, y = extended_eucild(b, a % b)
    x, y = y, x - a / b * y
    return d, x, y
```

#### 解法
说了这么多，本题的解法就是通过扩展欧几里得算法求解上文给定的一元线性同余方程组S：

```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-

a = [222, 203, 33, 135, 203, 62, 227, 82, 239, 82, 11, 220, 74, 92, 8, 308, 195, 165, 87, 4]
w = [257, 263, 269, 271, 277, 281, 283, 293, 307, 311, 313, 317, 331, 337, 347, 349, 353, 359, 367, 373]

def extended_eucild(a, b):
    if b == 0:
        return a, 1, 0
    d, x, y = extended_eucild(b, a % b)
    x, y = y, x - a / b * y
    return d, x, y

def crt(a, w):
    ret = 0
    n = 1
    for i in w:
        n *= i
    for i in range(len(w)):
        m = n / w[i]
        d, x, y = extended_eucild(w[i], m)
        ret = (ret + y * m * a[i] % n) % n
    return ret
    
if __name__ == "__main__":
    print hex(crt(a, w))[2:-1].decode('hex')
```

当然你也可以用 GNU MP 库来实现。

> flag{This_is_the_CRT_xwg)}



### RSA good
题目给定了N和e

```python
N = 342455709448748144126356744976385170973517744602059517490422045682543287960167955127769980654250125331171261846920903825693509591867402054748269545989173880386620770767057995165518626234085821335790902075953939551116777613078301529741199260825495593643848062203477826484698214686522001924292713782595019038086926834360866522789951283935502968545347160597915951673480253253216027297476774028106074570088497425654525031294571609018030761716007610673627163536370205798268831577480146622906265953470659107801115278898533958878045433701201601516984582294147038705649395688342773971893457527598221773710752744142729023770679
e = 65537
```
注意到，给定的N具有[已知的质因数分解](http://factordb.com/index.php?query=342455709448748144126356744976385170973517744602059517490422045682543287960167955127769980654250125331171261846920903825693509591867402054748269545989173880386620770767057995165518626234085821335790902075953939551116777613078301529741199260825495593643848062203477826484698214686522001924292713782595019038086926834360866522789951283935502968545347160597915951673480253253216027297476774028106074570088497425654525031294571609018030761716007610673627163536370205798268831577480146622906265953470659107801115278898533958878045433701201601516984582294147038705649395688342773971893457527598221773710752744142729023770679)：

>   3424557094...79<618> = 7 · 4892224420...97<617>

则p = 7, q = 4892224420...97。计算可得r, d = 1170425334...09。
 
利用[rsatool](https://github.com/ius/rsatool)创建PEM格式的私钥，通过openssl rsautil 解密给定密文：

```bash
     $ python rsatool.py -f PEM -o key.pem -n 3424557094...79 -d 1170425334...09
     $ openssl rsautl -decrypt -in pp.txt -inkey key.pem -out ppp.dec -raw
```

得到flag:

> SUCTF{Ju5t_hav3_fun_1n_R34_4Ga1N!}

### RSA

```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-

from Crypto.Random import random
import binascii
import hashlib

def invmod(a, n):
    t = 0
    new_t = 1
    r = n
    new_r = a
    while new_r != 0:
        q = r // new_r
        (t, new_t) = (new_t, t - q * new_t)
        (r, new_r) = (new_r, r - q * new_r)
    if r > 1:
        raise Exception('unexpected')
    if t < 0:
        t += n
    return t

smallPrimes = [2, 3, 5, 7, 11, 13, 17, 19]

def primefactor(p):
    for x in smallPrimes:
        if p % x == 0:
            return True
    return False

def isprime(p, n):
    for i in range(n):
        a = random.randint(1, p)
        if pow(a, p - 1, p) != 1:
            return False
    return True

def getprime(bit):
    while True:
        p = random.randint(2**(bit - 1), 2**bit - 1)
        if not primefactor(p) and isprime(p, 5):
            return p

def genKey(keybits):
    e = 3
    bit = (keybits + 1) // 2 + 1

    p = 7
    while (p - 1) % e == 0:
        p = getprime(bit)

    q = p
    while q == p or (q - 1) % e == 0:
        q = getprime(bit)

    n = p * q
    et = (p - 1) * (q - 1)
    d = invmod(e, et)
    pub = (e, n)
    priv = (d, n)

    return (pub, priv)



pub, priv = genKey(2048)
(e,n) = pub
(d,n) = priv
de_hash = set()



def b2n(s):
    return int.from_bytes(s, byteorder='big')

def n2b(k):
    return k.to_bytes((k.bit_length() + 7) // 8, byteorder='big')

def decrypt(cipher):
    md5 = hashlib.md5()
    md5.update(cipher)
    digest = md5.digest()
    if digest in de_hash:
        raise ValueError('Already decrypted')
    de_hash.add(digest)
    return n2b(pow(b2n(cipher), d, n))

if __name__ == '__main__':
    plain = 
    cipher = n2b(pow(b2n(plain), e, n))
    r = random.randint(2, n - 1)
    c = b2n(cipher)
    c2 = (pow(r, e, n) * c) % n
    print (e)
    print (d)
    print (c2,r,n)
```

解读代码，可以看见生成了随机值r,将r和flag分别加密后密文值相乘为c2。给定d, e, c2, r, N，求解flag。

换言之：

$$
\begin{aligned}
    flag^e \cdot r^e \equiv c_2 \pmod{N}
\end{aligned}
$$

考虑到上文[模逆元](https://zh.wikipedia.org/wiki/%E6%A8%A1%E5%8F%8D%E5%85%83%E7%B4%A0)的定义：

$$
\begin{aligned}
    c_2^d \equiv flag \cdot r \pmod{N} \\
    flag \equiv r^-1 \pmod{N}
\end{aligned}
$$

flag*r的值可以通过openssl rsautil解密获得。

问题即转化为求r对N的模逆元。注意，由于N, r较大，这里的扩展欧几里得算法需要用递推实现：

```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-

rmuln = 59115915707374791858043439923167683347161887030487325917423499796492203695323206558401630102578799900218436597230954437778111746830394958514693795703668608134688484361451977542361964879441862660907225537016230464396774501614633550453344389997701652088926779491939154277967969883770814387902879875613743223229395914926347385430187855896072954220734275197088587384423545384415702580062626846187660369433673963925368283292589562338733264483707085154679553150155966218540039768554264425242930122630759578034001784040206494331067331934081888223924875304452399021658433028042708859136403044098846651745078168587090692758339
N = 114074818133739504250047209185005541235104076834407454843024262517200619155182936436785962918603977053600646077595890908700137268868699889055060972806312126948522642693058329618120858621695314653366215387973749480530773400994063166095532724955861880985112526415669572580061355860224679573858183474455325088556643994673345465994162610420723170253345262031304174343327749668957998269532265104735742501097132399909676529715137398930849255495327761212026819673358330120772259339210848101393790624794141497382162858696771273350710124735773784422786465775396114824753700050260926501450377050949467981761553058188615697572753
r = 52590127432976083491238732503614965276544329818688634632767137973918045578992812985289716736906703169505549436697176020030544816837575642406120330044094101926756390272838108002439881182112360442668104629856258556842855755302886464339208354825118921549697616322247157559049488040135615532105368336491021854265473475528069047122826089655385379767218204046669380406790462989897640184187821995434927599001724468830237117370372883192568919508659630357441363324857869183828526986625221594440014568348246392587406204121296464310150086319496345975298783813760793880968818397424849582987543667279959113716063661758542453795636
k = 0


def extended_eucild(a, b):
    p = []
    if b == 0:
        return a, 1, 0
    while b != 0:
        p.append((a, b))
        a, b = b, a % b
    x, y = 1, 0
    while len(p) > 0:
        a, b = p.pop()      
        x, y = y, x - a / b * y
    return x, y
    
    
if __name__ == "__main__":    
    x, y = extended_eucild(r, N)        
    t = (x * rmuln) % N
    print ("%x" % t).decode('hex')
```

> SUCTF{Ju5t_hav3_fun_emmm}

## Misc
### Cycle

```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-

import flag

def encryt(key, plain):
    cipher = ""
    for i in range(len(plain)):
        cipher += chr(ord(key[i % len(key)]) ^ ord(plain[i]))
    return cipher

def getPlainText():
    plain = ""
    with open("plain.txt") as f:
        while True:
            line = f.readline()
            if line:
                plain += line
            else:
                break
    return plain

def main():
    key = flag.flag
    assert key.startswith("flag{")
    assert key.endswith("}")
    key = key[5:-1]
    assert len(key) > 1
    assert len(key) < 50
    assert flag.languageOfPlain == "English"
    plain = getPlainText()
    cipher = encryt(key, plain)
    with open("cipher.txt", "w") as f:
        f.write(cipher.encode("base_64"))

if __name__ == "__main__":
    main()
```

这道题是一个唯密文攻击。给定了通过xor加密后密文cipher.txt，同时给出了以下几个条件：

1. 密钥（即flag）长度在(1, 50)之间
2. 加密的对象是一段英文文本

总体来讲，由于flag较短且密文较长，条件比较充足，这道题相对还是比较简单的。

首先考虑到词频分析法。假定明文是空格（0x20），分析密钥处于可读字符集中的字频。通过分析可以发现，24位的时候有一个明显的高峰，事实也证明了flag确实是24位。

在获得这个24位的可能值之后，通过穷举法分析密钥。尝试可读取字符集中的任意字符，若解密得到的明文处于英文字符集内，即认为该字符有效。通过该法分析可得flag。

但存在一些问题:

1. 实际解法中没有使用词频分析的结论，而是穷举了所有的长度。穷举的时间复杂度是可以接受的。
2. 由于不明原因，使用简单的明文字符集并不能分析出结果，发现需要添加\xef\xbc\x8c等3个字符才能得到flag。这里是出于什么原因不太明确。

解题脚本如下

```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*- 

import logging
import sys

engchar = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789. \x0d\x0a\"'!?(),:;$%-_=\xef\xbc\x8c"

logger = logging.getLogger()
formatter = logging.Formatter('%(asctime)s %(levelname)-8s: %(message)s')
console_handler = logging.StreamHandler(sys.stdout)
console_handler.setFormatter(formatter) 
logger.addHandler(console_handler)
logger.setLevel(logging.WARNING)

with open('cipher.txt') as f:
    cipher = f.read().decode('base_64')
    
def isengchar(s):
    ret = True
    for i in s:
        if not i in engchar:
            return i
    return ret
    
if __name__ == '__main__':
    l = len(cipher)
    for slength in range(2, 50):
        logger.info('Fuzzing Length %d' % slength)
        a = ['@' for i in range(50)]
        for fuzzingchar in engchar:
            for p in range(slength):
                pr = ""
                for i in range(l / slength):
                    pr += chr(ord(cipher[i*slength+p]) ^ ord(fuzzingchar))
                ans = isengchar(pr)
                if ans == True:
                    logger.info('confirm char %s vaild at No. %d' % (fuzzingchar, p))
                    a[p] = fuzzingchar
                else:
                    logger.debug('found char %s(%.2x) in ret for %s(%.2x) at No. %d' % (ans, ord(ans), fuzzingchar, ord(fuzzingchar), p))
            # raw_input()
        print slength, ''.join(a)
```

> flag{Something Just Like This}

### TNT
压缩包层数实在太多……

题目给定的是一个PCAP包，可以看出这个包是SQLMap之类的工具注入DVWA造成的。分析注入本身，发现在guestbook表comment字段中注出一段base64（伪）：

> dont bother me,Im using hammer TNT

额不是这个。。。

> QlpoOTFBWSZTWRCesQgAAKZ///3ry/u5q9q1yYom/PfvRr7v2txL3N2uWv/aqTf7ep/usAD7MY6NHpAZAAGhoMjJo0GjIyaGgDTIyGajTI0HqAAGTQZGTBDaTagbUNppkIEGQaZGjIGmgMgMjIyAaAPU9RpoMjAjBMEMho0NMAjQ00eo9QZNGENDI0zUKqflEbU0YhoADQDAgAaaGmmgwgMTE0AGgAyNMgDIGmTQA0aNGg0HtQQQSBQSMMfFihJBAKBinB4QdSNniv9nVzZlKSQKwidKifheV8cQzLBQswEuxxW9HpngiatmLK6IRSgvQZhuuNgAu/TaDa5khJv09sIVeJ/mhAFZbQW9FDkCFh0U2EI5aodd1J3WTCQrdHarQ/Nx51JAx1b/A9rucDTtN7Nnn8zPfiBdniEzIZn0L1L90ATgJjogOUtiR77tVC3EVA1LJ0Ng2skZVCAt+Sv17EiHQMFt6u8cKsfMu/JaFFRtwudUYYo9OHGLvLxgN/Sr/bhQITPglJ9MvCIqIJS0/BBxpz3gxI2bArd8gnF+IbeQQM3c1.M+FZ+E64l1ccYFRa26TC6uGQ0HnstY5/yc+nAP8Rfsim4xoEiNEEZclCsLAILkjnz6BjVshxBdyRThQkBCesQg=

这不是一个合法的base64，可以看到少了大写"X"而多了"."。替换之，得到：

> 42 5a 68 39 31 41 59 26 ...

查询，425A6839是bzip2的文件头。bzip2 -d解压之，得到文件a:

> fd 37 7a 58 5a 00 00 04 ...

这次是LZMA。xz -d解压之：

> 1f 8b 08 08 03 e1 04 5b ...

又变成gzip（顺带一提，pcap包里的HTTP流量也是gzip压缩的）。gzip -d解压, 得到33.333：

> ff d8 ff 04 14 00 00 00 ...

貌似是JPEG，但JPEG的文件头不是FFD8FFE0么？改成jpg也打不开。

此时注意到，后几位是04 14 00 00 00 08 00 ...，似乎符合ZIP的文件构造？在下方寻找，也找到了ZIP的下一个magic number 504B0102。修改成ZIP的文件头 50 4b 03 04，顺利解压，得到文件22222：

> 50 4b 03 04 1a 07 00 cf

故技重施，1A0700CF很明显是RAR格式了。修改成RAR的文件头52 61 72 21，解压得到flag：

> suctf{233333th1s1sf1ag23333333333333333}

讲道理可能确实有一些脑洞吧，然而搞取证的对文件头实在是再熟悉不过了。

### GAME
和电脑玩取石子游戏，而且玩遍了Bash/Nimm/Wythoff三种花样。

解法参考[文档](https://blog.csdn.net/ojshilu/article/details/16812173)，详细解释了这三种取石子游戏的模式和解法，这里简单做个整理：

#### 先手胜利的条件:非奇异局势
引入奇异局势的概念：当面对局势a时，A先手无法获胜，则称局势a是奇异局势。显然石子数为0的局势也是奇异局势。

可以证明，所有的非奇异局势都可以经过一次操作达到奇异局势。那么A若想先手获胜，唯一的条件是当前局势不是奇异局势，并通过操作使对方面临奇异局势。

#### 取石子游戏的种类
##### Bash Game
有1堆含n个石子，两个人轮流从这堆物品中取物，规定每次至少取1个，最多取m个。取走最后石子的人获胜。

这里的奇异局势是使得石子数n符合\\(n \equiv 1 \pmod{m}\\)的情况。显然，游戏策略是取石子使石子数符合前式。

##### Nimm Game
有k堆各n个石子，两个人轮流从某一堆取任意多的物品，规定每次至少取一个，多者不限。取走最后石子的人获胜。

将k堆石子的数量取异或，若结果为0则为奇异局势。可以通过穷举法穷举出下一步策略：符合使xor结果为0，且所需取的数小于该堆石子总数的都是可行解。

##### Wythoff Game
有2堆各n个石子，两个人轮流从某一堆或同时从两堆中取同样多的物品，规定每次至少取1个，多者不限。取走最后石子的人获胜。

这里的奇异局势比较复杂。记石子堆分别为A，B，归纳法分析奇异局势的状况可以发现，石子堆A的数量符合黄金分割的增长规律。换言之，奇异局势数列符合如下通项公式：

$$\begin{aligned}
a_k &= \lfloor \frac{k \cdot (1 + \sqrt{5})}{2} \rfloor \\
b_k &= a_k + k
\end{aligned}$$

解法么。。。也是穷举，穷举达到奇异局势的方案，但是由于局面复杂多样，这里的解法要复杂一些。可以参考ACM题HDU 2177的解法。

#### 脚本
写了一夜大概。。。

```python
#!/usr/bin/env python
# -*- encoding: utf-8 -*-

from pwn import *
from hashlib import sha256
import re
import math

digits = re.compile(r'\d+')
wythoff_table_a = [ int(k*(1+math.sqrt(5.0))/2) for k in range(100000) ]
wythoff_table_b = [ k + wythoff_table_a[k] for k in range(100000) ]
wythoff_table = [wythoff_table_a, wythoff_table_b]
x = (1 + math.sqrt(5.0))/2.0;

def hashcollison(r, t):
    raw = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
    for i in raw:
        for j in raw:
            for k in raw:
                for l in raw:
                    p = "%s%s%s%s%s" % (t, i, j, k, l)
                    # print sha256(p).hexdigest()
                    # print r
                    if sha256(p).hexdigest().strip() == r.strip():
                        return "%s%s%s%s" % (i, j, k, l)
                        
def gottable(x):
    ret = []
    r = 0
    for i in range(40):
        ret.append(r)
        r += x
    return ret

def swap(p, q):
p = p ^ q
q = q ^ p
p = p ^ q
return p, q
    
if __name__ == '__main__':
    con = remote('game.suctf.asuri.org', 10000)
    con.recvuntil('sha256(')
    r = con.recvline()
    hash = r.split('==')[1].strip()
    text = r[:12]
    log.info("Collision Hash (%s+xxxx)==%s" % (text, hash))
    con.sendline(hashcollison(hash, text))
    log.info("Collision Done. Enter the game")

    con.recvuntil('skip.')
    # rnd = 0
    log.info("Bash Game Start")
    while True: # Bash Game
        # rnd += 1
        con.recvuntil("===========================================================================\n")
        r = con.recv(1)
        if r != "R":
            break
        con.recvuntil('ound ')
        rnd = con.recvline(keepends=False)
        log.info("Round %s Start" % rnd)
        p = con.recvline(keepends=False)
        v = digits.findall(p)
        stonecnt = int(v[0])
        maxgot = int(v[2])
        p = gottable(maxgot+1)
        log.info("Round %s: %d Total, Got %d max" % (rnd, stonecnt, maxgot))

        while True:
            r = con.recvline()
            stonecnt = int(digits.findall(r)[0])
            # log.info("Round %s: %d Left" % (rnd, stonecnt))
            con.recvline()
            i = 0
            while p[i] <= stonecnt:
                i += 1;
            ans = stonecnt - p[i-1]
            
            if ans == 0:
                snt = 'GG'
            else:
                snt = str(ans)
            con.sendline(snt)
            r = con.recvline(keepends=False)
            if snt == "GG":
                c = digits.findall(r)[0]
                log.info("Round %s GG. %s chances left." % (rnd, c))
                break
            if r == "You win!":
                log.info("Round %s Win" % rnd)
                break
    log.info("Wythoff Game Start")

    while True: # Wythoff Game
        con.recvuntil("===========================================================================\n")
        r = con.recv(1)
        if r != "R":
            break
        con.recvuntil('ound ')
        rnd = con.recvline(keepends=False)
        log.info("Round %s Start" % rnd)
        while True:
            r = con.recvline()
            piles = [ int(i) for i in digits.findall(r) ]
            # log.info("Round %s: %d %d" % (rnd, piles[0], piles[1]))

            if piles[0] <= piles[1]:
                varorder = [0, 1]
            else:
                piles[0], piles[1] = swap(piles[0], piles[1])
                varorder = [1, 0]
            # log.info("Round %s: varorder %d %d" % (rnd, varorder[0], varorder[1]))
            con.recvline()
            
            snt = ""
            k = piles[1] - piles[0]
            if int(k * x) == piles[0]:
                snt = "GG"
            else:
                for i in range(1, piles[0]+1):
                    n = piles[0] - i
                    m = piles[1] - i
                    k = m - n
                    if int(k*x) == n:
                        snt = "%d 2" % i
                if snt == "":
                    for i in range(piles[1]):
                        n = piles[0]
                        m = i
                        if n > m:
                            n,m = swap(n,m)
                        k = m - n
                        if int(k*x) == n:
                            snt = "%d %d" % (piles[1] - i, varorder[1])
            # log.info("Payload: %s" % snt)
            con.sendline(snt)
            r = con.recvline(keepends=False)
            # log.info(r)
            if snt == "GG":
                c = digits.findall(r)[0]
                log.info("Round %s GG. %s chances left." % (rnd, c))
                break
            if r == "You win!":
                log.info("Round %s Win" % rnd)
                break
    
    log.info("Nimm Game Start")

    while True: # Nimm Game
        con.recvuntil("===========================================================================\n")
        r = con.recv(1)
        if r != "R":
            break
        con.recvuntil('ound ')
        rnd = con.recvline(keepends=False)
        log.info("Round %s Start" % rnd)
        while True:
            r = con.recvline()
            piles = [ int(i) for i in digits.findall(r) ]
            # log.info("Round %s: %d %d %d %d %d" % (rnd, piles[0], piles[1], piles[2], piles[3],piles[4]))
            
            xorret = reduce(lambda x,y: x ^ y, piles)
            
            snt = ""
            if xorret == 0:
                snt = "GG"
            else:
                for i in range(5):
                    middle_xor = xorret ^ piles[i]
                    needed = middle_xor ^ 0
                    # log.info("piles[%d] = %d, needed is %d" % (i, piles[i], needed))
                    if needed < piles[i]:
                        snt = "%d %d" % (piles[i] - needed, i)
                    
            con.sendline(snt)
            r = con.recvline(keepends=False)
            if snt == "GG":
                c = digits.findall(r)[0]
                log.info("Round %s GG. %s chances left." % (rnd, c))
                break
            if r == "You win!":
                log.info("Round %s Win" % rnd)
                break
        if rnd == '20':
            con.interactive()
```

> SUCTF{gGGGGggGgGggGGggGGGggGgGgggGGGGGggggggGgGggggGg}

## Web

求大佬讲解Getshell做法。

### Anonymous

Hitcon 2017某题的一半

```php
<?php

    $MY = create_function("","die(`cat flag.php`);");
    $hash = bin2hex(openssl_random_pseudo_bytes(32));
    eval("function SUCTF_$hash(){"
        ."global \$MY;"
        ."\$MY();"
        ."}");
    if(isset($_GET['func_name'])){
        $_GET["func_name"]();
        die();
    }
    show_source(__FILE__);
```

创建了匿名函数$MY来显示flag。显然不能直接爆破$hash，但由于PHP Zend Engine实际上是通过一个特殊的函数名(\x00_lambda_*N*, *N*是当前进程的匿名函数编号)来标记匿名函数，所以调用这个函数就可以实现功能了。

还有另外一点需要注意，为了使函数编号为0，我们需要大量给服务器发包，迫使Apache Fork一个新进程处理我们的请求：

```python
#!/usr/bin/env python
# coding: UTF-8
# Author: orange@chroot.org
# Modified to adapt SUCTF 2018

import requests
import socket
import time
from multiprocessing.dummy import Pool as ThreadPool
try:
    requests.packages.urllib3.disable_warnings()
except:
    pass

def run(i):
    while 1:
        HOST = 'web.suctf.asuri.org'
        PORT = 81
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((HOST, PORT))
        s.sendall('GET / HTTP/1.1\nHost: web.suctf.asuri.org\nConnection: Keep-Alive\n\n')
        # s.close()
        print 'ok'
        time.sleep(0.5)

i = 8
pool = ThreadPool( i )
result = pool.map_async( run, range(i) ).get(0xffff)
```

在这个脚本运行的同时提交payload即可。

> SUCTF{L4GsMqu6gu5knFnCi2Te8SjSucxKfQj6tuPJokoFhTCJjpa6RSfK}

## PWN & RE
别想了 我们队哪有人会搞这个？