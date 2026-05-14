# Sing-box Advanced Proxy Configuration Documentation

This document provides a detailed explanation of a network proxy configuration based on the **Sing-box** universal format. This setup integrates modern encryption protocols and implements intelligent traffic splitting and DNS anti-pollution.

---

## 1. Core Architecture Overview

This configuration follows a "Split Routing" logic designed to allow domestic traffic to connect directly while automatically routing international traffic through a proxy.

- **Inbound Method**: Uses **TUN mode** for system-level global traffic interception.
- **DNS Strategy**: Implements a split DNS solution. Domestic domains use AliDNS (DoH), while international domains use Cloudflare DNS (DoH) via the proxy.
- **Encryption Protocols (Outbounds)**:
    - **VLESS-Reality**: Utilizes the XTLS-Vision protocol for superior stealth.
    - **Hysteria2**: A high-speed UDP-based protocol ideal for high-latency or packet-loss environments.
    - **TUIC v5**: A next-gen QUIC-based protocol offering excellent performance.
- **Auto-Selection**: Includes an `urltest` group that automatically probes latency every 5 minutes and switches to the fastest node.

---

## 2. Module Breakdown

### 2.1 Inbound Configuration
The `tun` mode is configured with key parameters:
- `address`: The virtual network interface address is set to `172.19.0.1/30`.
- `auto_route`: Automatically manages the system routing table.
- `strict_route`: Forces all system traffic through sing-box.
- `route_exclude_address`: Excludes private network ranges (e.g., 192.168.x.x) to ensure local device communication is unaffected.

### 2.2 DNS Configuration
The DNS module is critical for preventing pollution:
- **Local (AliDNS)**: Handles domestic domains for low latency.
- **Remote (Cloudflare DNS)**: Queries are sent through the proxy server to prevent DNS hijacking.
- **Routing Rules**:
    - Rejects PTR (Reverse Lookup) and `.local` domain requests.
    - Forces specific services (Google, GitHub, OpenAI, etc.) to use `remote` DNS.
    - Domestic services (QQ, WeChat, DingTalk, etc.) use `local` DNS.

### 2.3 Outbound Strategies
- **proxy**: A `selector` group allowing manual switching between `auto` (automatic selection) and specific nodes.
- **auto**: An `urltest` group that regularly checks latency against `https://www.gstatic.com/generate_204`.
- **Protocol Implementation**: Detailed VLESS, Hysteria2, and TUIC configurations with TLS/Reality encryption enabled.

### 2.4 Routing Rules (Route)
The routing engine processes packets in the following order:
1. **DNS Hijacking**: Intercepts DNS requests on port 53 for internal processing.
2. **Private IP Bypass**: Local network traffic is routed as `direct`.
3. **Domain Suffix Matching**: Specific lists (e.g., OpenAI, GitHub) are forced to `proxy`.
4. **Rule-Sets**:
    - Uses `geosite-cn` and `geoip-cn` binary rules.
    - Traffic matching Chinese IPs or domains is routed as `direct`.
5. **Final (Fallback)**: All unmatched traffic defaults to `proxy`.

---

## 3. Usage Instructions

### 3.1 Privacy and Security
This configuration is **de-sensitized**:
- **Critical**: Replace `111.111.111.111` in the `outbounds` section with your actual server IP.
- **Critical**: Replace `xxxxxxxxxxxxxxxxx` with your real `uuid`, `password`, `public_key`, and `short_id`.

### 3.2 Operating Environment
- Ensure your Sing-box version supports `vless-reality`, `hysteria2`, and `tuic`.
- Running Sing-box usually requires Administrator/Root privileges to create the TUN interface.

---

## 4. Configuration Preview

Ensure JSON syntax is valid when importing. The configuration is set to automatically download the necessary `.srs` rule-set files from GitHub.
