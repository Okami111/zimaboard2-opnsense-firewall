# Troubleshooting

Each entry lists the symptom, the likely causes and what to try, in order. If you're mid-switchover and stuck, [roll back](04-cutover.md#48-if-something-goes-wrong-roll-back) first and troubleshoot calmly afterwards.

## Firmware setup or boot menu won't open

- Use a **wired** USB keyboard plugged straight into the board, not through a hub.
- Start tapping the key the moment power connects; the window is short. **Esc** opens firmware setup and **F11** the boot menu.
- Unplug the power completely, wait ten seconds, and try again. A cold start gives the cleanest chance.

## The USB drive doesn't boot

- Check you downloaded the **vga** image, not dvd, serial or nano.
- Rewrite the drive. With Rufus, choose **DD Image mode**. Copying the file onto the drive doesn't work.
- Try the other USB port, or another USB drive.
- In firmware setup, check **Security → Secure Boot**. If it is enabled, disable it: OPNsense's boot loader isn't signed for Secure Boot.
- Use **Save & Exit → Boot Override** in firmware setup instead of F11, and pick the **UEFI** entry for the drive.

## The installer doesn't see the SSD

- Power off and unplug. Reseat the SATA Y-cable at both ends, or try its other drive connector.
- Check the SSD appears under **Chipset → PCH-IO Configuration → SATA Configuration** in firmware setup. If that page shows both ports as `Empty`, OPNsense can't see the drive either.
- Test the SSD in another computer, or try another one.

## It boots ZimaOS instead of OPNsense

The eMMC is still first in the boot order. In firmware setup, move the SSD to the top of the boot priority list and save. See [step 2.7](02-install.md#27-make-the-ssd-the-first-boot-device).

## Can't reach the web interface

- Check the computer is plugged into the **LAN** port, not WAN.
- Check the computer has an address in the LAN range (`ipconfig` on Windows, `ip addr` on Linux, `ifconfig` on macOS). If it shows `169.254.x.x`, it got no DHCP answer: check the cable and the port.
- Use `https://`, and accept the certificate warning.
- The OPNsense console shows the current LAN address above its menu. Use that one.
- If your old router uses the same range as OPNsense's LAN, see [step 3.1](03-first-configuration.md#31-check-for-an-address-clash-first).
- Turn off the computer's Wi-Fi so it doesn't try to reach the firewall over another network.

## No internet behind the firewall

Work outward from the firewall:

1. **Interfaces → Overview**: does WAN have an address? If not, see the next entry.
2. **Dashboard**: does the gateway show as online? On a WAN set to DHCP, the gateway is learned automatically. On a static WAN, check **Interfaces → WAN** has the upstream gateway selected; one forum report says the setup wizard can leave it unset.
3. From a computer on LAN, `nslookup example.com <LAN address>`. If names don't resolve but `ping 1.1.1.1` works, the problem is DNS: check **Services → Unbound DNS** is enabled.
4. **Firewall → NAT → Source NAT**: leave it in automatic mode unless you know why you changed it.
5. Check the cables aren't swapped. The ports look identical, which is why step 3 suggests labelling them.

## WAN gets no address after the switchover

- Confirm the modem or wall-port cable is in the **WAN** port.
- Power-cycle the modem. Many modems keep the previous router's hardware address for a while.
- If your internet provider registers devices by hardware address, register the ZimaBoard with them.
- From the console, **11) Reload all services**, or reboot the firewall.
- Still nothing? [Roll back](04-cutover.md#48-if-something-goes-wrong-roll-back) and check whether the old router gets an address; if it doesn't either, the problem is upstream.

## Ports drop their link now and then

Symptom: the general log (**System → Log Files → General**, search `igc`) shows a port going *down* and back *up*, with short internet drops to match.

Intel I225 and I226 ports on some Intel N-series mini PCs [drop their link under light traffic](https://computingforgeeks.com/fix-intel-i226-nic-drops-opnsense/), a fault tied to PCIe power saving.

1. Try another cable, and another port on the switch or modem.
2. In firmware setup, look for **ASPM** (PCIe Active State Power Management), usually under the chipset or PCI Express settings, and disable it.
3. Check Zima's site for a newer firmware (BIOS) version.

## VPN tools connect through relays

Peer-to-peer VPN tools such as Tailscale and ZeroTier try to connect devices directly, and fall back to slower relays when the firewall's address translation gets in the way. Your old router may have helped them through UPnP or NAT-PMP, which OPNsense doesn't run by default.

Options, from most to least contained:

1. **Endpoint-independent UDP mapping (full-cone NAT).** OPNsense 26.7.2 added this to **Firewall → NAT → Source NAT**. Applied to the device that runs the VPN tool, it usually allows direct connections.
2. **A static-port source NAT rule** for that device only, so its outgoing UDP port isn't rewritten.
3. **The UPnP / NAT-PMP plugin** (`os-upnp`), limited to the specific devices that need it. This lets those devices open ports by themselves, so keep the list short.

After each change, check again whether connections show as direct.

## Game console shows a strict NAT type

Consoles want to be reachable by other players, which a default-deny firewall prevents. The same three options as above apply, limited to the console's address. Give the console a DHCP reservation first so the rule always matches the right device.

## Locked out, or forgot the password

The console with a keyboard and screen is your way back in, which is why this guide keeps them handy.

- **3) Reset the root password** at the console menu.
- **13) Restore a backup** to return to an earlier configuration.
- **4) Reset to factory defaults** as a last resort. You then repeat step 3, or restore your downloaded backup through the web interface.

## Still stuck

- [OPNsense documentation](https://docs.opnsense.org/)
- [OPNsense forum](https://forum.opnsense.org/)
- [Zima community forum](https://community.zimaspace.com/), for hardware questions

When asking for help, never paste your configuration backup, and check screenshots for public addresses and account details first.
