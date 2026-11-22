# 1. Unboxing and hardware

This step takes the board from sealed box to a working firmware setup screen, with the SSD fitted and detected. Nothing here touches your network.

## 1.1 Open the box and check what's inside

Every ZimaBoard 2 box should contain these four items, according to [Zima's product page](https://shop.zimaspace.com/products/zimaboard2-single-board-server):

- [ ] ZimaBoard 2, with ZimaOS preinstalled on its eMMC
- [ ] 12 V / 5 A power adapter
- [ ] SATA Y-cable (data and power for up to two drives)
- [ ] Cooling fan

Bundles add accessories on top, and they vary by bundle. Depending on yours, you may also find a Mini DisplayPort to HDMI cable, a PCIe to NVMe adapter, a 2-bay HDD rack and Ethernet cables. Compare against your order confirmation.

Then:

- Look the board over for shipping damage: bent connectors, a loose part rattling in the case, a cracked power plug.
- Keep the box and the small parts bag until the build is finished.
- If a required item is missing, contact Zima before going further. The power adapter and SATA Y-cable are not optional for this build.

> **Filming your unboxing?** Cover the shipping label before the camera rolls, and check any stickers on the hardware for serial numbers or network (MAC) addresses you would rather not publish.

## 1.2 Get to know the board

| Part | What it does in this build |
|---|---|
| 2 x 2.5GbE Ethernet ports | One becomes **WAN** (towards the internet), the other **LAN** (towards your home network). You do not need to know which is which yet; step 3 finds out |
| 2 x USB 3.1 | Keyboard, and the USB drive with the installer |
| Mini DisplayPort 1.4 | Screen for the install. Use the adapter if your screen only has HDMI |
| DC power input | The 12 V adapter. **The board has no power button**: it starts as soon as power is connected, and again whenever power returns after an outage |
| 2 x SATA with power | The SSD that OPNsense installs to, through the SATA Y-cable |
| PCIe 3.0 x4 slot | Leave it empty for now. Its best later use is a 4-port network card for more physical ports |
| 32 GB eMMC | Holds the factory ZimaOS. This guide leaves it untouched as a fallback |
| Fan header | For the optional fan. The board is designed to run fanless |

Zima's team has said the Ethernet ports use Intel I226-V controllers. On OPNsense, which is built on FreeBSD, they appear as `igc0` and `igc1`.

## 1.3 Decide where everything lives

| Storage or slot | Use |
|---|---|
| SATA SSD | OPNsense |
| eMMC | Factory ZimaOS, left as it is. You can boot back into it later from the firmware boot menu |
| PCIe slot | Empty. The NVMe adapter is not needed for this build |
| Fan | Off for now. Fit it later if temperatures run high once heavier features such as intrusion detection are on |

Installing to a separate SSD rather than the eMMC gives OPNsense room for logs and updates, and means a mistake during the install cannot erase ZimaOS as long as you pick the right disk.

## 1.4 Fit the SSD

Do this with the power adapter unplugged.

1. Connect the board end of the SATA Y-cable to the board's SATA data and drive-power connectors. Zima's quick guide card, if your box has one, shows how it attaches.
2. Connect one of the cable's drive ends to the SSD. The second end stays free.
3. Rest the SSD somewhere flat and stable where the cable is not under strain.

If you have Zima's 2-bay HDD rack, note that it is built for 3.5-inch drives. A 2.5-inch SSD needs a 2.5 to 3.5-inch adapter bracket to mount in it. Without the rack, a flat surface next to the board is fine: an SSD has no moving parts and runs cool.

## 1.5 First power-on: open the firmware setup

Leave every Ethernet cable unplugged for this step.

1. Connect the screen (Mini DisplayPort, or the HDMI adapter) and a wired USB keyboard.
2. Plug in the power adapter. The board starts immediately.
3. Tap **Esc** repeatedly from the moment power connects. The firmware setup screen opens. Zima [published an owner's report](https://shop.zimaspace.com/pages/zimaboard-2-bios-access-esc-key) that Esc is the key, after other common keys only led to the ZimaOS boot loader.

If you miss it and the board boots ZimaOS instead, that is harmless with no network cable attached. Press **Ctrl+Alt+Delete** to restart, then tap Esc straight away.

The firmware setup is AMI's *Aptio Setup*, with six tabs: **Main**, **Advanced**, **Chipset**, **Security**, **Boot** and **Save & Exit**. The left and right arrow keys move between tabs, **Enter** opens a submenu, and **Esc** goes back one level.

Check these four things:

1. **Main: firmware and memory.**
   - Write down the **Build Date and Time**. It is the easiest way to tell later whether a firmware update has been applied.
   - **Total Memory** should read 8192 MB on the 832 (16 GB on the 1664), and the processor should show as Intel N150.
   - The **System Date** and **System Time** may be a few hours off your local time. That is fine if they match UTC: OPNsense expects the hardware clock to run on UTC, so leave them alone.
2. **Chipset → PCH-IO Configuration → SATA Configuration: the SSD.** The Main tab does not list drives; this page does. The port your SSD is on shows its name and size, for example `SATA SSD (120.0GB)`, and the other port reads `Empty`. If both ports read `Empty`, unplug the power, reseat both ends of the SATA Y-cable (or use its other drive connector) and check again.
3. **Security → Secure Boot: off.** It should read **Disabled** and **Not Active**. If it says Enabled, set it to Disabled: OPNsense's installer does not start with Secure Boot on. Turning it off does not stop ZimaOS from starting.
4. **Advanced: both network ports.** Two entries should read *Intel(R) Ethernet Controller I226-V*, each followed by a MAC address. Both present means both ports are detected. If you film this screen, blur the MAC addresses.

On the **Boot** tab, *Boot Option #1* reads **UEFI OS**. That is ZimaOS on the eMMC. The SSD is not listed because there is nothing on it to start yet. It appears after step 2.

To leave, go to **Save & Exit**. Choose **Discard Changes and Exit** if you changed nothing, or **Save Changes and Exit** if you turned Secure Boot off.

The boot menu, which you need in step 2, opens with **F11** during startup according to [a ZimaBoard 2 owner on Zima's forum](https://community.zimaspace.com/t/pxe-boot-on-zimaboard-2/6200). The **Save & Exit** tab also has a **Boot Override** list that does the same job.

## Done when

- Everything from the box is accounted for.
- The SSD appears under **Chipset → PCH-IO Configuration → SATA Configuration**.
- Secure Boot is disabled.
- Both I226-V network ports are listed under **Advanced**.
- You know the keys: **Esc** for firmware setup, **F11** for the boot menu.

Next: [2. Installing OPNsense](02-install.md)
