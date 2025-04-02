# Post Exploitation

## **File Transfers**

we sometimes have to host the files. We usually do it via python HTTP server.

In windows we can use certutil.exe

If we have RDP access to the machine we can just navigate to the file that has been hosted on the website address.

If certutil is getting blocked, it seems suspicious. Both Wnidows and Linux have FTP features

We can also use Metasploit. If you have a meterpreter shell it is very easy to upload and download a file.

![image.png](image.png)

## **Maintaining Access**

If something were to happen to the machine that we had access to. We have a way to get back access to that machine. If we have a shell on a machine and the user shuts down the computer, we are going to lose that shell. We can try to maintain access via Command and control.

Typically as a hacker you wont do it. Mostly you would add a user.

In a Pentest, not a red team engagement, if the team is not catching you. You will add a local user to the machine and can access it later using psexec and have a shell on it.

The first method is metasploit, and are dangerous as you are opening a port on a machine with zero authentication. As soon as you connect to that machine using that port you will get a shell, which is very dangerous.

The schedule method works like you have a malware on the system that runs on a schedule and if the system gets rebooted, the script will run when it is scheduled and you will get the shell access again.

It falls in line with red teaming, which is way more advanced then pentesting.

![image.png](image%201.png)

## **Pivoting**

Imagine if you have compromised a machine and it allows you access to 2 network interfaces, and those 2 NIC’s share a new network that was originally unavailable to you. We can setup a **proxy** and **pivot** through that to the new network.

![image.png](image%202.png)

we ssh into a machine, pivot is just the identity file that we are going to access through. we have root access, it is an Ubuntu machine.

![image.png](image%203.png)

![image.png](image%204.png)

We can see that this machine has 2 IP addresses, one is 10.10.155.5 and the other is 10.10.10.5.

If we try to ping the second IP address in a new tab we won’t get anything back. Because we can’t access that network, we don’t have a route to that network, we only have route to first network.

So we need to establish a pivot through first machine to access the second network.

### Pivoting using Proxychains

```bash
cat /etc/proxychains4.conf  # first we edit its conf file.
```

![image.png](image%205.png)

We are going to bind to the port 9050. You can change this too.

```bash
ssh -f -N -D 9050 -i pivot root@10.10.155.5
```

-f: It just backgrounds the SSH, nothing will happen once we run the command, it will just go into the background.

-N: We do not want to execute any commands, it is ideal for port forwarding.

-D: We are going to bind the port on 9050.

Now, we establish a connection on this machine and now we can proxy our traffic through this machine to access that next network.

In a new tab we can  run the following command

```bash
proxychains nmap -p88 10.10.10.225
```

we can run nmap through proxychains. proxychains <any-command>

we are using port 88 as there is a domain controller sitting on this machine.

![image.png](image%206.png)

I know that 88 is kerberos and it should exist.

You can do port scanning as well, and it will say OK if the port is open and socket disconnected if it isnt.

```bash
proxychains nmap 10.10.10.225
```

sometimes the scan doesnt work very well and you know that there is something on that network, so you can run the fol command

```bash
proxychains nmap 10.10.10.225 -sT
```

It is a TCP connect scan as apposed to running a TCP SYN scan, which is default. Somewhere it works somewhere it doesnt. 

You can also run attacks using proxychains like the kerberoast attack.

```bash
proxychains GetUserSPNs.py MARVEL.local/fcastle:Password1 -dc-ip 10.10.10.225 -request
```

We can access the machine using RDP by fol command

```bash
proxychains xfreerdp /u:administrator /p:'Hacker321!' /v:10.10.10.225
```

close the browser in you machine and you can open the browser using proxychains. If you have web addresses that are on that network, you can access them by using firefox through proxychains.

```bash
proxychains firefox
```

### Pivoting using sshuttle

```bash
sudo pip install sshuttle
```

we are not going to use port here, we are just going to say that I am this root user and I am going to establiss connection to this new network that I know about (10.10.10.0/24). And I want to run the following ssh command.

```bash
sshuttle -r root@10.10.155.5 10.10.10.0/24 --ssh-cmd "ssh-i pivot"
```

![image.png](image%207.png)

dont worry about the error.

As long as this is open you can perform anything on the new network. You dont need to add proxychains or sshuttle before every command.

### Pivoting using chisel

[https://github.com/jpillora/chisel](https://github.com/jpillora/chisel)

written in golang.

## **Cleaning Up**

In pentest perspective:

make sure you make the network as it was before you started.

![image.png](image%208.png)

In red teaming perspective:

make sure to make it like you were never there.

- Eliminating yourself from the log files, so you are harder to track in forensics.