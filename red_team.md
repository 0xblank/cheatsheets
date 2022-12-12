# Red team

## C2

## Staged vs stageless handler

[Great article about the subject](https://buffered.io/posts/staged-vs-stageless-handlers/)

Staged :
- Minimal size pyaload
- Dumb stager
- Easily detected by an IDS

Stageless:
- Larger payload
- Fully encrypted communication
- Less likely to get caught by an IDS

### Cobalt strike

Run the team server

`sudo ./teamserver <server_IP> <password> <malleable_profile_file>`

Run cobalt strike client

Create a listener:
- Egress listener: egress listener is one that allows Beacon to communicate outside of the target network to our team server. Beacon will encapsulate C2 traffic over HTTP/S or DNS.
- Peer-to-peer: P2P listeners differ from egress listeners because they don't communicate with the team server directly.  Instead, P2P listeners are designed to chain multiple Beacons together in parent/child relationships. P2P listeners can either use SMB or raw TCP.

Create a payload:
- HTML application: a .hta file uses embedded VBScript to run the payload.  Only generates payloads for egress listeners and is limited to x86.
- MS Office Macro: a piece of VBA that can be dropped into a macro-enabled MS Word or Excel document.  Only generates payloads for egress listeners but is compatible with both x86 and x64 Office.
- Stager Payload Generator: a payload stager in a variety of languages including C, C#, PowerShell, Python, and VBA. Only generates payloads for egress listeners, but supports x86 and x64.
- Stageless Payload Generator: As above, but generates stageless payloads rather than stagers. No PowerShell, but has the added option of specifying an exit function (process or thread).  It can also generate payloads for P2P listeners.
- Windows Stager Payload: a pre-compiled stager as an EXE, Service EXE or DLL
- Windows Stageless Payload: a pre-compiled stageless payload as an EXE, Service EXE, DLL, shellcode, as well as PowerShell.  This is also the only means of generating payloads for P2P listeners.
- Windows Stageless Generate All Payloads: every stageless payload variant, for every listener, in x86 and x64.

## DNS

You may want to use [PowerDNS](https://github.com/PowerDNS/pdns) to register a domain name. PowerDNS might also be usefull for dns exfiltration.

> To get a web interface for PowerDNS you can use [PowerDNS-Admin](https://github.com/PowerDNS-Admin/PowerDNS-Admin)