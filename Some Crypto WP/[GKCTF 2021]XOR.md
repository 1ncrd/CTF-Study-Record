# [GKCTF 2021]XOR

主要参考：<https://zhuanlan.zhihu.com/p/384418167>  
主要是深度搜索的思想，第一组 n1, x1 给出了 n1 = a*b, x1 = a^b，从低位开始爆破，验证在模 $2^k$ 的情况下，每次的乘积和异或结果是否满足给出的等式。  
第二组稍微麻烦一点，给出的异或式子中的 d 是经过二进制倒序处理的，为了能够利用这个条件，需要对 c, d 从低位和高位同时开始爆破，而且要多加几个约束：

1. $c * d\ \equiv n_2\ mod\ 2^k$
2. $c\oplus reverse(d) \equiv x_2\ mod\ 2^k$ （即在爆破时对每次枚举高位和低位的值根据异或进行约束）
3. $c * d < n_2$ （因为高位也在同时进行爆破）
4. $c_nc_{n-1}...111...c_1c_0*d_nd_{n-1}...111...d_1d_0 > n_2$

decrypt

```py
from Crypto.Util.number import *
from tqdm import tqdm
import itertools as it
from hashlib import md5

n1 = 83876349443792695800858107026041183982320923732817788196403038436907852045968678032744364820591254653790102051548732974272946672219653204468640915315703578520430635535892870037920414827506578157530920987388471203455357776260856432484054297100045972527097719870947170053306375598308878558204734888246779716599
x1 = 4700741767515367755988979759237706359789790281090690245800324350837677624645184526110027943983952690246679445279368999008839183406301475579349891952257846
n2 = 65288148454377101841888871848806704694477906587010755286451216632701868457722848139696036928561888850717442616782583309975714172626476485483361217174514747468099567870640277441004322344671717444306055398513733053054597586090074921540794347615153542286893272415931709396262118416062887003290070001173035587341
x2 = 3604386688612320874143532262988384562213659798578583210892143261576908281112223356678900083870327527242238237513170367660043954376063004167228550592110478

# solve a , b

length = 512
a_list = [0]
b_list = [0]
mod = 2

for i in tqdm(range(length)):
    a_next, b_next = [], []
    for al, bl in zip(a_list, b_list):
        for ah, bh in ((0, 0), (0, 1), (1, 0), (1, 1)):
            aa, bb = ah * (mod // 2) + al, bh * (mod // 2) + bl
            if aa * bb % mod == n1 % mod and aa ^ bb % mod == x1 % mod:
                a_next.append(aa)
                b_next.append(bb)
    a_list, b_list = a_next, b_next
    mod *= 2

for aa, bb in zip(a_list, b_list):
    if aa * bb == n1:
        a, b = aa, bb
        print("a =", a)
        print("b =", b)
        break

# solve c, d

cl_list, ch_list, dl_list, dh_list = [0], [0], [0], [0]
mod = 2

for i in tqdm(range(length // 2)):
    cl_next, ch_next, dl_next, dh_next = [], [], [], []
    for cl, ch, dl, dh in zip(cl_list, ch_list, dl_list, dh_list):
        for cl0, ch0, dl0, dh0 in it.product((0, 1), repeat=4):
            ccl, ddl = cl0 * (mod // 2) + cl, dl0 * (mod // 2) + dl
            cch, ddh = ch0 * 2**(512 - 1 - i) + ch, dh0 * 2**(512 - 1 - i) + dh
            # 这里如果使用左移操作代替 * 2 ** xx，会出现报错
            # OverflowError: too many digits in integer
            # 推测是由于位移操作是对内部 int 直接操作，而乘法操作如果结果过长，python 会自动拓展位数
            # 也就是 python 中 int 没有上限的原理
            cc, dd = ccl + cch, ddl + ddh
            mid = int('1' * (512 - 2 * (i + 1)) + '0' * (i + 1), 2)
            dd_rev = int(bin(dd)[2:][::-1], 2)
            if (cc * dd % mod == n2 % mod) and (cl0 ^ dh0 == (x2 >> i) & 1) \
                and (ch0 ^ dl0 == (x2 >> (512 - 1 - i)) & 1) and (cc * dd <= n2) and (cc + mid) * (dd + mid) >= n2:
                cl_next.append(ccl)
                ch_next.append(cch)
                dl_next.append(ddl)
                dh_next.append(ddh)
    cl_list, ch_list, dl_list, dh_list = cl_next, ch_next, dl_next, dh_next
    mod *= 2
for cl, ch, dl, dh in zip(cl_list, ch_list, dl_list, dh_list):
    cc, dd = cl + ch, dl + dh
    if (cc * dd == n2):
        c, d = cc, dd
        print("c =", cc)
        print("d =", dd)
flag = "GKCTF{" + md5(str(a + b + c + d).encode()).hexdigest() + "}"
print("flag =", flag)
# flag = GKCTF{f28ed218415356b4336e2f778f2981bb}
```