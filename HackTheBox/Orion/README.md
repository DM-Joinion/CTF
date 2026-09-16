# **Orion**

![Dificultad](https://img.shields.io/badge/Dificultad-Easy-green)
![SO](https://img.shields.io/badge/SO-Linux-yellow)
![Plataforma](https://img.shields.io/badge/Plataforma-HackTheBox-9FEF00)

---

&nbsp;

## Resume


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

- Puerto 22/tcp — SSH: Protocol to maintain a connexion between two devices
- Puerto 80/tcp — HTTP: Web aplication/page redirected to http://orion.htb/

### /etc/hosts

To access the webpage we add at the end of the `/etc/hosts` file the redirection to the mentioned page:

```
echo "[ip] connected.htb" | sudo tee -a /etc/hosts
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
https://nvd.nist.gov/vuln/detail/cve-2025-32432


On it...