# Step-by-Step Guide: Replacing PSBBN Autoboot with the New Dashboard

This guide provides step-by-step instructions to configure your PlayStation 2 to autoboot into this custom modern dashboard instead of PSBBN, while keeping an always-on FTP server active in the background.

---

## 1. Required Release Files & Downloads

| Component | Purpose | Download Link |
|---|---|---|
| **`wLaunchELF` (wLE)** | PS2 File Manager & ELF launcher | [wLaunchELF Releases](https://github.com/ps2homebrew/wLaunchELF/releases) |
| **`ps2ftpd` / `ftpd.irx`** | Always-on background FTP server for PS2 | [ps2sdk-ports / ps2ftpd](https://github.com/ps2dev/ps2sdk-ports) |
| **`Open PS2 Loader` (OPL)** | High-performance game launcher backend | [OPL Official Releases](https://github.com/ps2homebrew/Open-PS2-Loader/releases) |
| **`Free McBoot Configurator`** | Autoboot & launch path manager | Included in [FMCB / FHDB](https://github.com/israpps/FreeMcBoot-Installer/releases) |
| **`pfsshell`** *(PC tool)* | Direct file transfer to PS2 HDD partitions | [pfsshell Releases](https://github.com/ps2homebrew/pfsshell/releases) |

---

## 2. Directory Structure on PS2

Place the compiled dashboard binary and UI blob in your preferred boot location:

### Option A: Memory Card (`mc0:`)
```
mc0:/
└── APPS/
    └── DASHBOARD/
        ├── BOOT.ELF          <-- Dashboard executable
        ├── ui.uib            <-- Compiled OPHTML UI blob (from build/ui.uib)
        ├── IPCONFIG.DAT      <-- Static IP settings (if not using DHCP)
        └── ftpd.irx          <-- FTP daemon IOP module
```

### Option B: Internal HDD (`hdd0:`)
```
hdd0:/__sysconf/
└── DASHBOARD/
    ├── BOOT.ELF
    ├── ui.uib
    └── ftpd.irx
```

---

## 3. Step-by-Step Autoboot Setup

### Method 1: Using FMCB / FHDB Configurator (Cleanest & Safest)

1. **Boot your PS2** into FMCB / FHDB or launch **`wLaunchELF`**.
2. Launch **`Free McBoot Configurator`**.
3. Select **`Configure E1 launch keys...`** (or **`Configure OSDSYS options...`**).
4. Select **`Auto`** (the default boot application on power-up).
5. Navigate and set the path to:
   - `mc0:/APPS/DASHBOARD/BOOT.ELF` (or `hdd0:/__sysconf/DASHBOARD/BOOT.ELF`).
6. Set **`E1` / `E2`** failover paths if desired.
7. Return to the main menu and select **`Save CNF to MC0`** (or `Save CNF to HDD`).
8. Power cycle the PS2. It will now boot directly into the modern dashboard.

---

### Method 2: Replacing PSBBN's `osd100.elf` Entry Point

If your PS2 boots directly into PSBBN from the HDD `__system` partition without FMCB:

1. Launch **`wLaunchELF`** on your PS2.
2. In the FileBrowser, navigate to `hdd0:/__system/osd/` (or `hdd0:/__system/sys/`).
3. Locate `osd100.elf` (PSBBN's boot binary).
4. **Rename `osd100.elf` to `osd100_bbn.elf`** (this creates a backup so you never lose PSBBN).
5. Copy your new `BOOT.ELF` and `ui.uib` into `hdd0:/__system/osd/`.
6. **Rename `BOOT.ELF` to `osd100.elf`**.
7. Restart your console. The PS2 will load your new dashboard on power on.
8. *(Optional)*: Add a shortcut in the dashboard's **Tools** menu pointing to `hdd0:/__system/osd/osd100_bbn.elf` to launch back into PSBBN anytime.

---

## 4. Always-On FTP Server Setup & Connection

The dashboard runs `ps2ftpd` on the IOP processor in the background while displaying your network information in the sidebar:

### Default Connection Credentials:
- **Host / IP**: The IP shown in your dashboard sidebar (e.g. `192.168.1.150` via DHCP or `IPCONFIG.DAT`)
- **Port**: `21`
- **Username**: `ps2` (or `anonymous`)
- **Password**: *(leave blank / empty)*
- **Protocol**: `FTP - File Transfer Protocol` (Plain / unencrypted)
- **Transfer Mode**: `Passive`

### Configuring Static IP (Optional)
If your network does not use DHCP, place an `IPCONFIG.DAT` file alongside `BOOT.ELF`:
```
192.168.1.150 255.255.255.0 192.168.1.1
```
*(Format: `<IP_ADDRESS> <NETMASK> <GATEWAY>`)*

---

## 5. Testing & Compiling the UI Blob

Whenever you make changes to the HTML/CSS in this repository:

1. **Build the binary blob**:
   ```bash
   ps2ui build ps2ui.json
   ```
2. **Verify compliance (100% strict check)**:
   ```bash
   ps2ui-check build/ui.uib
   ```
3. **Transfer `build/ui.uib`** to your PS2 via FTP or USB drive.
