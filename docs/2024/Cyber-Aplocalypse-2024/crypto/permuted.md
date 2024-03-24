# Permuted

# code / Description

```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from Crypto.Util.number import long_to_bytes

from hashlib import sha256
from random import shuffle

from secret import a, b, FLAG

class Permutation:
    def __init__(self, mapping):
        self.length = len(mapping)

        assert set(mapping) == set(range(self.length))     # ensure it contains all numbers from 0 to length-1, with no repetitions
        self.mapping = list(mapping)

    def __call__(self, *args, **kwargs):
        idx, *_ = args
        assert idx in range(self.length)
        return self.mapping[idx]

    def __mul__(self, other):
        ans = []

        for i in range(self.length):
            ans.append(self(other(i)))

        return Permutation(ans)

    def __pow__(self, power, modulo=None):
        ans = Permutation.identity(self.length)
        ctr = self

        while power > 0:
            if power % 2 == 1:
                ans *= ctr
            ctr *= ctr
            power //= 2

        return ans

    def __str__(self):
        return str(self.mapping)

    def identity(length):
        return Permutation(range(length))


x = list(range(50_000))
shuffle(x)

g = Permutation(x)
print('g =', g)

A = g**a
print('A =', A)
B = g**b
print('B =', B)

C = A**b
assert C.mapping == (B**a).mapping

sec = tuple(C.mapping)
sec = hash(sec)
sec = long_to_bytes(sec)

hash = sha256()
hash.update(sec)

key = hash.digest()[16:32]
iv = b"mg'g\xce\x08\xdbYN2\x89\xad\xedlY\xb9"

cipher = AES.new(key, AES.MODE_CBC, iv)

encrypted = cipher.encrypt(pad(FLAG, 16))
print('c =', encrypted)
```


# Challenge Analysis

this challenge is about groups, we have a group which its elements are different permutation of numbers from `0` to `n-1`. our groups is defined by a Class named `Permutation`.

In case of a groups we should have couple of elements 

1. Identity Element which is defined by a function named `identity` like this
```python
def identity(length):
    return Permutation(range(length))
```
it will create a permutation of numbers which are sorted `(0,1,2,...,n-1)`, so this would be our identity element

2. Operations

**Multiplication**

this operation is defined by this function

```python
def __mul__(self, other):
    ans = []

    for i in range(self.length):
        ans.append(self(other(i)))

    return Permutation(ans)
```

Imagine we have two elements like this

<center>
$g_1 = {4,2,3,1,0}$

$g_2 = {2,3,0,1,4}$
</center>

we want to multiply these two elements, multiplication operation is defined like this

<center>
$g_1 \times g_2 = g_1[g_2[i]] \quad for \quad i=0,1,2, \ldots ,n-1$
</center>

so the multiplication of $g_1 \times g_2$ will be

<center>
$g_1 \times g_2 = {3,1,4,2,0}$
</center>

**Exponentiation**

this operation is defined by this function

```python
def __pow__(self, power, modulo=None):
    ans = Permutation.identity(self.length)
    ctr = self

    while power > 0:
        if power % 2 == 1:
            ans *= ctr
        ctr *= ctr
        power //= 2

    return ans
```

exponentiation by power `n` is defined by multiplying an element `g` for `n` times with itself

<center>
$g^n = \underbrace{g \times g \times g \times g}_{n \text{ times}}$
</center>

there is an optimal algorithm for exponentiation which is done in $O(logn)$ instead of $O(n)$, in cases of large `n` this algorithm is better and feasable

```python
input : g, n
output : g^n

ctr = g
while power > 0:
    if power % 2 == 1:
        ans *= ctr
    ctr *= ctr
    power //= 2
return ctr
```

This was the Permutation group class. now let's see what is the exact problem

```python
x = list(range(50_000))
shuffle(x)

g = Permutation(x)
print('g =', g)

A = g**a
print('A =', A)
B = g**b
print('B =', B)

C = A**b
assert C.mapping == (B**a).mapping

sec = tuple(C.mapping)
sec = hash(sec)
sec = long_to_bytes(sec)

hash = sha256()
hash.update(sec)

key = hash.digest()[16:32]
iv = b"mg'g\xce\x08\xdbYN2\x89\xad\xedlY\xb9"

cipher = AES.new(key, AES.MODE_CBC, iv)

encrypted = cipher.encrypt(pad(FLAG, 16))
print('c =', encrypted)
```

