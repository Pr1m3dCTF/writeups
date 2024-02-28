## Snapp Admin

````
I'm in a hurry and I don't have enough money. Find my [**discount**](https://snappctf.com/tasks/snapp_admin_d2cb1554b1532e3735e4c6f31d8cc396b356083c.txz) code.

`nc 91.107.177.236 1337`
````
## Solution

At first I ran the program to analyse how does it work, after that realizing that the first input is not vulnerable to overflow, I searched the program for the password that it was asking and reading the assembly code of the program I figured that with  correct password ( `9606` ) there is a way to a `gets` function which allows overflow ( 56 length padding ) to jump inside the `is_admin` function however there is a condition which can be bypassed easily by just passing the address after the condition which shows the flag.

payload:
`b'9606\naaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaa'+p64(0x00401307)`

![IMG/Pasted image 20240225192228.png](./IMG/Pasted image 20240225192228.png)