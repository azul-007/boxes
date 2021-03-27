# Kioptrix Level 1
## Discovery
- As always we start are search with netdiscover
- <img width="637" alt="netdiscover" src="https://user-images.githubusercontent.com/15880042/112721521-98a69200-8eda-11eb-831b-6065769aa8b0.png">
- The IP of the machine is 192.168.1.104

## Information Gathering

### nmap
* We run the following nmap scan: nmap -A -Pn 192.168.1.104
  * -A is an aggressive scan, that enables script scanning (-sC), version detection (-sV), OS detection (-O) and traceroute (-traceroute)
  * -Pn disables ping discovery

![nmap](https://user-images.githubusercontent.com/15880042/112721896-c987c680-8edc-11eb-850e-36058baedd07.png)

Nmap has returned 6 ports:
* 22/TCP - default SSH port. Potential for SSH keys to be left on this box.
* 80/TCP - default port for HTTP web browsing. Hmm there may be a CMS here.
* 111/TCP - provides remote procedure call port mapping
* 139/TCP - netbios/samba, file sharing port. *Use smbmap, nbtlookup, nbtstat*
* 443/TCP - standard HTTPS web browsing port
* 1024/TCP - 
* The scrips results returned the NetBIOS name of the box: KIOPTRIX.

## Enumeration

Now that we have all listed all the ports, let's begin enumerating them. We'll start by simply going to the browser on port 80

### http

Browsed to the web page and we are presented with a test page. After inspecting the html, doesn't look like there's any leads present. Let's 
check out that samba port, 139

![test_page](https://user-images.githubusercontent.com/15880042/112722773-c3e0af80-8ee1-11eb-97bc-76a2bf1c7d4a.png)


### samba

Earlier we found the hostname of the machine: KIOPTRIX. An nmblookup confirms this

![nmblookup](https://user-images.githubusercontent.com/15880042/112723018-e8895700-8ee2-11eb-8058-5daad5c279a4.png)

Using nbtscan, we found the username, KIOPTRIX

<img width="1280" alt="username" src="https://user-images.githubusercontent.com/15880042/112726988-92261380-8ef6-11eb-913a-de853464d1c2.png">

Now, although we don't have a password, let's see if we can attempt to connect via SSH with the KIOPTRIX username. We received an error message:
*Unable to negotiate with 192.168.1.104 port 22: no matching key exchange method found. Their offer: diffie-hellman-group-exchange-sha1,diffie-hellman-group1-sha1*

After some googling, I came across this article: https://unix.stackexchange.com/questions/402746/ssh-unable-to-negotiate-no-matching-key-exchange-method-found

This happens because "your system and the remote system don't share at least one cipher, there is no cipher to agree on and no encrypted channel is possible". You must specify which cipher to use.

Add -oKexAlgorithms=+diffie-hellman-group1-sha1 after ssh and before the username@server

![ssh](https://user-images.githubusercontent.com/15880042/112734300-4cc80d00-8f1b-11eb-83ce-cabbd68771f2.png)

