# Shared Clipboard Not Working (VirtualBox – Ubuntu Desktop)

## Problem /Context

The **Shared Clipboard**feature in VirtualBox does not work between the host (Windows/macOS/Linux) and the Ubuntu Desktop guest. Copy-pasting text or files between the host and virtual machine fails even though the *Bidirectional*option has been activated via the **Devices → Shared Clipboard**menu.

General symptoms:
-The clipboard only works one way, or doesn't work at all.
-No response after re-enabling Shared Clipboard option from VM menu.
-The problem reappeared after the VM was restarted.

## Root Cause

Failure of this feature is generally caused by one (or a combination) of the following factors:

1. **Guest Additions is not installed or its version is out of sync**with the host VirtualBox version — Shared Clipboard depends entirely on the kernel module and the `VBoxClient` service of Guest Additions.
2. **The `VBoxClient --clipboard` service is not running**in the guest desktop session, so the host-guest clipboard connecting daemon is not active.
3. **Shared Clipboard mode has not been set to *Bidirectional***in the VM configuration (either via GUI or `VBoxManage`).
4. **Kernel guest additions module failed to rebuild**after Ubuntu kernel update, so VM integration was silently broken.

## Resolution /Steps

### 1. Make sure Shared Clipboard Mode is Active

Via VirtualBox GUI:

```
Devices → Shared Clipboard → Bidirectional
```

Or via command line on the host (VM must be powered off):

```bash
VBoxManage Modifyvm "VM name" --clipboard-mode Bidirectional
```

### 2. Verify Guest Additions Installation

Inside the Ubuntu guest, check that the Guest Additions are installed and that the version is correct:

```bash
modinfo Vboxguest | grep Version
```

If it is not installed or is outdated, install/reinstall via image Guest Additions:

```bash
sudo Apt Update
sudo Apt Install Build essential Dkms Linux headers$(uname -r)
sudo Mount /Dev/cdrom /Min
sudo /Mnt/v box linux additions.run
```

Restart the VM once the installation is complete.

### 3. Make sure the `VBoxClient` Service is Running

Check whether the clipboard service process is active in the guest session:

```bash
ps To | grep V box client
```

If the `VBoxClient --clipboard` process does not appear, run it manually:

```bash
VBoxClient --clipboard
```
To ensure this service runs automatically at every login, add it to startup applications:

```bash
echo "VBoxClient --clipboard" >> ~/.config/autostart/vboxclient.desktop
```

### 4. Rebuild Kernel Modules (If Problems Appear After Kernel Update)

```bash
sudo /Sbin/rcvboxadd Setup
```

Restart the VM after the rebuild process is complete.

### 5. Final Verification
After all the steps above, restart the VM completely (not just re-login), then test simple text copy-paste from host to guest and vice versa.

---

**Note:**If the problem persists, check the VirtualBox log (`VBox.log`) in the VM directory for errors related to `vboxguest` or `vboxsf` modules.