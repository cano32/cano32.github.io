---
description: GreyCTF 2024 Pattern Enigma Matrix
title: GreyCTF 2024 Pattern Enigma Matrix
date: 2024-05-11 08:00:00 +0800
categories:
  - Writeups
  - Reversing
tags:
  - writeups
  - reversing
show_image_post: false
---

# GreyCTF 2024
## Pattern Enigma Matrix

> I made this program that checks if the flag is correct. However, I forgot the flag...
>
> Can you help me recover it?

Files given:
- [a_stripped.out](../../../assets/file/2024-GreyCTF/PatternEnigmaMatrix/dist-pattern-enigma-matrix.zip)<br /><br />

Testing the output:
- No input
  ![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-1-1.png){: width="100%" }<br /><br />
- Input "aaa"
  ![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-1-2.png){: width="100%" }<br /><br />

From the program output, we know we have to pass 11 cases.<br /><br />

### IDA
Upon opening the file in IDA, there was already a problem loading it in graph form.
![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-2-1.png){: width="100%" }<br /><br />

Go to Options > General… > Graph, and change “Max number of nodes” to 4096.
![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-2-2.png){: width="100%" }<br /><br />

Click “OK” and \<spacebar\> to change to the graph view.
![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-2-3.png){: width="100%" }<br /><br />

### Analysis Part 1
The program checks whether there is more than one argument.
![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-3-1.png){: width="100%" }<br /><br />

If there is no argument, it loads {flag_guess} to the output and exits.
![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-3-2.png){: width="100%" }<br /><br />

If there is an argument, the program will proceed to take it as an input.
![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-3-3.png){: width="100%" }<br /><br />

The rest of the code seems like a control flow graph.
![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-3-4.png){: width="100%" }<br /><br />

