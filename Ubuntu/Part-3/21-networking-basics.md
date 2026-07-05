# Chapter 21: Networking Basics

## Your Computer Is Not an Island

Every time you browse a website, download a package, send an email, or stream music, your computer participates in a vast, layered network of communication. On Ubuntu, all of that activity is manageable from the terminal — and understanding how it works gives you enormous control.

This chapter covers networking from a practical standpoint: checking your connection, finding your IP address, connecting to Wi-Fi, testing connectivity, downloading files, transferring data between machines, and troubleshooting when things go wrong. No networking degree required — we'll explain everything as we go.

## Network Concepts in Plain Language

Before diving into commands, here are the concepts you need to understand. Don't worry about memorizing — you'll see these terms applied practically throughout the chapter.

### IP Addresses

An **IP address** is your computer's network address — like a phone number for your machine. Every device on a network has one.

Two versions exist:

- **IPv4** — The classic format: four numbers separated by dots, e.g., `192.168.1.42`. Each number ranges from 0 to 255. Still the most common on home networks.
- **IPv6** — The newer format: eight groups of hexadecimal digits, e.g., `2001:db8::8a2e:370:7334`. Designed to solve IPv4's address exhaustion problem. Becoming more common, especially on mobile and enterprise networks.

### Public vs. Private IP Addresses

| Type | Range | Who Uses It |
|---|---|---|
| Private (local) | `10.x.x.x`, `172.16-31.x.x`, `192.168.x.x` | Devices on your home/office network |
| Public (external) | Everything else | Your router, websites, servers on the internet |

Your computer has a **private** IP address (e.g., `192.168.1.42`) visible only within your local network. Your router has a **public** IP address assigned by your ISP. When you visit a website, the router translates between private and public addresses using NAT (Network Address Translation).

### Network Interfaces

A **network interface** is the physical or virtual connection point between your computer and a network. Each interface has a name:

| Interface Name | What It Is |
|---|---|
| `lo` | Loopback — a virtual interface that connects to itself (`127.0.0.1`) |
| `enp3s0`, `eno1`, `eth0` | Ethernet (wired) connections |
| `wlp2s0`, `wlan0` | Wi-Fi (wireless) connections |
| `docker0`, `br-*` | Virtual bridges (created by Docker, VMs, etc.) |
| `tun0`, `wg0` | VPN tunnel interfaces |
| `virbr0` | Virtual bridge (created by libvirt/KVM) |

> 💡 **Why the Weird Names?**
>
> Older Ubuntu versions used predictable names like `eth0` (first Ethernet), `wlan0` (first Wi-Fi). Modern Ubuntu uses names derived from hardware topology: `enp3s0` breaks down as:
>
> - `en` — Ethernet
> - `p3` — PCI bus 3
> - `s0` — slot 0
>
> This naming scheme ensures the same interface always gets the same name, even if hardware is added or removed. It looks intimidating at first, but you'll get used to it. `ip link` shows you all interface names at a glance.

### DNS — The Internet's Phonebook

Computers communicate using IP addresses (e.g., `93.184.216.34`). Humans remember names (e.g., `example.com`). **DNS** (Domain Name System) translates names to addresses. When you type a website, DNS looks up the IP address behind it.

### Ports

If an IP address is a building, a **port** is a room number. A single computer can run many network services simultaneously — each listens on a different port.

| Port | Protocol | What Uses It |
|---|---|---|
| 22 | SSH | Secure shell (remote access) |
| 53 | DNS | Domain name lookups |
| 80 | HTTP | Web traffic (unencrypted) |
| 443 | HTTPS | Web traffic (encrypted) |
| 25 | SMTP | Email sending |
| 465, 587 | SMTPS | Email sending (encrypted) |
| 993 | IMAPS | Email receiving (encrypted) |
| 3306 | MySQL | Database |
| 5432 | PostgreSQL | Database |

Ports 1–1023 are "well-known" (reserved for standard services). Ports 1024–49151 are "registered." Ports 49152–65535 are "dynamic" (temporary connections).

## Checking Your Network Configuration

### `ip` — The Modern Network Command

The `ip` command is the modern replacement for the older `ifconfig`. It handles everything: viewing interfaces, assigning addresses, managing routes. On Ubuntu 26.04, `ip` is the standard.

**View all network interfaces and their IP addresses:**

ip addr

Or the shorter form:

ip a

