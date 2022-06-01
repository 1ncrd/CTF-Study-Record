# [MRCTF 2020]easy_RSA

task

```py
import sympy
from gmpy2 import gcd, invert
from random import randint
from Crypto.Util.number import getPrime, isPrime, getRandomNBitInteger, bytes_to_long, long_to_bytes
import base64

from zlib import *
flag = b"MRCTF{XXXX}"
base = 65537

def gen_prime(N):
    A = 0
    while 1:
        A = getPrime(N)
        if A % 8 == 5:
            break
    return A

def gen_p():
    p = getPrime(1024)
    q = getPrime(1024)
    assert (p < q)
    n = p * q
    print("P_n = ", n)
    F_n = (p - 1) * (q - 1)
    print("P_F_n = ", F_n)
    factor2 = 2021 * p + 2020 * q
    if factor2 < 0:
        factor2 = (-1) * factor2
    return sympy.nextprime(factor2)


def gen_q():
    p = getPrime(1024)
    q = getPrime(1024)
    assert (p < q)
    n = p * q
    print("Q_n = ", n)
    e = getRandomNBitInteger(53)
    F_n = (p - 1) * (q - 1)
    while gcd(e, F_n) != 1:
        e = getRandomNBitInteger(53)
    d = invert(e, F_n)
    print("Q_E_D = ", e * d)
    factor2 = 2021 * p - 2020 * q
    if factor2 < 0:
        factor2 = (-1) * factor2
    return sympy.nextprime(factor2)


if __name__ == "__main__":
    _E = base
    _P = gen_p()
    _Q = gen_q()
    assert (gcd(_E, (_P - 1) * (_Q - 1)) == 1)
    _M = bytes_to_long(flag)
    _C = pow(_M, _E, _P * _Q)
    print("Ciphertext = ", _C)
'''
P_n =  14057332139537395701238463644827948204030576528558543283405966933509944444681257521108769303999679955371474546213196051386802936343092965202519504111238572269823072199039812208100301939365080328518578704076769147484922508482686658959347725753762078590928561862163337382463252361958145933210306431342748775024336556028267742021320891681762543660468484018686865891073110757394154024833552558863671537491089957038648328973790692356014778420333896705595252711514117478072828880198506187667924020260600124717243067420876363980538994101929437978668709128652587073901337310278665778299513763593234951137512120572797739181693
P_F_n =  14057332139537395701238463644827948204030576528558543283405966933509944444681257521108769303999679955371474546213196051386802936343092965202519504111238572269823072199039812208100301939365080328518578704076769147484922508482686658959347725753762078590928561862163337382463252361958145933210306431342748775024099427363967321110127562039879018616082926935567951378185280882426903064598376668106616694623540074057210432790309571018778281723710994930151635857933293394780142192586806292968028305922173313521186946635709194350912242693822450297748434301924950358561859804256788098033426537956252964976682327991427626735740
Q_n =  20714298338160449749545360743688018842877274054540852096459485283936802341271363766157976112525034004319938054034934880860956966585051684483662535780621673316774842614701726445870630109196016676725183412879870463432277629916669130494040403733295593655306104176367902352484367520262917943100467697540593925707162162616635533550262718808746254599456286578409187895171015796991910123804529825519519278388910483133813330902530160448972926096083990208243274548561238253002789474920730760001104048093295680593033327818821255300893423412192265814418546134015557579236219461780344469127987669565138930308525189944897421753947
Q_E_D =  100772079222298134586116156850742817855408127716962891929259868746672572602333918958075582671752493618259518286336122772703330183037221105058298653490794337885098499073583821832532798309513538383175233429533467348390389323225198805294950484802068148590902907221150968539067980432831310376368202773212266320112670699737501054831646286585142281419237572222713975646843555024731855688573834108711874406149540078253774349708158063055754932812675786123700768288048445326199880983717504538825498103789304873682191053050366806825802602658674268440844577955499368404019114913934477160428428662847012289516655310680119638600315228284298935201
Ciphertext =  40855937355228438525361161524441274634175356845950884889338630813182607485910094677909779126550263304194796000904384775495000943424070396334435810126536165332565417336797036611773382728344687175253081047586602838685027428292621557914514629024324794275772522013126464926990620140406412999485728750385876868115091735425577555027394033416643032644774339644654011686716639760512353355719065795222201167219831780961308225780478482467294410828543488412258764446494815238766185728454416691898859462532083437213793104823759147317613637881419787581920745151430394526712790608442960106537539121880514269830696341737507717448946962021
'''
```

求解 _P 只需要求解一个方程组就行  
$$\begin{cases} [Pn] = p * q\\ [PFn] = (p - 1) * (q - 1)\\  \end{cases} $$  

```py
import gmpy2
# 已知 n, phi 分解 n = p * q
def getPQfromNandPhi(n, phi):
    x = n
    y = phi
    p = (x - y + 1 + gmpy2.iroot((x - y + 1)**2 - 4 * x, 2)[0]) // 2
    q = (x - y + 1 - gmpy2.iroot((x - y + 1)**2 - 4 * x, 2)[0]) // 2
    return p, q
```

求解 \_Q，问题即为 已知 e*d, n, 分解 n，找到如下代码  

