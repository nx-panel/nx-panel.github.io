---
title: "Changelog"
description: "What changed in each release of the panel, the node and the official addons, and what to do when you update."
order: 7
tags: [reference]
---

# Changelog

The panel and the node have their own version numbers. **Update the panel
first, then the nodes** ([[start/update]]). Downloads and full notes are on
the releases pages of [the panel](https://github.com/nx-panel/panel/releases)
and [the node](https://github.com/nx-panel/node/releases).

## Panel

### v0.1.2 — DATE {#panel-v0-1-2}

Installs from the update button ([[use/panel-update]]). Take a backup first,
and update the panel before the nodes: this panel works with nodes on v0.1.1,
and node v0.1.2 brings OpenConnect DTLS back and the WARP+ licence key. A node
from before refuses a WARP outbound that carries a key, so update the nodes
before you set one. **Everyone logs in once more** after the update, because
the login cookie changed ([[how/security#sessions]]).

- **Signed releases:** from this release on, the update button installs only
  releases NX has signed, and checks the signature before it takes a backup.
  A release published but not signed yet is shown as available and refused
  (*not signed yet*), and the panel is left untouched
  ([[use/panel-update#only-signed-releases]]).
- **The daily release check asks nx-panel.org first**. Without an answer it asks
  GitHub ([[start/update#what-nx-panel-org-records]]).
- **Import outbounds and endpoints:** paste share links to get outbounds; drop
  a WireGuard `.conf` or an `.ovpn` to get an endpoint; or sign in to NordVPN or
  PIA and pick servers from their live list to get WireGuard endpoints. Every
  row is shown before anything is saved, and credentials are not stored
  ([[use/outbounds#import]], [[use/endpoints#import]]).
- **ShadowTLS** serves through a detour set in the inbound's
  **Listen options** ([[protocols/more#shadowtls]]); **Shadowsocks 2022** keys
  fit their method, and a key of the wrong length is refused
  ([[protocols/shadowsocks]]); **MTProxy** has a FakeTLS site per inbound, and
  users get a Telegram link ([[protocols/more#mtproxy]]).
- **Add or remove a template on many users** at once, from the selection menu
  or on the current filter ([[use/user-operations#add-or-remove-a-template]]).
- **An inbound's place in subscriptions** is set by an order number on its
  Basic tab ([[use/inbounds#client-settings]]).
- **Usage by month, and as CSV**, for a node's per-user usage and a user's
  usage by node; per-user traffic older than 30 days is kept per day
  ([[use/nodes#stats-health-history-and-uptime]]).
- **Config files keep their extension** on Safari and Chrome on Android, which
  saved a `.conf` or an `.ovpn` as `.conf.txt` or `.ovpn.txt`
  ([[addons/sub-themes]]).
- **WARP:** a **WARP+ licence key** on a WARP outbound (needs node v0.1.2); a
  template whose WARP outbound names one profile on two or more nodes is
  warned of; no generate button on the WARP key fields
  ([[use/outbounds#warp]]).
- **OpenVPN:** sing-box subscriptions pin the server certificate's name, as
  the `.ovpn` does, and a certificate with no name is pinned by its
  fingerprint, so users download their file again after a renewal
  ([[protocols/openvpn]]).
- **Safer by default:** the panel refuses writes sent by a page on another
  origin, and over HTTPS the login cookie is `Secure`, bound to its route's
  path and `SameSite=Strict`; changing a route's prefix signs everyone out
  there ([[how/security#sessions]]). Nothing in a config can start a program
  on a node: OpenConnect's wrapper paths, Tor's executable path, extra
  arguments and plugins, and Tailscale's SSH server are refused on save
  ([[use/endpoints#connecting-out]]). A template that selects one tag twice is
  refused, and so is an update that drops a row its routing still names
  ([[use/templates#checked-on-save]]).
- A weekly reset anchored on Sunday stays on Sunday. Accounts and plans
  created with a Sunday anchor before were stored with Monday; check them
  ([[use/users#reset-cycles]]).
- Node health history is folded again on panels with a long history, and an
  outbound check through a server that never answers says *timed out*.
- Built with Go 1.27.4, which fixes a security issue in its HTTP library.

### v0.1.1 — 5 November 2026 {#panel-v0-1-1}

Installs from the update button ([[use/panel-update]]). Take a backup first,
and update the panel before the nodes: this panel works with nodes on v0.1.0,
and node v0.1.1 is needed only for the new certificate features on a node.

- **OpenVPN:** `.ovpn` files now verify in OpenVPN Connect, with a Let's Encrypt
  certificate and with the panel's self-signed one. **Users download their file
  again** ([[protocols/openvpn]]).
- **Tunnel addresses:** each origin of a tunnel can choose the addresses it
  dials, in failover order, including the node's management address, and
  **Node addresses for tunnels** on the Tunnels page sets them per node. Editing
  a node's addresses resyncs its tunnels at once
  ([[use/tunnels#which-addresses-are-dialed]]).
- **Tunnel status** is drawn as a diagram of the relay, the origins and the
  addresses between them, coloured by what each side reports, and a tunnel's
  usage graph shows only its own ends ([[use/tunnels]]).
- **Verify peer cert by name** in a certificate's or an inbound's client
  settings: names Xray apps check the server's certificate against instead of
  the SNI, so a link can send another SNI and still verify an ACME certificate
  without a pin. Links carry it as `vcn`; sing-box and Clash apps have no such
  option ([[use/inbounds#client-settings]]).
- **Certificates:** dns-01 gains ArvanCloud, Selectel, Tencent Cloud (DNSPod)
  and Huawei Cloud. Names are checked before the authority is asked, and a
  wildcard over http-01 or tls-alpn-01 is refused at once. A node's ingress
  inbound answers tls-alpn-01 on its own port, so the inbound keeps serving
  while a certificate is issued
  ([[use/certificates#checked-before-the-authority-is-asked]]).
- **Hysteria2 gecko obfuscation:** **Obfs type** offers *gecko* beside
  *salamander*; it also splits the opening handshake into randomly sized,
  padded pieces. Only newer apps speak it, so add a gecko inbound beside the
  salamander one. Xray subscriptions now carry obfuscated Hysteria2 entries
  ([[protocols/hysteria2#obfuscation]]).
- **Theme catalogue:** **Settings → Subscriptions → Catalogue** installs,
  updates and removes subscription page themes from the public catalogue. In
  Docker, `./sub-themes` must be mounted writable
  ([[use/subscriptions#theme-catalogue]]).
- **Resellers:** spent traffic stays spent. Deleting an account or handing it to
  another owner no longer gives a reseller its traffic back; the upgrade keeps
  every reseller's current figure ([[use/resellers#traffic-allowance]]).
- **WireGuard:** the node's warnings say when an endpoint's range has no room
  for an account, and new WireGuard, OpenVPN and OpenConnect endpoints start at
  a /16 ([[protocols/wireguard]]).
- **The free tier** serves 50 users and 2 nodes ([[use/licence#the-free-tier]]).
- A Hysteria2 or Hysteria link to a self-signed certificate carries
  `insecure=1` beside the pin, so the official Hysteria app connects; the pin
  is still checked ([[use/certificates#pins-in-links]]).
- An error the panel did not explain now says whether no answer came or the
  answer was not the panel's own.

### v0.1.0 — 30 October 2026 {#panel-v0-1-0}

The move to the NX name. **The update button cannot install this release**: it
fails on purpose and changes nothing. Run the new install script once, as
root, on the panel's server; it moves the existing install, keeping admins,
users, nodes, templates and settings, and puts the old install back if a step
fails ([[start/move-to-nx]]).

- **Before you update:** take a backup and keep it, because older backup
  archives cannot be restored into this version. Do not go back to an older
  version over this one. Update the panel first, then the nodes.
- **The move:** the panel lives in `/opt/nx-panel` and `/var/opt/nx-panel` and
  runs as `nx-panel`; environment variables are `NX_*`; everyone is logged out
  once. Nodes and addons show *Needs migration* until you update them from the
  Nodes and Addons pages. Users download their OpenVPN profile or their
  subscription again; tunnels reconnect by themselves; the licence carries over.
- **Addresses:** **Settings → Addresses** replaces the address settings of the
  General page. The panel can listen on several ports, each with its own TLS,
  and choose per host and path what it serves: the panel, subscriptions, the
  API, a static site, a redirect, a fixed answer or another program behind it.
  A change applies at once and is put back unless confirmed within two minutes
  ([[use/ingress]]).
- **A node's own configuration is gone.** A node no longer carries outbounds,
  endpoints or routing of its own; a server that needs something different gets
  a template of its own. The upgrade makes those copies by itself, so each node
  serves what it served before ([[use/templates]]).
- **Limit configs** on a user serves the account only on the inbounds and
  endpoints that match its rules: by protocol, tag, TLS or transport
  ([[use/users]]).
- **Nodes page:** **Test outbounds** measures each outbound the node runs, and
  **Usage graph** opens the node's traffic over time.
- **Interface:** ordered lists are reordered by dragging or from the keyboard,
  hints beside each field's label, a routing rule's inbound picked from the
  template's own inbounds.

### v0.0.6 — 25 October 2026 {#panel-v0-0-6}

**Update the panel before your nodes**: node v0.0.6 refuses a configuration
with a route rule using the `direct` action. This panel works with nodes on
v0.0.3 to v0.0.6.

- **Route rules with the `direct` action are removed.** The action never did
  anything: a rule using it matched, and routing went on to the next rule. The
  panel no longer offers it and removes the stored ones on its first start, so
  routing does not change. To send traffic out directly, use the action
  **route** with the `direct` outbound ([[use/routing]]).
- Built with Go 1.27.3, which fixes a security issue in certificate
  verification; interface libraries updated.

### v0.0.5 — 17 October 2026 {#panel-v0-0-5}

Panel and nodes can be updated in either order; this panel works with nodes on
v0.0.3 and v0.0.4.

- **Resellers limited to chosen templates.** Turn on **Only chosen templates**
  on the admin and pick the **Allowed templates**: its users are put only on
  those templates and reach only their nodes. Removing a template takes the
  users off it as soon as you save, with no node restarts ([[use/resellers]]).
- **OpenVPN and OpenConnect endpoints:** the forms cover routes, DNS servers
  and ping timers pushed to clients, control channel protection (`tls_auth`,
  `tls_crypt`, `tls_crypt_v2`, with a client key per user), a list of servers
  for an OpenVPN client, and a one-time-password secret for OpenConnect
  clients ([[use/endpoints]]).
- **Cloudflare WARP preset** for outbounds: each node registers its own WARP
  device on first start, so nothing is typed and no account is shared between
  nodes ([[use/outbounds]]).
- **Nodes:** warnings no longer list items the panel has since removed;
  installing after a port change moves the node to the new port; the install
  button saves the node first.
- The Docker image runs on Alpine 3.24.

### v0.0.4 — 14 October 2026 {#panel-v0-0-4}

Panel and nodes can be updated in either order. Node v0.0.4 has an important
crash fix, so update your nodes too. A port in a subscription domain setting
(`sub.example.com:2095`) is removed on upgrade; links take the port the panel
listens on.

- **Users and owners:** every admin and operator sees every user; an operator
  owns the users it creates, and the users page opens on **My users only**. A
  main admin can hand users to another operator or reseller, one at a time or
  for a selection ([[use/users]]).
- **Nodes:** a node is marked disconnected only after two missed heartbeats in
  a row; the node install command works with the panel's self-signed
  certificate; a node with no name, a scheme or port in its address, or a port
  outside 1–65535 is refused.
- **Subscriptions:** a Persian subscription page shows dates in the Solar Hijri
  calendar and numbers in Persian digits.
- **Interface:** date fields use your language's calendar; after you change the
  panel's address, port, base path or HTTPS, the restart sends your browser to
  the new address; the help button on each page opens that page in this
  documentation; rule sets get a **From a preset** button.

### v0.0.3 — 9 October 2026 {#panel-v0-0-3}

Update the panel first, then your nodes: nodes on v0.0.3 expect this panel.
This panel still works with older nodes, but live configuration changes and
clean quota cut-offs need nodes on v0.0.3.

- **Changes reach nodes without restarts.** Editing a template, routing, DNS or
  rule sets is applied to a running node; it restarts only when it must
  ([[how/config-sync]]).
- **Rule sets come from the panel.** The panel downloads every list and sends
  it to the nodes, so one unreachable list can no longer stop a node.
- **Warnings instead of failures.** Items a node cannot run are listed on the
  node and the rest keeps working; accounts missing the credential an inbound
  needs are left out of that inbound with a warning.
- **A node you switch off stops serving**, and deleting a node stops it too.
- **Addons on the panel's own server**, without SSH, with an HTTPS certificate
  the panel issues ([[addons/directory]]).
- **Subscriptions** load and connect on current Xray-based apps and sing-box
  apps; options that only matter to client apps are no longer offered on the
  server side; VLESS Encryption on outbounds.
- **Interface:** a getting-started card on the dashboard, addresses blurred
  until you hover them, a notice when a node is behind the newest node release.

### v0.0.2 — 5 October 2026 {#panel-v0-0-2}

- **Services menu:** Telegram and email notices for admins, delivery logs,
  backup destinations (S3-compatible, SFTP, Telegram) and single sign-on
  (Google, Microsoft, GitHub, GitLab, Keycloak, Authentik, Okta, Auth0, any
  OpenID Connect or OAuth 2.0 provider). Webhooks moved under Services.
- **Addons:** browse the [addon directory](https://addons.nx-panel.org),
  install with a claim code or over SSH, suspend, watch health, approve an
  update only when it asks for more.
- **Route test:** ask a node which rule and outbound a connection would take.
- **Users and resellers:** an account an admin switches off stays off; an
  expired reseller can no longer use the panel; new routes to read, patch and
  renew one user; a `public_address` setting for links.
- **Interface:** a menu that can be pinned, a second built-in subscription page
  theme (amber).
- **Security:** the panel can no longer be framed by another site.
- **Removed:** the rule-set download mirror and its setting (nodes get rule
  sets by push, so update nodes to v0.0.2 as well), the old `/module/*` API,
  and the per-addon user and admin tables.

### v0.0.1 — 27 September 2026 {#panel-v0-0-1}

The first stable release. From this release on, the panel updates itself from
its **Update** page ([[use/panel-update]]).

## Node

### v0.1.2 — DATE {#node-v0-1-2}

**Update the panel to v0.1.2 first**, then each node from the Nodes page. A
node on v0.1.1 keeps working with it, without OpenConnect DTLS and the WARP+
licence key.

- **OpenConnect DTLS connects again.** DTLS sessions resume, a client that
  leaves without saying so is noticed after 30 seconds, an idle one is dropped
  after 70, and a failed handshake is logged as a warning naming the user
  ([[protocols/openvpn#openconnect]]).
- **WARP+ licence key**, put on the node's own WARP device without
  registering a new one; a refused key is logged with Cloudflare's reason and
  the node stays on free WARP. A WARP outbound that names an existing profile
  by id and token now enrols ([[use/outbounds#warp-plus]]).
- **Nothing in a config can start a program:** the node refuses a
  configuration with OpenConnect's wrapper paths, Tor's executable path, extra
  arguments and plugins, or Tailscale's SSH server, naming the option.
- A Tor outbound's data directory and a log file named in the core settings
  are kept in the node's `cache/` directory
  ([[reference/files-and-ports#node-files]]).
- An outbound check has its full time on the node, so a server that never
  answers reads *timed out*.
- Built with Go 1.27.4, which fixes a security issue in its HTTP library.

### v0.1.1 — 5 November 2026 {#node-v0-1-1}

**Update the panel to v0.1.1 first.** A node on v0.1.0 keeps working with it.

- **OpenConnect clients stay connected.** Each packet now goes out in one TLS
  record; before, every client built on libopenconnect dropped the session
  right after connecting. DTLS does not connect yet, so traffic runs over TLS.
- The ingress inbound answers the tls-alpn-01 challenge on the port it holds,
  so the node issues and renews a certificate while the inbound keeps serving.
- dns-01 gains ArvanCloud, Selectel, Tencent Cloud (DNSPod) and Huawei Cloud.
- Fixes a crash in the tls-alpn-01 path.

### v0.1.0 — 30 October 2026 {#node-v0-1-0}

The move to the NX name. **Update the panel to v0.1.0 first**, then update each
node from the Nodes page, where it shows *Needs migration*, or run the install
command shown there on the node ([[start/move-to-nx]]).

- The node lives in `/opt/nx-node` and `/var/opt/nx-node` and runs as `nx-node`
  (`ghcr.io/nx-panel/node` for Docker). The install script moves a node
  installed under the old name, by script or in Docker, keeping its address and
  certificates, and puts the old node back if a step fails.
- Environment variables are `NX_*` (`NX_STATE_DIR`, `NX_NODE_LISTEN`, …).
- The protocols and features are those of v0.0.6.

### v0.0.6 — 25 October 2026 {#node-v0-0-6}

**Update the panel to v0.0.6 first**: this node refuses a whole configuration
with a route rule using the `direct` action, and older panels can still send
one.

- **urltest outbounds work.** The node picks the fastest member and switches
  when one fails; before, an outbound of this type stopped the whole
  configuration from starting.
- **QUIC and UDP sessions stay up on large packets**: Hysteria, Hysteria2 and
  TUIC sessions, and UDP relayed by other protocols, no longer close when a
  packet is too large for the network path.
- Tested with current Xray-based and sing-box apps on every protocol and
  transport.
- Built with Go 1.27.3, which fixes a security issue in certificate
  verification.

### v0.0.5 — 17 October 2026 {#node-v0-0-5}

No panel update is needed. In Docker with a Tailscale endpoint, keep
`/app/cache` on host storage, as the shipped `docker-compose.yml` does.

- **MikroTik (RouterOS) clients connect to the OpenVPN server.**
- **Tailscale endpoints keep their device** across restarts and new
  containers, and a one-off auth key stays logged in after it is spent.
- **Start warnings clear themselves** as soon as the panel adds, fixes or
  removes the item.
- The Docker image runs on Alpine 3.24.

### v0.0.4 — 14 October 2026 {#node-v0-0-4}

No panel update is needed. **Do not skip this one**: a crafted QUIC packet
could crash the whole node when a routing rule sniffs traffic, such as the
torrent-blocking preset.

- More input-validation fixes: a size cap on HTTP transport requests, the gRPC
  frame length check, a Trojan multiplex memory leak, a race in multi-user
  Shadowsocks.
- Changing an outbound updates the detours, selectors, URL tests and DNS
  servers that use it.
- Removed inbounds really stop serving: connected clients are cut off.
- Removing an OpenConnect endpoint while a client is connected no longer
  freezes the node; a refused configuration leaves nothing running.
- UDP fixes for Hysteria, Hysteria 2 and TUIC.

### v0.0.3 — 9 October 2026 {#node-v0-0-3}

- Routing, DNS, logging and rule-set changes apply without dropping
  connections.
- A refused configuration leaves the previous one running; items the node
  cannot run are reported as warnings.
- Adding or removing users on OpenVPN, OpenConnect, SOCKS, HTTP, mixed and
  naive inbounds no longer touches other users' connections.
- A node serves only while its panel is reachable: switched off in the panel,
  it stops; with the panel unreachable for three minutes, it stops until the
  connection returns ([[how/node-lifecycle]]).
- VLESS, VMess and Trojan outbounds can use XHTTP and mKCP; VLESS Encryption on
  outbounds.
- Faster transfers for Shadowsocks, VMess, Trojan and gRPC.

### v0.0.2 — 4 October 2026 {#node-v0-0-2}

Rule sets arrive from the panel by push. Pair it with panel v0.0.2 or later.

### v0.0.1 — 25 September 2026 {#node-v0-0-1}

The first stable release, paired with panel v0.0.1. Nodes are installed and
updated from the panel ([[start/add-node]]).

## Addons and tools

| Release | Date | Notes |
|---|---|---|
| NX Shop v0.1.3 | DATE | Package and security updates |
| NX Shop v0.1.2 | 7 November 2026 | Refuses writes sent from another origin; asks for trusted proxies and a listen address at install ([[addons/shop#install]]) |
| NX Shop v0.1.1 | 30 October 2026 | Install from the panel's **Addons** page or with `ghcr.io/nx-panel/shop:0.1.1` ([[addons/shop]]) |
| NX Shop v0.1.0 | 30 October 2026 | The move to the NX name: update it on its host from the **Addons** page, where it shows *Needs migration* |
| NX Shop v0.0.3 | 24 October 2026 | Package and security updates |
| NX Shop v0.0.2 | 9 October 2026 | |
| NX Shop v0.0.1 | 4 October 2026 | First release |
| Notifier v0.4.3 | DATE | Package and security updates |
| Notifier v0.4.2 | 7 November 2026 | Refuses writes sent from another origin; asks for trusted proxies and a listen address at install ([[addons/notifier#install]]) |
| Notifier v0.4.1 | 30 October 2026 | `ghcr.io/nx-panel/notif:0.4.1` ([[addons/notifier]]) |
| Notifier v0.4.0 | 30 October 2026 | The move to the NX name: update it on its host from the **Addons** page, where it shows *Needs migration* |
| Notifier v0.3.1 | 24 October 2026 | Package and security updates |
| Notifier v0.3.0 | 9 October 2026 | |
| Notifier v0.2.0 | 4 October 2026 | |
| Notifier v0.1.0 | 4 October 2026 | First release |
| nx-migrate v0.1.1 | 5 November 2026 | Works with panel 0.1.1. Users keep the inbounds they had (all panels except Hiddify, Marzneshin and Remnawave); PasarGuard's WireGuard comes across; Hysteria2 obfuscation, 3x-ui weekly renewals and Hiddify's reset cycle are carried; nodes are switched off during a transfer by default only on a panel with no users ([[start/migrate]]) |
| nx-migrate v0.1.0 | 30 October 2026 | The move to the NX name; package and security updates ([[start/migrate]]) |
| nx-migrate v0.0.2 | 9 October 2026 | Windows, Linux and macOS builds |
| nx-migrate v0.0.1 | 23 September 2026 | First release |
