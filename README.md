# Digital Forensics Coursework — Operation Karza Jaal

> **Sujal Wasnik · UID: 23015036**  
> B.Sc. (Hons) Cyber Forensics & Information Security

---

## Overview

This repository contains 9 hands-on forensic lab reports built around a single continuous investigation — **Operation Karza Jaal** — a multi-exhibit case involving predatory lending fraud, data destruction, and financial exfiltration. Each experiment follows the standard forensic process: **acquisition → preservation → analysis → reporting**, using industry-standard tools.

Each lab builds on the previous one, culminating in a full cross-exhibit synthesis in Experiment 9.

**Tools used across labs:**  
`Autopsy` · `FTK Imager` · `Volatility 3` · `Wireshark` · `steghide` · `The Sleuth Kit (TSK)` · `VMware` · `IPWHOIS` · `DB Browser for SQLite` · `keytool`

---

## Case Summary

| Suspect / Entity | Role Identified |
|---|---|
| **Vikrant Sable** | Primary suspect — predatory loan operator |
| **Bhavana Technologies Pvt Ltd** | Front company (Nagpur) behind `com.quickpaisa.loan` |
| **`accounts@apexcap-fin.sg`** | External financial contact, fraudulent settlements |
| **IP `103.253.146.77`** | Digital Ocean / Singapore — C2 / exfiltration node |

---

## Experiments

