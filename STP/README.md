[🇷🇺 Русская версия](README.ru.md)

## Lab 02 — Switched Network with Redundant Links (STP)

### Topology

![Scheme](img/Scheme.png)

### Addressing table

| Device | Interface | IP address  | Subnet Mask   |
| ------ | --------- | ----------- | ------------- |
| S1     | VLAN 1    | 192.168.1.1 | 255.255.255.0 |
| S2     | VLAN 1    | 192.168.1.2 | 255.255.255.0 |
| S3     | VLAN 1    | 192.168.1.3 | 255.255.255.0 |

### Goals

- **Part 1.** Build the network and configure basic device parameters
- **Part 2.** Select the Root Bridge
- **Part 3.** Observe STP port selection based on port cost
- **Part 4.** Observe STP port selection based on port priority

### Part 1 — Basic setup

<details>
<summary>S1</summary>
<pre><code>
enable
conf t
hostname S1
line con 0
exec-timeout 0 0
exit
interface vlan 1
ip address 192.168.1.1 255.255.255.0
no shutdown
</code></pre>
</details>
<details>
<summary>S2</summary>
<pre><code>
enable
conf t
hostname S2
line con 0
exec-timeout 0 0
exit
interface vlan 1
ip address 192.168.1.2 255.255.255.0
no shutdown
</code></pre>
</details>
<details>
<summary>S3</summary>
<pre><code>
enable
conf t
hostname S3
line con 0
exec-timeout 0 0
exit
interface vlan 1
ip address 192.168.1.3 255.255.255.0
no shutdown
</code></pre>
</details>

<details>
<summary>S1, S2, S3 — passwords and banner</summary>
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

### Part 2 — Root Bridge selection

Set all ports to trunk mode, then disable e0/1 and e0/3:

<details>
<summary>S1, S2, S3</summary>
<pre><code>
conf t
interface range e0/0-3
 switchport trunk encapsulation dot1q
 switchport mode trunk
interface e0/1
 shutdown
interface e0/3
 shutdown
</code></pre>
</details>

Verify STP: `show spanning-tree`

![STP status](img/stp_status.png)

Root is elected by lowest MAC address (BID priority default 32769). Port e0/0 on S3 is blocked.

### Part 3 — Port cost change

Lower cost on S3 root port to observe topology change:

<details>
<summary>S3</summary>
<pre><code>
interface e0/2
 spanning-tree cost 90
</code></pre>
</details>

![Changed cost](img/44res_cost.png)

Restore:
<details>
<summary>S3</summary>
<pre><code>
interface e0/2
 no spanning-tree cost 90
</code></pre>
</details>

### Part 4 — Port priority

Re-enable e0/1 and e0/3 on all switches, then observe port selection:

<details>
<summary>S1, S2, S3</summary>
<pre><code>
interface e0/1
 no shutdown
interface e0/3
 no shutdown
</code></pre>
</details>

![Port status after](img/port_status_after.png)

STP tie-breaking order: **path cost → BID → port priority → port number**.

---

[← 01. VLAN](../VLAN/) | [🏠 Home](../README.md) | [03. EtherChannel →](../EtherChannel/)
