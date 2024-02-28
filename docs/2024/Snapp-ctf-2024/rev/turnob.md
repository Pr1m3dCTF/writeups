# TurnOB

## Description

```
TurnOB, the elegant binary, covertly converts the input message to hexadecimal and discreetly prints the result, preserving its secrecy.
```

## Solution

After downloading, I checked the files and there are two files:

- turnob
- flag.enc

In many CTFs, this scheme points to an encrypt/decrypt operation. There are many ways to reverse these challenges. First I decide to interact with the program itself except disassembly algorithms. For the first attempt, I decided to create a dictionary of printable letters (ASCII numbers) and the program output:

```python
mapping = {
    'a2:a7': '1', 'af:a7': '2', 'bc:a7': '3', 'c9:a7': '4', 'd6:a7': '5', 'e3:a7': '6', 'f0:a7': '7',
    'fd:a7': '8', '0a:a7': '9', '95:a7': '0', '12:a7': 'a', '1f:a7': 'b', '2c:a7': 'c', '39:a7': 'd',
    '46:a7': 'e', '53:a7': 'f', '60:a7': 'g', '6d:a7': 'h', '7a:a7': 'i', '87:a7': 'j', '94:a7': 'k',
    'a1:a7': 'l', 'ae:a7': 'm', 'bb:a7': 'n', 'c8:a7': 'o', 'd5:a7': 'p', 'e2:a7': 'q', 'ef:a7': 'r',
    'fc:a7': 's', '09:a7': 't', '16:a7': 'u', '23:a7': 'v', '30:a7': 'w', '3d:a7': 'x', '4a:a7': 'y',
    '57:a7': 'z', '72:a7': 'A', '7f:a7': 'B', '8c:a7': 'C', '99:a7': 'D', 'a6:a7': 'E', 'b3:a7': 'F',
    'c0:a7': 'G', 'cd:a7': 'H', 'da:a7': 'I', 'e7:a7': 'J', 'f4:a7': 'K', '01:a7': 'L', '0e:a7': 'M',
    '1b:a7': 'N', '28:a7': 'O', '35:a7': 'P', '42:a7': 'Q', '4f:a7': 'R', '5c:a7': 'S', '69:a7': 'T',
    '76:a7': 'U', '83:a7': 'V', '90:a7': 'W', '9d:a7': 'X', 'aa:a7': 'Y', 'b7:a7': 'Z', 'f8:a7': '_',
    '6e:a7': '-', '64:a7': '{', '7e:a7': '}', 'd2:a7': '!'
}
```

Then I tried to match them with the `flag.enc` content. Also for each key, there is a suffix of `a7` I decide to just add it to my mapping keys. The final code:

```python

mapping = {
    'a2:a7': '1', 'af:a7': '2', 'bc:a7': '3', 'c9:a7': '4', 'd6:a7': '5', 'e3:a7': '6', 'f0:a7': '7',
    'fd:a7': '8', '0a:a7': '9', '95:a7': '0', '12:a7': 'a', '1f:a7': 'b', '2c:a7': 'c', '39:a7': 'd',
    '46:a7': 'e', '53:a7': 'f', '60:a7': 'g', '6d:a7': 'h', '7a:a7': 'i', '87:a7': 'j', '94:a7': 'k',
    'a1:a7': 'l', 'ae:a7': 'm', 'bb:a7': 'n', 'c8:a7': 'o', 'd5:a7': 'p', 'e2:a7': 'q', 'ef:a7': 'r',
    'fc:a7': 's', '09:a7': 't', '16:a7': 'u', '23:a7': 'v', '30:a7': 'w', '3d:a7': 'x', '4a:a7': 'y',
    '57:a7': 'z', '72:a7': 'A', '7f:a7': 'B', '8c:a7': 'C', '99:a7': 'D', 'a6:a7': 'E', 'b3:a7': 'F',
    'c0:a7': 'G', 'cd:a7': 'H', 'da:a7': 'I', 'e7:a7': 'J', 'f4:a7': 'K', '01:a7': 'L', '0e:a7': 'M',
    '1b:a7': 'N', '28:a7': 'O', '35:a7': 'P', '42:a7': 'Q', '4f:a7': 'R', '5c:a7': 'S', '69:a7': 'T',
    '76:a7': 'U', '83:a7': 'V', '90:a7': 'W', '9d:a7': 'X', 'aa:a7': 'Y', 'b7:a7': 'Z', 'f8:a7': '_',
    '6e:a7': '-', '64:a7': '{', '7e:a7': '}', 'd2:a7': '!'
}

# The encoded string
provided_string = "5c:1b:72:35:35:64:5c:6d:95:ef:69:a6:d6:09:f8:6d:bc:9d:f8:99:16:0e:d5:f8:16:f0:7a:a1:d2:09:aa:f8:a2:bb:f8:8c:d2:7e:a7"

# Split the provided string into substrings of length 4
substrings = provided_string.split(":")
decoded_string = ''.join(mapping.get(sub + ":a7", "*") for sub in substrings)
print(decoded_string)
```

The result:

`SNAPP{Sh0rTE5t_h3X_DuMp_u7il!tY_1n_C!}`

---