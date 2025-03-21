---

# TheFrizz 

---

## General Info

- IP: 10.10.11.60 
- OS: Windows  
- Difficulty: medium

---

## Enumeration

Nmap scan revealed multiple open ports
![nmap](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-18%20094524.png)
![hosts](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-18%20094559.png)

---

## Rabbit Hole (creator said it could work but I didn't find anything)
While analyzing the web app, I identified a service and its version number. I decided to search for vulnerabilities and came across an LFI vulnerability in the ?q= parameter of the staff login page. However, after enumerating through the files, I couldn’t find anything useful.
![service](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-18%20094656.png)
![lfi](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-18%20094738.png)

---

## Further Exploitation
While searching for additional unauthenticated vulnerabilities, I discovered an Arbitrary File Write Vulnerability. One of the ways an attacker can exploit this is by uploading a base64-encoded PHP shell (e.g., <?php echo system($_GET['cmd']); ?>) via the img parameter in the rubricsvisualisesaveAjax.php script.

steps to exploit this vulnerability:

- Intercept the request using Burp Suite.
- Change the HTTP method to POST and modify the Content-Type header to x-www-form-urlencoded.
- Alter the destination directory where the PHP script would be uploaded.
- Add the img parameter containing the PHP shell and specify the file name where the shell should be stored.

![burp](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-18%20100623.png)

Accessing the shell via the url
![url](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-03-18%20101454.png)

Getting a shell by URL-encoding a PowerShell  [reverse shell](https://www.revshells.com/) payload and injecting it into the ?cmd= parameter in the URL
![rev](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20102253.png)
![urlencod](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20101949.png)
![url](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20102010.png)

Intercepting the request with netcat  
![shell](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20101935.png)

Looking through the shell, theres an interesting file containing database credentials
![gibbon](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20102103.png)
![mysql db](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20102118.png)

Going back two directories leads us to a bunch of exectuable files

![db](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20102653.png)

One of them being a mysql executable. Using the credentials in the config.php file to run commands
![mysql.exe](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20102734.png)

Listing all databases

![db](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20104048.png)

Listing all tables within the Gibbon database
![gibbondb](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20104209.png)
![tables](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20104221.png)

After searching through the tables, I found a hash in the `gibbonperson` table
![hash](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20104238.png)

We can crack the salted hash by using the 1420 format on hashcat (hash:/salt)
![command](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20104551.png)
![format](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20105023.png)
![cracked](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20104910.png)

---

## getting a shell
Great, so we have fionas password we can try to ssh, but we get this permission denied error 
![error](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20105259.png)

gssapi-with-mic is a method to authenticate users and one of the ways it does that is Kerberos authentication. The kerberos realm (/etc/krb5.conf) is where you configure the authentication servers (KDCs). In other words, it's like the "kerberos system configuration" for kerberos related stuff, to let the system know where the server is.

![conf](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20111535.png)

Once the krb5.conf file is configured, we still need to authenticate with the Kerberos system. This is done by generating a Ticket-Granting Ticket (TGT) from the KDC, which is stored in the Kerberos ticket cache. After that, you set an environment variable to let SSH know the authentication is set up properly, allowing you to connect.

Using `kinit` to generate the tgt, and use `klist` to list out where the cache is stored
![ticket](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20112136.png)

Setting the enviornment variable

![env](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20112325.png)

SSH into f.frizzle and get user flag

![user](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20112609.png)

---

## Privilege Escalation

Users existing on the domain

![user](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20120254.png)

After enumerating for quite some time I couldn't find anything, so I looked in some of the more uncommon areas, such as the Recycle Bin. 
![recyle bin](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-20%20103419.png)

Command explanation:
`gci -Force 'C:\$Recycle.Bin' -Recurse`
- `gci` lists files and and directories
- `-Force` show hidden and system files
-  `C:\$Recycle.Bin` target directory
-  `-Recurse` lists all files and subdirectories recursively

Using `scp` to copy the file onto my machine
![scp](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20120101.png)
![scp1](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20120132.png)

Since its a 7zip file, we can extract it using the command `7z` which puts it into a "wapt" folder
![7x](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20120154.png)
![wapt](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20120223.png)

After manually looking through some of the folders, I couldnt find anything, so I used grep to recursively search through all the folders and files.

![manual](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20120718.png)
![automatic](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20121053.png)

The password is base64-encoded, so I decoded it using the [the base64decoder](https://www.base64decode.org/)
![decode](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20121145.png)

There are two ways we can go about identifying which password this user belongs to. We can either brute-force through all of the users listed on the domain (which might take a minute to set up), or we can simply reverse the order of the password by using the `rev` bash command, which gives a good indication of who this password belongs to.

![rev](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20122419.png)

Looking back through the net users results, we found a user called M.SchoolBus, so we can try to SSH into it.

We get the same permission denied error

![error](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20122653.png)

We can repeat the same process: set up a ticket, set the environment variable, and SSH into M.SchoolBus.
![ssh](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20122749.png)

Next, run nxc with BloodHound (-k for Kerberos authentication) to gather information about M.SchoolBus.
![nxc](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20153553.png)


We can use BloodHound to see what privileges the user M.SchoolBus has over certain resources.

Start the bloodhound docker container and access bloodhound via the web UI on localhost port 8080.
![docker](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20153641.png)

Search for the user and use the Cypher query box (e.g., MATCH p=(u:User)-[r1]->(n) WHERE r1.isacl=true RETURN p) to view all the ACLs for that specific user. In this case, M.SchoolBus has multiple privileges over certain resources, but we can focus on the WriteGPlink privilege over the domain controller. 
![Screenshot 2025-03-18 154123](https://github.com/user-attachments/assets/e3acb6b4-e3bf-4498-8623-41508d682e75)

Short explanation:

In Active Directory, the gPLink attribute controls which Group Policy Objects (GPOs) are applied to an Organizational Unit (OU). GPOs enforce security settings, scripts, and system policies for users and computers.

By default, only Domain Admins and Enterprise Admins can modify the gPLink. However, if an hackers gains sufficient permissions on an OU, they can:

Link a malicious GPO to the OU.
Force computers in that OU to run startup scripts, scheduled tasks, or registry changes.
Achieve SYSTEM-level execution, gaining full control over the affected machines.

The process goes as follows:

Transfer the [precompiled sharpgpoabuse script](https://github.com/byronkg/SharpGPOAbuse/releases/tag/1.0) script to the target machine.
![files](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-18%20172247.png)

Create a new GPO name.

![GPO](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-19%20104038.png)

Link the new GPO to the domain.
![link](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-19%20104222.png)

Make M.SchoolBus a local administrator on all machines affected by the GPO. Also, force the Group Policy to update immediately on the affected machines.
![admin](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-19%20104401.png)
![verify](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-19%20104437.png)

Since the change is only applied to new sessions, we can use [RunasCs](https://github.com/antonioCoco/RunasCs/releases) to escalate to administrator and obtain the root flag.
![runas](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-19%20105716.png)

![root flag](https://github.com/J4ck3lXploit/editing_htb-writeups/blob/main/images/Screenshot%202025-03-19%20164824.png)



