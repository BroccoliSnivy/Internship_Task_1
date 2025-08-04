# Internship – Day 1: Local Network Reconnaissance

## Task: Identify Local IP Range & Perform Basic Network Scan

---

### *Note: I am using Parrot OS on baremetal due to which network interfaces and open ports may differ.*

### Step 1: Identifying My Local IP Range

To begin, I used the `ip` command to list all my network interfaces and IPs:

```bash
┌─[spartan@parrot]─[~/Desktop]
└──╼ $ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp2s0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether xx.xx.xx.xx.xx.xx brd ff:ff:ff:ff:ff:ff
3: enxeeb354c2240d: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 1000
    link/ether xx.xx.xx.xx.xx.xx brd ff:ff:ff:ff:ff:ff
    inet 192.168.xx.xxx/24 brd 192.168.xx.255 scope global dynamic noprefixroute enxeeb354c2240d
       valid_lft 2463sec preferred_lft 2463sec
    inet6 XXXX:XXXX::XXXX:XXXX:XXXX:XXXX:XXXX/64 scope global dynamic noprefixroute 
       valid_lft 6997sec preferred_lft 6997sec
    inet6 abcd::XXXX:XXXX:XXXX:XXXX/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

## Step 2 : Performing the scan.

Here we can note that the interface 3, enxeeb354c2240d has a local IP range 192.168.xx.xxx/24 in this manner. (Let's say your's is 192.168.23.103). Then we can use the command ``sudo nmap -sS 192.168.23.103/24``.

Explaination of this command :

- sudo : nmap's ``-sS`` options requires root previlages.
  
- -sS : Perform's stealthy scan by just sending SYN packets to detect full open ports wihout completing the full TCP handshake.
  
- 192.168.23.103/24: The ip range we are supposed to scan.
  

After executing the command, these are the scan results we obtained.

*Note that the scan is performed on Parrot OS baremetal.*

```bash
┌─[spartan@parrot]─[~/Desktop]
└──╼ $sudo nmap -sS 192.168.23.103/24
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-08-04 22:53 IST
Nmap scan report for 192.168.23.103
Host is up (0.00055s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
53/tcp open  domain
MAC Address: xx:xx:xx:xx:xx:xx (Unknown)

Nmap scan report for 192.168.23.103
Host is up (0.0000060s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
53/tcp open  domain

Nmap done: 256 IP addresses (2 hosts up) scanned in 19.68 seconds
```

- **Port 53 (DNS):**  
  Translates human-readable domain names (like `google.com`) into IP addresses that computers use to communicate. It typically uses UDP for queries but can use TCP for larger responses like zone transfers. Attackers can abuse it for DNS poisoning, data exfiltration, or command-and-control traffic, so it's important to monitor carefully.

## Let's try scanning a more promosing target.

I have setup a vulnerable *Windows XP* vm on virutal box that is inside a Virtual Network so the it doesn't connect with the internet but is present for us to perform attacks. First as usual perform the ``ip`` command to indentify the network interfaces. This is the result we get :

```bash
4: vboxnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 0a:00:27:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.1/24 brd 192.168.56.255 scope global vboxnet0
       valid_lft forever preferred_lft forever
    inet6 xxxx::xxxx:xxxx:xxxx:0/64 scope link proto kernel_ll 
       valid_lft forever preferred_lft forever
```

The ip range we find for the vulnerable Windows XP vm is ```192.168.56.1/24``` . Let's perform the same stealth scan on this machine and see the out.

```bash
┌─[spartan@parrot]─[~/Desktop]
└──╼ $sudo nmap -sS 192.168.56.101/24
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-08-04 22:59 IST
Nmap scan report for 192.168.56.100
Host is up (0.000019s latency).
All 1000 scanned ports on 192.168.56.100 are in ignored states.
Not shown: 1000 filtered tcp ports (proto-unreach)
MAC Address: 08:00:27:97:69:44 (Oracle VirtualBox virtual NIC)

Nmap scan report for 192.168.56.101
Host is up (0.00020s latency).
Not shown: 991 closed tcp ports (reset)
PORT      STATE SERVICE
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
MAC Address: 08:00:27:F3:9D:C9 (Oracle VirtualBox virtual NIC)

Nmap scan report for 192.168.56.1
Host is up (0.0000020s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE
53/tcp open  domain

Nmap done: 256 IP addresses (3 hosts up) scanned in 19.07 seconds
```

- **Port 135 (msrpc):** 
  This port is used by Microsoft RPC (Remote Procedure Call) services. It allows Windows components to communicate over the network. Vulnerabilities in this service have been exploited in the past by malware like the Blaster worm, making it a potential security risk if exposed.
  
- **Port 139 (netbios-ssn):**  
  NetBIOS over TCP/IP uses this port for file and printer sharing in older Windows systems. It can be abused for information gathering through null sessions, where attackers connect without authentication and enumerate users, shares, and policies.
  
- **Port 445 (microsoft-ds):**  
  Port 445 is used by SMB (Server Message Block) over TCP for file sharing and network services. It has been heavily targeted by exploits like EternalBlue, which enabled the spread of ransomware like WannaCry. This port is high-risk if not properly secured.
  

---

## Capturing Packets using Wireshark

Anything happening on a specific network interface can be seen and analysed appropriately. In the packet capture file available along with this write, I have performed different Nmap scans to get different type of packets to be captured on the network. This packet capture is performed by performing different types of nmap scans on the vulnerable "Windows XP" virtual machine.

### Given below are the Nmap scan options I used on windows xp vm and captured them

- `nmap -sS 192.168.56.101` (Stealth scan)
  
- `nmap -sT 192.168.56.101` (TCP connect)
  
- `nmap -sU 192.168.56.101` (UDP scan)
  
- `nmap -sA 192.168.56.101` (ACK scan)
  
- `nmap -sN 192.168.56.101` (Null scan)
  
- `nmap -O 192.168.56.101` (OS detection)
  

---

## Potential security risks from open ports

###### **Port 135 (TCP) — MSRPC**

Used by Microsoft Remote Procedure Call. Helps Windows services talk to each other, like for DCOM and admin tasks.

**Risk:** Often targeted in remote code execution attacks (like WannaCry). Should not be open to the internet.

### **Port 139 (TCP) — NetBIOS Session Service**

Used for file and printer sharing in older Windows systems.

**Risk:** Can expose shared folders and usernames. Attackers can also try password brute-forcing.

### **Port 445 (TCP) — Microsoft-DS / SMB**

Used for file sharing, printer access, and authentication in Windows networks.

**Risk:** Very high. This port is often used in major exploits like EternalBlue. Leaving it open can lead to full system compromise.

### **Port 53 (TCP) — DNS**

Used for DNS queries, especially large ones or zone transfers.

**Risk:** If this system is not meant to be a DNS server, it may be misconfigured. Could be abused for DNS amplification or data leaks.
