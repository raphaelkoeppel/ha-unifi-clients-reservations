<p align="center">
  <img src="dhcp_reservations/logo.png" alt="IP Manager for UniFi" height="80">
</p>

<p align="center">
  <b>See and manage all UniFi clients and DHCP reservations in one table — right inside Home Assistant.</b><br>
  MAC · name · fixed IP · VLAN · firewall zone · WiFi / wired / Teleport · issues · versions · bulk changes
</p>

<p align="center">
  <a href="https://my.home-assistant.io/redirect/supervisor_add_addon_repository/?repository_url=https%3A%2F%2Fgithub.com%2Fha-unifi-friends%2Fha-ip-manager-unifi"><img src="https://my.home-assistant.io/badges/supervisor_add_addon_repository.svg" alt="Add repository to Home Assistant"></a>
  <br>
  <sub>or add manually: <a href="https://github.com/ha-unifi-friends/ha-ip-manager-unifi"><code>https://github.com/ha-unifi-friends/ha-ip-manager-unifi</code></a></sub>
</p>

![Overview](docs/screenshots/overview.png)

UniFi Network spreads DHCP reservations over individual client pages, and there is no single place to check which device has
which fixed IP in which VLAN. **IP Manager for UniFi** is a Home Assistant app (formerly called add-on) that gives you exactly that view. It is styled like
UniFi Network, and it adds bulk tools that UniFi doesn't have: subnet migration, CSV import, names from Home Assistant, and
versions you can compare and restore.

▶️ **[Watch the 1¾-minute demo video](docs/promo.mp4)**

## Features

### Overview & checks

| | |
|---|---|
| **One table for everything** | Every client and UniFi device (APs, switches, gateway) with MAC, name, IP address, fixed IP, network/VLAN, connection type (WiFi, wired with switch port, Teleport), locked AP, first and last seen. Sort and filter every column, reorder columns by drag & drop. |
| **Status at a glance** | Narrow icon columns for **UniFi Status** (online / offline) and **Home Assistant Status**, plus an **IP** pill: *Fix* (reservation or static IP) or *DHCP*. |
| **Issue hints** | Duplicate IP, fixed IP outside its network or inside the DHCP pool, **IP is not Fixed IP** (the fixed IP is shown in yellow), **not in the override VLAN**, random/private MAC, not on its locked AP, wrong network by your own rules, Telnet/FTP open, duplicate names / DNS records, inactive devices. Every issue is a filter in the sidebar. |
| **Views** | SharePoint-style views store columns, order, filters and sort. Built-in *All*, *Problems*, *New devices*, *Offline with fixed IP*, *Home Assistant*, *WiFi* and *DHCP Reservations*; views are shared by every browser. |
| **Inline edit** | Edit names and fixed IPs directly in the table (Enter / ↑ / ↓ move to the next row). |
| **Light & dark** | *Auto* (follows the system) → *Light* → *Dark*, remembered per browser. |

### Home Assistant

| | |
|---|---|
| **Integration icon & status** | Devices known to Home Assistant (matched by MAC) show the logo of their integration and their HA status – OK, partly unavailable, unavailable, integration error, disabled – as a column, a filter and in the detail panel. For a partly unavailable device the unavailable entities are listed. |
| **Home Assistant link** | *Open in HA ↗* opens the device page in Home Assistant in a new tab. |
| **Names from Home Assistant** | Proposes the names from the HA device registry for your UniFi clients – optionally with the **area** and/or the **integration** as prefix or suffix (e.g. *Office Desk Lamp*, *Desk Lamp - ESPHome*), since the UniFi app shows neither. Includes a “near match” for devices with separate WiFi/Ethernet MACs. Preview, pick, apply. |
| **New-device alerts** | A Home Assistant notification and the event `clients_reservations_new_device` when a never-seen MAC appears – optionally only for certain networks. |
| **HA in the IP plan** | The IP plan can show the integration logos, and the CleanUp Proposal can group fixed IPs **by HA integration** (all Shelly, then all ESPHome, …). |
| **HA status in versions** | Every version records the HA status, so *Compare* and the device history show e.g. “OK → Unavailable”. |

### Edit a device

