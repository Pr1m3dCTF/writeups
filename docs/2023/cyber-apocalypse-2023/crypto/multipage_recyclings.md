# 1 - Challenge code and Description
```
As your investigation progressed, a clue led you to a local bar where you met an undercover agent with valuable information.
He spoke of a famous astronomy scientist who lived in the area and extensively studied the relic.
The scientist wrote a book containing valuable insights on the relic's location, but encrypted it before he disappeared to keep it safe from malicious intent.
The old man disclosed that the book was hidden in the scientist's house and revealed two phrases that the scientist rambled about before vanishing.
```

**source.py**
```py
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
import random, os

FLAG = b'HTB{??????????????????????}'


class CAES:

    def __init__(self):
        self.key = os.urandom(16)
        self.cipher = AES.new(self.key, AES.MODE_ECB)

    def blockify(self, message, size):
        return [message[i:i + size] for i in range(0, len(message), size)]

    def xor(self, a, b):
        return b''.join([bytes([_a ^ _b]) for _a, _b in zip(a, b)])

    def encrypt(self, message):
        iv = os.urandom(16)

        ciphertext = b''
        plaintext = iv

        blocks = self.blockify(message, 16)
        for block in blocks:
            ct = self.cipher.encrypt(plaintext)
            encrypted_block = self.xor(block, ct)
            ciphertext += encrypted_block
            plaintext = encrypted_block

        return ciphertext

    def leak(self, blocks):
        r = random.randint(0, len(blocks) - 2)
        leak = [self.cipher.encrypt(blocks[i]).hex() for i in [r, r + 1]]
        return r, leak


def main():
    aes = CAES()
    message = pad(FLAG * 4, 16)

    ciphertext = aes.encrypt(message)
    ciphertext_blocks = aes.blockify(ciphertext, 16)

    r, leak = aes.leak(ciphertext_blocks)

    with open('output.txt', 'w') as f:
        f.write(f'ct = {ciphertext.hex()}\nr = {r}\nphrases = {leak}\n')


if __name__ == "__main__":
    main()
```

**output.txt**
```
ct = bc9bc77a809b7f618522d36ef7765e1cad359eef39f0eaa5dc5d85f3ab249e788c9bc36e11d72eee281d1a645027bd96a363c0e24efc6b5caa552b2df4979a5ad41e405576d415a5272ba730e27c593eb2c725031a52b7aa92df4c4e26f116c631630b5d23f11775804a688e5e4d5624
r = 3
phrases = ['8b6973611d8b62941043f85cd1483244', 'cf8f71416111f1e8cdee791151c222ad']
```

# 2 - Solution

Here we have a AES-CFB mode encryption and the output is like above and our message is actually the repition of the flag 4 times\
We also have some leaks which is encryption of encrypted message according to aes-cfb decryption process we should encrypt previous cipher text and xor it with current ciphertext to recover the message\
Here in leaks we have blocksp[3] and blocks[4] encrypton so if we xor blocks[4] with leaks[0] and blocks[5] with leaks[1] we should get two decrypted blocks of the message

```
d41e405576d415a5272ba730e27c593e ^ 8b6973611d8b62941043f85cd1483244 = _w34k_w17h_l34kz
b2c725031a52b7aa92df4c4e26f116c6 ^ cf8f71416111f1e8cdee791151c222ad = }HTB{CFB_15_w34k
```

So here is the flag:
```
HTB{CFB_15_w34k_w17h_l34kz}
```
