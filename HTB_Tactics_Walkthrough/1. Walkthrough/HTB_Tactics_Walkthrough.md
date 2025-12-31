---
title: "HTB: Tactics"
platform: "Hack The Box"
difficulty: "Easy"
os: "Windows"
date: "2025-12-30"
author: "NetSwipe"
tags: [htb, walkthrough, windows, smb]
---

# HTB Walkthrough: Tactics

## 0) Setup / Notes
- Attacker VM: `HTB Pwnbox (Kali)`
- Target IP: `10.129.16.202`
- Local IP (tun0): `10.10.15.126`
- Tools used: `ping, nmap, smbclient`
- Objective: `flag.txt`

## 1) Recon

### 1.1 Host discovery

Connectivity check
```bash
ping -c 1 10.129.16.202
```

![ping](../2.%20Images/ping.png)

Ping (ICMP echo) confirms basic reachability, but the task notes ICMP filtering, so we cannot rely on ping to discover the host. Using `-Pn` tells Nmap to skip ICMP host discovery and assume the target is up, which prevents false negatives when ICMP is blocked.

```bash
nmap -sS -Pn 10.129.16.202
```

![initial_nmap](../2.%20Images/initial_nmap.png)

The initial SYN scan (`-sS`) checks the default top ports quickly and quietly, giving us a first view of what is exposed. Ports 135 (MSRPC), 139 (NetBIOS-SSN), and 445 (SMB) are open, which is typical for a Windows target, while the rest appear filtered (no response), suggesting a firewall is dropping traffic. At this stage, SMB is the only clear entry point.

```bash
nmap -sSVC -Pn 10.129.16.202
```

![nmap_sSVC_Pn](../2.%20Images/nmap_sSVC_Pn.png)

Service and script detection (`-sV` for versions, `-sC` for default scripts) confirms a Windows host and provides more context around SMB. The scan reports SMB signing enabled but not required, which affects how SMB authentication and tooling behave later. This reinforces that SMB is the primary service to enumerate next.

```bash
nmap -sSVC -p- -Pn 10.129.16.202
```

![all_ports_nmap](../2.%20Images/all_ports_nmap.png)

An all-ports sweep (`-p-`) checks every TCP port to make sure nothing nonstandard is exposed. The results match the earlier scan, so there are no hidden services to pivot to. With only SMB available, the next steps focus on SMB enumeration and obtaining valid credentials.

### 1.2 SMB enumeration

I retry SMB authentication using the `Administrator` account. This time the login succeeds, and the share listing confirms access to the default administrative shares.

```bash
smbclient -L //10.129.16.202 -U Administrator
```

![smb_list](../2.%20Images/smb_list.png)

Next, I connect to the `C$` share to browse the filesystem. From there, I navigate into the Administrator profile to locate interesting files.

```bash
smbclient //10.129.16.202/C$ -U Administrator
```

![smb_dir_exploration1](../2.%20Images/smb_dir_exploration1.png)
![smb_exploration2](../2.%20Images/smb_exploration2.png)
![smb_exploration3](../2.%20Images/smb_exploration3.png)

Inside the SMB session, I enumerate the user directory and move into the Desktop folder where the flag is usually stored.

```text
dir
cd Users/Administrator
ls
cd Desktop
ls
```

### 1.3 Flag retrieval

Since `smbclient` does not support `cat`, I download the file locally with `get` and read it from my machine.

```text
get flag.txt
```
After downloading the file, I exit the SMB session and read it locally.

```bash
cat flag.txt
```

![flag](../2.%20Images/flag.png)

<b>FLAG:</b> f751c19eda8f61ce81827e6930a1f40c
