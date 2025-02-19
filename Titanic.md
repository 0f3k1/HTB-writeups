---
Titanic 
---
### General Info

- **IP:** 10.10.11.37
- **OS:** Linux
- **Difficulty:** Easy

### Initial Foothold
An Nmap scan revealed two open TCP ports:
- **22/tcp (SSH):** OpenSSH 8.9p1 running on Ubuntu.
- **80/tcp (HTTP):** Web application

![nmap scan](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20132816.png)

### Enumeration
Exploring the web application, there isn’t much to see, except for an interesting **"Book Now"** button.
![web app](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20132927.png)

Clicking it brings up a form, and after filling it out, the site generates a downloadable **JSON file** containing the information you just submitted.
![book now](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-19%20114251.png)
![json](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20133433.png)

Since the web app only allows you to download a JSON file, direct input manipulation isn’t possible. However, we can use **Burp Suite** to help analyze what happens when the form is submitted.
![burp suite](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20133530.png)

Following the redirection leads to a `/download` page containing a **ticket** parameter, which retrieves the JSON file.
![redirect](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20133703.png)

At this point, we can test if the **ticket** parameter is vulnerable to **Local File Inclusion (LFI)** by attempting to access system files such as `/etc/passwd`. 
![lfi](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20134052.png)

I couldn't find anything useful initially, so I bookmarked the section to revisit later.

I looked for subdomains and hidden pages using ffuf to see if anything popped up.
![ffuf scan](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20134624.png)

Brute-forcing revealed a subdomain called dev, which led me to a Gitea web application.
![gitea](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20134803.png)
Gitea is an open-source Git repository manager, similar to GitHub and GitLab.

Exploring the Gitea web app, I came across a Users section that contained a list of multiple user accounts.
![users](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20135029.png)

After checking each user's repositories, I found a **docker-config** repository belonging to the user **developer**. Inside, there were a few files that stood out.
![repo](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20135134.png)
![files](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20135205.png)

Inside the `mysql` folder, I found a **MySQL** username and password, which might be useful once we gain access to the machine.
![mysql creds](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20135336.png)

Inside the gitea folder, the path /home/developer/gitea/ caught my eye. I thought this could be useful, possibly in combination with the LFI vulnerability from the ticket parameter.
![gitea path](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20135314.png)

Looking at the Gitea documentation for potential misconfigurations or exploitable paths, I came across an interesting file called `app.ini`, located in the `conf` directory.
![documentation](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20135931.png)

Going back to the **ticket** parameter, we can try accessing the contents of `app.ini` by combining the path found in the **docker-config** repo with the **app.ini** path. This approach works, and we successfully retrieve more information about the configuration of the web app.
![lfi path](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20140104.png)

After reviewing all the files from the Burp Suite response, I noticed a gitea.db file in the database section.
![database](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20140441.png)

Since the retrieved format is messy and difficult to read, I downloaded the file to my system.
![downloadsing](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20140653.png)

Since the database is an SQLite database, I used sqlite3 to open and explore its contents in a more readable format.
![sqlite](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20140927.png)

To open a table or view its contents, use the command select * from <table>; 
![hashes](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20141214.png)

### Rabbit hole
Initially, I thought cracking the Gitea hashes with Hashcat would be straightforward. I put all the PBKDF2 hashes into a file, ran them through Hashcat using the correct format, and tried to get the password to access the system. 

### Continuing to crack the hashes (0xdf explanation)
However, it turned out to be more complicated than I expected. After some trial and error, I decided to research further and came across a helpful HTB write-up by 0xdf, [writeup](https://0xdf.gitlab.io/2024/12/14/htb-compiled.html#)

He explained that brute-forcing the Gitea hashes isn't simple because Gitea uses PBKDF2 with SHA256, and the hashes are stored in hex, whereas Hashcat expects them in base64. 
![hash format](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-18%20135139.png)

After using oxdf's method to properly format the hashes, I was able to convert them, crack the password, and proceed to the next steps
![formating hashes](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-17%20142557.png)
![cracking hash](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-18%20135709.png)
![cracked hash](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-18%20135719.png)

### Gaining Access
Since I now had credentials, I could SSH into the developer account and retrieve the user flag.
![user shell](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-18%20135827.png)

### Privilege Escalation
I checked if the developer account had any specific privileges, but it had none.
![sudo privs](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-18%20181050.png)

I ran LinPEAS on the target and found an interesting executable file `magick` 
![magick](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-18%20181344.png)
![version](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-18%20182156.png)

I also found a file in the `/opt/scripts` directory.
![directory](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-18%20181458.png)

The script goes to the `/opt/app/static/assets/images` folder, clears the `metadata.log` file, and finds all `.jpg` files. It then uses `magick identify` to gather information about each image and saves it to `metadata.log`
![bash script](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-18%20181441.png)

After researching ImageMagick vulnerabilities, I found an interesting proof of concept (PoC) that exploits how ImageMagick handles environment variables. 
![PoC](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-18%20182254.png)

There are essentially two ways to exploit this vulnerability [privEsc](https://github.com/ImageMagick/ImageMagick/security/advisories/GHSA-8rxc-922v-phg8). The first involves abusing `MAGICK_CONFIGURE_PATH`, which allows loading a malicious configuration file from the current directory. The second, which I used, takes advantage of how files are executed in `/opt/app/static/assets/images` by creating a fake `libc` file there. 

We can use the `gcc` command to compile the code into a malicious shared library (`libxcb.so.1`). After compiling the library, we execute `magick /dev/null /dev/null`, which triggers the library to load and executes the `system("chmod u+s /bin/bash")` command from within the library. Finally, we can use the command `/bin/bash -p` to spawn a root shell, as the `chmod` command sets the `setuid` bit on `/bin/bash`, granting us elevated privileges.
![root](https://github.com/J4ck3lXploit/HTB-writeups/blob/main/Images/Screenshot%202025-02-18%20183225.png)
