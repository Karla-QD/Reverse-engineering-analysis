
# Advanced Malware Analysis: RedLine Stealer (AutoIt-Compiled Wrapper)

## 📋 Executive Summary
This repository documents a deep-dive technical analysis of a **RedLine Stealer** sample. The malware utilizes a sophisticated multi-stage infection chain, involving a corrupted UPX-packed executable acting as a wrapper for a compiled AutoIt script. The analysis covers the full lifecycle from initial static triage to manual unpacking and final payload decompilation. No deep dynamic analysis (it could be added in the future).

## 🛠️ Laboratory Setup & Toolset
The analysis was performed in a virtualized, air-gapped environment using a dual-OS forensic approach:
* **Analysis Platforms:**
    * **FLARE-VM:** Primary Windows-based environment for dynamic debugging and static analysis.
    * **Remnux:** Linux-based toolkit used for network traffic simulation and behavioral monitoring.
* **Technical Stack:**
    * **Static Analysis:** Pestudio 9.61, PE-bear v0.7.1, Detect It Easy (DIE).
    * **Dynamic Debugging:** x32dbg (TitanEngine).
    * **Payload Extraction:** OllyDumpEx v1.84, AutoIt-Ripper (CLI & Python).
    * **Data Forensics:** CyberChef.

## 🔍 Malware Identification
* **Filename:** `Ascent Industries Limited Scan Documents.exe/ 1vhhubvmn.exe`.
* **SHA256:** `eae3f9fb3f39eece67fc6a0c61f1584ce0e40ac6befa19bfa24fac1b094a72fd`.
* **Architecture:** PE32 (32-bit executable).
* **Entropy:** **7.942** (Indicating high-density compression/encryption).
* **Threat Category:** Information Stealer (RedLine Family).

## 🚀 Technical Methodology

### 1. Static Triage & Indicator Assessment
Initial assessment via **Pestudio** revealed several "red flag" indicators:
* **Self-Modifying Code:** Sections flagged as both Writable and Executable (RWE).
* **Suspicious Imports:** High-risk APIs including `VirtualAlloc` (Memory Injection), `FtpOpenFileW` (Data Exfiltration), and `IcmpSendEcho` (C2 Connectivity check).
* **Packing Detection:** Identified as **UPX 3.91**, though standard automated unpacking failed.

### 2. Overcoming Anti-Analysis Protections
The sample featured a **corrupted PE header** technique to thwart automated unpackers. Standard `upx -d` commands returned a `CantUnpackException`, forcing a manual approach.
* **Manual Unpacking (x32dbg):** Analyzed the `UPX0` and `UPX1` sections. While `UPX0` occupied 0 bytes on disk, it allocated **851,968 bytes** in memory—a textbook indicator of runtime unpacking.
* **Memory Dumping:** After reaching the **Original Entry Point (OEP)** at `0x0012D670`, **OllyDumpEx** was used to acquire a clean process memory dump.

### 3. Payload Decompilation (The AutoIt Layer)
Post-dump analysis with **PE-bear** confirmed the payload was a compiled AutoIt script hidden within the `RC Data` resources.
* **Extraction:** Leveraged **AutoIt-Ripper** to strip the wrapper and decompile the bytecode.
* **Recovered Assets:** Successfully extracted `script.au3` (core malicious logic) and secondary resources named `Zama` and `myriopodous`.
### 4. 🎯 MITRE ATT&CK Mapping

See full mapping here:
➡️ [MITRE Mapping](./MITRE.md)
## 🛡️ Strategic Conclusion & IOCs
This analysis confirms the sample is a highly-evasive **RedLine Stealer** variant. By wrapping the malware in a compiled AutoIt script and corrupting the UPX headers, the threat actor successfully bypassed signature-based detection. The final payload is designed to harvest sensitive user credentials, browser data, and crypto-wallets before exfiltrating them via FTP.

For a complete list of IOCs extracted during the analysis, see:

➡️ [IOCs Documentation](./IOCS/iocs.md)
---

This project demonstrates practical skills in:
- Manual unpacking of packed malware
- Reverse engineering of obfuscated scripts
- In-memory execution analysis
- Threat intelligence mapping (MITRE ATT&CK)
  
*Disclaimer: This analysis is for educational and research purposes only. All activities were conducted in a secure, isolated malware laboratory.*
