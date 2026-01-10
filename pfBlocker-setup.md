# pfBlockerNG Setup and Configuration Guide
**Author:** Greg Diny

A guide to setting up pfBlockerNG for IP blocking and DNS filtering on pfSense.

## What is pfBlockerNG?

pfBlockerNG is a powerful pfSense package that provides:
- **IP reputation blocking** - Block malicious IPs from threat feeds
- **DNS-based filtering (DNSBL)** - Block ads, trackers, and malware domains
- **GeoIP blocking** - Block traffic from specific countries/regions
- **ASN blocking** - Block entire ISP/organization networks

## Prerequisites

- pfSense 2.6+ installed and configured
- Internet connectivity
- Basic understanding of pfSense firewall rules

## Installation

### 1. Install pfBlockerNG Package

1. Navigate to: **System → Package Manager → Available Packages**
2. Search for `pfblockerng`
3. Click **Install**
4. Wait for installation to complete

### 2. Initial Configuration

1. Navigate to: **Firewall → pfBlockerNG → General**
2. **Enable pfBlockerNG:** Check the box
3. **CRON Settings:**
   - **Daily:** `00:00` (midnight)
   - **Weekly:** Sunday at `01:00`
4. **Keep Settings:** `52 weeks` (keep historical data for a year)
5. Click **Save**

## IP Blocking Configuration

### 1. Configure IP Settings

Navigate to: **Firewall → pfBlockerNG → IP**

**Settings:**
- **Enable IP Blocking:** ✓
- **Outbound Firewall Rules:** ✓ (blocks internal devices from reaching malicious IPs)
- **Inbound Firewall Rules:** ✓ (blocks malicious IPs from reaching your network)
- **Logging:** Enable (helps with troubleshooting)

### 2. Add IP Threat Feeds

Navigate to: **Firewall → pfBlockerNG → Feeds**

**Recommended Threat Feeds:**
- **Emerging Threats:** Block known malicious IPs
- **Spamhaus DROP/EDROP:** Hijacked networks
- **Abuse.ch URLhaus:** Malware distribution sites
- **FireHOL Level 1:** Comprehensive malicious IP list

**To Add a Feed:**
1. Click **Add**
2. **Format:** IPv4
3. **Action:** Deny Both (block inbound and outbound)
4. **Frequency:** Daily
5. **URL:** (feed URL from provider)
6. **Description:** (descriptive name)
7. Click **Save**

### 3. GeoIP Blocking (Optional)

Navigate to: **Firewall → pfBlockerNG → IP → GeoIP**

**Example: Block High-Risk Regions**
- Select countries to block (e.g., known high-traffic spam sources)
- **Action:** Deny Inbound
- **Logging:** Enable

> **Warning:** GeoIP blocking can cause issues with legitimate services. Test carefully.

## DNS Filtering (DNSBL) Configuration

DNS filtering blocks domains before they resolve, preventing connections to malicious or unwanted sites.

### 1. Enable DNSBL

Navigate to: **Firewall → pfBlockerNG → DNSBL**

**Settings:**
- **Enable DNSBL:** ✓
- **DNSBL Mode:** Unbound mode (integrates with pfSense DNS)
- **DNSBL Listening Interface:** LAN (or your internal interface)
- **DNSBL SSL Certificate:** Generate (for HTTPS blocking)

### 2. Add DNSBL Feeds

**Recommended Feeds:**
- **EasyList:** Ad blocking
- **Steven Black Unified:** Ads + malware + trackers
- **Malware Domain List:** Known malware domains
- **Ransomware Tracker:** Ransomware C&C servers

**To Add DNSBL Feed:**
1. Navigate to: **Firewall → pfBlockerNG → DNSBL → DNSBL Groups**
2. Click **Add**
3. **Group Name:** (e.g., "Ads")
4. **Action:** Unbound
5. **Logging:** Enable
6. **Feed Sources:** Add feed URLs
7. Click **Save**

### 3. Configure DNS Resolver

Navigate to: **Services → DNS Resolver → General Settings**

Ensure:
- **Enable DNS Resolver:** ✓
- **DNSSEC:** ✓ (recommended)
- **Listen Interfaces:** LAN
- **Outgoing Interfaces:** WAN

## Firewall Rule Configuration

pfBlockerNG creates automatic firewall rules, but you may need to adjust:

Navigate to: **Firewall → Rules → Floating**

You should see pfBlockerNG auto-generated rules. These should be:
- **At the top of the rule list** (processed first)
- **Quick:** Enabled (stops processing once matched)
- **Interface:** Any

## Testing and Verification

### 1. Test IP Blocking

```bash
# Try to ping a blocked IP
ping <blocked-ip>
# Should timeout or be rejected
```

### 2. Test DNSBL

Visit a known tracking domain (e.g., doubleclick.net). It should:
- Fail to resolve
- Show pfBlockerNG block page (if configured)

### 3. Check Logs

Navigate to: **Firewall → pfBlockerNG → Reports**

Review:
- **Alerts:** Blocked connections
- **DNSBL:** Blocked DNS queries
- **Top Sources:** Devices making the most requests

## Troubleshooting

**Legitimate sites being blocked:**
1. Navigate to: **Firewall → pfBlockerNG → Reports**
2. Find the blocked domain/IP
3. Click **Suppress** to whitelist
4. Or add to **Custom Lists → Whitelist**

**pfBlockerNG not blocking anything:**
- Verify feeds are enabled and updated
- Check firewall rules are present: **Firewall → Rules → Floating**
- Ensure pfBlockerNG is enabled: **Firewall → pfBlockerNG → General**
- Force an update: **Firewall → pfBlockerNG → Update**

**High CPU usage:**
- Reduce feed frequency to weekly
- Disable verbose logging
- Limit number of feeds (start with 3-5)

**DNS not working:**
- Verify DNSBL mode is set correctly (Unbound)
- Check DNS Resolver is enabled
- Ensure devices point to pfSense for DNS

## Performance Tips

1. **Start small:** Enable 2-3 feeds, test, then add more
2. **Use daily updates:** Weekly for most feeds is sufficient
3. **Monitor logs:** Check false positives regularly
4. **Whitelist carefully:** Add trusted domains to prevent blocks
5. **Optimize rules:** Keep rule count reasonable (under 100k IPs)

## Maintenance

**Regular Tasks:**
- **Weekly:** Review blocked domains/IPs for false positives
- **Monthly:** Check for package updates
- **Quarterly:** Review and optimize feed selection
- **Yearly:** Clean up old logs and statistics

## Resources

- **pfBlockerNG Documentation:** Official pfSense docs
- **Feed Providers:** Research reputable threat intelligence sources
- **Community Forums:** pfSense forum for troubleshooting

---

*This guide reflects practical implementation experience with pfBlockerNG in a home network environment with multiple VLANs and diverse device types.*