```py
def getpq(n, ed):
    while True:
        k = ed - 1
        g = randint(0, n)
        while k % 2 == 0:
            k = k // 2
            temp = gmpy2.powmod(g, k, n)-1
            if gmpy2.gcd(temp, n) > 1 and temp != 0:
                return gmpy2.gcd(temp, n)
```

可以求解 _Q，然后又发现了另一种求法，  
有 $ed \equiv 1\ mod\ phi$  
$\Rightarrow ed = k * phi + 1$  
因为 phi 和 n 很接近，可以用整除得到 k  
$k = ed // n$  
$\therefore phi = (ed - 1) / k$  
即得到 phi 就和上一步一样的求法了。

```py
# 已知 N, ed, 得到 phi
def getPhiFromNandE_times_D(N, ed):
    k = ((ed - 1) // N) + 1
    phi = (ed - 1) // k
    return phi
```

因此 exp 中求 _Q 的方法可以更换成这个。  
最后注意语句 `assert (p < q)`，顺序一定要对。
exp

```py
import gmpy2
import sympy
from random import randint
from Crypto.Util.number import getPrime, isPrime, getRandomNBitInteger, bytes_to_long, long_to_bytes, inverse
import base64

from zlib import *

P_n = 14057332139537395701238463644827948204030576528558543283405966933509944444681257521108769303999679955371474546213196051386802936343092965202519504111238572269823072199039812208100301939365080328518578704076769147484922508482686658959347725753762078590928561862163337382463252361958145933210306431342748775024336556028267742021320891681762543660468484018686865891073110757394154024833552558863671537491089957038648328973790692356014778420333896705595252711514117478072828880198506187667924020260600124717243067420876363980538994101929437978668709128652587073901337310278665778299513763593234951137512120572797739181693
P_F_n = 14057332139537395701238463644827948204030576528558543283405966933509944444681257521108769303999679955371474546213196051386802936343092965202519504111238572269823072199039812208100301939365080328518578704076769147484922508482686658959347725753762078590928561862163337382463252361958145933210306431342748775024099427363967321110127562039879018616082926935567951378185280882426903064598376668106616694623540074057210432790309571018778281723710994930151635857933293394780142192586806292968028305922173313521186946635709194350912242693822450297748434301924950358561859804256788098033426537956252964976682327991427626735740
Q_n = 20714298338160449749545360743688018842877274054540852096459485283936802341271363766157976112525034004319938054034934880860956966585051684483662535780621673316774842614701726445870630109196016676725183412879870463432277629916669130494040403733295593655306104176367902352484367520262917943100467697540593925707162162616635533550262718808746254599456286578409187895171015796991910123804529825519519278388910483133813330902530160448972926096083990208243274548561238253002789474920730760001104048093295680593033327818821255300893423412192265814418546134015557579236219461780344469127987669565138930308525189944897421753947
Q_E_D = 100772079222298134586116156850742817855408127716962891929259868746672572602333918958075582671752493618259518286336122772703330183037221105058298653490794337885098499073583821832532798309513538383175233429533467348390389323225198805294950484802068148590902907221150968539067980432831310376368202773212266320112670699737501054831646286585142281419237572222713975646843555024731855688573834108711874406149540078253774349708158063055754932812675786123700768288048445326199880983717504538825498103789304873682191053050366806825802602658674268440844577955499368404019114913934477160428428662847012289516655310680119638600315228284298935201
Ciphertext = 40855937355228438525361161524441274634175356845950884889338630813182607485910094677909779126550263304194796000904384775495000943424070396334435810126536165332565417336797036611773382728344687175253081047586602838685027428292621557914514629024324794275772522013126464926990620140406412999485728750385876868115091735425577555027394033416643032644774339644654011686716639760512353355719065795222201167219831780961308225780478482467294410828543488412258764446494815238766185728454416691898859462532083437213793104823759147317613637881419787581920745151430394526712790608442960106537539121880514269830696341737507717448946962021
_E = 65537

def getPQfromNandPhi(n, phi):
    x = n
    y = phi
    p = (x - y + 1 + gmpy2.iroot((x - y + 1)**2 - 4 * x, 2)[0]) // 2
    q = (x - y + 1 - gmpy2.iroot((x - y + 1)**2 - 4 * x, 2)[0]) // 2
    return p, q

p, q = getPQfromNandPhi(P_n, P_F_n)
assert p * q == P_n
if p > q:
    p, q = q, p
factor2 = 2021 * p + 2020 * q
_P = sympy.nextprime(factor2)

def getpq(n, ed):
    while True:
        k = ed - 1
        g = randint(0, n)
        while k % 2 == 0:
            k = k // 2
            temp = gmpy2.powmod(g, k, n)-1
            if gmpy2.gcd(temp, n) > 1 and temp != 0:
                return gmpy2.gcd(temp, n)

p = getpq(Q_n, Q_E_D)
q = Q_n // p
assert p * q == Q_n
if p > q:
    p, q = q, p
factor2 = 2021 * p - 2020 * q
if factor2 < 0:
    factor2 = (-1) * factor2
_Q = sympy.nextprime(factor2)
_N = _P * _Q
_PHI = (_P - 1) * (_Q - 1)
_D = inverse(_E, _PHI)
print(long_to_bytes(int(pow(Ciphertext, _D, _N))))
```