| | |
|---|---|
| **Edit** | Name, note, fixed IP + network (with *Select*: pick a free address in a small IP plan with icons and see its neighbours), lock to an access point, local DNS record. Add reservations for MACs the controller has never seen. Remove a reservation or forget the client. |
| **Virtual Network Override** | Put a client into a network (VLAN) regardless of its WiFi or switch port. **UniFi offers this only in its mobile app** – here it is in the browser, with its own column, and an issue when a device is not in its override network. |
| **Reconnect** | After a fixed IP change, online WiFi clients can be reconnected right away so they pick up the new address – per device or for a whole subnet move. |
| **Open, ping & port check** | Open the device's web UI (HTTP / HTTPS), ping it, and check ~30 common TCP ports on demand; web UIs become links with their page title, Telnet/FTP are flagged. Vendor lookup for unknown MAC prefixes, Wake on LAN. |

### Bulk changes & IP planning

| | |
|---|---|
| **IP plan** | A full page next to the client list (switch *Clients / IP Plan*), filtered by the same sidebar filters (non-matching devices greyed out): networks as cards with utilization and a **Tidiness score** (0–100: fixed IPs outside the pool, close together, no conflicts), plus an address grid that shows conflicts at a glance, with *No icons / UniFi / Home Assistant* icons. Click an address to open the device – or a free one to reserve it – in the detail panel next to the plan. |
| **CleanUp Proposal** | Suggests a tidy layout (fixed block + DHCP pool behind it) – *pool only* (no device changes its address), *minimal*, *grouped by HA integration* or *grouped by vendor* – for one network or **all networks at once** (overall tidiness before → after), right in the IP plan as a tab. *Preview changes* shows every move as an arrow, **conflicts in pink** (dynamic clients in the new fixed block), and a hover card shows the concrete change per address. The **Home Assistant host** (HA blue) and the **gateway** (violet) are never moved, and addresses in use (online devices, IPs set on the device) are never handed out. Afterwards it checks which devices already use their new IP, and *Undo everything* sets pools and fixed IPs back. After one confirmation it runs in a safe order: save a version → change the pools → reconnect conflicting Wi-Fi clients → move the fixed IPs → reconnect the moved clients. |
| **Replace subnet** | Move all fixed IPs from e.g. `192.168.30.x` to `192.168.31.x`, keeping the host part. A preview catches collisions before anything changes. |
| **CSV import & export** | Bulk changes with a validated preview and a checkbox per row. An export can be re-imported as is. Hosts-file export of fixed IPs and DNS records. |
| **Network rules** | “Vendor contains Espressif → IoT”: devices in the wrong network are flagged. |
| **Safe bulk changes** | Every bulk action (CSV import, restore, replace subnet, names from HA, CleanUp) saves a version of the current settings first, so you can always go back via *Versions › Restore*. |

### Firewall (read only)

| | |
|---|---|
| **Zone matrix** | The zone-based firewall of UniFi Network 9+ on its own page (switch *Clients / IP Plan / Firewall*): Allow All, Block All, Allow Return and the number of custom policies per zone pair. Click a cell to filter the policy list below it. |
| **Policies** | Name, action, IP version, protocol, source and destination with zone, address and port – devices by name, address groups resolved – plus *Hits* and *Last Hit* from UniFi's counters. Filters for IPv4 / IPv6, built-in, custom and paused, plus a search. |
| **Zones everywhere** | Column and sidebar filter *Firewall zone* in the client table; the detail panel lists every policy that names the device (by MAC, IP, address group or network). |
| **IP changes check the firewall** | CleanUp, Replace subnet, CSV import, Restore, inline edit and the edit panel warn when a policy still points to the old IP, when the new IP falls under a policy, and when a device moves into another zone. |
| **Firewall in versions** | Versions keep the zones and custom policies; *Compare* shows new, removed and changed policies. |

Nothing is written to the firewall – policies are changed in UniFi.

### Versions & history

| | |
|---|---|
| **Versions** | Snapshot the table manually, daily, and automatically before every bulk action. **Compare** two versions (new / removed / changed devices; spots a device that came back with a new random MAC) and **restore** through a preview. |
| **Changed since …** | Sidebar filter for devices that are new or changed since a version. |
| **Device history** | Per device: which change, in which version. |

## Screenshots

**Overview & checks**

