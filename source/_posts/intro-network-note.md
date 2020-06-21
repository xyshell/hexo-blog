---
title: Intro Network Note
date: 2020-06-20 13:45:44
tags: network
toc: true
---

# OSI Model

![OSI_model](/photos/intro-network-note/OSI_model.png)

# Protocols

## Data Transfer

![http](/photos/intro-network-note/http.png)

## File Transfer

![ftp](/photos/intro-network-note/ftp.png)

## Email

![email](/photos/intro-network-note/email.png)

## Authentication

![auth](/photos/intro-network-note/auth.png)

## Network Service

![dhcp](/photos/intro-network-note/dhcp.png)

## Domain Name System(DNS)

![dns](/photos/intro-network-note/dns.png)

```bash
$ nslookup google.com # check google's ip
$ nslookup 
> facebook.com
```

```bash
$ nslookup
> server 8.8.8.8 # config dns server
```

## Network Time Protocol(NTP)

![ntp](/photos/intro-network-note/ntp.png)

## Network Management

![ssh_telnet](/photos/intro-network-note/ssh_telnet.png)

ssh: encryted; telnet: clear text

ssh used encrypt ftp

![snmp](/photos/intro-network-note/snmp.png)

Walk the tree: server collect information(statistics, log) from client

Trap: client send SNMP trap to server

## Remote Desktop Protocol(RDP)

![rdp](/photos/intro-network-note/rdp.png)

## Audio/Visual Protocol

![h323](/photos/intro-network-note/h323.png)

![sip](/photos/intro-network-note/sip.png)

session initiation protocol: voice over ip communication

# TCP and UDP 

TCP: transmission control protocol

UDP: user datagram protocol

## TCP

reliable, verifiable(sequence numbers / acknowledge numbers), notion of session

### The 3-way handshake

![3way](/photos/intro-network-note/3way.png)

1. SYN: send syn msg, wait for reply from server(change state to SYN-RECEIVED)
2. SYN-ACK: send msg to client
3. ACK: client respond to server

then session establish between client and server by layer 4 protocol

client or server can ask for missing / additional information from each other

then use layer 7 protocol

### The 4-way Disconnect

![4way](/photos/intro-network-note/4way.png)

1. FIN: server to client
2. FIN-ACK: client to server
3. FIN: client to server
4. FIN-ACK

shutdown the session

RST: tcp reset, server to client, to shutdown quickly

## UDP

no 3-way handshake, no reliable communication, no sequence numbers / acknowledge numbers

very efficient for small data transfer (e.x. DNS)

![udp](/photos/intro-network-note/udp.png)

## Port numbers(Transport layer addressing)

![port](/photos/intro-network-note/port.png)

### Source port and Destination port

![src_dstnt](/photos/intro-network-note/src_dstnt.png)

## Application layer portocol dependency

![protocol_dependency_1](/photos/intro-network-note/protocol_dependency_1.png)

![protocol_dependency_2](/photos/intro-network-note/protocol_dependency_2.png)

# IP Addressing

- unicast: class A, B, C(public internet), one device to one device

- multicast: class D(enterprise org's live video streamming), one device to many devices

- experimental: class E

## class A

![class_a](/photos/intro-network-note/class_a.png)

## class B

![class_b](/photos/intro-network-note/class_b.png)

## class C

![class_c](/photos/intro-network-note/class_c.png)

## class D

![class_d](/photos/intro-network-note/class_d.png)

## Address types

![address_types](/photos/intro-network-note/address_types.png)

## Private ip address

127.0.0.1: loopback address, localhost

![private_ip](/photos/intro-network-note/private_ip.png)

