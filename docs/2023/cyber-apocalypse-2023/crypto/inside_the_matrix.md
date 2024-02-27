# 1 - Challenge code and Description


```
As you deciphered the Matrix, you discovered that the astronomy scientist had observed that certain stars were not real.
He had created two 5x5 matrices with values based on the time the stars were bright, but after some time, the stars stopped emitting light.
Nonetheless, he had managed to capture every matrix until then and created an algorithm that simulated their generation.
However, he could not understand what was hidden behind them as he was missing something.
He believed that if he could understand the stars, he would be able to locate the secret tombs where the relic was hidden.
```

**source.py**:
```py
from sage.all_cmdline import *
# from utils import ascii_print
import os

FLAG = b"HTB{????????????????????}"
assert len(FLAG) == 25


class Book:

    def __init__(self):
        self.size = 5
        self.prime = None

    def parse(self, pt: bytes):
        pt = [b for b in pt]
        return matrix(GF(self.prime), self.size, self.size, pt)

    def generate(self):
        key = os.urandom(self.size**2)
        return self.parse(key)

    def rotate(self):
        self.prime = random_prime(2**6, False, 2**4)

    def encrypt(self, message: bytes):
        self.rotate()
        key = self.generate()
        message = self.parse(message)
        ciphertext = message * key
        return ciphertext, key


def menu():
    print("Options:\n")
    print("[L]ook at page")
    print("[T]urn page")
    print("[C]heat\n")
    option = input("> ")
    return option


def main():
    book = Book()
    ciphertext, key = book.encrypt(FLAG)
    page_number = 1

    while True:
        option = menu()
        if option == "L":
            # ascii_print(ciphertext, key, page_number)
            print(ciphertext, key, page_number)
        elif option == "T":
            ciphertext, key = book.encrypt(FLAG)
            page_number += 2
            print()
        elif option == "C":
            print(f"\n{list(ciphertext)}\n{list(key)}\n")
        else:
            print("\nInvalid option!\n")


if __name__ == "__main__":
    try:
        main()
    except Exception as e:
        print(f"An error occurred: {e}")
```

Let's explore the code first
1. It generates a random prime number from 16 to 64 (17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61) named `P`
2. Then it generates two matrix in finite field of P first is the flag characters named `M` second one is the key named `K`
3. Then it multiply two matrix and generates a ciphertext matrix named `C` `C = M * K`

Also we have a remote instance which gives us three options:\

1. L which shows current encrypted text `C` and the key `K`
2. T generates new `K` and `C` for flag matrix `M`
3. C which show `C & K` in list format

# 2 - Solution
here we have an equation like this:

```
C = M * K % P
```

We can solve this equation with sage like this and find M
```py
R = IntegerModRing(59)
C = Matrix(R, [[20, 22, 9, 10, 55],[5, 49, 30, 31, 28],[17, 22, 23, 31, 41],[30, 19, 31, 8, 21],[10, 44, 48, 32, 22]])
K = Matrix(R, [[23, 22, 54, 16, 53],[19, 58, 25, 10, 33],[44, 11, 34, 14, 28],[8, 56, 15, 21, 45],[15, 26, 13, 26, 9]])
M = C.solve_left(K)

print(M)
```

But how to find `P`. we can simply bruteforce it or we can use T option for remote instance couple of times to get an element of for example 58 or 60 to ensure that the P is 59 or 61
I generated several ciphertexts and for one I assumed the P is 59 so I generated the matrix `M` with above equations with value `P=59` and here is the matrix M:

```
[13 25  7  5 49]
[48 48 48 36  5]
[57 36 55 45 51]
[36 56 57 52 55]
[56 33 33 33  7]
```

We can confirm the `M` and prime `P=59` with:
```
ord('H') = 72 % 59 == 13
ord('T') = 84 % 59 == 25
ord('B') = 66 % 59 == 7
ord('{') = 123 % 59 == 5
```

After that with trial and error to find exact value of the flag characters and here is the code I used

```py
message = [13,25,7,5,49,48,48,48,36,5,57,36,55,45,51,36,56,57,52,55,56,33,33,33,7]
message = [13+p,25+p,7+p,5+2*p,49+p,48,48,48+p,36+p,5+p,57+p,36+p,55,45+p,51,36+p,56+p,57+p,52,55+p,56+p,33,33,33,7+2*p]

for m in message:
    print(chr(m), end="") 

print()
```

And here os the flag
```
HTB{l00k_@t_7h3_st4rs!!!}
```

According to this amazing [writeup](https://chovid99.github.io/posts/cyber-apocalypse-2023-crypto/#inside-the-matrix) You can also grab several M matrix with different `P` values and use Chinese Remainder Theorem to find exact value of flags without trial and error
