# SNMP Setup — Cisco IOS (Packet Tracer Lab)

**Tags**: #Networking #Cisco #SNMP #PacketTracer #LabNotes  
**Created**: 2025-08-05

---

## Introduction

Simple Network Management Protocol (SNMP) enables network monitoring and management by allowing devices (agents) to share data with a manager. This note provides a **minimum-viable** SNMP configuration for Cisco IOS devices in Packet Tracer, focusing on **read-only (RO)** access for monitoring. Optional **read/write (RW)** configuration is included for lab scenarios requiring remote changes.

> [!tip]+ Why Start with RO?  
> Most production environments use **read-only** SNMP to minimize security risks. Reserve **RW** for labs or automation with strict controls.

---

## 1. Prerequisites 🔧

- **Device (Agent)**: Any IOS router or multilayer switch in Packet Tracer (e.g., `Router0`).
- **Manager IP**: A PC or server configured as an SNMP manager (e.g., `192.168.1.10`).
- **Connectivity**: Ensure Layer 3 reachability (`ping 192.168.1.10` from the device).

> [!note]  
> Verify IP configuration and routing before enabling SNMP.

---

## 2. Configuration Cheat-sheet 📄

### 2.1 Read-Only (RO) Community 🕵️‍♂️

```plaintext
! Define a global RO community string
snmp-server community MyR0Public RO

! (Optional) Restrict access to the manager's IP
ip access-list standard SNMP-MANAGERS
 permit 192.168.1.10
end
snmp-server community MyR0Public RO SNMP-MANAGERS
```

**Explanation**:

- `MyR0Public`: User-defined community string (sent in plain text for SNMP v1/v2c).
- `RO`: Permits only read operations (e.g., GET, GETNEXT).
- `SNMP-MANAGERS`: Standard ACL restricting polling to the manager’s IP.

---

### 2.2 Read/Write (RW) Community ✍️

> [!warning]  
> Use RW only in secure, controlled environments (e.g., labs). Avoid in production without protections like VPNs or AAA.

```plaintext
snmp-server community MyR0Private RW SNMP-MANAGERS
```

- `MyR0Private`: RW community string.
- Applies the same ACL (`SNMP-MANAGERS`) for consistency.

---

### 2.3 House-keeping Info 🏷️

```plaintext
snmp-server location "Kandy HQ – MDF"
snmp-server contact "UdayaSri, NetOps, +94 712 345 678"
```

- Populates `sysLocation` and `sysContact` OIDs for inventory tracking.

---

### 2.4 Trap Destination (Optional) 📨

```plaintext
snmp-server host 192.168.1.10 MyR0Public
snmp-server enable traps snmp linkdown linkup
```

- `snmp-server host`: Sends traps to the manager (UDP 162) using the RO community string.
- `snmp-server enable traps`: Enables specific traps (e.g., `linkdown`, `linkup`) rather than all traps for better control.

---

## 3. Verification 🧪

Run these commands on the Cisco device:

```plaintext
show snmp                ! Check SNMP status
show snmp community      ! List community strings and ACLs
show snmp host           ! Verify trap receivers
debug snmp packets       ! Monitor real-time SNMP traffic (Ctrl-C to stop)
```

- **Manager Side**: Use Packet Tracer’s **SNMP Manager** tool (Desktop tab):
    - Click **Poll** to retrieve data (e.g., `sysName`, interfaces).
    - Disconnect a link and check the **Trap Log** for events.

---

## 4. Troubleshooting ⛑️

|Symptom|Likely Cause|Fix|
|---|---|---|
|Timeout / No response|Wrong community string or ACL|Verify string; `show access-lists`|
|“Bad community name”|RW used but only RO defined|Add RW or adjust manager settings|
|Trap log empty|Traps not enabled or wrong IP|Check `snmp-server host` and traps|
|Unreachable|IP/gateway/subnet mismatch|Use `ping` and `show ip int br`|

---

## 5. Next Steps 🚀

1. **Scale Up**: Configure multiple devices and poll from a central manager.
2. **Explore MIBs**: Use Packet Tracer’s **OID Browser** to inspect specific objects.
3. **SNMP v3**: Transition to SNMP v3 (in GNS3 or physical devices) for authentication and encryption.

> [!info]  
> SNMP v1/v2c sends community strings in plain text, making it less secure. SNMP v3 offers authentication and encryption but isn’t fully supported in Packet Tracerenglisch

---

**Happy Monitoring!** 🎉