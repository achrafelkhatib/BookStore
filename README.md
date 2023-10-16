# BookStore
This is a CTF write-up part of the tryhackme box called the bookstore

The first thing to do when enumerating a target is to look at what's open for access so I decided to do a Nmap scan on the target IP address

 ![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/b2b1de57-1e12-4483-92d2-7fef0be5S4312)

I've found that port 80 which indicates a web page, and port 5000 open so i decided to check it

![image](https://github.com/achrafelkhatib/BookStore/assets/61394291/b26051f7-ab73-477b-a695-881e1e40ad16)

I found that they are using a Foxy REST API v2.0 which would indicate an outdated version