Output:

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 3c:5a:b3:2f:1e:8a brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.42/24 brd 192.168.1.255 scope global dynamic noprefixroute enp3s0
       valid_lft 86381sec preferred_lft 86381sec
    inet6 2001:db8::a4b2:7e31:1f3c/64 scope global dynamic noprefixroute
       valid_lft 86381sec preferred_lft 86381sec

Breaking down the key fields:
Field	Meaning
lo	Loopback interface (always present, connects to self)
enp3s0	Ethernet interface
state UP	Interface is active and connected
link/ether 3c:5a:b3:...	MAC address (hardware address, unique to the NIC)
inet 192.168.1.42/24	IPv4 address with subnet mask (/24 = 255.255.255.0)
inet6 2001:db8::...	IPv6 address
valid_lft 86381sec	Lease time remaining (DHCP assigns addresses temporarily)

View just the interface names and states:

ip link

Or shorter:

ip l

View the routing table (how your computer decides where to send traffic):

ip route

Output:

default via 192.168.1.1 dev enp3s0 proto dhcp metric 100
192.168.1.0/24 dev enp3s0 proto kernel scope link src 192.168.1.42

The first line shows your default gateway — 192.168.1.1 — which is your router. All traffic destined for outside your local network goes through this address.

View only your IP addresses (quick and clean):

ip -br a

Output:

lo               UNKNOWN        127.0.0.1/8 ::1/128
enp3s0           UP             192.168.1.42/24 metric 100 2001:db8::a4b2/64

The -br (brief) flag strips out the details — just names, states, and addresses. This is the command you'll use most often.

    💡 ip vs. ifconfig

    You'll see ifconfig mentioned in older tutorials. It's from the net-tools package, which is deprecated. ip is the modern replacement. ifconfig still works if you install net-tools, but it's missing features, and future Ubuntu releases may drop it entirely. Use ip.

Managing Connections with NetworkManager

Ubuntu uses NetworkManager to handle network connections — both wired and wireless. It's the system service that automatically connects to your Ethernet or remembers your Wi-Fi passwords. From the terminal, you control it with nmcli.
nmcli — NetworkManager Command Line Interface

Check overall network status:

nmcli general status

Output:

STATE      CONNECTIVITY  WIFI-HW  WIFI     WWAN-HW  WWAN
connected  full          enabled  enabled  enabled  enabled

List all network devices and their states:

nmcli device status

Output:

DEVICE   TYPE      STATE         CONNECTION
enp3s0   ethernet  connected     Wired connection 1
wlp2s0   wifi      disconnected  --
lo       loopback  unmanaged     --

This tells you which interfaces are active, what type they are, and which connection profile they're using.

List saved connection profiles:

nmcli connection show

Shows all networks you've connected to before (Wi-Fi networks, wired profiles, VPNs), including their UUIDs and types.
Connecting to Wi-Fi

Scan for available Wi-Fi networks:

nmcli device wifi list

Output:

IN-USE  BSSID              SSID             MODE   CHAN  RATE        SIGNAL  BARS  SECURITY
        00:1A:2B:3C:4D:5E  HomeNetwork      Infra  6     270 Mbit/s  89      ▂▄▆█  WPA2
        00:1A:2B:3C:4D:6F  CoffeeShopWiFi   Infra  11    130 Mbit/s  67      ▂▄▆_  WPA2
        00:1A:2B:3C:4D:7A  Guest            Infra  1     54 Mbit/s   34      ▂▄__  WPA1

Key columns:

    IN-USE — an asterisk (*) means you're currently connected to this network
    SSID — the network name
    SIGNAL — signal strength (higher is better)
    SECURITY — encryption type (avoid open networks with -- for sensitive activity)

Connect to a Wi-Fi network:

nmcli device wifi connect "HomeNetwork" password "yourpassword"

Quotes are needed if the SSID or password contains spaces. On success, you'll see:

Device 'wlp2s0' successfully activated with 'a1b2c3d4-e5f6-...'

Connect to a hidden network (SSID not broadcast):

nmcli device wifi connect "HiddenNetwork" password "yourpassword" hidden yes

Disconnect from Wi-Fi:

nmcli device disconnect wlp2s0

Turn Wi-Fi on/off:

nmcli radio wifi on
nmcli radio wifi off

Managing Connection Profiles

When you connect to a network, NetworkManager saves a connection profile so it can reconnect automatically. You can manage these profiles:

View details of a specific connection:

