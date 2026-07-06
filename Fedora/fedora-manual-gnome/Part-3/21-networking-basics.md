# Chapter 21: Networking Basics — How Your Fedora System Connects

## The Invisible Foundation

Networking is the invisible foundation of everything you do on a computer. Every web page, every email, every software update, every streaming video — all of it travels through network connections that just... work. Until they don't.

When your network breaks on Windows, you right-click the network icon and run the troubleshooter, which restarts the adapter and hopes for the best. When your network breaks on Linux, you have tools to diagnose exactly what's wrong and fix it precisely.

This chapter teaches you how Fedora 44 handles networking: how it connects to Wi-Fi and Ethernet, how it gets an IP address, how DNS translates names to numbers, how the firewall controls traffic, and how to troubleshoot when things go wrong.

You'll learn tools that work on every Linux system — from your Fedora laptop to enterprise servers. These skills transfer universally.

---

## Table of Contents

| Topic | Section |
|---|---|
| Networking Concepts in Plain Language | 1 |
| NetworkManager: Fedora's Network Brain | 2 |
| The ip Command Family | 3 |
| DNS and Name Resolution | 4 |
| Testing Connectivity (ping, traceroute) | 5 |
| ss: Socket Statistics | 6 |
| curl and wget: Fetching Data | 7 |
| SSH: Secure Remote Access | 8 |
| firewalld: Controlling Traffic | 9 |
| Network Troubleshooting Methodology | 10 |

---

## 1. Networking Concepts in Plain Language

### The Postal Service Analogy

Think of networking as a postal system:

| Networking Concept | Postal Equivalent | Purpose |
|---|---|---|
| **IP address** | Street address | Identifies your machine on the network |
| **Subnet mask** | Neighborhood boundary | Defines which addresses are "local" vs. "remote" |
| **Gateway/router** | Post office | Forwards mail to addresses outside your neighborhood |
| **DNS** | Phone book | Translates names (google.com) to addresses (142.250.80.46) |
| **Port** | Apartment number | Specifies which service receives the data |
| **MAC address** | Resident's name | Hardware-level identifier burned into your NIC |

Every device on a network has an IP address. When you type `firefox` and navigate to `https://example.com`, here's what happens:

    You type: example.com
    DNS resolves: example.com → 93.184.216.34
    Your computer sends: "Give me your homepage" → Packet goes from your IP (192.168.1.50) → Through your gateway (192.168.1.1) → Through your ISP's routers → Arrives at 93.184.216.34 port 443
    Server responds: "Here's the HTML"
    Firefox renders it

IPv4 vs. IPv6

Two addressing systems coexist:
System	Format	Example	Address Space
IPv4	Four numbers (0–255)	192.168.1.50	~4.3 billion
IPv6	Eight hex groups	2001:db8::1	~340 undecillion

IPv4 is what most home networks still use. IPv6 exists because we're running out of IPv4 addresses — the internet simply has more devices than available IPv4 numbers. Fedora 44 supports both simultaneously (called dual-stack).

You'll mostly work with IPv4 in this chapter. IPv6 configuration follows the same principles with different syntax.
Private vs. Public Addresses

Not all IP addresses are reachable from the internet:
Range	Scope	Example
10.0.0.0/8	Private (large networks)	10.0.1.50
172.16.0.0/12	Private (medium networks)	172.16.1.50
192.168.0.0/16	Private (home/SOHO)	192.168.1.50
Everything else	Public (internet-routable)	142.250.80.46

Your home router performs NAT (Network Address Translation): it has one public IP from your ISP, and translates traffic for all your private devices. Your Fedora machine has a private IP (like 192.168.1.50); the router translates it to the public IP when communicating with the internet.
Subnet Masks and CIDR

The subnet mask defines which part of an IP address identifies the network and which part identifies the host:

IP:      192.168.1.50
Mask:    255.255.255.0
         ↑↑↑↑↑↑↑↑↑↑↑↑ ↑
         network      host

This means 192.168.1.xxx is your local network (255 possible hosts). Any address starting with 192.168.1 is "local" — your computer can reach it directly. Anything else must go through the gateway.

CIDR notation compresses this: 192.168.1.50/24 means the first 24 bits are the network (equivalent to 255.255.255.0). You'll see CIDR everywhere in Linux networking.

Common CIDR values:
CIDR	Mask	Hosts	Use Case
/8	255.0.0.0	16 million	Large enterprise
/16	255.255.0.0	65,534	Medium organization
/24	255.255.255.0	254	Home/office network
/30	255.255.255.252	2	Point-to-point links
Ports: The Apartment Numbers

An IP address gets data to the right machine. A port gets it to the right application:
Port	Protocol	Service
22	TCP	SSH
25	TCP	SMTP (email sending)
53	TCP/UDP	DNS
80	TCP	HTTP (web)
443	TCP	HTTPS (encrypted web)
6881	TCP/UDP	BitTorrent
3306	TCP	MySQL/MariaDB
5432	TCP	PostgreSQL
6379	TCP	Redis

Ports 1–1023 are privileged (require root to bind). Ports 1024–65535 are unprivileged (any user can use them).

    💡 Socket = IP + Port

    A socket is the combination of an IP address and a port number: 192.168.1.50:22 is the SSH socket on your machine. Every network connection is between two sockets: your IP:port ↔ remote IP:port.

2. NetworkManager: Fedora's Network Brain
What NetworkManager Does

NetworkManager is Fedora's network configuration daemon. It manages:

    Wired (Ethernet) connections
    Wireless (Wi-Fi) connections
    VPN connections
    Bonding and teaming
    Bridge interfaces
    DNS resolution integration

NetworkManager runs as a systemd service:

systemctl status NetworkManager
# Active: active (running)

It provides three interfaces:

    GUI — GNOME Settings → Network panel
    nmcli — full command-line interface
    nmtui — text-based (curses) menu interface

