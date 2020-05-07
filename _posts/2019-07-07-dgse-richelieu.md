---
layout: post
title: dgse - richelieu
categories: ctf
---
<!--more-->

---
## Arrivé sur la page d'accueil
---

On arrive sur une [page](https://www.challengecybersec.fr/) avec un compteur indiquant le nombre de jours restants avant la fin du challenge. Cependant, aucune information explicite sur le challenge à résoudre.

```html
<!doctype html>
<html lang="fr">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1,shrink-to-fit=no">
    <meta name="theme-color" content="#000000">
    <link rel="manifest" href="manifest.json">
    <title>Challenge Richelieu</title>
    <link href="static/css/main.ba9d9d7d.chunk.css" rel="stylesheet">
</head>
<body>
    <noscript>Vous devez activer JavaScript pour utiliser l'application.</noscript>			<script>let login = "rien";
        let password = "nothing";
        if (login === password) {
            document.location="./Richelieu.pdf";
        }
    </script>
</body>
</html>
```

Après un petit tour par le code source du site, on peut vois l'existence d'un `./Richelieu.pdf` dans l'arborescence du site.

```bash
$ wget https://www.challengecybersec.fr/Richelieu.pdf
```

Ce fichier est un PDF assez volumineux avec plus de **9 Mo.**

---
## PDF
---

Composé d'un grand nombre de page (**364**). Cependant, il n'y a du texte (visible) que sur la première page. On se demande alors quelle est l'utilité de toutes les autres pages. On remarque avec notre souris que du texte écrit en très petite police occupe toutes les pages blanches. Il faudrait donc extraire tout le texte de ce PDF.

```bash
$ pdftotext Richelieu.pdf text_extracted.txt
```

Nous allons donc extraire tout le texte du fichier `Richelieu.pdf` vers `text_extracted.txt` en utilisant `pdftotext`. 

```bash
$ cat text_extracted.txt
Richelieu
L’histoire de la cryptologie serait incomplète sans citer Richelieu qui, dès
1624, sut recruter les meilleurs spécialistes en mathématiques. Il a ainsi pu
fonder, par la suite, ce qui est considéré aujourd’hui comme l'un des tout
premiers bureaux du chiffre en Europe.
Lors du siège de La Rochelle (1627-1628), la cryptanalyse des messages des
Huguenots lui permit d’anticiper l’arrivée des Anglais venus aider ces derniers
par la mer. En octobre 1628, la ville finira par capituler sans condition.
/9j/2wBDADIiJSwlHzIsKSw4NTI7S31RS0VFS5ltc1p9tZ++u7Kfr6zI4f/zyNT/16yv+v/9////
////wfD/////////////wgALCA20CD4BAREA/8QAGQABAQEBAQEAAAAAAAAAAAAAAAECAwQF/9oA
CAEBAAAAAeFzUWUsiyzUFlgFgChLKud5lms2KluaLBNZs1neNSaRclSxqWLJ0xYVNQFyusxQlA1M
#...
x1x1eAsAAQToAwAABOgDAABQSwECHgMUAAkACACrg51OHIIIsGZHXwCfQ18ADwAYAAAAAAAAAAAA
pIFlCgAAbHNiX1JHQi5wbmcuZW5jVVQFAANCCsdcdXgLAAEE6AMAAAToAwAAUEsFBgAAAAAGAAYA
RAIAACRSXwAAAA==
```

On peut retrouver le texte déjà visible sur le PDF, accompagné de tout le texte cachait, qui été en blanc sur le PDF. On a sûrement à faire à de la `base64`.

```bash
$ cat text_extracted.txt | wc -l
117798
$ cat text_extracted.txt | sed -n 9,117797p > file.b64
$ cat file.b64 | base64 -di > file.bin
# -i, --ignore-garbage: when decoding, ignore non-alphabet characters
$ file file.bin
file.bin: JPEG image data, progressive, precision 8, 2110x3508, frames 1
$ cp file.bin file.jpg
```

On retire les 9 premières lignes et on crée le fichier `file.b64`. On le décode avec `base64 -id`. On remarque que l'on a à faire à un JPG, on le renomme alors. On voit alors une image de Richelieu. Il y a sûrement d'autres choses dans un fichier si grand.

---
## ZIP
---

```bash
$ binwalk file.jpg
DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
445628        0x6CCBC         Zip archive data, encrypted, name: .bash_history
445979        0x6CE1B         Zip archive data, encrypted, name: suite.zip
446405        0x6CFC5         Zip archive data, encrypted, name: prime.txt
446943        0x6D1DF         Zip archive data, encrypted, name: public.key
447670        0x6D4B6         Zip archive data, encrypted, name: motDePasseGPG.txt.enc
448289        0x6D721         Zip archive data, encrypted, name: lsb_RGB.png.enc
6693156       0x662124        End of Zip archive, footer length: 22
```

Avec `binwalk` on peut voir la présence d'une archive qui est composée de plusieurs fichiers intéressants.

```bash
$ binwalk --dd=".*" file.jpg
$ cd _file.jpg.extracted/
$ ll
-rw-r--r-- 1 adel adel      22 mai   24 19:58 662124
-rw-r--r-- 1 adel adel 6247550 mai   24 19:58 6CCBC
$ file 662124 
662124: Zip archive data (empty)
$ file 6CCBC 
6CCBC: Zip archive data, at least v2.0 to extract
$ cp 6CCBC file.zip
$ unzip file.zip 
Archive:  file.zip
[file.zip] .bash_history password:
```

Après extraction, on se retrouve avec 2 fichiers, un qui ne sert sûrement à rien (`662124`) et un qui parait être utile: `6CCBC`. C'est une archive chiffrée bien sûr. Il faut trouver un mot de passe. Après quelques recherches sans succès, revenons aux basiques.

```bash
$ strings file.zip
# ...
a#W|+
~8!6
.bash_historyUT
Le mot de passePK
suite.zipUT
de cette archivePK
prime.txtUT
est : DGSE{t.D=@Bx^A%n9FQB~_VL7Zn8z=:K^4ikE=j0EGHqI}PK
public.keyUT
motDePasseGPG.txt.encUT
lsb_RGB.png.encUT
```

Le mot de passe est sûrement sous nos yeux...

```bash
$ mkdir zip_files
$ unzip -P "DGSE{t.D=@Bx^A%n9FQB~_VL7Zn8z=:K^4ikE=j0EGHqI}" file.zip -d zip_files
Archive:  file.zip
  inflating: zip_files/.bash_history  
 extracting: zip_files/suite.zip     
  inflating: zip_files/prime.txt     
  inflating: zip_files/public.key    
 extracting: zip_files/motDePasseGPG.txt.enc  
  inflating: zip_files/lsb_RGB.png.enc 
```

Nous avons accès à tous les fichiers du zip.

```bash
$ cd zip_files/
$ ll
-rw-r--r-- 1 adel adel     578 avril 26 17:20 .bash_history
-rw-r--r-- 1 adel adel 6243231 avril 29 16:29 lsb_RGB.png.enc
-rw-r--r-- 1 adel adel     512 avril 29 16:29 motDePasseGPG.txt.enc
-rw-r--r-- 1 adel adel     868 avril 29 16:29 prime.txt
-rw-r--r-- 1 adel adel     800 avril 29 16:29 public.key
-rw-r--r-- 1 adel adel     331 avril 29 16:28 suite.zip
```

Essayons donc de les parcourir et de faire une analyse rapide.

#### `.bash_history`

```bash
$ file .bash_history 
.bash_history: ASCII text
$ cat .bash_history
 # gpg encryption
 1337  gpg -o lsb_RGB.png.enc --symmetric lsb_RGB.png
 
 # create file with futur GPG password ?
 1338  vim motDePasseGPG.txt
 
 # gen 4096 bits rsa private key
 1339  openssl genrsa -out priv.key 4096
 
 # gen public key
 1340  openssl rsa -pubout -out public.key -in priv.key
 
 # extract prime 1 from priv key ?
 1341  openssl rsa -noout -text -in priv.key | grep prime1 -A 18 > prime.txt
 # -A NUM, --after-context=NUM: Print NUM lines of trailing context after matching lines.
 
 # change file content with sed !!
 1342  sed -i 's/7f/fb/g' prime.txt
 1343  sed -i 's/e1/66/g' prime.txt
 1344  sed -i 's/f4/12/g' prime.txt
 1345  sed -i 's/16/54/g' prime.txt
 1346  sed -i 's/a4/57/g' prime.txt
 1347  sed -i 's/b5/cd/g' prime.txt
 
 # encryption of motDePasseGPG.txt with public.key
 1348  openssl rsautl -encrypt -pubin -inkey public.key -in motDePasseGPG.txt -out motDePasseGPG.txt.enc
```

Le fichier `.bash_history` sous Linux contient tout l'historique des commandes qui ont étaient exécutaient sur le terminal.

#### `motDePasseGPG.txt.enc `

```bash
$ file motDePasseGPG.txt.enc 
motDePasseGPG.txt.enc: data
$ cat motDePasseGPG.txt.enc 
# ciphered data
```

On comprend rapidement que notre but sera de retrouver la clé privée utilisée pour réaliser le déchiffrement du fichier `motDePasseGPG.txt.enc`.

#### `prime.txt`

```bash
$ file prime.txt 
prime.txt: ASCII text
$ cat prime.txt 
prim66:
    00:fb:40:dc:44:ba:03:d1:53:42:f7:59:08:e0:f9:
    30:05:96:64:4a:de:94:68:5e:08:e2:8c:9a:b1:64:
    0c:2f:62:c2:9a:b9:a2:39:82:4b:9e:be:eb:76:ae:
    6d:87:21:a3:5e:9e:d9:8d:7e:57:38:3e:59:09:34:
    a5:78:cd:f7:2e:89:5d:5c:37:52:ea:fd:f6:31:cc:
    ba:d2:d9:60:e4:45:1d:67:76:d2:1f:12:9c:9d:c9:
    b1:90:45:51:ed:d2:fb:dd:b6:74:b4:99:fb:b1:0a:
    d9:b7:c2:be:8b:57:07:22:0a:8e:3a:36:ff:6d:c1:
    1d:63:93:af:cb:4e:c0:47:9f:65:bf:df:e3:f0:5f:
    1e:98:61:45:74:ec:36:a7:a5:b1:f1:8d:3d:97:6b:
    5a:82:49:09:00:08:0d:9d:c2:74:57:4e:30:a1:39:
    68:2f:22:34:71:13:aa:3b:f2:20:4f:8e:10:eb:d4:
    d0:9b:cd:8c:c2:53:5f:9d:71:13:0c:0f:21:b6:6e:
    13:39:40:d3:a6:b1:eb:74:ad:dd:0a:29:14:81:b1:
    90:ad:e0:53:f0:89:c8:00:fe:dc:ad:56:59:fc:28:
    1d:c0:cf:5e:08:c0:54:33:24:a3:52:bb:f3:25:10:
    43:c3:73:b8:40:4f:fc:6b:6b:77:bd:5f:22:24:eb:
    fb:15
```

Voici donc le fichier manipulé dans le `.bash_history`, il ne faut pas oublier que ce dernier à été modifié après avoir été créé. Il faut donc penser à le modifier pour retrouver ses valeurs d'origines.

#### `public.key`

```bash
$ file public.key 
public.key: ASCII text
$ cat public.key 
-----BEGIN PUBLIC KEY-----
MIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAzV+KJMdgUAiJejySLA6B
Lnad4KRkQsNQy3jHhoU589OKrICz5qUGYFkQ6FmYBrTR0Ujy9rgdoEeWqKWu4Y8p
6D4Wd1oqCgCHBUH2V07RQ4Y2rgoMEW4HEE9I9yCUhjo4aeHI/CIGJyeJYvsihz4x
VvGOVd7JTpcAZOx/Tg6IRUAS4v1d/l+NGb8XD5zLP0bg/RAZvLAtkIOgcDxhf5lj
eeZHg1SnOubmrLzh9DM+z68kNmo+l30808v+jYo4e9h2v9q4SI9vR78fvjMBD9LX
4itNsuVneDzgtgbbhrk3WXFMT2OWp/ufdMQCEEOw89RtJjPr1DqHeGPffWgPUGWH
wRndZBAMqDHOKvM9lRtSTF8GtJ9b8ss4HnQYGTDQaoBQXAar1b9IcPDJ+1gb2A26
iJZgY5+Tbt6o/l0Mnq5YBi7WkyUlg8ccx4K6YT4BQ45ptD+eZOyoT56gToEa17Oe
/Xh20ba1AcT0iszm8kI59sBAKHiBNc2Iw9Fb4PLrt96enBmnqTA3AF7gqaZAutoz
LsDQXunwioMjVKBIepJ9XogGbiVp5sXUaI5CK/oLJ8YXHG178Cm/2RZXUq8ZqnGz
Oh6nC2w3H7IeR/Un2At9BPWCrZ+ZNa9yNoLcAcqYgGIYcN7LetFWSM307xUwFvPm
2HkzuOxUz6H9+HxGcCCj51MCAwEAAQ==
-----END PUBLIC KEY-----
```

Nous possédons également, la clé publique utilisée pour le chiffrement.

#### `lsb_RGB.png.enc`

```bash
$ file lsb_RGB.png.enc 
lsb_RGB.png.enc: GPG symmetrically encrypted data (AES256 cipher)
```

D'après le `.bash_history`, ce fichier a été chiffré, avec GPG.

#### `suite.zip`

```bash
$ file suite.zip 
suite.zip: Zip archive data, at least v2.0 to extract
$ unzip suite.zip 
Archive:  suite.zip
[suite.zip] suite.txt password:
```

Prochaine étape du challenge.

---
## Attaque sur RSA
---

Commençons par essayer d'attaquer la clé publique, même si je pense qu'il y a peu de chances d'y arriver.

```bash
$ rsactftool.py --publickey ./public.key --private
Killed
$ rsactftool.py --publickey ./public.key --uncipherfile ./motDePasseGPG.txt.enc
Killed 
```

Pas possible d'attaquer la clé publique (module bien trop grand).

```bash
$ cat prime.txt 
prim66:
    00:fb:40:dc:44:ba:03:d1:53:42:f7:59:08:e0:f9:
    30:05:96:64:4a:de:94:68:5e:08:e2:8c:9a:b1:64:
    0c:2f:62:c2:9a:b9:a2:39:82:4b:9e:be:eb:76:ae:
    6d:87:21:a3:5e:9e:d9:8d:7e:57:38:3e:59:09:34:
    a5:78:cd:f7:2e:89:5d:5c:37:52:ea:fd:f6:31:cc:
    ba:d2:d9:60:e4:45:1d:67:76:d2:1f:12:9c:9d:c9:
    b1:90:45:51:ed:d2:fb:dd:b6:74:b4:99:fb:b1:0a:
    d9:b7:c2:be:8b:57:07:22:0a:8e:3a:36:ff:6d:c1:
    1d:63:93:af:cb:4e:c0:47:9f:65:bf:df:e3:f0:5f:
    1e:98:61:45:74:ec:36:a7:a5:b1:f1:8d:3d:97:6b:
    5a:82:49:09:00:08:0d:9d:c2:74:57:4e:30:a1:39:
    68:2f:22:34:71:13:aa:3b:f2:20:4f:8e:10:eb:d4:
    d0:9b:cd:8c:c2:53:5f:9d:71:13:0c:0f:21:b6:6e:
    13:39:40:d3:a6:b1:eb:74:ad:dd:0a:29:14:81:b1:
    90:ad:e0:53:f0:89:c8:00:fe:dc:ad:56:59:fc:28:
    1d:c0:cf:5e:08:c0:54:33:24:a3:52:bb:f3:25:10:
    43:c3:73:b8:40:4f:fc:6b:6b:77:bd:5f:22:24:eb:
    fb:15
```

Nous devons donc travailler sur ce fichier.

Rappels sur RSA:
$$
p*q = n \\
p: prime 1 \\
q: prime 2 \\
n: modulus \\
$$
Tout d'abord, nous devons nettoyer `prime.txt` afin de retrouver la valeur de $p$.

```bash
1341  openssl rsa -noout -text -in priv.key | grep prime1 -A 18 > prime.txt
1342  sed -i 's/7f/fb/g' prime.txt
1343  sed -i 's/e1/66/g' prime.txt
1344  sed -i 's/f4/12/g' prime.txt
1345  sed -i 's/16/54/g' prime.txt
1346  sed -i 's/a4/57/g' prime.txt
1347  sed -i 's/b5/cd/g' prime.txt
```

La première commande permet de récupérer les 18 lignes se trouvant après la chaîne `prime1`, ce qui correspond donc à notre $p$.

```bash
$ cp prime.txt prime_good.txt
$ sed -i 's/cd/b5/g' prime_good.txt
$ sed -i 's/57/a4/g' prime_good.txt
$ sed -i 's/54/16/g' prime_good.txt
$ sed -i 's/12/f4/g' prime_good.txt
$ sed -i 's/66/e1/g' prime_good.txt
$ sed -i 's/fb/7f/g' prime_good.txt
```

Les lignes suivantes, réalisent des substitutions sur le fichier à l'aide de la commande `sed`, il faut donc les inverser pour connaître le contenu originel de `prime.txt`.

```bash
$ cat prime_good.txt 
prime1:
    00:7f:40:dc:44:ba:03:d1:53:42:f7:59:08:e0:f9:
    30:05:96:64:4a:de:94:68:5e:08:e2:8c:9a:b1:64:
    0c:2f:62:c2:9a:b9:a2:39:82:4b:9e:be:eb:76:ae:
    6d:87:21:a3:5e:9e:d9:8d:7e:a4:38:3e:59:09:34:
    a5:78:b5:f7:2e:89:5d:5c:37:52:ea:fd:f6:31:cc:
    ba:d2:d9:60:e4:45:1d:67:76:d2:1f:f4:9c:9d:c9:
    b1:90:45:51:ed:d2:7f:dd:b6:74:b4:99:7f:b1:0a:
    d9:b7:c2:be:8b:a4:07:22:0a:8e:3a:36:ff:6d:c1:
    1d:63:93:af:cb:4e:c0:47:9f:65:bf:df:e3:f0:5f:
    1e:98:61:45:74:ec:36:a7:a5:b1:f1:8d:3d:97:6b:
    5a:82:49:09:00:08:0d:9d:c2:74:a4:4e:30:a1:39:
    68:2f:22:34:71:13:aa:3b:f2:20:4f:8e:10:eb:d4:
    d0:9b:b5:8c:c2:53:5f:9d:71:13:0c:0f:21:b6:6e:
    13:39:40:d3:a6:b1:eb:74:ad:dd:0a:29:14:81:b1:
    90:ad:e0:53:f0:89:c8:00:fe:dc:ad:56:59:fc:28:
    1d:c0:cf:5e:08:c0:16:33:24:a3:52:bb:f3:25:10:
    43:c3:73:b8:40:4f:fc:6b:6b:77:bd:5f:22:24:eb:
    7f:15
```

Voici donc la vrai valeur de notre nombre $prime1$ ou $p$. Nous devons maintenant trouver $q$, qui vaut:
$$
q = \frac{n}{p}
$$
Nous avons le clé publique `public.key`, nous pouvons donc facilement extraire $n$.

```bash
$ openssl rsa -pubin -inform PEM -text -noout -in public.key > modulus.txt
RSA Public-Key: (4096 bit)
Modulus:
    00:cd:5f:8a:24:c7:60:50:08:89:7a:3c:92:2c:0e:
    81:2e:76:9d:e0:a4:64:42:c3:50:cb:78:c7:86:85:
    39:f3:d3:8a:ac:80:b3:e6:a5:06:60:59:10:e8:59:
    98:06:b4:d1:d1:48:f2:f6:b8:1d:a0:47:96:a8:a5:
    ae:e1:8f:29:e8:3e:16:77:5a:2a:0a:00:87:05:41:
    f6:57:4e:d1:43:86:36:ae:0a:0c:11:6e:07:10:4f:
    48:f7:20:94:86:3a:38:69:e1:c8:fc:22:06:27:27:
    89:62:fb:22:87:3e:31:56:f1:8e:55:de:c9:4e:97:
    00:64:ec:7f:4e:0e:88:45:40:12:e2:fd:5d:fe:5f:
    8d:19:bf:17:0f:9c:cb:3f:46:e0:fd:10:19:bc:b0:
    2d:90:83:a0:70:3c:61:7f:99:63:79:e6:47:83:54:
    a7:3a:e6:e6:ac:bc:e1:f4:33:3e:cf:af:24:36:6a:
    3e:97:7d:3c:d3:cb:fe:8d:8a:38:7b:d8:76:bf:da:
    b8:48:8f:6f:47:bf:1f:be:33:01:0f:d2:d7:e2:2b:
    4d:b2:e5:67:78:3c:e0:b6:06:db:86:b9:37:59:71:
    4c:4f:63:96:a7:fb:9f:74:c4:02:10:43:b0:f3:d4:
    6d:26:33:eb:d4:3a:87:78:63:df:7d:68:0f:50:65:
    87:c1:19:dd:64:10:0c:a8:31:ce:2a:f3:3d:95:1b:
    52:4c:5f:06:b4:9f:5b:f2:cb:38:1e:74:18:19:30:
    d0:6a:80:50:5c:06:ab:d5:bf:48:70:f0:c9:fb:58:
    1b:d8:0d:ba:88:96:60:63:9f:93:6e:de:a8:fe:5d:
    0c:9e:ae:58:06:2e:d6:93:25:25:83:c7:1c:c7:82:
    ba:61:3e:01:43:8e:69:b4:3f:9e:64:ec:a8:4f:9e:
    a0:4e:81:1a:d7:b3:9e:fd:78:76:d1:b6:b5:01:c4:
    f4:8a:cc:e6:f2:42:39:f6:c0:40:28:78:81:35:cd:
    88:c3:d1:5b:e0:f2:eb:b7:de:9e:9c:19:a7:a9:30:
    37:00:5e:e0:a9:a6:40:ba:da:33:2e:c0:d0:5e:e9:
    f0:8a:83:23:54:a0:48:7a:92:7d:5e:88:06:6e:25:
    69:e6:c5:d4:68:8e:42:2b:fa:0b:27:c6:17:1c:6d:
    7b:f0:29:bf:d9:16:57:52:af:19:aa:71:b3:3a:1e:
    a7:0b:6c:37:1f:b2:1e:47:f5:27:d8:0b:7d:04:f5:
    82:ad:9f:99:35:af:72:36:82:dc:01:ca:98:80:62:
    18:70:de:cb:7a:d1:56:48:cd:f4:ef:15:30:16:f3:
    e6:d8:79:33:b8:ec:54:cf:a1:fd:f8:7c:46:70:20:
    a3:e7:53
Exponent: 65537 (0x10001)
```

```bash
# autre façon
$ rsactftool.py --dumpkey --key ./public.key
[*] n: 837849563862443268467145186974119695264713699736869090645354954749227901572347301978135797019317859500555501198030540582269024532041297110543579716921121054608494680063992435808708593796476251796064060074170458193997424535149535571009862661106986816844991748325991752241516736019840401840150280563780565210071876568736454876944081872530701199426927496904961840225828224638335830986649773182889291953429581550269688392460126500500241969200245489815778699333733762961281550873031692933566002822719129034336264975002130651771127313980758562909726233111335221426610990708111420561543408517386750898610535272480495075060087676747037430993946235792405851007090987857400336566798760095401096997696558611588264303087788673650321049503980655866936279251406742641888332665054505305697841899685165810087938256696223326430000379461379116517951965921710056451210314300437093481577578273495492184643002539393573651797054497188546381723478952017972346925020598375000908655964982541016719356586602781209943943317644547996232516630476025321795055805235006790200867328602560320883328523659710885314500874028671969578391146701739515500370268679301080577468316159102141953941314919039404470348112690214065442074200255579004452618002777227561755664967507
[*] e: 65537
```

Nous connaissons maintenant la valeur de $n$, not:re $modulus$. C'est la forme en base 10 qui va nous intéresser.

```bash
$ python3
>>> p = ''.join(open('prime_good.txt').readlines())
>>> import re
>>> p = int(re.sub('[^0-9a-f]','',p[6:]), 16)
>>> import gmpy
>>> gmpy.is_prime(p)
0
```

Sauf que, problème, on se rend compte qu'après avoir passé notre `prime_good.txt` en base 10, celui-ci n'est pas premier... :(

Après réflexions, l'inversion des `sed` que l'on a fait plus haut n'est peut-être pas suffisante. Par exemple, si nous avons la chaîne ; `xxxxbbxx` et que l'on applique `sed -i s/bb/xx/g`, ce qui nous donne `xxxxxxxx`. On pense alors qu'il suffit d'inverser le `sed` en `sed -i s/xx/bb/g` pour retrouver le message originel. Sauf que non, on se retrouve avec `bbbbbbbb`. C'est sûrement le même problème avec notre `prime_good.txt`, l'inversion du `sed` ne garantis pas de retrouver le vrai $prime$ d'origine.

```bash
$ sed -i 's/cd/b5/g' prime_good.txt # 2 occurrences de 'b5'
$ sed -i 's/57/a4/g' prime_good.txt # 3 occurrences de 'a4'
$ sed -i 's/54/16/g' prime_good.txt # 1 occurrence de '16'
$ sed -i 's/12/f4/g' prime_good.txt # 1 occurrence de 'f4'
$ sed -i 's/66/e1/g' prime_good.txt # on ignore car présent au début
$ sed -i 's/fb/7f/g' prime_good.txt # 4 occurrences de '7f' dans godd_prime.txt
```

Il faut donc tester toutes les combinaisons possibles, chaque occurrence peut prendre 2 valeurs, soit garder sa valeur actuelle, soit revenir à sa valeur d'origine. Nous allons tester toutes les combinaisons possibles appliquées au fichier `prime_good.txt` puis tester à chaque fois si notre $q$ est premier.

Voici le script qui va effectuer cette tâche :

```python
#/usr/bin/env python3

import re
import itertools
import subprocess as sp
import gmpy

# read modulus (n)
n = ''.join(re.findall('[0-9a-f]{2}:.*:[0-9a-f]{2}', ''.join(open('modulus.txt').readlines()), re.DOTALL)).replace('\n', '').replace(' ', '').replace(':', '')
n = int(n,16)

# read prime to ajust
f = ''.join(open('prime_good.txt').readlines())

# hex to ajust
tab_reg = ['7f', 'f4', '16', 'a4', 'b5']

# find position for each previous hex
positions_occ = []
for reg in tab_reg:
    tmp = []
    for m in re.compile(reg).finditer(f):
        tmp.append(m.start())
    positions_occ.append(tmp)
 # we ill use binary to find all combinaisons
combinaisons_7f = list(itertools.product([0,1], repeat=len(positions_occ[0])))
combinaisons_f4 = list(itertools.product([0,1], repeat=len(positions_occ[1])))
combinaisons_16 = list(itertools.product([0,1], repeat=len(positions_occ[2])))
combinaisons_a4 = list(itertools.product([0,1], repeat=len(positions_occ[3])))
combinaisons_b5 = list(itertools.product([0,1], repeat=len(positions_occ[4])))

# adapt '7f'
for c_7f in combinaisons_7f:
    pos_7f = positions_occ[0]
    new_7f_0 = '7f' if c_7f[0] == 0 else 'fb'
    new_7f_1 = '7f' if c_7f[1] == 0 else 'fb'
    new_7f_2 = '7f' if c_7f[2] == 0 else 'fb'
    new_7f_3 = '7f' if c_7f[3] == 0 else 'fb'
    f = f[0:pos_7f[0]] + new_7f_0 + f[pos_7f[0]+2:pos_7f[1]] + new_7f_1 + f[pos_7f[1]+2:pos_7f[2]] + new_7f_2 + f[pos_7f[2]+2:pos_7f[3]] + new_7f_3 + f[pos_7f[3]+2:]
    
    # adapt 'f4'
    for c_f4 in combinaisons_f4:
        pos_f4 = positions_occ[1]
        new_f4_0 = 'f4' if c_f4[0] == 0 else '12'
        f = f[0:pos_f4[0]] + new_f4_0 + f[pos_f4[0]+2:]
        
        # adapt '16'
        for c_16 in combinaisons_16:
            pos_16 = positions_occ[2]
            new_16_0 = '16' if c_16[0] == 0 else '54'
            f = f[0:pos_16[0]] + new_16_0 + f[pos_16[0]+2:]
            
            # adapt 'a4'
            for c_a4 in combinaisons_a4:
                pos_a4 = positions_occ[3]
                new_a4_0 = 'a4' if c_a4[0] == 0 else '57'
                new_a4_1 = 'a4' if c_a4[1] == 0 else '57'
                new_a4_2 = 'a4' if c_a4[2] == 0 else '57'
                f = f[0:pos_a4[0]] + new_a4_0 + f[pos_a4[0]+2:pos_a4[1]] + new_a4_1 + f[pos_a4[1]+2:pos_a4[2]] + new_a4_2 + f[pos_a4[2]+2:]
                	
                    # adapt 'b5'
                for c_b5 in combinaisons_b5:
                    pos_b5 = positions_occ[4]
                    new_b5_0 = 'b5' if c_b5[0] == 0 else 'cd'
                    new_b5_1 = 'b5' if c_b5[1] == 0 else 'cd'
                    f = f[0:pos_b5[0]] + new_b5_0 + f[pos_b5[0]+2:pos_b5[1]] + new_b5_1 + f[pos_b5[1]+2:]

                    # final test
                    p = f.replace('\n', '').replace(' ', '').replace('prime1', '').replace(':', '')
                    p = int(p,16)
                    if gmpy.is_prime(p) == 1 or gmpy.is_prime(p) == 2:
                        q = n//p
                        if p*q == n:
                            print('p\n', p)
                            print('q\n', q)
                            exit(0)
```

Code pas du tout optimal, mais fonctionnel :)

```bash
$ python3 ./find_good_prime.py 
p
 31717798413454838971739311391870214101486054474438584033384974797696836002786889741922328038249283935148167589396475764515984792248325276562635675483085593307632384945480084324354199386711259069590701728377654610059849152461565385315663116675949927841648944186909571007960100266136674008112969369512988507367569334838093403144731951113528040013763615354283491955226704334394856253005551393545293491547587439064269735583121706098798182168292621767902095641352443871167789104818694502605762517497996162527138366803593402358312011933528177646501311411008602369245268966975849244259437732574655956250792264392115994984213
q
 26415754111957012456882978698568998595543408604540679602012008905307229528398419508696699258861541673208334031720118065722015191735056250991124159829757615035331104814626815428071461389740988358621575175288995137250131479702118666935818278843603742157096854979085966659730153703721295913600711482578441198179188796849780101926878867272222747848775311501107737838687624647749187764563156054858429897018261077616060929102880860789449995240869652963349913062375402021334173237153511868665597034141167420444312865576543827395764456254276820652449103574192677537410783040227047254038577626347718324544296319787177388746439
```

Nous avons maintenant notre $p$ et $q$ qui sont premiers.

```bash
$ rsatool.py -o priv.key -p 3171779841... -q 2641575411...
# ...
Saving PEM as priv.key
```

Avec `rsatool.py`nous créons notre clé privée connaissant $p$ et $q$.

```bash
$ openssl rsautl -decrypt --inkey priv.key -in motDePasseGPG.txt.enc -out motDePasseGPG.txt
$ cat motDePasseGPG.txt
DGSE{Ti,%yei3=stlh_,5@pIrrMU.^mJC:luYbt1Qe_-Y}
```

On déchiffre ensuite avec `openssl`.

```bash
 1337  gpg -o lsb_RGB.png.enc --symmetric lsb_RGB.png
```

À ce stade nous n'avons toujours pas utilisé cette commande, étrange...

```bash
$ unzip -d suite_files/ -P "DGSE{Ti,%yei3=stlh_,5@pIrrMU.^mJC:luYbt1Qe_-Y}" suite.zip
Archive:  suite.zip
   skipping: suite.txt               incorrect password
```

Et non, ce mot de passe ne permet pas de déchiffrer le zip `suite.zip`.

---
## GPG
---

```bash
 1337  gpg -o lsb_RGB.png.enc --symmetric lsb_RGB.png
```

En regardant la commande, on se rend compte, que nous avons à faire à un chiffrement `symetric`. Alors que naturellement, **GPG** est un système de chiffrement asymétrique avec une notion de clé privée et publique. Pour du symétrique, il ne faut que le mot de passe pour déchiffrer, pas besoin de clé privée.

```bash
$ gpg --batch --yes --passphrase "DGSE{Ti,%yei3=stlh_,5@pIrrMU.^mJC:luYbt1Qe_-Y}" -o lsb_RGB.png --decrypt lsb_RGB.png.enc
gpg: AES256 encrypted data
gpg: encrypted with 1 passphrase
```

En cherchant un peu, on trouve comment déchiffrer du symétrique issu de GPG. On peut alors utiliser notre mot de passe que nous avions trouvé. Le fichier déchiffré est une autre image de Richelieu encore une fois.

Le nom est assez explicite pour comprendre que la prochaine étape consiste à extraire des données en **LSB**.

---
## LSB
---

> In digital steganography, sensitive messages may be concealed by manipulating and storing  information in the least significant bits of an image or a sound file. In the context of an image, if a user were to manipulate the last two bits of a color in a pixel, the value of the color would change at most +/- 3 value places, which is likely to be indistinguishable by the human eye. The user may later recover this information by extracting the  least significant bits of the manipulated pixels to recover the original message. This allows for the storage or transfer of digital information to be kept concealed. 

Pour cette étape, nous devons utiliser l'image `lsb_RGB.png` afin d'extraire des données de celle-ci. Le LSB utilise la valeur des bits de poids faible des couleurs Rouge, Vert et Bleue. La difficulté du LSB réside dans la bonne sélection des bits de poids faibles. Nous utiliserons la librairie `PIL(Python Imaging Library)`.

```python
>>> from PIL import Image
>>> img = Image.open('lsb_RGB.png')
>>> img.size
(1562, 2424) # (y,x)
>>> img.mode
'RGB'
>>> img_data = img.load()
>>> img_data[0,0]
(60, 180, 75)
```

On peut tout d'abord voir que l'on a affaire à un PNG de type **RGB** (et non **RGBA**, car pas de valeur alpha). Chaque pixel correspond alors à un tuple de taille 3, par exemple `(60, 180, 75)` correspond à la valeur du pixel en position `[y=0,x=0]`. La valeur `60` correspond à la quantité de rouge (Red) du pixel (valeur maximale de 255), `180` est la quantité de vert (Green) et `75` la quantité de bleu (Blue).

```python
>>> [bin(i)[2:].zfill(8) for i in img_data[0,0]]
['00111100', '10110100', '01001011']
# zfill(8): prefix with "0" to fixing size
```

Chacune de ces valeurs est codée sur 8 bits. Par exemple, la valeur `63` en base 10, est défini par `00111100` en base 2.

```bash
'00111100' -> '.......0'
'10110100' -> '.......0'
'01001011' -> '.......1'
```

Pour ce premier pixel, la valeur LSB du RED est `0`, celle du GREEN est `0` et celle du BLUE est `1`.

La difficulté réside maintenant dans le fait de savoir si nous devons utiliser les 3 bits précédents ou simplement le LSB du RED, ou du GREEN, ... ou même seulement le LSB du GREEN et BLUE.

Également, nous devons savoir dans quelle direction parcourir les pixels de notre image, soit ligne par ligne (de gauche à droite) ou en colonnes (haut en bas).

Après pas mal de tests, on sait qu'il faut utiliser les 3 LSB, le RED, GREEN et BLUE, de plus on doit parcourir les pixels de notre image en colonne, de haut en bas.

Voici notre extracteur LSB:

```python
#!/usr/bin/env python3

import sys
from PIL import Image

def extract(src):
    
    img = Image.open(src)
    
    dimX,dimY = img.size
    data = img.load()
    all_bits = ''
    
    for y in range(dimX):
        for x in range(dimY):
            r,g,b = data[y,x]
            r_bit = bin(r)[2:].zfill(8)[-1] # RED last bit
            g_bit = bin(g)[2:].zfill(8)[-1] # GREEN last bit
            b_bit = bin(b)[2:].zfill(8)[-1] # BLUE last bit
            all_bits += r_bit + g_bit + b_bit
    
    tab_all_bits = [all_bits[i:i+8] for i in range(0, len(all_bits), 8)]
    
    return''.join([chr(int(c,2)) for c in tab_all_bits])

print('Start')
open('lsb_data', 'wb').write(extract(src='lsb_RGB.png').encode())
print('Done')
```

Ce script se charge de parcourir les pixels de l'image en colonne de haut en bas. Pour chaque pixel, on récupère le dernier bit des 3 composantes (RED, GREEN, BLUE). On construit ensuite un tableau d'octets (8 bits) sur lesquels on récupère leur caractère ASCII. Pour terminer, on enregistre tout ça dans le fichier `lsb_data`.

```bash
# autre manière de trouver les infos LSB
$ zsteg -a ./lsb_RGB.png
...
b1,rgb,lsb,yx       .. text: "00000000: 7f45 4c46 0201 0103 0000 0000 0000 0000  .ELF............\n00000010: 0200 3e00 0100 0000 107f 4400 0000 0000  ..>.......D.....\n00000020
...
```

```bash
$ python3 ./lsb.py 
Start
Done
$ file lsb_data 
lsb_data: ASCII text
$ head lsb_data 
00000000: 7f45 4c46 0201 0103 0000 0000 0000 0000  .ELF............
00000010: 0200 3e00 0100 0000 107f 4400 0000 0000  ..>.......D.....
00000020: 4000 0000 0000 0000 0000 0000 0000 0000  @...............
00000030: 0000 0000 4000 3800 0200 4000 0000 0000  ....@.8...@.....
00000040: 0100 0000 0500 0000 0000 0000 0000 0000  ................
00000050: 0000 4000 0000 0000 0000 4000 0000 0000  ..@.......@.....
00000060: 2487 0400 0000 0000 2487 0400 0000 0000  $.......$.......
00000070: 0000 2000 0000 0000 0100 0000 0600 0000  .. .............
00000080: 2854 0b00 0000 0000 2854 6b00 0000 0000  (T......(Tk.....
00000090: 2854 6b00 0000 0000 0000 0000 0000 0000  (Tk.............
```

Notre `lsb_data` correspond à la vue d'un éditeur hex d'un binaire, plus précisément un `ELF` (Executable and Linkable Format). La prochaine étape est sûrement d'extraire ce binaire et de l'analyser.

Mais avant de continuer, on doit se rappeler que notre script python précédent va réaliser l'extraction des bits LSB sur **TOUS** les pixels de l'image. Mais il y a de grande chance que notre extraction va trop loin. Essayons de voir cela en ouvrant `lsb_data` avec `vim`.

```bash
$ sed -n 18592,18595p lsb_data
000489f0: 414c 4421 0d16 0807 f3b9 2205 c301 7d67  ALD!......"...}g # ligne 18592
00048a00: 0009 0000 9a02 0000 5044 0b00 4918 006b  ........PD..I..k # ligne 18593
00048a10: bc00 0000                                ....				# ligne 18594
q?Ð qdG/A§Ge¿{`Üø)í÷Ö°Ú@¾ïÄt¤#&óP_WlòÛ<7_&f|á}ª						# ligne 18595
                                               oDî$õ¬ËÕzµ'ßØQ