nmcli connection show "HomeNetwork"

Shows all settings for that connection — IP assignment method, DNS servers, gateway, auto-connect behavior, and dozens of other parameters.

Delete a saved connection:

nmcli connection delete "CoffeeShopWiFi"

Removes the profile — you won't auto-connect to that network anymore.

Set a static IP address (advanced):

nmcli connection modify "Wired connection 1" ipv4.addresses 192.168.1.100/24
nmcli connection modify "Wired connection 1" ipv4.gateway 192.168.1.1
nmcli connection modify "Wired connection 1" ipv4.dns "8.8.8.8 1.1.1.1"
nmcli connection modify "Wired connection 1" ipv4.method manual
nmcli connection up "Wired connection 1"

This switches from DHCP (automatic IP) to a static IP. The final nmcli connection up activates the changes immediately.

    ⚠️ Changing Network Settings Remotely

    If you're connected to a machine over SSH, changing its network configuration can disconnect you. If you're experimenting, make sure you have physical access or a backup connection method.

nmtui — Text-Based Network Manager

If you prefer a menu-driven interface over typing commands, nmtui (NetworkManager Text User Interface) provides a fullscreen, navigable menu:

nmtui

You'll see a menu with three options:

    Edit a connection — Add, modify, or delete connection profiles
    Activate a connection — Connect to or disconnect from networks
    Set system hostname — Change your computer's network name

Use arrow keys to navigate, Enter to select, Esc to go back. nmtui is especially useful on servers or minimal installations where no graphical network settings app exists.
Testing Connectivity
ping — "Are You There?"

ping sends small packets to another computer and waits for a response. It's the simplest connectivity test — "can my computer reach that computer?"

Ping a website:

ping -c 4 google.com

The -c 4 flag sends exactly 4 packets (otherwise ping runs forever on Linux):

PING google.com (142.250.190.46) 56(84) bytes of data.
64 bytes from 142.250.190.46: icmp_seq=1 ttl=117 time=12.3 ms
64 bytes from 142.250.190.46: icmp_seq=2 ttl=117 time=11.8 ms
64 bytes from 142.250.190.46: icmp_seq=3 ttl=117 time=13.1 ms
64 bytes from 142.250.190.46: icmp_seq=4 ttl=117 time=12.0 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss
rtt min/avg/max/mdev = 11.8/12.3/13.1/0.5 ms

Key information:

    142.250.190.46 — The IP address DNS resolved google.com to
    ttl=117 — Time to Live (hops before the packet is discarded)
    time=12.3 ms — Round-trip latency (how long the response took)
    0% packet loss — All packets were answered (good)

Interpreting ping results:
Result	Meaning
Replies received, low latency	Connection is healthy
Some packets lost (e.g., 25% loss)	Unstable connection — interference, congestion, or failing hardware
100% packet loss	No response — destination unreachable, blocked by firewall, or DNS failed
"Temporary failure in name resolution"	DNS problem — can't resolve the hostname to an IP
"Destination Host Unreachable"	Your machine has no route to the destination network

Ping an IP address directly (bypass DNS):

ping -c 4 8.8.8.8

If this works but ping google.com doesn't, your DNS is the problem.

Continuous ping (no limit):

ping google.com

Press Ctrl+C to stop. Useful for watching for intermittent drops over time.
hostname — Your Computer's Name

hostname

Shows your computer's network name (e.g., ubuntu-desktop).

Full network identity:

hostnamectl

Output:

Static hostname: ubuntu-desktop
       Icon name: computer-desktop
         Chassis: desktop
      Machine ID: a1b2c3d4e5f6...
         Boot ID: 7g8h9i0j1k2l...
Operating System: Ubuntu 26.04 LTS
          Kernel: Linux 7.0.0-generic
    Architecture: x86-64
 Hardware Vendor: Dell Inc.
  Hardware Model: OptiPlex 7090

Change your hostname:

sudo hostnamectl set-hostname mycomputer

The change takes effect immediately for new sessions. Some programs may need a restart to pick up the new name.
DNS Lookups

When websites don't load but ping 8.8.8.8 works, DNS is the likely culprit. These tools help you diagnose DNS issues.
resolvectl — DNS Resolution

Ubuntu 26.04 uses systemd-resolved for DNS management. View your DNS configuration:

resolvectl status

