
# 🎯 MITRE ATT&CK Mapping

This section maps the observed behaviors of the analyzed sample to the **MITRE ATT&CK** framework.

---

## 🧩 Execution

* **T1059 – Command and Scripting Interpreter**

  * Use of a compiled AutoIt script to execute malicious logic.
  * Obfuscated execution via `Execute()` function.

---

## 🛠️ Defense Evasion

* **T1027 – Obfuscated Files or Information**

  * XOR + bit rotation encoding (`GHOGZQX`)
  * High entropy packed sections (UPX)
  * String obfuscation for API resolution

* **T1140 – Deobfuscate/Decode Files or Information**

  * Runtime decoding of payload (`Zama`) before execution

* **T1620 – Reflective Code Loading**

  * Payload executed directly in memory without being written as a valid PE file

* **T1055 – Process Injection**

  * Allocation and execution of payload in memory (VirtualAlloc + memory write + execution)

---

## 💾 Persistence

* **T1547 – Boot or Logon Autostart Execution**

  * Registry modification via:

    * `RegCreateKeyW`
    * `RegSetValueExW`

---

## 🔍 Discovery

* **T1082 – System Information Discovery**

  * Use of system APIs to gather environment information

* **T1069 – Permission Groups Discovery**

  * `GetAce` used to inspect access control entries

---

## 👤 Credential Access

* **T1555 – Credentials from Password Stores**

  * Access to user profile via `LoadUserProfileW`
  * Likely targeting browser credentials and stored data

---

## 🌐 Command and Control

* **T1071 – Application Layer Protocol**

  * HTTP communication via:

    * `InternetOpenW`
    * `HttpOpenRequestW`

* **T1095 – Non-Application Layer Protocol**

  * Use of raw sockets (`connect`)

* **T1016 – System Network Configuration Discovery**

  * `IcmpSendEcho` used to validate connectivity

---

## 📤 Exfiltration

* **T1041 – Exfiltration Over C2 Channel**

  * Data exfiltration via established C2 connection

* **T1048 – Exfiltration Over Alternative Protocol**

  * FTP usage via `FtpOpenFileW`

---

## 📦 Impact / Collection

* **T1005 – Data from Local System**

  * Collection of sensitive user data (credentials, files)

---

# 🧠 Summary

The analyzed malware demonstrates a **multi-stage, fileless execution model**, combining:

* Obfuscation
* Runtime decryption
* In-memory execution
* Network-based exfiltration

These behaviors strongly align with modern information stealers such as
**RedLine Stealer**, which leverage staged loaders and dynamic API resolution to evade detection.


