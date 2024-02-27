# 1 - Challenge code and Description

```
The final stage of your initialization sequence is mastering cutting-edge technology tools that can be life-changing.
One of these tools is quipqiup, an automated tool for frequency analysis and breaking substitution ciphers.
This is the ultimate challenge, simulating the use of AES encryption to protect a message. Can you break it?
```

**source.py**
```py
from os import urandom
from Crypto.Cipher import AES
from secret import MESSAGE

assert all([x.isupper() or x in '{_} ' for x in MESSAGE])


class Cipher:

    def __init__(self):
        self.salt = urandom(15)
        key = urandom(16)
        self.cipher = AES.new(key, AES.MODE_ECB)

    def encrypt(self, message):
        return [self.cipher.encrypt(c.encode() + self.salt) for c in message]


def main():
    cipher = Cipher()
    encrypted = cipher.encrypt(MESSAGE)
    encrypted = "\n".join([c.hex() for c in encrypted])

    with open("output.txt", 'w+') as f:
        f.write(encrypted)


if __name__ == "__main__":
    main()
```

**output.txt**
```
dfc8a2232dc2487a5455bda9fa2d45a1
305d4649e3cb097fb094f8f45abbf0dc
c87a7eb9283e59571ad0cb0c89a74379
60e8373bfb2124aea832f87809fca596
d178fac67ec4e9d2724fed6c7b50cd26
c87a7eb9283e59571ad0cb0c89a74379
34ece5ff054feccc5dabe9ae90438f9d
457165130940ceac01160ac0ff924d86
5d7185a6823ab4fc73f3ea33669a7bae
61331054d82aeec9a20416759766d9d5
5f122076e17398b7e21d1762a61e2e0a
....................
```


# 2 - Solution
We know that characters of the encrypted message is Upper letters and '{_} ' which is for teh flag\
So the message is a text message containing the flag inside it with all upper letter characters.\
Each character of the message is being encrypted with AES ECB with same key.\
So each character is mapped to its encrypted version using AES-ECB and we can do statistical analysis.\
But first we should map these characters to single character random letters except '{_}' we also need space characters to perform statistical analysis
How to solve this?
Those AES outputs which has only one occurance are '{}'. so `fbe86a428051747607a35b44b1a3e9e9` is actually `{` and `c53ba24fbbe9e3dbdd6062b3aab7ed1a` is `}`
We can guess `_` is only between `{}` inside flag and not other part of the message so those AES outputs which are between `fbe86a428051747607a35b44b1a3e9e9` and `c53ba24fbbe9e3dbdd6062b3aab7ed1a` is actually `_`
For space the characrter before the flag first character is space and that is `61331054d82aeec9a20416759766d9d5`.

and we should map remaining patterns to a random letter. Here is final script

**solve.py**
```py
import json

datas = open('./crypto_perfect_synchronization/output.txt', 'r').readlines()
raw = open('./crypto_perfect_synchronization/output.txt', 'r').read().strip()
data = []

for d in datas:
    data.append(d.strip())

stats = {}
for d in data:
    if d in stats.keys():
        stats[d] += 1
    else:
        stats[d] = 1

letters = set(data)

chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ '
print(len(chars))

raw = raw.replace('fbe86a428051747607a35b44b1a3e9e9', '{')
raw = raw.replace('a94f49727cf771a85831bd03af1caaf5', '_')
raw = raw.replace('c53ba24fbbe9e3dbdd6062b3aab7ed1a', '}')
raw = raw.replace('61331054d82aeec9a20416759766d9d5', ' ')

index = 0
output = ''
for d in letters:
    if d in['fbe86a428051747607a35b44b1a3e9e9','a94f49727cf771a85831bd03af1caaf5','c53ba24fbbe9e3dbdd6062b3aab7ed1a']:
        continue
    raw = raw.replace(d, chars[index]).strip()
    index += 1

print(raw.replace('\n', ''))
```

