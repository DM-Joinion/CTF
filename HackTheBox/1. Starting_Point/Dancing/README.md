![alt text](./img/Dancing.png)

# Enumeration

## NMAP

I start with nmap to see what ports are open and what services are running:

![imagen_nmap](./img/nmap.png)

Now lets see deeper information about versions and services.

![alt text](./img/nmap2.png)

```
135/tcp   - RPC      - Windows internal communication, not directly exploitable
139/445   - SMB      - File and printer sharing, high potential for enumeration
5985/tcp  - WinRM    - Windows Remote Management, allows remote command execution
47001/tcp - WinRM    - Alternative port for WinRM
```
With this information, the most obvious one is the SMB running on port 445. 
> It sticks out the service which port 445 is running, "microsoftds", which should contain shared files with sensible info

## SMB

>SMB (Server Message Block) is a network protocol that allows sharing files, printers, and other resources between computers within a network.

```
smbcclient -L <ip> -N
```
![alt text](./img/smb.png)

- The next step is to try accessing each shared data:

1. 
![alt text](./img/smb_admin.png)

2. 
![alt text](./img/smb_C.png)

3. 
![alt text](./img/smb_ws.png)

- The only data we have access to is 'WorkShares'. Let's see what's inside:

First of all, we need to see which comands are available with `help` comand.

![alt text](./img/smb_help.png)

Once we now the comands we can use, its time to navigate the shared resource.

![alt text](./img/smb_content.png)

It can be seen that there are some directories, inside there's:

Amy.J:  
![alt text](./img/smb_amy.png)

James.P:  
![alt text](./img/smb_james.png)


> The <u>single</u> flag is inside the last enumerated directory