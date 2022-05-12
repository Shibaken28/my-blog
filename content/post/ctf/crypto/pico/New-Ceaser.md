---
title: "picoGym - New Ceaser"
date: 2022-05-12T11:31:06+09:00
tags: ["ctf","crypto","writeup","ceaser","warmup"]
draft: false
---

## 概要
We found a brand new type of encryption, can you break the secret code? (Wrap with picoCTF{}) 
```none
mlnklfnknljflfmhjimkmhjhmljhjomhmmjkjpmmjmjkjpjojgjmjpjojojnjojmmkmlmijimhjmmj
```
```python
import string

LOWERCASE_OFFSET = ord("a")
ALPHABET = string.ascii_lowercase[:16]

def b16_encode(plain):
	enc = ""
	for c in plain:
		binary = "{0:08b}".format(ord(c))
		enc += ALPHABET[int(binary[:4], 2)]
		enc += ALPHABET[int(binary[4:], 2)]
	return enc

def shift(c, k):
	t1 = ord(c) - LOWERCASE_OFFSET
	t2 = ord(k) - LOWERCASE_OFFSET
	return ALPHABET[(t1 + t2) % len(ALPHABET)]

flag = "redacted"
key = "redacted"
assert all([k in ALPHABET for k in key])
assert len(key) == 1

b16 = b16_encode(flag)
enc = ""
for i, c in enumerate(b16):
	enc += shift(c, key[i % len(key)])
print(enc)

```

## 解答
フラグは，b16_encode，shiftの順に，2段階でエンコードされているので，逆順にデコードしていく．
shiftに関しては，`assert`の条件により，せいぜい16通りである．
```python
import string

LOWERCASE_OFFSET = ord("a")
ALPHABET = string.ascii_lowercase[:16]

def b16_decode(cipher):
       plain = ""
       i = 0
       while i<len(cipher):
              plain += chr((ord(cipher[i])-LOWERCASE_OFFSET)<<4)+ord(cipher[i+1])-LOWERCASE_OFFSET)
              i+=2
       return plain

def shift_rev(c, k):
	t1 = ord(c) - LOWERCASE_OFFSET
	t2 = ord(k) - LOWERCASE_OFFSET
	return ALPHABET[(t1 - t2 + len(ALPHABET)) % len(ALPHABET)]

enc = "mlnklfnknljflfmhjimkmhjhmljhjomhmmjkjpmmjmjkjpjojgjmjpjojojnjojmmkmlmijimhjmmj"

for k in ALPHABET:
       current_enc = ""
       for c in enc:
              current_enc+=shift_rev(c,k)
       print(k,":",b16_decode(current_enc))

```

結果であるが，フラグっぽい文字列が見つからない．
```
a : ËÚµÚÛµÇÇËÌÊËÈÉ
b : ºÉ¤ÉÊ
         ¤¶¹¶º¶»»
¹º·¶¸
c : ©¸¸¹s¥v¨¥u©u|¥ªx}ªzx}|tz}||{|z¨©¦v¥z§
d : §¨bedgliglkcilkkjkiei
e : qQqTSSZV[XV[ZRX[ZZYZX
                         TX

f : v
`
@`rCurBvBIrwEJwGEJIAGJIIHIGuvsCrGt
g : et_tu?_a2da1e18af49f649806988786deb2a6c
h : TcNcd.NP!SP T 'PU#(U%#('/%(''&'%STQ!P%R
i : CR=RS\x1dO\x10O\x1f\x1fOD\x12D\x14\x17\x1e\x17\x16\x16BC@\x10\x14
\x03\x05\x04\x032?\x0f\x03
k : !0\x1b01û\x1b-þ -ý!ýô-"ðõ"òðõôüòõôôóôò !.þ-ò/
l : \x10
/ ê
\x1c\xad\x1fì\x10\xacã\x1cïä\x11\xa1ïäãëáäããâãá\x1f\x1d\xad\x1c\xa1\x1e

\x0e\x88èúËýúÊþÊÁúÿÍÂÿÏÍÂÁÉÏÂÁÁÀÁÏýþûËúÏü
o : íü×üý·×éºìé¹í¹°éî¼±î¾¼±°¸¾±°°¿°¾ìíêºé¾ë
p : ÜëÆëì¦ÆØ©ÛØ¨Ü¨¯ØÝ« Ý­« ¯§­ ¯¯®¯­ÛÜÙ©Ø­Ú
```
実装を間違えてことを疑うが，検証してみるとそうでもなさそう．
実は，`picoCTF{et_tu?_a2da1e18af49f649806988786deb2a6c}`がフラグである．
実際に，`key`を指定して実行すると`mlnklfnknl(以下略)`が出力される．

## 余談
picoGymにはgoodとbadの評価ができるボタンがありますが，こういうフラグが間違いに見える問題の評価はあまり高くないですね．