```

On peut donc voir que nous devons couper en ligne 18594 pour ne pas polluer notre extraction.

```bash
$ sed -n 1,18594p lsb_data > lsb_data_clean
$ cat lsb_data_clean | less
00000000: 7f45 4c46 0201 0103 0000 0000 0000 0000  .ELF............
00000010: 0200 3e00 0100 0000 107f 4400 0000 0000  ..>.......D.....
00000020: 4000 0000 0000 0000 0000 0000 0000 0000  @...............
```

Nous avons maintenant une vision en hex propre. Pour continuer nous devons extraire toutes les valeurs hexadécimal des colonnes du milieu. La colonne de gauche représente les positions des valeurs hexa et la colonne de droite, leur traduction ASCII, ces deux informations ne nous sont pas utiles, nous devons les retirer.

```bash
$ cat lsb_data_clean | cut -d' ' -f2-9 > lsb_data_clean_center
$ python3
>>> content = open('lsb_data_clean_center').read()
>>> clean_content = content.replace(' ', '').replace('\n', '')
>>> import binascii
>>> final = binascii.unhexlify(clean_content)
>>> open('elf.bin', 'wb').write(final)
$ file elf.bin
elf.bin: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, no section header
$ chmod +x elf.bin
$ ./elf.bin 
usage : ./elf.bin <mot de passe>
$ ./elf.bin test
Mauvais mot de passe
```

Nous avons maintenant le fichier caché dans le PNG, c'est un exécutable ELF qui prend en paramètre un mot de passe que nous devons sûrement trouver.

---
## Reverse
---

```bash
$ file elf.bin
elf.bin: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, no section header
$ r2 -d ./elf.bin
Process with PID 3036 started...
= attach 3036 3036
bin.baddr 0x00400000
Using 0x400000
Warning: Cannot initialize section headers
Warning: Cannot initialize strings table
Warning: Cannot initialize dynamic strings
asm.bits 64
[0x00447f10]> afl
0x00447f10    3 50           entry0
0x00447f80   28 203          fcn.00447f80
0x00448125   15 107  -> 230  fcn.00448125
0x00448190    3 38   -> 40   fcn.00448190
```

On peut voir qu'il n'y a pas de `main()` dans notre ELF, sûrement parce que ce dernier a été packé, il faut trouver le packer utilisé.

```bash
$ strings ./elf.bin | grep upx
$Info: This file is packed with the ALD executable packer http://upx.sf.net
```

On a donc à faire à un packer UPX, il faut réussir à le unpacker.

```bash
$ upx -d ./elf.bin -o elf_unpacked.bin
# -d: decompress
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2017
UPX 3.94        Markus Oberhumer, Laszlo Molnar & John Reiser   May 12th 2017

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
upx: ./elf.bin: NotPackedException: not packed by UPX

