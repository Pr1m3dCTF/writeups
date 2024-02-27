# 1 - Challenge Code and Description

```
As you continue your journey, you must learn about the encryption method the aliens used to secure their communication from eavesdroppers.
The engineering team has designed a challenge that emulates the exact parameters of the aliens' encryption system, complete with instructions and a code snippet to connect to a mock alien server.
Your task is to break it.
```

```py
from Crypto.Util.number import getPrime, bytes_to_long

FLAG = b"HTB{???????????????}"
assert len(FLAG) == 20


class RSA:

    def __init__(self):
        self.q = getPrime(256)
        self.p = getPrime(256)
        self.n = self.q * self.p
        self.e = 3

    def encrypt(self, plaintext):
        plaintext = bytes_to_long(plaintext)
        return pow(plaintext, self.e, self.n)


def menu():
    print('[E]ncrypt the flag.')
    print('[A]bort training.\n')
    return input('> ').upper()[0]


def main():
    print('This is the second level of training.\n')
    while True:
        rsa = RSA()
        choice = menu()

        if choice == 'E':
            encrypted_flag = rsa.encrypt(FLAG)
            print(f'\nThe public key is:\n\nN: {rsa.n}\ne: {rsa.e}\n')
            print(f'The encrypted flag is: {encrypted_flag}\n')
        elif choice == 'A':
            print('\nGoodbye\n')
            exit(-1)
        else:
            print('\nInvalid choice!\n')
            exit(-1)


if __name__ == '__main__':
    main()
```
 # 2 - Solution
 Here we have a simple RSA encryption with small e (3) which is insecure.\
 because `c = m**3 % n` is smaller than `n` so `c == m ** 3` and we can compute m by getting 3rd root of `c`.
 
 ```py
 from Crypto.Util.number import *

def nth_root(x, n):
    # Start with some reasonable bounds around the nth root.
    upper_bound = 1
    while upper_bound ** n <= x:
        upper_bound *= 2
    lower_bound = upper_bound // 2
    # Keep searching for a better result as long as the bounds make sense.
    while lower_bound < upper_bound:
        mid = (lower_bound + upper_bound) // 2
        mid_nth = mid ** n
        if lower_bound < mid and mid_nth < x:
            lower_bound = mid
        elif upper_bound > mid and mid_nth > x:
            upper_bound = mid
        else:
            # Found perfect nth root.
            return mid
    return mid + 1

n = 4994987314155600304691453574807875996150564799718996509392679134987280603555645591343927213497932548816960938243148072674115512672389479749171011850599071
e = 3
c = 70407336670535933819674104208890254240063781538460394662998902860952366439176467447947737680952277637330523818962104685553250402512989897886053

m = nth_root(c, e)


flag = long_to_bytes(m)
print(flag)
 ```
 
 And here is the flag:
 ```
 HTB{5ma1l_E-xp0n3nt}
 ```
