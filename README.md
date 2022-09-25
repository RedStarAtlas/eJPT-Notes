# eJPT-Notes
This is a list of notes and commands to use for the eJPT.


**Ping Sweeps:**

**fping:**

- The ***-a*** option forces the tool to show only alive hosts.
- THe ***-g*** option tells the tool that we want to perform a ping sweep instead of a standard ping.

'''
fping -a -g {IP Range} 2>/dev/null
'''

**fping example:**
fping -a -g 192.168.51.0/24 2>/dev/null

**Nmap Ping Sweep**


**Enumerate Hosts Found on the Network**

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

**OS Fingerprinting**
During a penetration test, you will have to perform this step on every network node, i.e.:
- Routers
- Firewalls
- Hosts
- Servers
- Printers

**OS Fingerprinting with Nmap**
- ***-O*** Enables OS detection.
- ***-Pn*** use to skip the ping scan if you already know that the targets are alive.

'''
nmap -Pn -O {IP Addresses/target(s)}
'''

You can fine-tune the OS fingerprinting process by using the following options:
'''
OS Detection:
    --osscan-limit: Limit OS detection to promising targets
    --osscan-guess: Guess the OS more aggressively
'''

**Find Common Vulnerabilities**

**Common Ports with Vulnerabilities**

Port	Protocol
21	    FTP
22	    SSH
23	    TELNET
25	    SMTP
53	    DNS
80	    HTTP
443	    HTTPS
110	    POP3
115	    SFTP
143	    IMAP
135	    MSRPC
137	    NETBIOS
138	    NETBIOS
139	    NETBIOS
445	    SMB
3306	MYSQL
1433	MYSQL
3389	RDP

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