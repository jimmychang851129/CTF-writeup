# NahamCon CTF

## PWN

### shifts-ahoy

ROP + BOF(one line ROP)

**Conclusion: only one ROP instruction is allowed, construct system("/bin/sh") and for the only ROP instrcution, jump to the system("/bin/sh") we've inputted.**

The program contains a encrypt functio and a decrypt function.
The decrypt function is totally useless, since there is only a line of puts in the function.
There is a BOF vulnerability in encryption. The size of the char array is 72. However, we are able to write 96 char

```lang=c
int encrypt()
{
  char s[72]; // [rsp+0h] [rbp-50h]
  int i; // [rsp+48h] [rbp-8h]
  int v3; // [rsp+4Ch] [rbp-4h]

  printf("Enter the message: ");
  fgets(s, 96, stdin);
  v3 = strlen(s);
  if ( v3 > 64 )
    v3 = 64;
  for ( i = 0; i < v3; ++i )
    s[i] += 13;
  return printf("\nEncrypted Message: %s\n", s);
}
```
However, we are only able to write 8 bytes of ROP payload. For he 96 bytes we are able to write, we can modify the return address but we can't write further to the following address. Therefore, we are only able to construct a single ROP instruuction.

We can find out that in the encrypt function, there is actually a `MOV R15, RSP` right before the `ret` instruction, which means R15 will contain the data we've written.

Since the NX and PIE is disabled, we can simply write a system("/bin/sh") shellcode and try to jump to the address. What we're going to do is construct the following payload.

```
system("/bin/sh") + 'a'*n + address of jmp_r15
```

address of jmp_r15

```
jmp_r15 = list(elf.search(asm('jmp r15')))[0]
```

Due to ASLR, we cannot simply set ROP to 

```lang=c
ret <addr of system("/bin/sh")>
```

In this case, we can try to jump to the address of system("/bin/sh"), which we've just written.
jmp_r15 as we said, appears in the encrypt function. In the ROP, we place the address of jmp_r15 so we could jump to the system("/bin/sh") part and gain the reverse shell.

As for the system("/bin/sh"), since the encrypt function will have some operations on the input strings, so we need to reverse its operation on our input strings before really input our string.


## Misc(Bash Jail)

### Fake File

Find and its arguments(exec)

```
$ find . -type f  -exec cat {} \;

or 

$ find . -type f -exec cat {} +;

```


login to the server, and you will find a file named: "..", which is the same as the directory "..".

Our objective is to read the flag in the file "..". However if 

#### Reference

[writeup](https://github.com/skyf0l/CTF/blob/master/NahamCon_2020/Miscellaneous.md#vortex)
[find command on stackoverflow](https://unix.stackexchange.com/questions/292253/how-to-use-cat-command-on-find-commands-output)

### Alkatraz

payload:

```
$ `<flag.txt` 

or 

$ . flag.txt  # the same as source flag.txt(read and execute file)
```

We are not allowed to cat, less, more, head, tail the flag.txt
Therefore, we tried to source the file(read and execute the file).

source command normally find the file defined in the $PATH variable, if no file is found, then it turns to current directory to find the file.

#### Reference

[payload1 writeup](https://bigpick.github.io/TodayILearned/articles/2020-06/nahamConCTF-writeups#alkatraz)
[payload2 writeup](https://github.com/skyf0l/CTF/blob/master/NahamCon_2020/Miscellaneous.md#vortex)
[Bash challenge](http://blog.dornea.nu/2016/06/20/ringzer0-ctf-jail-escaping-bash/)
[linux dot command intro](https://unix.stackexchange.com/questions/114300/whats-the-meaning-of-a-dot-before-a-command-in-shell)
[linux source command](https://www.geeksforgeeks.org/source-command-in-linux-with-examples/)
[linux run command explanation](https://unix.stackexchange.com/questions/114300/whats-the-meaning-of-a-dot-before-a-command-in-shell)