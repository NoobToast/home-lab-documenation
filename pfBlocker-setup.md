# pfBlockerNG Setup and Configuration Guide

**Author:** Greg Diny

A guide to setting up pfBlockerNG for IP blocking and DNS filtering on pfSense. This guide assumes you have already completed the [pfSense installation](installing-pfsense.md) and [VLAN setup](vlan-setup-pfsense.md) guides.

At the time of updating:
Live for:

<img width="410" height="44" alt="Screenshot from 2026-02-28 15-58-30" src="https://github.com/user-attachments/assets/e4a593fb-7505-46d8-8135-6d6151c6c8a5" />

pfBlockerNG Activity:

<img width="550" height="309" alt="Screenshot from 2026-02-28 15-59-33" src="https://github.com/user-attachments/assets/07b5a8f1-bef3-4e25-abc7-5b42b098bde1" />

---

## What is pfBlockerNG?

pfBlockerNG is a pfSense package that extends your firewall into a full network-level content filter. It operates at two layers:

- **IP reputation blocking** — Blocks known malicious IPs using curated threat feeds, both inbound and outbound
- **DNS-based filtering (DNSBL)** — Blocks ads, trackers, and malware domains before they ever resolve, protecting every device on your network without needing per-device software
- **GeoIP blocking** — Blocks traffic from specific countries or regions
- **ASN blocking** — Blocks entire ISP or organization networks

Because DNSBL operates at the DNS level, it protects every device on your network automatically — phones, smart TVs, IoT devices — without installing anything on them.

---

## Prerequisites

- pfSense 2.6+ installed and configured
- VLANs configured (if applicable) — see [VLAN Setup Guide](vlan-setup-pfsense.md)
- Internet connectivity
- Basic familiarity with pfSense navigation

---

## Installation

### 1. Install the Package

1. Navigate to **System → Package Manager → Available Packages**
2. Search for `pfblockerng-devel` (the development version has more features and is what most users run)
3. Click **Install** and wait for it to complete

### 2. Initial Configuration

1. Navigate to **Firewall → pfBlockerNG → General**
2. Fill in the following:

   | Setting | Value |
   |---------|-------|
   | Enable pfBlockerNG | ✓ |
   | Daily CRON | `00:00` (midnight — runs feed updates and rule rebuilds daily) |
   | Weekly CRON | Sunday at `01:00` (runs heavier maintenance tasks) |
   | Keep Settings | `52 weeks` |

3. Click **Save**

### 3. Configure DNSBL

pfBlockerNG needs a dedicated virtual IP to serve its block page. Without this, blocked DNS queries will either fail silently or return errors instead of a block page.

Navigate to **Firewall → pfBlockerNG → DNSBL** and configure the following sections:

**DNSBL**

| Setting | Value |
|---------|-------|
| Enable DNSBL | ✓ |
| DNSBL Mode | Unbound mode |
| Wildcard Blocking (TLD) | Leave unchecked (advanced — read the infoblock before enabling) |
| Resolver Live Sync | Leave unchecked unless you need hourly updates without resolver reloads |

**DNSBL Webserver Configuration**

| Setting | Value |
|---------|-------|
| Virtual IP Address | `10.10.10.1` (must be RFC1918 compliant and not already in use on your network) |
| DNSBL VIP Type | IP Alias (default) |
| Web Server Interface | Localhost (default — leave this as-is) |

**DNSBL Configuration**

| Setting | Value |
|---------|-------|
| Permit Firewall Rules | ✓ — important for multi-VLAN setups (see note below) |
| Global Logging/Blocking Mode | No Global mode (default) |
| Blocked Webpage | `dnsbl_default.php` (default) |
| Resolver Cache | ✓ |

