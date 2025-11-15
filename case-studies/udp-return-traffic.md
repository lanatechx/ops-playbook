# Case Study: When UDP Return Traffic Never Arrives

## Scenario

Device: video decoder sending **UDP** (SRT/TS-over-UDP style) to a cloud server.  
Two networks tested:

- **Hotspot / simple router** → works.
- **Enterprise network (NAT + firewall)** → outbound OK, **return traffic dies**.

Typical customer complaint:  
> “Your decoder sends fine, but our viewer never receives anything. Must be your box / cloud.”

## Topology (simplified)

```text
[Decoder] --LAN--> [Customer Edge NAT/Firewall] --ISP--> [Internet] -->> [Cloud Server]

Return path:
[Cloud Server] --UDP reply--> [Customer Edge NAT/Firewall]  X  [Decoder never sees it]

## Symptoms

Decoder shows “streaming” / packets sent.
Cloud side sees incoming UDP from customer public IP.
No video / data comes back to the decoder or the internal player.

Same decoder + same cloud server work perfectly when:
- Connected to mobile hotspot
- Connected behind a simple home router

So: device + server are fine. Problem lives somewhere in the customer path.

---

## What I Checked

### 1. Cloud side
- Server receives UDP packets from **customer public IP:port**
- Server sends replies back to **same IP:port**

### 2. Across different networks
- **Hotspot - OK** - proves device + server OK  
- **Enterprise network - still dead** - problem is their edge path, not our gear

### 3. Nature of UDP
- No handshake  
- No session  
- NAT/firewall must infer “state” from outbound packet
- Strict timers often kill UDP “sessions” too fast

---

## Likely Root Causes 

### **1. Stateful firewall dropping return UDP**
Return traffic doesn’t match expected session → dropped as “unsolicited.”

### **2. NAT translation expires too fast**
UDP “state” ages out after a few seconds - return packet has nowhere to go.

### **3. Asymmetric routing**
Outbound takes router A - NAT  
Inbound comes back through router B - no matching state - dropped.

### **4. Over-strict firewall rules**
Cloud reply is using:
- unapproved ports  
- unapproved ranges  
- blocked inbound UDP  

### **5. Multiple layers of NAT (CGNAT / ISP / Enterprise)**
Return packets fail to map all the way through.

---

## Why Hotspot Works but Enterprise Doesn’t

### Hotspot / Home Router
- Simple 1-layer NAT  
- Very relaxed UDP tracking  
- If you send packets out, replies come back  

### Enterprise Edge
- Multiple firewalls, proxies, IDS/IPS  
- Strict state rules  
- VPN hairpinning  
- Multiple NAT layers  
- Policy-based routing  

Same device + same cloud server + same protocol  
- **different network attitude to UDP**.

---

## Summary

A totally normal situation in support:
- Outbound UDP OK  
- Cloud replies OK  
- Enterprise path silently kills return traffic  

Case resolved by escalating to customer’s network/security team.

