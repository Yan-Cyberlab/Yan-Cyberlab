# VMware Workstation Pro — Complete VM Setup Guide for Cybersecurity Labs

> **Author:** u/yan-cyberlab  
> **Date:** March 2026  
> **Purpose:** Step-by-step reference for setting up virtual machines in VMware Workstation Pro for cybersecurity learning  
> **VMware Version:** Workstation Pro 25H2 (free for all users since Nov 2024)

---

## Table of Contents

1. [Why VMware for Cybersecurity?](#1-why-vmware-for-cybersecurity)
2. [Pre-Setup: Host System Requirements](#2-pre-setup-host-system-requirements)
3. [Installing VMware Workstation Pro](#3-installing-vmware-workstation-pro)
4. [Creating a New Virtual Machine — Every Option Explained](#4-creating-a-new-virtual-machine--every-option-explained)
5. [Hardware Settings Deep Dive](#5-hardware-settings-deep-dive)
6. [Network Adapter Types — Critical for Security Labs](#6-network-adapter-types--critical-for-security-labs)
7. [Snapshots — Your Safety Net](#7-snapshots--your-safety-net)
8. [Cloning VMs](#8-cloning-vms)
9. [Recommended VM Configurations for Cybersecurity](#9-recommended-vm-configurations-for-cybersecurity)
10. [Troubleshooting Common Issues](#10-troubleshooting-common-issues)

---

## 1. Why VMware for Cybersecurity?

VMware Workstation Pro is the industry standard for desktop virtualization. In enterprise environments, VMware holds over 42% of the global virtualization market share (2026), with on-premises dominance exceeding 70%. Learning VMware means learning the tool your future employers are likely running.

**Key advantages for cybersecurity labs:**
- **Snapshot Manager** — Save and restore VM states instantly (essential for malware testing and CTF challenges)
- **Advanced networking** — Create isolated virtual networks for attack/defend scenarios
- **Multi-VM support** — Run attacker (Kali), target (Ubuntu/Windows), and monitoring (SIEM) VMs simultaneously
- **Clone support** — Create full or linked clones to save disk space
- **Better performance** — Generally faster than VirtualBox when running multiple VMs

---

## 2. Pre-Setup: Host System Requirements

Before creating any VM, check these requirements on your host machine:

### Minimum Requirements
| Component | Minimum | Recommended for Cyber Labs |
|-----------|---------|---------------------------|
| CPU | 64-bit, 2+ cores | 4+ cores (i5/Ryzen 5 or better) |
| RAM | 8 GB | 16 GB+ (you'll run 2-4 VMs) |
| Disk | 50 GB free | 200 GB+ SSD (VMs are disk-hungry) |
| Virtualization | VT-x/AMD-V enabled in BIOS | Same |
| OS | Windows 10/11 64-bit or Linux | Same |

### Critical: Enable Hardware Virtualization

1. Restart your computer and enter BIOS/UEFI (usually `F2`, `F12`, `Del`, or `Esc` during boot)
2. Find **Intel VT-x** or **AMD-V** (may be under "Advanced" → "CPU Configuration")
3. **Enable** it
4. Save and exit

> **How to check if it's enabled (Windows):** Open Task Manager → Performance → CPU → look for "Virtualization: Enabled"

### Critical: Hyper-V Conflict (Windows Only)

VMware and Windows Hyper-V can conflict. VMware can run with Hyper-V enabled but performance drops because VMs use Windows Hypervisor Platform instead of VMware's native hypervisor.

**To disable Hyper-V for best performance:**
```
# Run in PowerShell as Administrator
bcdedit /set hypervisorlaunchtype off
# Restart your computer
```

**To re-enable later:**
```
bcdedit /set hypervisorlaunchtype auto
```

---

## 3. Installing VMware Workstation Pro

### Download
1. Go to [Broadcom Support Portal](https://support.broadcom.com)
2. Register for a free Broadcom account (only needs email)
3. After login, navigate to: VMware Cloud Foundation → VMware Workstation Pro → "For Personal Use"
4. Download the latest version (25H2 or newer)

### Installation Options Explained

When running the installer, you'll encounter these choices:

| Option | What It Does | Recommendation |
|--------|-------------|----------------|
| **Install Windows Hypervisor Platform (WHP)** | Allows VMware to coexist with Hyper-V. Reduces performance but avoids manual Hyper-V disabling | **Skip this** — disable Hyper-V manually for best performance |
| **Enhanced Keyboard Driver** | Better key handling for VMs (e.g., special key combos) | **Skip** — not needed for most Linux VMs |
| **Add to system PATH** | Adds VMware CLI tools to your PATH | **Enable** — useful for `vmrun` commands later |
| **Check for updates on startup** | Auto-checks for new versions | Your choice |
| **Join CEIP** | Sends anonymous usage data to VMware | Your choice |

### First Launch
- Select **"Use VMware Workstation for Personal Use"**
- Click **Continue** → **Finish**
- No license key needed

---

## 4. Creating a New Virtual Machine — Every Option Explained

Click **"Create a New Virtual Machine"** from the home screen. You'll immediately face your first choice:

### Step 1: Configuration Type

| Option | What It Means | When to Use |
|--------|--------------|-------------|
| **Typical (recommended)** | Guided wizard with sensible defaults. Fewer questions, faster setup. | **Use this for your first VMs** — Kali, Ubuntu, etc. |
| **Custom (advanced)** | Full control over every hardware parameter: SCSI controller type, disk type, hardware compatibility level, etc. | Use when you need specific configs (e.g., older OS, nested virtualization, specific hardware version) |

> **For cybersecurity learning:** Start with **Typical**. You can always change hardware settings after creation.

### Step 2: Installer Disc / ISO Source

| Option | What It Means | When to Use |
|--------|--------------|-------------|
| **Installer disc** | Use a physical CD/DVD drive | Almost never in 2026 |
| **Installer disc image file (ISO)** | Point to an `.iso` file on your computer | **This is what you'll use 99% of the time** |
| **I will install the operating system later** | Creates a blank VM with no OS. You attach the ISO later. | Use when you want to configure hardware first, then install |

**For Kali Linux:**
1. Download the ISO from [kali.org/get-kali](https://www.kali.org/get-kali/) — choose **"Bare Metal"** → **"64-bit Installer"**
2. Select "Installer disc image file" and browse to your downloaded `.iso`

> **Important:** VMware may detect the OS type automatically ("Easy Install"). For Kali, it may offer to auto-configure. **Decline Easy Install** for Kali — it often misconfigures things. Choose "I will install the operating system later" and attach the ISO manually if Easy Install activates.

### Step 3: Guest Operating System Selection

This tells VMware how to optimize the VM's virtual hardware:

| Selection | Effect |
|-----------|--------|
| **Linux → Debian 11.x/12.x 64-bit** | Best match for Kali Linux (Kali is Debian-based) |
| **Linux → Ubuntu 64-bit** | For Ubuntu Server/Desktop VMs |
| **Windows → Windows Server 2022** | For your Active Directory lab later |
| **Other → Other 64-bit** | Generic — use only if your OS isn't listed |

> **Why this matters:** VMware tunes virtual hardware (mouse driver, graphics adapter, kernel parameters) based on this selection. Choosing the wrong OS type won't break anything, but may cause minor driver or performance issues.

### Step 4: Virtual Machine Name & Location

| Field | What It Means | Best Practice |
|-------|--------------|---------------|
| **Virtual machine name** | Display name in VMware's sidebar | Use descriptive names: `Kali-Attacker`, `Ubuntu-Target`, `Win-Server-AD` |
| **Location** | Folder where all VM files (.vmx, .vmdk, etc.) are stored | Create a dedicated folder like `D:\VMs\CyberLab\` — keep all lab VMs together |

> **Tip:** A single VM can use 20-80 GB of disk space. Store VMs on your largest/fastest drive (SSD preferred).

### Step 5: Disk Size & Storage Type

| Option | What It Means | Recommendation |
|--------|--------------|----------------|
| **Maximum disk size** | The max the virtual disk CAN grow to (not immediately allocated) | **Kali:** 80 GB, **Ubuntu Server:** 40 GB, **Windows Server:** 60 GB |
| **Store virtual disk as a single file** | One large `.vmdk` file | **Best for performance on SSD.** Faster I/O. |
| **Split virtual disk into multiple files** | Multiple 2 GB `.vmdk` files | Better if you need to move VMs to FAT32 drives or older file systems. Slightly slower. |

> **Allocate all disk space now?** (appears in Custom mode)
> - **Yes (pre-allocate):** Creates the full-size file immediately. Slightly better performance. Uses all disk space upfront.
> - **No (thin provisioning):** File starts small and grows as needed. Saves space. **Use this** — your lab VMs won't need full disk immediately.

### Step 6: Review & "Customize Hardware" (IMPORTANT)

Before clicking "Finish," click **"Customize Hardware..."** — this is where the real configuration happens. See Section 5 below.

---

## 5. Hardware Settings Deep Dive

After creating a VM (or by right-clicking → Settings), you can configure:

### Memory (RAM)

| VM Type | Minimum | Recommended |
|---------|---------|-------------|
| Kali Linux | 2 GB | 4 GB |
| Ubuntu Server (headless) | 1 GB | 2 GB |
| Ubuntu Desktop | 2 GB | 4 GB |
| Windows Server + AD | 4 GB | 6-8 GB |
| SIEM (Splunk/Wazuh) | 4 GB | 8 GB |

> **Rule of thumb:** Total VM RAM should not exceed 75% of your host RAM. If you have 16 GB host, keep total VM allocation under 12 GB.

### Processors

| Setting | What It Means |
|---------|--------------|
| **Number of processors** | Number of CPU sockets (virtual). Keep at **1**. |
| **Number of cores per processor** | Cores assigned to this VM. **2 cores** is good for most VMs. Kali with heavy tools: **4 cores**. |
| **Virtualize Intel VT-x/EPT or AMD-V/RVI** | Enables nested virtualization (running VMs inside VMs). **Leave off** unless specifically needed. |
| **Virtualize IOMMU** | For GPU passthrough. **Leave off**. |
| **Side channel mitigations** | Protects against Spectre/Meltdown in VMs. Slight performance cost. **Leave at default**. |

### Hard Disk (NVMe vs SCSI vs SATA)

In **Custom** mode, you choose the disk controller:

| Controller | Speed | Compatibility | Use When |
|-----------|-------|---------------|----------|
| **NVMe** | Fastest | Modern OS only (Win 10+, recent Linux) | **Default for Kali, Ubuntu** |
| **SCSI (LSI Logic)** | Fast | Broad compatibility | Good general choice |
| **SATA** | Moderate | Maximum compatibility | Older OS or troubleshooting |
| **IDE** | Slowest | Legacy OS | Only for Windows XP/old Linux |

### CD/DVD (SATA)

| Setting | Meaning |
|---------|---------|
| **Use ISO image file** | Mounts an ISO as a virtual DVD. **Set this to your Kali/Ubuntu ISO for installation.** |
| **Use physical drive** | Reads from actual DVD drive. Rarely needed. |
| **Connect at power on** | Auto-mounts when VM starts. **Enable this** during installation, disable after. |

### Display

| Setting | What It Does | Recommendation |
|---------|-------------|----------------|
| **Accelerate 3D graphics** | Enables GPU acceleration in the VM | **Enable** for Desktop environments (Kali, Ubuntu Desktop). Disable for headless servers. |
| **Graphics memory** | VRAM allocated | **256 MB** for desktops, leave default for servers |
| **Monitor resolution** | VM screen resolution | **Use host settings** or set specific (1920x1080) |

### USB Controller

| Version | Speed | Recommendation |
|---------|-------|----------------|
| **USB 2.0** | 480 Mbps | Default, compatible with everything |
| **USB 3.1** | 10 Gbps | Choose if you'll use USB WiFi adapters for wireless pen testing |

> **For cybersecurity:** If you plan to use a USB WiFi adapter (like Alfa AWUS036ACH) for wireless hacking labs, you'll need USB 3.1 and to connect the device to the VM via **VM → Removable Devices**.

---

## 6. Network Adapter Types — Critical for Security Labs

This is the **most important setting** for cybersecurity labs. VMware offers these network modes:

### Network Types Explained

| Type | How It Works | Internet? | Visible to Host Network? | Use Case |
|------|-------------|-----------|------------------------|----------|
| **Bridged** | VM gets its own IP on your physical network (like a real device) | Yes | Yes — other devices can see it | When you need the VM to appear as a real device on your network |
| **NAT** | VM shares host's IP via VMware's virtual NAT. VM can reach internet but isn't directly visible on your network | Yes | No — hidden behind NAT | **Default and safest for learning.** Your Kali VM can reach the internet for updates without exposing itself |
| **Host-Only** | VM can only talk to the host and other VMs on the same host-only network. No internet access. | No | No | **Isolated attack/defend labs.** Put your attacker (Kali) and target (Ubuntu) on the same host-only network |
| **Custom (VMnet)** | Connect to a specific virtual switch (VMnet0-VMnet19) | Depends on VMnet config | Depends | Advanced multi-network lab setups |
| **LAN Segment** | Private network between VMs. No host access at all. | No | No | Maximum isolation for malware analysis |

### Recommended Lab Network Setup

For your cybersecurity home lab (Month 4), use this architecture:

```
┌─────────────────────────────────────────────┐
│              Your Physical Network          │
│              (Home WiFi/Ethernet)           │
└──────────────┬──────────────────────────────┘
               │ Bridged (VMnet0)
               │
       ┌───────┴───────┐
       │   pfSense VM  │    ← Virtual firewall/router
       │   (Gateway)   │
       └───────┬───────┘
               │ Host-Only (VMnet1) - "Lab Network"
               │
    ┌──────────┼──────────┐
    │          │          │
┌───┴───┐  ┌───┴───┐  ┌───┴───┐
│ Kali  │  │Ubuntu │  │Windows│
│Attack │  │Target │  │Server │
└───────┘  └───────┘  └───────┘
  All on VMnet1 (Host-Only)
```

**Why this matters:**
- **Kali on Host-Only** = Your attacks stay inside the lab (you won't accidentally scan your neighbor's network)
- **pfSense bridges** the lab to the internet when VMs need updates
- **Isolated network** = Safe to run Metasploit, Nmap, etc. without legal risk

### How to Configure Virtual Networks

**VMware menu → Edit → Virtual Network Editor:**
- **VMnet0** = Bridged (auto-mapped to your physical adapter)
- **VMnet1** = Host-Only (default: 192.168.206.0/24) — **your lab network**
- **VMnet8** = NAT (default: 192.168.153.0/24)
- **VMnet2-7, 9-19** = Custom (create your own)

You can change subnet, DHCP range, and other settings here.

---

## 7. Snapshots — Your Safety Net

Snapshots save the **exact state** of a VM at a point in time — memory, disk, settings, everything.

### When to Take Snapshots

| When | Why |
|------|-----|
| **After fresh OS install** | Clean baseline to revert to |
| **Before installing new tools** | Roll back if something breaks |
| **Before running exploits/malware** | Restore to clean state after testing |
| **Before major config changes** | Undo network or service changes |
| **Before CTF challenges** | Reset between challenges |

### How to Use

- **Take snapshot:** `VM → Snapshot → Take Snapshot` (or `Ctrl+Shift+S`)
- **Revert to snapshot:** `VM → Snapshot → Revert to Snapshot`
- **Snapshot Manager:** `VM → Snapshot → Snapshot Manager` — see your entire snapshot tree

### Naming Convention

Use descriptive names:
```
Kali-Attacker
  ├── "Fresh Install - No Tools"
  ├── "Base Tools Installed (nmap, burp, etc.)"
  ├── "Month 5 - Pre-Vuln-Scan"
  │     └── "Month 5 - Post-Vuln-Scan"
  └── "Month 6 - CTF Ready"
```

> **Disk space warning:** Each snapshot stores changes since the last snapshot. Over time, snapshots can consume significant disk space. Delete old snapshots you no longer need via Snapshot Manager → Delete.

---

## 8. Cloning VMs

### Full Clone vs Linked Clone

| Type | What It Does | Disk Usage | Speed | Independence |
|------|-------------|------------|-------|-------------|
| **Full Clone** | Complete independent copy of the VM | High (duplicates everything) | Slow to create | Fully independent — can move/delete parent |
| **Linked Clone** | Shares base disk with parent, stores only changes | Low (only stores differences) | Fast to create | Depends on parent — cannot delete parent VM |

### When to Clone

- **Full Clone:** When you want to create a completely separate VM (e.g., make a second target machine)
- **Linked Clone:** When you want multiple similar VMs quickly (e.g., 3 Windows clients for AD lab) and want to save disk space

**How:** Right-click VM → Manage → Clone → Follow wizard

---

## 9. Recommended VM Configurations for Cybersecurity

### VM 1: Kali Linux (Attacker Machine)

| Setting | Value |
|---------|-------|
| OS Type | Linux → Debian 12.x 64-bit |
| RAM | 4 GB |
| CPU | 2 cores |
| Disk | 80 GB, single file, thin provisioned |
| Network 1 | NAT (for internet/updates) |
| Network 2 | Host-Only VMnet1 (for lab) |
| 3D Acceleration | Enabled |
| ISO | kali-linux-2025.x-installer-amd64.iso |

### VM 2: Ubuntu Server (Target Machine)

| Setting | Value |
|---------|-------|
| OS Type | Linux → Ubuntu 64-bit |
| RAM | 2 GB |
| CPU | 2 cores |
| Disk | 40 GB, single file, thin provisioned |
| Network | Host-Only VMnet1 only (isolated) |
| 3D Acceleration | Disabled (headless) |
| ISO | ubuntu-24.04-live-server-amd64.iso |

### VM 3: Windows Server (Active Directory)

| Setting | Value |
|---------|-------|
| OS Type | Windows → Windows Server 2022 |
| RAM | 6 GB |
| CPU | 2 cores |
| Disk | 60 GB, single file, thin provisioned |
| Network | Host-Only VMnet1 only |
| 3D Acceleration | Disabled |
| ISO | Windows Server 2022 Evaluation ISO (free 180-day trial from Microsoft) |

### VM 4: pfSense (Firewall/Router) — Month 4

| Setting | Value |
|---------|-------|
| OS Type | Other → FreeBSD 14 64-bit |
| RAM | 1 GB |
| CPU | 1 core |
| Disk | 10 GB, single file |
| Network 1 | Bridged (WAN — connects to your real network) |
| Network 2 | Host-Only VMnet1 (LAN — your lab network) |
| ISO | pfSense CE ISO from pfsense.org |

---

## 10. Troubleshooting Common Issues

### "Intel VT-x/EPT is not supported" or "This host does not support Intel VT-x"
**Fix:** Enter BIOS → Enable VT-x/AMD-V. See Section 2.

### VM is very slow / laggy
**Possible causes:**
- Hyper-V still enabled → Disable it (Section 2)
- Too much RAM allocated to VMs → Reduce total VM RAM to under 75% of host
- VM stored on HDD not SSD → Move VM folder to SSD

### "Could not open /dev/vmmon" (Linux hosts)
**Fix:** Re-run VMware kernel modules: `sudo vmware-modconfig --console --install-all`

### Network not working in VM
- Check adapter type (NAT/Bridged/Host-Only) matches your intended setup
- Check **Virtual Network Editor** that the VMnet is active
- Inside VM: run `ip a` (Linux) or `ipconfig` (Windows) to verify IP assignment
- Try: `VM → Settings → Network Adapter → check "Connect at power on"`

### "VMware Workstation and Hyper-V are not compatible"
**Fix:** Disable Hyper-V (Section 2) OR check "Install Windows Hypervisor Platform" during VMware setup (but accept reduced performance)

---

## File Structure for GitHub

When you push this to your GitHub repository, use this structure:

```
yan-cyberlab/
├── README.md                    ← Overview of your cybersecurity journey
├── month-01-linux/
│   ├── vmware-setup-guide.md    ← THIS DOCUMENT
│   ├── kali-install-notes.md
│   ├── screenshots/
│   │   ├── vmware-new-vm-wizard.png
│   │   ├── network-config.png
│   │   └── kali-first-boot.png
│   └── linux-hardening-script/
│       ├── hardening.sh
│       └── README.md
├── month-02-networking/
│   └── ...
└── ...
```

> **Tip:** Take screenshots at every step as you set up your VMs. This makes your GitHub documentation much more valuable to other learners — and to future employers reviewing your portfolio.

---

## Resources

- [VMware Workstation Pro Documentation](https://docs.vmware.com/en/VMware-Workstation-Pro/index.html)
- [Kali Linux Downloads](https://www.kali.org/get-kali/)
- [Ubuntu Server Downloads](https://ubuntu.com/download/server)
- [Windows Server Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)
- [pfSense Community Edition](https://www.pfsense.org/download/)

---

*This guide is part of my 9-month cybersecurity career launch journey. Follow along on Reddit: u/yan-cyberlab. Guide is generated with the help of ClaudeAI.*
