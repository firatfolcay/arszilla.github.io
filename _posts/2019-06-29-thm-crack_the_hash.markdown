---
title: "TryHackMe: Crack The Hash - Writeup"
date: 2019-06-29 00:00:00
categories: [cybersec, infosec, tryhackme]
tags: [brute-force, hash cracking]
---

## Information
CrackTheHash is a box on TryHackMe that's all about hash identification and hash cracking. It consists of 9 unique 
hashes that may or may not be identified for us beforehand and need to be cracked with `hashcat` or similar tools. 
However, the real challenge in this box is knowing how to crack the hashes as some may take seconds to crack whereas 
some may take days.

## Tasks
- Crack the given hashes
 
## Summary
- Identify the given hashes with `hashid` if it's not specified
- Crack the hashes using `hashcat` using `rockyou.txt` and `best64.rule` if needed
 
## Tools Used
- `hashid`
- `hashcat`
- `awk`

## Walkthrough
### Disclaimers
- Pasted each hash into a `.txt` so `hashid` and `hashcat` can read the hashes without any issues
- Copied `rockyou.txt` from `/usr/share/wordlists` to my working directory to shorten the inputs needed
- Copied `best64.rule` from `/usr/share/hashcat/rules` to my working directory to shorten the inputs needed
- Alternatively, all of these hashes can be cracked via brute-forcing but that may require lots of kH/s; especially on 
Hash 1-4
- Special thanks to [Vetronexe][Vetronexe] for mentoring with hashes and hash cracking

### Hash 1-1
The first hash is `48bb6e862e54f2a795ffc4e541caed4d` and it's hinted to us that it's a `MD5` hash.

```
$ hashcat -m 0 -a 0 -d 1 --force hash.txt rockyou.txt

[...]

48bb6e862e54f2a795ffc4e541caed4d:easy

[...]
```

### Hash 1-2
The second hash is `CBFDAC6008F9CAB4083784CBD1874F76618D2A97` and there is a hint saying it's `SHA` but no mention of 
which `SHA` variant it is. To find that, we'll use `hashid`.

```
$ hashid hash.txt
--File 'hash.txt'--
Analyzing 'CBFDAC6008F9CAB4083784CBD1874F76618D2A97'
[+] SHA-1 
[+] Double SHA-1 
[+] RIPEMD-160 
[+] Haval-160 
[+] Tiger-160 
[+] HAS-160 
[+] LinkedIn 
[+] Skein-256(160) 
[+] Skein-512(160)
--End of file 'hash.txt'--#   
```

`hashid` identified our hash to be `SHA-1`.

```
$ hashcat -m 100 -a 0 -d 1 --force hash.txt rockyou.txt

[...]

cbfdac6008f9cab4083784cbd1874f76618d2a97:password123

[...]
```

### Hash 1-3
The third hash we're given is `1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032` and similar to Hash 
1-2 it's stated that its `SHA`, but nothing on which type it's. As a result, the same methodology from Hash 1-2 applies 
here as well.

```
$ hashid hash.txt
--File 'hash.txt'--
Analyzing '1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032'
[+] Snefru-256 
[+] SHA-256 
[+] RIPEMD-256 
[+] Haval-256 
[+] GOST R 34.11-94 
[+] GOST CryptoPro S-Box 
[+] SHA3-256 
[+] Skein-256 
[+] Skein-512(256)
--End of file 'hash.txt'--#
```

`hashid` identified our hash to be `SHA-256`.


```
$ hashcat -m 1400 -a 0 -d 1 --force hash.txt rockyou.txt

[...]

1c8bfe8f801d79745c4631d09fff36c82aa37fc4cce4fc946683d7b336b63032:letmein

[...]
```

### Hash 1-4
The fourth hash we're given is `$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom` and there's a hint 
telling us that it's `bcrypt`. If we attempted to crack this hash with `rockyou.txt`, it would take days to crack it. 
If we take a look at the input box for Hash 1-4 over at TryHackMe, we'll see that our flag should be 4 characters long. 
As a result, we'll only choose the passwords that are 4 in length in `rockyou.txt` and then pass that new wordlist 
instead of `rockyou.txt` to speed up the cracking process.

```
$ awk 'length < 5' rockyou.txt > custom_rockyou.txt
$ hashcat -m 3200 -a 0 -d 1 --force hash.txt custom_rockyou.txt

[...]

$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom:bleh

[...]
```

### Hash 1-5
Our fifth hash is `279412f945939ba78ce0758d3fd83daa` and the hint states that it's a `MD4`. If we were to run it with 
`rockyou.txt`, we'd see that it wasn't able to crack the hash. In this case, we'd have to use `-r` to specify rules to 
mutate our wordlist. One of the best rules to use is `best64.rule`.

```
$ hashcat -m 900 -a 0 -d 1 --force hash.txt rockyou.txt -r best64.rule

[...]

279412f945939ba78ce0758d3fd83daa:Eternity22

[...]
```

### Hash 2-1
The sixth hash we're given is `F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85` and there's no hint 
given to us. As a result we'll have to identify the hash first.

```
$ hashid hash.txt
--File 'hash.txt'--
Analyzing 'F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85'
[+] Snefru-256
[+] SHA-256
[+] RIPEMD-256
[+] Haval-256
[+] GOST R 34.11-94
[+] GOST CryptoPro S-Box
[+] SHA3-256
[+] Skein-256
[+] Skein-512(256)
--End of file 'hash.txt'--#   
```

Taking a look at the possibilities above, it's more likely to be `SHA-256` as it's more commonly used than the rest of 
the other hashes. Do note that `SHA-256` is listed as `SHA2-256` in `hashcat`.

```
$ hashcat -m 1400 -a 0 -d 1 --force hash.txt rockyou.txt
[...]

f09edcb1fcefc6dfb23dc3505a882655ff77375ed8aa2d1c13f640fccc2d0c85:paule

[...]
```

### Hash 2-2
The seventh hash is `1DFECA0C002AE40B8619ECF94819CC1B` and the hint states that it's `NTLM`.

```
$ hashcat -m 1000 -a 0 -d 1 --force hash.txt rockyou.txt
[...]

1dfeca0c002ae40b8619ecf94819cc1b:n63umy8lkf4i    

[...]
```

### Hash 2-3
Our eighth hash is `$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.` 
and there's no indication on what this hash might be. As a result we'll use `hashid` to identify this hash.

```
$ hashid hash.txt
--File 'hash.txt'--
Analyzing '$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.'
[+] SHA-512 Crypt
--End of file 'hash.txt'--#
```

`hashid` identified our hash to be `SHA-512 Crypt`.

```
$ awk 'length < 7' rockyou.txt > custom_rockyou.txt
$ hashcat -m 1800 -a 0 -d 1 --force hash.txt custom_rockyou.txt
[...]

$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.:waka99
                                                 
[...]
```

### Hash 2-4
Our ninth and final hash is `e5d8870e5bdd26602cab8dbe07a942c8669e56d6` and it's hinted to us that its `HMAC-SHA1`.

```
$ hashcat -m 160 -a 0 -d 1 --force e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme /usr/share/wordlists/rockyou.txt
[...]

e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme:481616481616
                                                 
[...]
```

[Vetronexe]: https://twitter.com/Vetronexe