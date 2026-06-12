# Shadowrocket Config files

My version of Shadowrocket Configuration files for iOS and macOS that can be configured for many scenarios. Inlcudes detailed descriptions of all known parameters of Shadowrocket Configuration files.

## Profiles

| File | Description | Strategy |
|---|---|---|
| [h90_main.conf](h90_main.conf) | Main profile. Routes required domains and subnets based on rules through a proxy; everything else is direct. Has encrypted DNS enabled (Cloudflare + Google DoT/DoH). | Allowlist → PROXY, FINAL → DIRECT |
| [h90_direct.conf](h90_direct.conf) | Direct profile for trusted networks (e.g. home WiFi where the router already handles routing). All traffic goes direct, system DNS. Designed to be used as the config for a Scene on trusted networks. | No rules, FINAL → DIRECT |

### Raw URLs (paste to Shadowrocket Config to download)

**h90_main.conf:**
```
https://raw.githubusercontent.com/haritos90/shadowrocket-config-files/master/h90_main.conf
```

Use it for *Global Routing = Config* or Scene where you need to proxy.
 
**h90_direct.conf:**
```
https://raw.githubusercontent.com/haritos90/shadowrocket-config-files/master/h90_direct.conf
```

Use it for Scene where you want direct connection.

## Installation

1. Copy the raw URL(s) above.
2. In Shadowrocket: **Config** tab → **+** (top right) → paste the URL → **Download**.
3. Tap the downloaded config to set it as active (checkmark appears).

**After installation:**

