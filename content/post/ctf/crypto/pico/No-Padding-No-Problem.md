---
title: "picoGym - No Padding No Problem"
date: 2022-05-12T15:12:25+09:00
tags: ["ctf","crypto","writeup","RSA","warmup"]
draft: true
---


## 概要
Oracles can be your best friend, they will decrypt anything, except the flag's ciphertext. How will you break it? Connect with `nc mercury.picoctf.net 42248`.
`nc`すると，`n`と`e`と`c`のみが渡され，任意の暗号文を何度も解読してくれる．

## 解答
渡された`c`自身を暗号文渡すと`Will not decrypt the ciphertext. Try Again`と怒られてしまう．しかし，これは`c+n`を投げることで簡単に回避ができる．次の式が成り立つからである．
$(c+n)^d = c^d + (nが係数についている項たち) = c^d = m \mod n$
```none
picoCTF{m4yb3_Th0se_m3s54g3s_4r3_difurrent_7416022}
```