The strings that could form potential decisions are as follows:
- M4TCH/m4tch
- 1Ng/1nG
- 1M3/1m3
- T1M/t1m
- C0MP/c0mp
- 1LE/1le
- EF6F33A17D0B9C}/ef6f33a17d0b9c}
- GREY{/grey{
- DEF/def
- 669870FC2FEC/669870fc2fec
- @F/`f
- /9
- @Z/`z
- /9
- _
- @Z/`z
- /9
- _
- @Z/`z
- /9
- _
- @Z/`z
- g/G
- /9
- _
- @F/`f
- /9<br /><br />

Going to the first comparison (m4tch/M4TCH), there seems to be a check on byte [rbp-7Eh] which determines whether we go to the path with “M” or “m”.
- When tracing it in the code above the comparison, the byte is 0.
  ![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-3-5.png){: width="100%" }
  ...
  ![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-3-6.png){: width="100%" }
  ...
  ![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-3-7.png){: width="100%" }
  <br /><br />

Moving on:
![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-3-8.png){: width="100%" }
- Since check byte [rbp-7Eh] is 0, “test al, al” will give 0 and zero flag ZF will be set to 1. With “jz short loc_15D1”, the jump will be taken and we will go to the path with “m”.
- If ( (!check_byte \| char!=M) AND (char!=m) ), jump to loc_181D, which skips through all of the following comparisons.
  - The (!check_byte \| char!=M) would already be 1 since the first condition is fulfilled. With the AND operator, the character must be “m”, and it cannot be both “m” and “M” at the same time.
  - This happens across all of the Cases where there are 2 possible variations (uppercase/lowercase) of a character, and this is why only 1 path is taken.<br /><br />

After matching for "m", the program moves on to "4" -> "t" -> "c" -> "h" to form "m4tch". It moves on to the next 10 comparisons, until the end where they count how many cases you have passed out of the 11 cases.<br /><br />

The cases found so far in the same manner were as follows:
```
- Case 1: m4tch
- Case 2: 1nG
- Case 3: 1m3
- Case 4: t1m
- Case 5: c0mp
- Case 6: 1le
```
<br />

I deduced from the decisions found earlier that the string definitely starts with “grey{“ and ends with “ef6f33a17d0b9c}”.<br /><br />

Next, there is a for loop from 0 which breaks if the loop count exceeds 43h (67) (up to 68 characters).
![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-3-9.png){: width="100%" }
...
![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-3-10.png){: width="100%" }
- This simply checks the length of the input, whether it is 68 characters.<br /><br />

More cases:
```
- Case 7: ef6f33a17d0b9c}
- Case 8: grey{
- Case 9: 68 chars
```
<br />

Script bc lazy:
```
from pwn import *

# input = 'GREY{M4TCH1Ng_T1M3_C0MP1LE_EF6F33A17D0B9C}'   # 0/11 cases passed
# input = 'm4tch1nG'                                     # 2/11 cases passed
# input = 'grey{m4tch1nG}'                               # 2/11 cases passed
# input = 'grey{m4tch1nG_c0mp1le}'                       # 2/11 cases passed
# input = 'grey{m4tch1nG_t1m3}'                          # 4/11 cases passed
# input = 'grey{m4tch1nG_t1m3_c0mp1le}'                  # 6/11 cases passed
input = 'grey{m4tch1nG_t1m3_c0mp1le_ef6f33a17d0b9c}'     # 8/11 cases passed

p = process(['./dist (2)/a_stripped.out', input])
resp = p.recvall()
print(resp)
```
<br />

### Analysis Part 2
The next part is different: rather than simply matching a few word segments, we will need to match a specific number of occurrence of regex.<br /><br />

This article provides some explanation and has a graph of a similar structure - [Regular expressions obfuscation under the microscope](https://doar-e.github.io/blog/2013/08/24/regular-expressions-obfuscation-under-the-microscope/).<br /><br />

At the next function where DEF/def is at, there is a for loop from 0 which breaks if the loop count exceeds 3 (aka a few comparisons will take place for up to 4 characters).
![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-4-1.png){: width="100%" }
...
![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-4-2.png){: width="100%" }<br /><br />

Recall the comparison with "M" or "m" earlier, where there was a check on byte [rbp-7Eh] which determines whether we go to one path or another.
>   - This happens across all of the Cases where there are 2 possible variations (uppercase/lowercase) of a character, and this is why only 1 path is taken.<br /><br />

If ( (check_byte \| char==D) AND (char==d) )<br />
-> check_byte = 1. With this, in the comparison with "D" or "d", we will take the path to "d". 
- Following the comparison with "d":
  - If the character is "d", zero flag ZF will be set to 1, so “setz al” sets al to 1. “test al, al” results in 1, so ZF will be set to 0. The jump is taken at “jnz loc_F36F”. This eventually increments the loop count.
  ![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-4-3.png){: width="100%" }
- If the character is not "d", continue the next comparison with "e" and if it is also not "e", continue the next comparison with ‘f’. This whole thing loops again until 4 of our input characters have been compared. It then moves on to the next few comparisons, which also involves this kind of comparison in a for loop.
- This gives the regex ```[def]{4}```.<br /><br />

After the for loop, there is the comparison to ```669870fc2fec```.<br /><br />

Following that is another for loop from 0 that breaks if the loop count exceeds 15 (up to 16 characters).
![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-4-4.png){: width="100%" }
...
![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-4-5.png){: width="100%" }<br /><br />

In the for loop, the comparisons set the following conditions for each character:
- If ( ( check_byte AND (char > "@" (40h) AND char <= "F" (46h)) ) \| (char > "`" (60h) AND char <= "f" (66h)) )<br />
  -> check_byte = 1, char has to be [a-f]
  ![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-4-6.png){: width="100%" }
  - If ( char > ‘/’ (2Fh) OR char <= ‘9’ (39h) )<br />
    -> 0-9
    ![img](../../../assets/img/2024-GreyCTF/PatternEnigmaMatrix/pem-4-7.png){: width="100%" }
- This gives the regex ```[a-f0-9]{16}```.<br /><br />

All of these 3 ([def]{4}, 669870fc2fec, [a-f0-9]{16}) are in 1 big loop, so it forms a single case.<br /><br />

Another one:
```
- Case 10: [def]{4}669870fc2fec[a-f0-9]{16}
```
<br />

After this, there are multiple other similar loops:
- Checks for 7 char, a-z/0-9
- ‘_’
- Checks for 4 char, a-z/0-9
- ‘_’
- Checks for 7 char, a-z/G/0-9
- ‘_’
- Checks for 8 char, a-z/0-9
- ‘_’
- Checks for 32 char, a-f/0-9
<br />

This gives the regex ```[a-z0-9]{7}\_[a-z0-9]{4}\_[a-zG0-9]{7}\_[a-z0-9]{8}\_[a-z0-9]{32}```.<br /><br />

Last one:
```
- Case 11: [a-z0-9]{7}_[a-z0-9]{4}_[a-zG0-9]{7}_[a-z0-9]{8}_[a-z0-9]{32}
```
<br />

> Case 1: m4tch
>
> Case 2: 1nG
>
> Case 3: 1m3
>
> Case 4: t1m
>
> Case 5: c0mp
>
> Case 6: 1le
>
> Case 7: ef6f33a17d0b9c}
>
> Case 8: grey{
>
> Case 9: 68 chars
>
> Case 10: [def]{4}(669870FC2FEC\|669870fc2fec)[a-f0-9]{16}
>
> Case 11: [a-z0-9]{7}\_[a-z0-9]{4}\_[a-z0-9]{7}\_[a-z0-9]{8}\_[a-z0-9]{32}

<br />

**Flag:<br />
```grey{c0mp1le_t1m3_aaaaaaa_m4tch1nG_deff669870fc2fec5cef6f33a17d0b9c}```**
<br /><br />

Exceptions:
- "deff" can be any 4 char that is either "d", "e" or "f".
- "aaaaaaa" can be any 7 char as there are no other restrictions other than Case 11.
- "compile" and "aaaaaaa" can be swapped around as they are both 7 char (Case 11).<br />

The intended flag by the challenge authors is<br />
```grey{c0mp1le_t1m3_p4tt3rn_m4tch1nG_eefd669870fc2fec5cef6f33a17d0b9c}```.