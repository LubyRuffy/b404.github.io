---
title: 'reading-notes'
layout: post
tags: 
  - readings
  - sec
  - information
category: 
  - reading-notes
  - sec
comments: true
share: true
---
reading-notes

* TOC
{:toc}

<!--more-->

<p align="left">1.在www.iana.org上有一份由国际因特网地址分配委员会维护的官方已分配的端口列表。在linux和unix系统的/etc/services中还可以找到此列表:</p>

```Shell
➜ /home/b404 >cd /etc/
➜ /etc >cat services 
# Network services, Internet style
#
# Note that it is presently the policy of IANA to assign a single well-known
# port number for both TCP and UDP; hence, officially ports have two entries
# even if the protocol doesn't support UDP operations.
#
# Updated from http://www.iana.org/assignments/port-numbers and other
# sources like http://www.freebsd.org/cgi/cvsweb.cgi/src/etc/services .
# New ports will be added on request if they have been officially assigned
# by IANA and used in the real-world or are needed by a debian package.
# If you need a huge list of used numbers please install the nmap package.

tcpmux      1/tcp               # TCP port service multiplexer
echo        7/tcp
echo        7/udp
discard     9/tcp       sink null
discard     9/udp       sink null
systat      11/tcp      users
daytime     13/tcp
daytime     13/udp
netstat     15/tcp
qotd        17/tcp      quote
msp     18/tcp              # message send protocol
msp     18/udp
chargen     19/tcp      ttytst source
chargen     19/udp      ttytst source
ftp-data    20/tcp
ftp     21/tcp
fsp     21/udp      fspd
ssh     22/tcp              # SSH Remote Login Protocol
ssh     22/udp
telnet      23/tcp
smtp        25/tcp      mail
time        37/tcp      timserver
time        37/udp      timserver
rlp     39/udp      resource    # resource location
nameserver  42/tcp      name        # IEN 116
whois       43/tcp      nicname
tacacs      49/tcp              # Login Host Protocol (TACACS)
tacacs      49/udp
re-mail-ck  50/tcp              # Remote Mail Checking Protocol
re-mail-ck  50/udp
domain      53/tcp              # Domain Name Server
domain      53/udp
mtp     57/tcp              # deprecated
tacacs-ds   65/tcp              # TACACS-Database Service
tacacs-ds   65/udp
bootps      67/tcp              # BOOTP server
bootps      67/udp
bootpc      68/tcp              # BOOTP client
bootpc      68/udp
tftp        69/udp
gopher      70/tcp              # Internet Gopher
gopher      70/udp
rje     77/tcp      netrjs
finger      79/tcp
http        80/tcp      www     # WorldWideWeb HTTP
http        80/udp              # HyperText Transfer Protocol
link        87/tcp      ttylink
kerberos    88/tcp      kerberos5 krb5 kerberos-sec # Kerberos v5
kerberos    88/udp      kerberos5 krb5 kerberos-sec # Kerberos v5
supdup      95/tcp
hostnames   101/tcp     hostname    # usually from sri-nic
iso-tsap    102/tcp     tsap        # part of ISODE
acr-nema    104/tcp     dicom       # Digital Imag. & Comm. 300
acr-nema    104/udp     dicom
csnet-ns    105/tcp     cso-ns      # also used by CSO name server
csnet-ns    105/udp     cso-ns
rtelnet     107/tcp             # Remote Telnet
rtelnet     107/udp
pop2        109/tcp     postoffice pop-2 # POP version 2
pop2        109/udp     pop-2
pop3        110/tcp     pop-3       # POP version 3
pop3        110/udp     pop-3
sunrpc      111/tcp     portmapper  # RPC 4.0 portmapper
sunrpc      111/udp     portmapper
auth        113/tcp     authentication tap ident
sftp        115/tcp
uucp-path   117/tcp
nntp        119/tcp     readnews untp   # USENET News Transfer Protocol
ntp     123/tcp
ntp     123/udp             # Network Time Protocol
pwdgen      129/tcp             # PWDGEN service
pwdgen      129/udp
loc-srv     135/tcp     epmap       # Location Service
loc-srv     135/udp     epmap
netbios-ns  137/tcp             # NETBIOS Name Service
netbios-ns  137/udp
netbios-dgm 138/tcp             # NETBIOS Datagram Service
netbios-dgm 138/udp
netbios-ssn 139/tcp             # NETBIOS session service
netbios-ssn 139/udp
imap2       143/tcp     imap        # Interim Mail Access P 2 and 4
imap2       143/udp     imap
snmp        161/tcp             # Simple Net Mgmt Protocol
snmp        161/udp
snmp-trap   162/tcp     snmptrap    # Traps for SNMP
snmp-trap   162/udp     snmptrap
cmip-man    163/tcp             # ISO mgmt over IP (CMOT)
cmip-man    163/udp
cmip-agent  164/tcp
cmip-agent  164/udp
mailq       174/tcp         # Mailer transport queue for Zmailer
mailq       174/udp
xdmcp       177/tcp             # X Display Mgr. Control Proto
xdmcp       177/udp
nextstep    178/tcp     NeXTStep NextStep   # NeXTStep window
nextstep    178/udp     NeXTStep NextStep   #  server
bgp     179/tcp             # Border Gateway Protocol
bgp     179/udp
....
```
