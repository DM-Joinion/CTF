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

It looks like the basic web page, but there is nothing useful in it. Probably there are directories or subdomains to enumerate.
