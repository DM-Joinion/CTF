Welcome to my writeup version of the HTB machine Sequel.

![alt text](./img/Sequel.png)

# Enumeration

## NMAP

I start with nmap to see what ports are open and what services are running:

![imagen_nmap](./img/nmap.png)

Now lets see deeper information about versions and services.

![alt text](./img/nmap2.png)

The result comes up as a MariaDB. 

# Foothold

I couldn't access to the database in a normal way, so I decided to skip the conexión and send the query straight to the database.

![alt text](./img/sql.png)

Then inspect the database "htb":

![alt text](./img/sql2.png)

We see a "users" table, which may contain sensible information. Normally I would inspect that one, but seeing that we only have an instance of MySQL i'll inspect "config".

![alt text](./img/sql3.png)

And thare we have the flag!