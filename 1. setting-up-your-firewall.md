# Installing a Firewall

**By:** Greg Diny  
**Date:** 10/22/25

---

Installing a firewall on your home network is one of the most impactful steps you can take toward cybersecurity and privacy. A firewall is hardware (or software) that monitors incoming and outgoing network traffic based on a defined set of security rules, acting as the first line of defense between your trusted internal network and the untrusted internet. Common examples include hardware firewalls like Cisco appliances and software firewalls like Windows Defender Firewall.

This guide focuses on **pfSense** — an open-source firewall platform and what I run on my own network. I'll cover required materials, creating a bootable USB, installation, basic configuration, firewall rules, and setting up your first VLAN. I'll also share the hard lessons learned from my first few iterations so you can avoid the same mistakes.

---

## Picking a Firewall

The two most popular open-source options are **pfSense** and **OPNsense**. Both are community-driven, free, and built on FreeBSD. I've only used pfSense, so that's what this guide covers — but OPNsense is a legitimate alternative worth researching.

What makes pfSense stand out is its package ecosystem. Through the built-in package manager, you can extend it with tools like **Suricata** (IDS/IPS) and **pfBlockerNG** (IP/DNS filtering), transforming a basic router into a full-featured network security platform.

---

## Gather Your Materials

You'll need the following before getting started:

