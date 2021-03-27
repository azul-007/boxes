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