- **Home > Global Routing** → set to **Config** for a single profile, or **Scene** when using multiple profiles (see [Scene](#scene) below).
- **Settings > UDP > Enable Forwarding** → turn on.
- Add your proxy Server(s) and connect.

**Auto-update:** Enable in **Settings > Update > Config** (e.g. every 24 hours). Requires **iOS Settings > General > Background App Refresh > Shadowrocket** to be on.

## Scene

Scene mode makes Shadowrocket automatically switch routing based on network conditions. This is useful when you want proxy routing on untrusted networks but want to go fully direct on trusted ones (e.g. home WiFi where the router already handles routing at the network level).

**How the two profiles work together:**

| Scene condition | Config file | Effect |
|---|---|---|
| Home WiFi (trusted, router handles routing) | `h90_direct.conf` | All traffic direct, system DNS |
| All other networks (cellular, unknown WiFi) | `h90_main.conf` | Selective proxy via allowlist rules, encrypted DNS |

A separate config file is needed for the trusted-network Scene because Shadowrocket reads config settings (DNS, skip-proxy, TUN routes, etc.) even when routing is set to Direct. The settings in `h90_main.conf` — encrypted DNS, IPv6 enabled — are not appropriate for a home network where the router handles everything.

**Setup:**

1. Install both config files (see [Installation](#installation) above).
2. **Home > Global Routing** → tap **Scene** → **+** to create a new Scene.
3. **Scene for home WiFi:** set Condition to Wi-Fi SSID → enter your home network name. Set Config to `h90_direct.conf`.
4. **Scene for everything else:** set as the default/fallback Scene (no SSID condition). Set Config to `h90_main.conf`.
5. Set **Home > Global Routing** to **Scene**.

---

## How it works

### h90_main.conf

Uses an **allowlist** approach: only traffic explicitly matched by a rule-set is proxied. Everything else goes direct.

```
Matched rule-sets  →  PROXY
Everything else    →  DIRECT
```

Rule-sets are external `.list` files fetched from GitHub; Shadowrocket can auto-update them on a schedule. DNS is resolved exclusively via encrypted resolvers — all DNS traffic on port 53 is hijacked to enforce this.

### h90_direct.conf

No rules — all traffic goes directly via the `FINAL,DIRECT` fallback. DNS uses the system resolver (typically the home router). Intentionally minimal: the same TUN/skip-proxy settings are kept for network compatibility, but all proxy-specific DNS configuration is removed.

---

## Key settings

### h90_main.conf

| Parameter | Value | Notes |
|---|---|---|
| `ipv6` | true | IPv6 enabled; IPv4 takes priority |
| `prefer-ipv6` | false | |
| `dns-server` | Cloudflare + Google DoT/DoH | See [DNS](#dns) |
| `hijack-dns` | :53 | Intercepts all DNS on port 53 |
| `private-ip-answer` | true | Allows private IP responses from DNS |
| `dns-direct-system` | false | Direct traffic also uses encrypted DNS |
| `dns-direct-fallback-proxy` | true | Uses proxy if direct DNS resolution fails |
| `udp-policy-not-supported-behaviour` | REJECT | UDP dropped if Server doesn't support forwarding |
| `icmp-auto-reply` | false | Ping auto-reply disabled |

### h90_direct.conf

| Parameter | Value | Notes |
|---|---|---|
| `ipv6` | false | IPv6 disabled |
| `prefer-ipv6` | false | |
| `dns-server` | system | Uses router/system DNS |
| `fallback-dns-server` | system | |
| `hijack-dns` | :53 | Intercepts DNS on port 53 |
| `private-ip-answer` | true | Allows private IP responses from DNS |
| `dns-direct-system` | true | Direct traffic uses system DNS |
| `dns-direct-fallback-proxy` | true | Uses proxy if direct DNS fails |
| `udp-policy-not-supported-behaviour` | REJECT | |
| `icmp-auto-reply` | false | |

---

## Rule-sets

Rule-sets are maintained in a separate repository: **[haritos90/allow-domains](https://github.com/haritos90/allow-domains)**. That repo contains the full list of available domain and subnet lists, how they are built, and what each covers.

`no-resolve` on subnet rules means domain requests skip IP-based matching — only already-resolved IPs are checked.

### Current configuration (h90_main.conf)

| Rule-set | Raw URL | Policy |
|---|---|---|
| All domains (combined) | [russia-all-surge.list](https://raw.githubusercontent.com/haritos90/allow-domains/main/Russia/russia-all-surge.list) | PROXY |
| IPv4 subnets (combined) | [Subnets/IPv4/all-surge.list](https://raw.githubusercontent.com/haritos90/allow-domains/main/Subnets/IPv4/all-surge.list) | PROXY, no-resolve |
| IPv6 subnets (combined) | [Subnets/IPv6/all-surge.list](https://raw.githubusercontent.com/haritos90/allow-domains/main/Subnets/IPv6/all-surge.list) | PROXY, no-resolve |

### Customising rule-sets

Two common approaches:

- **Single Server** — use the combined lists (`all-surge.list`) and route everything matched to one PROXY policy.
- **Split by category** — use per-category lists and assign each to a different Server or Proxy Group. To avoid overlap with the combined list, use the `minus-*` files from that repo, which subtract a given category from the combined set.

See the [allow-domains README](https://github.com/haritos90/allow-domains#readme) for the full list of available files and instructions.

### Managing rules via Shadowrocket UI

**To add a rule-set:**

1. **Config** tab → tap **ⓘ** next to the active config → **Edit**.
2. Tap **Rules** → tap **+** (top right).
3. Set **Type** to `RULE-SET`.
4. Paste the raw URL of the `.list` file into the **URL** field.
5. Set **Policy** to `PROXY` or `DIRECT` as needed.
6. Tap **Save**. Drag to reorder if necessary — rules are matched top to bottom.

**To remove a rule-set:** swipe left on the rule → **Delete**.

**To change the policy on an existing rule:** tap the rule → change the **Policy** field → save.

> Rule-sets added via the UI are saved into the config file. After making changes, tap **Done** to apply them.

---

<details>
<summary>DNS (h90_main.conf)</summary>

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

`h90_direct.conf` uses `dns-server = system` — no encrypted DNS, the home router handles resolution.

</details>

---

<details>
<summary>Config file reference</summary>

> Based on [LOWERTOP/Shadowrocket](https://github.com/LOWERTOP/Shadowrocket) as of commit [2180f8cc](https://github.com/LOWERTOP/Shadowrocket/commit/2180f8cc63386bff9cfd0507d854f67a47c52b77) (2026-06-05). Check that repo periodically for new commits and update accordingly.
>
> Where a parameter is configured in `h90_main.conf`, the examples below reflect its actual values from that profile.

### Global Routing

Shadowrocket has four Global Routing modes, set from **Home > Global Routing**:

| Mode | Behaviour |
|---|---|
| **Config** | Traffic is routed according to the rules in the active config file. This is the normal operating mode. |
| **Proxy** | All traffic is proxied, ignoring config rules. |
| **Direct** | All traffic goes directly, but config settings (DNS, skip-proxy, etc.) are still applied. |
| **Scene** | Automatically switches between predefined Scenes that each have their own Routing mode and Config file. Useful for routing traffic only on certain networks (e.g. always proxy on cellular, go direct on trusted Wi-Fi). |

**Enable Fallback** (`Home > Global Routing > Enable Fallback`): when the active Server fails (after 3 consecutive failures), Shadowrocket switches to a different Server at random. If a Proxy Group is selected via Easy Mode, fallback is limited to Servers within that Group.

**Proxy Type** (`Settings > Proxy > Proxy Type`):

| Value | Behaviour |
|---|---|
| HTTP | System proxy mode. Connections Shadowrocket cannot handle via the proxy interface are handed over to TUN. |
| None | TUN mode. All network requests are processed through the TUN interface. Equivalent to `compatibility-mode = 3` in the config file. |

---

### [General]

#### Proxy interface

**`skip-proxy`** — Forces the listed domains and IP ranges to be handled by the TUN interface instead of the proxy interface. Does not mean direct connection — traffic still goes through Shadowrocket, just via TUN instead of the proxy port. Used to fix compatibility issues with apps that don't work well through a proxy. If TUN mode is enabled globally (`Proxy Type = None`), this setting has no effect — TUN handles everything already.

```
skip-proxy = 192.168.0.0/16,10.0.0.0/8,172.16.0.0/12,localhost,*.local,captive.apple.com
```

---

**`tun-excluded-routes`** — IP ranges that bypass the TUN interface entirely, allowing non-TCP protocols (UDP, ICMP, etc.) to pass through. The TUN interface can only handle TCP; everything else needs to be excluded here or it will be dropped.

```
tun-excluded-routes = 10.0.0.0/8,100.64.0.0/10,...
```

---

**`tun-included-routes`** — Adds a more specific routing table entry so that traffic on Wi-Fi interfaces (which have a smaller route table) passes through Shadowrocket. Rarely needed. Use only if you know what you are doing.

---

**`dns-server`** — Overrides the system DNS. Supports plain DNS, DoH, DoH3, DoQ, and DoT. Multiple servers are queried in parallel; the first response is used. Only resolves directly connected domains — proxy domains are always resolved by the proxy Server itself.

Supported formats:

| Format | Example |
|---|---|
| Plain DNS | `1.1.1.1` |
| DNS-over-HTTPS (DoH) | `https://dns.google/dns-query` |
| DNS-over-HTTP/3 (DoH3) | `h3://dns.alidns.com/dns-query` |
| DNS-over-QUIC (DoQ) | `quic://223.5.5.5` |
| DNS-over-TLS (DoT) | `tls://1.1.1.1:853` |

Some DoH servers support HTTP/3; Shadowrocket tries it first. To disable: append `#no-h3` to the URL.

**DNS-over-PROXY** — forwards DNS queries through a proxy Server instead of resolving locally:
```
dns-server = https://dns.google/dns-query#proxy
dns-server = https://dns.google/dns-query#proxy=ServerName
```
The `proxy=name` form specifies a named Server; the name must be URL-encoded. If the Server name is incorrect, Shadowrocket falls back to the default Server.

**ECS (EDNS Client Subnet)** — passes subnet information to the DNS server so it can return results optimised for a specific exit location rather than your actual client IP:
```
dns-server = https://dns.google/dns-query#ecs=120.76.0.0/14|2620:149:af0::10/56&ecs-override=true
```
`ecs-override=true` forces the specified subnet even if the client IP provides different geolocation data.

---

**`fallback-dns-server`** — Used when the primary DNS query fails or takes longer than 2 seconds. Accepts the same formats as `dns-server`. Use `system` to fall back to the ISP's DNS.

---

**`ipv6`** — Enables IPv6 support. When enabled, Shadowrocket queries both A (IPv4) and AAAA (IPv6) records but uses IPv4 addresses preferentially.

**`prefer-ipv6`** — When enabled, queries AAAA records first from IPv6-capable DNS servers and prefers IPv6 addresses. Enable only if your network and proxy Server fully support IPv6.

---

**`private-ip-answer`** — When `false`, if DNS returns a private IP address (RFC 1918 / RFC 6598 range), Shadowrocket treats the domain as DNS-hijacked and forces it through the proxy. Setting to `false` may break access to local network resources.

---

**`always-real-ip`** — Forces Shadowrocket to return real IP addresses (instead of fake/virtual IPs) when handling DNS requests via TUN. Needed for some apps that inspect IP addresses directly.

⚠️ Using `always-real-ip = *` can leak your real IP over QUIC/WebRTC connections. If you must enable it, also set `block-quic = all-proxy` or add the rule `AND,((PROTOCOL,UDP),(DST-PORT,443)),REJECT-NO-DROP`. In networks that use fake-IP routing (e.g. OpenWrt with dnsmasq), use Global Routing = Scene with a separate config where this is disabled.

```
#always-real-ip = *.apple.com,*.icloud.com,captive.apple.com,time.apple.com,*.ntp.org
```

---

**`hijack-dns`** — Intercepts DNS queries sent directly to the specified addresses. Useful for apps or devices that ignore the system DNS and use hard-coded servers (e.g. some streaming apps that query Google DNS directly).

```
hijack-dns = :53          # intercepts all DNS on port 53
hijack-dns = 8.8.8.8:53  # intercepts only queries to Google DNS
```

---

**`include`** — Embeds another config file. The current file takes precedence over the included one. Useful for maintaining a shared base config across multiple profiles.

```
include = base.conf
```

---

#### Implicit parameters

Implicit parameters are not visible in the Shadowrocket UI and can only be set in plain text editing mode.

---

**`dns-direct-system`** — When `true`, domain rules matched to DIRECT are resolved using the system DNS (usually ISP DNS) instead of `dns-server`. Default: `false`.

---

**`icmp-auto-reply`** — When `true`, Shadowrocket automatically responds to ping (ICMP) packets. Useful for network diagnostics, but may increase battery usage. Default: `true` in the source; often set to `false` to save battery.

---

**`always-reject-url-rewrite`** — When `false` (default), REJECT rules from URL Rewrite only apply when Global Routing is set to Config. When `true`, REJECT rules also apply in Proxy, Direct, and Scene modes.

---

**`dns-direct-fallback-proxy`** — When `true`, if DNS resolution for a direct connection fails, the request is retried via the proxy. Default: `true`.

---

**`udp-policy-not-supported-behaviour`** — What to do with UDP traffic when it matches a rule that points to a Server or policy that does not support UDP forwarding. `DIRECT` passes it through unproxied; `REJECT` drops it.

```
udp-policy-not-supported-behaviour = REJECT
```

---

**`stun-response-ip` / `stun-response-ipv6`** — Returns a fake IP address in response to WebRTC STUN requests, preventing real IP leakage. STUN is used by WebRTC to discover the device's public IP — if the STUN server is routed differently from the media relay, the real IP can be exposed even when using a proxy.

Setting these overrides the "Disable STUN" toggle in Settings.

```
stun-response-ip = 1.1.1.1
stun-response-ipv6 = ::1
```

---

**`compatibility-mode`** — Sets a specific network compatibility mode. Takes precedence over the related toggle in Settings.

| Value | Mode |
|---|---|
| 0 | Auto / Disable |
| 1 | Proxy with Loopback Address |
| 2 | Proxy Only |
| 3 | TUN Only (same as `Settings > Proxy > Proxy Type = None`) |
| 4 | Proxy without Loopback Address |
| 5 | No Default Route |

Other values may be used for compatibility with specific software; test before deploying.

---

**`always-ip-address`** — Forces all domain names to be resolved using local DNS. Set to `true` with caution — this can break CDN resolution for affected domains.

---

**`proxy-dns-server`** — Specifies which DNS server resolves proxy Server domain names. If not set, `dns-server` is used by default. DNS-over-PROXY connections use the system DNS if this is unset.

---

**`close-if-proxy-chain-missing`** — Controls behaviour when a relay Server in a Proxy Chain is unavailable. `true`: reject the connection. `false` (default): skip the unavailable relay and connect directly to the exit Server.

---

**`ipv6-only-if-no-ipv4-dns`** — When `true`, if no IPv4 DNS servers are reachable, Shadowrocket treats the network as IPv6-Only. Default behaviour when unset is the same as `false`.

---

**`block-quic`** — Controls QUIC protocol blocking. QUIC runs over UDP/443 and can bypass proxy rules that only match TCP.

| Value | Behaviour |
|---|---|
| `all-proxy` | Blocks QUIC only for proxied connections; direct connections can use QUIC normally. |
| `all` | Blocks all QUIC connections (proxied and direct). Equivalent to the rule `AND,((PROTOCOL,UDP),(DST-PORT,443)),REJECT-NO-DROP`. |
| `always-allow` | QUIC is never blocked. |

---

**`use-local-host-item-for-proxy`** — By default, domain names in proxy connections are always resolved by the remote Server. When set to `true`, if a local DNS mapping exists for the domain (in `[Hosts]`), Shadowrocket uses that mapped address in the proxy connection instead.

---

**`allow-dns-svcb`** — Allows DNS SVCB record queries. By default these are blocked: the system may issue SVCB queries instead of standard A queries, which prevents fake-IP addresses from being returned. Setting to `true` enables SVCB (and HTTPS record) queries. Only enable if you specifically need SVCB resolution.

---

**`allow-dns-all`** — When `true` (default), all DNS query types are forwarded: A/AAAA return fake-IP, all others are forwarded to upstream DNS normally. When `false`, queries other than A/AAAA (e.g. HTTPS, SVCB, TXT) return empty responses.

---

### [Hosts]

Local DNS mapping. Three supported syntaxes:

```
# Map domain to a fixed IP (bypasses DNS resolution entirely)
example.com = 1.2.3.4

# Route DNS queries for a domain to a specific DNS server
example.com = server:1.2.3.4

# Route DNS queries for a specific Wi-Fi network (by SSID) to a specific DNS server
ssid:My-Network = server:1.2.3.4
```

By default, host mappings do not apply to proxied connections (those are resolved remotely). Set `use-local-host-item-for-proxy = true` in `[General]` to apply mappings to proxy connections as well.

To add mappings via the UI: **Config > ⓘ > Hosts**.

---

### [URL Rewrite]

Rewrites HTTP(S) request URLs before the request is sent. Applied regardless of whether the connection is proxied or direct.

**Rewrite types:**

| Type | Behaviour |
|---|---|
| `HEADER` | Rewrites the URL and continues with the modified request using the current proxy policy. |
| `302` | Returns a 302 redirect to the client. |
| `307` | Returns a 307 redirect to the client (method-preserving). |
| `reject` | Rejects the request. |

The `always-reject-url-rewrite` implicit parameter controls whether `reject` rules apply outside Config routing mode.

**Syntax:** `<url-regex> <replacement> <type>`

Each rule matches the full request URL with a regular expression, then replaces it with the replacement string.

<details>
<summary>Regex syntax reference</summary>

| Pattern | Meaning | Example |
|---|---|---|
| `.` | Any single character | `a.c` matches `abc`, `a1c` |
| `*` | Zero or more of the preceding | `ab*c` matches `ac`, `abc`, `abbc` |
| `+` | One or more of the preceding | `ab+c` matches `abc`, `abbc` but not `ac` |
| `?` | Zero or one of the preceding; or makes quantifier non-greedy | `colou?r` matches `color` and `colour` |
| `^` | Start of string | `^https` matches only strings starting with `https` |
| `$` | End of string | `\.js$` matches strings ending with `.js` |
| `\.` | Literal dot (escape it — unescaped `.` matches any character) | `example\.com` |
| `[abc]` | Any one of the characters inside | `[Hh]ttp` matches `Http` or `http` |
| `[^abc]` | Any character NOT in the set | `[^/]+` matches a path segment with no slashes |
| `(abc)` | Capturing group — can be referenced in replacement as `$1`, `$2` … | `(www\.)?(example\.com)` |
| `a\|b` | Alternation — matches `a` or `b` | `http\|https` |
| `https?` | The `?` makes `s` optional | matches `http` and `https` |

**Common URL patterns:**

```
# Match any HTTP or HTTPS URL
^https?://

# Match a domain with or without www
^https?://(www\.)?example\.com

# Capture the path after the domain and reuse it in the replacement
^https?://old\.example\.com(/.*) https://new.example.com$1 HEADER

# Match any URL containing /ads/ in the path
^https?://[^/]+/ads/
```

</details>

```
^https?://(www\.)?g\.cn https://www.google.com 302
```

To edit via UI: **Config > ⓘ > URL Rewrite**.

---

### [Header Rewrite]

Modifies HTTP request or response headers. Useful for adding authentication, stripping tracking headers, or spoofing User-Agent strings.

**Parameters:**

- **type**: `http-request` (applies to request headers) or `http-response` (applies to response headers)
- **behavior**: the modification to perform:
  - `header-del` — removes a header field
  - `header-add` — adds a header field with a value
  - `header-replace` — replaces the value of an existing header field
  - `header-replace-regex` — replaces a header value using a regular expression
- **expression**: a regular expression matching the target URL — see [URL Rewrite regex syntax](#url-rewrite) for reference

Commonly used header fields: `Content-Type`, `User-Agent`, `Accept-Language`, `Authorization`, `Host`, `Referer`, `Cookie`, `X-Forwarded-For`, `Location`.

To edit via UI: **Config > ⓘ > Header Rewrite**.

---

### [Body Rewrite]

Modifies HTTP request or response body content. Supports [jq](https://jqlang.org/) syntax for JSON processing.

**Parameters:**

- **type**:
  - `http-request` — applies to the request body
  - `http-response` — applies to the response body
  - `http-request-jq` — processes the request body using jq syntax
  - `http-response-jq` — processes the response body using jq syntax
- **expression**: a regular expression matching the target URL — see [URL Rewrite regex syntax](#url-rewrite) for reference

Useful for modifying API responses, stripping unwanted fields, or injecting content into pages. Requires HTTPS Decryption to be enabled for HTTPS traffic.

To edit via UI: **Config > ⓘ > Body Rewrite**.

---

### [Map Local]

Returns a locally defined response for matched URLs without sending the request to the network. Useful for mocking responses during development, or for returning specific content instead of a generic REJECT.

**Parameters:**

- **type**: the response format:
  - `text` — plain text body
  - `file` — file content (local path or remote URL)
  - `tiny-gif` — 1×1 pixel transparent GIF (useful for ad pixel blocking)
  - `base64` — base64-encoded content
- **expression**: a regular expression matching the target URL
- **data**: the content to return
- **Additional headers**: optional HTTP response headers

To edit via UI: **Config > ⓘ > Map Local**.

---

### [MITM]

Enables HTTPS decryption (man-in-the-middle) for specified domains. Required for URL Rewrite, Header Rewrite, Body Rewrite, and Script to work on HTTPS traffic.

**How to enable:**

1. **Config > ⓘ > HTTPS Decryption > Certificates > Generate New CA Certificate > Install Certificate**
2. **iOS Settings > Downloaded Profile > Install**
3. **iOS Settings > General > About > Certificate Trust Settings** — enable trust for the Shadowrocket certificate

HTTPS decryption is per-config: if you switch config files, re-enable it for the new one. To avoid reinstalling the certificate, use a certificate module.

**Multi-device setup:** If multiple devices share a config via iCloud, use the **ⓘ certificate copy/paste** function to transfer and install the same certificate on each device rather than generating new ones.

**HTTP/2 MitM** (v2.2.81+): Shadowrocket can decrypt HTTPS traffic using HTTP/2. Enable via **Config > ⓘ > HTTPS Decryption > Man-in-the-Middle via HTTP/2**, or set `h2 = true` in `[MITM]`. If a module sets this, it overrides the config file setting.

**`hostname`**: comma-separated list of domains to decrypt. Wildcards supported. Prefix with `-` to exclude a domain:
```
hostname = *.example.com, -secure.example.com
```

⚠️ Some system frameworks and apps (e.g. anything using certificate pinning, or domains like `*.apple.com`) may fail or behave unexpectedly when decrypted. Only decrypt domains you specifically need.

---

### [Script]

Extends Shadowrocket with JavaScript. Scripts can intercept and modify requests, act as custom DNS resolvers, fire on events, or run on a schedule.

To edit via UI: **Config > ⓘ > Script**.

**`type`** — when the script runs:

| Type | Behaviour |
|---|---|
| `http-request` | Intercepts and can modify outgoing HTTP requests |
| `http-response` | Intercepts and can modify incoming HTTP responses |
| `event` | Runs when a specified system event occurs |
| `rule` | Used as a custom rule (returns a policy decision) |
| `dns` | Used as a custom DNS resolver |
| `cron` | Runs on a schedule (cron expression) |

**`engine`** — the JavaScript runtime to use:

| Value | When to use |
|---|---|
| `auto` | Default; Shadowrocket selects automatically |
| `jsc` | JavaScriptCore — fast, low overhead; best for `rule` and `dns` types |
| `webview` | WKWebView — higher memory, better compatibility; best for complex scripts that parse large JSON or use browser APIs |

For `http-request` and `http-response` types, Shadowrocket defaults to WebView when the `engine` parameter is not set. It is recommended to set it explicitly.

```
[Script]
my-script = type=http-request,engine=jsc,script-path=request.js
```

---

### [Proxy Group]

Proxy Groups allow routing different traffic to different Servers, or implementing automatic failover and load balancing. Groups can be nested — a Group can contain other Groups as its members.

To manage via UI: **Config > ⓘ > Proxy Groups**.

#### Group types

| Type | Behaviour |
|---|---|
| `select` | Manual selection. The user picks which policy (Server or Group) to use. In plain text, `select=0` picks the first policy, `select=1` the second, etc. |
| `url-test` | Automatically switches to the Server with the lowest latency, based on periodic tests. |
| `fallback` | Automatically switches to another available Server when the current one fails. Available Servers are determined by the most recent test. |
| `load-balance` | Distributes traffic across Servers in the Group. Requests to the same domain always use the same Server (sticky routing). |
| `random` | Randomly selects a Server for each request, regardless of domain. |

For all types except `select`, Shadowrocket uses the result of the most recent test and runs the next test cycle asynchronously. If a Server fails between test cycles, the failed Server may still be used briefly — enable the rollback option to reduce this window.

#### Group parameters

| Parameter | Description |
|---|---|
| `interval` | How often (in seconds) to re-run latency tests |
| `timeout` | Maximum time (in seconds) to wait for a test to complete before abandoning it |
| `tolerance` | Minimum latency improvement (in ms) required before switching to a new winner |
| `url` | URL to test against (e.g. `https://cp.cloudflare.com/generate_204`) |
| `hidden` | Set `hidden=1` to hide the Group from the UI. Useful for Groups used only as dependencies of rules, not directly by the user. Only configurable in plain text mode. |

#### Server filtering with REGEX

Use `policy-regex-filter` to select Servers from a subscription by name pattern. The regex is matched against each **Server name** (not URL).

<details>
<summary>REGEX filter patterns</summary>

| Goal | Pattern | Example match |
|---|---|---|
| Contains keyword A | `A` | `HK` matches `HK 01`, `HK Premium` |
| Contains A or B | `A\|B` | `HK\|Hong Kong` |
| Contains A and B | `(?=.*(A))(?=.*(B)).*` | `(?=.*(HK))(?=.*(01)).*` matches `HK 01` but not `HK 02` |
| Excludes A | `^((?!(A)).)*$` | `^((?!(Premium)).)*$` excludes any Server with "Premium" |
| Excludes A and B | `^((?!(A\|B)).)*$` | `^((?!(Premium\|Trial)).)*$` |
| Contains A, excludes B | `(?=.*(A))^((?!(B)).)*$` | `(?=.*(HK))^((?!(Premium)).)*$` — HK but not Premium |

**How the lookahead patterns work:**

`(?=.*(HK))` is a positive lookahead — it asserts that `HK` appears somewhere in the string without consuming characters. Chaining two lookaheads (`(?=.*(A))(?=.*(B))`) requires both to be present anywhere in the name. `^((?!(A)).)*$` uses a negative lookahead repeated for every character position to reject any string containing `A`.

</details>

#### Subscription-based Group

Enable the subscription switch on a Group to automatically populate it with Servers from a specific subscription. Combine with `policy-regex-filter` to select only matching Servers from that subscription.

#### Example (plain text)

```
# Without REGEX filter
My-Group = url-test,Server1,Server2,Server3,interval=600,tolerance=100,timeout=5,url=https://cp.cloudflare.com/generate_204

# With REGEX filter (matches Servers from a subscription)
HK-Group = url-test,My-Subscription,use=true,policy-regex-filter=HK|Hong Kong,interval=300,url=https://cp.cloudflare.com/generate_204

# Hidden technical group
Base-Group = url-test,Server1,Server2,url=https://cp.cloudflare.com/generate_204,hidden=1
```

---

### [Rule]

Rules determine which policy (PROXY, DIRECT, REJECT, or a Proxy Group) handles each connection. Matched top to bottom; first match wins.

**Priority order:**
1. Module rules override config file rules
2. Rules are matched top to bottom
3. Domain-based rules take priority over IP-based rules within the same compilation pass
4. During automatic compilation: explicit rules first → inferred/database rules (GeoIP) → FINAL last

#### Rule types

| Type | Matches | Example |
|---|---|---|
| `DOMAIN` | Exact domain name | `DOMAIN,www.example.com,DIRECT` |
| `DOMAIN-SUFFIX` | Domain and all subdomains | `DOMAIN-SUFFIX,example.com,DIRECT` → matches `a.example.com` |
| `DOMAIN-KEYWORD` | Any domain containing the keyword | `DOMAIN-KEYWORD,example,DIRECT` → matches `a.example.com` |
| `DOMAIN-WILDCARD` | Domain pattern with `*` and `?` wildcards | `DOMAIN-WILDCARD,a*.example*.com,DIRECT` |
| `IP-CIDR` | IPv4 or IPv6 address range | `IP-CIDR,192.168.1.0/24,DIRECT` |
| `IP-ASN` | IP addresses belonging to an ASN | `IP-ASN,13335,PROXY` (Cloudflare) |
| `GEOIP` | IP geolocation database | `GEOIP,RU,PROXY` |
| `DST-PORT` | Destination port | `DST-PORT,443,DIRECT` |
| `USER-AGENT` | User-Agent string (supports `*` wildcard) | `USER-AGENT,curl*,DIRECT` |
| `URL-REGEX` | Full URL matched against a regex | `URL-REGEX,^https?://.+/ad\.js,REJECT` |
| `RULE-SET` | External rule list (must include rule types) | `RULE-SET,https://…/list.list,PROXY` |
| `DOMAIN-SET` | External domain list (no rule type prefix) | `DOMAIN-SET,https://…/domains.list,PROXY` |
| `SCRIPT` | Custom JavaScript rule | `SCRIPT,my-rule-script,PROXY` |
| `AND` | Logical AND of sub-rules | `AND,((PROTOCOL,UDP),(DST-PORT,443)),REJECT-NO-DROP` |
| `OR` | Logical OR of sub-rules | `OR,((DST-PORT,80),(DST-PORT,443)),PROXY` |
| `NOT` | Logical NOT of a sub-rule | `NOT,((DST-PORT,443)),DIRECT` |
| `PROTOCOL` | Transport protocol (TCP/UDP) — only inside AND/OR/NOT | _(see AND example above)_ |
| `FINAL` | Fallback when nothing else matches | `FINAL,DIRECT` |

When the same value is set for DOMAIN, DOMAIN-SUFFIX, DOMAIN-WILDCARD, and DOMAIN-KEYWORD simultaneously, only one will take effect.

When a domain request hits an IP-based rule (IP-CIDR, IP-ASN, GEOIP), Shadowrocket queries local DNS to resolve the domain and checks the result against the rule. Add `no-resolve` to skip this for domain requests:
```
IP-CIDR,172.16.0.0/12,DIRECT,no-resolve
```

#### Policies

| Policy | Behaviour |
|---|---|
| `PROXY` | Forwards traffic through a Server |
| `DIRECT` | Direct connection, no Server |
| `REJECT` | Returns HTTP 404, no body |
| `REJECT-200` | Returns HTTP 200, no body |
| `REJECT-DICT` | Returns HTTP 200 with `{}` |
| `REJECT-ARRAY` | Returns HTTP 200 with `[]` |
| `REJECT-IMG` | Returns HTTP 200 with a 1×1 pixel GIF |
| `REJECT-TINYGIF` | Returns HTTP 200 with a 1×1 pixel GIF |
| `REJECT-DROP` | Drops the IP packet silently |
| `REJECT-NO-DROP` | Returns ICMP port unreachable |

Policies can also be a Proxy Group name, subscription name, or individual Server name.

#### Policy extension flags

- `extended-matching` — matches both the TLS SNI and the HTTP `Host` header. Enabled by default in Shadowrocket for domain rules.
- `pre-matching` — evaluates the rule before normal rule processing with low overhead. Only supported on REJECT policies. Rules with this flag have the highest priority.

```
DOMAIN,ad.example.com,REJECT,pre-matching
```

</details>

---

## Legal Notice

This software is intended for development, testing, and research purposes only.

The author does not provide any guarantees regarding:

- availability of network access
- compatibility with specific services
- compliance with any external restrictions

Users are solely responsible for how they use this software and must comply with applicable laws.

**Usage Restrictions**

This software is not intended to be used for bypassing access restrictions or violating applicable laws. The author does not support or encourage such use.
