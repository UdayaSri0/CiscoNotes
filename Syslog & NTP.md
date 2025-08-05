## title: Syslog & NTP – Cisco IOS Configuration Note  
tags: cisco, syslog, ntp, packet-tracer, lab-notes  
created: 2025-08-05

# Syslog & NTP Configuration (Packet Tracer / CCNA Lab)

> **Goal** – Learn to forward device logs to a central Syslog server and synchronize clocks with NTP, both essential for proper troubleshooting, correlation, and compliance. Accurate time synchronization ensures that log events from different devices can be correctly ordered and correlated.  
> 
> **Scope** – IOS 15.x routers/switches in Cisco Packet Tracer 8.2+; CLI syntax is identical on real gear.

---

## 1 Lab Topology 🗺️

```
+--------------------+      192.168.1.0/24        +--------------------+
|  Router/MLS (R1)   |———(F0/1)———┐               |  PC/Server (S1)    |
|  192.168.1.1/24    |           |               |  192.168.1.10/24   |
|  (Syslog *client*, |           |               |  • Syslog daemon   |
|   NTP *client*)    |           |               |  • NTP server      |
+--------------------+           |               +--------------------+
                                 |
                         (other routers/switches)
```

_In Packet Tracer, choose **“Server”** for S1, then enable both “Syslog” and “NTP” services in the **Services** tab._

---

## 2 Syslog 📑

### 2.1 Why Centralized Logging?

- **Single Pane of Glass** – Consolidates logs into one location instead of checking each device individually.
- **Security** – Logs persist on the server even if a device is compromised (attackers must also target the remote store).
- **Time-Correlation** – When paired with NTP, events align across devices for easier analysis.

### 2.2 IOS Configuration (Client Side)

```plaintext
!--- GLOBAL PARAMETERS ----------------------------------------------
service timestamps log datetime msec            ! Precise timestamps
logging on                                      ! Enable logging (default)
logging host 192.168.1.10                       ! Syslog server IP (default UDP 514)
logging trap warnings                           ! Send level 4 (warnings) and higher
logging source-interface Loopback0              ! Use Loopback0 as source IP (optional)
logging facility local4                         ! Set facility for server filtering
logging origin-id hostname                      ! Include hostname in log messages
```

|Directive|Purpose|
|---|---|
|`service timestamps … msec`|Adds date, time, and milliseconds to logs; use _datetime_ for readability, _uptime_ for device uptime.|
|`logging host`|Specifies the Syslog server destination (default UDP 514). Multiple hosts are supported.|
|`logging trap`|Sets severity level to send: 7 (debug) to 0 (emergencies).|
|`logging source-interface`|Ensures consistent source IP, useful with multiple interfaces.|
|`logging facility`|Overrides default _local7_; aids server-side filtering.|
|`logging origin-id`|Adds device identifier (e.g., hostname) to log messages for traceability.|

> **Tip** – Disable console logging (`no logging console`) on busy routers to reduce CPU load.

### 2.3 Syslog Server Setup (Packet Tracer)

1. Select **Server S1** → **Services** tab.
2. Choose **Syslog**, set to **On**.
3. Note the default port is **514/UDP**.
4. View received messages in the **Log** tab when events occur (e.g., interface state changes).

### 2.4 Severity Table (Memorize for CCNA!)

|Level|Keyword|Typical Events|
|---|---|---|
|0|**emergencies**|Router down, kernel panic|
|1|**alerts**|Temperature critical|
|2|**critical**|Power supply failed|
|3|**errors**|Interface error state|
|4|**warnings**|Duplex mismatch|
|5|**notifications**|Line protocol up/down|
|6|**informational**|Normal adjacencies|
|7|**debugging**|Debug outputs|

### 2.5 Verification

```plaintext
show logging               ! Check local buffer and remote stats
show logging | i 192.168   ! Filter for server-specific counters
```

- To generate a test log message, trigger a level 5 event:
    
    ```plaintext
    interface f0/1
     shutdown
     no shutdown
    ```
    
