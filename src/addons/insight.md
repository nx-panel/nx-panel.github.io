---
title: "NX Insight"
description: "See which of your nodes and protocols still reach which of your users' networks, get warned when a path is blocked or probed, and put the links that work first in every subscription, with the NX Insight addon."
order: 10
tags: [addons, node, subscriptions, monitoring]
---

# NX Insight

NX Insight is the official [[~addon]] that shows which paths from your users
to your [[~node|nodes]] still work. A path is one client network (the ISP, by
its ASN), one node and one [[~inbound]]. Insight shows how each path fares,
warns you when one is blocked or a censor starts probing a node, and puts the
links that work for each network first in its users' subscriptions. This
page covers what it needs, installing it, each of its pages, and what it
sends where.

::: warning Estimates, not proof
Everything Insight shows is worked out from counts: a network that has gone
quiet can look blocked, a probe score can rise on a scan, and an AI finding
can simply be wrong. Check a finding against the numbers beside it, and with
a real user on that network if you can, before you move users or rotate an
address.
:::

## What it needs

- A panel from **v0.2.1** and nodes from **v0.2.0** on; nodes on v0.2.1
  count OpenConnect, OpenVPN and WireGuard more exactly. The nodes count; the
  panel relays. A node on an older version keeps serving and simply reports nothing, and
  Insight shows that node as not supported.
- The counting is off on every node until Insight reads it. The first read
  turns it on; it turns off again ten minutes after the last read, or within
  a minute of Insight's token being deleted or losing its scope. Nothing is counted for an
  addon that is not there.

## What stays on your server

A node never reports a client's address, only its /24 (IPv4) or /48 (IPv6)
network, or /40 for a 6to4 address. The panel passes those numbers on without storing them. Insight
turns each one into its ISP and region the moment it reads it and keeps only
the ISP, the country and the region. No client address or network prefix is
written anywhere, by the panel or by Insight.

Install Insight on the panel's own server and the networks never cross the
internet at all.

## Install

On the panel: **Addons → Browse → NX Insight → Install**. The panel asks its
questions and installs it on its own server or another one over SSH, or gives
you a command to run. Every way to install is in [[directory]].