| # | Topic | Key Tool(s) | Report |
|---|-------|-------------|--------|
| 1 | [Lab Setup — Tsurugi Linux VM](#exp-1--lab-setup) | VMware | [📄 Report](exp1/Experiment_1_Lab_Setup_Report_23015036.docx) |
| 2 | [Disk Acquisition & Examination](#exp-2--disk-acquisition--examination) | FTK Imager, Autopsy | [📄 Report](<exp2/23015036_Sujal_Wasnik_Exp2 (2).docx>) |
| 3 | [File Recovery & Obfuscation Detection](#exp-3--file-recovery--obfuscation-detection) | Autopsy, DB Browser | [📄 Report](<exp3/Experiment3_Report (1).docx>) |
| 4 | [Web Browser Artefacts](#exp-4--web-browser-artefacts) | Autopsy | [📄 Report](<exp4/Sujal_Wasnik_23015036_Exp4_Web_Browser_Artefacts_Report (1).pdf>) |
| 5 | [Email Forensics](#exp-5--email-forensics) | Manual Header Analysis, IPWHOIS | [📄 Report](exp5/Email_Forensics_Report.pdf) |
| 6 | [Memory Forensics — RAM Analysis](#exp-6--memory-forensics) | Volatility 3 | [📄 Report](exp6/Exp6_Memory_Forensics_Report.docx) |
| 7 | [Mobile Device Forensics](#exp-7--mobile-device-forensics) | Physical Extraction, keytool | [📄 Report](exp7/EXPERIMENT_7_Examination_Report.docx) |
| 8 | [Network Forensics — Router / PCAP](#exp-8--network-forensics) | Wireshark | [📄 Report](exp8/Experiment_8_Examination_Report.docx) |
| 9 | [Capstone — Steganography + Cross-Exhibit Synthesis](#exp-9--capstone-report) | steghide, TSK, Volatility 3 | [📄 Report](exp9/Experiment9_Forensic_Report_Completed.docx) |

---

## Exp 1 · Lab Setup

**Objective:** Configure a forensic workstation and establish a repeatable case environment.

Set up a **Tsurugi Linux** VM in VMware, took a `BASELINE-CLEAN` snapshot, and created a structured `Forensics_Case/Exp1–Exp9` directory hierarchy for evidence management across all 9 experiments.

---

## Exp 2 · Disk Acquisition & Examination

**Objective:** Acquire a forensically sound disk image and perform initial evidence examination.

- **Part A:** Acquired own pen drive as an `.E01` image using FTK Imager; verified MD5 and SHA-256 hashes pre/post acquisition.
- **Part B:** Loaded `Vikrant.E01` into Autopsy and recovered 3 CTF flags: hidden text in a PDF, a deleted file in the Recycle Bin, and exposed credentials visible in a corkboard photograph.

---

## Exp 3 · File Recovery & Obfuscation Detection

**Objective:** Recover deleted files and identify deliberate evidence tampering.

Examined disk images from **SITABULDI-PC03** and **SITABULDI-PC07**. Discovered an identically-hashed deleted `.xlsx` file on both machines — which, on analysis, was an **XOR-obfuscated SQLite database** containing a 2,100-row loan-target list attributed to `Ops.HYD` (Hyderabad).

> **Key finding:** File extension spoofing + XOR obfuscation used to hide a SQLite database of victims.

---

## Exp 4 · Web Browser Artefacts

**Objective:** Extract and analyse browser history, saved credentials, and cookies.

Analysed Chrome artefacts from `Vikrant.E01` in Autopsy. Identified:
- Saved login credentials for `dashboard.bhavanatech.in`
- Repeated searches for "Bhavana Technologies" as the dominant browsing pattern
- Cookie data corroborating active session use of the fraudulent dashboard

---

## Exp 5 · Email Forensics

**Objective:** Investigate a suspicious email for evidence of financial fraud.

Triaged a mailbox using keyword filter `"settlement"`. Analysed the email from `accounts@apexcap-fin.sg`:
- Reconstructed the **4-hop transmission chain** via raw headers
- Traced originating IP **`103.253.146.77`** → Digital Ocean, Singapore (via IPWHOIS)
- Confirmed missing **DKIM / DMARC** authentication (spoofing indicators)
- Flagged attachment: `Settlement_Reconciliation_Dec2025.xlsx`

---

## Exp 6 · Memory Forensics

**Objective:** Analyse a live RAM dump to identify active processes and in-memory artefacts.

Analysed **`Vikrant-Sable-Laptop-RAM.raw`** (256 MB) using Volatility 3. Key findings:

| Process | Significance |
|---------|-------------|
| `ssh` → `103.253.146.77` | Active exfiltration session to same Singapore IP |
| `chromium` on `bhavanatech` dashboard | Live fraudulent dashboard session |
| `clipmgrd` — clipboard held a **Bitcoin wallet address** | Cryptocurrency payment trail |
| `shred` deleting `customer_export_final.sqlite` | Active evidence destruction at time of capture |

Evidence file integrity: `exp6/hash-sheet-exp6.sha256`

---

## Exp 7 · Mobile Device Forensics

**Objective:** Extract and analyse artefacts from an Android mobile device.

Performed physical extraction forensics on **Aniket Rangari's Xiaomi Redmi 9A** — a victim device. Discovered:
- Predatory loan app **`com.quickpaisa.loan`** — signed by **Bhavana Technologies Pvt Ltd (Nagpur)**
- SMS database confirming **photo-based extortion messages** sent to the victim
- A malicious update APK signed by `"Bhavantech"` (Hyderabad) with an **encrypted payload**

---

## Exp 8 · Network Forensics

**Objective:** Analyse router traffic from a seized network device.

Loaded **`Sitabuldi_Router_20251219.pcap`** (19-Dec-2025 full working day capture from a seized TP-Link Archer C6) into Wireshark. Key tasks:
- Mapped internal DHCP layout (SITABULDI-PC03, PC07, `floor-nas.local`)
- Corrected timezone offset (UTC → UTC+5:30) for timestamp correlation
- Identified encrypted SSH sessions with visible version banners
- Located one unencrypted HTTP request of evidentiary interest

Evidence file integrity: `exp8/hash-sheet-exp8.sha256` · Capture: `exp8/Sitabuldi_Router_20251219.pcap`

---

## Exp 9 · Capstone Report

**Objective:** Synthesise all prior exhibits into a unified forensic narrative.

A comprehensive cross-exhibit report linking all 8 prior investigations:
- **Linux ext4 partition image** analysis: `.bash_history` artefacts, `shred` execution traces, Trash records
- **Steganography:** Recovered a hidden file via `steghide` (blank passphrase) from a JPEG embedded in a ZIP → extracted `CoinSetu_Statement.pdf`
- **Cross-case synthesis:** Connected `103.253.146.77`, the Bitcoin clipboard wallet, and Bhavana Technologies across all exhibits to form a complete forensic picture of the fraud operation

---

## Repository Structure

```
Digital-Forensics-Coursework/
├── exp1/     # Lab Setup — Tsurugi Linux VM
├── exp2/     # Disk Acquisition & Examination (FTK Imager, Autopsy)
├── exp3/     # File Recovery & XOR-obfuscated SQLite detection
├── exp4/     # Web Browser Artefacts (Chrome, Autopsy)
├── exp5/     # Email Forensics — header analysis, IP tracing
│   └── Email_Forensics_Report.pdf
├── exp6/     # Memory Forensics — Volatility 3 (RAM dump)
│   └── hash-sheet-exp6.sha256
├── exp7/     # Mobile Device Forensics — Android extraction
├── exp8/     # Network Forensics — Router PCAP (Wireshark)
│   ├── Sitabuldi_Router_20251219.pcap
│   └── hash-sheet-exp8.sha256
└── exp9/     # Capstone — Steganography + Cross-exhibit synthesis
```

---

## Integrity & Chain of Custody

SHA-256 hash sheets are provided for all exhibits where evidence files were handled:

| File | Covers |
|------|--------|
| `exp6/hash-sheet-exp6.sha256` | RAM image (`Vikrant-Sable-Laptop-RAM.raw`) |
| `exp8/hash-sheet-exp8 (1) (1).sha256` | PCAP capture (`Sitabuldi_Router_20251219.pcap`) |

---

## Skills Demonstrated

- **Forensic Acquisition** — write-blocked E01 imaging, MD5/SHA-256 hash verification
- **File System Analysis** — FAT/NTFS/ext4 artefacts, deleted file recovery, metadata examination
- **Obfuscation Analysis** — XOR-encoded file detection, extension spoofing identification
- **Memory Forensics** — process listing, clipboard extraction, DLL analysis with Volatility 3
- **Network Analysis** — PCAP dissection, protocol filtering, DHCP mapping, timezone correlation
- **Mobile Forensics** — physical extraction, APK signing verification, SMS database analysis
- **Email Investigation** — raw header reconstruction, IP tracing, DKIM/DMARC analysis
- **Steganography Detection** — steghide extraction, carrier file analysis
- **Report Writing** — court-ready examination reports following ACPO / ISO 27037 principles

---

<p align="center">
  <img src="https://img.shields.io/badge/Autopsy-Forensics-blue?style=flat-square" />
  <img src="https://img.shields.io/badge/FTK_Imager-Acquisition-green?style=flat-square" />
  <img src="https://img.shields.io/badge/Volatility_3-Memory_Analysis-purple?style=flat-square" />
  <img src="https://img.shields.io/badge/Wireshark-Network_Forensics-009fbe?style=flat-square" />
  <img src="https://img.shields.io/badge/steghide-Steganography-red?style=flat-square" />
  <img src="https://img.shields.io/badge/The_Sleuth_Kit-File_System-orange?style=flat-square" />
</p>