> **Important — Permit Firewall Rules:** If you have multiple VLANs, check this box and select all your interfaces from the list (LAN, DESKTOP_VLAN10, SERVERS_VLAN30, etc.). This creates floating firewall rules that allow each VLAN to reach the DNSBL webserver and display the block page correctly. Without it, devices on VLANs other than LAN will get a connection error instead of a block page when a domain is blocked. Since your DHCP server already hands out pfSense as the DNS server for each VLAN, all devices route DNS through pfBlockerNG automatically — this setting just ensures the block page renders correctly across all segments.

Click **Save**.

---

## IP Blocking Configuration

Navigate to **Firewall → pfBlockerNG → IP**

### IP Configuration

| Setting | Value | Notes |
|---------|-------|-------|
| De-Duplication | ✓ | Removes duplicate IPs across lists — leave enabled |
| CIDR Aggregation | ✓ | Merges adjacent IP ranges into larger blocks for efficiency — leave enabled |
| Suppression | ✓ | Prevents RFC1918 and loopback addresses from being accidentally blocked — leave enabled |
| Force Global IP Logging | Leave unchecked | Only needed if you want to override per-feed logging settings globally |
| Placeholder IP Address | `127.1.7.7` | Used as a placeholder for blocked IPs — leave as default unless it conflicts with something on your network |

### ASN Configuration

ASN (Autonomous System Number) blocking lets you block entire networks owned by a specific organization or ISP. This requires a free IPinfo account.

| Setting | Value | Notes |
|---------|-------|-------|
| ASN Reporting | Disabled | Enable if you want ASN data appended to block/reject log entries |
| ASN IPinfo Token | (leave blank) | Register at IPinfo.io for a free token if you want ASN functionality |

> **Note:** If you use Suricata, check for IPinfo blocked events before enabling ASN features — they can conflict.

### MaxMind GeoIP Configuration

GeoIP blocking requires a free MaxMind account. Without credentials entered here, the GeoIP tab will not function.

| Setting | Value | Notes |
|---------|-------|-------|
| MaxMind Account ID | (your account ID) | Register for free at MaxMind.com — use GeoLite2 update version 3.1.1 or newer |
| MaxMind License Key | (your license key) | Generated in your MaxMind account dashboard |
| MaxMind Localized Language | English | Controls locale data for country names |
| MaxMind CSV Updates | Leave unchecked | Only check this if you want to disable the monthly CSV database update |

> GeoIP is optional — skip this section if you don't plan to use country-based blocking. See the GeoIP section below for caveats before enabling it.

### IP Interface/Rules Configuration

This is where you define which interfaces pfBlockerNG applies IP blocking rules to and how.

| Setting | Value | Notes |
|---------|-------|-------|
| Inbound Firewall Rules | Select all interfaces (WAN, LAN, SERVERS_VLAN30, DESKTOP_VLAN10, WIFI_VLAN20, IOT_VLAN40) | Action: **Block** — drops inbound traffic from malicious IPs before it reaches your network |
| Outbound Firewall Rules | Select all interfaces | Action: **Reject** — prevents your devices from reaching known malicious IPs outbound |
| Floating Rules | Leave unchecked | pfBlockerNG manages its own floating rules automatically |
| Firewall Auto Rule Order | `pfB_Pass/Match/Block/Reject \| All other Rules` (default) | Controls where pfBlockerNG rules sit relative to your manual rules — default is correct |
| Firewall Auto Rule Suffix | `auto rule` (default) | Label applied to auto-generated rules — leave as-is |
| Kill States | Leave unchecked | When enabled, clears active firewall states for newly blocked IPs after a cron run or force update — useful but aggressive; enable once you're confident in your feed selection |

> **Inbound vs. Outbound actions:** Block (inbound) silently drops packets. Reject (outbound) sends a reset back to the source, which is more appropriate for outbound traffic from your own devices since it produces a faster failure rather than a timeout.

### 2. Add IP Threat Feeds

Navigate to **Firewall → pfBlockerNG → Feeds**

The following feeds are a solid starting point. Add them one at a time, test after each batch, and expand from there.