result:
```
LKBCVBRMP XRXZPNDN DN SXNBE OR TYB LXMT TYXT DR XRP UDFBR NTKBTMY OL IKDTTBR ZXRUVXUB MBKTXDR ZBTTBKN XRE MOQSDRXTDORN OL ZBTTBKN OMMVK IDTY FXKPDRU LKBCVBRMDBN QOKBOFBK TYBKB DN X MYXKXMTBKDNTDM EDNTKDSVTDOR OL ZBTTBKN TYXT DN KOVUYZP TYB NXQB LOK XZQONT XZZ NXQGZBN OL TYXT ZXRUVXUB DR MKPGTXRXZPNDN LKBCVBRMP XRXZPNDN XZNO HROIR XN MOVRTDRU ZBTTBKN DN TYB NTVEP OL TYB LKBCVBRMP OL ZBTTBKN OK UKOVGN OL ZBTTBKN DR X MDGYBKTBJT TYB QBTYOE DN VNBE XN XR XDE TO SKBXHDRU MZXNNDMXZ MDGYBKN LKBCVBRMP XRXZPNDN KBCVDKBN ORZP X SXNDM VREBKNTXREDRU OL TYB NTXTDNTDMN OL TYB GZXDRTBJT ZXRUVXUB XRE NOQB GKOSZBQ NOZFDRU NHDZZN XRE DL GBKLOKQBE SP YXRE TOZBKXRMB LOK BJTBRNDFB ZBTTBK SOOHHBBGDRU EVKDRU IOKZE IXK DD SOTY TYB SKDTDNY XRE TYB XQBKDMXRN KBMKVDTBE MOEBSKBXHBKN SP GZXMDRU MKONNIOKE GVWWZBN DR QXAOK RBINGXGBKN XRE KVRRDRU MORTBNTN LOK IYO MOVZE NOZFB TYBQ TYB LXNTBNT NBFBKXZ OL TYB MDGYBKN VNBE SP TYB XJDN GOIBKN IBKB SKBXHXSZB VNDRU LKBCVBRMP XRXZPNDN LOK BJXQGZB NOQB OL TYB MORNVZXK MDGYBKN VNBE SP TYB AXGXRBNB QBMYXRDMXZ QBTYOEN OL ZBTTBK MOVRTDRU XRE NTXTDNTDMXZ XRXZPNDN UBRBKXZZP YTS{X_NDQGZB_NVSNTDTVTDOR_DN_IBXH} MXKE TPGB QXMYDRBKP IBKB LDKNT VNBE DR IOKZE IXK DD GONNDSZP SP TYB VN XKQPN NDN TOEXP TYB YXKE IOKH OL ZBTTBK MOVRTDRU XRE XRXZPNDN YXN SBBR KBGZXMBE SP MOQGVTBK NOLTIXKB IYDMY MXR MXKKP OVT NVMY XRXZPNDN DR NBMOREN IDTY QOEBKR MOQGVTDRU GOIBK MZXNNDMXZ MDGYBKN XKB VRZDHBZP TO GKOFDEB XRP KBXZ GKOTBMTDOR LOK MORLDEBRTDXZ EXTX GVWWZB GVWWZB GVWWZB
```

Now we can use [quipqiup](https://quipqiup.com/) to find the original text from output abow and here is the original text:
```
	FREQUENCY ANALYSIS IS BASED ON THE FACT THAT IN ANY GIVEN STRETCH OF WRITTEN LANGUAGE CERTAIN LETTERS AND COMBINATIONS OF LETTERS OCCUR WITH VARYING FREQUENCIES MOREOVER THERE IS A CHARACTERISTIC DISTRIBUTION OF LETTERS THAT IS ROUGHLY THE SAME FOR ALMOST ALL SAMPLES OF THAT LANGUAGE IN CRYPTANALYSIS FREQUENCY ANALYSIS ALSO KNOWN AS COUNTING LETTERS IS THE STUDY OF THE FREQUENCY OF LETTERS OR GROUPS OF LETTERS IN A CIPHERTEXT THE METHOD IS USED AS AN AID TO BREAKING CLASSICAL CIPHERS FREQUENCY ANALYSIS REQUIRES ONLY A BASIC UNDERSTANDING OF THE STATISTICS OF THE PLAINTEXT LANGUAGE AND SOME PROBLEM SOLVING SKILLS AND IF PERFORMED BY HAND TOLERANCE FOR EXTENSIVE LETTER BOOKKEEPING DURING WORLD WAR II BOTH THE BRITISH AND THE AMERICANS RECRUITED CODEBREAKERS BY PLACING CROSSWORD PUZZLES IN MAJOR NEWSPAPERS AND RUNNING CONTESTS FOR WHO COULD SOLVE THEM THE FASTEST SEVERAL OF THE CIPHERS USED BY THE AXIS POWERS WERE BREAKABLE USING FREQUENCY ANALYSIS FOR EXAMPLE SOME OF THE CONSULAR CIPHERS USED BY THE JAPANESE MECHANICAL METHODS OF LETTER COUNTING AND STATISTICAL ANALYSIS GENERALLY HTB{A_SIMPLE_SUBSTITUTION_IS_WEAK} CARD TYPE MACHINERY WERE FIRST USED IN WORLD WAR II POSSIBLY BY THE US ARMYS SIS TODAY THE HARD WORK OF LETTER COUNTING AND ANALYSIS HAS BEEN REPLACED BY COMPUTER SOFTWARE WHICH CAN CARRY OUT SUCH ANALYSIS IN SECONDS WITH MODERN COMPUTING POWER CLASSICAL CIPHERS ARE UNLIKELY TO PROVIDE ANY REAL PROTECTION FOR CONFIDENTIAL DATA PUZZLE PUZZLE PUZZLE
```

And here is the flag:
```
HTB{A_SIMPLE_SUBSTITUTION_IS_WEAK}
```

