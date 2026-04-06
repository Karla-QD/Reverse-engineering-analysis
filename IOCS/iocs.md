# 🛡️ Indicators of Compromise (IOCs)

## 📌 File Hashes

| Artifact       | Type   | Hash                                                               |
| -------------- | ------ | ------------------------------------------------------------------ |
| Initial Sample | SHA256 | `eae3f9fb3f39eece67fc6a0c61f1584ce0e40ac6befa19bfa24fac1b094a72fd` |
| Zama           | SHA256 | `0dcec519d5eadfdd8b70f15f53506c33497a38d9b3ca7de1b7a48e9bd41b2d4f` |
| myriopodous    | SHA256 | `d90af9f7d7e56bc5829597f070a18d2ddfdf0721c22b0eae9a32ea1c020e1a86` |

---

## 📁 Dropped Files

| File Name   | Path                 | Description                                        |
| ----------- | -------------------- | -------------------------------------------------- |
| Zama        | `%TEMP%\Zama`        | Encrypted intermediate payload (decoded in memory) |
| myriopodous | `%TEMP%\myriopodous` | Secondary payload / potential final stage          |

---
