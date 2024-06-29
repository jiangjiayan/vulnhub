# EvilBox-One | 89/100
## Résume 
Cette boîte intermédiaire classée par la communauté comportait le fuzzing et l'exploitation d'une vulnérabilité de traversée de répertoire, avec laquelle nous avons pu accéder à la clé SSH d'un utilisateur. Une fois sur la cible, nous avons réalisé que le fichier /etc/passwd était accessible en écriture, nous avons donc créé un nouvel utilisateur, root2, avec des autorisations de niveau racine.
1. Collecte d'informations :

```bash
arp-scan -l
```
mon address est **10.1.12.54**,l'adress de la machine virtuelle:**10.1.12.95**

![请添加图片描述](https://img-blog.csdnimg.cn/direct/ae494833c9c64d1eb397ffa4aa774095.png)


2. Énumération
Je vais commencer par énumérer cette case avec une analyse Nmap couvrant tous les ports TCP. Ici, j'utiliserai également les indicateurs -sC et -sV pour utiliser des scripts de base et énumérer les versions.

![请添加图片描述](https://img-blog.csdnimg.cn/direct/30ff569cd926454bb4470709ae226555.png)

3. En nous dirigeant vers le site sur le port 80, nous trouvons une page de destination Apache par défaut :
![请添加图片描述](https://img-blog.csdnimg.cn/direct/48843437f1ad4bfca9c076989867e102.png)

4. Scanne le site web 10.1.12.95：

```bash
dirsearch -u 10.1.12.95

```
![请添加图片描述](https://img-blog.csdnimg.cn/direct/d391c2eedff0450489c9ec155e1b90d4.png)

5. En essayant manuellement une page robots.txt, nous trouvons :
![请添加图片描述](https://img-blog.csdnimg.cn/direct/55e376a9e7614433a557565b66e609d7.png)

6. En fuzzant les répertoires, nous trouvons un /secret/evil.php:

```bash
gobuster dir -u http://10.1.12.95/secret/ -w /usr/share/wordlists/dirb/common.txt -x txt,php,html

```
![请添加图片描述](https://img-blog.csdnimg.cn/direct/bd3681887f2e4a5da28c204aade2b3cf.png)

7. Mais en naviguant vers la page, c'est juste une page blanche.

En essayant quelques traversées de répertoires différentes et en obtenant continuellement des pages vierges plutôt que des erreurs 404/Not Found, je me suis demandé si la page était vulnérable.

Nous pouvons automatiser le fuzzing pour la vulnérabilité avec ffuf :

sympa, ffuf a trouvé quelque chose pour nous.
![请添加图片描述](https://img-blog.csdnimg.cn/direct/6f7a81861d034eef8facbe2651bdac1e.png)

8. Nous pouvons accéder à http://192.168.171.212/secret/evil.php?command=/etc/passwd et confirmer la vulnérabilité

![请添加图片描述](https://img-blog.csdnimg.cn/direct/44cc6026af29424c92bec8bc9be816f0.png)

9. Exploitation
C'est formidable que nous ayons trouvé cette vulnérabilité, mais en soi, elle ne nous est pas encore très utile.

En parcourant le fichier /etc/passwd, nous voyons qu'il y a un utilisateur nommé mowree :`mowree:x:1000:1000:mowree,,,:/home/mowree:/bin/bash
`
Puisque nous savons maintenant que SSH est également ouvert sur la cible, voyons si nous pouvons accéder à leur clé SSH :

Ça a marché!
![请添加图片描述](https://img-blog.csdnimg.cn/direct/70e37207858446cd866d5cbc565384a2.png)
En changeant de mode et en essayant d'utiliser la clé, nous voyons qu'elle est protégée par phrase secrète.

```bash
 wget http://10.1.12.95/secret/evil.php command=/home/mowree/.ssh/id_rsa -o id_rsa
```

```bash
cat id_rsa
```
![请添加图片描述](https://img-blog.csdnimg.cn/direct/f1aa8547425b474b811270324a7a76ac.png)

```bash
ssh2john evil.php?command=%2Fhome%2Fmowree%2F.ssh%2Fid_rsa > code

```

```bash
john code --wordlist=/usr/share/wordlists/rockyou.txt  
```

![请添加图片描述](https://img-blog.csdnimg.cn/direct/f9486e1b3f9e4459b8c78b22f1e31969.png)
Cool, maintenant que nous avons la phrase secrète, nous pouvons nous connecter en SSH et récupérer l'indicateur local.txt .
