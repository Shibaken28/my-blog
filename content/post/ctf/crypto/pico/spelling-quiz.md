---
title: "picoGym - Spelling Quiz"
date: 2022-05-12T00:46:00+09:00
draft: true
---

## 概要
I found the flag, but my brother wrote a program to encrypt all his text files. He has a spelling quiz study guide too, but I don't know if that helps.
flag.txt
```
brcfxba_vfr_mid_hosbrm_iprc_exa_hoav_vwcrm
```
encrypto.py
```python
import random
import os

files = [
    os.path.join(path, file)
    for path, dirs, files in os.walk('.')
    for file in files
    if file.split('.')[-1] == 'txt'
]

alphabet = list('abcdefghijklmnopqrstuvwxyz')
random.shuffle(shuffled := alphabet[:])
dictionary = dict(zip(alphabet, shuffled))

for filename in files:
    text = open(filename, 'r').read()
    encrypted = ''.join([
        dictionary[c]
        if c in dictionary else c
        for c in text
    ])
    open(filename, 'w').write(encrypted)

```

study-guide.txt
```
gocnfwnwtr
sxlyrxaic
dcrrtfrxcv
uxbvwavcq
lwvicwtiwm
pwtmwnxvicq
avingciisa
ylwtmrcawx
mwaxdcrrxuwlwvq
yciflwnf
mwaxsrtwvq
iovabxcabqwtd
(以下略)
```
encrypto.pyは，そのディレクトリの全てのファイルに対して，各アルファベットをそれぞれ異なるアルファベットに対応させて，換え字式暗号で暗号化している．
アルファベット26文字に対して，$26!\simeq 10^{26}$通りの組み合わせがあるので，全探索はできない．
そこで，flag.txtと一緒に暗号化されたstudy-guide.txtを参考に復元をする．

## 解答
頻度分析(Frequency analysis)である．study-guide.txtの中身が元々意味のある英単語であった場合，`e`に対応する文字が一番多い，といった推測ができる．

```python
alphabet = "abcdefghijklmnopqrstuvwxyz"
dict = {}
for a in alphabet:
       dict[ord(a)]=0

with open("./study-guide.txt","r") as f:
       for c in f.read():
              if c in alphabet:
                     dict[ord(c)] += 1

fre = []
for a,c in dict.items():
       fre.append((c,chr(a)))

fre.sort(reverse=True)

print(fre)
```

```python
[(311363, 'r'), (270080, 'w'), (239284, 'x'), (216936, 't'), (214772, 'i'), (206355, 'a'), (205401, 'c'), (198197, 'v'), (162351, 'l'), (131465, 'n'), (107082, 'o'), (96529, 'b'), (90628, 'm'), (87009, 's'), (76513, 'f'), (66435, 'd'), (57699, 'q'), (49432, 'u'), (30493, 'y'), (27458, 'p'), (17173, 'g'), (14940, 'e'), (11862, 'k'), (8354, 'z'), (4794, 'j'), (3251, 'h')]
```
`r`が最も多く，`e`に対応する文字だと推測できる．

さて，ここまで記事を書いてきたが，調べたところ，この類の暗号の解読をしてくれる[quipquip](https://www.quipqiup.com/)というサイトがあるらしい．

ということで，`flag.txt`と`study-guide.txt`(全部は長いので，一部)を投げると，フラグが得られる．
```none
picoCTF{perhaps_the_dog_jumped_over_was_just_tired}
```