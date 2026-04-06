###  Analysis & Manual Unpacking
#### X32DBG

The dynamic analysis initiated by identifying the initial entry point at `0x00CBD670`. The presence of the pushad instruction confirmed the beginning of the UPX decompression stub, serving as the starting point for the manual OEP (Original Entry Point) discovery.

<img width="1158" height="870" alt="image" src="https://github.com/user-attachments/assets/db60d254-515b-4e44-824e-4f00295093bb" />


##### **Phase 2: Executing the ESP Trick & Locating the Tail Jump**

Once I identified the initial entry point and the `PUSHAD` instruction, my objective was to determine exactly when the malware finished decompressing its payload into memory. To achieve this without manually tracing thousands of instructions, I utilized the **ESP Trick**, a forensic memory technique.

**Technical Procedure:**
1.  **Step-Over Execution:** I performed a single step-over (`F8`) to execute the `PUSHAD` instruction. This action pushed the current state of all general-purpose registers onto the Stack.
2.  **Hardware Breakpoint on ESP:** In the CPU registers window, I right-clicked the **ESP** value and selected **"Follow in Dump"**. In the Hex Dump window, I selected the first 4 bytes (DWORD) of the pushed data and set a **Hardware Breakpoint on Access**.
    * **Rationale:** This forces the debugger to pause execution the moment the malware attempts to restore the original register state using a `POPAD` instruction—a definitive sign that the decompression stub has finished its routine.
3.  **Execution to Breakpoint:** I resumed execution (`F9`). The malware ran its internal decompression logic, writing the hidden payload into the `UPX0` memory section. As predicted, the debugger triggered the breakpoint immediately after the `POPAD` instruction was executed.
4.  **Tail Jump Identification:** Right after the `POPAD`, I successfully located a "long jump" (**JMP**) `00CBD84D` instruction. This is the **Tail Jump**, which acts as the transition point between the decompression stub and the malware's real code. In this specific sample, the jump led me directly to the code responsible for the AutoIt environment.

##### **Key Analytical Takeaway:**
Using a hardware breakpoint on the stack ensures a high degree of precision. Rather than guessing where the packer ends, I leveraged the CPU's own memory management to force the malware to reveal its **Original Entry Point (OEP)**. This methodology is critical for bypassing packers with corrupted headers that break automated tools.

<img width="1156" height="863" alt="image" src="https://github.com/user-attachments/assets/e480a021-59fa-4347-86b7-024aeadcb944" />

##### **Phase 3: Reaching the OEP & Dumping the Payload**

With the **EIP** (Instruction Pointer) positioned at the final `JMP` instruction at **`00CBD84D`**, I executed the final leap to the malware's true code.

 **Technical Execution:**
1.  **The Leap (F8):** I performed a single **Step Over** on the "Tail Jump." This redirected the execution flow out of the UPX stub and directly into the decrypted payload.
2.  **OEP Confirmation:** The **EIP** landed on the **Original Entry Point (OEP)**. I verified this by observing the transition from obfuscated loops to a standard Windows executable prologue (e.g., `PUSH EBP`, `SUB ESP`).

<img width="1155" height="825" alt="image" src="https://github.com/user-attachments/assets/04d1eb40-748e-432d-a0de-79d90141f59e" />

---

3.  **Memory Dump:** Using the **OllyDumpEx** plugin in **x32dbg**, I "froze" this unpacked state:
    * **Entry Point:** Set to the current OEP.
    * **Action:** Dumped the process memory to disk as **`Malware_unpacked.exe`**.

- OEP: 0x00BB04F7
<img width="614" height="495" alt="image" src="https://github.com/user-attachments/assets/85b31989-4ea8-4df1-88cb-445031292247" />

### Post-Unpacking Verification  

#### Detect It Easy (DIE)
Confirmed the unpacked binary (Malware_unpacked.exe) has a valid MZ header and PE structure, indicating a successful dump.
<img width="1146" height="510" alt="Screenshot 2026-03-02 223357" src="https://github.com/user-attachments/assets/64e30fbf-09e8-435d-86ea-f6ae8703aafe" />



### **CyberChef: Post-Unpacking String Analysis**

After dumping the binary, I used **CyberChef** to extract strings from `Malware_unpacked.exe`. This confirmed the success of the manual unpacking and identified the malware's functional capabilities.

#### **1. Network & C2 Communication**
The presence of **WinInet** and **IPHLPAPI** libraries confirms outbound capabilities:
* **`HttpOpenRequestW` / `InternetOpenW`**: Used to establish connections with a Command & Control (C2) server.
* **`IcmpSendEcho`**: Likely used to "ping" the C2 to verify connectivity before exfiltrating data.

#### **2. Persistence & Registry Manipulation**
The sample shows extensive interaction with the Windows Registry, a hallmark of **Persistence**:
* **`RegSetValueExW` / `RegCreateKeyW`**: Used to modify registry keys to ensure the malware executes automatically upon system reboot.
* **`RegDeleteValueW`**: Suggests the ability to remove security settings or its own traces to evade detection.

#### **3. Information Gathering**
* **`LoadUserProfileW`**: Indicates the malware targets sensitive user data (cookies, saved credentials).
* **`GetAce`**: Used to inspect system permissions, likely for privilege escalation or identifying security weaknesses.

> **Summary:** The string analysis validates that the unpacked binary is a functional **Infostealer/Loader**, equipped to survive reboots, survey the system, and communicate with remote attackers.


<img width="1057" height="220" alt="image" src="https://github.com/user-attachments/assets/2169eee9-7d26-4a47-8e4a-ef7d28e07ee4" />

### Payload Extraction

#### AutoIt Ripper
Since the binary is a compiled AutoIt script, I used AutoIt-Ripper to extract the internal resources. This resulted in the discovery of script.au3 and supporting files like `myriopodous` and `zama`.
The AutoIt payload was extracted using AutoIt-Ripper:
```
python -m autoit_ripper.cli Malware_unpacked.exe .
```