<table>
  <tr>
    <td width="50%" valign="top"><img src="docs/screenshots/views.png" alt="Views"><br><b>Views</b> – e.g. the built-in <i>DHCP Reservations</i> view.</td>
    <td width="50%" valign="top"><img src="docs/screenshots/light.png" alt="Light theme"><br><b>Light & dark</b> – the switch at the bottom of the filter pane.</td>
  </tr>
  <tr>
    <td valign="top"><img src="docs/screenshots/inline-edit.png" alt="Inline edit"><br><b>Inline edit</b> – names and fixed IPs right in the table.</td>
    <td valign="top"><img src="docs/screenshots/actions.png" alt="Actions menu"><br><b>Actions</b> – names from Home Assistant, replace subnet, CSV import/export, hosts file, network rules.</td>
  </tr>
</table>

**Home Assistant**

<table>
  <tr>
    <td colspan="2" valign="top"><img src="docs/screenshots/home-assistant.png" alt="Home Assistant integration icon and status"><br><b>Integration icon & status</b> – column, filter and the <i>Home Assistant</i> section in the detail panel with <i>Open in HA ↗</i> and the unavailable entities.</td>
  </tr>
  <tr>
    <td width="50%" valign="top"><img src="docs/screenshots/names-from-ha.png" alt="Names from Home Assistant"><br><b>Names from Home Assistant</b> – rename UniFi clients after your HA device names, with area / integration as prefix or suffix.</td>
    <td width="50%" valign="top"><img src="docs/screenshots/cleanup-proposal.png" alt="CleanUp Proposal for all networks"><br><b>CleanUp Proposal for all networks</b> – grouped by HA integration, <i>Preview changes</i> with arrows, conflicts in pink and the hover card for an address.</td>
  </tr>
</table>

**Edit a device**

<table>
  <tr>
    <td width="50%" valign="top"><img src="docs/screenshots/edit-panel.png" alt="Edit panel with Virtual Network Override"><br><b>Edit</b> – Virtual Network Override, fixed IP, AP lock and local DNS in the detail panel.</td>
    <td width="25%" valign="top"><img src="docs/screenshots/open-ports.png" alt="Network check"><br><b>Network check</b> – open the web UI, ping, open ports.</td>
    <td width="25%" valign="top"><img src="docs/screenshots/history.png" alt="Device history"><br><b>History</b> – what changed, in which version.</td>
  </tr>
</table>

**Bulk changes & IP planning**

<table>
  <tr>
    <td width="50%" valign="top"><img src="docs/screenshots/ip-plan.png" alt="IP plan"><br><b>IP plan</b> – full page next to the client list: networks with utilization and Tidiness, large cells with Home Assistant logos.</td>
    <td width="50%" valign="top"><img src="docs/screenshots/ip-plan-all.png" alt="IP plan, all networks"><br><b>All networks</b> – compact view (32 addresses per row) with the overall tidiness; the sidebar filters grey out non-matching devices.</td>
  </tr>
  <tr>
    <td width="50%" valign="top"><img src="docs/screenshots/replace-subnet.png" alt="Replace subnet with collision check"><br><b>Replace subnet</b> – preview with collision check and optional reconnect of the affected WiFi clients.</td>
    <td width="50%" valign="top"><img src="docs/screenshots/csv-import.png" alt="CSV import with validation"><br><b>CSV import</b> – every row is validated before anything changes.</td>
  </tr>
  <tr>
    <td valign="top"><img src="docs/screenshots/network-rules.png" alt="Network rules"><br><b>Network rules</b> – which devices belong in which network; violations show up as issues.</td>
    <td valign="top"></td>
  </tr>
</table>

**Versions & history**

<table>
  <tr>
    <td width="50%" valign="top"><img src="docs/screenshots/compare-versions.png" alt="Compare versions"><br><b>Compare versions</b> – new, removed and changed devices since a snapshot.</td>
    <td width="50%" valign="top"><img src="docs/screenshots/versions.png" alt="Versions"><br><b>Versions</b> – manual, daily and before-bulk-action snapshots.</td>
  </tr>
  <tr>
    <td valign="top"><img src="docs/screenshots/changed-since.png" alt="Changed since filter"><br><b>Changed since …</b> – sidebar filter for new or changed devices.</td>
    <td valign="top"></td>
  </tr>
</table>

<sub>All screenshots and the video use a fictional demo network.</sub>

## Requirements

- Home Assistant **OS** or **Supervised** (apps are required), on `amd64` or `aarch64`.
- A UniFi OS console running UniFi Network (UDM / UDM Pro / UDR / UCG / UX / Cloud Key Gen2+, or a self-hosted UniFi OS Server).
- A **local** UniFi account (not a Ubiquiti cloud account, no 2FA) with permission to edit clients.
- For the firewall page: UniFi Network **9.0 or later** with the zone-based firewall, and an account that may view the firewall settings. Everything else works without it.

