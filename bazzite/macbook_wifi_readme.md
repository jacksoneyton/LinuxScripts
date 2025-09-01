# MacBook Pro WiFi Fix for Bazzite

## Problem Description

MacBook Pro users (especially 2015 models) running Bazzite often experience WiFi issues where:
- WiFi works during the Bazzite installation process
- After rebooting, the WiFi interface disappears completely  
- Manual intervention is required every boot: `modprobe brcmfmac` + `ujust configure-broadcom-wl`
- The issue persists across reboots and system updates

## Root Cause

The issue occurs because:
1. Bazzite includes a blacklist for `brcmfmac` in `/usr/lib/modprobe.d/broadcom-wl-blacklist.conf`
2. This blacklist prevents the Broadcom WiFi driver from loading at boot
3. The `ujust configure-broadcom-wl` command works temporarily but doesn't persist

## Solution

This automated script creates persistent overrides that:
- Override the `brcmfmac` blacklist
- Configure automatic module loading at boot
- Create a systemd service for proper WiFi initialization
- Include configurations in initramfs for persistence across updates
- Add kernel arguments for early driver loading

## Compatibility

**Tested on:**
- MacBook Pro Mid-2015 with BCM43602 WiFi chip
- Bazzite (Fedora Atomic Desktop based)

**Likely compatible with:**
- Other MacBook Pro models with Broadcom WiFi chips
- Other Universal Blue images with similar configurations

## Usage

### Quick Install & Run

```bash
# Download and run the script
curl -fsSL https://raw.githubusercontent.com/[your-repo]/macbook-wifi-fix.sh | bash
```

### Manual Download & Run

```bash
# Download the script
wget https://raw.githubusercontent.com/[your-repo]/macbook-wifi-fix.sh

# Make it executable  
chmod +x macbook-wifi-fix.sh

# Run the script
./macbook-wifi-fix.sh
```

## What the Script Does

1. **Detects your system** - Verifies you're running Bazzite and have Broadcom hardware
2. **Creates backups** - Backs up existing configurations before making changes
3. **Override blacklist** - Creates `/etc/modprobe.d/99-unblacklist-brcmfmac.conf`
4. **Module loading** - Creates `/etc/modules-load.d/macbook-wifi.conf`
5. **Systemd service** - Creates `macbook-wifi.service` for proper initialization
6. **Initramfs config** - Ensures persistence across system updates
7. **Kernel arguments** - Adds `rd.driver.pre=brcmfmac` for early loading

## After Running the Script

1. **Reboot required** - Changes take effect after reboot
2. **WiFi should work automatically** - No more manual intervention needed
3. **Persists across updates** - Configuration survives Bazzite system updates

## Verification

After reboot, verify the fix is working:

```bash
# Check if brcmfmac module is loaded
lsmod | grep brcmfmac

# Check for WiFi interface
ip link show | grep wl

# Check systemd service status
systemctl status macbook-wifi.service

# Verify no deny-listed messages
systemctl status systemd-modules-load
```

## Troubleshooting

### WiFi still not working after reboot?

1. Check if the systemd service ran successfully:
   ```bash
   systemctl status macbook-wifi.service
   journalctl -u macbook-wifi.service
   ```

2. Check if the module loaded:
   ```bash
   lsmod | grep brcmfmac
   ```

3. Manually run the commands:
   ```bash
   sudo modprobe brcmfmac
   ujust configure-broadcom-wl
   ```

### Need to undo the changes?

The script creates backups in `~/.macbook-wifi-fix-backup-[timestamp]/`. You can:

1. Disable the systemd service:
   ```bash
   sudo systemctl disable macbook-wifi.service
   sudo rm /etc/systemd/system/macbook-wifi.service
   ```

2. Remove the configuration files:
   ```bash
   sudo rm /etc/modprobe.d/99-unblacklist-brcmfmac.conf
   sudo rm /etc/modules-load.d/macbook-wifi.conf
   sudo rm /etc/dracut.conf.d/macbook-wifi-fix.conf
   ```

3. Remove kernel argument:
   ```bash
   rpm-ostree kargs --delete="rd.driver.pre=brcmfmac"
   ```

## Contributing

Found this helpful? Have suggestions or improvements? Please contribute back to the Bazzite community!

## License

MIT License - Feel free to modify and redistribute.

---

**Note**: This fix addresses a specific interaction between Bazzite's Broadcom driver management and MacBook Pro hardware. It may not be necessary for all Broadcom WiFi issues.