Unpacked 0 files.
```

Sauf que, `upx -d` n'y arrive pas. La décompression ne fonctionne pas.

```bash
$ strings -a ./elf.bin | grep -i upx
$Info: This file is packed with the ALD executable packer http://upx.sf.net $
```

Si nous revenons à cette étape, on peut remarquer que l'URL `http://upx.sf.net` correspond bien à une preuve d'UPX, mais il y a un problème sur `This file is packed with the ALD executable packer`. En effet, à la place de `ALD` on est censé avoir `UPX`. Également, notre `grep -i upx` est censé trouvé la chaîne `UPX!` plusieurs fois.

Après un peu de réflexion, peut-être que la chaîne `UPX` a été remplacé par `ALD` ?

```bash
$ strings -a ./elf.bin | grep -i ald
ALD! 
;AlD
$Info: This file is packed with the ALD executable packer http://upx.sf.net $
$Id: ALD 3.91 Copyright (C) 1996-2013 the ALD Team. All Rights Reserved. $
ALD!
ALD!
```

Donc oui, tous les `UPX` ont été remplacé par des `ALD`. Nous devons donc faire l'inverse pour continuer.

```bash
$ file elf.bin 
elf.bin: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, no section header
$ cat elf.bin | sed s/ALD/UPX/g > elf_upx.bin
$ upx -d ./elf_upx.bin -o ./elf_upx_unpacked.bin
# -d: decompress
                       Ultimate Packer for eXecutables
                          Copyright (C) 1996 - 2017
UPX 3.94        Markus Oberhumer, Laszlo Molnar & John Reiser   May 12th 2017

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
    738384 <-    297492   40.29%   linux/amd64   elf_upx_unpacked.bin

Unpacked 1 file.
$ file ./elf_upx_unpacked.bin 
./elf_upx_unpacked.bin: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, for GNU/Linux 2.6.32, BuildID[sha1]=280771ce69bcde48b3a996331811768a49c7c120, stripped
$ chmod +x ./elf_upx_unpacked.bin
$ ./elf_upx_unpacked.bin 
usage : ./elf_upx_unpacked.bin <mot de passe>
$ ./elf_upx_unpacked.bin test
Mauvais mot de passe
```

