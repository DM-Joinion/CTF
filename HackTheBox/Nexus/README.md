# **Nexus**

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
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-20 22:17 CEST
Initiating SYN Stealth Scan at 22:17
Scanning 10.129.105.221 [65535 ports]
Discovered open port 80/tcp on 10.129.105.221
Discovered open port 22/tcp on 10.129.105.221
Completed SYN Stealth Scan at 22:17, 14.79s elapsed (65535 total ports)
Nmap scan report for 10.129.105.221
Host is up, received user-set (0.052s latency).
Scanned at 2026-09-20 22:17:00 CEST for 15s
Not shown: 58448 closed tcp ports (reset), 7085 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 15.01 seconds
           Raw packets sent: 76886 (3.383MB) | Rcvd: 60591 (2.424MB)
```

- **Second Nmap**

```bash
nmap  -sCV -p<ports> [IP]
```

- **Result:**

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-20 22:19 CEST
Nmap scan report for 10.129.105.221
Host is up (0.058s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://nexus.htb/
|_http-server-header: nginx/1.24.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.22 seconds
```

### Services Resume

- Port 22/tcp — SSH: Protocol to maintain a connexion between two devices
- Port 80/tcp — HTTP: Web aplication/page redirected to http://nexus.htb/

### /etc/hosts

To access the webpage we add at the end of the `/etc/hosts` file the redirection to the mentioned page:

```
echo "[ip] nexus.htb" | sudo tee -a /etc/hosts
```


###  <u>Web Aplication Fingerprinting</u>
- Use whatweb to see the specifications about the web app and the technologies used

```bash
whatweb http://nexus.htb
```
- **Output**

```
http://nexus.htb [200 OK] Country[RESERVED][ZZ], Email[careers@nexus.htb,j.matthew@nexus.htb], HTML5, HTTPServer[Ubuntu Linux][nginx/1.24.0 (Ubuntu)], IP[10.129.105.221], Script, Title[Nexus Energy Authority — Powering the Nation's Future], nginx[1.24.0]
```

We already have some emails to list. The **manager** one, will probably be important.


###  <u>Subdomain Enumeration</u>

- With *wfuzz*:

```
wfuzz -c --hc 302  -t 200 -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.nexus.htb" http://nexus.htb
 ```

With wfuzz we find a subdomain called `git`.

>To access this subdomain, remember to add it to /etc/hosts

---

# Foothold

- Once inside the subdomain we'll find a button "explore" on the top left. By clicking the button a repository `admin/krayin-docker-setup` is discovered.

- Inside the `.env` of the repo, we'll find some usefull information.

First thing, there is another subdomain to add to /etc/hosts: `http://billing.nexus.htb`

Inside this subdomain, we'll find a login to the service, which I presume is a DataBase following the info found on .env

- Enumerate this repository

On previous commits, a password for the DB can be found: `DB_PASSWORD=N27xh!!2ucY04`

- Using the login `j.matthew@nexus.htb:N27xh!!2ucY04`, we'll access the dashboard.

Clicking on the user profile, the version is `Krayin CRM 2.2.0`. There is a CVE related to this version of krying.

###  <u>CVE-2026-38526</u>




---

## User Flag
- Explain how to get to a user once foothold is aquired
>If the foothold starts with a user thats not html or default, explain how to get the user flag

```bash
[used commands]
```

**User Flag:** Aquired!

---

## Privilege Escalation (Root Flag)

- Explain vectors to reach root/admin

```bash
[used commands]
```


**Root Flag:** Aquired!