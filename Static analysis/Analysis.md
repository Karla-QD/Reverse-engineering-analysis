## Static Analysis - Initial Assessment

In this section I will detail in general the different tools that I used for static analysis for this file.

### PeStudio

- SHA256: eae3f9fb3f39eece67fc6a0c61f1584ce0e40ac6befa19bfa24fac1b094a72fd
- File Type: Win32 Executable (32-bit)
- Entropy: 7.942 (Indicative of packing/encryption).
- Date of creation: Tuesday Oct 28 00:22:05 2025 (UTC)
- Packer: UPX
- Language: c++
- Payload Environment: AutoIt Compiled Script.
<img width="1205" height="616" alt="image" src="https://github.com/user-attachments/assets/188baf94-601d-4717-be22-9c54fa08a255" />


#### Deep Analysis: Section Assessment
The section structure confirms the binary is a compressed "wrapper" designed to unpack itself into memory during execution.
- ***Virtual vs. Raw Size Discrepancy***: UPX0 occupies 0 bytes on disk but allocates 851,968 bytes in memory, a classic indicator of runtime unpacking.
- ***Extreme Entropy***: Both UPX1 (7.937) and .rsrc (7.902) are near the maximum limit (8.0), proving the data is heavily compressed or encrypted.
- ***Self-Modifying Characteristics***: Sections are flagged as Writable and Executable, allowing the malware to write new code to its own memory space and execute it immediately.
- ***Execution Entry Point***: The Entry-Point (0x0012D670) is located at the end of UPX1, where the decompression logic begins.
<img width="1054" height="733" alt="image" src="https://github.com/user-attachments/assets/774a9e38-b36a-4ce3-b876-2bf9719deb44" />

#### Imports Deep Analysis

The Imports section provides a granular view of the malware's malicious behavior, specifically how it intends to manipulate the system and communicate externally. 10 out of 23 imports are flagged as high-risk, pinpointing the sample's primary operational goals.

#### Network & Exfiltration
* **FtpOpenFileW (WININET.dll):** Confirms capability to interact with remote FTP servers for data exfiltration or downloading malicious scripts.
* **IcmpSendEcho (IPHLPAPI.DLL):** Used to "ping" or check network connectivity to a C2 server.
* **4 (connect) (WSOCK32.dll):** Establishes low-level outbound network connections.

#### Memory Injection & Anti-Analysis
* **VirtualAlloc & VirtualProtect (KERNEL32.dll):** Critical for runtime unpacking; these functions allow the malware to allocate and change permissions on memory pages to execute the hidden AutoIt payload.
* **GetProcessMemoryInfo (PSAPI.DLL):** Used to monitor the footprint of other processes, likely to identify and avoid security tools or sandboxes.

#### Information Gathering & Privileges
* **GetAce (ADVAPI32.dll):** Allows the malware to inspect system access control lists, likely to find security weaknesses or escalate privileges.
* **LoadUserProfileW (USERENV.dll):** Used to access user-specific data, such as browser cookies or saved credentials.
<img width="1149" height="531" alt="image" src="https://github.com/user-attachments/assets/b31ad535-95ce-4938-8507-172c43b2ee0d" />

> **Conclusion:** The presence of **VirtualAlloc** combined with **FtpOpenFileW** and **GetAce** characterizes this sample as an active loader/stealer. It is designed to unpack itself into memory (VirtualAlloc), survey the environment (GetProcessMemoryInfo), and communicate over the network (connect, FtpOpenFileW).
