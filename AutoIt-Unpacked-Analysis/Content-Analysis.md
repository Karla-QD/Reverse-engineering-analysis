# 📄 Reverse Engineering Report – AutoIt Malware Loader

## 1. Introduction

During this analysis, I examined an obfuscated AutoIt script extracted from a suspicious sample. My objective was to understand its execution flow, decoding mechanisms, and runtime behavior.

From the initial inspection, I identified that the script acts as a **multi-stage loader**, designed to decode and execute a payload directly in memory while evading static analysis.

---

## 2. High-Level Execution Flow

Based on my analysis, the script follows this sequence:

1. Drops embedded resources (`Zama`, `myriopodous`)
2. Reads and decodes the main payload (`Zama`)
3. Dynamically resolves API calls
4. Allocates memory and injects the decoded payload
5. Executes the payload in memory
6. Executes large amounts of junk code to evade analysis

---

## 3. Stage 1 – Embedded Payload Deployment

The script begins by extracting an embedded file:

```autoit
FileInstall("Zama", @TempDir & "\Zama", 1)
```

**Figure 1 – Embedded payload extraction using FileInstall**

<img width="598" height="261" alt="image" src="https://github.com/user-attachments/assets/bfebeafb-01b9-4a11-b8f8-42e34251cfb9" />


I observed that the script drops the file into the system's temporary directory and immediately reads it into memory.

This indicates that the payload is **embedded within the script itself** and not downloaded externally.

---

## 4. Stage 2 – Payload Deobfuscation

The payload is processed using a custom function:

```autoit
GHOGZQX(data, key=93)
```

Inside this function, I identified the following operations:

* Byte-by-byte iteration
* XOR with a static key (`93`)
* Bit rotation (3-bit shift)

This can be summarized as:

```
decoded_byte = ROTR(byte XOR 93, 3)
```

---

### 4.1 Bit Manipulation via Dynamic Execution

 **Figure 2 – Bit rotation implemented via Execute()**

<img width="617" height="105" alt="image" src="https://github.com/user-attachments/assets/6c4962b4-87ed-46e9-8e9c-c0a57b98bed6" />

The rotation is implemented using:

* `BitShift`
* `BitAND`

However, these are wrapped inside an `Execute()` call with string concatenation, which prevents static detection of the operation.

I concluded that this technique is used to:

* Hide transformation logic
* Evade signature-based detection

---

## 5. Stage 3 – Dynamic API Resolution

Instead of using plain API names, the script decodes them at runtime:

```autoit
DllCall(GHOGZQX("encoded_string"), ...)
```

 **Figure 3 – Obfuscated API resolution via GHOGZQX**

<img width="637" height="218" alt="image" src="https://github.com/user-attachments/assets/38692350-fd50-41bf-a7dd-796d707ac1b2" />


From this, I determined that:

* DLL names and API functions are stored obfuscated
* They are only resolved during execution

This prevents detection by:

* Static scanners
* String-based signatures

---

## 6. Stage 4 – Memory Allocation and Execution

The script then performs a sequence consistent with in-memory execution.

### 6.1 Memory Allocation

**Figure 4 – Memory allocation via DllCall**

<img width="624" height="256" alt="image" src="https://github.com/user-attachments/assets/6c47758c-1130-414c-a817-1617473ab2d0" />


I observed a call pattern consistent with `VirtualAlloc`.

---

### 6.2 Writing Payload to Memory

```autoit
DllStructCreate(...)
DllStructSetData(...)
```

 **Figure 5 – Writing decoded payload into allocated memory**

<img width="620" height="623" alt="image" src="https://github.com/user-attachments/assets/b85e97e1-3e71-4cc2-855c-14b2a615a0cf" />


This step copies the decoded payload into the allocated memory region.

---

### 6.3 Payload Execution

 **Figure 6 – Execution of injected payload**


The script then executes the payload using another dynamically resolved API call.

Based on the pattern, I assess that the following APIs are likely used:

* `VirtualAlloc`
* `RtlMoveMemory`
* `CreateThread` (or equivalent execution method)

---

## 7. Stage 5 – Secondary Payload Deployment

The script also drops another embedded file:

```autoit
FileInstall("myriopodous", @TempDir & "\myriopodous", 1)
```
<img width="573" height="300" alt="image" src="https://github.com/user-attachments/assets/8852a9f1-633b-46a1-a3a1-b8fd064bc295" />

 **Figure 7 – Secondary payload deployment**


This suggests the presence of:

* A second-stage payload
* Possible persistence or additional functionality

---

## 8. Obfuscation and Anti-Analysis Techniques

A significant portion of the script consists of non-functional code.

I observed:

* Deeply nested loops
* Random variable names
* Irrelevant GUI and file operations
* Contradictory conditions

These blocks do not contribute to execution and serve only to:

* Obfuscate logic
* Break static analysis tools
* Increase analysis complexity

---

## 9. Behavioral Summary

From my analysis, the script performs the following actions:

* Drops embedded payloads
* Decodes payload using XOR + bit rotation
* Dynamically resolves Windows API calls
* Allocates executable memory
* Injects payload into memory
* Executes payload (fileless)
* Deploys secondary payload
* Uses heavy obfuscation to evade detection

