# eJPT/Pen-Testing Notes
These are my personal notes and commands to use for the eJPT and general pen-testing.

Any links you see refer to the section it resides in.

## Ping Sweeps

### fping:

- The ***-a*** option forces the tool to show only alive hosts.
- The ***-g*** option tells the tool that we want to perform a ping sweep instead of a standard ping.

`fping -a -g {IP Range} 2>/dev/null`

**fping example:**

`fping -a -g 192.168.51.0/24 2>/dev/null`

## Nmap

The syntax for Nmap looks like the following:

`nmap <scan types> <options> <target>`

The following are some commonly used scan techniques:

`-A: Aggressive option`

***This scans the target with multiple options as service detection (-sV), OS detection (-O), traceroute (--traceroute), and with the default NSE scripts (-sC).***

`-sA: TCP ACK scan`

***This scan only sends a TCP packet with only the ACK flag. The packets with the ACK flag are often passed by the firewall because the firewall cannot determine whether the connection was first established from the external network or the internal network.***

`-sC: Script scan`
  
***Runs a set of default scripts against the target(s)***

`-sF: FIN scan`
  
***Sometimes, because of firewall, SYN Packets might be blocked. In such case, FIN Scan works by by passing the firewall. FIN packets are send to closed ports, if no response is received, it is because either the packet is dropped by firewall or the port is open.***

`-sI: Idle scan`

***Idle scan is an advance scan that does not send any packets from your IP address, instead it uses another host from the target network to send the packets.***

`-sP: Ping scan`

***This technique is only used to find out whether the host is available or not. Ping Scan is not used to detect open ports.***

`-sS: TCP SYN scan`

***Nmap sends SYN packets to the destination, but does not create any session. As a result, target computer won’t be able to create any log of interaction as no session was initiated.***

`-sT: TCP Connect() scan`

***UNIX socket uses a system call named connect() to begin TCP connection and if it succeeds, connection can be made and if it fails, connections cannot be made, basically because the port might be closed.***

`-sU: UDP scan`

***If sU receives an error message stating that the ICMP is unreachable, this means that the port is closed. But, if gets any approachable response, then it means the port is open.***

`-sV: Version Detection`

***This technique is used to find out about specific service running on open port, it’s version and product Name. It is not used to detect open ports. However, this scan needs open ports in order to detect the version.***

`--packet-trace`

***Nmap will create an additional line of output for every packet sent or received.***

## Enumerating the Hosts Found on the Network

### Host Discovery:

Always store every single scan.

`sudo nmap 1.2.3.0/24 -sn -oA tnet | grep for | cut -d" " -f5`

### Port Scanning:

There are 6 different states for a scanned port:

**open:**	

***This indicates that the connection to the scanned port has been established. These connections can be TCP connections, UDP datagrams as well as SCTP associations.***

**closed:**

***When the port is shown as closed, the TCP protocol indicates that the packet we received back contains an RST flag. This scanning method can also be used to determine if our target is alive or not.***
***filtered:***	

***Nmap cannot correctly identify whether the scanned port is open or closed because either no response is returned from the target for the port or we get an error code from the target.***

**unfiltered:**

***This state of a port only occurs during the TCP-ACK scan and means that the port is accessible, but it cannot be determined whether it is open or closed.***

**open|filtered:**

***If we do not get a response for a specific port, Nmap will set it to that state.This indicates that a firewall or packet filter may protect the port.***

**closed|filtered:**	

***This state only occurs in the IP ID idle scans and indicates that it was impossible to determine if the scanned port is closed or filtered by a firewall***

### OS Detection:

**OS Fingerprinting**

During a penetration test, you will have to perform this step on every network node, i.e.:
- Routers
- Firewalls
- Hosts
- Servers
- Printers

**OS Fingerprinting with Nmap**
`-O:` Enables OS detection.
`-Pn:` use to skip the ping scan if you already know that the targets are alive.

`nmap -Pn -O {IP Addresses/target(s)}`

You can fine-tune the OS fingerprinting process by using the following options:

**OS Detection:**
    `--osscan-limit: Limit OS detection to promising targets`
    `--osscan-guess: Guess the OS more aggressively`

### Timing:
Nmap offers six different timing templates to use. The 0-5 values dertimine the aggressiveness of the scan:

`-T 0 : -T paranoid`
`-T 1 : -T sneaky`
`-T 2 : -T polite`
`-T 3 : -T normal` ***This one is used by default, so it will not do anything***
`-T 4 : -T aggressive`
`-T 5 : -T insane`

https://nmap.org/book/performance-timing-templates.html

**Nmap TCP Quick Scan**

'''
nmap -sC -sV {IP Address}
'''

**Nmap TCP Full Scan**

'''
nmap -sC -sV -p- {IP Address}
'''

**Nmap UDP Quick Scan**

'''
nmap -sU -sV {IP Address}
'''

### Testing Firewalls: 

If you run into this result when running an nmap scan:

`Too many fingerprints match this host to give specific OS details`

Manually specify the source IP address to test if we get better results.

`sudo nmap **IP we're testing** -n -p **port that is giving the result** -O -S ***different source IP** -e tun0`

example: `sudo nmap 10.43.1.27 -n -p 445 -O -S 10.43.1.200 -e tun0`

### DNS Proxying:

By default, Nmap performs a reverse DNS resolution unless otherwise specified to find more important information about our target. These DNS queries are also passed in most cases because the given web server is supposed to be found and visited. The DNS queries are made over the UDP port 53. The TCP port 53 was previously only used for the so-called "Zone transfers" between the DNS servers or data transfer larger than 512 bytes. 

