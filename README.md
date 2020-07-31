# SmagWriteUp
So I'll show you my way to hack this very nice, but no so easy tryhackme room :
Smag
# Recon
First, we need to scan with nmap, as always :
![Capture d’écran de 2020-07-31 18-50-30](https://user-images.githubusercontent.com/50116433/89050911-b889b400-d353-11ea-8889-df85a7b67fdb.png)
We see http and ssh. The apache webserver isn't vulnerable and ssh isn't either.
Let's run an nmap scan on all ports in the background, just in case : 
```nmap -p- 10.10.43.30```
# Get a shell
Now I wanna look at the webserver, but it's under construction so nothing appears to be here...
I added smag.thm to /etc/hosts, but nothing changed.

One thing you should do everytime you're on a webpage is checking the file extension. Here the index file is index.php
The webserver is running php. 
Let's run gobuster, since this is the only thing we can do now :
![Capture d’écran de 2020-07-31 18-58-27](https://user-images.githubusercontent.com/50116433/89051347-68f7b800-d354-11ea-887f-5454af53f124.png)

You noticed that I used the .php extension, and the txt too ( I always, it's just in case )
After finding /mail we can see a page saying the server is using mail2web software to displays mails on the page, but I didn't find anything with it in searchsploit :(
There is a .pcap file, after opening it in wireshark we can see an http request with a username and a password. Nice !
I tried to use these in ssh but it didn't work.
![censored_stuff](https://user-images.githubusercontent.com/50116433/89052543-26cf7600-d356-11ea-8f2c-34e3d0ba5225.jpg)
We also notice that the http request is going to development.smage.htb, and after adding it to my /etc/hosts files.
The domain development.swag.thm is redirecting to a login page where we can login with the credentials we found earlier.
There is an admin command pannel :
![Capture d’écran de 2020-07-31 19-08-13](https://user-images.githubusercontent.com/50116433/89053375-65b1fb80-d357-11ea-80c1-3d531d7418f1.png)
I try to ping my machine, just to see if it works, I use tcpdump to listen for incoming ping request : 
```sudo tcpdump -i tun0 icmp and icmp[icmptype]=icmp-echo```
Then I run the command on the webpage and tadaaa:
![Capture d’écran de 2020-07-31 19-08-33](https://user-images.githubusercontent.com/50116433/89053686-e40e9d80-d357-11ea-92fe-cfa5eb7c0dd5.png)
It's working ! Unfortunately, it's not displaying any output, even in the source code, so we cannot read flags.
I tried to wget a php file from my machine, and the file is downloaded correctly by the machine but somehow I couldn't have access to it with my browser... 
At this point I tried many reverse shells, but I was sending commands with burp, and they didn't worked because the url encoding wasn't correctly executed.
I tried again few hours later but I was sending my reverse sshells command from the website and after few minutes I remebered that the webserver was running php, so php should be installed on the machine. And it was : my reverse php shell worked !
![Capture d’écran de 2020-07-31 19-11-31](https://user-images.githubusercontent.com/50116433/89054531-42884b80-d359-11ea-8237-04c09f3f4b4c.png
# PrivEsc
So there is a user jake, but we cannot acces the flag in his home directory and we can't either read the .ssh folder.
I ran our favourite enumeration tool : [LinPeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS) !
