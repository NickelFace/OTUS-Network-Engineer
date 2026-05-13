[🇷🇺 Русская версия](README.ru.md)

## Lab 01 — Configuring Extended VLAN, VTP and DTP

#### Topology

![Topology](img/VLAN.png)

#### Addressing table

| Device | Interface | IP address   | Subnet Mask   |
| ------ | --------- | ------------ | ------------- |
| S1     | VLAN 99   | 192.168.99.1 | 255.255.255.0 |
| S2     | VLAN 99   | 192.168.99.2 | 255.255.255.0 |
| S3     | VLAN 99   | 192.168.99.3 | 255.255.255.0 |
| PC-A   | NIC       | 192.168.10.1 | 255.255.255.0 |
| PC-B   | NIC       | 192.168.20.1 | 255.255.255.0 |
| PC-C   | NIC       | 192.168.10.2 | 255.255.255.0 |

#### Goals

**Goal:** Setting up VTP, Setting up DTP, Adding VLANs and Assigning Ports, Setting up an Extended VLAN

All switches use VTP for VLAN updates. S2 is configured as VTP server, S1 and S3 as clients.  
Part 4 switches S1 to VTP transparent mode and creates an extended-range VLAN.

#### Part 1 — Basic device setup

<details>
<summary>S1</summary>
<pre><code>
enable
configure terminal
interface vlan 1
ip address 192.168.1.1 255.255.255.0
no shutdown
exit
hostname S1
do copy run start
</code></pre>
</details>
<details>
<summary>S2</summary>
<pre><code>
enable
configure terminal
interface vlan 1
ip address 192.168.1.2 255.255.255.0
no shutdown
exit
hostname S2
do copy run start
</code></pre>
</details>
<details>
<summary>S3</summary>
<pre><code>
enable
configure terminal
interface vlan 1
ip address 192.168.1.3 255.255.255.0
no shutdown
exit
hostname S3
do copy run start
</code></pre>
</details>

Disable DNS lookup, set passwords, enable logging synchronous:

<details>
<summary>S1, S2, S3</summary>
<pre><code>
no ip domain-lookup
enable secret cisco
line console 0
password cisco
login
logging synchronous
banner motd "** This is a secure system. Authorized Access Only!"
</code></pre>
</details>

#### Part 2 — VTP configuration

<details>
<summary>S2 (VTP Server)</summary>
<pre><code>
vtp domain CCNA
vtp password cisco
vtp version 3
vtp mode server
end
vtp primary server force
</code></pre>
</details>
<details>
<summary>S1, S3 (VTP Client)</summary>
<pre><code>
vtp domain CCNA
vtp password cisco
vtp version 3
vtp mode client
</code></pre>
</details>

#### Part 3 — DTP / Trunk setup

<details>
<summary>S1, S2, S3</summary>
<pre><code>
interface Ethernet0/3
 switchport trunk encapsulation dot1q
 switchport mode trunk
interface Ethernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
</code></pre>
</details>

Verify: `show interfaces trunk`

#### Part 4 — Create VLANs on S2 (server)

<details>
<summary>S2</summary>
<pre><code>
vlan 999
 name VTP_Lab
vlan 10
 name Red
vlan 20
 name Blue
vlan 30
 name Yellow
vlan 99
 name Management
</code></pre>
</details>

Verify propagation: `show vlan brief` on S1 and S3.

#### Part 5 — Extended VLAN on S1 (transparent mode)

<details>
<summary>S1</summary>
<pre><code>
vtp mode transparent
vlan 2000
 name Extended_VLAN
</code></pre>
</details>

---

← *Start* | [🏠 Home](../README.md) | [02. STP →](../STP/)