| Feed | URL | Description |
|------|-----|-------------|
| Emerging Threats | `https://rules.emergingthreats.net/blockrules/compromised-ips.txt` | Known compromised hosts |
| Spamhaus DROP | `https://www.spamhaus.org/drop/drop.txt` | Hijacked/leased networks used for attacks |
| Spamhaus EDROP | `https://www.spamhaus.org/drop/edrop.txt` | Extended version of DROP |
| Abuse.ch URLhaus | `https://urlhaus.abuse.ch/downloads/text/` | Active malware distribution IPs |
| FireHOL Level 1 | `https://raw.githubusercontent.com/firehol/blocklist-ipsets/master/firehol_level1.netset` | Curated high-confidence malicious IPs |

**To add each feed:**

1. Click **Add**
2. Fill in:

   | Field | Value |
   |-------|-------|
   | Format | IPv4 |
   | Action | Deny Both |
   | Frequency | Daily |
   | URL | (from table above) |
   | Description | (feed name) |

3. Click **Save**

### 3. GeoIP Blocking (Optional)

Navigate to **Firewall → pfBlockerNG → IP → GeoIP**

GeoIP blocking lets you block traffic from entire countries or regions. It can be effective but comes with significant caveats for home use.

> **Warning:** Major cloud providers (AWS, Google, Cloudflare, Azure) operate infrastructure in virtually every country. Blocking a country often means blocking services hosted there, not just users from that country. For most home lab setups, GeoIP blocking causes more false positives than it's worth. If you do enable it, start with **Deny Inbound only** and monitor logs closely before switching to Deny Both.

If you choose to enable it:
- Select target countries
- **Action:** Deny Inbound
- **Logging:** Enable

---

## DNS Filtering (DNSBL) Configuration

### 1. Add DNSBL Feeds

Navigate to **Firewall → pfBlockerNG → DNSBL → DNSBL Groups**

The following feeds cover ads, trackers, and malware domains:

| Feed | URL | Description |
|------|-----|-------------|
| EasyList | `https://easylist.to/easylist/easylist.txt` | Standard ad blocking list |
| Steven Black Unified | `https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts` | Ads, malware, and trackers combined |
| Malware Domain List | `https://www.malwaredomainlist.com/hostslist/hosts.txt` | Known malware domains |
| hagezi Pro | `https://raw.githubusercontent.com/hagezi/dns-blocklists/main/domains/pro.txt` | Comprehensive multi-purpose blocklist |

**To add each feed:**

1. Click **Add**
2. Fill in:

   | Field | Value |
   |-------|-------|
   | Group Name | (descriptive name, e.g., `Ads` or `Malware`) |
   | Action | Unbound |
   | Logging | Enable |

3. Add the feed URL under **Feed Sources**
4. Click **Save**

### 2. Verify DNS Resolver Settings

Navigate to **Services → DNS Resolver → General Settings** and confirm:

| Setting | Value |
|---------|-------|
| Enable DNS Resolver | ✓ |
| DNSSEC | ✓ |
| Listen Interfaces | All (or every VLAN interface) |
| Outgoing Interfaces | WAN |

If you followed the VLAN setup guide, your DHCP server is already handing out `192.168.x.1` (pfSense) as the DNS server for each interface. This means every device on every VLAN will automatically route DNS through pfBlockerNG — no per-device configuration needed.

---

## Step 3: Force Initial Update

After saving your feed configuration, you need to force pfBlockerNG to pull all feeds for the first time. It won't do this automatically until the next scheduled CRON run.

1. Navigate to **Firewall → pfBlockerNG → Update**
2. Select **Reload All**
3. Click **Run**
4. Watch the output log — it should show each feed being downloaded and processed
5. When complete, navigate to **Firewall → Rules → Floating** and verify pfBlockerNG rules are present

This step is required after initial setup and after adding new feeds.

---

## Firewall Rule Verification

pfBlockerNG automatically creates floating firewall rules for each VLAN its monitoring. Verify they're in place:

Navigate to **Firewall → Rules → VLAN**

