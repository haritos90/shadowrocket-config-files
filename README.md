# shadowrocket-config-files

Personal Shadowrocket configuration files for iOS and macOS.

## Profiles

| File | Description | Strategy |
|---|---|---|
| [h90_main.conf](h90_main.conf) | Main profile. Routes selected traffic through a proxy; everything else is direct. Encrypted DNS only (Cloudflare + Google DoT/DoH). | Allowlist → PROXY, FINAL → DIRECT |

To install, copy the raw file URL and add it in Shadowrocket under **Config → +**.

Raw URL:
```
https://raw.githubusercontent.com/haritos90/shadowrocket-config-files/master/h90_main.conf
```

## Legal Notice

This software is intended for development, testing, and research purposes only.

The author does not provide any guarantees regarding:

- availability of network access
- compatibility with specific services
- compliance with any external restrictions

Users are solely responsible for how they use this software and must comply with applicable laws.

**Usage Restrictions**

This software is not intended to be used for bypassing access restrictions or violating applicable laws. The author does not support or encourage such use.

---

## How it works

The profile uses an **allowlist** approach: only traffic explicitly matched by a rule-set is proxied. Everything else goes direct.

```
Matched rule-sets  →  PROXY
Everything else    →  DIRECT
```

This keeps the proxy load minimal and avoids routing unnecessary traffic through a Server. Rule-sets are external `.list` files fetched from GitHub; Shadowrocket can auto-update them on a schedule.

DNS is handled entirely via encrypted resolvers (Cloudflare and Google, DoT + DoH). All DNS traffic on port 53 is hijacked to enforce this.

## Rule-sets

Rule-sets are maintained in a separate repository: **[haritos90/allow-domains](https://github.com/haritos90/allow-domains)**. That repo contains the full list of available domain and subnet lists, how they are built, and what each covers.

The table below shows the current configuration of `h90_main.conf` as an example. `no-resolve` on subnet rules means domain requests skip IP-based matching — only already-resolved IPs are checked.

### Current configuration

| Rule-set | Raw URL | Policy |
|---|---|---|
| All domains (combined) | [russia-all-surge.list](https://raw.githubusercontent.com/haritos90/allow-domains/main/Russia/russia-all-surge.list) | PROXY |
| IPv4 subnets (combined) | [Subnets/IPv4/all-surge.list](https://raw.githubusercontent.com/haritos90/allow-domains/main/Subnets/IPv4/all-surge.list) | PROXY, no-resolve |
| IPv6 subnets (combined) | [Subnets/IPv6/all-surge.list](https://raw.githubusercontent.com/haritos90/allow-domains/main/Subnets/IPv6/all-surge.list) | PROXY, no-resolve |

### Customising rule-sets

The [haritos90/allow-domains](https://github.com/haritos90/allow-domains) repository provides both combined and per-category lists. Two common approaches:

- **Single Server** — use the combined lists (`all-surge.list`) and route everything matched to one PROXY policy.
- **Split by category** — use per-category lists and assign each to a different Server or Proxy Group. To avoid overlap between the combined list and individual categories, use the `minus-*` files from that repo, which subtract a given category from the combined set.

See the [allow-domains README](https://github.com/haritos90/allow-domains#readme) for the full list of available files and instructions.

### Managing rules via Shadowrocket UI

Rules can be added, removed, or reordered without editing the config file in plain text mode.

**To add a rule-set:**

1. **Config** tab → tap **ⓘ** next to the active config → **Edit**.
2. Tap **Rules** → tap **+** (top right).
3. Set **Type** to `RULE-SET`.
4. Paste the raw URL of the `.list` file into the **URL** field.
5. Set **Policy** to `PROXY` or `DIRECT` as needed.
6. Tap **Save**. Drag to reorder if necessary — rules are matched top to bottom.

**To remove a rule-set:**

In the same Rules list, swipe left on the rule and tap **Delete**.

**To change the policy on an existing rule:**

Tap the rule to open it, change the **Policy** field, then save.

> Rule-sets added via the UI are saved into the config file. After making changes, tap **Done** to apply them.

## DNS

Primary DNS resolvers (parallel querying, first response wins):

| Resolver | Protocol | Notes |
|---|---|---|
| `2606:4700:4700::1112` / `2606:4700:4700::1002` | DoT :853, plain | Cloudflare for Families (malware blocking), IPv6 |
| `1.1.1.2` / `1.0.0.2` | DoT :853, plain | Cloudflare for Families, IPv4 |
| `security.cloudflare-dns.com` | DoH | Cloudflare for Families |
| `2001:4860:4860::8888` / `2001:4860:4860::8844` | DoT :853, plain | Google, IPv6 |
| `8.8.8.8` / `8.8.4.4` | DoT :853, plain | Google, IPv4 |
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
| `ipv6` | true | IPv6 enabled; IPv4 takes priority |
| `prefer-ipv6` | false | |
| `hijack-dns` | :53 | Intercepts all DNS on port 53 |
| `private-ip-answer` | true | Allows private IP responses from DNS |
| `dns-direct-fallback-proxy` | true | Uses proxy if direct DNS resolution fails |
| `udp-policy-not-supported-behaviour` | REJECT | UDP dropped if the matched Server doesn't support forwarding |
| `icmp-auto-reply` | false | Ping auto-reply disabled |

## Installation

1. Copy the raw URL of the config file (see [Profiles](#profiles) above).
2. In Shadowrocket: **Config** tab → **+** (top right) → paste the URL → **Download**.
3. Tap the downloaded config to set it as active (checkmark appears).

**After installation:**

- **Home > Global Routing** → set to **Config**.
- **Settings > UDP > Enable Forwarding** → turn on.
- Add your proxy Server(s) and connect.

**Auto-update:**

Enable in **Settings > Update > Config** (e.g. every 24 hours). Requires **iOS Settings > General > Background App Refresh > Shadowrocket** to be on.