nmtui: The Visual CLI

For beginners, nmtui (NetworkManager Text User Interface) provides a menu-driven approach:

nmtui

┌─────────────────────────────────────────────┐
│              NetworkManager                 │
│                                             │
│ ┌─────────────────────────────────────────┐ │
│ │ Edit a connection                      │ │
│ │ Activate a connection                  │ │
│ │ Set system hostname                    │ │
│ │ Quit                                   │ │
│ └─────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘

Use arrow keys to navigate, Enter to select, Tab to move between fields. No memorization required — everything is menu-driven.
nmcli: The Power Tool

nmcli (NetworkManager Command Line Interface) is the primary CLI for managing networking. It's scriptable, comprehensive, and works over SSH.
Checking Status

# Overall NetworkManager status:
nmcli general status
# STATE      CONNECTIVITY  WIFI-HW  WIFI     WWAN-HW  WWAN
# connected  full          enabled  enabled  enabled  enabled

# List network devices (interfaces):
nmcli device status
# DEVICE  TYPE      STATE      CONNECTION
# enp3s0  ethernet  connected  Wired connection 1
# wlp2s0  wifi      connected  HomeNetwork
# lo      loopback  unmanaged  --

# Show all saved connection profiles:
nmcli connection show
# NAME                 UUID                                  TYPE      DEVICE
# Wired connection 1   a1b2c3d4-...                          ethernet  enp3s0
# HomeNetwork           e5f6g7h8-...                          wifi      wlp2s0
# VPN-Work             i9j0k1l2-...                            vpn       --

Device Naming Convention

Fedora uses predictable interface names based on hardware topology:
Prefix	Meaning	Example
en	Ethernet (wired)	enp3s0, eno1, enx001122334455
wl	Wireless (Wi-Fi)	wlp2s0, wlo1
ww	Wireless WAN (cellular)	wwp0s20u4
lo	Loopback (localhost)	lo
br	Bridge	br0
virbr	Virtual bridge (libvirt)	virbr0
tun	Tunnel	tun0
wg	WireGuard	wg0

The letters after the prefix encode physical location: p3s0 means PCI bus 3, slot 0. This replaces the old eth0, eth1 naming that could change between boots.
Connecting to Wi-Fi

# List available Wi-Fi networks:
nmcli device wifi list
# SSID               SECURITY  SIGNAL  BARS  CHANNEL
# HomeNetwork        WPA2      95      ▂▄▆█  6
# NeighborWifi       WPA2      40      ▂▄__  11
# CoffeeShop_Open    --        25      ▂___  1

# Connect to a Wi-Fi network:
nmcli device wifi connect "HomeNetwork" password "your-password"

# Connect to a hidden network (not broadcasting SSID):
nmcli device wifi connect "HiddenNetwork" password "your-password" hidden yes

Managing Connections

# Disconnect an interface:
nmcli device disconnect enp3s0

# Reconnect:
nmcli device connect enp3s0

# Bring a connection up/down:
nmcli connection up "Wired connection 1"
nmcli connection down "Wired connection 1"

# Delete a saved connection:
nmcli connection delete "OldWiFi"

# Rename a connection:
nmcli connection modify "Wired connection 1" connection.id "Home Ethernet"

Configuring Static IP

By default, Fedora uses DHCP (your router assigns an IP automatically). For servers or devices needing a fixed address:

# Set static IP, gateway, and DNS:
nmcli connection modify "Home Ethernet" \
  ipv4.addresses 192.168.1.100/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns "8.8.8.8 1.1.1.1" \
  ipv4.method manual

# Apply the change:
nmcli connection up "Home Ethernet"

# Verify:
ip address show enp3s0

To revert to DHCP:

nmcli connection modify "Home Ethernet" ipv4.method auto
nmcli connection up "Home Ethernet"

Viewing Detailed Connection Info

# Full details of a connection:
nmcli connection show "Home Ethernet"

# Filter for specific fields:
nmcli -f ipv4.addresses,ipv4.gateway,ipv4.dns connection show "Home Ethernet"
# ipv4.addresses: 192.168.1.100/24
# ipv4.gateway:   192.168.1.1
# ipv4.dns:       8.8.8.8,1.1.1.1

Setting Hostname

# View current hostname:
nmcli general hostname
# fedora-workstation

