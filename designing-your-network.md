# Designing Your Network

**By:** Greg Diny  
**Date:** 10/15/25

---

Designing your home network is the first step on your journey toward cybersecurity and privacy. Cybersecurity is a layered process, and network design forms the foundation. By laying everything out ahead of time, you can easily identify vulnerabilities, pain points, opportunities for efficiency, and how much cabling you'll need.

Good design also makes implementation smoother. When you know what you want, why you want it, and how it fits into the bigger picture, configuring software and applying security tools becomes a matter of execution rather than guesswork. Without a plan, you end up with patchwork fixes, weak security, and unpredictable performance. With one, you gain scalability, visibility, and the ability to prevent problems before they happen.

This guide focuses on designing a home network and lab setup with privacy and manageability in mind.

---

## Ask Yourself a Few Questions

Before you start buying hardware or installing software, answer a few key questions to streamline your process:

- **What's the goal?** Are you upgrading your ISP's router for better performance, or are you building a full-blown enterprise-style lab at home?
- **What software will you need?** The software stack determines the hardware you'll need.
- **How many devices will connect?** This determines your Ethernet port count. I have a desktop, access point, file server, and a future LLM server — so I need at least six ports.
- **What hardware supports the stack?** Since pfBlockerNG and Suricata both inspect packets, your pfSense box needs a capable CPU and plenty of RAM. I run four wired devices (desktop, AP, file server, and LLM server) and several wireless clients including Rokus, a Chromecast, and a laptop — so my router needs to keep up with that load.
- **What's the network's physical footprint?** The size of your home determines Ethernet cable length and Wi-Fi coverage requirements.

---

## Draw a Map

Once you've answered those questions, sketch out a network map showing how everything connects. This doesn't need to be fancy — MS Paint, LibreOffice Draw, or even pen and paper works fine.

Label each device with its VLAN, subnet, and static IP address. This makes configuration easier later. The goal is to visualize how data flows between devices and where segmentation occurs, not to document software just yet.

I used LibreOffice Draw to map my own network. The final diagram labels every device by name, VLAN ID, and IP address, clearly showing how traffic moves across segments.

---

## From Blueprint to Build – Software and Hardware

Next, decide which software runs on which devices, and confirm that your hardware can handle it. The complexity of your network determines how powerful your systems need to be.

Here's a quick breakdown of my own setup:

- 📡 **Modem:** Motorola – Handles ISP connection
- 🧱 **Firewall:** HP EliteDesk 800 G2 ($60) – Running pfSense with pfBlockerNG (IP/DNS filtering) and Suricata (IDS/IPS)
- 🔀 **Switch:** TP-Link SG108E ($20) – Managed gigabit switch with VLAN and QoS support
- 📶 **Wireless Access Point:** TP-Link Omada AC-1350 ($50) – Dual-band, VLAN-capable access point for Home / Guest / IoT networks
- 🖥️ **Network Utility Server:** HP EliteDesk 800 G3 Mini ($75) – Ubuntu Desktop running: Wazuh (SIEM), Shuffle (SOAR), ClamAV (Antivirus), Netdata (Monitoring)
- 💾 **File Server:** HP EliteDesk 800 G3 ($50) – Ubuntu Server hosting Apache2, Samba, MariaDB

Because software like Suricata inspects every packet, CPU power matters. Both of my EliteDesks use i5-6500 CPUs — plenty for this workload.

---

## The Software

### Firewall – pfSense

pfSense is an open-source firewall and router platform that delivers enterprise-grade network security and traffic management. Acting as the central gatekeeper, it manages VLANs, VPNs, firewall rules, and bandwidth monitoring. With add-ons like pfBlockerNG for threat intelligence, Suricata for intrusion detection, and Squid for caching, pfSense is a flexible, all-in-one solution for both home labs and business environments.

### IP Blocking & DNS Filter – pfBlockerNG

pfBlockerNG enhances pfSense by combining IP reputation blocking and DNS-based filtering. It uses curated threat feeds to block known malicious IPs, botnets, and regions, while its DNSBL feature prevents domains tied to ads, trackers, or malware from resolving. This dual-layer protection turns pfSense into a DNS blackhole, automatically shielding every device on your network.

### Intrusion Detection & Prevention – Suricata

Suricata is an open-source IDS/IPS that monitors network traffic in real time to detect and block malicious activity. Using deep packet inspection and rule sets like Emerging Threats, it identifies exploits, scans, and malware before they reach your devices. In IDS mode, it alerts you to suspicious traffic; in IPS mode, it actively blocks it. Suricata integrates with Wazuh and Elastic Stack for log analysis and visualization, providing a powerful, automated layer of defense.

