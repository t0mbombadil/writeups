# Description

![image](https://user-images.githubusercontent.com/18641572/214147188-333fb0e4-d1a3-46ac-a170-989470cb4b67.png)

Type: Forensics (?)


# Solution


## Inspection of file
So as a only resource in challenge beside description, we have a .pcapng file. We can take look inside with Wireshark

![image](https://user-images.githubusercontent.com/18641572/214147574-b2a0656e-a74e-48d9-9db7-1620b7df793e.png)

Around 100 different type requests there, many DNS & ICMP. 

## Quick googling
And not sure what we are looking for. But challenge description is
telling us that `Sir vignere came to my dreams and sent me this packet capture and told me to find the flag from it which is the key to my success`.
Who the hell is Sir vignere? Name sounds specific enough to be a keyword hint that are usually left in challenge's descriptions.

Quick googling shows that it was a great shot - https://en.wikipedia.org/wiki/Vigen%C3%A8re_cipher . 
Vigenère is a cipher, a little more complicated version of Ceasar cipher (and notably named after actual sir Vigenère).

Ok, so by quick educated guess it seems that we are looking for flag encrypted with this cipher in our file, because we have no other resources provided.


## Looking through file
I tried to quickly go through file to with Wireshark, using find feature, to see if there is some worth checking frames, 
using keywords (case insensitive) like `kctf` / `ctf` / `flag` / `vignere` / `key`, without any luck. 

So having no better idea, I started going quickly to each request, with just a glance at frame content, to see if there is something that could catch my attention.
And only thing that I spotted, that seems to notably differ between same type of requests, was actually this domain name in DNS queries:

<img width="655" alt="image" src="https://user-images.githubusercontent.com/18641572/214150175-b087c3d5-59be-438d-8622-13003d566d1e.png">

Quite unusual domain. So I filtered just for similar DNS queries and seems I got something:
![image](https://user-images.githubusercontent.com/18641572/214150474-00e84e8e-9d0f-44b8-b537-ea3df5765319.png)

Queried domain names were different, and there was enough of them to maybe create some kind of string, that maybe could contain flag. I did not want to
write them by hand even if that would be faster than finding how to do it with some kind a program, because that's not why we have computers to do stuff by hand.

## Parsing file in terminal

So I turned my attention to wireshark terminal version (?) `tshark` . I was able to quickly find out the way how to get just proper field from DNS queries
```bash
% tshark -r find-me.pcapng -T fields -e dns.qry.name -Y "dns.flags.response eq 0"
knightctf.com
knightctf.com
knightctf.com
knightctf.com
knightctf.com
knightctf.com
knightctf.com
knightctf.com
knightctf.com
knightctf.com
knightctf.com
knightctf.com
V.knightctf.com
V.knightctf.com
B.knightctf.com
C.knightctf.com
T.knightctf.com
H.knightctf.com
t.knightctf.com
v.knightctf.com
M.knightctf.com
V.knightctf.com
9.knightctf.com
t.knightctf.com
c.knightctf.com
j.knightctf.com
N.knightctf.com
h.knightctf.com
X.knightctf.com
2.knightctf.com
V.knightctf.com
u.knightctf.com
M.knightctf.com
F.knightctf.com
9.knightctf.com
o.knightctf.com
a.knightctf.com
z.knightctf.com
N.knightctf.com
f.knightctf.com
a.knightctf.com
T.knightctf.com
B.knightctf.com
o.knightctf.com
f.knightctf.com
Q.knightctf.com
=.knightctf.com
=.knightctf.com
```


Yep again looks like we got something. So I wanted to get just first part of the domain, those single characters. 

After few attempts to join it together to single strin - finally with query:
```bash
% tshark -r find-me.pcapng -T fields -e dns.qry.name -Y "dns.flags.response eq 0" | grep -Ev "^knight" | sed 's/.knightctf.com//g' | xargs | sed 's/ //g'
VVBCTHtvMV9tcjNhX2VuMF9oazNfaTBofQ==
```

And since result strings ends with two `==` it looks like base64 padding. So trying to decode it:
```bash
% echo "VVBCTHtvMV9tcjNhX2VuMF9oazNfaTBofQ==" | base64 -d
UPBL{o1_mr3a_en0_hk3_i0h}%
```

## Decryption


And bingo! That's definetely a flag looking at format. But we still need to fight with Sir Vigenère to decrypt it.
And important thing we don't have a key. And since I read it is more complicated version of Caesar cipher, I just assumed that it
is not really strong cipher, so probably we don't need a key. 

Maybe just friendly bruteforce..

Unfortunately ciphey does not like saying that what if found is not what you are looking for.. 

<img width="438" alt="image" src="https://user-images.githubusercontent.com/18641572/214153829-7af47a2a-5841-4def-872e-1fc3503b54d3.png">

But it did tell, me that I was definetely lookin at Vigenère cipher

<img width="938" alt="image" src="https://user-images.githubusercontent.com/18641572/214154900-5da54017-b418-40a5-ad5b-a366beb4ca90.png">


So after couple of minutes of attemps with different tools, and even writing some quick decryptor (but I was too lazy) I ended up with [Vigenère Cipher Decoder and Solver
](https://www.boxentriq.com/code-breaking/vigenere-cipher). 


I tried with autosolving options of this web app, but again hit the issue I got with ciphey - it looked for english like words in text, and showed you only most probable decrypted texts, that looked like english

<img width="838" alt="image" src="https://user-images.githubusercontent.com/18641572/214154569-3d177e7a-8e9d-452f-9774-a99b49395b48.png">


In both I have not found option to filter it, by format I was looking for, so flag format. 

But I noticed that when I put single letter in key area, I can make first decrypted letter to be `K` as I want, by quickly going through different characters on keyboard
So little manual bruteforce. Then second letter bruteforce and decrypted text starts with `KC` as I want. 

And I could probably at this point make a tool doing if for me much quicker, but with third letter I knew what the key would be, and my guess was successful

![image](https://user-images.githubusercontent.com/18641572/214155410-815ae942-f11e-4b92-b009-1d81c8effdd8.png)



Voila! C'est une clé envoyée par Monsieur Vigenère.


