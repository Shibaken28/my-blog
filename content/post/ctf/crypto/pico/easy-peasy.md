---
title: "picoGym - Easy Peasy"
date: 2022-05-12T10:46:41+09:00
tags: ["ctf","crypto","writeup","xor","warmup"]
draft: false
---


## 概要
A one-time pad is unbreakable, but can you manage to recover the flag? (Wrap with picoCTF{}) `nc mercury.picoctf.net 64260 otp.py`

```python
#!/usr/bin/python3 -u
import os.path

KEY_FILE = "key"
KEY_LEN = 50000
FLAG_FILE = "flag"


def startup(key_location):
	flag = open(FLAG_FILE).read()
	kf = open(KEY_FILE, "rb").read()

	start = key_location
	stop = key_location + len(flag)

	key = kf[start:stop]
	key_location = stop

	result = list(map(lambda p, k: "{:02x}".format(ord(p) ^ k), flag, key))
	print("This is the encrypted flag!\n{}\n".format("".join(result)))

	return key_location

def encrypt(key_location):
	ui = input("What data would you like to encrypt? ").rstrip()
	if len(ui) == 0 or len(ui) > KEY_LEN:
		return -1

	start = key_location
	stop = key_location + len(ui)

	kf = open(KEY_FILE, "rb").read()

	if stop >= KEY_LEN:
		stop = stop % KEY_LEN
		key = kf[start:] + kf[:stop]
	else:
		key = kf[start:stop]
	key_location = stop

	result = list(map(lambda p, k: "{:02x}".format(ord(p) ^ k), ui, key))

	print("Here ya go!\n{}\n".format("".join(result)))

	return key_location


print("******************Welcome to our OTP implementation!******************")
c = startup(0)
while c >= 0:
	c = encrypt(c)

```


## 解答
裏で何が行われているかを見る．
長さ50000の`key`が用意されていて，`key_location`が0にセットされている．
入力があると，`key`の`key_location`文字目から，その入力の文字数文だけが取り出され，XORされる．そして，`key_location`はその文字数分だけ動く．

xorの性質より，50000文字の入力を与えて，出力を得れば，keyがわかる．
特に，文字コード0`\x00`を50000文字連ねれば，そのままkeyが出力される．

次のスクリプトでは，50000-(フラグ長)の分だけ`key_location`を動かして，次の入力でちょうどフラグとxorされた値が出力されるようにしている．

```python
from pwn import *

io = remote("mercury.picoctf.net",64260)

S = io.recvuntil("?").decode()
print("[+] ",S)

KEY_LEN = 50000
FLAG_LEN = len("51466d4e5f575538195551416e4f5300413f1b5008684d5504384157046e4959")//2
out = b"\x00" * (KEY_LEN-FLAG_LEN)
io.sendline(out)
io.recvuntil("?").decode()

out = b"\x00" * FLAG_LEN
io.sendline(out)
S = io.recvuntil("?").decode()
print("[+] ",S)
```


```none
[+] Opening connection to mercury.picoctf.net on port 64260: Done
solve.py:8: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  S = io.recvuntil("?").decode()
[+]  ******************Welcome to our OTP implementation!******************
This is the encrypted flag!
51466d4e5f575538195551416e4f5300413f1b5008684d5504384157046e4959

What data would you like to encrypt?
solve.py:15: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  io.recvuntil("?").decode()
solve.py:19: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  S = io.recvuntil("?").decode()
[+]   Here ya go!
62275c786663615c783165725c786237225c7863315c7831375c7861305c7838
```
これにより，次の関係がわかる．
```none
key        = 62275c786663615c783165725c786237225c7863315c7831375c7861305c7838
flag ^ key = 51466d4e5f575538195551416e4f5300413f1b5008684d5504384157046e4959
```

```python
>>> a = 0x51466d4e5f575538195551416e4f5300413f1b5008684d5504384157046e4959 ^ 0x62275c786663615c783165725c786237225c7863315c7831375c7861305c7838
>>> from Crypto.Util.number import long_to_bytes
>>> long_to_bytes(a)
b'3a16944dad432717ccc3945d3d96421a'
```
数字の羅列が出てきて，失敗したように見えるが，実はこれがそのまま`picoCTF{3a16944dad432717ccc3945d3d96421a}`とフラグになる．

## 感想
間違いだと勘違いするので，フラグの内容は意味のある文字列にしてほしいです！