# Change hostname:
sudo nmcli general hostname my-machine
# Takes effect immediately; also updates /etc/hostname

    💡 NetworkManager Stores Profiles as Files

    Connection profiles are stored in /etc/NetworkManager/system-connections/:

    sudo ls /etc/NetworkManager/system-connections/
    # "Home Ethernet.nmconnection"
    # "HomeNetwork.nmconnection"

    These are INI-style text files you can inspect (though editing them directly isn't recommended — use nmcli or nmtui instead):

    sudo cat "/etc/NetworkManager/system-connections/Home Ethernet.nmconnection"

3. The ip Command Family

The ip command (from the iproute2 package) is the modern tool for inspecting and configuring network interfaces. It replaced the ancient ifconfig command.
Viewing Interfaces

# Show all interfaces and addresses:
ip address show
# Or shorter:
ip a

# Show a specific interface:
ip addr show enp3s0

Output decoded:

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 00:1a:2b:3c:4d:5e brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.50/24 brd 192.168.1.255 scope global dynamic noprefixroute enp3s0
       valid_lft 86398sec preferred_lft 86398sec
    inet6 fe80::21a:2bff:fe3c:4d5e/64 scope link noprefixroute
       valid_lft forever preferred_lft forever

Breaking down the Ethernet interface:
Field	Meaning
enp3s0	Interface name
<BROADCAST,MULTICAST,UP,LOWER_UP>	Capabilities and state (UP = enabled)
mtu 1500	Maximum Transmission Unit (bytes per packet)
state UP	Interface is active
link/ether 00:1a:2b:3c:4d:5e	MAC address (hardware address)
inet 192.168.1.50/24	IPv4 address with CIDR prefix
brd 192.168.1.255	Broadcast address
dynamic	Address obtained via DHCP
valid_lft 86398sec	DHCP lease time remaining
inet6 fe80::...	IPv6 link-local address
Viewing Routes (Gateways)

Routes tell the kernel where to send packets:

# Show routing table:
ip route show
# default via 192.168.1.1 dev enp3s0 proto dhcp metric 100
# 192.168.1.0/24 dev enp3s0 proto kernel scope link src 192.168.1.50

Decoded:
Route	Meaning
default via 192.168.1.1 dev enp3s0	Send everything not local through the gateway (192.168.1.1) via the enp3s0 interface
192.168.1.0/24 dev enp3s0 ... src 192.168.1.50	The local network — deliver directly, source from your IP

# View only the default route (gateway):
ip route show default
# default via 192.168.1.1 dev enp3s0

# Show route to a specific destination:
ip route get 8.8.8.8
# 8.8.8.8 via 192.168.1.1 dev enp3s0 src 192.168.1.50
#    ↑ destination ↑ gateway      ↑ interface ↑ source IP

Viewing Link/Layer 2 Info

# Show link-layer details (MAC, state, statistics):
ip link show

# Show only interface state:
ip -br link show
# lo               UNKNOWN        00:00:00:00:00:00
# enp3s0           UP             00:1a:2b:3c:4d:5e
# wlp2s0           UP             00:1c:2d:3e:4f:60

The -br (brief) flag creates compact output — extremely useful for quick checks:

ip -br a
# lo               UNKNOWN        127.0.0.1/8 ::1/128
# enp3s0           UP             192.168.1.50/24 fe80::.../64
# wlp2s0           UP             192.168.1.75/24 fe80::.../64

Statistics

# Show per-interface statistics (packets sent/received/errors):
ip -s link show enp3s0

2: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 00:1a:2b:3c:4d:5e brd ff:ff:ff:ff:ff:ff
    RX:  bytes packets errors dropped  overrun mcast
          1.2Gi  8.5M    0       0       0       0
    TX:  bytes packets errors dropped carrier collsns
          250Mi  3.2M    0       0       0       0

Check for errors (dropped packets, collisions) when diagnosing network problems.

    💡 ip vs. ifconfig

    Old tutorials and documentation reference ifconfig. Modern Linux (including Fedora 44) uses ip exclusively. ifconfig isn't installed by default anymore. Mapping:
    Old (ifconfig)	New (ip)
    ifconfig	ip a
    ifconfig eth0 up	ip link set eth0 up
    ifconfig eth0 192.168.1.50	ip addr add 192.168.1.50/24 dev eth0
    netstat -r	ip route
    netstat -tulpn	ss -tulpn
    arp -a	ip neigh

4. DNS and Name Resolution
How DNS Works on Fedora

When you type ping google.com, your system needs to resolve google.com to an IP address. On Fedora 44, DNS is managed by systemd-resolved:

Application: "What's the IP for google.com?"
    ↓
systemd-resolved (caches results, manages DNS)
    ↓
/etc/resolv.conf (points to DNS servers)
    ↓
Forward queries to: 192.168.1.1 (your router)
                     or 8.8.8.8 (Google DNS)
                     or 1.1.1.1 (Cloudflare DNS)
    ↓
DNS server: "google.com is 142.250.80.46"

Inspecting DNS Configuration

# Check systemd-resolved status:
resolvectl status

Global
  Protocols: LLMNR=resolve -mDNS +DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub

Link 2 (enp3s0)
  Current DNS Server: 192.168.1.1
  DNS Servers: 192.168.1.1 8.8.8.8 1.1.1.1
  DNS Domain: lan
  Default Route: yes

# View /etc/resolv.conf (what applications read):
cat /etc/resolv.conf
# Generated by NetworkManager
# nameserver 127.0.0.53   ← systemd-resolved stub resolver
# options edns0 trust-ad
# search lan

    💡 The Stub Resolver Loop

    /etc/resolv.conf points to 127.0.0.53 — your own machine. This is systemd-resolved's stub listener. Applications query 127.0.0.53, which forwards to the real DNS servers (like 192.168.1.1 or 8.8.8.8). This allows caching and per-interface DNS configuration.

    Don't edit /etc/resolv.conf directly — NetworkManager manages it. Use nmcli to configure DNS servers per connection.

Querying DNS

# Resolve a hostname:
resolvectl query google.com
# google.com: 142.250.80.46
# google.com: 2607:f8b0:4004:809::200e
# -- Information acquired via protocol(s): DNS, LLMNR, mDNS in 2.3ms
# -- Data is authenticated: no

# Reverse lookup (IP to name):
resolvectl query 8.8.8.8
# 8.8.8.8: dns.google
# -- Information acquired via protocol(s): DNS in 12.4ms

# Flush DNS cache:
sudo resolvectl flush-caches

Alternative DNS query tools:

# dig (detailed DNS query):
dig google.com
# ;; ANSWER SECTION:
# google.com.    300  IN  A   142.250.80.46

# Just the IP:
dig +short google.com
# 142.250.80.46

# nslookup (older, simpler):
nslookup google.com
# Name:   google.com
# Address: 142.250.80.46

# host (compact):
host google.com
# google.com has address 142.250.80.46
# google.com has IPv6 address 2607:f8b0:4004:809::200e

Changing DNS Servers

# Set DNS for a connection via nmcli:
nmcli connection modify "Home Ethernet" ipv4.dns "1.1.1.1 8.8.8.8"
nmcli connection modify "Home Ethernet" ipv4.ignore-auto-dns yes
nmcli connection up "Home Ethernet"

# Verify:
resolvectl status | grep "DNS Server"

The /etc/hosts File

Before DNS existed, /etc/hosts was THE name resolution method. It's still checked first:

cat /etc/hosts
# 127.0.0.1   localhost localhost.localdomain
# ::1         localhost
# 192.168.1.50  mymachine.lan  mymachine

You can add entries for local machines without DNS:

sudo nano /etc/hosts
# Add: 192.168.1.200  printerserver  printer
# Now: ping printerserver works without DNS

    ⚠️ Resolution Order

    Fedora checks in this order:

        /etc/hosts (local overrides)
        systemd-resolved cache
        Configured DNS servers

    Entries in /etc/hosts ALWAYS override DNS. This is useful for blocking (point ads.example.com to 127.0.0.1) but can cause confusion if stale entries linger.

5. Testing Connectivity (ping, traceroute)
ping: "Are You There?"

ping sends ICMP echo request packets and waits for replies. It's the most basic connectivity test:

# Ping a host continuously (Ctrl+C to stop):
ping google.com
# PING google.com (142.250.80.46) 56(84) bytes of data.
# 64 bytes from 142.250.80.46: icmp_seq=1 ttl=118 time=12.3 ms
# 64 bytes from 142.250.80.46: icmp_seq=2 ttl=118 time=11.8 ms
# 64 bytes from 142.250.80.46: icmp_seq=3 ttl=118 time=12.1 ms
# ^C
# --- google.com ping statistics ---
# 3 packets transmitted, 3 received, 0% packet loss
# round-trip min/avg/max/mdev = 11.8/12.0/12.3/0.2 ms

# Ping specific number of times:
ping -c 4 google.com    # 4 packets then stops

# Ping a specific IP:
ping -c 3 8.8.8.8

# Flood ping (fast, for stress testing — use carefully):
ping -f 192.168.1.1    # Requires root; sends 100+ packets/sec

# Adjust interval (slow ping):
ping -i 5 192.168.1.1  # One packet every 5 seconds

# Set packet size:
ping -s 1000 google.com  # 1000-byte payloads

Reading ping output:
Field	Meaning	Healthy Values
icmp_seq	Packet sequence number	Incrementing
ttl	Time To Live (hops remaining)	>60 for reachable hosts
time	Round-trip time	<50ms local, <100ms regional
packet loss	Percentage of lost packets	0%

    💡 ping Fails — What Does It Mean?

    ping failing doesn't always mean the network is broken:

        Many servers block ICMP (firewall policy) but are reachable via HTTP/SSH
        DNS resolution might fail (try ping 8.8.8.8 instead of ping google.com)
        Your machine might block outgoing ICMP (check firewalld)

    Always test with BOTH a name and an IP to isolate DNS vs. connectivity issues.

traceroute and tracepath

When ping shows high latency or failures, traceroute shows WHERE the problem occurs along the path:

# Install traceroute if not present:
sudo dnf install traceroute

# Trace path to a host:
traceroute google.com

traceroute to google.com (142.250.80.46), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)        0.4 ms   0.3 ms   0.5 ms
 2  10.0.0.1 (10.0.0.1)           5.2 ms   5.1 ms   5.3 ms
 3  isp-router.example.net        12.1 ms  11.9 ms  12.0 ms
 4  core1.example.net             15.3 ms  15.1 ms  15.2 ms
 5  google-peer.example.net       18.7 ms  18.5 ms  18.6 ms
 6  142.250.80.46                 20.1 ms  19.9 ms  20.0 ms

