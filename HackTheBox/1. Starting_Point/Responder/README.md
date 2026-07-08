Welcome to my writeup version of the HTB machine Appointment.

![alt text](./img/Responder.png)

# Enumeration

## NMAP

- Start with nmap to see what ports are open and what services are running:

![imagen_nmap](./img/nmap.png)

- Next, see deeper information about versions and services.

![alt text](./img/nmap2.png)

- We can see that there are two ports opened:

    - Port 80: An http service running based on Apache.
    - Port 5985:Hosts a service called WinRM(Windows Remote Management).
    > As the name sugests, the port 5985 is commonly used for automation and remote management of windows tools and sistems.

## HTTP

When searching for the HTTP service on a navigator, it gets redirected to a website with the url `http://unika.htb/`

- Edit the /etc/hosts file:

`sudo nano /etc/hosts`

and add

`[machine_ip] unika.htb`

- Now that we are able to acces the web page:

![alt text](./img/http.png)

It looks like the basic web page. From all the features, the only thingg that seems to do something interesting is the "EN" button.

![alt text](./img/EN.png)

When presing either of the two buttons will change the lenguage of the website.

![alt text](./img/param.png)

Seeing that the url is charging a file, this may imply a posible Local File Inclusion(LFI) if the site is not sanitized

# LFI Vulnerability

A LFI vulnerability consists of using an unsanitized function to trick the page into including a file which is not intended to.

One of the most common files used by pentesters is: `/windows/system32/drivers/etc/hosts`

This file will follow `../../../../`, which we use to escape whateveer directory the website may be hosted in.

It would look like this:
![alt text](./img/lfi1.png)

We can see that the file has been included. In this case a unsanitized "include()" function in php.