The auto-generated rules should be:
- At the **top** of the rule list (processed before your manual rules)
- **Quick** enabled (stops processing once matched)
- **Interface:** Any

If you don't see these rules, go back to **Firewall → pfBlockerNG → Update** and run **Reload All** again.

---

## Testing and Verification

### Test IP Blocking

```bash
# Attempt to reach a known blocked IP — should timeout or be rejected
ping <blocked-ip>
```

### Test DNSBL

Visit `doubleclick.net` in a browser. It should either:
- Fail to resolve entirely
- Redirect to the pfBlockerNG block page (at your virtual IP)

### Check Logs and Reports

Navigate to **Firewall → pfBlockerNG → Reports**

| Tab | What to look for |
|-----|-----------------|
| Alerts | Blocked IP connections |
| DNSBL | Blocked DNS queries |
| Top Sources | Devices generating the most blocked requests |

Review these regularly, especially in the first week, to catch false positives before they become a problem.

---

## Troubleshooting

**Legitimate sites being blocked**
1. Navigate to **Firewall → pfBlockerNG → Reports → DNSBL**
2. Find the blocked domain
3. Click **Suppress** to whitelist it, or add it manually under **Firewall → pfBlockerNG → DNSBL → Whitelist**

**pfBlockerNG not blocking anything**
- Verify feeds are enabled: **Firewall → pfBlockerNG → Feeds**
- Check floating rules exist: **Firewall → Rules → Floating**
- Confirm pfBlockerNG is enabled: **Firewall → pfBlockerNG → General**
- Force a reload: **Firewall → pfBlockerNG → Update → Reload All**

**DNS not resolving at all**
- Verify DNSBL mode is set to Unbound
- Check DNS Resolver is enabled: **Services → DNS Resolver**
- Confirm the DNSBL Virtual IP doesn't conflict with any existing IP on your network
- If your DHCP is configured correctly per the previous guides, devices are already pointed at pfSense for DNS — check that the DNS Resolver listen interfaces include all your VLANs

**Block page not appearing (blank page or connection error instead)**
- Verify the DNSBL Virtual IP is set and doesn't conflict with anything
- Check that the DNSBL SSL certificate was generated
- Some browsers cache DNS aggressively — try a different browser or flush DNS with `ipconfig /flushdns` (Windows) or `sudo dscacheutil -flushcache` (macOS)

**High CPU usage**
- Reduce feed update frequency to weekly
- Disable verbose logging
- Limit feeds to 3–5 to start — you can always add more once things are stable
- Avoid feeds with tens of millions of entries unless your hardware can handle it

---

## Performance Tips

1. **Start small:** Enable 2–3 feeds, run for a week, review logs, then add more
2. **Weekly updates are usually enough:** Daily updates for every feed add unnecessary load
3. **Monitor for false positives:** The first week is the most important time to review logs
4. **Whitelist proactively:** Add known-good domains before users report problems
5. **Keep rule count reasonable:** Aim to stay under 100k IPs in your block lists until you've verified your hardware can handle more

---

## Ongoing Maintenance

| Frequency | Task |
|-----------|------|
| Weekly | Review DNSBL and IP alert logs for false positives |
| Monthly | Check for pfBlockerNG package updates |
| Quarterly | Review feed selection — remove underperforming or redundant feeds |
| Yearly | Clean up old logs and statistics |

---

## Resources

- [pfBlockerNG Documentation](https://docs.netgate.com/pfsense/en/latest/packages/pfblocker.html) — Official Netgate docs
- [Hagezi DNS Blocklists](https://github.com/hagezi/dns-blocklists) — Well-maintained, categorized blocklists
- [FireHOL IP Lists](https://github.com/firehol/blocklist-ipsets) — Curated IP reputation feeds
- [pfSense Community Forum](https://forum.netgate.com) — Best place for pfBlockerNG troubleshooting

---

*This guide reflects practical implementation experience with pfBlockerNG in a home lab environment with multiple VLANs and diverse device types.*
