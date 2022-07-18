---
title: "crypto問でよく使うライブラリ，関数など"
date: 2022-05-09T01:34:02+09:00
tags: ["ctf","crypto"]
draft: false
---


## 目的
しょっちゅう忘れてはググってるのでまとめます．

## Python標準機能
### 文字コード変換


### 基数変換


## Crypto.Util.number

### long_to_bytes,bytes_to_long
bytes型とlong型を変換する．
```python
>>> bytes_to_long(b'hello')
448378203247
>>> long_to_bytes(0x68656c6c6f)
b'hello'
```

### inverse
`inverse(a,b)`で`a`をmod`b`での逆元を得られる．
```python
>>> inverse(3,7)
5
>>> 3*inverse(3,7)%7
1
```
なお，デフォルトで使える`pow`関数で代用可能．
```python
>>> pow(3,-1,7)
5
```

### getPrime
`getPrime(a)`で`a`ビットの素数が生成される
```python
>>> getPrime(10)
911
>>> bin(911)
'0b1110001111'
>>>
```


## Crypto.PublicKey
pemファイルを開くことができる．
openssl rsa -text -in idk.key

使用例
```python
from Crypto.PublicKey import RSA
f=open('key.pem','rb')
pubkey = RSA.importKey(f.read())
print(pubkey)
print("n:",pubkey.n)
print("e:",pubkey.e)
```

## gmpy2

### iroot
`iroot(a,b)`で`a`の`b`乗根が得られる．タプル型で返る．

## sagemath

### 

