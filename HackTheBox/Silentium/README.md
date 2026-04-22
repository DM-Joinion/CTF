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

While trying to load the rest of the page we recive an ```401 Unauthorised``` code.

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

ONGOING...