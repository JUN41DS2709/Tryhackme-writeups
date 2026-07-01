# TryHackMe - Introductory Networking

> **Platform:** TryHackMe
> **Module:** Introductory Networking
> **Difficulty:** Beginner
> **Status:** ✅ Completed

## Overview

This learning module introduces the fundamentals of computer networking, including:

* The OSI Model
* TCP/IP Model
* Encapsulation & De-encapsulation
* TCP vs UDP
* Network troubleshooting tools

  * Ping
  * Traceroute
  * WHOIS
  * DIG (DNS)

This README serves as my personal study notes and documentation while progressing through my cybersecurity learning journey.

---

# Task 1 - The OSI Model: An Overview

The OSI (Open Systems Interconnection) Model is a conceptual framework that standardizes communication between computer systems into seven layers.

## Questions & Answers

### Which layer would choose to send data over TCP or UDP?

**Answer:** `4`

---

### Which layer checks received information to make sure that it hasn't been corrupted?

**Answer:** `2`

---

### In which layer would data be formatted in preparation for transmission?

**Answer:** `2`

---

### Which layer transmits and receives data?

**Answer:** `1`

---

### Which layer encrypts, compresses, or otherwise transforms the initial data to give it a standardised format?

**Answer:** `6`

---

### Which layer tracks communications between the host and receiving computers?

**Answer:** `5`

---

### Which layer accepts communication requests from applications?

**Answer:** `7`

---

### Which layer handles logical addressing?

**Answer:** `3`

---

### When sending data over TCP, what would you call the "bite-sized" pieces of data?

**Answer:** `Segments`

---

### [Research] Which layer would the FTP protocol communicate with?

**Answer:** `7`

---

### Which transport layer protocol would be best suited to transmit a live video?

**Answer:** `UDP`

---

# Task 2 - Encapsulation

Encapsulation is the process of adding protocol information to data as it moves down the networking stack before transmission.

## Questions & Answers

### How would you refer to data at Layer 2 of the encapsulation process (OSI Model)?

**Answer:** `Frames`

---

### How would you refer to data at Layer 4 if the UDP protocol has been selected?

**Answer:** `Datagrams`

---

### What process would a computer perform on a received message?

**Answer:** `De-encapsulation`

---

### Which is the only layer of the OSI model to add a trailer during encapsulation?

**Answer:** `Data Link`

---

### Does encapsulation provide an extra layer of security?

**Answer:** `Aye`

---

# Task 3 - The TCP/IP Model

The TCP/IP model is the practical networking model used on the Internet today.

## Questions & Answers

### Which model was introduced first, OSI or TCP/IP?

**Answer:** `TCP/IP`

---

### Which TCP/IP layer covers the functionality of the OSI Transport layer?

**Answer:** `Transport`

---

### Which TCP/IP layer covers the functionality of the OSI Session layer?

**Answer:** `Application`

---

### The Network Interface layer covers Data Link and which other OSI layer?

**Answer:** `Physical`

---

### Which TCP/IP layer handles the functionality of the OSI Network layer?

**Answer:** `Internet`

---

### What kind of protocol is TCP?

**Answer:** `Connection based`

---

### What is SYN short for?

**Answer:** `Synchronise`

---

### What is the second step of the three-way handshake?

**Answer:** `SYN/ACK`

---

### What is the short name for the "Acknowledgement" segment?

**Answer:** `ACK`

---

# Task 5 - Ping

`ping` is a network utility used to test connectivity between hosts and measure latency.

## Questions & Answers

### What command would you use to ping the bbc.co.uk website?

```bash
ping bbc.co.uk
```

---

### Ping muirlandoracle.co.uk

### What is the IPv4 address?

**Answer:** `217.160.0.152`

---

### What switch lets you change the interval of sent ping requests?

**Answer:** `-i`

---

### What switch would allow you to restrict requests to IPv4?

**Answer:** `-4`

---

### What switch would give you a more verbose output?

**Answer:** `-v`

---

# Task 6 - Traceroute

Traceroute displays the path packets take from your computer to the destination host.

## Questions & Answers

### Use traceroute on tryhackme.com

**Answer:** `No answer needed`

---

### What switch would you use to specify an interface?

**Answer:** `-i`

---

### What switch would you use if you wanted to use TCP SYN requests?

**Answer:** `-T`

---

### Which layer of the TCP/IP model will traceroute run on by default (Windows)?

**Answer:** `Internet`

---

# Task 7 - WHOIS

WHOIS is used to retrieve domain registration information and ownership records.

## Questions & Answers

### Perform a WHOIS search on facebook.com

**Answer:** `No answer needed`

---

### What is the registrant postal code for facebook.com?

**Answer:** `94025`

---

### When was the facebook.com domain first registered?

**Answer:** `29/03/1997`

---

### Perform a WHOIS search on microsoft.com

**Answer:** `No answer needed`

---

### Which city is the registrant based in?

**Answer:** `Redmond`

---

### [OSINT] What is the name of the golf course near the registrant address?

**Answer:** `Bellevue Golf Course`

---

### What is the registered Tech Email for microsoft.com?

**Answer:** `msnhst@microsoft.com`

---

# Task 8 - DIG

`dig` (Domain Information Groper) is a command-line tool used for querying DNS servers.

## Questions & Answers

### What is DNS short for?

**Answer:** `Domain Name System`

---

### What is the first type of DNS server your computer would query?

**Answer:** `Recursive`

---

### What type of DNS server contains records specific to domain extensions?

**Answer:** `Top-level Domain`

---

### Where is the very first place your computer would look to find the IP address of a domain?

**Answer:** `Hosts File`

---

### Google runs two public DNS servers. One is 8.8.8.8. What is the other?

**Answer:** `8.8.4.4`

---

### If a DNS query has a TTL of 24 hours, what number would the dig query show?

**Answer:** `86400`

---

# Key Concepts Learned

* OSI 7-Layer Model
* TCP/IP Architecture
* Encapsulation & De-encapsulation
* TCP Three-Way Handshake
* TCP vs UDP
* Network Layers & Data Units
* ICMP & Ping
* Traceroute
* WHOIS
* DNS Resolution
* DIG Command
* Basic Networking Fundamentals

---

# Tools Used

* TryHackMe AttackBox / Kali Linux
* `ping`
* `traceroute`
* `whois`
* `dig`

---

# Completion Status

* ✅ Task 1 — OSI Model
* ✅ Task 2 — Encapsulation
* ✅ Task 3 — TCP/IP Model
* ✅ Task 5 — Ping
* ✅ Task 6 — Traceroute
* ✅ Task 7 — WHOIS
* ✅ Task 8 — DIG

---

## Notes

This repository documents my progress through TryHackMe learning modules. The purpose is to reinforce networking concepts while building a public knowledge base of my cybersecurity learning journey.
