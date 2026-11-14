# Changelog

## 1.32.0 – 2026-11-14

**Firewall analysis and recommendations** (new tab *Analysis* next to *Matrix*, in the look of the IP plan's CleanUp Proposal):
- **Device types** from the Home Assistant integration, hostname, name, vendor and open ports (ESPHome, Shelly, Tasmota, WLED, Zigbee coordinators, Hue, Cast, Sonos, TVs, printers, cameras, NAS …), with confidence and reasons; each can be set by hand.
- **Zone roles** (Trusted, IoT, Cameras, Servers …) guessed from the zone names, editable under *Zone roles…*.
- **Checks per device:** wrong zone for its type, no internet although it needs it, internet although it does not, Home Assistant cannot reach it, phones cannot reach it, mDNS needed.
- **Consistency of similar devices:** devices of the same HA integration, vendor or type with a different zone, internet access, rules or HA access – e.g. one Shelly that is missing from the rule the other Shellys are in.
- **Zone and VLAN structure:** empty or unused zones and networks, zones that allow each other everything (merge?), zones and VLANs that mix computers with IoT devices (split?), devices that share the same exceptions (own zone?), guest networks outside the Hotspot zone, flat networks.
- **Recommendations** with *Create rule…* (opens the wizard pre-filled) and *Prepare move…* (sets the Virtual Network Override in the edit panel).
- **Segmentation score** and **Tidiness score** (rule hygiene, like the IP plan's tidiness) per zone in the sidebar.
- The sidebar filter *Firewall analysis* finds devices with findings in the client table.

**Firewall rules:**
- **Rule chain** of a zone pair: a swimlane from left to right (✓ allow, ✕ block, 🔒 built-in) that shows which rule decides first, rules that never apply, redundant rules and rules without effect.
- **Change the order** with the ⠿ handle or ↑ ↓ and save it in UniFi (needs the new option `api_key`); a version is saved first.
- **Add firewall rule** (header button on the Firewall page, and *Create firewall rule for this client…* in a client's settings): a guided wizard – allow or block, from, to, traffic, check. It shows what decides the traffic today, warns when the rule would change nothing or another rule wins, and refuses rules that would lock Home Assistant out of the UniFi console. A version is saved first.
- Matrix cells show the ports custom rules open or close; risky Allow All pairs are marked.
- *Changed since* a version marks new and changed rules; the version list shows the number of policies; *Compare* shows when a device changed its firewall zone.
- If the zone-based firewall is switched off in UniFi, the page says so (with the number of classic rules) and how to switch it on.

**Client panel:** coloured zone strips (*Reaches* / *Reachable from*), a table with what applies to the client zone by zone and which rule decides, open ports per zone, paused rules with what would change if they were active.

**Relations** (fourth view): a graph of who talks to whom from UniFi's traffic flows (1 h – 7 days or from – to), one arrow per direction with the width of the data volume and the colour of the port. Filters: within a zone, within a VLAN, across VLANs, to the internet, same vendor, same HA integration, with Home Assistant, unexpected, suspicious, against the firewall rules, blocked by UniFi; Internet, Home Assistant, UniFi devices, NAS / server and printers can be hidden.
- **Suspicious network traffic:** scans, many blocked attempts, remote-access ports from IoT or guest devices, connections from untrusted into trusted zones – optionally as a Home Assistant notification and the event `clients_reservations_suspicious_traffic` (checked every 10 minutes).

## 1.31.0 – 2026-11-13

**Firewall (read only)** – the zone-based firewall of UniFi Network 9+ next to the clients and the IP plan:
- New page **Firewall** (switch *Clients / IP Plan / Firewall*): the **zone matrix** like in UniFi (Allow All, Block All, Allow Return and the number of custom policies per zone pair). Click a cell, a row or a column header to filter the **policy list** below; filters for IPv4, IPv6, built-in, custom and paused policies, and a search. A click on a policy shows source and destination in full, with links to the devices.
- The sidebar lists the **zones** with their networks, devices and policies; *Show … devices* opens the clients of a zone.
- New column and sidebar filter **Firewall zone** in the client table (at the end of the built-in view *All*; drag it where you like).
- Detail panel: new section **Firewall** with the device's zone and every policy that names it – by MAC, by its IP, through an address group or its network. A click opens the policy in the firewall view.
- **IP changes check the firewall:** CleanUp, Replace subnet, CSV import, Restore, the edit panel and inline edit warn when a policy uses the old IP (e.g. *Allow NAS 2 NVR → 192.168.1.10*), when the new IP falls under a policy, and when a device moves into another zone.
- **Versions** also save the zones and the custom policies. *Compare* has a new filter *Firewall* with new, removed and changed policies and zones.
- **Hits** and **Last Hit** per policy, read from UniFi's own counters (they count also without logging); sortable, and shown per policy in the detail panel.
- Sources and destinations as UniFi defines them: devices, IPs (also bound to one MAC), address groups, networks, domains, regions and IPv6 interface IDs, each also inverted (*Not …*).
- *Export JSON* on the firewall page downloads zones, policies and groups as the app read them – handy to check the view against UniFi.
- Nothing is written to the firewall. Consoles without the zone-based firewall show a hint instead.

## 1.30.0 – 2026-11-01

- **New standard views** in *Clients* (view menu): **Problems** (problems, IP ≠ fixed IP, not in override VLAN, duplicates, wrong network), **New devices** (first seen in the last 7 days, newest first), **Offline with fixed IP** (reservations of devices that are gone, oldest first), **Home Assistant** (devices known to HA, errors first) and **WiFi** (SSID, AP, AP lock, VLAN).
- New filter **New** in the sidebar: first seen today / in the last 7 / 30 days.

## 1.29.1 – 2026-11-01

- **Auto-Scale Network:** with UniFi's "Auto-Scale Network" on, UniFi manages subnet and DHCP range itself. CleanUp now says so for the affected networks and switches Auto-Scale off when it sets the pool; *Undo everything* switches it on again.
- Networks with Auto-Scale show an *Auto-Scale* badge in the network list and in the CleanUp table.
- CleanUp for all networks: a network with a problem (e.g. pool would get too small) is left out automatically instead of blocking the others.
- Translations: the remaining English sentences in the CleanUp steps are translated.

## 1.29.0 – 2026-11-01

Safer CleanUp, after a review by a network specialist:
- **New strategy "Pool only"** (now the default): the DHCP pool starts behind the highest fixed IP – no device changes its address.
- **Addresses in use are respected:** new addresses are never taken from devices seen online, UniFi devices, devices with their IP set on the device (they never renew), clients seen in the last 14 days or the Home Assistant host – in every strategy. Devices with an IP set on the device that would end up in the new pool are listed with a warning.
- **Home Assistant host without a fixed IP** (IP set on the host) is protected too, with a hint to reserve it.
- **DHCP pool:** its end stays where it is, it keeps at least max(30, 2 × dynamic clients) addresses, it never covers the gateway. Networks with the DHCP server off or relayed are not offered. The server re-reads the network right before the change and checks afterwards that UniFi took it.
- **Stop on failure:** if a pool change fails, no fixed IP is changed.
- **Waits for the gateway** to apply a new pool before devices are reconnected.
- **Check afterwards:** the result shows which devices already use their new IP, which take it at their next renewal (with the network's lease time) and which got a pool address instead.
- **Undo everything:** after a CleanUp, one click sets the pools and fixed IPs back to how they were (same safe order, Wi-Fi clients reconnected).
- *Forget client* shows a 5 s progress bar, then refreshes; a version is saved before (name, note and reservation can be restored).

## 1.28.2 – 2026-11-01

Fixes from a code review:
- **CleanUp no longer shrinks the DHCP pool** to make room for the Home Assistant host. The fixed block grows up to the host only automatically and only if the pool keeps a useful size; otherwise the host stays inside the pool as a reservation. A value typed into *up to* is never changed.
- The Home Assistant host and the gateway are protected everywhere: inline edit asks for the same confirmation (gateway: not editable), *Replace subnet* and *CSV import* skip them with a note.
- *Escape* in the IP picker or the Home Assistant warning no longer closes the device panel (and loses unsaved edits).
- *Forget client*: UniFi can take long – the app now waits longer, checks whether the client is gone and reloads; the panel closes instead of showing a time-out error. After *Remove reservation only* the panel shows the fresh state.
- Device names, notes and network names are never translated (a device called "Light" stayed "Light").
- Reserving the address the Home Assistant host already uses no longer shows the warning; host detection is stricter (Supervisor data first, name only as fallback).
- *Refresh* is an icon button that always stays visible; on narrow screens *Versions* is hidden first.
- Translations are cached per app version (faster start); "Conflict" in the hover box is translated.

## 1.28.1 – 2026-11-01

- IP plan tabs (*Current* / *CleanUp proposal*): calmer look – a box with background and slightly rounded tabs.

## 1.28.0 – 2026-11-01

- **More languages:** Nederlands, Polski, Português (Brasil) and Svenska – in the app and on its configuration page in Home Assistant. *Auto* picks them from the browser language.
- **Network check & link** (device details): the port check comes first, ping below it. Open web ports (HTTP/HTTPS) are larger blue buttons with a link icon and open the device's web page in a new tab.

## 1.27.1 – 2026-11-01

- The sidebar footer (version, language, theme) stays on one line in every language: the language is picked with a flag button (menu with flag and name, 🌐 = Auto), the theme switch shows only its icon (the tooltip names the theme).

## 1.27.0 – 2026-11-01

- **Languages:** new language selector at the bottom of the sidebar next to the theme switch – *Auto* (browser language), English, Deutsch, Schwiizerdütsch (Mundart), Français, Italiano, Español. The translations live in `app/static/i18n/<lang>.json` (key = English text, `{0}`, `{1}` … = placeholders) – corrections and new languages are welcome as pull requests.
- The app's configuration page in Home Assistant is translated too (German, French, Italian, Spanish).
- Dates and times follow the chosen language.

## 1.26.0 – 2026-11-01

- **Gateway:** the fixed IP settings are locked in its edit panel – its address comes from the network settings in UniFi. A note says so; *Remove reservation only* is hidden.
- **Home Assistant host:** a note in its edit panel, *Forget client* is hidden (for the gateway too), and changing or removing its fixed IP needs a confirmation: tick *I am aware that …* and click *Change anyway*.
- The *Free: …* suggestion link below the IP field is gone – use *Select* to pick an address in the IP plan.

## 1.25.1 – 2026-11-01

- *Pick a free IP*: no focus ring around *No icons* when the popup opens.

## 1.25.0 – 2026-11-01

- **Pick a free IP visually:** new *Select* button next to the IP field in the device settings. It opens the IP plan of the chosen network – click a free address to select it and confirm with *OK* (double-click takes it right away, *Cancel* keeps the field as it was); the icon switch (None / UniFi / Home Assistant) and the cell size work as in the IP plan; the DHCP pool, fixed IPs, dynamic clients, the gateway and the Home Assistant host are shown as in the IP plan, and the hover box tells who uses each neighbour. The address in the field and the device itself are outlined.

## 1.24.1 – 2026-10-31

- CleanUp: *Fixed IPs up to* can go back to the automatic suggestion – clear the field or click *Auto*. The suggestion is shown greyed in the empty field (`auto · 99`). Before, clearing the field set the end to .2.

## 1.24.0 – 2026-10-31

- **CleanUp never moves the Home Assistant host.** All integrations point to its address, so renumbering it would break them. The app reads the host's addresses from the Supervisor (new `hassio_api` permission, info calls only) and also recognises clients named *homeassistant*, *hassio* or *home-assistant*. The fixed block always reaches up to its address, so it never ends up in the DHCP pool either.
- The gateway (from the network settings) has its own colour (violet) in all plan grids and legends and a note in the hover box; a gateway that UniFi only lists as a client is never moved either.
- The Home Assistant host is shown in Home Assistant blue in the IP plan, the CleanUp grids and the hover box, with its own legend entry.

## 1.23.6 – 2026-10-31
- Wording: Home Assistant calls add-ons **apps** since 2026.2 – texts, hints and documentation now say *app* (e.g. *Settings › Apps › IP Manager for UniFi › Configuration*, *app log*). Nothing else changes; existing installations update as usual.

## 1.23.5 – 2026-10-31
- **CleanUp runs in a safer order:** 1 · save a version, 2 · change the DHCP pools, 3 · reconnect the conflicting (pink) Wi-Fi clients so they move into the new pool and free their address, 4 · move the fixed IPs, 5 · reconnect the moved Wi-Fi clients. Changing the pools first means no new device gets a dynamic address in the fixed block while the moves run; moved devices find their new address free.
- One confirmation up front instead of a second one for the pool at the end: *Apply …* lists all steps in this order, *Confirm & apply* runs them. Changing an option goes back to editing.
- CleanUp strategy order: *Minimal · Grouped by HA integration · Grouped by vendor*.
- New measure *Reconnect conflicting Wi-Fi clients* (on by default); wired or offline conflicts are listed and clear at their next DHCP renewal.

## 1.23.4 – 2026-10-31
- **Hover box in the IP plan:** a larger card per address – network, device(s) with icon, MAC, vendor, online status, fixed and current IP, Home Assistant integration – and in the CleanUp Proposal the concrete changes: who moves away or arrives (from → to, reconnect), conflicts (pink) and whether the address joins or leaves the DHCP pool.
- **Conflicts are pink** everywhere in the CleanUp Proposal – cells, arrows, the *Conflicts* column and summary – so they stand out from the orange dynamic leases.
- Conflicts are now also shown with *Preview changes* off (in the resulting plan), not only in the preview.
- The summary pill *Dynamic in fixed range* is now called *Conflicts*.

## 1.23.3 – 2026-10-31
- *Preview changes*: **conflicts are orange** – addresses in the new fixed block that a dynamic client currently uses get a bold orange frame, and a move onto such an address is drawn as an orange arrow on top of the others.
- UniFi devices with an IP set on the device are no longer counted as dynamic conflicts.

## 1.23.2 – 2026-10-31
- **Play** moved up into the *Preview changes* switch (appears when the preview is on).
- CleanUp for all networks: new column **Conflicts** – dynamic clients that currently use an address inside the new fixed block (0 when none; hover for the devices). They get a pool address at their next DHCP renewal.
- Uniform rounded look: the *Current / CleanUp Proposal* tabs have round tops, all switches are rounded pills and the selected option is blue – e.g. the CleanUp strategy *Minimal / Grouped by vendor / Grouped by HA integration* and the naming options in *Names from Home Assistant*.

## 1.23.1 – 2026-10-31
- **IP plan cell size Small / Large** (switch next to the icon switch, remembered): *Small* is a compact view – lower cells and 32 addresses per row when there is room; *Large* has tall cells with big icons. The height no longer changes with the icons: the address sits on the left, the icon on the right using the full inner height.
- **Fixed IP, in use** is now a blue frame with a light transparent fill (like *Dynamic, in use*) instead of a solid block; *Fixed IP, not in use* has a dashed frame.

## 1.23.0 – 2026-10-31
- **CleanUp Proposal for all networks at once:** select *All networks* and open *CleanUp Proposal*. Same strategy for every network; the fixed block is suggested per network or set as a percentage of each network (*Fixed block … %*). A table lists every network with something to tidy – fixed block, pool change, moves and tidiness before → after – each with a checkbox; the grids (and *Preview changes* with arrows) follow below. Every device stays in its network.
- **Overall tidiness** (average over all networks, weighted by their fixed IPs) on the *All networks* card, in its title and before → after in the multi-network proposal.
- Applying moves runs in a safe order: an address that is still reserved by another moving device is freed first, swaps go through a temporary free address (no more “already reserved” errors when renumbering in blocks). All pool changes are confirmed together at the end.
- Clicking a device in the CleanUp grid opens its details next to the proposal; the arrows follow when the panel opens.
- Icons in the *Clients / IP Plan* switch (list / grid). The header adapts to narrow widths (shorter labels instead of cutting off buttons).
- Fix: in the single-network proposal the pool change was skipped after the moves since 1.22.0.

## 1.22.1 – 2026-10-31
- **All networks** as first entry under *Networks*: all networks one below the other to scroll through (click a network title to show only that one), with totals and an average tidiness. *CleanUp Proposal* is disabled there – it works per network. The legend stays visible at the top while scrolling.
- **Preview changes** is now a prominent switch with an eye icon in the CleanUp options (blue when on).
- The tabs **Current / CleanUp Proposal** sit on a shared track like real tabs – the selected one is blue like the active switches and ends exactly on the header line, which is blue in the IP plan.
- IP plan with icons: the address now sits small in the top-left corner of each cell and the icon (UniFi or Home Assistant logo) is shown larger in the middle.

## 1.22.0 – 2026-10-31
- **IP Plan is a full page:** a switch *Clients / IP Plan* at the left of the header replaces the IP plan button and popup. In *IP Plan* the sidebar shows the networks on top and **all filters and the search below** – devices outside the filters are greyed out in the plan (e.g. search *shelly* or *Issues › In DHCP pool*). Clicking an address opens the device (or a new reservation) in the detail panel while the plan stays open, and the plan refreshes after saving. The choice is remembered. The tabs *Current / CleanUp Proposal* sit in the header next to the switch, the icon switch on the right.
- The detail panel now pushes the list / plan aside on tablets and narrow windows too instead of covering it (only phones still show it as an overlay).
- **CleanUp Proposal is a tab of the IP plan** instead of a separate dialog: *Current* (default) and *CleanUp Proposal*. The cleanup options (strategy, fixed range, pool, tidiness before → after) appear in the header of that tab, the measures below the grid, and *Apply* in the footer. Switching networks keeps you in the tab.
- **Preview changes** (checkbox) replaces the *Proposal / Changes* tabs: off shows the resulting plan, on shows every move as an arrow and plays the animation.
- The move arrows are now drawn in front of the address cells.

## 1.21.0 – 2026-10-31
- **IP plan:** the networks are now a pane on the left – one card per network with subnet, DHCP pool, fixed / in pool / dynamic / free and utilization. The selected network's title is larger and sits next to the *CleanUp Proposal* button.
- New **Tidiness score** (0–100) per network: fixed IPs outside the DHCP pool (up to 50 points), close together (30 – gaps of up to 10 addresses, e.g. one block per vendor, are fine) and without conflicts or devices on another IP (20). Green from 80, orange from 50, red below; the tooltip explains the points.
- **CleanUp Proposal** shows the tidiness before → after the checked measures, e.g. *56 → 100*.

## 1.20.0 – 2026-10-31
- The UniFi device icon now sits in the **UniFi Status** column, left of the online dot – the same layout as the Home Assistant column (logo + status). The *Name* column shows only the name.
- The *Columns* button is gone: **Clear filters** and **Customize columns** are now at the bottom of the filter pane, next to the version.
- IP plan: **CleanUp Proposal** moved up, right next to the network title. The icon switch *No icons / UniFi / Home Assistant* is shown as a clear pill switch.
- IP plan with *Home Assistant* icons: devices without a Home Assistant integration show their UniFi icon instead of nothing.

## 1.19.3 – 2026-10-31
- The project moved to the GitHub organization **ha-unifi-friends**: https://github.com/ha-unifi-friends/ha-ip-manager-unifi. The old repository URL keeps working (GitHub redirects it), so existing installations need no change.

## 1.19.2 – 2026-10-31
- *Names from Home Assistant*: **Already in the name** – *Add anyway / Skip / Replace* (replaces the *Skip if already in the name* checkbox). *Replace* first removes the area or integration from the name and then adds it at the chosen place, e.g. *Kitchen Light* with area as suffix → *Light-Kitchen*, *shellyLEDmatrix* with integration as prefix → *Shelly-LEDmatrix*. A name is never stripped completely.
- *Names from Home Assistant*: the column is now called **New name** (it includes area / integration when chosen). Fix: the help text no longer breaks into separate lines around *UniFi-managed*.

## 1.19.1 – 2026-10-31
- **Names from Home Assistant** never renames UniFi devices (access points, switches, gateway) or UniFi Protect cameras: UniFi is the golden record for their names, a new name would be synced back into Home Assistant. They are listed as *UniFi-managed* and cannot be selected.

## 1.19.0 – 2026-10-31
- **Names from Home Assistant** can add the **area** and/or the **integration** to the name – each *Off / Prefix / Suffix* – with a separator of your choice (`a b`, `a-b`, `a_b`, `a - b` or none), e.g. *Office Desk Lamp* or *Desk Lamp - ESPHome*. The proposed names, the *Rename / Same name* counts and an example update live; *Skip if already in the name* (on by default) does not add a part the name already contains – e.g. *shellyLEDmatrix* gets no *Shelly*. *CamelCase* starts every word upper case and continues lower case (e.g. *OfficeDeskLamp* with no separator). The setting is remembered per browser.
- Integrations are shown with their readable name (ESPHome, Shelly, UniFi Protect …).

## 1.18.3 – 2026-10-31
- *Not in override VLAN* now also lists offline devices whose last known network differs from their virtual network override (info *Last seen outside <network> (override)*; online devices keep the warning).

## 1.18.2 – 2026-10-31
- The **Fixed IP** is shown in yellow when the device's current (or last known) IP address is a different one – e.g. it has not renewed its lease yet. The tooltip shows the address in use.
- New filter **Issues › IP is not Fixed IP** for exactly these devices.
- **Virtual network override** in the device *Settings* (as in the UniFi app): put a client into a network regardless of its Wi-Fi or switch port. A fixed IP follows the chosen network; the *Overview* shows the active override.
- New column **Virtual network override** (between *Network* and *WiFi*, sortable and filterable); shown in yellow when the device is not in that network.
- New issue **Not in <network> (override)** and filter **Issues › Not in override VLAN**: the device is online but not in the network its virtual network override points to.

## 1.18.1 – 2026-10-31
- Fix: clicking a device further down the list made the table jump back to the top (Safari / Home Assistant app). Selecting a device now only moves the highlight; the scroll position stays.
- New default column order (built-in view *All* and *Columns › Restore*): UniFi Status, Home Assistant Status, IP, Name, IP address, Fixed IP, MAC address, Vendor, Connection type, Connection, Locked to AP, Network, WiFi, Issues, Reservation VLAN, First seen, Last seen, Note. *All* is sorted by IP address; devices without an IP come last.

## 1.18.0 – 2026-10-31
- Columns **UniFi Status** (formerly the unnamed status dot) and **Home Assistant Status**: the header shows only an icon so the columns stay narrow (full name as tooltip and in *Columns*). Both can now be sorted and filtered (UniFi: online / offline; HA: OK, partly unavailable, unavailable, integration error, disabled, known / not in HA).
- The built-in view *All* shows *Home Assistant Status* as second column.
- **Versions** record the Home Assistant status and integrations of every device (info only). *Compare* and the device *History* show changes as *HA status* (e.g. “OK (shelly) → Unavailable (shelly)”); the version CSV contains it too.
- Fix: with the filter pane hidden on a desktop, the table was squeezed into the narrow sidebar column.
- New column **IP** with a pill *Fix* (DHCP reservation, or a static IP set on a UniFi device) or *DHCP*; sortable and filterable. It is the third column of the built-in view *All* (use *Discard changes* on *All* if you changed that view before).
- **Light / dark switch** at the bottom of the filter pane: *Auto* (follows the system / browser) → *Light* → *Dark*, remembered per browser.
- The **add-on version** is shown at the bottom left of the filter pane.
- The toolbar always stays on one line: when the detail panel is open (or the window is narrow), the buttons collapse to icons instead of wrapping into a second row.
- **Open in HA** opens the Home Assistant device page in a new tab.
- **IP plan icons:** the *Show icons* checkbox is now a switch *No icons / UniFi / Home Assistant* on all IP plan views; *Home Assistant* shows the logo of the device's integration.
- **CleanUp Proposal:** new strategy *Grouped by HA integration* – fixed IPs in blocks per Home Assistant integration (e.g. all Shelly, then all ESPHome), devices not in HA last.
- **Home Assistant status:** buttons, events and updates no longer count – e.g. the UniFi integration's PoE *power cycle* buttons are unavailable on every port without PoE, so all switches and gateways showed *partly unavailable*. For a partly unavailable device, the detail panel lists the unavailable entities.

## 1.16.0 – 2026-10-31
- **Home Assistant link per device** (matched by MAC): new section *Home Assistant* in the detail panel with the integration logo, the HA device name, area and status (OK / partly unavailable / unavailable / integration error / disabled) and **Open in HA ›**, which opens the device page in Home Assistant.
- New column **Home Assistant** (integration logos + status dot) and sidebar filter *Home Assistant* (known / not known / unavailable, per integration). For clients, the UniFi integration's device trackers are ignored; for UniFi devices (access points, switches, gateway) the UniFi integration device is linked.
- Integration logos come from brands.home-assistant.io (allowed in the Content-Security-Policy).

## 1.15.1 – 2026-10-31
- Replace subnet, CSV import and restore: *Reconnect Wi-Fi clients afterwards* is now on by default.
- Conflicts in the previews (address already reserved, address in use, duplicate) link to the other device: *Open <device> ›* shows its details, and closing the panel returns to the dialog with everything still filled in.
- Dialog footers keep the buttons on one line on wide screens.

## 1.15.0 – 2026-10-31
- Renamed to **IP Manager for UniFi** (sidebar panel: *IP Manager*). Nothing else changes: the add-on slug, the stored versions, views and settings, and the event `clients_reservations_new_device` stay the same.

## 1.14.0 – 2026-10-31
Security hardening after a code review (SAST-style).
- **Certificate pinning** for the UniFi controller: new option `certificate_fingerprint` (SHA-256). At start the add-on logs the controller's fingerprint so it can be copied; with a pin, the connection (and the UniFi login) is only accepted if the certificate matches.
- **CSV exports** (table, comparison, version): cells starting with `=`, `+`, `-`, `@`, tab or CR get a leading `'` so spreadsheets do not execute them as formulas (device hostnames are chosen by the devices themselves). The CSV import removes this prefix again.
- **Ping, port check and Wake on LAN** only accept addresses inside the UniFi networks; the port check no longer follows HTTP redirects when it reads the page title.
- POST requests must be `application/json` (CSRF protection in depth).
- New-device notifications escape Markdown in names, vendors and networks.
- Unexpected errors return a generic message (details in the add-on log); responses carry `X-Content-Type-Options`, `Referrer-Policy` and a Content-Security-Policy for the UI.
- At most 32 requests in parallel and at most 2 pings at a time; the device history caches the versions; the files for known devices and vendors are written under a lock.
- Fix: the issues filter could fail right after a refresh while unknown vendors were being filled in.
- **Better error messages**: the banner now shows a hint what to check (controller address, local account / 2FA, certificate, site) and where to change it.
- **Site**: besides the internal name (usually `default`) the site can now be given by its display name (e.g. “Dream Machine Pro”); with only one site on the controller it is used automatically. Previously a name with spaces caused “URL can't contain control characters”.

## 1.13.2 – 2026-10-31
- IP plan: all networks are always listed; the whole dialog scrolls instead of the network table.
- The blue icon top left now hides and shows the filter pane – on phones as a slide-in menu (hamburger), on desktop to gain room for the table (remembered per browser).
- CleanUp Proposal: *Cancel*, *✕*, Esc and *Close* return to the IP plan.

## 1.13.1 – 2026-10-31
- CleanUp Proposal: tabs **Current / Proposal / Changes** to switch quickly between today's layout and the proposal. *Changes* draws an arrow from every old to the new address, marks the future DHCP pool and plays the moves as an animation (*▶ Play moves*).
- The moves are now listed directly below measure 1 (instead of in a separate tab); *Reconnect* is on by default.
- **Show icons** switch in every IP plan view – the device icon in each cell (remembered per browser).
- Button with a broom icon.

## 1.13.0 – 2026-10-31
- **CleanUp Proposal** (IP plan › *CleanUp Proposal* for the selected network): suggests a tidy layout – fixed IPs in a block at the start (e.g. .2 – .99), the DHCP pool behind it (e.g. .100 – .254), sized from the current number of fixed and dynamic clients.
  - *Proposal* tab: the IP plan after the cleanup (moved addresses highlighted, freed addresses dashed) and the measures, each with a checkbox: move fixed IPs, reconnect affected Wi-Fi clients, change the DHCP pool.
  - Strategies: *Minimal* (only fixed IPs in the future pool, on the gateway or used twice move) or *Grouped by vendor* (renumber in blocks of ten per vendor). The end of the fixed range can be adjusted.
  - *Moves* tab: like Replace subnet – checkbox per device, sortable, hint for devices known to Home Assistant (IP-based integrations may need an update).
  - The DHCP pool is changed in UniFi as the last step and only after a second confirmation. A version is saved before anything changes.

## 1.12.1 – 2026-10-31
- **Ping** in the detail panel (section *Network check*, next to the port check): 4 ICMP echos from Home Assistant's network with replies and min / avg / max round-trip time.
- **Vendor lookup**: the detail panel links the MAC to maclookup.app (*Lookup vendor*, only the vendor prefix is sent); random / private MACs are marked as such instead.
- Empty vendors are filled in the background via macvendors.com (option `vendor_lookup`, default on): only the 3-byte prefix is sent, max. 1 request per second, results are cached in the add-on. Such vendors show “(lookup)” in the table.

## 1.12.0 – 2026-10-31
- **Open ports** in the detail panel: *Check ports* tests about 30 common TCP ports (FTP, SSH, Telnet, HTTP/HTTPS and typical web UI ports such as 5000/5001, 8080, 8123, 8443, MQTT, RTSP, SMB, RDP, VNC, printers …) on demand, from Home Assistant's network. Web ports become links (with the page title); the result is stored per device.
- New issues *Telnet open* / *FTP open* and the filter *Issues › Insecure ports*.
- The check runs in small, slowed-down batches and only one device at a time, so UniFi IDS/IPS does not see a port scan burst.

## 1.11.0 – 2026-10-31
- **IP plan** (header): utilization per network (subnet, DHCP pool, fixed IPs in/outside the pool, dynamic leases, free addresses outside the pool, utilization bar) and an address grid per network – fixed IP in use / not in use, dynamic, conflict, gateway, free in pool, free. Click a device to open it, or a free address to reserve it.
- **New-device notifications**: the add-on checks every 2 minutes for MACs it has never seen and creates a Home Assistant notification plus the event `clients_reservations_new_device` (for your own automations). Options `notify_new_devices` (default on), `notify_networks` (names or VLAN IDs, empty = all) and `notify_random_mac` (default off). On the first start all current devices are learned silently.
- **Duplicates**: new issues *Duplicate name* and *Duplicate DNS name*; filter *Issues › Duplicates (IP / name / DNS)*.
- **Export hosts file** (Actions): fixed IPs with their local DNS record and a host name derived from the device name – for `/etc/hosts`, Pi-hole or AdGuard.
- **Network rules** (Actions): e.g. *Vendor contains “Espressif” → IoT*. Devices that match a rule but sit in another network get the issue *Expected in …*; filter *Issues › Wrong network*.
- **Wake on LAN** button in the detail panel for offline wired devices (via Home Assistant's `wake_on_lan` service if set up, plus a magic packet to the subnet broadcast and the device IP).
- New sidebar filter **Inactive**: not seen for more than 30 / 90 / 365 days, or never seen – combine with *Reservation › With fixed IP* to find stale reservations.

## 1.10.0 – 2026-10-31
- **Reconnect** (UniFi *kick-sta*): online Wi-Fi clients whose IP changes can be disconnected briefly so they request the new IP right away.
  - Edit: *Reconnect device after saving* (on by default) when the new fixed IP differs from the device's current IP. Wired devices show that they take the new IP at their next DHCP renewal.
  - Replace subnet and CSV import / restore: *Reconnect N Wi-Fi clients afterwards* in the footer (off by default); the preview shows per row whether the device will be reconnected. Clients are reconnected one per second; the table refreshes after about 20 s.
- Preview tables (CSV import, restore, replace subnet, names from HA, compare) can be **sorted by clicking a column header**; click again to reverse.

## 1.9.8 – 2026-10-31
- Detail panel: the *Edit reservation* button is replaced by **Open `http://<IP>`** (plus *HTTPS*), which opens the device's web interface in a new tab. It uses the current IP when the device is online, otherwise the fixed IP. Editing stays on the *Settings* tab.

## 1.9.7 – 2026-10-31
- **History** in the detail panel: for each device a timeline of what changed and in which version (name, note, fixed IP + network, AP lock, local DNS; IP and network moves in grey), plus the changes made since the latest version.

## 1.9.6
- Bulk dialogs (CSV import, restore, replace subnet, names from HA) now say in the footer that a version of the current settings is saved first, so every bulk change can be undone via *Versions › Restore*.

## 1.9.5
- Baseline filter simplified: one entry *Changed since …* per version (new devices + changed name, note, fixed IP, AP lock or DNS), for the baseline (★) and the latest versions, max. 5. IP or network moves alone do not count.
- Fix: an active baseline filter no longer hides all rows while the versions are still loading.

## 1.9.4
- The *Baseline* filter is always shown: with an explicit baseline it offers *New since …* and *Known at baseline*; otherwise *New since …* for each of the 5 latest versions; without versions a short hint.

## 1.9.3
- *Daily automatic version* moved from the Versions dialog to the add-on configuration (`daily_version`, default on).

## 1.9.2
- Fix: *Compare* reported every fixed IP without a stored network as changed (e.g. "192.168.1.23 → 192.168.1.23 · Management"). The reservation network is now resolved the same way on both sides; this also fixes already saved versions.
- New option *Keep automatic versions (days)* (`auto_version_retention_days`, default 30) in the add-on configuration.

## 1.9.1
- Replace subnet: dropdowns with the known networks above *From subnet* (with number of fixed IPs) and *To subnet*; picking a target network also sets *Target network*. Placeholders now read “e.g. …”.

## 1.9.0
- **Versions**: save the table as a version (manual, optional *Daily automatic version*, and always automatically *Before …* every bulk action: CSV import, replace subnet, names from HA, restore). Automatic versions are kept 30 days, manual / pinned / baseline until deleted. Rename, pin, delete, download as CSV.
- **Compare** a version with the current table or two versions with each other: new devices (with first seen, vendor, network, random-MAC hint and “same hostname as a removed device – probably a new MAC”), removed, changed (name, note, fixed IP, AP lock, DNS), IP changed and moved network. Export the comparison as CSV; *Open* jumps to the device.
- **Restore** a version through the import preview (checkbox per row); choose names, fixed IPs, notes, AP lock, DNS and re-creating forgotten clients. Devices added after the version are left untouched.
- **Baseline**: mark a version as baseline → sidebar filter *New since …* / *Known at baseline*. New column *First seen*.
- Header: *Names from HA*, *Replace subnet*, *Import CSV* and *Export CSV* are grouped under an **Actions** menu; on narrow screens the buttons show icons only.

## 1.8.1
- Fix: read the Home Assistant API token also from the s6 environment store (Names from HA failed with "API not available").

## 1.8.0
- **Inline edit** (toggle in the header): edit Name and Fixed IP directly in the table. Enter / ↑ / ↓ move to the next row, Esc reverts, an empty Fixed IP removes the reservation. The network is taken from the IP.
- **Names from HA**: proposes names from the Home Assistant device registry (matched by MAC), with preview, checkbox per row and “near match” for devices whose Wi-Fi/Ethernet MAC differ in the last byte. The add-on now has access to the Home Assistant API for this.
- Built-in view *DHCP Reservations* now includes *Last seen*.
- *Last seen*: “Never” when the controller has no timestamp; older than 24 h shown as YYYY-MM-DD.

## 1.7.0
- Renamed to **Clients & Reservations**; the count reads "Devices" (UniFi devices are included).
- **Views** (SharePoint-style): built-in *All* and *DHCP Reservations* (MAC, Name, Fixed IP, IP; only clients with a fixed IP). Save changes, save as new view, rename, delete, discard changes, set default. A view stores visible columns, column order, column filters, sidebar filters and sort. Views are stored in the add-on, so they are shared across browsers and devices.
- Dates in YYYY-MM-DD format (table, detail panel with exact last-seen time, CSV export).
- **Replace subnet**: move all fixed IPs from one subnet to another while keeping the host part (e.g. 192.168.30.x → 192.168.31.x), with preview, checkbox per row and checks for missing target network, gateway address, existing reservations and duplicates.

## 1.6.0
- Columns can be reordered by dragging the column header directly (a short click still sorts).

## 1.5.1
- Gateway shows "Internet (WAN)" as connection instead of its WAN port.

## 1.5.0
- UniFi devices (access points, switches, gateway) now show their real status, IP, network and uplink port (read from `stat/device`); they were shown as offline before. New sidebar filter *Type: Clients / UniFi devices*.
- Device pictures from Ubiquiti's fingerprint catalogue (same as the UniFi UI), with the built-in icon as fallback.
- Column order can be changed in *Customize columns* by dragging the grip (or arrow keys on the grip); remembered per browser, *Restore* resets it.

## 1.4.0
- Header bar at the top (deliberate deviation from UniFi): client count, last update, Refresh, Columns, Import CSV, Export CSV, Add reservation.
- Customize columns: show/hide each column (All, Restore, Done), remembered per browser. Also reachable from the sidebar.

## 1.3.0
- Everything in English: UI, messages, configuration, documentation.
- CSV export uses English column names; the import accepts both English and the German names from 1.2.

## 1.2.0
- CSV import with preview (new / change / unchanged / error), checkbox per row incl. select all, template download.
- Export also contains the local DNS record; an exported file can be re-imported unchanged.

## 1.1.0
- Look & feel modelled on UniFi Network: filter sidebar with counts, detail panel (overview/settings), dark/light.
- Connection type column (WiFi / Wired / Teleport), switch port for wired devices, WiFi SSID.
- Locked to access point: show, filter, set/clear; issue when a device is not on its locked AP.
- VLAN override is shown.

## 1.0.0
- First version: sortable/filterable table, edit, new, remove reservation, forget client, CSV export.
