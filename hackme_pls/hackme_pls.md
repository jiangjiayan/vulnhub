# HACK ME PLEASE: 1
## Résume 
Cette boîte intermédiaire classée par la communauté comportait le fuzzing et l'exploitation d'une vulnérabilité de traversée de répertoire, avec laquelle nous avons pu accéder à la clé SSH d'un utilisateur. Une fois sur la cible, nous avons réalisé que le fichier /etc/passwd était accessible en écriture, nous avons donc créé un nouvel utilisateur, root2, avec des autorisations de niveau racine.
1. Collecte d'informations :

```bash
arp-scan -l
```
mon address est **110.1.12.61**,l'adress de la machine virtuelle:**10.1.12.119**

![请添加图片描述](https://img-blog.csdnimg.cn/direct/e0596ccaa114417daab744ec32317e63.png)



2. Énumération
Je vais commencer par énumérer cette case avec une analyse Nmap couvrant tous les ports TCP. Ici, j'utiliserai également les indicateurs -sC et -sV pour utiliser des scripts de base et énumérer les versions.

![请添加图片描述](https://img-blog.csdnimg.cn/direct/969fbedd47d943709e70114e7acf0ddc.png)


3. En nous dirigeant vers le site sur le port 80, nous trouvons une page de destination Apache par défaut :


![请添加图片描述](https://img-blog.csdnimg.cn/direct/429fae60ff0d4753b594d8a4922d3d8f.png)

4. Scanne le site web 10.1.12.119：
J'ai utilisé deux méthodes de force brute, dirsearch et gobuster, pour vérifier s'il y a des répertoires cachés.
```bash
dirsearch -u 10.1.12.119

```
![请添加图片描述](https://img-blog.csdnimg.cn/direct/dac61008083049d4ad7a831e8d557300.png)

```bash
gobuster dir -u http://10.1.12.119 -w /usr/share/wordlists/dirb/common.txt -q -r
```

![请添加图片描述](https://img-blog.csdnimg.cn/direct/c65134969b0c4fb2b56625d565f516e4.png)

Selon ces deux résultats, il semble que ce défi n'ait pas de répertoires cachés et que seul le répertoire /js soit accessible. Donc, nous allons vérifier son code JavaScript

5. En regardant la source de la page, nous pouvons voir quelques fichiers javascript.
![请添加图片描述](https://img-blog.csdnimg.cn/direct/7e82913d15d14732878bc9b726fb9b44.png)


Verfier le site 10.1.12.119/js/main.js :
![请添加图片描述](https://img-blog.csdnimg.cn/direct/7af01c6f4a324a4e996cc45f28c6d152.png)


6. En ouvrant le **main.js**, nous pouvons voir le répertoire **/seeddms51x/seeddms-5.1.22/**

![请添加图片描述](https://img-blog.csdnimg.cn/direct/9376ee362afe4f8896fcbb91eea81526.png)


7. Nous avons de nouveau fait un test de force brute sur **http://10.1.12.119/seeddms51x/** et nous avons trouvé les répertoires **/conf** et **/data**.（Cependant, nous n'avons pas la permission d'accéder à ces deux URL.）

![请添加图片描述](https://img-blog.csdnimg.cn/direct/8f0b9fda3ccd402c9c710a60f5595f6a.png)
（En fait ,j’étais bloqué ici,rien à faire ,rien de trouver. Après j'ai regardé les méthodes d'autres per, il a trouvé la vulnérabilité de configuration seeddms sur github,voici le lien de github:`https://github.com/JustLikeIcarus/SeedDMS/blob/master/conf/settings.xml.template`）

Il existe un fichier de configuration (paramètre) qui contient généralement des informations d'identification ou d'autres informations importantes.

À partir de ce dépôt github, nous pouvons avoir une idée simple des emplacements des fichiers et des dossiers dans SeedDMS.

Dans le dépôt, nous pouvons voir le fichier **settings.xml**.template sous le dossier /conf.

![请添加图片描述](https://img-blog.csdnimg.cn/direct/0364da37cdd04390a305025ded107762.png)



9. Nous pouvons accéder à `http://10.1.12.119/seeddms51x/conf/settings.xml`

![请添加图片描述](https://img-blog.csdnimg.cn/direct/aa9f127f3a9943ecb72f173474a53457.png)
Nous avons trouvé la section sur la base de données, qui nous indiquait que le nom et le mot de passe de la base de données sont **seeddms**.

10. Essayer de connecter à cette base de données .(sur kali)

```bash
mysql -u seeddms -h 10.1.12.119 -p 
```

![请添加图片描述](https://img-blog.csdnimg.cn/direct/3d722502cffc48948d4d1967aa64515f.png)
et puis 

```bash
show databases;
```
![请添加图片描述](https://img-blog.csdnimg.cn/direct/6ee1b7145b0e4f6b9c4b7b2bdd4d7be4.png)
il existe 5 bases de données ,avec grande proba,les infos on a besoin sont dans seeddms,du coup on fais:

```bash
use seeddms
```
![请添加图片描述](https://img-blog.csdnimg.cn/direct/c7e895749be14c8596f9ded35b2ebf49.png)
Nous sommes intéressés par deux tableaux sur les noms d'utilisateur, l'un est **user** (dans la dernière ligne) et l'autre est **tbluser**.

11. Nous regardons respectivement user et tbluer:

```bash
select * from users;
```
![请添加图片描述](https://img-blog.csdnimg.cn/direct/8d88e19db5da48898418bc754b48421c.png)

```bash
select * from tblUsers;
```
![请添加图片描述](https://img-blog.csdnimg.cn/direct/85dd2304fa1e4111a2586f83c15cd5dd.png)
Nous obtenons un ensemble de nom d'utilisateur et de mot de passe
 user:   **admin** 
 mots de pass:**f9ef2c539bad8a6d2f3432b6d49ab51a** 
 Cependant, ce mot de passe ne semble pas être utilisé directement. Il a été crypté par MD5.

La méthode normale consiste à effectuer un déchiffrement MD5, et le résultat est 123456.
Mais j'ai essayé de nombreuses méthodes et je n'ai pas pu l'obtenir, j'ai donc choisi une autre méthode pour **changer le mot de passe de la base de données.**

12. Changer le mot de passe de la base de données

```bash
update tblUsers set pwd =MD5(202407) where id =1;
```
（on change le mot de pass de admin à 202407）
![请添加图片描述](https://img-blog.csdnimg.cn/direct/2214ac3c229f46dca2a2a0caf8c16f99.png)

13. Rentrer l’étape  6,et login avec admin et 202407.
![请添加图片描述](https://img-blog.csdnimg.cn/direct/e78d676fc73b4f84845c6d0421cfa982.png) 
Étant donné la présence de "add document", nous considérons d'abord la possibilité d'une vulnérabilité de téléchargement de fichier. Essayons de télécharger un shell inversé PHP.

14.  shell inversé PHP
![请添加图片描述](https://img-blog.csdnimg.cn/direct/ccdd402d073c4386b1d8bd907118b720.png)
![请添加图片描述](https://img-blog.csdnimg.cn/direct/995ca3a1014946eca3b06103f4f50105.png)
Sur la page Web, nous n'avons pas vu qu'il avait été téléchargé avec succès. ![请添加图片描述](https://img-blog.csdnimg.cn/direct/53239005adfc486ea6487e3ae0e6a440.png)
Revenez à la base de données et vérifiez si le script php est dans la base de données.
![请添加图片描述](https://img-blog.csdnimg.cn/direct/b5233515da4c43b794c95a4cd57824b0.png)
En fait, j'ai vérifié beaucoup de tables et j'ai finalement trouvé le script que j'ai téléchargé dans tblDocuments.

15. Nous vérifions son chemin de téléchargement : **http://10.1.12.119/seeddms51x/data/1048576/**

Pourquoi 1048576 ?Dans le fichier de configuration(l'étape 9), il y a une ligne 

`<server coreDir="" luceneClassDir="" contentOffsetDir="1048576" maxDirID="0" updateNotifyTime="86400" extraPath="/var/www/html/seeddms51x/pear/" maxExecutionTime="30" cmdTimeout="10" enableDebugMode="false"> </server>,` 
![请添加图片描述](https://img-blog.csdnimg.cn/direct/df39cad265464b6083306b1a98b070ed.png)

où contentOffsetDir="1048576" indique le dossier où les fichiers sont stockés.

Mais on n'a pas le droit d’accès `http://10.1.12.119/seeddms51x/data/1048576/`
![请添加图片描述](https://img-blog.csdnimg.cn/direct/6ba59144ea6043b1acb8e0e27057c0d1.png)

16. Nous effectuons à nouveau un test de force brute sur les sites.

```bash
dirsearch -u http://10.1.12.119/seeddms51x/data/1048576/
```
![请添加图片描述](https://img-blog.csdnimg.cn/direct/f9f285b415de4544ae2f8be82d38367f.png)

encore forbidden,encore chercher les sites

```bash
dirsearch -u http://10.1.12.119/seeddms51x/data/1048576/4
```
![请添加图片描述](https://img-blog.csdnimg.cn/direct/d2b160bb07f9413b927245c3a74a45c5.png)
on a trouvé `http://10.1.12.119/seeddms51x/data/1048576/4/1.php`
Il devrait renommer le php script que nous avons téléchargé
![请添加图片描述](https://img-blog.csdnimg.cn/direct/fcb81c31daca493ca7fb4067678e5932.png)

17. À ce moment , nous pouvons commencer à surveiller.
![请添加图片描述](https://img-blog.csdnimg.cn/direct/b1d8b48f4dca4435b0c18887e0cad246.png)
18.  

```bash
cat etc/passwd
```
![请添加图片描述](https://img-blog.csdnimg.cn/direct/e166e8e9ad9c44afb9ec8b1f733ec8ad.png)
Dans l'avant-dernière ligne, on peut voir un **saket** d'utilisateur (on rentre l'étape 11,il existe aussi un saket avec son mot de pass)
Nous avons deviné qu'il pourrait s'agir du nom d'utilisateur et du mot de passe root sur cette machine, nous avons essayé `sudo -l` et utilisé le mot de passe de l'étape 11.
![请添加图片描述](https://img-blog.csdnimg.cn/direct/92c6e71909464c91be7491a20d52a964.png)
il est réussi d'access root!



