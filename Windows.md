# 📘 Guide: Fixing Windows 11 Boot + Creating EFI + Bypassing "This PC can't run Windows 11"

## ⚠️ Situation

- The **Windows 11 disk has no EFI partition**, so when the Windows 10 disk is disconnected — the Windows 11 bootloader simply doesn't exist → PC goes straight to BIOS.
- The command `bootrec /rebuildbcd` throws an error:  
    **"The requested system device cannot be found"**  
    because there's no EFI.
- `create partition efi size=400` doesn't work → **no free space at the beginning of the disk**.
- Windows shows installation error:  
    **"This PC can't run Windows 11"**  
    — need to bypass Secure Boot / TPM requirements.

## 1️⃣ Checking Disk Structure

```powershell
diskpart
list disk
select disk 0
list part
```

|#|Type|Size|
|---|---|---|
|1|Reserved|16 MB|
|2|Primary|953 GB|

👉 This means the **disk is GPT**, but the **EFI partition is completely missing**.

## 2️⃣ Creating EFI Partition (if no free space)

To create EFI (~100–300 MB), you need to free up space **at the beginning of the disk**.

### Method A — Safe (using MiniTool / AOMEI / GParted)

1. Boot from any LiveUSB:
    - **AOMEI Partition Assistant**
    - **MiniTool Partition Wizard**
    - **GParted**
2. Select the Windows 11 disk (Disk 0).
3. **Shrink the main partition (Primary) by 300–600 MB FROM THE LEFT**.
4. Create a new partition:
    - type: **FAT32**
    - size: **300 MB**
    - label: **EFI**
    - ID (if available): **EF00**

Now you have an EFI partition.

## 3️⃣ Linking Windows 11 Bootloader to the New EFI

Boot from Windows Install USB → Shift+F10 → command prompt:

### 3.1. Find the EFI drive letter

```powershell
diskpart
list disk
select disk 0
list part
select part X   ← EFI
assign letter=Z
exit
```

### 3.2. Copy the bootloader

```powershell
bcdboot C:\Windows /s Z: /f UEFI
```

If successful, you'll see:

```
Boot files successfully created.
```

## 4️⃣ Fixing bootrec Errors

If you see this again:

```
The requested system device cannot be found
```

This means EFI:

- doesn't exist
- is not FAT32
- is not mounted
- has no drive letter assigned
- or is not recognized as GPT

Solution — mount EFI and repeat:

```powershell
assign letter=Z
bcdboot C:\Windows /s Z: /f UEFI
```

## 5️⃣ Bypassing "This PC can't run Windows 11" Check

### Method 1 — Simple (via Shift+F10)

1. On the error screen, press **Shift + F10**.
2. Launch the registry editor:

```powershell
regedit
```

3. Navigate to:

```
HKEY_LOCAL_MACHINE\SYSTEM\Setup
```

4. Create a new key:

```
LabConfig
```

5. Inside LabConfig, create DWORD (32-bit) values:

```
BypassTPMCheck        = 1
BypassSecureBootCheck = 1
BypassRAMCheck        = 1
BypassCPUCheck        = 1
```

6. Close the window and continue the installation.