Each line is a hop (router) between you and the destination. The three times are round-trip measurements for three probes.

# tracepath is similar but doesn't require root and discovers MTU:
tracepath google.com

Reading traceroute output:

    A * * * means that hop didn't respond (likely ICMP-blocking firewall — not necessarily a problem)
    High latency jumps between hops indicate where the delay occurs
    Fewer hops = closer/faster connection

mtr: Combined ping + traceroute

mtr (My Traceroute) continuously updates a live view combining ping and traceroute:

sudo dnf install mtr
mtr google.com

                              My traceroute  [v0.95]
Keys:  Help   Display mode   Restart statistics   Order of fields   quit
                                       Packets               Pings
 Host                                Loss%   Snt   Last   Avg  Best  Wrst StDev
 1. _gateway                          0.0%   12    0.4   0.4   0.3   0.5   0.1
 2. 10.0.0.1                          0.0%   12    5.2   5.1   4.9   5.5   0.2
 3. isp-router.example.net            0.0%   12   12.1  12.0  11.8  12.3   0.2
 4. core1.example.net                 0.0%   12   15.3  15.2  15.0  15.6   0.2
 5. google-peer.example.net           0.0%   12   18.7  18.6  18.4  19.0   0.2
 6. 142.250.80.46                     0.0%   12   20.1  20.0  19.8  20.3   0.2

mtr runs continuously (press q to quit), updating statistics in real time. It's the best tool for diagnosing intermittent network problems — leave it running while you reproduce the issue.
6. ss: Socket Statistics
What Is ss?

ss (socket statistics) shows active network connections. It replaced the older netstat command:

# Check if netstat exists:
which netstat
# Usually not installed on Fedora 44

Listening Ports

# Show all listening TCP ports:
ss -tlnp

State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port  Process
LISTEN  0       128     0.0.0.0:22          0.0.0.0:*          users:(("sshd",pid=987,fd=3))
LISTEN  0       5       127.0.0.1:631        0.0.0.0:*          users:(("cupsd",pid=1234,fd=7))
LISTEN  0       128     *:5355              *:*                users:(("systemd-resolve",pid=456,fd=12))

