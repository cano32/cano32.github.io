---
layout: default
ctf: Flare-On 2024
chal: 1-7
category: reversing
---

# Flare-On 11 (2024)

Get the binaries from the [Flare-On website](http://flare-on.com/files/Flare-On11_Challenges.zip) and
[Flare-On 11 stats and solutions by Mandiant team](https://cloud.google.com/blog/topics/threat-intelligence/flareon-11-challenge-solutions).<br />

- [1](#1)
- [2](#2)
- [3](#3)
- [4](#4)
- [5](#5)
- [6](#6)
- [7](#7)
- [8](#8)<br /><br />

## 1

> frog
>
> Welcome to Flare-On 11! Download this 7zip package, unzip it with the password 'flare', and read the README.txt file for launching instructions. It is written in PyGame so it may be runnable under many architectures, but also includes a pyinstaller created EXE file for easy execution on Windows.
>
> Your mission is get the frog to the "11" statue, and the game will display the flag. Enter the flag on this page to advance to the next stage. All flags in this event are formatted as email addresses ending with the @flare-on.com domain.
>
> File: frog.7z

<br />
Folder contents<br />
![img](./2024-FlareOn11/flareon2024-1-1.png)<br /><br />

Run frog.py. You need to move the frog through a maze to reach the statue.<br />
![img](./2024-FlareOn11/flareon2024-1-2.png)<br /><br />

I didn't even read the code... just clicked randomly and you can get past here

| 1 | 2 |
| :---: | :---: |
| <img src="./2024-FlareOn11/flareon2024-1-3.png" width="260"> | <img src="./2024-FlareOn11/flareon2024-1-4.png" width="260"> |

![img](./2024-FlareOn11/flareon2024-1-5.png)<br /><br />

![img](./2024-FlareOn11/flareon2024-1-6.png)<br /><br />

Flag: ```welcome_to_11@flare-on.com```<br /><br />

Anyways time to actually look at the code:<br />
```python
def BuildBlocks():
    ...
    Block(15, 4, True),
    ...
    Block(13, 10, True),
    ...
```

These are the exact blocks that we could past through earlier.<br /><br />

## 2

> checksum
>
> We recently came across a silly executable that appears benign. It just asks us to do some math... From the strings found in the sample, we suspect there are more to the sample than what we are seeing. Please investigate and let us know what you find!
> 
> File: checksum.7z

<br />
![img](./2024-FlareOn11/flareon2024-2-1.png)<br /><br />

Opening it up in IDA, we observe in function main_main, this will be our goal - to produce REAL_FLAREON_FLAG.JPG<br />
![img](./2024-FlareOn11/flareon2024-2-2.png)<br /><br />

Starting near the top, you would want to go to the right path (green line) to continue and avoid the math.<br />
![img](./2024-FlareOn11/flareon2024-2-3.png)<br /><br />

Edit > Patch program > Change byte<br />
Patch jge to jl<br />
![img](./2024-FlareOn11/flareon2024-2-4.png)<br />
- Original<br />
![img](./2024-FlareOn11/flareon2024-2-5.png)<br />
- New<br />
![img](./2024-FlareOn11/flareon2024-2-6.png)<br /><br />

Now it only waits for an input<br />
![img](./2024-FlareOn11/flareon2024-2-7.png)<br />
![img](./2024-FlareOn11/flareon2024-2-8.png)<br /><br />

I initially tried to debug and patch everywhere to get the image file produced but it’s empty<br />
![img](./2024-FlareOn11/flareon2024-2-9.png)<br />
![img](./2024-FlareOn11/flareon2024-2-10.png)<br /><br />

From static analysis,
- It uses ChaCha20-Poly1305, which has a key length of 32, nonce 12 and tag 16.<br />
![img](./2024-FlareOn11/flareon2024-2-11.png)<br />
- There is a main_encryptedFlagData, at offset main__gobytes_1 (addr 1ADA80 to 1D9FAB), which is size 2C52Ch.<br />
![img](./2024-FlareOn11/flareon2024-2-12.png)<br />
In hex view, select bytes from 1ADA80 to 1D9FAB and Shift+e, save as raw bytes.
- It takes the sha256sum of the main_encryptedFlagData (d32cda47783eb42302583dafba4cfb151bfa145ccfb4e9d011cbd862b1075667), which happens to be length 32.<br />
![img](./2024-FlareOn11/flareon2024-2-13.png)<br />
- REAL_FLAREON_FLAG image file will be produced in %LocalAppData% path.<br />
![img](./2024-FlareOn11/flareon2024-2-14.png)<br /><br />

main_b seems to be for printing the output to command prompt.<br /><br />

main_a seems to be performing some sort of base64 encoding and XOR encryption.
- DAT_004c8035/aFlareon2024 = “FlareOn2024”
- Comparison with string length 88 = 0x58
- memequal is called and compared to “cQoFRQErX1YAVw1zVQdFUSxfAQNRBXUNAxBSe15QCVRVJ1pQEwd/WFBUAlElCFBFUnlaB1ULByRdBEFdfVtWVA==” in rbx<br />
(weird looks like i used ghidra here...)<br />
![img](./2024-FlareOn11/flareon2024-2-15.png)<br />
![img](./2024-FlareOn11/flareon2024-2-16.png)<br /><br />

Script based on main_a
```python
import base64

flareon2024 = [ord(c) for c in "FlareOn2024"]
base64_encoded_str = "cQoFRQErX1YAVw1zVQdFUSxfAQNRBXUNAxBSe15QCVRVJ1pQEwd/WFBUAlElCFBFUnlaB1ULByRdBEFdfVtWVA=="
decoded_str = base64.b64decode(base64_encoded_str)

# Decrypt using XOR
decrypted_data = bytes([decoded_str[i] ^ flareon2024[i % len(flareon2024)] for i in range(len(decoded_str))])
print(decrypted_data.decode('utf-8'))
```
<br />

The resulting checksum is 7fd7dd1d0e959f74c133c13abb740b9faa61ab06bd0ecd177645e93b1e3825dd (turns out to be sha256sum of REAL_FLAREON_FLAG.JPG).<br />
![img](./2024-FlareOn11/flareon2024-2-17.png)<br /><br />
![img](./2024-FlareOn11/flareon2024-2-18.png)<br /><br />
Flag: ```Th3_M4tH_Do_b3_mAth1ng@flare-on.com```<br /><br />

## 3

> aray
>
> And now for something completely different. I'm pretty sure you know how to write Yara rules, but can you reverse them?
>
> File: aray.7z

<br />
The file is a yara file.<br /><br />

These are useless:
- uint8\(\d+\) < \d+\n
- uint8\(\d+\) > \d+\n
- uint8\(\d+\) % \d+ < \d+\n
- filesize \^ uint8\(\d+\) != \d+\n
- uint8\(\d+\) & 128 == 0\n
<br /><br />

New condition without useless stuff:
```
import "hash"

rule aray
{
	meta:
    	description = "Matches on b7dc94ca98aa58dabb5404541c812db2"
	condition:
    	filesize == 85 and
    	hash.md5(0, filesize) == "b7dc94ca98aa58dabb5404541c812db2" and
    	uint8(58) + 25 == 122 and
    	uint32(52) ^ 425706662 == 1495724241 and
    	uint32(17) - 323157430 == 1412131772 and
    	hash.crc32(8, 2) == 0x61089c5c and
    	hash.crc32(34, 2) == 0x5888fc1b and
    	uint8(36) + 4 == 72 and
    	uint8(27) ^ 21 == 40 and
    	uint32(59) ^ 512952669 == 1908304943 and
    	uint8(65) - 29 == 70 and
    	uint8(45) ^ 9 == 104 and
    	uint32(28) - 419186860 == 959764852 and
    	uint8(74) + 11 == 116 and
    	hash.crc32(63, 2) == 0x66715919 and
    	hash.sha256(14, 2) == "403d5f23d149670348b147a15eeb7010914701a7e99aad2e43f90cfa0325c76f" and
    	hash.sha256(56, 2) == "593f2d04aab251f60c9e4b8bbc1e05a34e920980ec08351a18459b2bc7dbf2f6" and
    	uint8(75) - 30 == 86 and
    	uint32(66) ^ 310886682 == 849718389 and
    	uint32(10) + 383041523 == 2448764514 and
    	uint32(37) + 367943707 == 1228527996 and
    	uint32(22) ^ 372102464 == 1879700858 and
    	uint8(2) + 11 == 119 and
    	hash.md5(0, 2) == "89484b14b36a8d5329426a3d944d2983" and
    	uint32(46) - 412326611 == 1503714457 and
    	hash.crc32(78, 2) == 0x7cab8d64 and
    	uint8(7) - 15 == 82 and
    	uint32(70) + 349203301 == 2034162376 and
    	hash.md5(76, 2) == "f98ed07a4d5f50f7de1410d905f1477f" and
    	uint32(80) - 473886976 == 69677856 and
    	uint32(3) ^ 298697263 == 2108416586 and
    	uint8(21) - 21 == 94 and
    	uint8(16) ^ 7 == 115 and
    	uint32(41) + 404880684 == 1699114335 and
    	hash.md5(50, 2) == "657dae0913ee12be6fb2a6f687aae1c7" and
    	uint8(26) - 7 == 25 and
    	hash.md5(32, 2) == "738a656e8e8ec272ca17cd51e12f558b" and
    	uint8(84) + 3 == 128 and
}
```
<br />

CRC32, SHA256 script:
```python
import zlib
import hashlib

# Target CRC32 value
target_crc = [0x61089c5c, 0x5888fc1b, 0x66715919, 0x7cab8d64]

for target_crc_idx in target_crc:
	# Brute force search for two bytes
	for b1 in range(256):
		for b2 in range(256):
			# Create a byte array with the two bytes
			data = bytes([b1, b2])
			# Calculate CRC32
			crc = zlib.crc32(data) & 0xffffffff  # Ensure it's a uint32
			if crc == target_crc_idx:
				print(f"[CRC] Found bytes: {chr(b1)}, {chr(b2)} -> CRC32: {crc:#x}")

# Target SHA-256 hash
sha256_target_hash = ["403d5f23d149670348b147a15eeb7010914701a7e99aad2e43f90cfa0325c76f", "593f2d04aab251f60c9e4b8bbc1e05a34e920980ec08351a18459b2bc7dbf2f6"]

for target_hash_idx in sha256_target_hash:
	# Brute force search for two bytes
	for b1 in range(256):
		for b2 in range(256):
			# Create a byte array with the two bytes
			data = bytes([b1, b2])
			# Calculate SHA-256
			sha256_hash = hashlib.sha256(data).hexdigest()
			# Check if the hash matches
			if sha256_hash == target_hash_idx:
				print(f"[SHA256] Found bytes: {chr(b1)}, {chr(b2)} -> SHA-256: {sha256_hash}")

# Target MD5 hash
md5_target_hash = ["89484b14b36a8d5329426a3d944d2983", "f98ed07a4d5f50f7de1410d905f1477f", "657dae0913ee12be6fb2a6f687aae1c7",  "738a656e8e8ec272ca17cd51e12f558b"]

for target_hash_idx in md5_target_hash:
	# Brute force search for two bytes
	for b1 in range(256):
		for b2 in range(256):
			# Create a byte array with the two bytes
			data = bytes([b1, b2])
			# Calculate MD5
			md5_hash = hashlib.md5(data).hexdigest()
			# Check if the hash matches
			if md5_hash == target_hash_idx:
				print(f"[MD5] Found bytes: {chr(b1)}, {chr(b2)} -> MD5: {md5_hash}")
```
<br />

Obvious ones:

| Rule | Meaning |
| :--- | :------ |
| uint8(58) + 25 == 122 | aray[58] = 97 = "a" |
| hash.crc32(8, 2) == 0x61089c5c | aray[8, 9] = 114, 101 = "re" |
| hash.crc32(34, 2) == 0x5888fc1b | aray[33, 34] = 101, 65 = "eA" |
| uint8(36) + 4 == 72 | aray[36] = 68 = "D" |
| uint8(27) ^ 21 == 40 | aray[27] = 61 = "=" |
| uint8(65) - 29 == 70 | aray[65] = 99 = "c" |
| uint8(45) ^ 9 == 104 | aray[45] = 97 = "a" |
| uint8(74) + 11 == 116 | aray[74] = 105 = "i" |
| hash.crc32(63, 2) == 0x66715919 | aray[63, 64] = 110, 46 = "n." |
| hash.sha256(14, 2) == "403d5f23d149670348b147 a15eeb7010914701a7e99aad2e43f90cfa0325c76f" | aray[14, 15] = 32, 115 = " s" |
| hash.sha256(56, 2) == "593f2d04aab251f60c9e4b 8bbc1e05a34e920980ec08351a18459b2bc7dbf2f6" | aray[56, 57] = 102, 108 = "fl" |
| uint8(75) - 30 == 86 | aray[75] = "t" |
| uint8(2) + 11 == 119 | aray[2] = 108 = "l" |
| hash.md5(0, 2) == "89484b14b36a8d5329426a3d944d2983" | aray[0, 1] = 114, 117 = "ru" |
| hash.crc32(78, 2) == 0x7cab8d64 | aray[78, 79] = 110, 58 = "n:" |
| uint8(7) - 15 == 82 | aray[7] = 97 = "a" |
| hash.md5(76, 2) == "f98ed07a4d5f50f7de1410d905f1477f" | aray[76, 77] = 105, 111 = "io" |
| uint8(21) - 21 == 94 | aray[21] = 115 = "s" |
| uint8(16) ^ 7 == 115 | aray[16] = 116 = "t" |
| hash.md5(50, 2) == "657dae0913ee12be6fb2a6f687aae1c7" | aray[50, 51] = 51, 65 = "3A" |
| uint8(26) - 7 == 25 | aray[26] = 32 = " " |
| hash.md5(32, 2) == "738a656e8e8ec272ca17cd51e12f558b" | aray[32, 33] = 117, 108 = "ul" |
| uint8(84) + 3 == 128 | aray[84] = 125 = "}" |

This gives us<br />
```rul____are____ st____s____ =____uleAD________a____3A____fla____n.c________ition:____}```<br /><br />

Script for uint32 stuff:
```python
def decimal_to_hex_ascii(val1, val2, operation):
	# Convert decimal values to hexadecimal
	hex1 = hex(val1)
	hex2 = hex(val2)
	   
	# Perform the specified operation
	if operation == '+':
	   	result = val1 + val2
	elif operation == '-':
	   	result = val1 - val2
	elif operation == '^':
	   	result = val1 ^ val2
	else:
	   	raise ValueError("Unsupported operation. Use '+', '-', or '^'.")
   
	# Convert the result to hexadecimal
	hex_result = hex(result)
    
	# Convert the hexadecimal result to ASCII
	try:
	   	ascii_result = bytes.fromhex(hex_result[2:]).decode('ascii')
	except UnicodeDecodeError:
	   	ascii_result = None  # Handle cases where conversion isn't valid ASCII
    
	print({
	   	'hex1': hex1,
	   	'hex2': hex2,
	   	'result': hex_result,
	   	'ascii_result': ascii_result
	})

decimal_to_hex_ascii(2108416586, 298697263, "^")	# 3
decimal_to_hex_ascii(2448764514, 383041523, "-")	# 10
decimal_to_hex_ascii(1412131772, 323157430, "+")	# 17
decimal_to_hex_ascii(1879700858, 372102464, "^")	# 22
decimal_to_hex_ascii(959764852, 419186860, "+")    	# 28
decimal_to_hex_ascii(1228527996, 367943707, "-")	# 37
decimal_to_hex_ascii(1699114335, 404880684, "-")	# 41
decimal_to_hex_ascii(1503714457, 412326611, "+")	# 46
decimal_to_hex_ascii(1495724241, 425706662, "^")	# 52
decimal_to_hex_ascii(1908304943, 512952669, "^")	# 59
decimal_to_hex_ascii(849718389, 310886682, "^")    	# 66
decimal_to_hex_ascii(2034162376, 349203301, "-")	# 70
decimal_to_hex_ascii(69677856, 473886976, "+")    	# 80
```
<br />

This gives us:
```
{'hex1': '0x5926f0d1', 'hex2': '0x195fc4a6', 'result': '0x40793477', 'ascii_result': '@y4w'}
{'hex1': '0x542b6bbc', 'hex2': '0x1342fdb6', 'result': '0x676e6972', 'ascii_result': 'gnir'}
{'hex1': '0x71be6c2f', 'hex2': '0x1e93095d', 'result': '0x6f2d6572', 'ascii_result': 'o-er'}
{'hex1': '0x3934d974', 'hex2': '0x18fc48ac', 'result': '0x52312220', 'ascii_result': 'R1" '}
{'hex1': '0x32a5ac75', 'hex2': '0x1287c11a', 'result': '0x20226d6f', 'ascii_result': ' "mo'}
{'hex1': '0x91f52e62', 'hex2': '0x16d4bff3', 'result': '0x7b206e6f', 'ascii_result': '{ no'}
{'hex1': '0x4939d97c', 'hex2': '0x15ee601b', 'result': '0x334b7961', 'ascii_result': '3Kya'}
{'hex1': '0x7009f57a', 'hex2': '0x162dd540', 'result': '0x6624203a', 'ascii_result': 'f$ :'}
{'hex1': '0x59a0dc99', 'hex2': '0x18939ad3', 'result': '0x7234776c', 'ascii_result': 'r4wl'}
{'hex1': '0x793edac8', 'hex2': '0x14d06b65', 'result': '0x646e6f63', 'ascii_result': 'dnoc'}
{'hex1': '0x4273320', 'hex2': '0x1c3ef100', 'result': '0x20662420', 'ascii_result': ' f$ '}
{'hex1': '0x7dabe24a', 'hex2': '0x11cdc22f', 'result': '0x6c662065', 'ascii_result': 'lf e'}
{'hex1': '0x65466d5f', 'hex2': '0x1821fd2c', 'result': '0x4d247033', 'ascii_result': 'M$p3'}
```
<br />

Fill in the blanks from earlier:<br />
```rule flareon { strings: $f = “1RuleADayK33p$Malw4r3Aw4y@flare-on.com” condition: $f ```<br />

Flag: ```1RuleADayK33p$Malw4r3Aw4y@flare-on.com```.<br /><br />

## 4

> Meme Maker 3000
>
> You've made it very far, I'm proud of you even if noone else is. You've earned yourself a break with some nice HTML and JavaScript before we get into challenges that may require you to be very good at computers.
>
> File: mememaker3000.7z

<br />
It’s a .html file, and you can select memes and edit the captions<br />
![img](./2024-FlareOn11/flareon2024-4-1.png)<br /><br />

Make it prettier with [https://deobfuscate.io/](https://deobfuscate.io/).<br />
I edited the JavaScript code to deobfuscate and run on its own as .js file.<br /><br />
[Zip file](./2024-FlareOn11/4-mememaker3000.zip)
- mememakerprettify.js
- mememakerprettify_edited.js
<br /><br />

Excluding mainly the huge var assignment, we get this:
```js
const a0p = a0b;
(function (a, b) {
  const o = a0b, c = a();
  while (true) {
	try {
  	const d = 356255;
  	if (d === b) break; else c.push(c.shift());
	} catch (e) {
  	c.push(c.shift());
	}
  }
}(a0a, 356255));

function a0f() {
  const q = a0p;
  document[getElementById]("caption1")[hidden] = true, document[getEle + "mentBy" + "Id"](caption2)[hidden] = true, document[getElementById]("caption3").hidden = true;
  const a = document[getElementById]("meme-template");
  var b = a[value][split](".")[0];

 const r = q;
  a0d[b][forEach](function (c, d) {
	const r = q;
	var e = document[getElementById](caption + (d + 1));
	e[hidden] = false, e.style[top] = a0d[b][d][0], e.style[left] = a0d[b][d][1], e[textContent] = a0c[Math[floor](Math[random]() * (a0c[length] - 1))];
  });
}

a0f();
function a0b(a, b) {
  const c = a0a();
  return a0b = function (d, e) {
	d = d - 475;
	let f = c[d];
	return f;
  }, a0b(a, b);
}

const a0g = document[getElementById](meme-image), a0h = document[getElementById](meme-container), a0i = document[getElementById](remake), a0j = document[getElementById](meme-template);
a0g[src] = a0e[a0j.value], a0j[addEventListener](change, () => {
  const s = a0p;
  a0g[src] = a0e[a0j[value]], a0g[alt] = a0j[value], a0f();
}), a0i[addEventListener]("click", () => {
  a0f();
});

function a0k() {
  const t = a0p, a = a0g[alt].split("/")[pop]();
  if (a !== Object[keys](a0e)[5]) return;
  const b = a0l.textContent, c = a0m[textContent], d = a0n.textContent;
  if (a0c[indexOf](b) == 14 && a0c[indexOf](c) == a0c[length] - 1 && a0c[indexOf](d) == 22) {
	var e = (new Date)[getTime]();
	while ((new Date)[getTime]() < e + 3e3) {}
	// flag is in var f
	var f = d[3] + "h" + a[10] + b[2] + a[3] + c[5] + c[c[length] - 1] + "5" + a[3] + "4" + a[3] + c[2] + c[4] + c[3] + "3" + d[2] + a[3] + "j4" + a0c[1][2] + d[4] + "5" + c[2] + d[5] + "1" + c[11] + "7" + a0c[21][1] + b[replace](" ", "-") + a[11] + a0c[4][substring](12, 15);
	// Congratulations! Here you go: + f
	f = f[toLowerCase](), alert(atob(Q29uZ3JhdHVsYXRpb25zISBIZXJlIHlvdSBnbzog) + f);
  }
}

const a0l = document[getElementById]("caption1"), a0m = document[getElementById](caption2), a0n = document.getElementById(caption3);
a0l[addEventListener]("keyup", () => {
  a0k();
}), a0m[addEventListener](keyup, () => {
  a0k();
}), a0n[addEventListener](keyup, () => {
  a0k();
});
```
<br />

Take note of the comments. To get f, we need to type a specific text in caption1, caption2 and caption3 to trigger the keyup event and call function a0k().<br /><br />

In function a0k(), 
- The function first retrieves the alt text of an image (from a0g) and splits it by /, taking the last part (using pop()).
  - It checks if this last part matches the sixth key in a0e (using Object.keys(a0e)[5]). If it doesn’t match, the function returns early and does nothing.
  - a0e stores the image files in order {“doge1.png", "draw.jpg", "drake.jpg", "two_buttons.jpg", "boy_friend0.jpg", "success.jpg", "disaster.jpg, "aliens.jpg"}, meaning a0e[5] would require "boy_friend0.jpg", which has 3 captions.
  ![img](./2024-FlareOn11/flareon2024-4-2.png)
- It gets the text content from caption1, caption2 and caption3, saving to const b, c, d respectively.
- These conditions are checked to compare the text content of the captions to specific indices in the array a0c:
  - a0c.indexOf(b) == 14
  - a0c.indexOf(c) == a0c.length - 1
  - a0c.indexOf(d) == 22
<br /><br />

I also printed the required captions in the JavaScript code in the zip file.<br /><br />

The captions are:
- FLARE On
- Security Expert
- Malware
<br />

![img](./2024-FlareOn11/flareon2024-4-3.png)<br /><br />
Flag: ```wh0a_it5_4_cru3l_j4va5cr1p7@flare-on.com```<br /><br />

## 5

> sshd
>
> Our server in the FLARE Intergalactic HQ has crashed! Now criminals are trying to sell me my own data!!! Do your part, random internet hacker, to help FLARE out and tell us what data they stole! We used the best forensic preservation technique of just copying all the files on the system for you.
>
> File: sshd.7z

<br />
It is a Linux file system.<br />
If a server crashed, look at program crash data in the core dump.<br />
```bash
$ locate coredump    
/home/kali/Desktop/ssh_container/var/lib/systemd/coredump
/home/kali/Desktop/ssh_container/var/lib/systemd/coredump/sshd.core.93794.0.0.11.1725917676
```
<br />

There is a core dump file generated by sshd.<br />
```bash
$ file ssh_container/var/lib/systemd/coredump/sshd.core.93794.0.0.11.1725917676
ssh_container/var/lib/systemd/coredump/sshd.core.93794.0.0.11.1725917676: ELF 64-bit LSB core file, x86-64, version 1 (SYSV), SVR4-style, from 'sshd: root [priv]', real uid: 0, effective uid: 0, real gid: 0, effective gid: 0, execfn: '/usr/sbin/sshd', platform: 'x86_64'
```
<br />

Use GDB, use “set sysroot” to load symbols from there.<br />
```bash
cd /home/kali/Desktop/ssh_container
gdb -ex "set sysroot /home/kali/Desktop/ssh_container" usr/sbin/sshd var/lib/systemd/coredump/sshd.core.93794.0.0.11.1725917676
(gdb) bt
#0  0x0000000000000000 in ?? ()
#1  0x00007f4a18c8f88f in ?? () from /home/kali/Desktop/ssh_container/lib/x86_64-linux-gnu/liblzma.so.5
#2  0x000055b46c7867c0 in ?? ()
#3  0x000055b46c73f9d7 in ?? ()
#4  0x000055b46c73ff80 in ?? ()
#5  0x000055b46c71376b in ?? ()
#6  0x000055b46c715f36 in ?? ()
#7  0x000055b46c7199e0 in ?? ()
#8  0x000055b46c6ec10c in ?? ()
#9  0x00007f4a18e5824a in __libc_start_call_main (main=main@entry=0x55b46c6e7d50, argc=argc@entry=4, argv=argv@entry=0x7ffcc6602eb8)
	at ../sysdeps/nptl/libc_start_call_main.h:58
#10 0x00007f4a18e58305 in __libc_start_main_impl (main=0x55b46c6e7d50, argc=4, argv=0x7ffcc6602eb8, init=<optimized out>,
	fini=<optimized out>, rtld_fini=<optimized out>, stack_end=0x7ffcc6602ea8) at ../csu/libc-start.c:360
#11 0x000055b46c6ec621 in ?? ()
```
- From this, we know some code from liblzma.so.5 was loaded and executed. 
(refer to [xzutils](https://www.wiz.io/blog/cve-2024-3094-critical-rce-vulnerability-found-in-xz-utils), which has 3 hooks: RSA_public_decrypt, EVP_PKEY_set1_RSA and RSA_get0_key)
- We can also tell from the asm code that the program stopped in liblzma.so.5 0x0010988f.
![img](./2024-FlareOn11/flareon2024-5-1.png)<br /><br />

Analyze liblzma.so.5:<br />
In FUN_00109820,
- local_108: bytearray of size 200
- _Var1: stores current user ID. 
- The important code only executes if the user is root (user ID 0) and if *param_2 address matches 0xc5407a48.
- FUN_001093f0
  - Prepares local_108, cryptographic data used for decryption in FUN_00109520.
  - Contains “expand 32-byte k” -> chacha20
- __dest = mmap((void *)0x0,(long)DAT_00132360,7,0x22,-1,0);
  - Allocate a block of memory __dest of size DAT_00132360 (F96h = 3990).
- pcVar2 = (code *)memcpy(__dest,&DAT_00123960,(long)DAT_00132360);
  - The bytes in the core dump at 0x00007f4a188a1000 appear in FUN_00109820 &DAT_00123960 (encrypted shellcode).
  - Data (F96h = 3990 bytes) is copied from DAT_00123960 into allocated memory __dest.
  - At that line, pcVar2 points to the start of the block of memory that has been filled with the data from DAT_00123960. This allows the program to call that code indirectly using pcVar2 as a function pointer later in the execution.
- FUN_00109520
  - Decryption: byte manipulation and operations.
  - Mathematical Operations: The heavy use of addition, bitwise operations, and conditionals resembles the operations performed in ChaCha20, which relies on mixing data using modular arithmetic and bitwise operations.
  - State Update: ChaCha20 uses a state that is updated through rounds of operations. The complexity and pattern of operations in FUN_00109520 might suggest that it is updating a similar internal state.
- (*pcVar2)();
  - This executes the code stored in DAT_00123960.
<br /><br />

Referring back to the core dump file, we find the stack at 0x7ffcc6601e90.<br />
![img](./2024-FlareOn11/flareon2024-5-2.png)
- 0x7ffcc6601e90: 0x55b46d51dde0: value 0xc5407a48, an address in liblzma.so.5 that contains data to prepare the key and nonce
- 0x7ffcc6601e98: 0x00007f4a18c8f88f: crashed code in liblzma.so.5
- 0x7ffcc6601ea0: 0x55b46d58df60: 1D 14 1E 36 B1 55 00 00 26 2C 77 41 EF 4B 0D 59
- 0x7ffcc6601ea8: 0x00007f4a188a1000: encrypted shellcode
- 0x7ffcc6601eb0: DAT_55b46d51dde4: contains chacha20 key
- 0x7ffcc6601eb8: DAT_55b46d51de04: contains chacha20 nonce
<br /><br />

Refer to the chacha20 key and nonce format here:
- [https://xilinx.github.io/Vitis_Libraries/security/2019.2/guide_L1/internals/chacha20.html](https://xilinx.github.io/Vitis_Libraries/security/2019.2/guide_L1/internals/chacha20.html)
- [https://www.uptycs.com/blog/threat-research-report-team/rtm-locker-ransomware-as-a-service-raas-linux](https://www.uptycs.com/blog/threat-research-report-team/rtm-locker-ransomware-as-a-service-raas-linux)
- [https://ar5iv.labs.arxiv.org/html/1907.11941](https://ar5iv.labs.arxiv.org/html/1907.11941)
- [https://zenn.dev/mahiro33/articles/40d0efb0b5b32a](https://zenn.dev/mahiro33/articles/40d0efb0b5b32a)
<br /><br />

```
65 78 70 61 6e 64 20 33 32 2d 62 79 74 65 20 6b // expand 32-byte k
94 3d f6 38 a8 18 13 e2 de 63 18 a5 07 f9 a0 ba // key
2d bb 8a 7b a6 36 66 d0 8d 11 a6 5e c9 14 d6 6f // key
3f 00 00 00 f2 36 83 9f 4d cd 71 1a 52 86 29 55 // nonce
```

Input the bytes from core dump file's 0x00007f4a188a1000 and the key and nonce into CyberChef.<br />
![img](./2024-FlareOn11/flareon2024-5-3.png)<br /><br />

The end of the decrypted output looks like this:<br />
![img](./2024-FlareOn11/flareon2024-5-4.png)<br /><br />

In ghidra, load the decrypted file, select all code and click ‘d’ to disassemble it. We can see that it makes several syscalls.<br /><br />

We need to do the following:
- Write a file to load the liblzma.so.5.4.1 library.
- Call the same function that decrypts and calls the shellcode (liblzma.so.5.4.1 FUN_00109820).
- Pass in the size of the shellcode, and the key (obtained from memory in core dump file).
<br /><br />

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <dlfcn.h>
#include <cstdint>
 
typedef void (*func_t)(unsigned int, char*);

int main()
{
	void *lib = dlopen("./liblzma.so.5.4.1", RTLD_NOW);
	if (!lib)
	{
    	puts("Lib failed to load");
    	return 0;
	}
 
	unsigned char *version = (unsigned char*) dlsym(lib, "lzma_version_string");
 
	// Calculate decrypt function location
	func_t decrypt = (func_t)(version + 0x4760);
 
	char key[] = "\x48\x7a\x40\xc5"
             	"\x94\x3d\xf6\x38\xa8\x18\x13\xe2\xde\x63\x18\xa5\x07\xf9\xa0\xba"
             	"\x2d\xbb\x8a\x7b\xa6\x36\x66\xd0\x8d\x11\xa6\x5e\xc9\x14\xd6\x6f"
             	"\xf2\x36\x83\x9f\x4d\xcd\x71\x1a\x52\x86\x29\x55\x58\x58\xd1\xb7"
             	"\xf9\xa7\xc2\x0d\x36\xde\x0e\x19\xea\xa3\x05\x96\xda\x59\xb9\xb9";
 
	puts("Call starting");
	decrypt(0xF96, key);
 
	return 0;
}
```
<br />

Compile and run it in strace. Take note we need to set uid to root as discussed above.<br />
```bash
g++ test.cpp -o test -ldl -g
sudo strace -u root ./test
```
We can see all the syscalls like open, read...<br /><br />

At this point, we have not set up our server.
![img](./2024-FlareOn11/flareon2024-5-5.png)<br /><br />
Together with reference to Ghidra on the decrypted file, we learn that it
- Reads 32 bytes, 12 bytes, 4 bytes of data from server.
- Reads file name to be read.
- Reads data from address in memory where the read data should be stored, 128 bytes.
<br /><br />

Set up our server.<br />
```bash
sudo ip addr add 10.0.2.15/24 dev eth0		// set ip addr
ip addr show						// verify ip addr set
```
<br />

Server script (to be modified later):
```python
import socket

# write bytes to test file
byte_data = b"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA"

with open("test.txt", 'wb') as file:
	file.write(byte_data)

# Set up the server
host = '10.0.2.15'  # The server's IP address
port = 1337     	# The port to listen on

# Create a TCP/IP socket
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Bind the socket to the address and port
server_socket.bind((host, port))

# Listen for incoming connections
server_socket.listen()

print(f'Server listening on {host}:{port}')

while True:
	# Wait for a connection
	client_socket, client_address = server_socket.accept()
	print(f'Connection from {client_address}')

	# Handle the connection (for demonstration, we'll just send a message)
	client_socket.send(b'aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa')	# 32 bytes
	client_socket.send(b'bbbbbbbbbbbb')                     # 12 bytes
	client_socket.send(b'cccc')			        # 4 bytes
	client_socket.send(b'test.txt')	# filename - file has data 0x80 = 128 bytes
```
<br />

Run the python file with dummy inputs and run gdb.
```bash
sudo gdb -ex "set sysroot /home/kali/Desktop/ssh_container" ./test

(gdb) r
(gdb) info proc mappings
(gdb) dump memory dump_file.bin 0x7ffffffde000 0x7ffffffff000	// for stack
```

Search in the dump_file.bin for the dummy inputs and filename
- 32 + 12 bytes input offset: 0x1EE98; filename offset: 0x1EEC8<br />
![img](./2024-FlareOn11/flareon2024-5-6.png)
- 4 bytes input offset: 0x20048
![img](./2024-FlareOn11/flareon2024-5-7.png)
- 32 + 12 bytes input offset: 0x20098, 0x200E0
![img](./2024-FlareOn11/flareon2024-5-8.png)
...
![img](./2024-FlareOn11/flareon2024-5-9.png)<br /><br />

On the test.cpp file stack,
- From "aaaaa…test.txt" to "expand 32-byte k" (which also appears in sshd core dump stack)
  - 0x201F0 - 0x1EE98 = 0x1358
  - This is the space between the stack of the main process and the stack in the shellcode.
<br /><br />

Using this information, in sshd core,
- 0x7ffc c660 1f40 - 0x1358 = 0x7ffc c660 0be8<br />
![img](./2024-FlareOn11/flareon2024-5-10.png)
  - 0x7ffc c660 0be8: 32 byte data
  - 0x7ffc c660 0c08: 12 byte data
  - 0x7ffc c660 0c15: 4 byte data
  - 0x7ffc c660 0c18: filename /root/certificate_autority_signing_key.txt
<br /><br />

```
8d ec 91 12 eb 76 0e da 7c 7d 87 a4 43 27 1c 35 d9 e0 cb 87 89 93 b4 d9 04 ae f9 34 fa 21 66 d7     // 32 bytes
11 11 11 11 11 11 11 11 11 11 11 11     // 12 bytes
20 00 00 00                             // 4 bytes
2f 72 6f 6f 74 2f 63 65 72 74 69 66 69 63 61 74 65 5f 61 75 74 68 6f 72 69 74 79 5f 73 69 67 6e 69 6e 67 5f 6b 65 79 2e 74 78 74 00      // filename
```
<br />

Using the info from strace without server.py
```bash
getuid()                            	= 0
mmap(NULL, 3990, PROT_READ|PROT_WRITE|PROT_EXEC, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7efdcda14000
socket(AF_INET, SOCK_STREAM, IPPROTO_TCP) = 3
connect(3, {sa_family=AF_INET, sin_port=htons(1337), sin_addr=inet_addr("10.0.2.15")}, 16) = -1 ECONNREFUSED (Connection refused)
recvfrom(-111, 0x7ffde85cf288, 32, 0, NULL, NULL) = -1 EBADF (Bad file descriptor)
recvfrom(-111, 0x7ffde85cf2a8, 12, 0, NULL, NULL) = -1 EBADF (Bad file descriptor)
recvfrom(-111, 0x7ffde85d0438, 4, 0, NULL, NULL) = -1 EBADF (Bad file descriptor)
recvfrom(-111, 0x7ffde85cf2b8, 0, 0, NULL, NULL) = -1 EBADF (Bad file descriptor)
open("", O_RDONLY)                  	= -1 ENOENT (No such file or directory)
read(-2, 0x7ffde85cf3b8, 128)       	= -1 EBADF (Bad file descriptor)
sendto(-111, "\6\0\0\0", 4, 0, NULL, 0) = -1 EBADF (Bad file descriptor)
sendto(-111, "yV\253\254O\227", 6, 0, NULL, 0) = -1 EBADF (Bad file descriptor)
```
- recvfrom() reads from the 32 bytes, 12 bytes, 4 bytes data. 
- Lastly it reads the filename at 0x7ffde85cf2b8 and then read() reads 128 bytes at 0x7ffde85cf3b8.
- 0x7ffde85cf3b8 - 0x7ffde85cf2b8 = 0x100
<br /><br />

Taking this,
- Add to the address of the filename in the sshd core dump
  - 0x7ffc c660 0c18 + 0x100 = 0x7ffc c660 0d18
![img](./2024-FlareOn11/flareon2024-5-11.png)<br /><br />

Updated server.py:
```python
import socket

# write bytes to test file
byte_data = b"\xa9\xf6\x34\x08\x42\x2a\x9e\x1c\x0c\x03\xa8\x08\x94\x70\xbb\x8d\xaa\xdc\x6d\x7b\x24\xff\x7f\x24\x7c\xda\x83\x9e\x92\xf7\x07\x1d\x02\x63\x90\x2e\xc1\x58\x00\x00\xd0\xb4\x58\x6d\xb4\x55\x00\x00\x20\xea\x78\x19\x4a\x7f\x00\x00"

with open("test", 'wb') as file:
	file.write(byte_data)

# Set up the server
host = '10.0.2.15'  # The server's IP address
port = 1337     	# The port to listen on

# Create a TCP/IP socket
server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# Bind the socket to the address and port
server_socket.bind((host, port))

# Listen for incoming connections
server_socket.listen()

print(f'Server listening on {host}:{port}')

while True:
	# Wait for a connection
	client_socket, client_address = server_socket.accept()
	print(f'Connection from {client_address}')

	# Handle the connection
	byte_arr_32 = bytearray([0x8d, 0xec, 0x91, 0x12, 0xeb, 0x76, 0x0e, 0xda, 0x7c, 0x7d, 0x87, 0xa4, 0x43, 0x27, 0x1c, 0x35, 0xd9, 0xe0, 0xcb, 0x87, 0x89, 0x93, 0xb4, 0xd9, 0x04, 0xae, 0xf9, 0x34, 0xfa, 0x21, 0x66, 0xd7])
	byte_arr_12 = bytearray([0x11, 0x11, 0x11, 0x11, 0x11, 0x11, 0x11, 0x11, 0x11, 0x11, 0x11, 0x11])
	byte_arr_4 = bytearray([0xf2, 0xf2, 0xf2, 0xf2])

	client_socket.send(byte_arr_32)	# 32 bytes
	client_socket.send(byte_arr_12)	# 12 bytes
	client_socket.send(byte_arr_4)	# 4 bytes
	client_socket.send(b'test')

	# Receive data from clients
	data = client_socket.recv(1024)  # Buffer size is 1024 bytes
	print(f"Received message from: {data.decode('utf-8', errors='replace')}")
```
<br />

Set up the server IP address as shown earlier, run the server.py script.
```bash
sudo ip addr add 10.0.2.15/24 dev eth0
python server.py
```
<br />

In another terminal tab, run
```bash
sudo strace -u root ./test -e trace=network -f -d
```
![img](./2024-FlareOn11/flareon2024-5-12.png)<br /><br />

Flag: ```supp1y_cha1n_sund4y@flare-on.com```<br /><br />

## 6

> bloke2
>
> You've been so helpful lately, and that was very good work you did. Yes, I'm going to put it right here, on the refrigerator, very good job indeed. You're the perfect person to help me with another issue that come up. One of our lab researchers has mysteriously disappeared. He was working on the prototype for a hashing IP block that worked very much like, but not identically to, the common Blake2 hash family. Last we heard from him, he was working on the testbenches for the unit. One of his labmates swears she knew of a secret message that could be extracted with the testbenches, but she couldn't quite recall how to trigger it. Maybe you could help?
>
> File: bloke2.7z

<br />
![img](./2024-FlareOn11/flareon2024-6-1.png)<br />
Read the README.md. We do not need to understand Blake2 hash as this is a modified version.<br /><br />

How to build and run the testbenches:
```bash
$ sudo apt install iverilog
$ cd bloke2
$ make
iverilog -g2012 -o bloke2.out bloke2.v f_sched.v f_unit.v g_over_2.v g.v g_unit.v data_mgr.v bloke2s.v bloke2b.v

$ make tests
```
![img](./2024-FlareOn11/flareon2024-6-2.png)<br /><br />

Notice this TEST_VAL in data_mgr.v:<br />
![img](./2024-FlareOn11/flareon2024-6-3.png)<br /><br />

It is used in some operations in this line here, involving h_in and tst value. 
The $display(...) lines were originally commented out but I uncommented them all (in this file, 1 not in the pic).<br />
![img](./2024-FlareOn11/flareon2024-6-4.png)<br /><br />

Upon running “make tests”, notice that in all iterations of the 3 test cases, tst is 0.<br />
![img](./2024-FlareOn11/flareon2024-6-5.png)
- If tst = 0, {(W*16){0}} will be all 0s, meaning TEST_VAL & {(W*16){0}} will be 0. So h_in will remain unchanged.
- If tst = 1, {(W*16){1}} will be all 1s, meaning it will have no effect on TEST_VAL, and h_in will be XORed with TEST_VAL. This is what we want to happen.
<br /><br />

I went ahead and commented out all except the line we are concerned with, to monitor the tst value t.<br />
![img](./2024-FlareOn11/flareon2024-6-6.png)<br /><br />

This is the output and you can clearly see that the tst values across the board are all 0s.<br />
![img](./2024-FlareOn11/flareon2024-6-7.png)<br /><br />

To get tst = 1, look more into data_mgr.v. We see that the tst signal is affected by the finish signal, which is controlled in bloke2b_tb.v.<br />
![img](./2024-FlareOn11/flareon2024-6-8.png)<br /><br />

We notice in bloke2b_tb.v, the finish signal is not set correctly.<br />
The correct way is for the finish signal to be asserted (1) for one clock cycle and then deasserted (0).
![img](./2024-FlareOn11/flareon2024-6-9.png)<br /><br />

We can infer from the other lines where the finish signal is set correctly.<br />
![img](./2024-FlareOn11/flareon2024-6-10.png)<br /><br />

Make the change accordingly.<br />
![img](./2024-FlareOn11/flareon2024-6-11.png)<br /><br />

![img](./2024-FlareOn11/flareon2024-6-12.png)<br /><br />

Flag: ```please_send_help_i_am_trapped_in_a_ctf_flag_factory@flare-on.com```<br /><br />