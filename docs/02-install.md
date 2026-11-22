# 2. Installing OPNsense

This step installs OPNsense onto the SSD. Keep every Ethernet cable unplugged from the ZimaBoard until step 3: the installer does not need a network.

Tested with the **OPNsense 26.7** image, which step 3 updates to the latest 26.7.x release. Newer images work the same way; replace `26.7` in file names with the version you download.

## 2.1 Download the installer

On your everyday computer:

1. Go to [opnsense.org/download](https://opnsense.org/download/).
2. Choose architecture **amd64**, image type **vga**, and a mirror near you.
3. Download the image, for example `OPNsense-26.7-vga-amd64.img.bz2`.

The **vga** image is the one for a USB stick on a machine with a screen. It boots on UEFI systems such as the ZimaBoard 2. (The **dvd** image is an ISO for optical drives and virtual machines; **serial** and **nano** are for other hardware.)

Install the newest major release you can find. Step 3 updates it to the latest patch release.

## 2.2 Check the download

A checksum proves the file arrived complete and is the one the OPNsense project published. Compute it, then compare it with the SHA-256 value for the same `.img.bz2` file in the [release announcement](https://docs.opnsense.org/releases/CE_26.7.html) or the checksum file on the mirror.

Windows (PowerShell, in the folder with the download):

```powershell
Get-FileHash -Algorithm SHA256 .\OPNsense-26.7-vga-amd64.img.bz2
```

macOS:

```bash
shasum -a 256 OPNsense-26.7-vga-amd64.img.bz2
```

Linux:

```bash
sha256sum OPNsense-26.7-vga-amd64.img.bz2
```

If the values differ, download the file again from another mirror. Do not use an image whose checksum does not match.

**Optional, stronger check.** The mirror also carries a signature file (`.sig`) for the image and the project's public key (`.pub`). On macOS or Linux, the [OPNsense manual](https://docs.opnsense.org/manual/install.html) verifies them as below. Run it on the still-compressed `.bz2` file, and put the names of the files you downloaded in place of the placeholders:

```bash
openssl base64 -d -in <image>.bz2.sig -out /tmp/image.sig
openssl dgst -sha256 -verify <key>.pub -signature /tmp/image.sig <image>.bz2
```

The output should be `Verified OK`.

## 2.3 Unpack the image

The download is compressed and must be unpacked into a plain `.img` file before writing.

- **Windows:** right-click the file and choose **WinRAR → Extract Here**. If you don't see the WinRAR entries, look under **Show more options**. You can also double-click the file to open it in [WinRAR](https://www.win-rar.com/) and click **Extract To**. If the file doesn't open in WinRAR, tick **BZ2** under **Options → Settings → Integration** and try again.
- **macOS and Linux:**

  ```bash
  bzip2 -d OPNsense-26.7-vga-amd64.img.bz2
  ```

You now have `OPNsense-26.7-vga-amd64.img`.

## 2.4 Write the image to a USB drive

Writing the image **erases everything on the USB drive.** Use an empty one, and double-check you have picked the USB drive and not another disk.

Copying the file onto the drive does not work. It has to be written with an image tool:

**Windows: [balenaEtcher](https://etcher.balena.io/)**

1. Plug in the USB drive and open balenaEtcher.
2. **Flash from file**, then pick the unpacked `.img` file from 2.3, not the `.bz2`.
3. **Select target**: your USB drive. Check the name and size before you continue.
4. **Flash!** If Windows asks for administrator permission, allow it. Etcher writes the image and then checks it; wait until it says the flash is complete.
5. If Windows then says the drive needs to be formatted, click **Cancel**. Windows can't read the OPNsense file system, and the drive is fine.

**macOS**

```bash
diskutil list                          # find the USB drive, e.g. /dev/disk4
diskutil unmountDisk /dev/diskN        # replace N with your drive's number
sudo dd if=OPNsense-26.7-vga-amd64.img of=/dev/rdiskN bs=1m
diskutil eject /dev/diskN
```

**Linux**

```bash
lsblk                                  # find the USB drive, e.g. /dev/sdb
sudo dd if=OPNsense-26.7-vga-amd64.img of=/dev/sdX bs=4M status=progress conv=fsync
```

Replace `diskN` or `sdX` with your USB drive. Pointing `dd` at the wrong disk overwrites it without warning.

balenaEtcher also runs on macOS and Linux, if you'd rather use it than `dd`. The steps are the same as on Windows.

## 2.5 Boot the installer

1. With the board unplugged, insert the USB drive. Keyboard and screen stay connected; Ethernet stays unplugged.
2. Plug in the power and tap **F11** to open the boot menu. It's a blue box titled **Please select boot device:** with three entries:
   - `UEFI OS`: ZimaOS on the board's built-in eMMC. Don't pick this one.
   - `UEFI: <your USB drive's name>, Partition 1`: the OPNsense installer.
   - `Enter Setup`: the firmware setup.
3. Use the arrow keys to select the line that starts with `UEFI:` and shows your USB drive's name, then press **Enter**.

Your SSD isn't in this list yet. That's expected: there's nothing on it to boot until step 2.6.

**Don't press Esc while this menu is open.** In this menu, Esc boots the default entry, which is ZimaOS. If you need the firmware setup, choose **Enter Setup** instead.

If F11 shows nothing at all, restart and tap **Esc** as soon as the power connects to open the firmware setup, go to the **Save & Exit** tab, and choose the USB drive under **Boot Override**.

OPNsense starts a live system from the USB drive. This takes a minute or two. When the login prompt appears, log in as:

- user: `installer`
- password: `opnsense`

## 2.6 Install to the SSD

The installer is a text menu: arrow keys to move, **Space** to select, **Enter** to confirm. Labels can differ slightly between releases.

1. **Keymap.** Keep the default (US) or choose your layout, then continue.
2. **Install (ZFS).** ZFS is the recommended choice and copes better with sudden power loss. *Install (UFS)* also works if you prefer it.
3. **Pool type: stripe.** One disk, no redundancy. That is expected with a single SSD.
4. **Choose the disk.** This is the step that matters. Pick the SSD, going by its size: budget drives often report a generic model name such as `SATA SSD`. On FreeBSD:
   - a SATA SSD is usually **`ada0`**
   - the onboard eMMC is usually **`mmcsd0`**
   - the USB installer is usually **`da0`**

   Selecting the eMMC erases the factory ZimaOS.
5. **Confirm** the final warning. The installer erases the SSD and copies OPNsense to it.
6. **Root Password.** When the install finishes, choose *Root Password* and set a strong password. Store it in a password manager. This password unlocks both the console and the web interface.
7. **Complete Install.** The board reboots. Pull the USB drive out as it restarts.

## 2.7 Make the SSD the first boot device

If the board comes back up in ZimaOS instead of OPNsense, the eMMC is still first in the boot order:

1. Restart (**Ctrl+Alt+Delete** at the ZimaOS screen) and tap **Esc**.
2. On the **Boot** tab, select **Boot Option #1** and choose the SSD's entry.
3. On **Save & Exit**, choose **Save Changes and Exit**.

The eMMC's entry is called **UEFI OS**, and the SSD's entry may carry a similar generic name. If you can't tell them apart, use the **F11** boot menu to start each entry once, see which one brings up OPNsense, and set that one as Boot Option #1.

ZimaOS stays on the eMMC, and you can still start it once from the **F11** boot menu whenever you want.

## 2.8 First boot into OPNsense

OPNsense starts from the SSD and shows a console with a numbered menu. Log in as `root` with the password you just set.

Out of the box, OPNsense:

- sets its **LAN** address to `192.168.1.1/24` and runs a DHCP server on it
- asks for an address on **WAN** by DHCP
- blocks all unsolicited traffic arriving on WAN

Which physical port is WAN and which is LAN gets settled in step 3.

**Test the unattended start.** Choose **5) Power off system** from the console menu, wait until the screen goes dark, then unplug the power and plug it back in. The board should start by itself and arrive at the OPNsense console without a keypress. A firewall has to come back on its own after a power cut.

## Done when

- The USB drive is out and the board boots OPNsense from the SSD with nobody touching it.
- You can log in at the console as `root`.
- ZimaOS is still on the eMMC, untouched.

Next: [3. First configuration](03-first-configuration.md)