Nous avons donc réussi à décompresser (unpacker) notre `elf.bin` en `elf_upx_unpacked`. Nous pouvons maintenant commencer une analyse classique du binaire.

```assembly
$ r2 -d ./elf_upx_unpacked.bin
> aaaa
> afl
0x00400990    1 43           entry0
0x00400aae    9 114          sub.w0c__:_aae
0x00400b20    6 83           main
0x00401080   66 828          fcn.00401080
0x00407840    3 158          fcn.00407840
0x00408010   30 445  -> 417  fcn.00408010
0x004440f0  296 11505 -> 7157 sub.n_in_writable_segment_detected_f0
> s main
> pdf
/ (fcn) main 83
|   main (int arg2);
|           ; arg int arg2 @ rsi
|           ; DATA XREF from entry0 (0x4009ad)
|           0x00400b20      53             push rbx
|           0x00400b21      83ff01         cmp edi, 1                  ; 1
|       ,=< 0x00400b24      7e1f           jle 0x400b45 # on s'assure d'un argv[1]
|       |   0x00400b26      488b7e08       mov rdi, qword [rsi + 8]    ; arg2
|       |   0x00400b2a      e87fffffff     call sub.w0c__:_aae # appel de fonction
|       |   0x00400b2f      89c3           mov ebx, eax # on récupére une valeur de
|       |   0x00400b31      85c0           test eax, eax # retour qui va nous dire si mdp
|      ,==< 0x00400b33      752b           jne 0x400b60 # OK ou NOT OK
|      ||   0x00400b35      488d3d238d08.  lea rdi, qword str.Mauvais_mot_de_passe
|      ||   0x00400b3c      e8cf740000     call fcn.00408010 # NOT OK
|      ||   ; CODE XREFS from main (0x400b5e, 0x400b71)
|    ..---> 0x00400b41      89d8           mov eax, ebx
|    ::||   0x00400b43      5b             pop rbx
|    ::||   0x00400b44      c3             ret
|    ::||   ; CODE XREF from main (0x400b24) # si pas d'argument
|    ::|`-> 0x00400b45      488b36         mov rsi, qword [rsi]        ; arg2
|    ::|    0x00400b48      488d3df58c08.  lea rdi, qword str.usage_:__s__mot_de_passe
|    ::|    0x00400b4f      b800000000     mov eax, 0
|    ::|    0x00400b54      e8e76c0000     call fcn.00407840 # OK
|    ::|    0x00400b59      bb02000000     mov ebx, 2
|    `====< 0x00400b5e      ebe1           jmp 0x400b41
|     :|    ; CODE XREF from main (0x400b33)
|     :`--> 0x00400b60      488d3d118d08.  lea rdi, qword str.Bravo___Vous_pouvez_utiliser_ce_mot_passe_pour_la_suite
|     :     0x00400b67      e8a4740000     call fcn.00408010
|     :     0x00400b6c      bb00000000     mov ebx, 0
\     `===< 0x00400b71      ebce           jmp 0x400b41
```

