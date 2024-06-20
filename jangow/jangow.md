# Jangow 1.0.1
1.scan-arp -l :eth0 本机地址

2.nmap -sP adrr()/24靶机地址

3.nmap -sC -sV -p- adrr :查看靶机信息，查看开放端口，有域名用域名，没有域名直接使用ip进入网页，并查看到它开放了21，80端口。

21为ftp,80为http,先从ftp入手

**ftp可能存在的漏洞：匿名登陆，即用户名为anonymous,密码也是anonymous**（这里没有，显示登录失败）

小目标为找到ftp的用户名和密码

4.使用ip进入网页，dirb 进行字典爆破（或者借助工具link grabber）

5.发现Buscar点击后，**URL存在“=”，表示可能存在命令行注入攻击，可以使用ls -all测试，多条命令使用“；”隔开**

5.ls -all后发现在二级目录下存在 assets, busque.php等文件，因此 cd ..回到root目录，发现root目录下有两个文件，site 和.backup 我们猜测.backup中含有我们需要的用户名和密码，在URL中添加命令 cat .backup，果然找到用户名为"jangow01"，密码为"abygurl69"

6.回到3. 使用ftp登录，命令为ftp arr ,随后输入用户名密码，登录成功！

7.此时 cd/home查看主目录，发现可疑文件jangow01,cd jangow01,发现.txt格式的可疑文件，cat打开，得到d41d8cd98f00b204e9800998ecf8427e，但不是最终的drapeau,无用

找其他办法

8.打开虚拟机靶机，查看其版本是否存在可利用漏洞：使用用户名为"jangow01"，密码为"abygurl69"登录靶机，uname-a查看版本号，（我的机器无法使用uname,**hostnamectl也可以**）查到它的使用的为Linux 4.4.0-31-generic，对应CVE是2017-16995。

9.Github 下载CVE：2017-16995的.c代码

10.回到攻击机，进入CVE：2017-16995的.c代码所在的文件夹，再次使用ftp连接，在攻击机中进入jangow01目录

11.**漏洞上传**：命令为 put cve.c 

12.回到靶机  ls -all查看cve.c是否在jangow01目录//gcc cve.c -o cve 编译，生成cve可执行//运行cve

“[.] (-_-t) exploit for counterfeit grsec kernels such as KSPP and linux-hardened
[.]
[.]
[.*] UID from cred structure: 1000, matches the current: 1000 .1 
[.*] hammering cred structure at ffff880033d4d480 
[.*] credentials patched, launching shell...” 表示运行注入成功

13.再次测试whoami，此时显示我的身份已经变成root

14.此时我们有权查看root目录，ls /root 发现可疑文件proof.txt

15.打开proof.txt——————成功