### Network Antivirus – ClamAV

ClamAV is an open-source antivirus engine that scans files, emails, and network traffic for malware using frequently updated signature databases. It integrates with mail servers and tools like Wazuh or pfSense to provide an extra layer of protection, catching threats that slip past perimeter defenses.

## Eventual Additions

### Network Monitoring – Netdata

Netdata is a lightweight, real-time monitoring tool that visualizes system and network performance through an interactive dashboard. It tracks CPU, memory, disk, and network usage with second-by-second updates, integrates with Wazuh or Prometheus, and helps identify bottlenecks or anomalies instantly.

### SIEM – Wazuh

Wazuh is an open-source SIEM platform that centralizes security monitoring across endpoints, servers, and network devices. It detects intrusions, malware, and policy violations in real time, alerting administrators through a unified dashboard. Integrated with Elastic Stack and OSSEC, Wazuh provides scalable, automated threat detection and compliance visibility for both home and enterprise setups.

### SOAR – Shuffle

Shuffle is an open-source SOAR tool that automates and links security workflows between platforms like Wazuh, pfSense, and email scanners. With its drag-and-drop interface, you can create workflows that trigger automatic responses — such as alerting, IP blocking, or ticket creation — reducing manual work and speeding up incident response.


### How It All Fits Together

Together, these tools form a layered security ecosystem that protects, monitors, and automates your entire network. pfSense acts as the central firewall and router, enforcing rules and managing traffic. pfBlockerNG filters out malicious IPs and domains, while Suricata inspects packets for threats in real time. Wazuh collects all activity and alerts into a single dashboard for correlation and analysis, and Shuffle automates responses like blocking or notifications. ClamAV adds host-level malware protection, and Netdata continuously monitors system performance and health. Working in harmony, they create a unified, self-sustaining defense system that delivers enterprise-grade visibility, automation, and reliability for any home lab or business network.

---

## The Hardware

### Ethernet Cable

Your cabling is the backbone of your network — don't cut corners. Cat6 supports 1 Gbps up to full length and 10 Gbps on shorter runs, while Cat6a or Cat7 offer better shielding and full 10 Gbps performance. Always use solid copper (not CCA) and avoid cheap flat cords that fail easily or pick up interference. Run quality cable once, and you'll never have to redo it.

### Modem

The modem connects your network to your ISP, translating the external signal into Ethernet. Choose a model that matches your connection type and speed — DOCSIS for cable or an ONT for fiber. It's your network's handshake with the internet, so reliability is key.

### Firewall

After the modem, the firewall is where control begins. Running pfSense or OPNsense gives you enterprise-grade tools for managing VPNs, VLANs, and rules while monitoring traffic in real time. I run pfSense on an HP EliteDesk with Intel NICs — rock solid for under $100. Avoid USB or Realtek NICs, as they cause packet loss. Fanless boxes like Protectli or Qotom are quiet and reliable but cost more upfront. My first unit, a Dell Wyse 5070, overheated due to poor cooling — a good reminder that strong hardware and airflow matter.

### Network Switch

The switch branches your network, connecting desktops, servers, and access points. Look for VLAN support, QoS, and enough ports for future growth. My TP-Link SG108E is affordable, VLAN-capable, and managed via a simple web interface. If you power access points or cameras, a PoE model helps eliminate power bricks. A solid switch keeps your internal traffic fast, clean, and expandable.

### Wireless Access Point

Your access point manages Wi-Fi coverage and stability. My TP-Link AC1350 broadcasts 2.4 GHz and 5 GHz networks across multiple SSIDs — Home, Guest, and IoT — each tied to a VLAN. Placement matters: keep it central and elevated for the best coverage. With PoE support, it runs on a single cable for both power and data — simple, tidy, and reliable.

### Advanced Setup – Local Servers

Running local servers adds privacy and capability to your network. I use a Dell Wyse 5070 with Ubuntu Server, Apache, Samba, and MariaDB for a local file share, soon to host Nextcloud over VPN. Samba alone can replace cloud storage: install it, share a folder, and connect using a local account.

For AI experimentation, I'm building a local LLM server using an AM4 system with a GTX 1050 Ti (4 GB VRAM) to run Phi-3 via Ollama — a great entry point into self-hosted AI computing.

---

## Final Thoughts

A well-designed network doesn't just work — it scales, adapts, and defends itself. By combining robust hardware with smart, open-source software, you can build a home setup that rivals small enterprise networks in both security and performance. Keep in mind that simply having these devices and software running on your network does not automatically secure it — you need to configure each piece of software and device.
