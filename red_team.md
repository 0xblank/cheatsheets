# Red team

## C2

### Cobalt strike

Run the team server

`sudo ./teamserver <server_IP> <password> <malleable_profile_file>`

Run cobalt strike client

Create a listener
- Egress listener
    - HTTP
    - HTTPS
    - DNS

## DNS

You may want to use [PowerDNS](https://github.com/PowerDNS/pdns) to register a domain name. PowerDNS might also be usefull for dns exfiltration.

> To get a web interface for PowerDNS you can use [PowerDNS-Admin](https://github.com/PowerDNS-Admin/PowerDNS-Admin)