
title: Cisco Packet Tracer Config Vault 📂
tags: cisco, packet-tracer, configs, networking, lab-notes
created: 2025-08-05
---

> [!abstract]+ Quick-glance  
> **Purpose** – centralise, version, and cross-link all your Cisco Packet Tracer configurations and lab notes inside one Obsidian vault.  
> **Audience** – network students, lab enthusiasts, and future-you.  
> **How to use** – browse the *[[toc]]*, open any lab note, copy-paste the config blocks into Packet Tracer, and hack away.  

[[toc]]

---

## 1  Vault Layout 🏗️  

| Folder          | What goes inside                                          | Tip                                                                |
|-----------------|-----------------------------------------------------------|--------------------------------------------------------------------|
| `01_Labs/`      | One note per practical lab (e.g. `STP_Redundancy.md`)     | Keep filenames identical to `.pkt` files for quick 🔍 search      |
| `02_Configs/`   | Raw device configs (`R1.txt`, `SW-Core01.txt`)            | Store *pre-* and *post-*lab snapshots                              |
| `03_Reference/` | Cheat-sheets, command explainers, IOS quirks              | Tag heavily for speedy lookup                                      |
| `assets/`       | Images, topology PNGs, Mermaid exports                    | Embed with `![[...]]`                                              |

> [!info]- Naming Convention  
> `DeviceType_LabShortName_Stage.ext` → **`SW_01_VLANs_post.cfg`** keeps lists tidy.

---

## 2  Adding a New Lab 🆕  

1. **Duplicate** the template note `01_Labs/_LabTemplate.md`.  
2. Rename it descriptively – e.g. `RIPv2_Authentication.md`.  
3. Fill the *Scope*, *Diagram*, *Tasks*, and *Verification* sections.  
4. Paste device configs inside fenced blocks for syntax highlighting.  
5. Drag-and-drop the Packet Tracer `.pkt` file into the note to attach it.  

~~~shell
! Sample config (router)
hostname R1
interface g0/0
 ip address 192.0.2.1 255.255.255.0
 no shutdown
!
router ospf 1
 network 192.0.2.0 0.0.0.255 area 0
~~~

> [!tip] Syntax Highlighting  
> Use `shell` for IOS CLI, `mermaid` for diagrams, and callouts for notes/warnings.

---

## 3  Typical Lab Note Skeleton 🗂️  

```

## Scope

## Topology 🖼️

```

~~~mermaid
graph TD
    PC1 -->|Fa0/0| SW1
    SW1 --> R1
~~~

```

## Device Initialisation

## Configuration Steps

## Verification ✔️

## Troubleshooting 🛠️

## Reflection 📝

```

Copy this skeleton whenever you spin up a new scenario.

---

## 4  Dataview Magic ✨  

~~~dataview
table file.name as "Lab", Scope, Status
from "01_Labs"
where Status != "Done"
sort file.name asc
~~~

Add `Status:: In-Progress` in each lab note’s YAML and watch the list update.

---

## 5  Best-Practices 📜  

- **Atomic commits** – tweak one lab at a time for a clean Git history.  
- **Screenshots** – paste right after command output—future-you will thank you.  
- **Version tags** – `#ios15`, `#pkt820` highlight feature-specific quirks.  
- **Callouts over comments** – Obsidian callouts keep notes readable.  
- **Backlinks** – link theory pages (e.g. `[[STP Concepts]]`) from each lab for fast revision.

---

## 6  Contribution Guide 🤝  

1. Fork → branch → commit (use `feat: add dhcp-relaying lab`).  
2. Run spell-check (`Codespell` extension) & preview in Obsidian.  
3. Open PR; CI linters check YAML front-matter and Markdown links.  
4. Once merged, delete your branch to keep things tidy.

---

## 7  Future Ideas 🚀  

- 🔄 **CI-Driven Validation** – GitHub Action that spins up `iosv` containers with posted configs.  
- 📺 **Short demo GIFs** – visualise packet flow in Wireshark.  
- 🧩 **Terraform for Cisco Modeling Labs** – auto-convert Packet Tracer labs to CML files.

> [!quote] “Documentation is never finished; only published.” – *Every NetEng Ever*

Happy labbing! 🔌