Flag breakdown:
Flag	Meaning
-t	TCP sockets
-u	UDP sockets
-l	Listening (server) sockets only
-n	Numeric (don't resolve IPs to hostnames — faster)
-p	Process info (requires root for other users' processes)
-a	All sockets (listening and connected)

Common combinations:

# All TCP listening ports with process info:
sudo ss -tlnp

# All UDP listening ports:
sudo ss -ulnp

# All TCP and UDP, listening and connected:
ss -tuna

# Compact summary:
ss -s
# Total: 15
# TCP:   8 (estab 2, closed 0, orphaned 0, timewait 0)
# Transport Total  IP  IPv6
# TCP       8      5   3
# UDP       3      2   1

# Established connections:
ss -tn state established
# Recv-Q Send-Q Local Address:Port  Peer Address:Port
# 0      0      192.168.1.50:47834 142.250.80.46:443
# 0      0      192.168.1.50:52022 140.82.112.3:22

Reading ss Output
Column	Meaning
State	LISTEN (waiting for connections), ESTAB (established/active), TIME-WAIT (closing)
Recv-Q	Bytes queued for receiving (should be 0 in healthy connections)
Send-Q	Bytes queued for sending (should be 0 in healthy connections)
Local Address:Port	Your side of the connection
Peer Address:Port	The remote side
Process	Which program owns this socket

    💡 When to Use ss

        "Is my web server running?" → ss -tlnp | grep :80
        "What's connected to my machine?" → ss -tn state established
        "Is port 22 open?" → ss -tlnp | grep :22
        "Who is using port 8080?" → sudo ss -tlnp | grep :8080

    Non-zero Recv-Q or Send-Q values indicate congestion — data is backing up because the application isn't processing it fast enough.

7. curl and wget: Fetching Data
curl: Transfer Data with URLs

curl (client URL) transfers data to/from servers using protocols like HTTP, HTTPS, FTP:

# Fetch a webpage (prints HTML to terminal):
curl https://example.com

# Download to a file:
curl -o page.html https://example.com

# Download with original filename:
curl -O https://example.com/file.tar.gz

# Follow redirects:
curl -L https://example.com/redirect

# Show response headers only:
curl -I https://example.com
# HTTP/2 200
# content-type: text/html; charset=UTF-8
# content-length: 1256
# ...

# Show both headers and body:
curl -i https://example.com

# Send GET request with custom header:
curl -H "Authorization: Bearer mytoken123" https://api.example.com/data

# Send POST request with JSON data:
curl -X POST -H "Content-Type: application/json" \
  -d '{"name":"John","email":"john@example.com"}' \
  https://api.example.com/users

# Download with progress bar:
curl -L -O --progress-bar https://example.com/largefile.iso

# Resume interrupted download:
curl -C - -O https://example.com/largefile.iso

# Rate-limit download:
curl --limit-rate 1M -O https://example.com/largefile.iso

# Set timeout (abort after 10 seconds):
curl --max-time 10 https://slow-website.com

# Verbose output (for debugging):
curl -v https://example.com 2>&1 | head -20

wget: Download Manager

wget specializes in downloading files with recursive and resume capabilities:

# Download a file:
wget https://example.com/file.tar.gz

# Download to specific path:
wget -O /tmp/archive.zip https://example.com/file.zip

# Resume interrupted download:
wget -c https://example.com/largefile.iso

# Mirror a website (recursively, follow links):
wget -m https://example.com/

# Download in background:
wget -b https://example.com/largefile.iso

# Set retry attempts:
wget --tries=5 --waitretry=10 https://example.com/file.zip

# Download with rate limit:
wget --limit-rate=1M https://example.com/largefile.iso

# Download all files of specific type from a page:
wget -r -A.pdf https://example.com/documents/

curl vs. wget
Feature	curl	wget
Focus	Transfer data (any direction)	Download files
Protocols	HTTP, HTTPS, FTP, SFTP, SCP, many more	HTTP, HTTPS, FTP
POST/PUT	Yes (designed for API calls)	Limited
Recursive download	No	Yes (website mirroring)
Resume	Yes (-C -)	Yes (-c)
Background	No (use &)	Yes (-b)
Scripting	Excellent	Good
Default behavior	Prints to stdout	Saves to file

Use curl for API interactions and quick fetches. Use wget for downloading files and mirroring websites.
8. SSH: Secure Remote Access
What SSH Does

SSH (Secure Shell) encrypts all traffic between you and a remote machine. It replaces insecure protocols like Telnet and FTP. With SSH, you can:

    Execute commands on remote servers
    Transfer files securely
    Tunnel/forward ports
    Set up VPN-like tunnels

Fedora 44 includes the SSH client by default. The SSH server (sshd) may need installation:

# Install SSH server:
sudo dnf install openssh-server

# Start and enable:
sudo systemctl enable --now sshd

Connecting to Remote Machines

# Basic connection:
ssh user@hostname
# ssh john@192.168.1.200
# Enter password for john@192.168.1.200: ******
# [john@remote-machine ~]$

# Connect with a different port:
ssh -p 2222 user@hostname

# Connect with verbose output (debugging):
ssh -v user@hostname

# Execute a single command and return:
ssh user@hostname "uname -a"
# Linux remote-machine 6.19.0-50.fc44.x86_64 ...

# Compress traffic (useful on slow connections):
ssh -C user@hostname

# Forward X11 (run graphical apps remotely):
ssh -X user@hostname
# Then run: firefox &  →  GUI appears on your screen

# Allocate a pseudo-terminal:
ssh -t user@hostname "sudo systemctl restart httpd"

SSH Key Authentication

Passwords are insecure for SSH — they can be guessed, intercepted, or reused. SSH keys use public-key cryptography: you generate a key pair, put the public key on the server, and authenticate without typing passwords.

# Generate an Ed25519 key (modern, fast, secure):
ssh-keygen -t ed25519 -C "john@workstation"
# Generating public/private ed25519 key pair.
# Enter file in which to save the key (/home/john/.ssh/id_ed25519): [Enter]
# Enter passphrase (empty for no passphrase): ********
# Enter same passphrase again: ********
# Your identification has been saved in /home/john/.ssh/id_ed25519
# Your public key has been saved in /home/john/.ssh/id_ed25519.pub

Key files:
File	Type	Permissions	Purpose
~/.ssh/id_ed25519	Private key	600	NEVER share this. Stays on YOUR machine.
~/.ssh/id_ed25519.pub	Public key	644	Safe to share. Goes on remote servers.

Copy your public key to a remote server:

# Using ssh-copy-id (easiest method):
ssh-copy-id user@hostname
# /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s)
# /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed
# user@hostname's password: ******
# Number of key(s) added: 1
# Now try logging into the machine with: "ssh user@hostname"
# and check that you are NOT asked for a password.

# Manual method (if ssh-copy-id unavailable):
cat ~/.ssh/id_ed25519.pub | ssh user@hostname "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

After this, ssh user@hostname connects without a password prompt (though it may ask for your key's passphrase, which is different from the server password).
SSH Agent: Remembering Your Passphrase

If you set a passphrase on your key, typing it every time is tedious. ssh-agent caches the passphrase:

# Start the agent (if not already running):
eval "$(ssh-agent -s)"
# Agent pid 12345

# Add your key:
ssh-add ~/.ssh/id_ed25519
# Enter passphrase for /home/john/.ssh/id_ed25519: ********
# Identity added: /home/john/.ssh/id_ed25519

# Now SSH connections won't ask for the passphrase:
ssh user@hostname
# Connects immediately

GNOME automatically starts an SSH agent on login. Keys with passphrases are loaded via the GNOME keyring prompt.
SSH Configuration File

Simplify frequent connections with ~/.ssh/config:

nano ~/.ssh/config

# My home server
Host homeserver
    HostName 192.168.1.200
    User john
    Port 22
    IdentityFile ~/.ssh/id_ed25519

# Work dev server
Host workserver
    HostName dev.company.com
    User j.smith
    Port 2222
    IdentityFile ~/.ssh/work_key
    ForwardAgent yes

# Short alias for GitHub
Host github
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_key

Now instead of typing:

ssh -p 2222 j.smith@dev.company.com -i ~/.ssh/work_key

You simply type:

ssh workserver

SCP and rsync: File Transfer Over SSH

# Copy a file TO a remote machine:
scp /home/john/report.pdf user@hostname:/home/user/

# Copy a file FROM a remote machine:
scp user@hostname:/home/user/report.pdf /home/john/Downloads/

# Copy a directory recursively:
scp -r ~/projects/ user@hostname:~/projects/

# Use a custom port:
scp -P 2222 file.txt user@hostname:~/

# rsync (better for large/synced transfers):
rsync -avz ~/projects/ user@hostname:~/projects/
rsync -avz --delete ~/projects/ user@hostname:~/projects/
# --delete mirrors exactly (removes files on destination not present on source)

Port Forwarding (Tunnels)

SSH can tunnel traffic through an encrypted connection:

# Local port forwarding: access remote service locally
# Access a database running on the remote server (port 5432) from your machine:
ssh -L 5432:localhost:5432 user@hostname
# Now: psql -h localhost -p 5432 connects to the remote database

# Remote port forwarding: expose local service to remote network
ssh -R 8080:localhost:80 user@hostname
# Remote users can access your local web server via http://hostname:8080

# Dynamic port forwarding (SOCKS proxy):
ssh -D 1080 user@hostname
# Configure your browser to use SOCKS5 proxy at localhost:1080
# All traffic routes through the SSH tunnel

9. firewalld: Controlling Traffic
What firewalld Does

firewalld is Fedora's firewall management tool. It uses nftables as its backend (replacing the older iptables backend). firewalld provides a friendlier abstraction over raw nftables rules.

Core concepts:
Concept	Meaning
Zone	Trust level for a network interface
Service	Predefined set of rules for a protocol (HTTP, SSH, etc.)
Port	Individual TCP/UDP port to open/close
Runtime	Changes apply immediately but are temporary (lost on restart)
Permanent	Changes saved to config and persist across reboots
Checking Firewall Status

# Is firewalld running?
systemctl status firewalld
# Active: active (running)

# View active zones and interfaces:
firewall-cmd --get-active-zones
# FedoraWorkstation
#   interfaces: enp3s0 wlp2s0

# View default zone:
firewall-cmd --get-default-zone
# FedoraWorkstation

# List all available zones:
firewall-cmd --get-zones
# FedoraWorkstation block dmz drop external home internal public trusted work

# List services allowed in current zone:
firewall-cmd --list-services
# dhcpv6-client mdns samba-client ssh

Understanding Zones
Zone	Trust Level	Typical Use
trusted	Allow all	Never use on public networks
home	High trust	Home network
internal	High trust	Internal LAN
work	Medium trust	Office network
public	Low trust	Public Wi-Fi
block	Reject all	Block all incoming
drop	Drop all (silent)	Maximum stealth
FedoraWorkstation	Custom	Fedora's default (allows SSH, mDNS, DHCPv6)
Opening Ports and Services

# List all currently allowed services:
firewall-cmd --list-all
# FedoraWorkstation (active)
#   target: default
#   icmp-block-inversion: no
#   interfaces: enp3s0 wlp2s0
#   sources:
#   services: dhcpv6-client mdns samba-client ssh
#   ports:
#   protocols:
#   forward: yes
#   masquerade: no
#   forward-ports:
#   source-ports:
#   icmp-blocks:
#   rich rules:

# Add a service temporarily (runtime only):
sudo firewall-cmd --add-service=http

# Add permanently (survives reboot):
sudo firewall-cmd --permanent --add-service=http

# Apply permanent changes to runtime:
sudo firewall-cmd --reload

# Open a specific port:
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload

# Remove a service:
sudo firewall-cmd --permanent --remove-service=http
sudo firewall-cmd --reload

Available Services

# List all predefined services:
firewall-cmd --get-services
# RH-Satellite-6 amanda-client amanda-k5-client amqp ... http https imap ...

# View what a service includes:
firewall-cmd --info-service http
# http
#   ports: 80/tcp
#   protocols:
#   destination:
#   source-ports:
#   modules:
#   description: HTTP is the protocol used to serve web pages.

Changing Interface Zones

# Assign an interface to a different zone:
sudo firewall-cmd --zone=public --change-interface=wlp2s0

# Make permanent:
sudo nmcli connection modify "HomeNetwork" connection.zone public

Rich Rules (Advanced)

For more granular control:

# Allow SSH from a specific IP only:
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.50" service name="ssh" accept'

# Block a specific IP:
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="10.0.0.5" reject'

# Rate-limit SSH (max 5 connections per minute):
sudo firewall-cmd --permanent --add-rich-rule='rule service name="ssh" limit value="5/m" accept'

sudo firewall-cmd --reload

    💡 FedoraWorkstation Zone

    Fedora Workstation uses a custom zone called FedoraWorkstation instead of the standard public zone. This zone allows more outbound services (like mDNS for device discovery and Samba for file sharing) while still protecting against unsolicited inbound traffic. If you install a web server or other service, you'll need to explicitly open its ports.

10. Network Troubleshooting Methodology

When networking breaks, follow this systematic approach from bottom to top:
Step 1: Physical/Hardware Layer

# Is the interface up?
ip -br link show
# enp3s0           UP             00:1a:2b:3c:4d:5e
# wlp2s0           DOWN           00:1c:2d:3e:4f:60

# If DOWN, bring it up:
sudo ip link set enp3s0 up
# Or via NetworkManager:
nmcli device connect enp3s0

# Check for errors:
ip -s link show enp3s0
# Look for non-zero values in errors/dropped columns

Step 2: Link-Local (Can you reach the gateway?)

# Can you ping your gateway?
ip route show default
# default via 192.168.1.1 dev enp3s0

ping -c 3 192.168.1.1
# If this fails: cable issue, Wi-Fi not connected, or IP not configured

Step 3: DNS (Can you resolve names?)

# Can you resolve a hostname?
host google.com
# If this fails but ping 8.8.8.8 works: DNS problem

# Check DNS config:
resolvectl status

# Try a different DNS server:
dig @8.8.8.8 google.com
# If this works but normal resolution fails: your DNS server config is wrong

Step 4: External Connectivity (Can you reach the internet?)

# Ping a public IP:
ping -c 3 8.8.8.8

# If ping fails but gateway works: ISP issue or routing problem

# Trace the path:
mtr 8.8.8.8
# Find where packets stop

Step 5: Service/Port (Can you reach a specific service?)

# Is the port open on the remote machine?
curl -v --connect-timeout 5 https://example.com
# Connection refused = service not running
# Timeout = firewall blocking or host unreachable

# Check if your firewall is blocking:
sudo firewall-cmd --list-all

Quick Diagnostic Cheat Sheet

# Run all these in sequence to pinpoint issues:

# 1. Is the interface up and has an IP?
ip -br a

# 2. Can you reach your gateway?
ping -c 3 $(ip route show default | awk '/default/ {print $3}')

# 3. Can you resolve DNS?
host fedoraproject.org

# 4. Can you reach the internet?
ping -c 3 8.8.8.8

# 5. Can you reach a web server?
curl -sI https://fedoraproject.org | head -5

# 6. What's listening on your machine?
sudo ss -tlnp

# 7. What's your firewall doing?
sudo firewall-cmd --list-all

# 8. What does NetworkManager think?
nmcli general status
nmcli device status

Try It Yourself
Exercise 1: Survey Your Network

# Document your network configuration:
ip -br a                    # Your IP addresses
ip route show default       # Your gateway
ip neigh show               # ARP cache (known local devices)
nmcli device status         # NetworkManager's view
firewall-cmd --list-all     # Firewall rules
sudo ss -tlnp               # Listening services

Save the output to a reference file — compare it later if something breaks.
Exercise 2: DNS Detective

# Resolve your favorite websites:
host google.com
dig +short github.com
nslookup wikipedia.org

# Find out which DNS servers you're using:
resolvectl status | grep "DNS Server"

# Try a specific DNS server:
dig @1.1.1.1 example.com +short

# Add a local hosts entry:
sudo nano /etc/hosts
# Add: 127.0.0.1 mytestsite
# Then: ping -c 1 mytestsite → resolves to 127.0.0.1
# Remove the line afterward

Exercise 3: Connection Testing

# Ping your gateway:
GATEWAY=$(ip route show default | awk '/default/ {print $3}')
ping -c 4 $GATEWAY

# Ping a public server:
ping -c 4 8.8.8.8

# Trace your path to the internet:
mtr -c 5 --report google.com

# Check what's connecting to your machine:
ss -tn state established

Exercise 4: Firewall Exploration

# View your current firewall configuration:
firewall-cmd --list-all

# List available services:
firewall-cmd --get-services | tr ' ' '\n' | head -20

# Temporarily open port 8080:
sudo firewall-cmd --add-port=8080/tcp
# Verify:
firewall-cmd --list-ports

# Remove it:
sudo firewall-cmd --remove-port=8080/tcp

# View a service definition:
firewall-cmd --info-service ssh

Exercise 5: SSH Setup

# Generate a key pair (if you don't have one):
ssh-keygen -t ed25519 -C "$USER@$(hostname)"

# View your keys:
ls -la ~/.ssh/
# Private key: id_ed25519 (permissions should be 600)
# Public key: id_ed25519.pub (permissions should be 644)

# Create an SSH config entry:
nano ~/.ssh/config
# Add:
# Host localhost
#     User $USER
#     IdentityFile ~/.ssh/id_ed25519

# Test SSH to your own machine:
ssh localhost
# Should connect without password (using your key)

# Check what's listening:
sudo ss -tlnp | grep :22
# SSH daemon should be on port 22

Exercise 6: curl API Practice

# Fetch a webpage:
curl -s https://example.com | head -20

# Get HTTP headers:
curl -sI https://fedoraproject.org

# Test JSON API:
curl -s https://api.github.com/repos/fedora-linux/fedora | python3 -m json.tool | head -20

# Download a file with progress:
curl -L -O --progress-bar https://example.com/smallfile.txt

# Use wget to download:
wget -q https://example.com/ -O /tmp/example_page.html
head /tmp/example_page.html

Troubleshooting Reference
Symptom	Likely Cause	First Check
No internet at all	Interface down or no IP	ip -br a — is interface UP with an address?
Can ping IP but not hostname	DNS failure	host google.com — does it resolve?
Can reach gateway but not internet	ISP or routing issue	traceroute 8.8.8.8 — where does it stop?
SSH connection refused	sshd not running or firewalled	sudo ss -tlnp | grep :22 and firewall-cmd --list-services
SSH "connection timed out"	Firewall blocking or host unreachable	ping the host first, check firewalld on both sides
Web server unreachable	Port 80/443 not open in firewall	sudo firewall-cmd --add-service=http && sudo firewall-cmd --add-service=https
Wi-Fi won't connect	Wrong password or driver issue	nmcli device wifi list — is the network visible? journalctl -u NetworkManager
Intermittent drops	Signal strength or driver issue	nmcli device wifi list (check signal), dmesg | grep -i firmware
Slow network	Congestion or hardware issue	mtr to find bottleneck, ip -s link for errors
Can't resolve local hostnames	No search domain or mDNS issue	Check /etc/hosts, try resolvectl status
Duplicate IP address	DHCP conflict	ip neigh for ARP conflicts, sudo nmcli connection modify "name" ipv4.addresses for static
VPN not routing	firewalld zone or forwarding issue	Check zone assignment, firewall-cmd --list-all
What's Next

You now understand the complete networking stack on Fedora 44: conceptual foundations (IP addresses, subnets, gateways, DNS, ports, NAT), NetworkManager (nmcli device/connection management, nmtui interface, static IP and DHCP configuration, Wi-Fi connectivity, hostname management), the ip command family (address viewing, route inspection, link statistics, brief output mode), DNS resolution (systemd-resolved, /etc/resolv.conf stub resolver, /etc/hosts overrides, dig/host/nslookup/resolvectl queries), connectivity testing (ping, traceroute, tracepath, mtr for continuous monitoring), socket statistics with ss (listening ports, established connections, process attribution), data transfer with curl and wget, SSH (password and key authentication, ssh-agent, ~/.ssh/config, SCP, rsync, port forwarding), firewalld (zones, services, ports, rich rules, runtime vs. permanent changes, the FedoraWorkstation zone), and a systematic troubleshooting methodology from physical layer through DNS to service-level connectivity.

Chapter 22 explores hardware and drivers — how Fedora 44 detects and manages your physical devices, how to install proprietary drivers (particularly NVIDIA GPUs), how to check hardware compatibility, and how to troubleshoot device issues. You'll learn about lspci, lsusb, lscpu, lshw, hwinfo, kernel modules, and the /sys and /dev hardware interfaces introduced in Chapter 14.

Until then, survey your network. Run the diagnostic cheat sheet from Section 10 on your own machine. Understanding what "normal" looks like makes diagnosing "broken" dramatically easier.

    ✅ Chapter 21 Complete You understand networking concepts in plain language (postal service analogy mapping IP/subnet/gateway/DNS/port/MAC to familiar concepts, IPv4 vs. IPv6 dual-stack, private vs. public address ranges, CIDR notation with subnet mask equivalents, privileged vs. unprivileged ports), NetworkManager (systemd service status, nmtui menu interface, nmcli device/connection/general commands, Wi-Fi scanning and connection, static IP configuration with gateway/DNS, DHCP revert, hostname setting, profile storage in /etc/NetworkManager/system-connections/), the ip command family (ip address/link/route/neigh show, -br brief mode, statistics with -s, predictable interface naming enp/wl/ww/lo/br/virbr/tun/wg, mapping to deprecated ifconfig/netstat commands), DNS and name resolution (systemd-resolved stub resolver at 127.0.0.53, resolvectl status/query/flush-caches, /etc/resolv.conf managed by NetworkManager, /etc/hosts local overrides, dig/host/nslookup/resolvectl query tools, changing DNS servers via nmcli), connectivity testing (ping with -c/-i/-s/-f flags, ICMP caveats, traceroute and tracepath for path discovery, mtr for continuous monitoring with loss/latency statistics), ss socket statistics (replacing netstat, -t/-u/-l/-n/-p flag combinations, reading State/Recv-Q/Send-Q/Local/Peer/Process columns, listening vs. established connections, ss -s summary), curl (GET/POST requests, headers, redirects, downloads with -o/-O/-C, rate limiting, timeouts, verbose output, API interaction with JSON), wget (recursive downloads, -c resume, -b background, -m mirror, --limit-rate, -A file type filter), SSH (client and server installation, ssh user@host connections, -p/-L/-R/-D/-X/-C/-t flags, Ed25519 key generation, ssh-copy-id for key deployment, ssh-agent for passphrase caching, ~/.ssh/config for connection aliases, scp and rsync for file transfer, local/remote/dynamic port forwarding), firewalld (nftables backend, zones with trust levels, services vs. ports, runtime vs. permanent changes with --reload, FedoraWorkstation custom zone, rich rules for IP-specific filtering and rate limiting, nmcli zone assignment), systematic troubleshooting methodology (physical → gateway → DNS → internet → service-level with diagnostic commands at each step), practical exercises covering network surveying/DNS investigation/connectivity testing/firewall exploration/SSH setup/API interaction, and comprehensive troubleshooting reference table for twelve common network problems.