- Blank USB drive (4GB or larger)
- [pfSense ISO](https://www.pfsense.org/download/)
- [Rufus](https://rufus.ie/en/) (Windows utility for creating bootable drives)
- Firewall hardware — an HP EliteDesk or Protectli Vault are solid choices. Any machine with **two NICs** will work.

### Notes on Choosing Hardware

**Use an Intel NIC.** Realtek NICs are commonly problematic with pfSense — I've had pfSense fail to detect a Realtek interface entirely. If your hardware has a Realtek NIC, plan on adding an Intel PCIe NIC.

**Avoid USB NICs.** They might work in a pinch, but they're unreliable for a primary WAN or LAN connection and will drop packets under load.

**Make sure it can run 24/7.** Your firewall needs to stay on all the time. If the hardware is fanless, verify it's rated for continuous operation. My first firewall was a Dell Wyse 5070 — a fanless mini PC — and I came home one day to find my internet down, the unit hot to the touch, and it wouldn't power back on. It almost certainly overheated.

---

## Creating a Bootable Flash Drive

We'll use Rufus to flash the pfSense ISO onto the USB drive.

1. Download and install Rufus
2. Open Rufus
3. Under **Device**, select your USB drive
4. Under **Boot selection**, select the pfSense `.iso` image
5. Click **Start** and confirm any prompts
6. When finished, eject the flash drive

<img width="512" height="636" alt="image" src="https://github.com/user-attachments/assets/52a5650b-907e-412f-a195-696d19de9d74" />

The USB drive is reusable — you can use it to install pfSense on as many machines as you want.

---

## Installing pfSense

We'll boot from the USB and install pfSense directly onto the machine's SSD.

1. Insert the USB drive into the firewall hardware
2. Power on and enter the **Boot Menu** (the key varies by machine — common ones are F12, F8, or Esc)
3. Select the USB drive from the boot menu
4. pfSense will boot into the installer — follow the prompts

> **Note:** The boot menu key and BIOS behavior vary widely between machines. If you get stuck, a quick Google search or an LLM with your hardware model will get you sorted fast.

### Installer Walkthrough

Once the installer loads:

1. **Installer Menu** → Select *Install pfSense*
2. **Partitioning** → Choose *Auto UFS*
3. The installer copies files to disk and prompts you to reboot — remove the USB when asked
4. After rebooting, pfSense boots to its console menu and asks you to assign interfaces

**Interface assignment** (names vary by hardware):

- WAN → `em0`
- LAN → `em1`

pfSense will then prompt you to set a username and password for the web GUI, and will assign the default LAN IP: **192.168.1.1**. That's the address you'll use to access everything going forward.

---

## First Time in the WebGUI

Open a browser on a device connected to the LAN side and navigate to `192.168.1.1`. You'll see an SSL warning — go ahead and proceed past it. Log in with the credentials you set during installation.

pfSense will walk you through a **Setup Wizard**. Work through it in order:

1. Assign a **hostname**
2. Set your **DNS resolver**
3. Configure **WAN settings**:
   - Enable *Block Bogon Networks*
   - Enable *Block Private Networks*
4. **Verify SSH is disabled**: System → Advanced → Admin Access (it's off by default — leave it that way)
5. **Enable HTTPS** for the web GUI

### Default Firewall Rules

After the wizard completes, pfSense will have two rules in place:

- **LAN** — Anti-lockout rule (prevents you from locking yourself out of the GUI)
- **WAN** — Blocks private networks and bogon networks

At this point you have a working, flat network — WAN into LAN with no VLAN segmentation. That's a solid starting point.

---

## Setting Up Aliases

An **Alias** in pfSense is a named shortcut for one or more IP addresses, networks, hostnames, or ports. Instead of typing individual values into every firewall rule, you reference the alias. This makes rules cleaner and easier to maintain.

> Aliases are different from static IP assignments in your DHCP server. Static IPs are tied to device MAC addresses so a device always gets the same IP — we'll set those up after configuring VLANs.

**Create an RFC1918 alias now** — you'll use it in every firewall rule going forward:

- Firewall → Aliases → Add
- Name: `RFC1918`
- Type: Network
- Add the following networks:
  - `10.0.0.0/8`
  - `172.16.0.0/12`
  - `192.168.0.0/16`
- Save and Apply

---

## Basic Firewall Rules

### WAN Rules

| Rule | Purpose |
|------|---------|
| Block RFC1918 on WAN | Prevents spoofed private IP traffic from the internet |
| Block Bogon Networks | Blocks reserved/unallocated IP ranges |
| Default Deny Inbound | Implicit deny-all — only explicitly allowed traffic gets through |

The first two are typically configured automatically by the Setup Wizard. Verify they're in place under Firewall → Rules → WAN.

### LAN Rules

| Rule | Purpose |
|------|---------|
| Anti-lockout rule | Keeps you from losing GUI access (default, don't remove it) |
| Allow LAN → Any | Permits all outbound traffic from LAN |

<img width="1005" height="108" alt="image" src="https://github.com/user-attachments/assets/37ddca4c-fef5-424a-af1c-76affc362114" />

<img width="1014" height="759" alt="image" src="https://github.com/user-attachments/assets/a4e8be49-875a-4458-82a1-3e824c9b695c" />


---

## Setting Up a VLAN

VLANs let you segment your network and apply different firewall rules to different groups of devices. In this example, we'll create a VLAN for the desktop PC — useful for experimentation without taking down your whole network.

### Step 1 — Create the VLAN

Interfaces → Assignments → VLANs → Add

| Field | Value |
|-------|-------|
| Parent Interface | Your LAN NIC (e.g., `igb1`) |
| VLAN Tag | `10` |
| Description | `VLAN10_Desktop` |

Save.

### Step 2 — Assign the Interface

Interfaces → Assignments → click **+ Add** in the Available Network Ports dropdown

- Select the VLAN you just created (e.g., VLAN 10 on igb1)
- Click Add, then click the new interface name (it'll show as OPT1 or similar)
- Enable the interface
- Set the following:

| Field | Value |
|-------|-------|
| Description | `Desktop` |
| IPv4 Configuration Type | Static IPv4 |
| IPv4 Address | `192.168.10.1 /24` |
| IPv6 | None |

Save → Apply Changes.

### Step 3 — Enable DHCP

Services → DHCP Server → Desktop tab

| Field | Value |
|-------|-------|
| Enable DHCP | ✓ |
| Range | `192.168.10.100 – 192.168.10.200` |
| DNS Servers | Leave blank (pfSense hands out `192.168.10.1` automatically) |

Save. Optionally add static mappings here for devices that need a pinned IP.

### Step 4 — Update DNS Resolver and NTP

- **Services → DNS Resolver** — make sure *Network Interfaces* includes Desktop (or is set to All). Save/Apply.
- **Services → NTP** — set *Interface(s)* to All or include Desktop. Save.

### Step 5 — Firewall Rules

Go to Firewall → Rules → Desktop and add these rules **in order, top to bottom**:

**Rule 1 — Block lateral traffic (no access to other VLANs)**

| Field | Value |
|-------|-------|
| Action | Block |
| Interface | Desktop |
| Protocol | Any |
| Source | Desktop net |
| Destination | RFC1918 (Alias) |
| Description | Block Desktop → RFC1918 (no lateral movement) |

**Rule 2 — Allow DNS to pfSense**

| Field | Value |
|-------|-------|
| Action | Pass |
| Protocol | TCP/UDP |
| Source | Desktop net |
| Destination | This Firewall |
| Destination Port | 53 (DNS) |
| Description | Desktop → pfSense DNS |

Duplicate this rule with port 123 (UDP) if you want pfSense to serve NTP as well. Add a Pass ICMP rule to This Firewall if you want to ping the gateway.

**Rule 3 — Allow internet access**

| Field | Value |
|-------|-------|
| Action | Pass |
| Protocol | Any |
| Source | Desktop net |
| Destination | not RFC1918 |
| Description | Desktop → Internet |

Click **Save and Apply Changes**.

> **NAT note:** If you're using Automatic Outbound NAT (the default), internet access for this VLAN will work without any extra steps. If you've switched to Manual Outbound NAT, add a rule under Firewall → NAT → Outbound to translate `192.168.10.0/24` out your WAN interface.

### Step 6 — Configure Your Switch

After configuring pfSense, you'll need to log into your managed switch and tag the appropriate port with VLAN ID 10. The pfSense-facing port should remain **untagged**. Since switch interfaces vary, consult your switch's documentation — the general concept is the same across managed switches.

---

## Some Irony

Now that you've got the theory down, here's what actually happened when I set all this up.

**First attempt:** The Dell Wyse 5070 I started with either couldn't handle the load or had a defect. I came home from work to find it shut down and hot to the touch — sitting in an open cubby with no airflow. It never powered back on.

**Second attempt:** I bought another Wyse 5070 and added fans, which solved the heat problem. But the USB NIC I was using for the second interface was a constant headache — intermittent drops, unreliable under load. USB NICs are fine for a one-off connection but are not suitable as a permanent WAN or LAN interface. Lesson learned the hard way.

**What I'm running now:** An HP EliteDesk with an Intel PCIe NIC. Quad-core CPU, 16GB RAM — handles pfSense, Suricata, and pfBlockerNG without breaking a sweat. The old Dell Wyse got repurposed as a file server running Samba, which it handles just fine.

**Future upgrade:** A Protectli Vault would be the ideal dedicated firewall appliance — purpose-built, fanless, Intel NICs built in. But at around $400, it's hard to justify when an $80 repurposed desktop does the same job. Maybe someday.

---

## Conclusion

That's it — a basic but solid pfSense installation. When you're done you'll have:

- A reusable USB drive with pfSense on it
- Your own firewall, independent of your ISP's hardware
- Foundational WAN and LAN firewall rules
- A VLAN to isolate your desktop from the rest of the network

From here, the natural next steps are installing **pfBlockerNG** for DNS and IP filtering and **Suricata** for intrusion detection and prevention. Both are available directly from the pfSense package manager and substantially level up your security posture. The next article in this series covers VLAN segmentation in more depth.
