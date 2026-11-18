# Add-ons

Optional projects that build on the base firewall. Each one gets its own folder once it has been built and tested on real hardware. Until then it is listed here as planned.

| Add-on | What it adds | Status |
|---|---|---|
| DNS filtering | Blocks ad and tracking domains for the whole network, using OPNsense's built-in Unbound blocklists | Planned |
| WireGuard VPN gateway | Sends chosen devices through a VPN provider while the rest use your normal connection, with a kill switch | Planned |
| VLAN segmentation | Separate networks for consoles, guests or smart-home devices, with rules between them | Planned |
| Network monitoring and intrusion detection | Traffic insight and Suricata alerts | Planned |

Finish the base build in [`docs/`](../docs/) first. Every add-on assumes a working firewall with a current configuration backup.
