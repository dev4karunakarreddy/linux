# Removing Unwanted Boot Entries and Cleaning Up EFI Partition

## **Overview**
If you have removed Linux distributions like **Fedora, Kali, or Debian** but still see them in your boot options, you need to manually delete their boot entries and clean up the EFI partition.

This guide provides step-by-step instructions to remove these entries using **Windows Command Prompt (Admin)** and **disk management tools**.

---

## **Step 1: Identify Unwanted Boot Entries**
1. Open **Command Prompt as Administrator**:
   - Press `Win + R`, type `cmd`, and press `Ctrl + Shift + Enter`.
2. Run the following command to list all boot entries:
   ```sh
   bcdedit /enum firmware
   ```
3. Locate the **identifier** for Fedora, Kali, and Debian in the output. Example:
   ```
   identifier              {96bba342-f6a2-11ef-b4fa-806e6f6e6963}
   description             Fedora
   
   identifier              {9c165910-a6b8-11ef-b16f-90754eefef65}
   description             kali
   
   identifier              {9c165911-a6b8-11ef-b16f-90754eefef65}
   description             debian
   ```

---

## **Step 2: Remove Unwanted Boot Entries**
Run the following commands to remove the Fedora, Kali, and Debian entries (replace identifiers with yours):
```sh
bcdedit /delete {96bba342-f6a2-11ef-b4fa-806e6f6e6963}
bcdedit /delete {9c165910-a6b8-11ef-b16f-90754eefef65}
bcdedit /delete {9c165911-a6b8-11ef-b16f-90754eefef65}
```

Once done, restart your system and check if the boot entries are removed.

---

## **Step 3: Clean EFI Partition (Optional, but Recommended)**
Even after removing boot entries, their files remain in the **EFI partition**. You need to delete them manually.

### **Mount the EFI Partition**
1. Open **Command Prompt as Administrator** and run:
   ```sh
   mountvol S: /S
   ```
   This mounts the EFI partition to drive `S:`.

### **Navigate to EFI Directory**
```sh
S:
cd EFI
```

### **Delete Unwanted Boot Folders**
Run these commands to remove Fedora, Kali, and Debian boot folders:
```sh
rmdir /S /Q Fedora
rmdir /S /Q Kali
rmdir /S /Q Debian
```

### **Unmount the EFI Partition**
```sh
mountvol S: /D
```

---

## **Step 4: Verify Changes**
1. Restart your computer.
2. Enter BIOS/UEFI (Press **F10** for HP laptops).
3. Check the **Boot Options** – Fedora, Kali, and Debian should be gone.
4. If Windows boots directly, the cleanup was successful.

---

## **Conclusion**
After following these steps, your system should boot directly into **Windows**, and all unnecessary Linux boot entries should be removed. This process ensures a clean bootloader without leftover files from removed operating systems.