```bash
0x00400b20      53             push rbx
0x00400b21      83ff01         cmp edi, 1
0x00400b24      7e1f           jle 0x400b45 # on s'assure d'un argv[1]
```

Rapidement, on peut dire que notre programme s'assure de la présence d'un `argv[1]`, si `rdi <= 1` alors on envoi le message d'erreur `usage: ./elf_upx_unpacked.bin <mot de passe>` avec un `jle 0x400b45`. Le registre `rdi` contient le nombre d'arguments du programme. Lorsque `rdi = 0x00000001` alors message d'erreur, lorsque `rdi = 0x00000002` on continu l'exécution.

> `rdi` et `edi` sont la même entité.

```assembly
0x00400b26      488b7e08       mov rdi, qword [rsi + 8]    ; arg2
0x00400b2a      e87fffffff     call sub.w0c__:_aae
```

Ensuite, on se charge de récupérer le `argv[1]` et d'appeler la fonction `sub.w0c__:_aae` juste derrière.

```assembly
0x00400b2a      e87fffffff     call sub.w0c__:_aae
0x00400b2f      89c3           mov ebx, eax 
0x00400b31      85c0           test eax, eax # test de la valeur de retour de la fonction
0x00400b33      752b           jne 0x400b60 # prise de décision succès ? ou échec ?
0x00400b35      488d3d238d08.  lea rdi, qword str.Mauvais_mot_de_passe # échec
0x00400b3c      e8cf740000     call fcn.00408010
0x00400b41      89d8           mov eax, ebx
0x00400b43      5b             pop rbx
0x00400b44      c3             ret
...
0x00400b60      488d3d118d08.  lea rdi, qword str.Bravo___Vous_pouvez_utiliser_ce_mot_passe_pour_la_suite # succès
0x00400b67      e8a4740000     call fcn.00408010
0x00400b6c      bb00000000     mov ebx, 0
0x00400b71      ebce           jmp 0x400b41
```

