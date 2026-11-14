# IP Manager for UniFi

Shows every known UniFi client with MAC, name, fixed IP and VLAN in one table, styled like
UniFi Network › Client Devices. Every column can be sorted (click the header) and filtered
(row below the header); the sidebar adds UniFi-style filters with counts.

## Features

- **Edit:** click a row → detail panel → *Settings*: name, note, fixed IP (network/VLAN + IP),
  lock to access point, local DNS record. Changing the network suggests a free IP outside the DHCP pool.
- **New reservation:** also for a MAC the controller has never seen.
- **Remove:** either remove only the reservation (the device stays known) or forget the client entirely.
- **Connection type:** WiFi / Wired / Teleport (Teleport via the controller's v2 API), switch port for wired devices.
- **Locked to AP:** shown, filterable and editable.
- **Issues:** IP outside its network, duplicate IP, device currently has a different IP,
  reservation inside the DHCP pool, random MAC (e.g. iOS "Private Wi-Fi Address"),
  not on the locked AP, VLAN override.
- **Header bar:** View selector, Refresh, Versions, Inline edit, Actions (Names from HA, Replace subnet, Import/Export CSV, hosts file, Network rules) and Add reservation. *Clear filters*, *Customize columns*, the version and the light/dark switch are at the bottom of the filter pane.
- **Versions:** *Versions* in the header. *Save version* stores the table (names, notes, fixed IPs, AP locks, DNS records and,
  for information, current IP, network, first/last seen) in `/data/versions/` – part of Home Assistant backups. A version is
  also saved automatically before every bulk action, and once a day (option `daily_version`, default on). Automatic versions
  are kept for `auto_version_retention_days` (default 30); manual, pinned and baseline versions until deleted.
  - *Compare* shows new devices, removed devices, changed fields, IP changes and network moves against the current table or
    between two versions. New devices with the same hostname as a removed one are flagged as probably the same device with a new MAC.
  - *Restore* runs like a CSV import from the version (preview, checkbox per row, chosen fields) and leaves devices that were
    added later untouched. A version of the current state is saved first.
  - *History* in the detail panel (Overview) shows, per device, a timeline of the changes and the version that first contains them, plus what changed since the latest version.
  - The sidebar filter *Baseline* is always there: *Changed since …* for the baseline (★, set via *Set baseline*) and the
    latest versions (max. 5). It lists devices that are new or whose name, note, fixed IP, AP lock or DNS record changed.
- **Inline edit:** switch it on in the header to edit Name and Fixed IP directly in the table. A change is saved when you leave the field
  (Enter / ↑ / ↓ jump to the next row, Esc reverts). An empty Fixed IP removes the reservation; the network is derived from the IP.
- **Names from HA:** reads the Home Assistant device registry (devices with a MAC connection) and proposes the name you gave the
  device in Home Assistant, otherwise the integration's device name. Devices known only via the UniFi integration are skipped.
  UniFi devices (access points, switches, gateway) and UniFi Protect cameras are listed as *UniFi-managed* and never renamed –
  UniFi is the golden record for their names, a change would be synced back into Home Assistant.
  Preview with checkbox per row; “near match” covers devices whose MAC differs only in the last byte (separate Wi-Fi/Ethernet MAC).
  *Area* and *Integration* can each be added as prefix or suffix (*Off / Prefix / Suffix*), with the separator `a b`, `a-b`, `a_b`, `a - b` or none –
  e.g. *Office Desk Lamp* or *Desk Lamp - ESPHome*. *Already in the name*: *Add anyway*, *Skip* (default – leaves out parts the name already contains) or *Replace* (removes the part from the name first, then adds it at the chosen place, e.g. *Kitchen Light* → *Light-Kitchen*);
  *CamelCase* starts every word upper case and continues lower case (*OfficeDeskLamp*). The choice is remembered per browser.
- **Views:** like SharePoint views. A view stores the visible columns, their order, column filters, sidebar filters and sort.
  Built-in: *All* and *DHCP Reservations* (MAC, Name, Fixed IP, IP, Last seen – clients with a fixed IP). Use the view menu to
  save changes, save as a new view, rename, delete, discard changes or set the default view. Views are stored in the
  app (`/data/views.json`) and shared by every browser.
- **Replace subnet:** moves every fixed IP of a source subnet to the same host address in a target subnet
  (e.g. `192.168.30.0/24` → `192.168.31.0/24`; shorthand `192.168.31` works too). The target network must exist in UniFi.
  The preview flags rows that cannot be moved (no target network, gateway address, IP already reserved, duplicate in the
  source) and lets you pick rows. Devices take the new address at their next DHCP renewal or reconnect.
- **Customize columns:** show, hide and reorder columns (drag a column header in the table, or the grip in the Columns panel); remembered in the browser.
- **UniFi devices:** access points, switches and the gateway are listed with live status, IP and uplink (filter *Type*).
- **Virtual network override:** in *Settings* of a client – puts it into the chosen network (VLAN) regardless of SSID or switch port, like *Virtual Network Override* in the UniFi app. A fixed IP has to be in that network; the form switches the reservation network along.
  If an online device is not in its override network, it gets the issue *Not in <network> (override)*; an offline device last seen in another network gets the info *Last seen outside <network> (override)*. Filter: *Issues › Not in override VLAN*.
- **IP is not Fixed IP:** the fixed IP is shown in yellow when the device currently (or last) uses a different address – it has not renewed its lease yet. Filter: *Issues › IP is not Fixed IP*; *Reconnect* helps for Wi-Fi clients.
- **Issue “In DHCP pool”:** the fixed IP lies inside the network's DHCP range. UniFi accepts this, but keeping fixed IPs outside the pool avoids conflicts with an address that was already leased before the reservation, and keeps the pool size meaningful.
- **Reconnect:** when a fixed IP changes, online Wi-Fi clients can be reconnected (UniFi *kick-sta*) so they take the new IP right away – a checkbox in the edit panel (on by default) and in Replace subnet / CSV import / restore (off by default, one client per second). Wired and offline devices take the new IP at their next DHCP renewal.
- **Sorting in previews:** click a column header in any preview table to sort; click again to reverse.
- **IP plan:** switch *Clients / IP Plan* at the left of the header. The plan is a full page; the sidebar shows the networks as cards above the usual filters and search – devices outside the filters are greyed out in the plan (subnet, pool, fixed / in pool / dynamic / free, utilization, **Tidiness**) and an address grid (fixed / dynamic / conflict / gateway / free in pool / free). Click a device to open it or a free address to reserve it – the detail panel opens next to the plan.
  Cell size *Small* (compact, up to 32 addresses per row) or *Large* (big icons) – next to the icon switch.
  *All networks* (first card) shows every network one below the other with an overall tidiness (weighted by the fixed IPs). Its CleanUp Proposal works on all networks at once: same strategy, fixed block suggested per network or as a percentage, one checkbox per network, all pool changes confirmed together.
  *Tidiness* (0–100): fixed IPs outside the DHCP pool (50 points), close together – gaps up to 10 addresses are fine (30), no conflicts or devices on another IP (20). The CleanUp Proposal shows the value before → after.
- **CleanUp Proposal** (tab in the IP plan, next to *Current*): proposes a fixed-IP block at the start of the network and the DHCP pool behind it, shows the resulting plan and – with *Preview changes* – every move as an arrow / animation, and lists the measures (move fixed IPs, reconnect Wi-Fi clients, change the pool) with a checkbox each. *Minimal* moves only what is in the way; *Grouped by vendor* and *Grouped by HA integration* renumber in blocks (devices not in Home Assistant last). The cells of all IP plan views can show *No icons*, the *UniFi* device icon or the *Home Assistant* integration logo. Applying runs in this order after one confirmation: save a version, change the pools, reconnect conflicting Wi-Fi clients, move the fixed IPs, reconnect the moved Wi-Fi clients. Devices that other systems reach by IP (e.g. Home Assistant integrations configured with an IP, firewall rules) must be updated after a move.
- **New-device notifications:** every 2 minutes the app looks for MACs it has never seen and creates a Home Assistant notification and the event `clients_reservations_new_device` (data: mac, name, hostname, vendor, ip, network, vlan, connection, random_mac) – use it in automations, e.g. to send a push message. The first run only learns the current devices.
- **Network rules** (Actions › Network rules): “Vendor / Name / Hostname contains … → network”. Devices in another network get the issue *Expected in …*.
- **Export hosts file** (Actions): fixed IPs with local DNS record and a host name derived from the device name.
- **Wake on LAN:** button for offline wired devices. Uses Home Assistant's `wake_on_lan` service when it is set up (`wake_on_lan:` in configuration.yaml) and additionally sends a magic packet to the subnet broadcast and the device IP. Waking devices in another VLAN only works if the network forwards the packet.
- **Ping:** *Network check › Ping* sends 4 ICMP echo requests from Home Assistant's network and shows replies and round-trip times.
- **Open ports:** *Check ports* in the detail panel tests about 30 common TCP ports on the device (on demand, one device at a time, in small batches). Web ports become links with the page title; Telnet and FTP are flagged as issues. Only addresses inside the UniFi networks can be checked. The result reflects what Home Assistant's network can reach – from another VLAN it may differ. If UniFi Threat Management (IDS/IPS) is on, it may still log the check.
- **Vendor lookup:** *Lookup vendor* next to the MAC opens maclookup.app for the vendor prefix. Vendors UniFi does not know are looked up in the background via macvendors.com (option `vendor_lookup`); only the 3-byte prefix is sent and results are cached.
- **Home Assistant link:** devices that Home Assistant knows (matched by MAC – for clients through an integration other than UniFi, for access points, switches and the gateway through the UniFi integration) show the integration logo and their HA status in the column *Home Assistant* and in the detail panel, with *Open in HA ›* to jump to the device page. Sidebar filter *Home Assistant*.
- **Inactive filter:** not seen for > 30 / 90 / 365 days or never seen.
- **IP column:** *Fix* (DHCP reservation, or for UniFi devices a static IP configured on the device) or *DHCP*; filter and sort like any column.
- **Light / dark:** the button at the bottom of the filter pane switches *Auto* (follows the system) → *Light* → *Dark*; the app version is shown next to it.
- **Firewall (read only):** switch *Clients / IP Plan / Firewall*. Needs UniFi Network 9.0 or later with the zone-based firewall;
  the app only reads it, policies are changed in UniFi.
  - *Zone matrix*: one row per source zone, one column per destination zone. A cell shows the default of the zone pair
    (*Allow All*, *Block All*, *Allow Return*, derived from UniFi's built-in policies) and in brackets the number of custom
    policies. Click a cell to list the policies of that pair, a zone name to list everything from or to it, *All policies* to reset.
  - *Policy list*: name, action, IP version, protocol, source / destination zone, address and port, *Hits* and *Last Hit*
    (UniFi's counters, also without logging) and the ID (UniFi's order). *Export JSON* downloads what the app read.
    Filters for IPv4, IPv6, built-in, custom and paused policies plus a search. Click a policy for the details with links to the devices.
  - The sidebar shows the zones with their networks, device counts and custom policies.
  - The client table has the column and sidebar filter *Firewall zone* (the zone of the device's network; the gateway is in *Gateway*,
    Teleport clients in *VPN*).
  - The detail panel shows the zone and every policy that names the device: by MAC, by its fixed or current IP, through an address
    group, or through its network.
  - When a fixed IP changes (edit panel, inline edit, CleanUp, Replace subnet, CSV import, Restore) the app warns if a policy uses
    the old IP, if the new IP falls under a policy, or if the device moves into another zone – update those policies in UniFi.
  - Versions also store the zones and the custom policies; *Compare › Firewall* lists new, removed and changed policies and zones.
- **Firewall analysis** (tab *Analysis* on the Firewall page, read only): device types (from the HA integration, hostname, name,
  vendor and open ports – set by hand in the device table), zone roles (*Zone roles…*), checks per device, consistency of similar
  devices (same HA integration, vendor or type), zone and VLAN structure (merge, split, own zone), recommendations and a
  segmentation and tidiness score per zone. *Create rule…* opens the rule wizard pre-filled; *Prepare move…* sets the Virtual Network
  Override in the edit panel – nothing changes until you apply.
- **Rule chain and order:** click a zone pair in the matrix. The chain shows the rules in UniFi's order with what decides first and
  rules that never apply or have no effect. *Change the order* (⠿ or ↑ ↓) saves the order in UniFi through the official Integration
  API – this needs the option `api_key`. Rules are matched by name within the zone pair, so names must be unique there.
- **New firewall rule:** *Add firewall rule* on the Firewall page or *Create firewall rule for this client…* in a client's settings
  opens a guided wizard. A version is saved before the rule is created; rules that would block Home Assistant from the UniFi console
  are refused.
- **Relations:** fourth view. A graph of the traffic flows UniFi recorded (Insights › Flows), up to 7 days back, with filters for
  zones, VLANs, the internet, vendors, HA integrations, unexpected and suspicious traffic and traffic against the current rules.
  *Notify me in Home Assistant* checks the flows every 10 minutes and creates a notification and the event
  `clients_reservations_suspicious_traffic` (data: kind, name, ip, mac, zone, text) for scans, many blocked attempts, remote-access
  ports from IoT or guest devices and connections from untrusted into trusted zones. The same pattern is reported again after 6 hours.
- **CSV export** of the current filtered view.
- **CSV import (bulk):** Actions → *Import CSV*. Required column `MAC`; optional `Name`, `Note`,
  `Fixed IP`, `Reservation VLAN` (name or VLAN ID, empty = derived from the IP), `Reserved` (yes/no),
  `Locked to AP` (AP name or "no"), `Local DNS`. Empty cells leave values unchanged. A preview lists
  every change with a checkbox per row (plus *select all*); rows with errors (invalid MAC/IP, duplicate IP,
  IP outside the network, unknown AP) are never applied. An exported file can be re-imported as is.
  German column names from version 1.2 are still accepted.

## Configuration

| Option | Description |
|---|---|
| `unifi_host` | IP of the gateway, e.g. `192.168.1.1` |
| `username` / `password` | Local UniFi account with permission to edit clients |
| `site` | Internal site name, usually `default`. The display name (e.g. “Dream Machine Pro”) also works; with a single site it is detected automatically. |
| `verify_ssl` | Keep `false` while the gateway uses a self-signed certificate |
| `api_key` | UniFi API key (UniFi Network › Settings › Control Plane › Integrations) – only needed to change the order of firewall rules |
| `certificate_fingerprint` | SHA-256 fingerprint of the controller certificate (pinning). The app log shows the current value at start; with a pin the connection is only accepted if it matches. |
| `daily_version` | Save one version of the table per day (default on) |
| `auto_version_retention_days` | Days to keep automatic versions (daily / before bulk actions), default 30 |
| `notify_new_devices` | Notify Home Assistant about devices with a MAC never seen before (default on) |
| `notify_networks` | Only watch these networks (names or VLAN IDs); empty = all |
| `notify_random_mac` | Also notify about random/private MACs (default off) |
| `vendor_lookup` | Fill unknown vendors via macvendors.com (only the vendor prefix is sent; default on) |

Use a dedicated local UniFi account for this app rather than sharing one with another
integration, so the sessions don't log each other out.

The UI is only reachable through Home Assistant (Ingress); no port is exposed.

## Troubleshooting

When the table cannot be loaded, the red banner shows the reason and a hint:

| Message | What to do |
|---|---|
| Controller not reachable | `unifi_host` must be the IP / host name of the UniFi console without `https://`, reachable from Home Assistant. |
| Login failed (HTTP 401/403) | Use a **local** UniFi account (UniFi OS › Admins & Users › *Restrict to local access only*); cloud/SSO accounts and accounts with 2FA do not work. |
| Site not found | Set `site` to the internal name listed in the hint (usually `default`). |
| Certificate could not be verified / fingerprint mismatch | Turn `verify_ssl` off or pin the certificate with `certificate_fingerprint` (the current value is printed in the app log at start). |
| Firewall not available | The firewall page needs UniFi Network 9.0 or later with the zone-based firewall, and the UniFi account needs permission to view the firewall settings. Everything else works without it. |
| Does not look like a UniFi OS console | The app needs UniFi OS (UDM, UDR, UCG, UX, Cloud Key Gen2+, UniFi OS Server); the classic standalone controller is not supported. |

After changing the configuration, restart the app and press *Refresh*.