| Question | What it sets |
| --- | --- |
| Port | `8099` by default. With HTTPS on, Insight's only port. |
| Database | SQLite, or PostgreSQL with its connection string. |
| The first admin | A name and a password of 10 characters to 72 bytes. |
| Admin path | Everything Insight serves is under it. Drawn at random; empty puts it at the root. |
| Public address | Where you open Insight: scheme, host and port. |
| HTTPS | `panel`, `acme`, `acme-http`, `self-signed` or `off`, as for every addon ([[directory#https-for-the-addon]]). |
| Trusted reverse proxies | Only behind a reverse proxy: its addresses or ranges. Empty trusts no `X-Forwarded-For`. |
| Listen address | Empty: every interface. `127.0.0.1` only behind a reverse proxy on the same server. |
| Countries to tell regions apart in | `IR,RU,CN` by default. Clients there get their province or oblast as well; clients elsewhere get their network and country. |
| Telegram bot, chat, proxy, Bot API address | Where alerts go. Empty: alerts are only kept in Insight. A Bot API address of `https://tapi.bale.ai` sends them through a Bale bot. |
| Language of the alerts | English, Persian, Russian or Chinese. |

By hand, on a Linux server as root:

```bash
sh install.sh --method docker --opt port=8099 --opt admin_password=… \
  --panel-url https://panel.example.com --claim-code …
```

`--method script` runs the binary under systemd instead of Docker. Running
the same command again updates in place; `--uninstall` removes it, and
`--purge` deletes its data too.

When you approve Insight, it asks for three things: reading what the nodes
count (which turns the counting on), ordering the links of every
subscription, and reading the nodes' addresses, to show where each node is
hosted.

### Sign-in

Sign in at the public address with the admin path. Insight has its own admins,
not the panel's. Turn on two-factor sign-in under **Settings → Account**. A lost
password is reset on the server:

```bash
insight admin reset-password -user admin -pass …
```

### Settings {#settings}

**Settings** holds what can change while Insight runs, a tab each:

| Tab | |
| --- | --- |
| Alerts | The Telegram or Bale bot, its chat, a proxy, the Bot API address and the alerts' language, with a test message. The installer's answers hold until you save here; after that yours do, until **Go back to the installer's values**. A new Bot API address needs the bot's token typed again, so a saved token is never sent somewhere new. |
| AI model | Mode A, below. |
| Prometheus | The token `/metrics` asks for; none turns it off. Taken at once, without a restart. |
| Account | Your password and two-factor sign-in. |
| Installation | What the installer set — port, admin path, address, HTTPS, database, the countries told apart by region — to read only. Changing one means running the installer again. |

## Overview

The top of the page says whether Insight is reading the panel:

| State | What it means |
| --- | --- |
| Waiting | Not registered with a panel yet. |
| Reading | Working; the time of the last read is beside it. |
| Not supported | The panel is older than v0.2.0. Update it, then the nodes. |
| No permission | Insight's token lacks the scope to read. Approve its permissions again under **Addons**. |
| Failed | The last read failed; Insight tries again every minute. |

Below it:

- **Networks cut off.** When every path to one ISP fails on two or more nodes
  at once, it is not your nodes that are blocked but that network that is
  cut off: a shutdown, or a whitelist that lets only some sites through.
  Insight says so once and suggests nothing to change.
- **Tunnels with no link.** A user connects to a tunnel's relay node and
  goes out through its origin; when the link between them is cut, every
  network on the relay seems to fail at once. Insight asks the panel about
  each tunnel every minute, and while one has no link it says that once —
  no alert for each of the relay's paths, and nothing suggested to change on
  the relay.
- **Blocked paths**, each with the paths that still work for the same network.
- **The matrix**: client networks (or regions) down the side, nodes and
  inbounds across, over the last hour, six hours, day or week. Each cell is
  the share of that network's connections that got through. Point at it for
  the numbers: connections, authenticated, failed handshakes, sessions cut
  within seconds, round-trip time.
  A WireGuard endpoint's cell shows the handshakes that worked instead:
  WireGuard answers no handshake that fails, so a failure cannot be seen.
  Each country whose regions you tell apart gets a table of its own, and
  every other country goes in one more, **abroad** (by region, a row per
  country there). A network is shown by its short name; the ⓘ beside it
  gives its AS number and full registered name, on hover or a tap.

## Nodes

The network each node is hosted on, from its address. Where the network
database places a node wrong (a provider's ranges, an address behind a
front), correct it here; the correction stays over later updates of the
database.

Beside it: what the node serves — each inbound and endpoint, with its
protocol, transport, security and whether a domain front reaches it — and
the [[use/tunnels|tunnels]] it is the relay or an origin of: linked or not,
and over the last day the link attempts, links up, failures and links that
dropped right after they came up. Reading the tunnels needs the
`tunnels:read` permission, which Insight asks for when it is installed.

## Probing

Before a censor blocks a server, it often tests it: it sends again a
handshake it saw a real client send, or tries the port right after a real
user connected. The nodes count these signs; nothing on a node reacts to
them, so a prober learns nothing from the counting.

Each node and inbound gets a score from 0 to 100 over the last six hours. A
replayed handshake adds 20, up to 80; a real client never sends the same
handshake twice. A surge of failed handshakes right after real users' sessions
adds 20. At 40 an alert goes out, with what to do: change the inbound's port
or SNI (with REALITY, its target site too) or give the node another address,
before the block lands. The **Blocked after?** column says whether a path on
that inbound degraded within 48 hours of the alert.

The counts of unknown SNI on an [[protocols/ingress|ingress]] and of stray QUIC
packets on a Hysteria2 port range are shown beside the score, not added to it:
they are scanners rather than probes of this endpoint.

## Alerts

Every change, newest first: a path degraded or recovered, a network cut off
or back, an endpoint probed, a tunnel down or back. A path is judged against its own normal for that
hour of the week, learned from its history; a path needs a few hours of
traffic before it can be judged, and two bad checks five minutes apart before
it alerts. With a Telegram or Bale bot set at install or under **Settings →
Alerts**, each alert is also sent there, in the language chosen there.

## Subscriptions, best link first

For every ISP seen on two or more nodes, Insight tells the panel which nodes
and inbounds work best for it, and the panel puts those links first in the
[[~subscription|subscriptions]] fetched from that network. Nothing is removed:
a user still gets every link, in a better order. The order is updated as
paths change and expires after six hours without an update, so removing
Insight returns every subscription to the panel's own order. During a network
cut-off the order for that network is left as it was.

## AI analysis

Optional: your own AI model reads your paths once a day for each country
whose regions you chose to tell apart, and again after an alert there, and says what it sees: a protocol
blocked for one ISP, a node's address listed, where to move users, what to
change. Set it up under **Settings → AI model**; the **AI analysis** page then
shows the findings, per country, on its **Insight AI** tab. Until a model is set
up, that tab only says so.

| Field | |
| --- | --- |
| API address | Any OpenAI-compatible API: `https://api.openai.com/v1`, OpenRouter, or Ollama on your own server at `http://127.0.0.1:11434/v1`. |
| Model | A small local model reads poorly; prefer one of 7B or more, or a hosted model. |
| API key | Kept in Insight's database and never shown again. A local model needs none. Changing the API address’s scheme, host or port, or the proxy, clears it: enter it again. |
| Proxy | For a server that cannot reach the provider directly. |
| Daily token budget | Past it, nothing is sent until the next day (UTC). |

What is sent: for each ISP (its AS number and registered name), node number
and inbound tag, the counts and ratios of the last 24 hours; each node's
hosting country and provider; the probing scores and how many alerts there
were; each tunnel touching those nodes, by its name, with its relay and
origin node numbers and whether it has a link. Never a client's or a node's
address, never a node's name. Inbound tags,
tunnel names and a provider you typed on the Nodes page go as you wrote them, cut to
up to forty letters, digits, spaces, dashes and underscores (dots are dropped), so do not put an address in them. Nothing of this goes to NX (Mode B, below, is separate). Every
finding is checked against the numbers it was given, and one they do not bear
out is left out, so a model that guesses wrong shows nothing rather than
something false.

## NX network (Mode B) {#nx-network}

Off until you turn it on on the **NX network** page. Mode B shares this
panel's counts with NX, anonymously, and brings back NX's regional report:
which protocol shapes work, are degraded or are blocked for each network of
your users' countries, pooled from many independent operators.

What is sent, once an hour, for the hour just closed:

- per client network: its AS number and registered name, its country and
  estimated region;
- per hosting network of your nodes: its AS number, name and country;
- per protocol shape: protocol, transport, TLS or REALITY, direct or through
  a CDN, and TCP or UDP;
- connections accepted, authenticated and cut early, failures by class and
  by signature (bytes read, seconds to failure), replayed handshakes, and
  the sums of round-trip and session times;
- Insight's own reading of each: fine, degraded, network outage, the node's
  address blocked, its tunnel down.

Never a client's or a node's address, a node's name or number, an inbound's
tag, a domain, an SNI, a path or your panel's address. The page shows every
batch as it went out.

Each batch is signed with a key Insight makes for this install. Turning
Mode B on registers the key with your panel, and the panel passes it to
NX's licence service (see [[use/licence#contributor-key]]); that is how NX
tells a licensed panel from a free one, and it learns nothing else about
your licence. Turning it off removes the key from the panel and drops
every batch not sent yet.

NX publishes a report only for a network that at least five independent
operators share, so nothing in it can be traced to one panel. A panel with
a licence receives the whole report; a panel without one receives outages
and throttling only, not which protocols work. Reports go to installs that
sent a batch in the last three days. Insight asks for a newer report when
it starts and when you open the page, sending the time of the report it
holds; NX sends a newer one or says there is none. The report is on the
**AI analysis** page, on its **NX** tab. It is signed by
NX, and Insight shows none whose signature does not check. Like every
finding here, it is an estimate: check it against your own numbers.

## Prometheus and Grafana

With a token given to the **Token for Prometheus** question at install, or
set under **Settings → Prometheus**, Insight serves `/metrics` under its admin path for your own Prometheus, and
the panel's repository ships a Grafana dashboard for it: success, sessions,
failures and short cuts per node and client network, the paths degraded now,
tunnels with no link and probe scores. Setting it up is in
[[how/monitoring#nx-insight-paths]]. Like everything here, it carries no
client address.

## Data

IP address to ASN data from [iptoasn.com](https://iptoasn.com). IP
geolocation by [DB-IP](https://db-ip.com), licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Each install
downloads both itself: the networks daily, the regions monthly.

Insight keeps hourly numbers for seven days and daily ones for 90 days.