Output includes:

    Your DNS server addresses (usually your router or ISP's DNS)
    The domains being searched
    Per-interface DNS settings

Query a specific DNS record:

resolvectl query google.com

Output:

google.com: 142.250.190.46
           -- link: enp3s0
           -- Information acquired via protocol: DNS/IPv4 via UDP

dig — Detailed DNS Queries

dig google.com

Output:

; <<>> DiG 9.18.28 <<>> google.com
;; QUESTION SECTION:
;google.com.            IN  A

;; ANSWER SECTION:
google.com.     300  IN  A   142.250.190.46

;; Query time: 24 msec
;; SERVER: 192.168.1.1#53(192.168.1.1)

Key fields:

    ANSWER SECTION — The resolved IP address
    Query time — How long the DNS lookup took (over 100ms suggests slow DNS)
    SERVER — Which DNS server answered (usually your router)

Query a specific DNS server:

dig @8.8.8.8 google.com

Forces the query through Google's DNS server (8.8.8.8), bypassing your configured DNS. Useful for testing whether your ISP's DNS is the problem.
host — Simple DNS Lookup

host google.com

Output:

google.com has address 142.250.190.46
google.com has IPv6 address 2001:4860:4860::8888
google.com mail is handled by 10 smtp.google.com.

Simpler than dig — just the answers, no technical details. Good for quick lookups.

host 142.250.190.46

Reverse lookup — finds the hostname for an IP address. Useful when analyzing logs.

    💡 dig vs. host vs. nslookup

        host — Simple, human-readable output. Best for quick checks.
        dig — Detailed, technical output. Best for troubleshooting and learning.
        nslookup — Older tool, still works but deprecated. Use dig or host instead.

Checking Open Connections with ss

The ss (socket statistics) command shows active network connections — what's talking to what. It replaces the older netstat command.
Viewing Active Connections

All TCP connections:

ss -t

All TCP connections with listening (server) sockets:

ss -tln

Breaking down the flags:
Flag	Meaning
-t	TCP connections
-u	UDP connections
-l	Listening sockets only (services waiting for connections)
-n	Numeric addresses (don't resolve hostnames — faster)
-p	Show process using the socket (requires sudo)
-a	All sockets (listening and non-listening)

Output of ss -tln:

State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port  Process
LISTEN  0       128     0.0.0.0:22          0.0.0.0:*          users:(("sshd",pid=890,fd=3))
LISTEN  0       4096    127.0.0.1:631       0.0.0.0:*          users:(("cupsd",pid=1023,fd=7))
LISTEN  0       4096    *:5432              *:*                users:(("postgres",pid=1102,fd=5))

This tells you:

    Port 22 is open — SSH is listening for connections
    Port 631 is open — CUPS print server (bound to localhost only, so only accessible locally)
    Port 5432 is open — PostgreSQL database (bound to all interfaces)

    💡 Security Implication

    Every listening port is a potential entry point for attackers. ss -tlnp (with sudo) shows exactly what's listening. If you see unexpected open ports, investigate with ps aux | grep PID to identify the process.

See established connections (active talks):

ss -tn state established

Shows currently active connections — who's connected to your machine and to whom you're connected:

Recv-Q  Send-Q  Local Address:Port   Peer Address:Port
0       0       192.168.1.42:51234   142.250.190.46:443
0       0       192.168.1.42:56789   140.82.121.4:22

This shows an HTTPS connection to Google (port 443) and an SSH connection (port 22).
Downloading Files
curl — Transfer Data from/to Servers

curl is a versatile tool for transferring data over HTTP, HTTPS, FTP, and more. It's the swiss army knife of network data transfer.

Download a file:

curl -O https://example.com/file.zip

The -O (capital O) saves the file with its original name. Without it, curl dumps the file content to your terminal.

Download and rename:

curl -o myfile.zip https://example.com/file.zip

View HTTP response headers:

curl -I https://example.com

Output:

HTTP/2 200
content-type: text/html; charset=UTF-8
content-length: 1256
server: ECS (dcb/7EA2)
last-modified: Thu, 17 Oct 2024 07:18:26 GMT

Shows the status code, server type, content type, and modification date — without downloading the body.

Download with progress bar:

curl -# -O https://example.com/largefile.iso

Follow redirects:

curl -L -o page.html https://example.com/redirect

The -L flag follows HTTP redirects (when a URL sends you to a different URL). Essential for downloading from shortened or redirected links.

Send data with POST:

curl -X POST -d "name=John&email=john@example.com" https://api.example.com/submit

Sends an HTTP POST request with form data. Useful for interacting with APIs or testing web forms.

Download with authentication:

curl -u username:password -O https://example.com/protected/file.zip

Resume a partially downloaded file:

curl -C - -O https://example.com/largefile.iso

The -C - flag resumes the download from where it left off — invaluable for large files on unreliable connections.
wget — Simple Recursive Downloader

wget is simpler than curl for straightforward file downloads. It retries failed downloads automatically and supports recursive downloading (grabbing entire websites).

Download a file:

wget https://example.com/file.zip

Saves with the original filename — no flags needed.

Download to a specific filename:

wget -O myfile.zip https://example.com/file.zip

Continue a partial download:

wget -c https://example.com/largefile.iso

The -c (continue) flag resumes interrupted downloads.

Download quietly (no output):

wget -q https://example.com/file.zip

Mirror a website (download all pages):

wget -m -k -K https://example.com

    -m — mirror (recursive download)
    -k — convert links to local references
    -K — keep backup of original files

    💡 curl vs. wget

    Both download files. Choose based on need:

        curl — Better for API calls, HTTP headers, POST requests, and pipes
        wget — Better for simple downloads, recursive site mirroring, automatic retries

    Neither is "better" — they overlap heavily. Most Linux users use both regularly.

Transferring Files Between Machines
scp — Secure Copy Over SSH

scp (secure copy) copies files between your computer and a remote machine over an encrypted SSH connection.

Copy a file TO a remote machine:

scp document.txt john@192.168.1.100:/home/john/Documents/

Breakdown:

    document.txt — the local file to send
    john — username on the remote machine
    192.168.1.100 — remote machine's IP address (or hostname)
    :/home/john/Documents/ — destination directory on the remote machine

Copy a file FROM a remote machine:

scp john@192.168.1.100:/home/john/report.pdf ~/Downloads/

Copy a directory recursively:

scp -r ~/projects john@192.168.1.100:/home/john/

The -r flag copies the entire projects directory and its contents.

Specify a non-default SSH port:

scp -P 2222 document.txt john@192.168.1.100:/home/john/

Capital -P specifies the port (SSH default is 22).

    💡 scp Syntax Confusion

    The hardest part of scp is remembering where the local file goes and where the remote file goes. The pattern is always:

    scp SOURCE DESTINATION

    Either (or both) can be remote. Remote locations always have the format user@host:path.

rsync — Efficient File Synchronization

rsync is scp on steroids. It only transfers the differences between files — if a file already exists on the destination and only part of it changed, rsync sends only the changed portions. For large files or frequent syncs, this saves enormous bandwidth and time.

Basic sync:

rsync -av ~/Documents/ john@192.168.1.100:/home/john/Documents/

Flags:

    -a — archive mode (preserves permissions, timestamps, symbolic links, recurses directories)
    -v — verbose (shows what's being transferred)

Important: the trailing slash matters!
Command	What Happens
rsync -av ~/Documents/ remote:/path/	Copies the contents of Documents into /path/
rsync -av ~/Documents remote:/path/	Copies the Documents directory itself into /path/Documents/

This subtle difference trips up many users. Be deliberate about the trailing slash.

Show progress during transfer:

rsync -av --progress ~/Videos/ john@192.168.1.100:/home/john/Videos/

Delete files on destination that no longer exist on source:

rsync -av --delete ~/Documents/ john@192.168.1.100:/home/john/Documents/

The --delete flag makes the destination an exact mirror of the source. Use with caution — files on the destination that don't exist on the source will be deleted.

Dry run (see what would happen without changing anything):

rsync -avn --delete ~/Documents/ john@192.168.1.100:/home/john/Documents/

The -n (dry run) flag is invaluable — always use it before a destructive rsync command to preview the results.

Local rsync (no remote machine needed):

rsync -av /home/john/important/ /backup/important/

rsync works locally too — it's an excellent backup tool (covered more in Chapter 27).

    💡 scp vs. rsync

        scp — Simple, always available, good for one-off quick copies
        rsync — More efficient for large or repeated transfers, supports resuming, deleting, dry runs

    Rule of thumb: use scp for quick single-file transfers. Use rsync for anything involving directories, large files, or regular backups.

Firewall Basics with ufw

UFW (Uncomplicated Firewall) is Ubuntu's default firewall management tool. It provides a simple interface to iptables (the kernel's packet filtering system).
Checking Firewall Status

sudo ufw status

Output:

Status: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
443/tcp                    ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
443/tcp (v6)               ALLOW       Anywhere (v6)

If it says Status: inactive, the firewall isn't running.
Enabling the Firewall

sudo ufw enable

Enables the firewall and starts it at boot. You'll see a warning that existing SSH connections may be disrupted.

    ⚠️ Don't Lock Yourself Out

    If you're connected over SSH, always allow SSH before enabling the firewall:

    sudo ufw allow ssh
    sudo ufw enable

    Otherwise, the firewall will block your SSH connection, locking you out of the remote machine.

Allowing and Denying Services

Allow a specific port:

sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

Allow by service name:

sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https

Deny a port:

sudo ufw deny 3306

Blocks external access to MySQL (port 3306). Local connections still work.

Allow from a specific IP address:

sudo ufw allow from 192.168.1.50 to any port 22

Only the machine at 192.168.1.50 can access SSH. More restrictive and secure.

Delete a rule:

sudo ufw delete allow 80/tcp

Disable the firewall:

sudo ufw disable

Reset to defaults:

sudo ufw reset

Removes all rules and returns to default settings.

    💡 Default UFW Policy

    UFW's default policy is:

        Deny all incoming connections (blocks unsolicited traffic)
        Allow all outgoing connections (your computer can reach out freely)

    This is the safest default. You selectively allow incoming connections only for services you want to expose (SSH, web server, etc.).

Troubleshooting Network Issues

When your network isn't working, follow this systematic approach. Each step narrows down the problem.
Step 1: Check Your Interface

ip -br a

Is your interface UP with an IP address? If no IP, you're not connected to any network. Check your cable, Wi-Fi, or NetworkManager:

nmcli device status

Step 2: Test Local Gateway (Router)

ping -c 4 192.168.1.1

(Replace with your actual gateway from ip route.) If this fails, your computer can't reach your router — cable, Wi-Fi, or local network issue.
Step 3: Test External IP (No DNS)

ping -c 4 8.8.8.8

If Step 2 worked but this fails, your router can't reach the internet — ISP problem, router configuration, or internet outage.
Step 4: Test DNS Resolution

ping -c 4 google.com

If Step 3 worked but this fails, DNS is broken. Try a different DNS server:

nmcli connection modify "Wired connection 1" ipv4.dns "8.8.8.8 1.1.1.1"
nmcli connection up "Wired connection 1"

Step 5: Check for DNS Specifically

host google.com
dig google.com

If these fail or return no results, DNS is definitely the problem. Check /etc/resolv.conf:

cat /etc/resolv.conf

On Ubuntu 26.04, this file is managed by systemd-resolved and should contain nameserver 127.0.0.53. If it's empty or wrong:

sudo systemctl restart systemd-resolved

Step 6: Check Listening Ports

sudo ss -tlnp

Verify that the services you expect are listening. If a web server isn't serving pages, check if it's actually running and listening on port 80 or 443.
Step 7: Check NetworkManager

systemctl status NetworkManager

If NetworkManager isn't running or has failed, that explains connection problems. Restart it:

sudo systemctl restart NetworkManager

Step 8: Check Network Logs

journalctl -u NetworkManager -b | tail -30

Shows NetworkManager's log entries from this boot. Look for error messages, failed DHCP leases, or authentication failures.
Step 9: Trace the Route

traceroute google.com

(sudo apt install traceroute if not installed.)

Shows every hop (router) between you and the destination. If the trace stops at a certain hop, that's where the problem lies:

traceroute to google.com (142.250.190.46), 30 hops max
 1  192.168.1.1      1.2 ms   (your router)
 2  10.0.0.1         8.3 ms   (ISP gateway)
 3  172.16.50.1     12.1 ms   (ISP backbone)
 4  *  *  *                  (timeout — problem here)

Quick Troubleshooting Summary
Symptom	Likely Cause	First Check
No internet, no local network	Interface down or no IP	ip -br a, nmcli device status
Can reach router, not internet	ISP outage or router issue	ping 8.8.8.8
Can reach IP, not hostname	DNS failure	host google.com, /etc/resolv.conf
Slow loading, intermittent drops	Wireless interference or congestion	ping -c 20 for packet loss
Specific service unreachable	Firewall blocking port	sudo ufw status, sudo ss -tlnp
SSH connection refused	SSH not running or blocked	systemctl status ssh, sudo ufw status
Websites load but wrong content	DNS poisoning or proxy	dig google.com, check VPN/proxy
Networking Cheat Sheet
Task	Command
Show IP addresses	ip -br a
Show all interface details	ip a
Show interface names/states	ip link
Show routing table	ip route
Network status overview	nmcli general status
List network devices	nmcli device status
List saved connections	nmcli connection show
Scan Wi-Fi networks	nmcli device wifi list
Connect to Wi-Fi	nmcli device wifi connect "SSID" password "pass"
Disconnect Wi-Fi	nmcli device disconnect wlp2s0
Wi-Fi on/off	nmcli radio wifi on / off
Text-mode network manager	nmtui
Test connectivity	ping -c 4 hostname
Show hostname	hostname
Full host info	hostnamectl
Change hostname	sudo hostnamectl set-hostname name
DNS status	resolvectl status
DNS lookup (detailed)	dig hostname
DNS lookup (simple)	host hostname
Reverse DNS	host IP_ADDRESS
List listening ports	ss -tln
List all connections	ss -tna
List with processes	sudo ss -tlnp
Show established connections	ss -tn state established
Download file (curl)	curl -O https://example.com/file.zip
Download file (wget)	wget https://example.com/file.zip
Resume download (curl)	curl -C - -O URL
Resume download (wget)	wget -c URL
Copy file to remote	scp file user@host:/path/
Copy file from remote	scp user@host:/path/file .
Copy directory recursively	scp -r dir/ user@host:/path/
Sync directory	rsync -av dir/ user@host:/path/
Dry-run sync	rsync -avn dir/ user@host:/path/
Mirror with deletes	rsync -av --delete dir/ user@host:/path/
Firewall status	sudo ufw status
Enable firewall	sudo ufw enable
Allow port	sudo ufw allow 80/tcp
Deny port	sudo ufw deny 3306
Delete rule	sudo ufw delete allow 80/tcp
Disable firewall	sudo ufw disable
Trace network path	traceroute hostname
Network logs	journalctl -u NetworkManager -b
What's Next

You now understand how Linux networking works from the ground up: IP addresses, interfaces, DNS, ports, and routing. You can configure connections with nmcli and nmtui, test connectivity with ping, diagnose DNS with dig and host, inspect open ports with ss, download files with curl and wget, transfer data between machines with scp and rsync, manage your firewall with ufw, and follow a systematic troubleshooting process when things break.

In Chapter 22, we'll cover hardware and drivers — how Ubuntu detects and manages hardware components, how to install proprietary drivers (especially for NVIDIA GPUs), and how to troubleshoot device issues from the terminal.
Try It Yourself

    Explore your interfaces. Run ip -br a. Identify your Ethernet and/or Wi-Fi interface, your IP address, and the loopback interface.

    Check your routing table. Run ip route. Identify your default gateway — that's your router's IP address.

    Test connectivity layers. Follow the troubleshooting sequence:
        ping -c 4 192.168.1.1 (your router — from the route table)
        ping -c 4 8.8.8.8 (Google's DNS — tests internet)
        ping -c 4 google.com (tests DNS)

    Check DNS. Run host google.com and dig google.com. Compare the outputs. Note the IP address that google.com resolves to.

    List your connections. Run nmcli device status and nmcli connection show. See what NetworkManager knows about your network.

    Check open ports. Run ss -tln. Identify which services are listening on your machine. Run sudo ss -tlnp to see which processes own those ports.

    Download a file. Try both methods:

    curl -O https://raw.githubusercontent.com/torvalds/linux/master/LICENSES/preferred/MIT
    wget https://raw.githubusercontent.com/torvalds/linux/master/LICENSES/preferred/MIT

    Compare how each tool reports progress.

    Check your firewall. Run sudo ufw status. If it's inactive, consider enabling it: sudo ufw enable (make sure to sudo ufw allow ssh first if you're on SSH).

    Try a traceroute. Run traceroute google.com (install with sudo apt install traceroute if needed). See how many hops your packets take to reach Google.

    ✅ Chapter 21 Complete You understand IP addresses, network interfaces, DNS, and ports. You can configure network connections with nmcli and nmtui, test connectivity with ping, diagnose DNS with dig and host, inspect open ports with ss, download files with curl and wget, transfer data with scp and rsync, manage your firewall with ufw, and systematically troubleshoot network problems. In Chapter 22, we'll cover hardware detection and driver management.

