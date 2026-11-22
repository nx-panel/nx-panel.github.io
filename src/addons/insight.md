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

- A panel and nodes from **v0.1.3** on. The nodes count; the panel relays.
  A node on an older version keeps serving and simply reports nothing, and
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
not the panel's. Turn on two-factor sign-in under **Security**. A lost
password is reset on the server:

```bash
insight admin reset-password -user admin -pass …
```

## Overview

The top of the page says whether Insight is reading the panel:

| State | What it means |
| --- | --- |
| Waiting | Not registered with a panel yet. |
| Reading | Working; the time of the last read is beside it. |
| Not supported | The panel is older than v0.1.3. Update it, then the nodes. |
| No permission | Insight's token lacks the scope to read. Approve its permissions again under **Addons**. |
| Failed | The last read failed; Insight tries again every minute. |

Below it:

- **Networks cut off.** When every path to one ISP fails on two or more nodes
  at once, it is not your nodes that are blocked but that network that is
  cut off: a shutdown, or a whitelist that lets only some sites through.
  Insight says so once and suggests nothing to change.
- **Blocked paths**, each with the paths that still work for the same network.
- **The matrix**: client networks (or regions) down the side, nodes and
  inbounds across, over the last hour, six hours, day or week. Each cell is
  the share of that network's connections that got through. Point at it for
  the numbers: connections, authenticated, failed handshakes, sessions cut
  within seconds, round-trip time.

## Nodes

The network each node is hosted on, from its address. Where the network
database places a node wrong (a provider's ranges, an address behind a
front), correct it here; the correction stays over later updates of the
database.

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
or back, an endpoint probed. A path is judged against its own normal for that
hour of the week, learned from its history; a path needs a few hours of
traffic before it can be judged, and two bad checks five minutes apart before
it alerts. With a Telegram or Bale bot set at install, each alert is also sent
there, in the install's language.

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
change. Set it up on the **AI analysis** page:

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
were. Never a client's or a node's address, never a node's name. Inbound tags
and a provider you typed on the Nodes page go as you wrote them, cut to
up to forty letters, digits, spaces, dashes and underscores (dots are dropped), so do not put an address in them. Nothing goes to NX. Every
finding is checked against the numbers it was given, and one they do not bear
out is left out, so a model that guesses wrong shows nothing rather than
something false.

## Data

IP address to ASN data from [iptoasn.com](https://iptoasn.com). IP
geolocation by [DB-IP](https://db-ip.com), licensed under
[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Each install
downloads both itself: the networks daily, the regions monthly.

Insight keeps hourly numbers for seven days and daily ones for 90 days.
