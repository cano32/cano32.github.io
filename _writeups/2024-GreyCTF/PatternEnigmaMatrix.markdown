---
layout: default
ctf: Grey CTF 2024
chal: Pattern Enigma Matrix
category: reversing
flag: hm
---

# GreyCTF 2024
## Pattern Enigma Matrix

> I made this program that checks if the flag is correct. However, I forgot the flag...
>
> Can you help me recover it?

Files given:
- [a_stripped.out](./2024-GreyCTF/PatternEnigmaMatrix/dist-pattern-enigma-matrix.zip)<br /><br />

Testing the output:
- No input
  ![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-1.png){: width="100%" }<br /><br />
- Input "aaa"
  ![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-2.png){: width="100%" }<br /><br />

### IDA
Upon opening the file in IDA, there was already a problem loading it in graph form.
![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-3.png){: width="100%" }<br /><br />

Go to Options > General… > Graph, and change “Max number of nodes” to 4096.
![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-4.png){: width="100%" }<br /><br />

Click “OK” and \<spacebar\> to change to the graph view.
![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-5.png){: width="100%" }<br /><br />

### Analysis
The program takes in the input here:
![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-6.png){: width="100%" }<br /><br />

The rest of the code seems like a control flow graph.
![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-7.png){: width="100%" }<br /><br />

The decisions were as follows:
- 2x M4TCH/m4tch
- 2x 1Ng/1nG
- 2x 1M3/1m3
- 2x T1M/t1m
- 2x C0MP/c0mp
- 2x 1LE/1le
- 2x EF6F33A17D0B9C}/Ef6f33a17d0b9c}
- 6x GREY{/grey{
- 2x DEF/def
- 2x 669870FC2FEC/669870fc2fec
- 2x @F/`f
- 2x /9
- 2x @Z/`z
- 2x /9
- 2x _
- 2x @Z/`z
- 2x /9
- 2x _
- 2x @Z/`z
- 2x /9
- 2x _
- 2x @Z/`z
- 2x g/G
- 2x /9
- 2x _
- 2x @F/`f
- 2x /9<br /><br />

I deduced that the string definitely starts with “grey{“ and ends with “ef6f33a17d0b9c}”. So I started to piece them together in the way I find most appropriate and guess, slowly working my way up the cases.<br /><br />

Script:
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

The cases found so far were as follows:
- Case 1: m4tch
- Case 2: 1nG
- Case 3: 1m3
- Case 4: t1m
- Case 5: c0mp
- Case 6: 1le
- Case 7: ef6f33a17d0b9c}
- Case 8: grey{<br /><br />

A-Z was not accepted. These cases were passing but why???<br /><br />

### Analysis - Regex
Enter regex - [Regular expressions obfuscation under the microscope](https://doar-e.github.io/blog/2013/08/24/regular-expressions-obfuscation-under-the-microscope/).<br /><br />

There is a for loop from 0 which breaks if >43h (67).
![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-8.png){: width="100%" }
...
![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-9.png){: width="100%" }
- This simply checks the length of the input, whether it is 68 characters.
- Case 9: 68 chars<br /><br />

At the next function where 2x DEF/def is at, there is a for loop from 0 which breaks if >3.
![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-10.png){: width="100%" }
...
![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-11.png){: width="100%" }<br /><br />

In the for loop, a few comparisons take place for up to 4 characters.
- Following the comparison with ‘D’
  - If the character is ‘D’, zero flag ZF will be set to 1. The jump is taken in “jnz short loc_F2B1” only if ZF is cleared (ZF=0), so the jump is not taken. With “mov eax, 1”, eax will be 1 and “test al, al” results in 1, so ZF will be set to 0. The jump is taken at “jnz loc_F36F”. This eventually increments the loop count.
- Following the comparison with ‘d’
  - If the character is ‘d’, ZF will be set to 1, so “setz al” sets al to 1. “test al, al” results in 1, so ZF will be set to 0. The jump is taken at “jnz loc_F36F”. This eventually increments the loop count.
  ![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-12.png){: width="100%" }
- If the character is not ‘d’/’D’, continue the next comparison with ‘e’/’E’ and if it is also not ‘e’/’E’, continue the next comparison with ‘f’/’F’.
- This gives the regex [def]{4}. This code also seems similar to the earlier 8 Cases.<br /><br />

After the for loop, there is the comparison to 669870FC2FEC/669870fc2fec.<br /><br />

Following that is another for loop from 0 that breaks if >15.
![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-13.png){: width="100%" }
...
![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-14.png){: width="100%" }<br /><br />

In the for loop, the comparisons set the following conditions for each character:
- If ( char > ‘@’ (40h) and char <= ‘F’ (46h) ) OR ( char > ‘`’ (60h) and char <= ‘f’ (66h) )
  -> A-F OR a-f
  - If ( char > ‘/’ (2Fh) OR char <= ‘9’ (39h) )
    -> 0-9<br /><br />

Through testing, I found out that uppercase was generally not accepted.
Going back up to the first comparison (m4tch/M4TCH), there seems to be a check on byte [rbp-7Eh] which determines whether we go to the path with “M” or “m”.<br /><br />
![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-15.png){: width="100%" }
- When tracing it in the code above the comparison, the byte is 0.
  ![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-16.png){: width="100%" }
  ...
  ![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-17.png){: width="100%" }
  ...
  ![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-18.png){: width="100%" }
  <br /><br />

Referring to this again:
![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-15.png){: width="100%" }
- Since [rbp-7Eh] is 0, “test al, al” will give 0 and zero flag ZF will be set to 1. With “jz”, the jump will be taken and we will go to the path with “m”.
- If ( (!byte \| char!=M) AND (char!=m) ), jump to loc_181D, which skips through all the following comparisons.
  - The (!byte \| char!=M) would already be 1 since the first condition is fulfilled. With the AND operator, the character must be “m”, and it cannot be both “m” and “M” at the same time.
  - This happens across all of the Cases where there are 2 possible variations (uppercase/lowercase) of a character, and this is why only 1 path is taken.
- Case 10: [def]{4}669870fc2fec[A-Fa-f0-9]{16}<br /><br />

After this, there are multiple other similar loops:
- Checks for 7 char, A-Z/a-z/0-9
- ‘_’
- Checks for 4 char, A-Z/a-z/0-9
- ‘_’
- Checks for 7 char, A-Z/a-z/g/G/0-9
- ‘_’
- Checks for 8 char, A-Z/a-z/0-9
- ‘_’
- Checks for 32 char, A-F/a-f/0-9
![img](./2024-GreyCTF/PatternEnigmaMatrix/pem-19.png){: width="100%" }<br /><br />

Only the second case is accepted, aka a-z instead of A-Z, a-f instead of A-F and G instead of g.
- Case 11: [a-z0-9]{7}_[a-z0-9]{4}_[a-zG0-9]{7}_[a-z0-9]{8}_[a-z0-9]{32}<br /><br />

According to a blog post on obfuscated regular expressions https://sysexit.wordpress.com/2013/09/04/a-black-box-approach-against-obfuscated-regular-expressions-using-pin/, “This obfuscated version includes (fake) state transitions for non-valid input, so even non-valid input chars may increase the count of executed basic blocks”. This may be why there were other characters (like most uppercase characters) as bait.<br /><br />

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
> Case 10: [def]{4}(669870FC2FEC\|669870fc2fec)[A-Fa-f0-9]{16}
>
> Case 11: [a-z0-9]{7}_[a-z0-9]{4}_[a-z0-9]{7}_[a-z0-9]{8}_[a-z0-9]{32}
