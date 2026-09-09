# GlobalProtect VPN — Omarchy bar widget

A status widget for the [Omarchy](https://omarchy.org) shell (Quickshell bar)
for the Palo Alto Networks **GlobalProtect** VPN client.

![Preview](./preview.png)

While the VPN tunnel is up it shows a shield icon and the **elapsed connection
time** (e.g. `1:23:45`) in the bar. When the tunnel goes down the widget
disappears automatically — usually within 2 seconds. Clicking the widget opens
a popup with connection status and **Connect / Disconnect** buttons.

## Requirements

- The proprietary `globalprotect` CLI installed and its services running
  (`gpd` system service + `gpa` user service). Palo Alto ships only `.deb` /
  `.rpm` packages; on Arch/Omarchy the deb payload can be extracted manually
  into `/opt/paloaltonetworks/globalprotect` — the UI deb is a superset of the
  agent deb. The tray GUI (`PanGPUI`) is **not** required: SAML/SSO portal
  login works through the system default browser via the
  `globalprotectcallback://` URL scheme.
- No additional packages: the widget uses only shell built-ins, `ip`-style
  `/sys` reads and the `globalprotect` CLI.
- Install globalprotect-bin from [AUR](https://aur.archlinux.org/packages/globalprotect-bin)

> This plugin is an independent community widget. It is not affiliated with
> or endorsed by Palo Alto Networks.

## Installation

```bash
omarchy plugin add https://github.com/kholis/gp-vpn.git --enable
```

Or manually:

```bash
git clone https://github.com/kholis/gp-vpn.git \
  ~/.config/omarchy/plugins/io.github.kholis.gp-vpn
omarchy plugin enable io.github.kholis.gp-vpn --section right
```

The widget defaults to the bar's right section, next to the tray.

## Settings

Settings live in the widget's layout entry in `~/.config/omarchy/shell.json`
(also editable from the bar settings UI):

```json
{
  "id": "io.github.kholis.gp-vpn",
  "interface": "gpd0",
  "portal": "vpn.example.com"
}
```

| Setting    | Default | Description                                            |
| ---------- | ------- | ------------------------------------------------------ |
| `interface`| `gpd0`  | Tunnel interface to watch                              |
| `portal`   | (empty) | Portal hostname used by the popup's *Connect* button   |

## How it works

- Tunnel state is read from `/sys/class/net/<interface>/flags` bit 0
  (`IFF_UP`) — `operstate` reports `unknown` for point-to-point tunnels and
  cannot be used.
- The connection start time is detected from a marker file in
  `$XDG_RUNTIME_DIR` (survives bar reloads), falling back to the mtime of
  GlobalProtect's `ipt_mangle.txt` (rewritten whenever the daemon applies
  tunnel routing rules), falling back to the current time.
- Diagnostics via IPC:

  ```bash
  omarchy-shell io.github.kholis.gp-vpn status
  omarchy-shell io.github.kholis.gp-vpn refresh
  ```

## Removal

```bash
omarchy plugin remove io.github.kholis.gp-vpn
```

## License

[MIT](LICENSE) © Nur Kholis Majid
