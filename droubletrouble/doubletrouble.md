@[TOC](Vulnhub：Doubletrouble)
# Résume:
C'est une machine cible intéressante, avec une machine virtuelle à l'intérieur d'une machine virtuelle. La première machine virtuelle utilise la stéganographie et un shell inverse en PHP pour l'élévation de privilèges. La deuxième machine utilise le déchiffrement SQL pour l'exploration des répertoires. 

# Doubletrouble 1 Walkthrough
1. `arp-scan -l`scanner les address ,trouver la machine vulnérable
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/aa037f9e45164d4fa6723950c0e742dd.png#pic_center)
2. La page 192.168.1.101
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/eec8f220424345ca937da3a7e5861f92.png#pic_center)
3. `dirsearch` est un outil de ligne de commande utilisé pour la reconnaissance de répertoires sur les serveurs web. Il permet de découvrir des répertoires et des fichiers cachés ou non indexés.  On fais `dirsearch -u 192.168.1.101`,dans son résultat ,tout d'abord, nous pensons que "secret" est une URL très suspecte et qu'il pourrait y avoir des indices utiles, nous allons donc prioritairement vérifier `http://192.168.1.101/secret/`.
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/3e88a7e4f25b402194a5e7a050058447.png#pic_center)
4. Ici,on a trouvé l'image s’appelle 'doubletrouble' cliquez ça.
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/db0a9c82a2f84d5caa52988c682d1752.png#pic_center)
5. On n'a pas obtenu d'informations visibles sur l'image, nous envisageons donc qu'il pourrait y avoir de la stéganographie dans l'image.
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/aa406842ac6f43979b65dbf86690d06d.png#pic_center)
6. Maintenant ,on va introduire un nouvel outil pour examiner la stéganographie dans l'image:`stegseek`.Téléchargez d’abord l’image.
Sur Kali linux ,si vous avez pas l'outile ,il faut s’installer avec `sudo apt-get install stegseek
`,et puis on prend `stegseek doubletrouble.jpg -wl /usr/share/wordlists/rockyou.txt -xf img`
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/c4c7d44ae6db4a56ad838d3bd1fb70ad.png#pic_center)
  Il y a un petit problème ici. L'outil StegSeek a signalé que le fichier du dictionnaire est introuvable. Nous devons installer manuellement le dictionnaire 'rockyou'.
  

```bash
sudo apt-get install wordlists
```

```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
```
au final ,sgrâce à cet outil, on vois les informations cachées dans l'image
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/eb622a3be78b4899bf125d5eac0a4123.png#pic_center)

7.   ![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/2155132bfa434855b67aa2e459ea73e2.png#pic_center)Cela ressemble à un nom d'utilisateur sous la forme d'un email et d'un mot de passe,du coup nous considérons  l'étape 2, la page de 192.168.1.101,log in avec l'info dans l'image.(il est réussi )![请添加图片描述](https://img-blog.csdnimg.cn/direct/80649081aa264cd99c17d767f2cd2358.png)

8. Nous avons un point final pour télécharger des fichiers sur le serveur(pour télécharger le photo)![请添加图片描述](https://img-blog.csdnimg.cn/direct/0090d397f71b49979dd98ee938da0e37.png)
![请添加图片描述](https://img-blog.csdnimg.cn/direct/6c8f735d200146c7a155bee6b16c1405.png)
(Il pourrait y avoir une vulnérabilité d'injection de fichiers ici, ce qui nous permet d'essayer d'obtenir un shell inversé pour élever nos privilèges.)

9. Nous ouvrons l'écoute sur le port 4322 et téléversons un script PHP de shell inversé dans la section photo. (Dans le même répertoire, vous pouvez trouver ce script. Vous devez apporter quelques modifications au script : changez l'adresse IP pour celle de votre machine d'attaque, c'est-à-dire votre propre adresse, et changez le port pour celui que vous avez ouvert. Ici, j'ouvre le port 4322.) ![请添加图片描述](https://img-blog.csdnimg.cn/direct/9d89379f10a44133a82a5098033aeb19.png)
10.  L'écoute sur le port 4322：`nc -l 4322`,et puis télécharger shell_inersé.php ,sur le terminal ,![请添加图片描述](https://img-blog.csdnimg.cn/direct/bd03c33461e648868cc0eec5adcdfdf0.png)

      Élévation de privilèges
     ![请添加图片描述](https://img-blog.csdnimg.cn/direct/eb98e88d24fc4c4f99a2328da0fc9e29.png)
Recherche de programmes exécutés avec les privilèges root sans mot de passe

11. `cd root` ;`ls`
![请添加图片描述](https://img-blog.csdnimg.cn/direct/5f4745b2ac9246dc9879c71a20c7f518.png)
Nous voyons la deuxième machine virtuelle

12. Utiliser Python 3 pour accéder à la page web et télécharger la machine virtuelle,avec `python3 -m http.server 4322`.
À ce stade, nous avons terminé l'attaque de la première machine virtuelle et avons réussi à installer la deuxième. Nous l'avons lancée et avons découvert l'adresse.(192.168.1.57）![请添加图片描述](https://img-blog.csdnimg.cn/direct/442fcf3f08fc48238d2551409d9bbe2c.png)
Il y a encore une interface de connexion ici. Nous essayons une attaque anonyme, mais elle est inefficace. Nous envisageons donc une injection SQL.

13. Nous introduisons un nouvel outil, **sqlmap**, et nous expliquerons cet outil à la fin de l'article.
Détection des injections SQL ，on fais `sqlmap -u "http://192.168.1.57"`,![请添加图片描述](https://img-blog.csdnimg.cn/direct/cb5d109962fa4d0584aeb557f10f4306.png)
Les résultats montrent qu'il existe une vulnérabilité d'injection basée sur le temps.

14. Ainsi, cela montre que l'attaque par injection SQL est efficace. Nous procédons donc à l'énumération de ses bases de données et obtenons les bases de données **doubletrouble** et **information_schema**.  `sqlmap -u "http://192.168.1.57 -forms -dbs"`![请添加图片描述](https://img-blog.csdnimg.cn/direct/d04b753f4d0e4bffb9191e296dca631e.png)
15. Il est évident que les données dont nous avons besoin devraient se trouver dans la base de données doubletrouble. Nous essayons donc d'obtenir les tables de données `sqlmap -u "http://192.168.1.57 -forms -D doubletrouble -tables` et les données des tables `sqlmap -u "http://192.168.1.57 -forms -D doubletrouble -T users -dump`.![请添加图片描述](https://img-blog.csdnimg.cn/direct/43a1f01e03ff4e13ac27a0ca3c05d357.png)
![请添加图片描述](https://img-blog.csdnimg.cn/direct/a1d2b413ab024740827627d658829375.png)
16. Nous obtenons le nom d'utilisateur et le mot de passe, nous essayons de nous connecter à l'étape 12, mais la connexion échoue. Nous envisageons que les informations d'identification obtenues soient pour la connexion au port, et nous scannons les ports pour découvrir une connexion SSH.Finalement, nous découvrons que l'utilisation de **clapton** et de son mot de passe permet de se connecter avec succès.`ssh username@addr.`
![请添加图片描述](https://img-blog.csdnimg.cn/direct/3f7a2b8964a6489bb4876e301b196b26.png)
17. Pour élever les privilèges sous la connexion SSH, nous envisageons d'abord d'utiliser des CVE. Nous utilisons `uname -a` pour vérifier sa version et ses vulnérabilités.On trouve que sa version est linux **3.2.0-4-arm64**,sur github on peut trouver son vulnérabilité s’appelle **dirty cow**.
18. Téléchargez le script CVE, ouvrez à nouveau un terminal et téléversez-le via la connexion SSH. Revenez à l'interface de connexion SSH,`scp dirtycow.c clapton@192.168.1.57:~`   nous pouvons voir le script en C de la vulnérabilité.
![请添加图片描述](https://img-blog.csdnimg.cn/direct/ed7b5f4846f94457bc2b32d84712c119.png)
![请添加图片描述](https://img-blog.csdnimg.cn/direct/873d2da28eea4843b0a602733c3f2bb2.png)
19. Nous compilons le script en C. La compilation ici est un peu particulière  `gcc -pthread dirtycow.c -o dirty -lcrypt`. Après la compilation, nous l'exécutons `./dirty`
20. Ici, nous devons réinitialiser un mot de passe. Selon le site `https://github.com/firefart/dirtycow/blob/master/README.md`, le fonctionnement de cette vulnérabilité est le suivant : 
Cet exploit utilise l'exploit Pokémon de la vulnérabilité dirtycow comme base et génère automatiquement une nouvelle ligne de mot de passe. L'utilisateur sera invité à entrer le nouveau mot de passe lorsque le binaire sera exécuté. Le fichier /etc/passwd original est ensuite sauvegardé dans /tmp/passwd.bak et remplace le compte root par la ligne générée. Après avoir exécuté l'exploit, vous devriez pouvoir vous connecter avec le nouvel utilisateur créé.

21. Enfin, nous ouvrons un nouveau terminal, utilisons le nom d'utilisateur **FireFart** du code et le mot de passe que nous avons réinitialisé pour nous connecter via SSH. À ce moment-là, nous entrons en tant que root et obtenons le drapeau , le défi est donc réussi.
![请添加图片描述](https://img-blog.csdnimg.cn/direct/fb478bd8d4a947bf93975f8729ce3c8c.png)







