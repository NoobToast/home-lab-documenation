# VLAN Setup Guide in pfSense 2.8
**Author:** Greg Diny

A step-by-step guide to configuring VLANs in pfSense for network segmentation.

## Step 1: Create VLAN Interface in pfSense

1. Log in to pfSense WebGUI
2. Navigate to: **Interfaces → Assignments → VLANs**
3. Click **Add**
   - **Parent Interface:** Select your LAN NIC (e.g., `igb0`)
   - **VLAN Tag:** `10` (or your chosen VLAN ID)
   - **Description:** Describe the VLAN purpose
   - Click **Save**
4. Go to **Interfaces → Assignments**
   - Add the new VLAN as an interface
   - Name it `DEVICE_VLAN10` (or similar)
   - **Enable** the interface
   - Set **IPv4 Configuration Type** to **Static IPv4**
   - Enter **IP Address:** `192.168.10.1/24`
   - Click **Save & Apply**

## Step 2: Configure Firewall Rules

1. Navigate to: **Firewall → Rules → [New VLAN]**
2. Click **Add**
   - **Action:** Pass
   - **Interface:** New VLAN
   - **Protocol:** Any
   - **Source:** New VLAN net
   - **Destination:** Any
   - **Description:** Allow All from New VLAN
3. Click **Save** and **Apply Changes**

> **Note:** You can refine these rules later if you want the VLAN isolated from your main LAN but still internet-enabled.

## Step 3: Enable DHCP for New VLAN

1. Go to: **Services → DHCP Server → [New VLAN]**
2. Check **Enable DHCP server on this interface**
3. Set DHCP Range:
   - **Start:** `192.168.10.100`
   - **End:** `192.168.10.199`
4. Add Static Mappings (optional):
   - Example: Desktop NIC MAC → `192.168.10.10`
5. Click **Save**

## Step 4: Configure Switch

### Uplink Port (to pfSense)
- **Mode:** Trunk (Tagged)
- **Allowed VLANs:** Default VLAN + VLAN 10

### Device Port
- **Mode:** Access (Untagged)
- **VLAN ID:** 10

## Step 5: Desktop Setup

1. Plug desktop into VLAN 10 access port on switch
2. Device should receive IP via DHCP automatically
3. **Verify Configuration:**
   - IP address should be in range `192.168.10.100-199`
   - Gateway should be `192.168.10.1`
   
4. **Test Connectivity:**
   ```bash
   # Test pfSense VLAN interface
   ping 192.168.10.1
   
   # Test internet connectivity
   ping 8.8.8.8
   
   # Test DNS resolution
   ping google.com
   ```

## Verification

At this point, VLAN 10 is live with:
- ✅ Its own IP space (`192.168.10.0/24`)
- ✅ DHCP server for automatic IP assignment
- ✅ Firewall rules allowing internet access
- ✅ Switch port assignments
- ✅ Network segmentation from main LAN

Your desktop is now isolated on its own network segment but can still reach pfSense and the internet.

## Troubleshooting

**No IP address assigned:**
- Verify DHCP is enabled on the VLAN interface
- Check switch port configuration (correct VLAN ID)
- Ensure device NIC is set to obtain IP automatically

**Can't reach internet:**
- Check firewall rules allow outbound traffic
- Verify pfSense gateway is configured
- Test DNS resolution separately from connectivity

**Can't access pfSense GUI:**
- Ensure firewall rule allows access to pfSense IP
- Check anti-lockout rule is enabled
- Verify correct gateway IP in device network settings

---

*This guide was developed through hands-on implementation in my home lab. All steps were personally tested with pfSense 2.8.*
