# Shadowrocket Config — h90_main

Personal Shadowrocket configuration for routing Russian internet traffic through a proxy while keeping everything else direct.

## How it works

The config uses an **allowlist** approach: only traffic matching the rule-sets below is proxied. All other traffic goes direct. This is the inverse of the typical "block/proxy everything except known domestic sites" approach — it is minimal, fast, and easy to maintain.

```
Russian domains & IPs  →  PROXY
Specific subnets       →  PROXY
Everything else        →  DIRECT
```

DNS is handled entirely via encrypted resolvers (Cloudflare and Google, DoT + DoH). There is no plain DNS fallback except Yandex Safe DNS as a last resort. All DNS on port 53 is hijacked to enforce this.

## Rule-sets

Rule-sets are maintained in a separate repository: [haritos90/allow-domains](https://github.com/haritos90/allow-domains)

| Rule-set | Policy | Description |
|---|---|---|
| `Russia/russia-all-surge.list` | PROXY | Russian domains that require a proxy (media, services, etc.) |
| `Subnets/IPv4/all-surge.list` | PROXY, no-resolve | IPv4 subnets that require a proxy |
| `Subnets/IPv6/all-surge.list` | PROXY, no-resolve | IPv6 subnets that require a proxy |
| _(fallback)_ | DIRECT | All other traffic |

`no-resolve` on the subnet rules means domain requests skip those rules — only already-resolved IPs are matched against them, avoiding unnecessary local DNS lookups.

## DNS

Primary DNS resolvers (parallel querying, first response wins):

| Resolver | Protocol | Notes |
|---|---|---|
| `2606:4700:4700::1112`, `2606:4700:4700::1002` | DoT :853, plain | Cloudflare for Families (malware blocking), IPv6 |
| `1.1.1.2`, `1.0.0.2` | DoT :853, plain | Cloudflare for Families, IPv4 |
| `security.cloudflare-dns.com` | DoH | Cloudflare for Families |
| `2001:4860:4860::8888`, `2001:4860:4860::8844` | DoT :853, plain | Google, IPv6 |
| `8.8.8.8`, `8.8.4.4` | DoT :853, plain | Google, IPv4 |
| `dns.google` | DoH | Google |

Fallback DNS (used if primary fails or times out after 2 s):

| Resolver | Protocol |
|---|---|
| `77.88.8.88` | DoT :853, plain |
| `safe.dot.dns.yandex.net` | DoH |
| `2a02:6b8::feed:bad` | plain, IPv6 |
| system DNS | last resort |

## Key settings

| Parameter | Value | Notes |
|---|---|---|
| `ipv6` | true | IPv6 enabled, IPv4 takes priority |
| `prefer-ipv6` | false | |
| `hijack-dns` | :53 | Intercepts all DNS on port 53 |
| `private-ip-answer` | true | Allows private IP responses from DNS |
| `dns-direct-fallback-proxy` | true | Uses proxy if direct DNS resolution fails |
| `udp-policy-not-supported-behaviour` | REJECT | UDP is dropped if the matched Server doesn't support it |
| `icmp-auto-reply` | false | Ping auto-reply disabled |

## Installation

### Option A — add via URL (auto-updates)

1. Copy the raw URL of this file:
   ```
   https://raw.githubusercontent.com/haritos90/shadowrocket-config-files/main/h90_main.conf
   ```
2. In Shadowrocket: **Config** tab → top right **+** → paste the URL → **Download**.
3. Tap the downloaded config to set it as active (checkmark).

### Option B — manual

1. Download `h90_main.conf` and transfer it to your device (AirDrop, Files app, iCloud Drive, etc.).
2. In Shadowrocket: **Config** tab → import the file → set as active.

### After installation

- **Home > Global Routing** → set to **Config**.
- **Settings > UDP > Enable Forwarding** → turn on.
- Add your proxy Server(s) and connect.

## Auto-update

To keep rule-sets current, enable auto-update in Shadowrocket:

**Settings > Update > Config** → set an interval (e.g. every 24 hours).

Background refresh must be enabled in **iOS Settings > General > Background App Refresh > Shadowrocket**.
