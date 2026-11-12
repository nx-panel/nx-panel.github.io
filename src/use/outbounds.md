---
title: "Outbounds"
description: "Choose where traffic goes after it reaches a node: straight out, blocked, or on through another proxy, and check that a way out works."
order: 11
group: core
tags: [routing]
---

# Outbounds

An [[~outbound]] is a way out for traffic that has reached a node: straight to
the internet, blocked, or on to another proxy server. [[routing|Routing]] picks
the outbound for each connection. Outbounds live in a shared pool under
**Core configs → Outbounds** and reach nodes through [[templates]].

<NxShot page="outbounds" alt="The Outbounds page: a summary strip counting outbounds in templates and unused, above a list of tags, types and servers" />

## When you need one

Every node has a built-in outbound named `direct`, and with no routing rules
everything goes out through it, straight to the destination. That is what
most fleets want. You add outbounds when traffic should go somewhere else:

- **Block** something: ads, torrents, a list of destinations.
- **Chain** through another server: send some or all traffic on to an
  upstream proxy, so the internet sees that server's address.
- **Choose between** several upstreams, by hand or by measured delay.

## Types

| Type | What it does |
| --- | --- |
| `direct` | Connects straight to the destination from the node. |
| `block` | Refuses the connection. |
| `socks`, `http` | Forwards to a SOCKS or HTTP proxy. |
| `shadowsocks`, `vmess`, `vless`, `trojan` | Forwards to another proxy server speaking that protocol, with its own transport and TLS. |
| `hysteria`, `hysteria2`, `tuic` | Forwards over QUIC to a server speaking that protocol. |
| `shadowtls`, `anytls`, `ssh`, `naive`, `mieru`, `snell`, `sudoku`, `trusttunnel`, `tor` | Further upstream types, for operators who already run such a server. |
| `masque` | Cloudflare WARP: traffic leaves through Cloudflare's network instead of the node's own address. See [WARP](#warp). |
| `selector` | A group of other outbounds; one of them is used, chosen by you. |
| `urltest` | A group of other outbounds; the one with the lowest measured delay is used. |

For a proxy upstream, fill in the server address, port and credentials on
**Basic** and **Protocol**, and its transport and TLS on their tabs. The TLS
tab offers **None**, **TLS** or **REALITY**, as the far server expects.

A `selector` or `urltest` lists the tags of the outbounds it chooses from.
Those outbounds must be in the same template.

A `tor` outbound never starts a program on the node. An executable path,
extra arguments or transport plugins in its JSON are refused when it is
saved, with the option named, and an outbound stored with one before has it
left out on its way to the node. Its data directory is kept in the node's
cache directory, whatever path is typed.

## Adding an outbound

1. Open **Core configs → Outbounds** and click **Add**.
2. Pick the **Type** and give it a **Tag**. The tag is how routing names the
   outbound, and it is fixed after creation.
3. Fill in the tabs the type shows.
4. Click **Save**.
5. Select it in a template's **Outbounds**, and name it in a routing rule or
   as the **Final outbound**.

An outbound that no template selects does nothing. The summary strip counts
those under **Unused**.

## Importing outbounds {#import}

The **⋯** button (**Operations**) beside **Add** opens the import menu.
**From links** turns share links into outbounds: paste them one per line, or
as one base64 block.

1. Paste the links and click **Convert**.
2. Every row is shown before anything is saved: its tag, its type, its
   **Name in the source** and any **Notes**. Change a tag where you want
   another; each row needs one that no other row and no existing outbound
   has. Untick a row to leave it out.
3. Click **Add** *n*. Each row is created the ordinary way, through the same
   save as **Add**, so every check a save applies still applies. A row the
   save refuses stays in the list with the reason, to be fixed and sent
   again.

A line that does not convert is listed under **Not converted** by its line
number, never by the link, which carries credentials. Naive links are refused,
because some node builds cannot run naive. Links make outbounds only: a
WireGuard server comes in as its `.conf` on the endpoints page
([[endpoints#import]]). The links are not kept. The new outbounds are on no
template yet: add them to a template and write the routing rules that use
them.

## Cloudflare WARP {#warp}

A WARP outbound sends traffic out through Cloudflare's network, so the
destination sees a Cloudflare address instead of the node's. It is useful for
sites that refuse your server's address.

1. Open **Core configs → Outbounds** and click **From a preset**.
2. Pick **Cloudflare WARP**. The ordinary form opens with the type `masque`
   and the tag `warp` filled in. Leave the profile fields empty; a WARP+ key
   is the one exception ([WARP+](#warp-plus)).
3. Click **Save**, select it in a template's **Outbounds**, and write the
   routing rules that send traffic to `warp` yourself. The preset adds no
   rules.

There is no account to create. On its first start each node registers its own
WARP device with Cloudflare and keeps it, so no two nodes share one. That first
registration needs the node to reach Cloudflare's API. Where it cannot, set
**Profile detour** to an outbound that can reach it.

A profile filled in by hand (its id and token) names one WARP device, which a
node enrols on. On a template that two or more enabled nodes run, they would
all share that device, and the device keeps only the last node's key: one
node holds the tunnel and the others stop connecting. The panel saves such a
template but warns of it: its row on the Templates page shows **Check**, and
its editor names the outbound ([[templates#checked-on-save]]). Leave the
profile empty and use a WARP+ key instead, or put each node on a template of
its own. The WARP key fields have no generate button: a made-up value is never
right there.

### WARP+ {#warp-plus}

A WARP+ key moves the nodes onto Cloudflare's paid WARP. Paste it in **WARP+
licence key** in the outbound's form and save. Each node puts the key on the
WARP device it already has; no new device is registered.

- Every node is a device of its own, and a key covers only a few devices
  (five at the time of writing). Past that, Cloudflare refuses the key and the
  node stays on free WARP; the node's log says why.
- Clearing the field does not take the nodes back to free WARP. **Recreate
  profile** does, with a new device on each node.
- A node from before this field refuses an outbound carrying it. Update the
  nodes before you set a key ([[start/update]]).

## How routing names an outbound

Routing refers to an outbound by its tag, in four places:

- a routing rule's **Outbound**;
- the template's **Final outbound**, which takes every connection no rule
  matched;
- another outbound that dials through it (a detour);
- a `selector` or `urltest` that lists it.

The **Block torrents** routing preset adds a `block` outbound of its own,
named `block-torrent`, which the node counts. See [[routing#presets]].

## Checking an outbound

**Route test**, in the menu of a node's row or a user's row, walks the node's
routing rules for a connection you describe and names the outbound that takes
it. **Test and connect** also opens one real connection from the node down
that path and reports whether the destination was reached and how long it
took. Through a proxy upstream the answer can be "not refused, unconfirmed",
because a proxy does not always say whether its far end connected. See
[[nodes]].

**Test outbounds**, in the menu of a node's row, lists every outbound and
endpoint the node is running, read from the node itself, so the built-in
`direct` and any tunnel are there too. **Test all**, or the gauge on one row,
has the node fetch a test URL through it and shows how many milliseconds the
answer took, or the node's error; a server that never answers reads *timed
out*. Leave **Test URL** empty for the node's
default. It needs the right to change nodes, because it makes the node open
connections.

## An outbound for one node only {#per-node-outbounds}

An outbound reaches the nodes whose template selects it. When one server needs
an upstream the others do not, such as a different exit for one region, give
that node a template of its own that selects it. See
[[templates#a-template-of-its-own]].

## Editing and deleting

Editing an outbound sends the change to every node that carries it. The
**Duplicate** action copies one under a new tag.

The panel refuses to delete an outbound that something still names by tag in
a template that selects it: a routing rule, the template's final outbound, a
detour, or a group. It lists those references. Repoint them, then delete. An
outbound tagged `direct` always deletes, because every node has a built-in
`direct` of its own. A node also refuses to drop an
outbound its running routing still needs, so a change that removes an
outbound and the rule naming it should be saved together.

## Related

- [[routing]]: the rules that pick an outbound.
- [[templates]]: selecting outbounds for nodes.
- [[endpoints]]: interfaces that act as a way out too.
- [[use/tunnels]]: links between your own nodes, which routing can also send
  traffic to.
