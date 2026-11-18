# 5. Verification

Work through this list after the switchover. Each check says what to do and what you should see. Anything that fails has an entry in [troubleshooting](troubleshooting.md).

## Right after the switchover

| Check | How | Expected |
|---|---|---|
| WAN is up | **Interfaces → Overview** | WAN has an address, and the gateway shows as online on the Dashboard |
| Devices get addresses | **Services → Dnsmasq DNS & DHCP → Leases** | Your devices are listed, inside your DHCP range |
| Reservations work | Same page, plus each device's own network settings | Reserved devices have their usual addresses |
| DNS works | `nslookup example.com <your LAN address>` from a computer | An answer, served by your LAN address |
| Internet works | Open several sites on a few different devices | They load normally |
| Speed is as before | Speed test from the same wired computer as your baseline | Close to the baseline you recorded in step 4 |

## Security

| Check | How | Expected |
|---|---|---|
| Nothing allowed in from outside | **Firewall → Rules → WAN** | No pass rules that you created |
| No port forwards | **Firewall → NAT → Destination NAT** | Empty, unless you added one on purpose |
| Web interface not on the internet | If your WAN address is public: on a phone using **mobile data** (Wi-Fi off), open `https://<your WAN address>` | The page never loads |
| Default password gone | Log in to the web interface | Only your own password works, not `opnsense` |
| Backup is current and private | **System → Configuration → Backups** | You hold an encrypted backup from after the switchover, stored offline |

## Things people often forget

| Check | How | Expected |
|---|---|---|
| Survives a power cut | Choose **5) Power off system** at the console, wait, unplug, plug back in | It starts on its own and the internet returns without a keypress |
| Boots the right disk | Same test | OPNsense comes up, not ZimaOS |
| Remote-access tools connect directly | If you use Tailscale, run `tailscale status` on a device; for ZeroTier, check the peer list | Connections show as direct rather than relayed. If they're relayed, see [troubleshooting](troubleshooting.md#vpn-tools-connect-through-relays) |
| Consoles are happy | Run each console's network test | Same NAT type as before. If stricter, see [troubleshooting](troubleshooting.md#game-console-shows-a-strict-nat-type) |
| Media and file sharing work | Stream from your media server, open a network share | Same as before |

## Over the first few days

- [ ] **Link stability.** Check **System → Log Files → General** (search for `igc`) once a day. A port that goes *down* and *up* again repeatedly needs the fix in [troubleshooting](troubleshooting.md#ports-drop-their-link-now-and-then).
- [ ] **Temperature.** Glance at the Dashboard's temperature readings now and then, especially once you add heavier features. Fit the fan from the box if it runs hot.
- [ ] **Updates.** Check **System → Firmware** about once a month. Take a backup before every update.
- [ ] **Old router.** Still untouched and within reach.

When everything here passes, the build is done.
