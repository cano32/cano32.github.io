---
layout: default
ctf: SINCON 2024
chal: 1-7
category: misc
---

# SINCON 2024

Main CTF page: [https://hackerware.io/sincon2024](https://hackerware.io/sincon2024)<br />

![img](./2024-SINCON/sincon2024-start.png)
- [1](#1)
- [2](#2)
- [3](#3)
- [4](#4)
- [5](#5)
- [6](#6)
- [7](#7)
- [8](#8)<br /><br />

## 1

> And that's the way...   6330 306b 3133 2063 7275 6d62 6c33 73

<br />
Use Cyberchef:
![img](./2024-SINCON/sincon2024-1-1.png)<br /><br />

![img](./2024-SINCON/sincon2024-1-2.png)<br /><br />


Flag: ```c00k13 crumbl3s```<br /><br />

## 2

> Any cookie crumbs on the lanyard?

<br />
![img](./2024-SINCON/sincon2024-2-1.jpg)<br /><br />

Looking at the symbol ciphers on [dcode](https://www.dcode.fr/symbols-ciphers), the symbols on the lanyard seem like pigpen cipher.<br /><br />
![img](./2024-SINCON/sincon2024-2-2.png)<br /><br />

Decode it with the tool [dcode pigpen cipher](https://www.dcode.fr/pigpen-cipher).
![img](./2024-SINCON/sincon2024-2-3.png)<br /><br />

![img](./2024-SINCON/sincon2024-2-4.png)<br /><br />

Flag: ```sincookie```<br />
(Uppercase letters didn't work)<br /><br />

## 3

> Do you really talk logic?
> 
> aGFja2Vyd2FyZS5pby9jb29rMWVsb2cxYw==

<br />
Use Cyberchef:
![img](./2024-SINCON/sincon2024-3-1.png)<br /><br />

In the website [https://hackerware.io/cook1elog1c](https://hackerware.io/cook1elog1c):
![img](./2024-SINCON/sincon2024-3-2.png)<br /><br />

Each image is of a bunch of logic gates and light bulbs.
- [cook1elog1c-1.png](./2024-SINCON/cook1elog1c-1.png)<br />
- [cook1elog1c-2.png](./2024-SINCON/cook1elog1c-2.png)<br />
- [cook1elog1c-3.png](./2024-SINCON/cook1elog1c-3.png)<br />
- [cook1elog1c-4.png](./2024-SINCON/cook1elog1c-4.png)<br />

Refer to this [image about logic gates](https://media.geeksforgeeks.org/wp-content/uploads/logic-gates.jpg) for help.

... இ௰இ<br />
Here are the images with my annotations:

| cook1elog1c-1.png | cook1elog1c-2.png |
| :---: | :---: |
| <img src="./2024-SINCON/sincon2024-3-3.png" width="450"> | <img src="./2024-SINCON/sincon2024-3-4.png" width="260"> |
| **cook1elog1c-3.png** | **cook1elog1c-4.png** |
| :---: | :---: |
| <img src="./2024-SINCON/sincon2024-3-5.png" width="450"> | <img src="./2024-SINCON/sincon2024-3-6.png" width="450"> |

Together, it gives 01100011 01100001 01101011 00110011. Use Cyberchef:
![img](./2024-SINCON/sincon2024-3-7.png)<br /><br />

![img](./2024-SINCON/sincon2024-3-8.png)<br /><br />

Flag: ```cak3```<br /><br />

## 4

> All the layers of baking a cookie...
> 
> aGFja2Vyd2FyZS5pby9jb29rMWVsNHkzci56aXA=

<br />
Use Cyberchef:
![img](./2024-SINCON/sincon2024-4-1.png)<br /><br />

From the website [https://hackerware.io/cook1el4y3r.zip](https://hackerware.io/cook1el4y3r.zip):
- [cook1el4y3r.zip](./2024-SINCON/cook1el4y3r.zip)<br /><br />

Most of the files are “.gbr” files, which are Gerber files. A similar challenge found online involving Gerber files can be found in [DEFCON 29 HHV Challenge](https://rhye.org/post/defcon-29-hhv-ctf/).<br /><br />
![img](./2024-SINCON/sincon2024-4-2.png)<br /><br />

Use [Gerber online file viewer](https://www.pcbway.com/project/OnlineGerberViewer.html): 
![img](./2024-SINCON/sincon2024-4-3.png)<br /><br />

While viewing layers:
![img](./2024-SINCON/sincon2024-4-4.png)<br /><br />

When removing the yellow layer Badge_V1-B_Cu_Main.gbr, we see “ch3rry 0n b0tt0m”.
![img](./2024-SINCON/sincon2024-4-5.png)<br /><br />

![img](./2024-SINCON/sincon2024-4-6.png)<br /><br />

Flag: ```ch3rry 0n b0tt0m```<br /><br />

## 5

> Secret ingredient hidden inside the cookie?
> 
> aGFja2Vyd2FyZS5pby9jb29rMWVzdDNnNG5v

<br />
Use Cyberchef:
![img](./2024-SINCON/sincon2024-5-1.png)<br /><br />

In the website [https://hackerware.io/cook1est3g4no](https://hackerware.io/cook1est3g4no):
![img](./2024-SINCON/sincon2024-5-2.png)<br /><br />

There were no findings from the source code or network.<br /><br />

Adding ".jpg" to the end of the URL gives an image [https://hackerware.io/cook1est3g4no.jpg](https://hackerware.io/cook1est3g4no.jpg). 
- [cook1est3g4no.jpg](./2024-SINCON/cook1est3g4no.jpg)

Using passphrase "cook1est3g4no":
![img](./2024-SINCON/sincon2024-5-3.png)<br /><br />

![img](./2024-SINCON/sincon2024-5-4.png)<br /><br />

Flag: ```c0rn st4rch```<br /><br />

## 6

> What if the secret ingredient is Apple Cider Vignere?
> 
> d3a1r10fw

<br />
Use Cyberchef and the key "apple":
![img](./2024-SINCON/sincon2024-6-1.png)<br /><br />

![img](./2024-SINCON/sincon2024-6-2.png)<br /><br />

Flag: ```d3l1c10us```<br /><br />

## 7

> |&nbsp;|**1**|**2**|**3**|**4**|**5**|<br />
> |**1**|A|B|C|1|2|<br />
> |**2**|D|T|F|3|4|<br />
> |**3**|G|H|I|5|6|<br />
> |**4**|J|K|L|R|8|<br />
> |**5**|S|N|U|9|9|<br />
> 
> 32255221 1452 412544<br /><br />

<br />
For each set of 2 numbers, follow the table's left column for the first and the top row for the second.

| **L/R** | 32 | 25 | 52 | 21 | 14 | 52 | 41 | 25 | 44 |
| **chr** | H | 4 | N | D | 1 | N | J | 4 | R |

![img](./2024-SINCON/sincon2024-7-1.png)<br /><br />

Flag: ```h4nd 1n j4r```<br />
(Uppercase letters didn't work)<br /><br />

## 8

> $9$9vgrp0IKvLVs4EcSrlK7NgoJGUHP5F3n/UjHm5FAtp0O1IcSylMWxIE7VY4DjTz369p

<br />
Google:
![img](./2024-SINCON/sincon2024-8-1.png)<br /><br />

Use this sus decryption tool hxxp://password-decrypt[.]com/:
![img](./2024-SINCON/sincon2024-8-2.png)<br /><br />

![img](./2024-SINCON/sincon2024-8-3.png)<br /><br />

Flag: ```give that man a cookie```<br /><br />

## yay

![img](./2024-SINCON/sincon2024-end.png)<br /><br />

The badge:<br />
<img src="./2024-SINCON/badge.gif" width="450">

(▀̿Ĺ▀̿ ̿)ノ♪