# sing-box-config
自用sing-box单节点配置,随版本更新

# Sing-box 高级代理配置说明文档

这是一份基于 **Sing-box** 通用格式生成的网络代理配置文件说明。该配置集成了多种现代加密协议，并实现了智能分流与 DNS 防污染功能。

## 1. 核心架构概述

该配置采用了典型的“分流控制”逻辑，旨在实现国内流量直连、国外流量自动走代理的效果。

- **入站方式 (Inbound)**: 使用 TUN 模式，实现系统级全局接管。
- **DNS 策略**: 采用分流 DNS 方案。国内域名使用 AliDNS (DoH)，国外域名通过代理使用 Cloudflare DNS (DoH)。
- **加密协议 (Outbounds)**:
    - **VLESS-Reality**: 使用 XTLS-Vision 协议，具备极强的隐蔽性。
    - **Hysteria2**: 基于 UDP 的高速传输协议，适合高延迟或丢包环境。
    - **TUIC v5**: 新一代基于 QUIC 的协议，性能卓越。
- **自动选择**: 内置 `urltest` 组，每 5 分钟自动探测延迟并切换至最快节点。

---

## 2. 模块详解

### 2.1 入站协议 (Inbounds)
配置了 `tun` 模式，其关键参数包括：
- `address`: 指定虚拟网卡地址 `172.19.0.1/30`。
- `auto_route`: 自动配置系统路由表。
- `strict_route`: 强制所有流量进入 sing-box。
- `route_exclude_address`: 排除内网网段（如 192.168.x.x），确保局域网设备互访不受影响。

### 2.2 DNS 配置
DNS 模块是防污染的核心：
- **Local (阿里 DNS)**: 处理国内域名，响应速度快。
- **Remote (Cloudflare DNS)**: 经过代理服务器查询，防止 DNS 劫持和污染。
- **分流规则**:
    - 拒绝 PTR (反向解析) 和 `.local` 域名请求。
    - 针对 Google、GitHub、OpenAI 等知名服务，强制指定使用 `remote` DNS。
    - 针对国内域名（QQ、微信、钉钉等）使用 `local` DNS。

### 2.3 出站策略 (Outbounds)
- **proxy**: 这是一个 `selector` 组，允许用户在 `auto`（延迟自动选路）和具体节点间手动切换。
- **auto**: 这是一个 `urltest` 组，定期对 `https://www.gstatic.com/generate_204` 进行测速。
- **协议实现**: 详细配置了 VLESS、Hysteria2 和 TUIC，均启用了 TLS/Reality 加密。

### 2.4 路由分流 (Route)
路由引擎根据以下顺序处理数据包：
1. **DNS 劫持**: 拦截 53 端口的 DNS 请求并转发给 sing-box 内部处理。
2. **私有 IP 绕过**: 局域网流量直接连接（`direct`）。
3. **域名后缀分流**: 针对特定列表（如 OpenAI, GitHub）强制走 `proxy`。
4. **Rule-Set (规则集)**: 
    - 使用 `geosite-cn` 和 `geoip-cn` 规则。
    - 匹配中国 IP 或域名的流量全部走 `direct`。
5. **Final (兜底)**: 所有未匹配的流量默认走 `proxy`。

---

## 3. 使用须知

### 3.1 隐私安全
该配置已进行**脱敏处理**：
- 请务必在 `outbounds` 部分将 `111.111.111.111` 替换为你真实的服务器 IP。
- 将 `xxxxxxxxxxxxxxxxx` 替换为你真实的 `uuid`、`password`、`public_key` 和 `short_id`。

### 3.2 运行环境
- 请确保 sing-box 版本支持 `vless-reality`、`hysteria2` 和 `tuic`。
- 运行 sing-box 时通常需要管理员/Root 权限以创建 TUN 网卡。

---

## 4. 配置文件预览

如果你需要将此配置导入 sing-box，请确保 JSON 语法正确，并且已下载对应的 `.srs` 规则集文件（配置中已设置自动下载）。
