---
description: >-
  The VPN module connects MikoPBX to a remote network over a secure tunnel.
  Supports OpenVPN, WireGuard, AmneziaWG and Tailscale on the x86_64 and
  arm64 platforms.
---

# VPN

The **VPN** module connects MikoPBX to a remote network over a secure tunnel. This is useful when the PBX sits behind NAT and needs to be reachable from outside, or when several sites have to be joined into a single network.

MikoPBX acts as a **VPN client**: you prepare the configuration on the VPN server side and then paste it into the module. The module brings the tunnel up automatically after the network is configured and on system startup, monitors its state, and re-establishes the connection if it drops.

{% hint style="warning" %}
**Important!** In accordance with the legislation on information, information technologies and the protection of information, this module may be used solely for the purpose of building and operating a virtual private network that ensures the secure transmission of data whose transmission is not prohibited by law. MIKO is not responsible for the user's actions if the user employs this information for purposes that contradict the legislation of the Russian Federation.
{% endhint %}

## Supported VPN types <a href="#types" id="types"></a>

The module supports four connection types. The type is chosen when creating a connection and determines the configuration format.

| Type | Purpose | Notes |
| --- | --- | --- |
| **WireGuard** | Modern, fast tunnel | Minimal configuration, high speed, ChaCha20‑Poly1305 encryption |
| **AmneziaWG** | WireGuard with traffic obfuscation | Same WireGuard capabilities plus a changed traffic signature |
| **OpenVPN** | Universal, compatible tunnel | TUN/TAP support, including legacy ciphers (BF‑CBC, DES, RC4) |
| **Tailscale** | Managed mesh network | Key-based authorization, works via Tailscale cloud or a self-hosted Headscale |

{% hint style="info" %}
Binaries for all VPN clients (including the AmneziaWG kernel module) ship inside the module itself, statically built for both **x86_64** and **arm64**. Nothing has to be installed on the PBX manually.
{% endhint %}

## Creating a connection <a href="#create" id="create"></a>

1. Make sure the **VPN** module is installed and enabled under **Module Management**.
2. Open the module settings and add a new connection.
3. Fill in the common fields and, depending on the selected type, the configuration or the Tailscale parameters.
4. Enable the connection and save. The tunnel comes up automatically.

Every connection is described by a common set of fields:

| Field | Purpose |
| --- | --- |
| **Connection name** | An arbitrary name for convenience (required). |
| **VPN type** | OpenVPN, WireGuard, AmneziaWG or Tailscale. |
| **Configuration** | The VPN configuration file text (replaced by separate fields for Tailscale). Required. |
| **Description** | An optional comment. |
| **Enabled** | Whether to bring the tunnel up. A disabled connection is stored but not started. |

{% hint style="warning" %}
The configuration is validated on save. If it is missing the mandatory directives for the selected type (see below), the module shows a warning.
{% endhint %}

For each type below you will find a sample client configuration to paste into the **Configuration** field and a breakdown of the available options.

## WireGuard <a href="#wireguard" id="wireguard"></a>

WireGuard is a compact, modern protocol: a plain-text config, high speed and strong encryption. It is the recommended default for most scenarios.

### Sample configuration

```ini
[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY>
Address = 10.10.0.2/24

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = <SERVER_IP>:51820
AllowedIPs = 10.10.0.0/24
PersistentKeepalive = 25
```

### Options

| Parameter | Section | Purpose |
| --- | --- | --- |
| `PrivateKey` | `[Interface]` | The PBX private key. Unique, stored only on the PBX. **Required.** |
| `Address` | `[Interface]` | The PBX IP address and subnet inside the tunnel. |
| `PublicKey` | `[Peer]` | The VPN server public key. **Required.** |
| `Endpoint` | `[Peer]` | Server address and port (`IP:port`, `51820` by default). **Required.** |
| `AllowedIPs` | `[Peer]` | Subnets whose traffic is routed into the tunnel. **Required.** |
| `PersistentKeepalive` | `[Peer]` | Keepalive interval (sec). Needed when the PBX is behind NAT (`25` recommended). |

{% hint style="info" %}
The `DNS = …` directive is removed automatically on startup: MikoPBX has no `resolvconf`, and its presence would prevent the interface from coming up.
{% endhint %}

Required sections and parameters: `[Interface]` with `PrivateKey`, and `[Peer]` with `PublicKey`, `AllowedIPs` and `Endpoint`.

## AmneziaWG <a href="#amneziawg" id="amneziawg"></a>

AmneziaWG is a layer on top of WireGuard that adds **traffic obfuscation**: junk noise and modified headers make the stream's signature differ from standard WireGuard. Performance stays close to WireGuard.

### Sample configuration

```ini
[Interface]
PrivateKey = <CLIENT_PRIVATE_KEY>
Address = 10.30.0.2/24
Jc = 4
Jmin = 40
Jmax = 70
S1 = 53
S2 = 17
H1 = 983740153
H2 = 462767079
H3 = 1539016181
H4 = 805765999

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = <SERVER_IP>:51821
AllowedIPs = 10.30.0.0/24
PersistentKeepalive = 25
```

### Obfuscation options

The standard WireGuard parameters are extended with masking parameters. They must **match the server exactly**, otherwise the tunnel will not come up.

| Parameter | Purpose |
| --- | --- |
| `Jc` | Number of junk packets added to add noise to the stream. |
| `Jmin` / `Jmax` | Minimum and maximum size of the random jitter. |
| `S1` / `S2` | Sizes of the "magic" headers that mask service packets. |
| `H1`–`H4` | Numeric header markers that change the packet signature relative to standard WireGuard. |

