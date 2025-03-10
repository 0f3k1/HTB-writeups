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

Judtih to Management
![start](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115052.png)

Judith has WriteOwner privileges over the management group meaning we can add Judith to the management group.

How it works:
- Using `owneredit.py` Judith can change the owner of the Management group to themselves which gives them full control over the group

![join](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115058.png)

- Using `dacledit.py` Judith grants themselves "AddMember" permissions on the group which means they can add themselves into the group.

![privs](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115105.png)

- Using `net rpc group addmem` we can add Judith to the group.

![add](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115111.png)

Management to Managament_svc
![manage](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115118.png)

Since we have Generic Write privileges over the user `management_svc` we can launch a shadow credential attack. 

How it works:
- Using `pywhisker` we can generate a certificate. The public key from this certificate is then stored in the `msDS-KeyCredentialLink` attribute in Active Directory for the target use.

![cert](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115335.png)

- Since we generated a certificate, we now have the public key, which allows us to utilize a tool called PKINIT to authenticate to the Key Distribution Center (KDC) and obtain the users TGT and encryption key.

![tgt](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115353.png)

- Once Judith has the TGT, you can set the environment variable to point to the **TGT cache**. Which allows them to reuse the TGT for further authentication without needing to re-authenticate each time.

![cache](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115359.png)

- To retrieve the NT hash we can use the `getnthash` script to give us the management_svc users hash.

![NT hash](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115405.png)

Evil-winrm as MANAGEMENT_SVC
![user](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-10%20115411.png)
