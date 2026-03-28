---
sidebar_position: 8
---

# Configuring the Client

In this chapter you will configure the Prisma client to connect to your server.

## Client config flow

```mermaid
graph TD
    START["Start"] --> CREDS["Gather credentials\n(Client ID + Auth Secret)"]
    CREDS --> METHOD{"GUI or CLI?"}
    METHOD -->|"GUI"| GUI["Create profile\nin prisma-gui"]
    METHOD -->|"CLI"| TOML["Write client.toml"]
    GUI --> TRANSPORT["Select transport\n(QUIC / TCP / WS / ...)"]
    TOML --> TRANSPORT
    TRANSPORT --> PROXY["Configure proxy settings\n(browser or system-wide)"]
    PROXY --> DONE["Ready to connect!"]

    style DONE fill:#22c55e,color:#000
```

## Option 1: prisma-gui

1. Open prisma-gui, click **New Profile**
2. Fill in: Server Address, Client ID, Auth Secret, Transport (QUIC), Cipher Suite
3. Enable **Skip Certificate Verify** for self-signed certs
4. Save and Connect

## Option 2: CLI configuration

```toml title="client.toml"
socks5_listen_addr = "127.0.0.1:1080"
http_listen_addr = "127.0.0.1:8080"
server_addr = "YOUR-SERVER-IP:8443"
cipher_suite = "chacha20-poly1305"
transport = "quic"
skip_cert_verify = true

[identity]
client_id = "YOUR-CLIENT-ID"
auth_secret = "YOUR-AUTH-SECRET"

[logging]
level = "info"
format = "pretty"
```

## Transport selection guide

```mermaid
graph TD
    Q1{"Network blocks\nUDP?"} -->|"No"| QUIC["<b>QUIC</b>\ntransport = quic"]
    Q1 -->|"Yes"| Q2{"Have domain\n+ CDN?"}
    Q2 -->|"No"| Q3{"Heavy DPI?"}
    Q2 -->|"Yes"| Q4{"Censorship\nlevel?"}
    Q3 -->|"No"| TCP["<b>TCP</b>\ntransport = tcp"]
    Q3 -->|"Yes"| PTLS["<b>PrismaTLS</b>\ntransport = prisma-tls"]
    Q4 -->|"Light"| WS["<b>WebSocket</b>\ntransport = ws"]
    Q4 -->|"Heavy"| XPORTA["<b>XPorta</b>\ntransport = xporta"]

    style QUIC fill:#22c55e,color:#000
    style TCP fill:#3b82f6,color:#fff
    style WS fill:#f59e0b,color:#000
    style XPORTA fill:#ef4444,color:#fff
    style PTLS fill:#8b5cf6,color:#fff
```

| Situation | Transport | Config |
|-----------|-----------|--------|
| Normal network | QUIC | `transport = "quic"` |
| UDP blocked | TCP | `transport = "tcp"` |
| Hide IP (CDN) | WebSocket | `transport = "ws"` |
| Enterprise (CDN) | gRPC | `transport = "grpc"` |
| HTTP/2 stealth | XHTTP | `transport = "xhttp"` |
| Maximum stealth | XPorta | `transport = "xporta"` |
| Never blocked | SSH | `transport = "ssh"` |
| Kernel perf | WireGuard | `transport = "wireguard"` |

## Subscription management

Import servers from subscription URLs (Prisma, Clash YAML):

```toml
[[subscriptions]]
url = "https://example.com/sub/token"
name = "My Provider"
auto_update = true
update_interval_hours = 24
```

## Proxy groups

```mermaid
graph LR
    subgraph "Proxy Groups"
        AUTO["auto-best\n(latency test)"]
        FB["fallback\n(first available)"]
        LB["load-balance\n(round-robin)"]
    end

    subgraph "Servers"
        S1["tokyo-1"]
        S2["singapore-1"]
        S3["us-west-1"]
    end

    AUTO --> S1
    AUTO --> S2
    AUTO --> S3
    FB --> S1
    FB --> S2
    LB --> S2
    LB --> S3
```

```toml
[[proxy_groups]]
name = "auto-best"
type = "auto-url"
servers = ["tokyo-1", "singapore-1"]
test_url = "https://www.google.com/generate_204"
test_interval_secs = 300
```

## Port forwarding

```toml
[[port_forwards]]
name = "web-app"
local_addr = "127.0.0.1:3000"
remote_port = 3000
```

## DNS and TUN mode

```toml
[dns]
mode = "tunnel"    # DNS goes through the proxy tunnel

[tun]
enabled = true     # Capture all system traffic
```

## Browser/system proxy setup

**Firefox:** Settings > Network > SOCKS5 Host `127.0.0.1:1080`

**Chrome:** Use SwitchyOmega extension with SOCKS5 `127.0.0.1:1080`

**System-wide:** Set SOCKS proxy to `127.0.0.1:1080` in OS network settings

## Next step

Everything configured! Head to [Your First Connection](./first-connection.md).

---

## TUN Mode

TUN mode captures **all system traffic** through a virtual network interface, routing it through the Prisma tunnel. This is the most comprehensive proxy mode — no app configuration needed.

### Enabling TUN Mode

In the **GUI**: Toggle "TUN" in the proxy modes on the Home page.

In the **CLI** config:
```toml
[tun]
enabled = true
device_name = "prisma-tun0"  # or "utun0" on macOS
mtu = 1500
include_routes = ["0.0.0.0/0"]  # all traffic
exclude_routes = ["192.168.0.0/16"]  # optional: bypass local network
```

### Requirements

- **Windows**: Run as Administrator (wintun.dll is bundled with the GUI)
- **macOS**: Run with `sudo` (uses kernel utun interface)
- **Linux**: Requires `CAP_NET_ADMIN` capability

### DNS Modes with TUN

TUN mode works best with **Fake DNS** (the default when TUN is active):

| Mode | How it works | Best for |
|------|-------------|----------|
| **Fake** | Assigns fake IPs from a reserved pool; maps back to domains in the tunnel | Most users (default) |
| **Tunnel** | Forwards all DNS through the encrypted tunnel | Maximum privacy |
| **Smart** | Blocked domains via tunnel, others direct | Selective routing |
| **Direct** | No DNS processing | Advanced configurations |

---

## Per-App Proxy

Per-App mode lets you choose which applications route through the proxy. It requires TUN mode (automatically enabled when Per-App is toggled on).

### Quick Setup

1. Toggle **Per-App** in the proxy modes on the Home page
2. Go to the **Per-App** page in the sidebar
3. Choose **Include** mode (only selected apps proxied) or **Exclude** mode (all except selected)
4. Click **Refresh** to see running applications
5. Select the apps you want and click **Apply**

### Presets

Save your app selections as named presets for quick switching between configurations (e.g., "Work" with browser + Slack, "Gaming" with Steam excluded).