- On S1, check the **Syslog** service’s **Log** tab for messages from R1 (e.g., 192.168.1.1).

---

## 3 NTP ⏰

### 3.1 Concepts

- **Stratum** – Distance from the reference clock. Stratum 0 is the reference (e.g., atomic clock), stratum 1 are servers synced to stratum 0, and so on. Lower stratum numbers indicate higher accuracy.
- **Client/Server** – IOS devices can act as both NTP clients and servers.
- **Authentication** – MD5 keys prevent syncing with rogue clocks (Packet Tracer supports MD5 only).
- **Why It Matters** – Consistent time across devices is crucial for correlating logs and troubleshooting multi-device issues.

### 3.2 Server Configuration (on S1)

In Packet Tracer:

1. Select **Server S1** → **Services** tab.
2. Choose **NTP**, set to **On**.
3. Select **Master** and set **Stratum 5** (suitable for this lab).

> **Real IOS Alternative**
> 
> ```plaintext
> ntp master 5
> ```

### 3.3 Client Configuration (R1 & Peers)

```plaintext
!--- BASIC SYNC ------------------------------------------------------
clock timezone LKT +5 30                      ! Set local time zone (Sri Lanka)
ntp server 192.168.1.10 prefer                ! Primary server
ntp update-calendar                           ! Sync hardware clock (if present)

!--- OPTIONAL AUTH ---------------------------------------------------
ntp authentication-key 10 md5 SEcReT 7
ntp authenticate
ntp trusted-key 10
ntp server 192.168.1.10 key 10 prefer
```

|Command|Comment|
|---|---|
|`clock timezone`|Adjusts time display to local zone (e.g., UTC+5:30).|
|`ntp server … prefer`|Prioritizes this server if multiple are configured.|
|`ntp update-calendar`|Updates hardware clock periodically (for cold boots).|
|`ntp authentication-key`|Defines key ID, algorithm, and secret.|
|`ntp authenticate` + `trusted-key`|Enables authentication and specifies trusted keys.|

> **Note** – In real networks, use multiple NTP servers (e.g., public pools) for redundancy.

### 3.4 Verification

```plaintext
show clock detail            ! Current time and source
show ntp associations        ! Server status, reachability, stratum
show ntp status              ! Synchronization state
```

- In `show ntp associations`, look for a `*` next to 192.168.1.10 (synced peer) and **reach ≥ 377** (octal).
- In `show ntp status`, confirm “synchronized to NTP server.”
- Expect **offset ≤ ±100 ms** in a stable LAN.

---

## 4 Combined Quick Cheat-Sheet 📜

```plaintext
! Syslog (ROUTER)
service timestamps log datetime msec
logging host 192.168.1.10
logging trap warnings
logging origin-id hostname
!
! NTP (ROUTER)
clock timezone LKT +5 30
ntp server 192.168.1.10 prefer
```

---

## 5 Troubleshooting 🛠️

|Symptom|Likely Cause|Fix|
|---|---|---|
|`show ntp status` – unsynchronized|NTP port 123 blocked|`ping 192.168.1.10`; check ACLs for UDP 123|
|Syslog messages missing on server|Wrong `logging trap` level|Lower to `informational` or `debug` for testing|
|Timestamps drift after reload|Missing `ntp update-calendar`|Add command if hardware clock exists|
|`show ntp associations` reach=0|Server unreachable|Verify connectivity and UDP 123|
|NTP association shows “reject”|Authentication mismatch|Check key ID and secret (`show ntp associations`)|
|Log flood on console|Default `logging console debugging`|`no logging console` or raise level|

---

## 6 Next Steps

- Configure **multiple Syslog hosts** and use `logging discriminator` for filtering.
- Set up **authenticated NTP** with public Stratum 2 servers (e.g., pool.ntp.org).
- Export Syslog to a **SIEM** (e.g., ELK Stack) for advanced analysis.

> **Key Takeaway** – Accurate time and centralized logs are the backbone of effective network management and incident response. Master these fundamentals!