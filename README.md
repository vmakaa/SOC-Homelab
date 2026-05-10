# SOC-Homelab
This is the full documentation for my SOC homelab that I presented on

# Tech Stack
I decided on a fully cloud-based approach using DigitalOcean.

I had one virtual machine running the [SANS ISC D-Shield Honeypot](https://isc.sans.edu/honeypot.html), which included honeypot services for ssh, telnet, and also had a web instance running to capture web attacks.

In a separate VM, I installed Suricata with all the latest rules and also installed a lightweight eve.json visualization program called Evebox. On the same machine, I also deployed a docker container of [TheHive](https://docs.strangebee.com/thehive/installation/docker/), an open source Incident Management System.

My vision was to have Suricata monitor inbound and outbound traffic, and when Suricata would alert on malicious traffic the data of thta alert could be viewed in the nice GUI provided by Evebox. For Incident Management, I had Claude write me a python script that would periodically fetch eve.json and import them to TheHive as alerts so that an alert may be escalated to a case.

# Technical Details
In order for Suricata to have visbility on ingress and egress traffic coming from the honeypot, I created a two gre tunnels, one from the suricata machine to the honeypot (Tunnel A) and vice versa (Tunnel B). I then used iptables to send a copy of all inbound and outbound traffic thrught the IP address of tunnel B.


The following two commands are needed on the honeypot to forward all of its inbound and outbound traffic:

iptables -t mangle -A PREROUTING -j TEE --gateway IP of Tunnel B

iptables -t mangle -A POSTROUTING -j TEE --gateway IP of tunnel B


The following commands are how to set up a GRE link between two machines, this is essential so that the honeypot has an interface to send its copied traffic to:

Machine1 (M1):

ip tunnel add gre1 mode gre remote <M2 IP> local <M1 IP> ttl 255
ip link set gre1 up
ip addr add 172.16.0.1/30 dev gre1

Machine2 (M2):

ip tunnel add gre1 mode gre remote <M1 IP> local <M2 IP> ttl 255
ip link set gre1 up
ip addr add 172.16.0.2/30 dev gre1

# Alert Analysis
With everything now setup

