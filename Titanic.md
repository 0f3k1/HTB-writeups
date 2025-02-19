---
Titanic 
---
### General Info

- **IP:** 10.10.11.37
- **OS:** Linux
- **Difficulty:** Easy


### Initial Foothold
Running an **Nmap** scan revealed two open TCP ports:
- **22/tcp (SSH):** OpenSSH 8.9p1 running on Ubuntu.
- **80/tcp (HTTP):** Web application
![nmap scan](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20132816.png)


### Enumeration
Exploring the web application, there isn’t much to see at first, except for an interesting **"Book Now"** button.
![web app](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20132927.png)