On peut voir que cette fonction est très importante, en effet, c'est elle qui décide quel message afficher. Lors d'un succès, on affiche `Vous pouvez utiliser ce mot de passe pour la suite` et lors d'un échec, on affiche `Mauvais mot de passe`.

Nous allons donc passer à l'analyse de la fonction `sub.w0c__:_aae`.

```assembly
> s sub.w0c__:_aae
> pdf
/ (fcn) sub.w0c__:_aae 114
|   sub.w0c__:_aae (int arg1);
|           ; arg int arg1 @ rdi
|           ; CALL XREF from main (0x400b2a)
|           0x00400aae      4889fe         mov rsi, rdi                ; arg1
|           0x00400ab1      b800000000     mov eax, 0
|           0x00400ab6      48c7c1ffffff.  mov rcx, -1
|           0x00400abd      f2ae           repne scasb al, byte [rdi]
|           0x00400abf      b800000000     mov eax, 0
|           0x00400ac4      4883f9e0       cmp rcx, -0x20
|       ,=< 0x00400ac8      7402           je 0x400acc
|       |   ; CODE XREF from sub.w0c__:_aae (0x400b1e)
|      .--> 0x00400aca      f3c3           ret
|      :|   ; CODE XREF from sub.w0c__:_aae (0x400ac8)
|      :`-> 0x00400acc      0fb60e         movzx ecx, byte [rsi]
|      :    0x00400acf      84c9           test cl, cl
|      :,=< 0x00400ad1      7446           je 0x400b19
|      :|   0x00400ad3      488d15e68d08.  lea rdx, qword str.w0c__:   ; 0x4898c0 ; "w0c&]:\x0e;\rM*\x1f.\x1f-O(Q7z\x14v x\x0f!M!l\x11"
|      :|   0x00400ada      488d7e01       lea rdi, qword [rsi + 1]    ; r11
|      :|   0x00400ade      b801000000     mov eax, 1
|      :|   0x00400ae3      be33000000     mov esi, 0x33               ; '3' ; 51
|      :|   0x00400ae8      41b900000000   mov r9d, 0
|     ,===< 0x00400aee      eb16           jmp 0x400b06
|     |:|   ; CODE XREFS from sub.w0c__:_aae (0x400b0b, 0x400b15)
|   ..----> 0x00400af0      410fb6c0       movzx eax, r8b
|   ::|:|   0x00400af4      0fb632         movzx esi, byte [rdx]
|   ::|:|   0x00400af7      0fb60f         movzx ecx, byte [rdi]
|   ::|:|   0x00400afa      4883c201       add rdx, 1
|   ::|:|   0x00400afe      4883c701       add rdi, 1
|   ::|:|   0x00400b02      84c9           test cl, cl
|  ,======< 0x00400b04      7411           je 0x400b17
|  |::|:|   ; CODE XREF from sub.w0c__:_aae (0x400aee)
|  |::`---> 0x00400b06      4589c8         mov r8d, r9d
|  |:: :|   0x00400b09      85c0           test eax, eax
|  |`=====< 0x00400b0b      74e3           je 0x400af0
|  | : :|   0x00400b0d      31f1           xor ecx, esi
|  | : :|   0x00400b0f      3a0a           cmp cl, byte [rdx]
|  | : :|   0x00400b11      410f94c0       sete r8b
|  | `====< 0x00400b15      ebd9           jmp 0x400af0
|  `------> 0x00400b17      f3c3           ret
|      :|   ; CODE XREF from sub.w0c__:_aae (0x400ad1)
|      :`-> 0x00400b19      b801000000     mov eax, 1
\      `==< 0x00400b1e      ebaa           jmp 0x400aca
```

```assembly
0x00400aae      4889fe         mov rsi, rdi
0x00400ab1      b800000000     mov eax, 0
0x00400ab6      48c7c1ffffff.  mov rcx, -1
0x00400abd      f2ae           repne scasb al, byte [rdi]
0x00400abf      b800000000     mov eax, 0
0x00400ac4      4883f9e0       cmp rcx, -0x20
0x00400ac8      7402           je 0x400acc
0x00400aca      f3c3           ret
```

Les premières instructions servent sûrement à gérer l'argument de la fonction (qui est notre mot de passe). Après cela, on compare `rcx` à `0x20`. Si cette comparaison et vraie, alors on continue le l'exécution (on évite le `ret`). Sinon on va sur le `ret` qui va nous faire quitter la fonction avec un `eax` à 0, ce qui va nous afficher `Mauvais mot de passe`.

Pour faire simple, le `cmp rcx, -0x20` va s'assurer que le `argv[1]` fourni au programme va avoir une taille de 32 caractères.

| `argv[1]` |        `rcx`         |       `-0x20`        | `cmp rcx, -0x20` |
| :-------: | :------------------: | :------------------: | :--------------: |
|  `AAAA`   | `0xfffffffffffffffa` | `0xffffffffffffffdf` |    NOT EQUAL     |
| `AAAAAA`  | `0xfffffffffffffff8` | `0xffffffffffffffdf` |    NOT EQUAL     |
|  `A`*32   | `0xffffffffffffffdf` | `0xffffffffffffffdf` |    **EQUAL**     |

Nous devons donc avoir un `argv[1]` de taille 0x20, soit 32 caractères pour respecter cette condition.

```bash
> ood AAAA
> db 0x00400ac8
> dc
hit breakpoint at: 400ac8
> dr rip
0x00400ac8
> dr rip=0x00400acc
0x00400ac8 ->0x00400acc
```

Normalement, nous devons fournir un argument de taille 32 octets, mais nous allons juste donner `ABCD` au programme, placer un breakpoint sur `je 0x400acc` afin de modifier la valeur de `rip` pour passer outre la condition des 32 caractères.

```assembly
0x00400acc      0fb60e         movzx ecx, byte [rsi]
0x00400acf      84c9           test cl, cl
0x00400ad1      7446           je 0x400b19
0x00400ad3      488d15e68d08.  lea rdx, qword str.w0c__:   ; 0x4898c0 ; "w0c&]:\x0e;\rM*\x1f.\x1f-O(Q7z\x14v x\x0f!M!l\x11"
0x00400ada      488d7e01       lea rdi, qword [rsi + 1]
0x00400ade      b801000000     mov eax, 1
0x00400ae3      be33000000     mov esi, 0x33               ; '3' ; 51
0x00400ae8      41b900000000   mov r9d, 0
0x00400aee      eb16           jmp 0x400b06
```

Dans ce bloc on peut remarquer que l'on va manipuler la chaîne `w0c&]:\x0e;\rM*\x1f.\x1f-O(Q7z\x14v x\x0f!M!l\x11`.

```assembly
0x00400af0      410fb6c0       movzx eax, r8b
0x00400af4      0fb632         movzx esi, byte [rdx]
0x00400af7      0fb60f         movzx ecx, byte [rdi]
0x00400afa      4883c201       add rdx, 1
0x00400afe      4883c701       add rdi, 1
0x00400b02      84c9           test cl, cl
0x00400b04      7411           je 0x400b17
0x00400b06      4589c8         mov r8d, r9d
0x00400b09      85c0           test eax, eax
0x00400b0b      74e3           je 0x400af0
0x00400b0d      31f1           xor ecx, esi
0x00400b0f      3a0a           cmp cl, byte [rdx]
0x00400b11      410f94c0       sete r8b
0x00400b15      ebd9           jmp 0x400af0
0x00400b17      f3c3           ret
0x00400b19      b801000000     mov eax, 1
0x00400b1e      ebaa           jmp 0x400aca
```

Dans la suite on observe une ligne intéressante `xor ecx, esi`.

```assembly
0x00400b0d      31f1           xor ecx, esi
0x00400b0f      3a0a           cmp cl, byte [rdx]
```

On réalise un **XOR** puis un comparaison, c'est sûrement une partie importante du prgramme. Pour ce xor on utilise `ecx` et `esi`.

```assembly
0x00400af4      0fb632         movzx esi, byte [rdx]
0x00400af7      0fb60f         movzx ecx, byte [rdi]
0x00400afa      4883c201       add rdx, 1
0x00400afe      4883c701       add rdi, 1
```

Après pas mal de debug sur le programme sur les instructions importantes, on comprend comment fonctionne la vérification du mot de passe.

On peut retrouver ce dernier avec le script suivant :

```python
#!/usr/bin/env python3

string = '3w0c&]:\x0e;\rM*\x1f.\x1f-O(Q7z\x14v x\x0f!M!l\x11'

for i in range(1, len(string)):
    print(chr(ord(string[i-1]) ^ ord(string[i])), end='')
print()
# DGSE{g456@g5112bgyfMnbVXw.llM}
```

---
## ZIP
---

```bash
$ unzip -d suite_files/ -P "DGSE{g456@g5112bgyfMnbVXw.llM}" suite.zip 
Archive:  suite.zip
  inflating: suite_files/suite.txt

$ cat suite_files/suite.txt 
Suite du challenge Richelieu :

ssh defi1.challengecybersec.fr -l defi1 -p 2222

mot de passe : DGSE{2f77c517b06f1cd1ce864f79f41f25ca8874413c8c1204f7ec9c6728c87f270a}
```

C'est ici que le challenge se termine pour moi.