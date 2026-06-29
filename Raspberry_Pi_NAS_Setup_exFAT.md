# 📦 Raspberry Pi NAS Setup (Samba + External HDD)

A lightweight, reliable NAS setup using Raspberry Pi OS Lite, Samba, and an external HDD.

## 🧰 Prerequisites

- Raspberry Pi 4 (or similar)
- Raspberry Pi OS Lite (Debian Bookworm 12)
- External HDD (exFAT in this setup)
- Network connection (Ethernet recommended)

**System updated:**
```bash
sudo apt update && sudo apt upgrade -y
```

## 🧭 Step-by-Step Setup

### 1️⃣ Identify the HDD

**Command:**
```bash
lsblk -f
```

**Reason:**
- Identify correct device (`/dev/sda1`)
- Determine filesystem type

**Verify:**
- HDD appears with correct label and filesystem

---

### 2️⃣ Create Mount Point

```bash
sudo mkdir -p /mnt/hdd
```

**Reason:**
- Linux requires a directory to mount storage

**Verify:**
```bash
ls /mnt
```

---

### 3️⃣ Mount the HDD

**For exFAT:**
```bash
sudo mount -t exfat /dev/sda1 /mnt/hdd
```

**Reason:**
- Makes HDD accessible to system

**Verify:**
```bash
ls /mnt/hdd
```

---

### 4️⃣ Make Mount Persistent

**Get UUID:**
```bash
blkid /dev/sda1
```

**Edit fstab:**
```bash
sudo nano /etc/fstab
```

**Add:**
```
UUID=YOUR-UUID  /mnt/hdd  exfat  defaults,nofail,uid=1000,gid=1000,umask=0000  0  0
```

**Apply:**
```bash
sudo mount -a
```

**Reason:**
- Ensures auto-mount on boot
- Prevents manual mounting

**Verify:**
```bash
lsblk
```

**Reboot Test:**
```bash
sudo reboot
```

---

### 5️⃣ Install Samba

```bash
sudo apt install samba -y
```

**Reason:**
- Enables network file sharing (SMB protocol)

**Verify:**
```bash
systemctl status smbd
```

---

### 6️⃣ Configure Samba Share

**Edit config:**
```bash
sudo nano /etc/samba/smb.conf
```

**Add at bottom:**
```ini
[hdd]
   path = /mnt/hdd
   browseable = yes
   read only = no
   writable = yes
   guest ok = no
   valid users = YOUR_USERNAME
   create mask = 0777
   directory mask = 0777
   force user = YOUR_USERNAME
```

**Reason:**
- Defines network-accessible folder

---

### 7️⃣ Create Samba User

```bash
sudo smbpasswd -a YOUR_USERNAME
```

**Reason:**
- Authentication for secure access

---

### 8️⃣ Restart Samba

```bash
sudo systemctl restart smbd
```

**Verify:**
```bash
testparm
```

---

### 9️⃣ Access NAS

**Windows:**
```
\\<RASPBERRY_PI_IP>\hdd
```

**macOS / Linux:**
```
smb://<RASPBERRY_PI_IP>/hdd
```

**iOS:**
- Files app → Connect to Server

---

## ⚠️ Common Errors & Fixes

### ❌ "wrong fs type / bad superblock"

**Cause:**
- Corrupted or unsupported filesystem

**Fix:**
```bash
sudo fsck.exfat /dev/sda1
```

---

### ❌ "target is busy" (unmount issue)

**Fix:**
```bash
cd ~
sudo umount -l /mnt/hdd
```

---

### ❌ Read-only access from clients

**Causes:**
- Incorrect mount permissions
- Samba config missing write access

**Fix:**

Ensure `fstab`:
```
umask=0000
```

Ensure `Samba config`:
```ini
read only = no
force user = YOUR_USERNAME
```

---

### ❌ iOS read-only issue

**Cause:**
- iOS SMB client limitation

**Fix:**
- Reconnect share
- Use third-party apps (better SMB support)

---

### ❌ Samba not running

```bash
sudo systemctl start smbd
sudo systemctl enable smbd
```

---

### ❌ Unplugged or Restarted

```bash
sudo mount -a
```

---

## 🔐 Security Notes

- NAS is only accessible within local network
- **Do NOT expose SMB to internet**
- Use strong passwords
- Disable guest access (already done)

---

## ⚡ Reliability Notes

**Current setup (exFAT):**
- No journaling → risk during power loss

**Improvements:**
- Use ext4 filesystem (recommended)
- Use UPS for power backup

---

## 🚀 Future Improvements

### 🔧 System Enhancements
- Static IP configuration
- Automount recovery scripts
- Disk health monitoring (SMART)

### 📺 Media & Streaming
- Jellyfin (self-hosted Netflix)
- Plex media server

### ☁️ Personal Cloud
- Nextcloud (Google Drive alternative)
- File sync across devices

### 🔄 Backup Strategy
- Second HDD mirror
- Scheduled rsync backups
- Cloud backup integration

### 🔐 Remote Access
- VPN (WireGuard / Tailscale)
- Secure external access

### ⚡ Performance
- Samba tuning
- Gigabit Ethernet optimization

---

## 🧠 Summary

This setup provides:

- ✅ Lightweight NAS solution
- ✅ Full control over system
- ✅ Cross-platform compatibility
- ✅ Easy extensibility

---

## 📌 Notes

- Keep backups of critical data
- Always shutdown properly:
  ```bash
  sudo poweroff
  ```
- Avoid sudden power cuts

---