Nmap gives us a way to specify DNS servers ourselves `--dns-server <ns>,<ns>`

This means we could use them to interact with the hosts of the internal network. Moreover, we can use TCP port 53 as a source port (--source-port) for our scans. If the administrator uses the firewall to control this port and does not filter IDS/IPS properly, our TCP packets will be trusted and passed through.

For example, say we run this **SYN-Scan of a filtered port:**

`sudo nmap 10.134.3.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace`

And we get this result:

`PORT      STATE    SERVICE`
`50000/tcp filtered ibm-db2`

Now let's try running a **SYN-Scan from a DNS Port:**

`sudo nmap 10.134.3.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53`

We instead get this result:

`PORT      STATE SERVICE`
`50000/tcp open  ibm-db2`
`MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)`

Now that we have found out that the firewall accepts TCP port 53, it is very likely that IDS/IPS filters might also be configured much weaker than others. We can test this by trying to connect to this port by using Netcat.

**Connect to the filtered port wit Netcat:**

`ncat -nv --source-port 53 10.134.3.28 50000`

If it's successful, you would see something like the following:

`Ncat: Version 7.80 ( https://nmap.org/ncat )`
`Ncat: Connected to 10.129.2.28:50000.`
`220 ProFTPd`

**Find Common Vulnerabilities**

**Common Ports with Vulnerabilities**

**Port: Protocol**

`21: FTP`

`22: SSH`

`23: TELNET`

`25: SMTP`

`53: DNS`

`80: HTTP`

`443: HTTPS`

`110: POP3`

`115: SFTP`

`143: IMAP`

`135: MSRPC`

`137: NETBIOS`

`138: NETBIOS`

`139: NETBIOS`

`445: SMB`

`3306: MYSQL`

`1433: MYSQL`

`3389: RDP`

**Use Nmap as a Lightweight Vulnerability Scanner**

**Port 21 - FTP Enumeration**

FTP Exploitation Methodology:
''' 
1. Gather version numbers
2. Check Searchsploit
3. Check for Default Creds
4. Use Creds previously gathered
5. Download the software
'''

Enumerate FTP Service with Nmap:

'''
nmap --script=ftp-anon,ftp-bounce,ftp-libopie,ftp-proftpd-backdoor,ftp-vsftpd-backdoor,ftp-vuln-cve2010-4221,tftp-enum -p 21 {IP Address}
'''

Check for FTP Vulnerabilities with Nmap:
'''
nmap --script=ftp-* -p 21 {IP Address}
'''

Connect to FTP Service:
'''
ftp {IP Address}
'''

'''
ncftp {IP Address}
'''

***Some FTP Default Credentials***
- username: ***anonymous*** password: ***anonymous***
- username: ***admin***  password: ***password***

Bruteforce FTP with a Known Username:
'''
hydra -l $user -P /usr/share/john/password.lst ftp://10.10.10.1:21
'''

'''
hydra -l $user -P /usr/share/wordlistsnmap.lst -f 10.10.10.1 ftp -V
'''

'''
medusa -h 10.10.10.1 -u $user -P passwords.txt -M ftp
'''

Vulnerable FTP Versions:
'''
• ProFTPD-1.3.3c Backdoor
• ProFTPD 1.3.5 Mod_Copy Command Execution
• VSFTPD v2.3.4 Backdoor Command Execution
'''

**Searchsploit**

**Hydra**

**Port 445 -SMB Enumeration**

Scan for NETBIOS/SMB Service with Nmap:
'''
nmap -p 139,445 --open -oG smb.txt 192.168.1.0/24
'''

Scan for NETBIOS/SMB Service withnbtscan:
'''
nbtscan -r 192.168.1.0/24
'''

Enumerate the Hostname:
'''
nmblookup -A 10.10.10.1
'''

Check for Null Sessions:
'''
smbmap -H {IP Address}
'''
'''
rpcclient -U "" -N {IP Address}
'''
'''
smbclient \\\\{IP Address}\\ShareName
'''

List Shares:
'''
smbmap -H {IP Address}
'''
'''
echo exit | smbclient -L \\\\{IP Address}
'''
'''
nmap --script smb-enum-shares -p 139,445 {IP Address}
'''

Check for SMB Vulneravilities with Nmap:
'''
nmap --script smb-vuln* -p 139,445 {IP Address}
'''

**Ports 80, 443, 8080: -Web Application Enumeration/Exploitation**

Web Application Enumeration Checklist:
'''
1. Checkout the entire webpage and what it is displaying.
2. Read every page, look for emails, names, user info, etc.
3. Directory Discovery (time to dir bust!)
4. Enumerate the interface, what is the CMS & Version? Server installation page?
5. Check for potential Local File Inclusion, Remote File Inclusion, SQL Injection, XXE, and Upload vulnerabilities
6. Check for a default server page, identify the server version
7. View Source Code:
      a. Check for hidden values
      b. Check for comments/developer remarks
      c. Check for Extraneous Code
      d. Check for passwords
8. Check for robots.txt file
9. Web Scanning
'''

Directory Discovery/Dir Busting:

Gobuster

'''
gobuster dir -u 10.10.10.181 -w /usr/share/seclists/Discovery/Web-Content/common.txt
'''
'''
wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/Web-Content/Top1000-RobotsDisallowed.txt; gobuster -u http://10.10.10.10. -w Top1000-RobotsDisallowed.txt
'''
wfuzz search with files:
'''wfuzz -c -z file,/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt --sc 200 http://10.10.10.10/FUZZ
'''
