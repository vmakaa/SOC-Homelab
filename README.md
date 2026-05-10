# SOC-Homelab
This is the full documentation for my SOC homelab that I presented on

# Tech Stack
I decided on a fully cloud-based approach using DigitalOcean.

I had one virtual machine running the SANS ISC D-Shield Honeypot, which included honeypot services for ssh, telnet, and also had a web instance running to capture web attacks.

In a separate VM, I installed Suricata with all the latest rules and also installed a lightweight eve.json visualization program called Evebox. On the same machine, I also deployed a docker container of TheHive, an open source Incident Management System.

My vision was to have Suricata monitor inbound and outbound traffic, and when Suricata would alert on malicious traffic the data of thta alert could be viewed in the nice GUI provided by Evebox. For Incident Management, I had Claude write me a python script that would periodically fetch eve.json and import them to TheHive as alerts so that an alert may be escalated to a case.

# Technical Details
In order for Suricata to have visbility on ingress and egress traffic coming from the honeypot, I created a two gre tunnels, one from the suricata machine to the honeypot (Tunnel A) and vice versa (Tunnel B). I then used iptables to send a copy of all inbound and outbound traffic thrught the IP address of tunnel B.

iptables -t mangle -A PREROUTING -j TEE --gateway IP of Tunnel B
iptables -t mangle -A POSTROUTING -j TEE --IP of tunnel B
