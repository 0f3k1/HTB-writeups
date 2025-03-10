---

Certified

---

## General Info

- IP: 10.10.11.41
- OS: Windows  
- Difficulty: Medium

Credentials
![creds](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20114939.png)

---

## Initial Foothold

Nmap scan
![nmap scan](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20114956.png)

---

## Enumeration 

Since we have a bunch of LDAP ports open (389, 636, 3268, 3269) we can use bloodhound
![ldap ports](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115008.png)
