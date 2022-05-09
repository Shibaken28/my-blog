---
title: "CrewCTF 2022 - The Huge E"
date: 2022-05-09T12:59:30+09:00
tags: ["ctf","crypto","writeup","RSA","smooth"]
draft: false
---

## 問題
[ctfTimeの問題ページ](https://ctftime.org/task/20470)
`out.txt`は次のようになっています．
```python
p = 127557933868274766492781168166651795645253551106939814103375361345423596703884421796150924794852741931334746816404778765897684777811408386179315837751682393250322682273488477810275794941270780027115435485813413822503016999058941190903932883823
e1 = 219560036291700924162367491740680392841
e2 = 325829142086458078752836113369745585569
e3 = 237262361171684477270779152881433264701
c = 962976093858853504877937799237367527464560456536071770645193845048591657714868645727169308285896910567283470660044952959089092802768837038911347652160892917850466319249036343642773207046774240176141525105555149800395040339351956120433647613
```
平文を$m$，ある素数を$p$とすると，$m^e \bmod p = c$として暗号文$c$を求めています．ただし，$e = {e_1}^{{e_2}^{e_3}}$です．
$e_1,e_2,e_3$が与えられているので，$e$を求めれば
$d = e^{-1} \pmod{\phi(p)} $として$c^d \pmod p$で平文が求まります．

しかし，$e$を実際に計算しようとすると無限時間かかります．
どう計算するかが問題です．

## 解答
$e$の値は$\bmod {\phi(p)}$の世界で考えても問題ありません．
${e_1}^{{e_2}^{e_3}} \pmod{\phi(p)}$の値を求めていきます．
オイラーの定理より，次が成り立ちます．
$${e_1}^{\phi(\phi(p))} = 1 \pmod{\phi(p)} $$
$p$は素数であるので，$\phi(p)=p-1$です(フェルマーの小定理)．よって，次のように書き換えられます．
$${e_1}^{\phi(p-1)} = 1 \pmod{p-1}$$

整数$k$と$l$を使って，${e_2}^{e_3} = \phi(p-1)k+l$と表されたとします．これは，${e_2}^{e_3}$を$\phi(p-1)$で割ったときに商が$k$，余りが$l$になったことを表しています．
すると，
$${e_1}^{{e_2}^{e_3}} = {e_1}^{ \phi(p-1)k+l} = ({e_1}^{\phi(p-1)})^k\cdot {e_1}^l = 1^k \cdot {e_1}^l = {e_1}^l \pmod{p-1}$$

と計算できます．つまり，$l={e_2}^{e_3} \pmod{\phi(p-1)}$として，${e_1}^l \pmod{p-1}$を計算すれば$e$が求まります．

$\phi(p-1)$を計算するには，$p-1$を素因数分解しなければなりませんが，$p-1$は小さい素因数で構成(smooth呼ばれる性質です)されているため，愚直なプログラムで素因数が求まります．

以下，pythonでのプログラムを示します．
なお，sagemathには`euler_phi`関数が用意されています．

```python
from Crypto.Util.number import long_to_bytes

def euler_phi(N):
    factor = []
    i = 2
    # 素因数分解
    while True: 
        if i*i>N:
            factor.append(N)
            break
        if N%i==0:
            factor.append(i)
            N = N // i
        else:
            i+=1
    
    # 素因数-1を掛け算していく
    phi = 1
    for p in factor:
        phi *= p-1
    
    return phi

p = 127557933868274766492781168166651795645253551106939814103375361345423596703884421796150924794852741931334746816404778765897684777811408386179315837751682393250322682273488477810275794941270780027115435485813413822503016999058941190903932883823
e1 = 219560036291700924162367491740680392841
e2 = 325829142086458078752836113369745585569
e3 = 237262361171684477270779152881433264701
c = 962976093858853504877937799237367527464560456536071770645193845048591657714868645727169308285896910567283470660044952959089092802768837038911347652160892917850466319249036343642773207046774240176141525105555149800395040339351956120433647613

phi_p = p - 1
phi_phi_p = euler_phi(phi_p)

e = pow(e1,pow(e2,e3,phi_phi_p), phi_p)
d = pow(e,-1,phi_p)
m = pow(c,d,p)

print("phi(p)     : ",phi_p)
print("phi(phi(p)): ",phi_phi_p)
print(long_to_bytes(m))

```

結果
```none
phi(p)     :  127557933868274766492781168166651795645253551106939814103375361345423596703884421796150924794852741931334746816404778765897684777811408386179315837751682393250322682273488477810275794941270780027115435485813413822503016999058941190903932883822
phi(phi(p)):  63775594668498404422995279661693309334486076035802944116275814269950078792958445557761589097717204934857369990271713664698474867142217580223510594284968730411939236198524531363514002763605853593498040656788050786948899096447734618521600000000
b'crew{7hi5_1s_4_5ma11er_numb3r_7han_7h3_Gr4ham_numb3r}'
```