1. the code creates a random permutation of numbers from 0 to 49999 named g
2. then it calculates power $a$ and $b$ of $g$ and name them $A,B$. The exponents $a,b$ are secret
3. then it will calculate a $C$ like $C = A^b$ or $C = B^a$, because of associative propery of this group $A^b = B^a = g^{ab}$
4. Finally it will create a hash for $C$ named secret and it will encrypt the flag with sha256 hash of this secret

So we have a DLP (Discrete Logarithm Problem) here. we have a generator `g` and two secrets `a,b` and the power `a,b` of the `g` which are `A,B`.
If we can find one of `a,b` values we can calculate `C` and the secret key and decrypt the flag.

# Solution

let's first see what is the order of our generator `g`. order of a generator in a group is lowest possible value $x$ such that g to the power of $x$ is equal to identity element

<center>
$g^x = e$
</center>

![../imgs/Screenshot 2024-03-16 at 4.08.55 PM.png](../imgs/Screenshot 2024-03-16 at 4.08.55 PM.png)

as we see the order of random generator is dependant on the elements of that generator. and computing order of a random `g` with this size (50000 elements!) is not feasable and requires a lot of trial/error and calculation. so what should we do now? brute force the `a,b` and iterate though all possible values to see when we can reach `A` or `B`. that's not a good idea because in case of a DLP problem `a,b` are secret and high enough to prevent brute forcing and that's the [trapdoor function](https://en.wikipedia.org/wiki/Trapdoor_function) of a DLP. becaue calculating the exponent is easy enough because of the algorithm we discussed in previous section but the reverse operation is not practical in feasable time.

When discussing with one of my good crypto player friends(I name him Ehsan Crypto :D) he mentioned we can calculate each elements period independantly to see after how many iteration each element resides its correct location. for example value 0 may be seen in first element section after $x_1$ loops ($g^{x_1}$). value 1 may be in its correct position after $x_2$ loops ($g^{x_2}$) and so on. so we can calculate each element's order seperately and see what's happening. let's see another example with 50 elements

![../imgs/Screenshot 2024-03-16 at 4.58.04 PM.png](../imgs/Screenshot 2024-03-16 at 4.58.04 PM.png)

the order of the `g` is 390. but the order of each element is either 39 or 10 and we know that $390 = 10 \times 39$. so the order of the whole `g` is LCM (Least Common Multiple) of each elements period. now we can find the order of whole `g`.

![../imgs/Screenshot 2024-03-16 at 5.41.40 PM.png](../imgs/Screenshot 2024-03-16 at 5.41.40 PM.png)

Now let's write a code to find order of generator `g`

```python
n = 50000

g = Permutation(g)
A = Permutation(A)
B = Permutation(B)

e = Permutation(list(range(50000)))
periods = {i:0 for i in range(len(g.mapping))}

i = 2
while not all(periods.values()):
    mul = g**i
    for j in range(len(g.mapping)):
        if (mul).mapping[j] == e.mapping[j] and periods[j] == 0:
            periods[j] = i
    i += 1

order = lcm(*list(periods.values()))
print(order)
```

and here is order of g:
```
3311019189498977856900
```

before we explain our solution, let's define these terms

$X_i$ is period or order of element $g[i]$ if:

<center>
$g^{X_i}[i] = e[i] \quad for \quad i=0,1,2,...,n-1$
</center>


$Y_i$ is distance of element $g[i]$ from $A[i]$ if:

<center>
$g^{X_i}[i] = A[i] \quad for \quad i=0,1,2,...,n-1$
</center>


tbh there is no need to calculate the order of `g` _lol_, but anyway. now we have the order and we should see how much distance each element of `g` has from its coresponding element in `A`. The reason that we wanna mesaure this distance is to find secret value `a` or `b`, but how these are related? we need to find just one of them so I pick `a`. we know that `a` and `b` are a large value(they should be) and each element of `g`'s period (the periods array we calculated in previous python code) are less than `a` or `b`. we have `n` equations like this

<center>
$a \equiv Y[i] \mod{X[i]} \quad for \quad i=0,1,2,...,n-1$
</center>

now we have 50000 modular equations. we can use CRT (chinese Remainder Theorem) to find $a$.

First we calculated all moduli and residue with this python code

```python
n = 50000

gg = Permutation(g)
AA = Permutation(A)
BB = Permutation(B)

e = Permutation(list(range(n)))
periods = {i:0 for i in range(len(gg.mapping))}
residues = {i:0 for i in range(len(gg.mapping))}


i = 2
while not all(periods.values()):
    mul = gg**i
    for j in range(len(gg.mapping)):
        if mul.mapping[j] == e.mapping[j] and periods[j] == 0:
            periods[j] = i
        if mul.mapping[j] == AA.mapping[j] and residues[j] == 0:
            residues[j] = i
    i += 1
```

then we use sage `crt` function to find the secret value `a`.

```python
a = crt(list(residues.values()), list(periods.values()))
print(a)
839949590738986464
```

now we have the secret value `a`, we can first verify if `a` is correct then easily calulate `C` and `secret` and findally decrpt the flag:

```python
a = 839949590738986464
C = B ** a
sec = tuple(C.mapping)
sec = hash(sec)
print(f"{sec=}")

sec = long_to_bytes(sec)

hash = sha256()
hash.update(sec)
key = hash.digest()[16:32]

print(f"key = {key.hex()}")
```

here is the AES key

```python
a0e58d9b8a93cc1b17e60110bb59cc2a
```

![../imgs/Screenshot 2024-03-24 at 4.03.50 PM.png](../imgs/Screenshot 2024-03-24 at 4.03.50 PM.png)

finally I used [cyberchef](https://gchq.github.io/CyberChef/#recipe=AES_Decrypt(%7B'option':'Hex','string':'a0e58d9b8a93cc1b17e60110bb59cc2a'%7D,%7B'option':'Hex','string':'6d672767ce08db594e3289aded6c59b9'%7D,'CBC','Hex','Raw',%7B'option':'Hex','string':''%7D,%7B'option':'Hex','string':''%7D)&input=ODliYTMxNGE5Y2ZkZThkMGU1NDEyYWEwMGQ3MTNmMjE3NzY3YjA4NWViY2U5ZjA2Y2I0Nzg0NGZlZGRiY2RjMjE4MzgwYzU0YTBhYTQ4MGM5ZTM5ZTc5ZDQwNTI5YmJk) to decrypt the flag

![Screenshot 2024-03-24 at 4.09.35 PM.png](../imgs/Screenshot 2024-03-24 at 4.09.35 PM.png)


# Final code
here is the final code (without g,A,B matrix because of large amount of data)

```python
from hashlib import sha256
from math import lcm
from sage.all import *
from Crypto.Util.number import *

class Permutation:
    def __init__(self, mapping):
        self.length = len(mapping)

        assert set(mapping) == set(range(self.length))     # ensure it contains all numbers from 0 to length-1, with no repetitions
        self.mapping = list(mapping)

    def __call__(self, *args, **kwargs):
        idx, *_ = args
        assert idx in range(self.length)
        return self.mapping[idx]

    def __mul__(self, other):
        ans = []

        for i in range(self.length):
            ans.append(self(other(i)))

        return Permutation(ans)

    def __pow__(self, power, modulo=None):
        ans = Permutation.identity(self.length)
        ctr = self

        while power > 0:
            if power % 2 == 1:
                ans *= ctr
            ctr *= ctr
            power //= 2

        return ans

    def __str__(self):
        return str(self.mapping)

    def identity(length):
        return Permutation(range(length))


n = 50000

gg = Permutation(g)
AA = Permutation(A)
BB = Permutation(B)

e = Permutation(list(range(n)))
periods = {i:0 for i in range(len(gg.mapping))}
residues = {i:0 for i in range(len(gg.mapping))}


i = 2
while not all(periods.values()):
    mul = gg**i
    for j in range(len(gg.mapping)):
        if mul.mapping[j] == e.mapping[j] and periods[j] == 0:
            periods[j] = i
        if mul.mapping[j] == AA.mapping[j] and residues[j] == 0:
            residues[j] = i
    i += 1

# order of g is not necessary
order = lcm(list(periods.values()))
print(f"order of g : {order}")

a = crt(list(residues.values()), list(periods.values()))
print(f"secret value a : {a}")

assert (gg**a).mapping==AA.mapping

C = BB ** a
sec = tuple(C.mapping)
sec = hash(sec)
print(f"sec : {sec}")

sec = long_to_bytes(sec)

key = sha256(sec).digest()[16:32]
print(f"aes key : {key.hex()}")
```

# Flag
Here is the flag

```
HTB{w3lL_n0T_aLl_gRoUpS_aRe_eQUaL_!!}
```

**AUTHOR**:
> pi3 / Pr1m3d Team