{% hint style="warning" %}
AmneziaWG runs in the kernel and requires the `amneziawg.ko` module built for the MikoPBX kernel (**6.12.73‑MikoPBX**). It already ships inside the VPN module — nothing extra needs to be installed. As with WireGuard, the `DNS = …` directive is removed automatically.
{% endhint %}

Required parameters: all the mandatory WireGuard parameters plus the `Jc` obfuscation parameter.

## OpenVPN <a href="#openvpn" id="openvpn"></a>

OpenVPN is the most universal option: it works with most existing servers, supports TUN and TAP modes, and certificate-based or static-key authentication. The configuration is supplied as a regular `.ovpn` file.

### Sample configuration (static key)

```ini
dev tun
remote <SERVER_IP> 1194
proto udp
cipher AES-256-CBC
ifconfig 10.20.0.2 10.20.0.1
secret [inline]
keepalive 10 60
persist-tun
persist-key
verb 3
allow-deprecated-insecure-static-crypto

<secret>
-----BEGIN OpenVPN Static key V1-----
<STATIC_KEY>
-----END OpenVPN Static key V1-----
</secret>
```

### Options

| Directive | Purpose |
| --- | --- |
| `dev tun` / `dev tap` | Interface type: `tun` (IP layer) or `tap` (Ethernet layer). **Required.** |
| `remote` | Server address and port. **Required** (or a `<connection>` block). |
| `proto` | Transport: `udp` (faster) or `tcp` (passes filters more reliably). |
| `cipher` | Encryption algorithm (e.g. `AES-256-CBC`). |
| `ifconfig` | Client and server tunnel IPs (for static-key mode). |
| `secret` | Static key: `[inline]` — embedded in the `<secret>` block, or a path to a file. |
| `keepalive` | Connection check interval and timeout. |
| `allow-deprecated-insecure-static-crypto` | Enables static-key mode in OpenVPN 2.7+. |

{% hint style="info" %}
If the configuration uses legacy ciphers (**BF‑CBC, DES, RC4, IDEA, CAST5, SEED**), the module automatically starts a dedicated `openvpn-legacy` build with the OpenSSL legacy provider linked in. This is only needed for compatibility with old OpenVPN servers (e.g. 2.2.x); no manual switching is required.
{% endhint %}

Required directives: `remote` (or a `<connection>` block) and `dev tun`/`dev tap`.

## Tailscale <a href="#tailscale" id="tailscale"></a>

Tailscale is not just a tunnel but a **managed mesh network**. Nodes discover each other through a control plane, receive addresses from the `100.64.0.0/10` range and connect directly. For Tailscale the module uses separate fields instead of a text config.

### Tailscale fields

| Field | Purpose |
| --- | --- |
| **Login Server** | Control plane URL. Leave empty for the Tailscale cloud, or enter the address of your own Headscale. |
| **Auth Key** | Auth key for non-interactive login. If left empty, browser-based authorization is offered instead. |
| **Hostname** | The PBX name in the tailnet, e.g. `mikopbx-office`. |
| **Additional arguments** | Other `tailscale up` flags, one per line (e.g. `--advertise-exit-node`). |

### Sample additional arguments

```bash
--advertise-exit-node
--advertise-routes=192.168.0.0/24
--accept-routes
```

{% hint style="info" %}
The Tailscale state (node identity and keys) is kept in the module's persistent storage and survives a PBX reboot and a module reinstall. Subsequent starts therefore perform a "warm" start — without re-authorization.
{% endhint %}

For an already connected session the interface offers **re-authorization** (get a new login link) and **logout** (log out and remove the stored state).

{% hint style="info" %}
The Tailscale build in the module supports transport over **AmneziaWG**: when obfuscation parameters are present, they are applied to the Tailscale tunnel automatically. This combines the convenience of a mesh network with transport obfuscation.
{% endhint %}

The required parameter for automatic (non-interactive) connection is the auth key (`--authkey`). Without it, manual authorization via a link is needed.

## Startup and status monitoring <a href="#status" id="status"></a>

The module brings all enabled connections up automatically: after the network is configured and after the PBX boots. The state of every tunnel is checked on a schedule (once a minute) — if a connection has dropped, it is brought back up.

The connection list shows the current status, and for active tunnels — the assigned IP address, the time of the last handshake and the amount of transferred data.

{% hint style="warning" %}
When a connection is disabled or deleted, its tunnel is stopped immediately and the associated network routes are removed.
{% endhint %}

## Configuring the VPN server <a href="#server" id="server"></a>

The module is the client side. The configuration you paste in is prepared on the VPN server side. The module repository (the `samples-server-configs` directory) contains ready-to-use scripts for quickly spinning up a test server of each type (WireGuard, AmneziaWG, OpenVPN with a static key, Headscale for Tailscale) — they generate key pairs and produce the client config right away, which you then copy into the module.

Key default parameters used in those samples:

| Type | Port | Tunnel subnet |
| --- | --- | --- |
| WireGuard | `51820/udp` | `10.10.0.0/24` |
| AmneziaWG | `51821/udp` | `10.30.0.0/24` |
| OpenVPN | `1194/udp` | `10.20.0.0/24` |
| Tailscale / Headscale | `443/tcp` | `100.64.0.0/10` |

For a step-by-step example of joining two MikoPBX systems over WireGuard (including the manual server-side setup), see:

{% content-ref url="../../faq/interconnections/wireguard-vpn.md" %}
[wireguard-vpn.md](../../faq/interconnections/wireguard-vpn.md)
{% endcontent-ref %}