## Installation

1. Click the button above, **or** go to *Settings → Apps*, open the app store and choose *⋮ → Repositories*, then add
   `https://github.com/ha-unifi-friends/ha-ip-manager-unifi`
2. Install **IP Manager for UniFi**. Home Assistant builds the image locally, so this takes a minute.
3. On the *Configuration* tab, enter the gateway IP, username and password, then start the app.
4. Turn on *Show in sidebar* (the panel is called *IP Manager*), or open it via *Open Web UI*.

## Configuration

| Option | Default | Description |
|---|---|---|
| `unifi_host` | `192.168.1.1` | IP or hostname of the UniFi console (without `https://`) |
| `username` / `password` | – | Local UniFi account |
| `site` | `default` | UniFi site name |
| `verify_ssl` | `false` | Keep off while the console uses its self-signed certificate |
| `certificate_fingerprint` | – | Pin the controller certificate (SHA-256); the app log shows the value to paste at start |
| `daily_version` | `true` | Save one version of the table per day |
| `auto_version_retention_days` | `30` | Days to keep automatic versions; manual, pinned and baseline versions are kept until deleted |
| `notify_new_devices` | `true` | Notify Home Assistant about devices with a never-seen MAC |
| `notify_networks` | `[]` | Only watch these networks (names or VLAN IDs); empty = all |
| `notify_random_mac` | `false` | Also notify about random/private MACs (phones, tablets) |
| `vendor_lookup` | `true` | Fill unknown vendors via macvendors.com (only the 3-byte prefix is sent, cached) |

**Security notes**

- Create a dedicated local UniFi user for this app, with the *Network* admin role (needed to edit clients) and not as owner or super admin.
- The password is stored in the app's options inside Home Assistant (masked in the UI). It is never logged and never sent to the browser.
- Home Assistant backups contain app options, so keep your backups encrypted.
- The UI is only reachable through Home Assistant Ingress. The app does not expose a port.
- Pin the controller certificate with `certificate_fingerprint` – the self-signed UniFi certificate is otherwise not verified.
- Ping, port check and Wake on LAN only work for addresses inside your UniFi networks. CSV exports are protected against formula injection.

Full documentation: [`dhcp_reservations/DOCS.md`](dhcp_reservations/DOCS.md) · Changes: [`CHANGELOG.md`](dhcp_reservations/CHANGELOG.md)

## How it works

A small Python service (standard library only) inside the app talks to the UniFi Network API of your console:

- **Read:** `rest/user`, `stat/sta`, `stat/device`, `rest/networkconf`, and v2 `clients/active` for Teleport.
- **Write:** `rest/user` for name, note, fixed IP, AP lock, local DNS and the virtual network override; `cmd/stamgr` (`kick-sta` to reconnect, `forget-sta` to forget a client); `rest/networkconf` only for the DHCP pool of a CleanUp Proposal.

For the Home Assistant features, the service reads the device registry through the Supervisor API (read-only), reads the host's own addresses (`network/info`, so CleanUp never moves the Home Assistant host) and creates notifications. Views and versions are stored in the app's `/data` folder, so they are included in Home Assistant backups.

## Languages

The UI is available in English, Deutsch, Schwiizerdütsch (Mundart), Français, Italiano, Español, Nederlands, Polski, Português (Brasil) and Svenska – pick it at the bottom of the sidebar (*Auto* follows the browser). Translations are plain JSON files in [`dhcp_reservations/app/static/i18n/`](dhcp_reservations/app/static/i18n/): the key is the English text, `{0}`, `{1}` … are placeholders for numbers, names or addresses and may be reordered. Corrections and new languages are welcome as pull requests.

## Contributing

The project lives in the GitHub organization [ha-unifi-friends](https://github.com/ha-unifi-friends) – ideas, bug reports and pull requests are welcome. Please open an [issue](https://github.com/ha-unifi-friends/ha-ip-manager-unifi/issues) first for larger changes.

## Disclaimer

This is a community project. It is not affiliated with or endorsed by Ubiquiti Inc. or the Home Assistant project.
UniFi is a trademark of Ubiquiti Inc. It uses the controller's unofficial local API, which can change with UniFi Network updates.
Always keep a backup. The app saves a version before every bulk action, but you use it at your own risk.

## License

[MIT](LICENSE)
