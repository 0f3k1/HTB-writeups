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


You can download BloodHound from this [URL](https://github.com/SpecterOps/BloodHound/releases/tag/v7.0.1). After downloading the ZIP file, extract it and navigate to the `BloodHound/examples/docker-compose` directory. From there, run the following command to start the Docker container:

```docker compose pull && docker compose up```

![pass](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115016.png)

To log into BloodHound you have to reset the password using the default credentials: `spam@example.com` and `whatever the password was set to on when you ran he docker`.
![login](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115023.png)

Create a new password and you'll be logged in 
![new](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115028.png)

We can then import our zip file by navigating to the `start by uploading your data`
![start](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115035.png)
![import](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115041.png)

After exploring through Judith's path we get this nice map
![map](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115047.png)

Judtih to Management_svc
![start](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115052.png)
