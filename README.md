# kvm_linux_agent
# 🌐 KVM Linux Gateway Audit & Traffic Analysis System

[![Status](https://img.shields.io/badge/Status-Operational-green)](https://github.com/Sean-Cai-X/kvm_linux_agent)
[![Audit Readiness](https://img.shields.io/badge/Audit_Status-Verified-blue)](https://github.com/Sean-Cai-X/kvm_linux_agent)

## 🔬 概述 (Overview)

本项目是一个基于 KVM Linux 网关的深度网络流量审计和安全分析系统。其核心目标是监控、记录和验证所有流经 KVM 网关的 "Image + Text" 交互数据，确保网络流量的完整性、合规性，并验证数据从网络捕获到数据库存储的完整链条（Audit Chain）。

该系统集成了 DNS 解析审计、流量过滤 (nftables)、代理流量监控 (sing-box)、网络包捕获 (tcpdump/tshark) 和日志存储 (SQLite)，形成了完整的**“网络流量捕获 -> 核心业务逻辑验证 -> 数据审计留痕”**的闭环。

---

## 🏗️ 核心架构与组件 (Architecture & Components)

本项目基于 KVM Linux Gateway 运行，关键组件包括：

| 组件 | 技术栈 | 功能描述 |
| :--- | :--- | :--- |
| **网络核心** | KVM Linux Gateway | 提供虚拟网络环境和流量处理基础。 |
| **流量控制** | `nftables` | 执行精细化的端口、协议和IP地址阻断规则，实现精确的网络治理。 |
| **DNS 审计** | `dnsmasq` | 记录和审计所有的 DNS 查询和解析行为，实现 DNS 卫生探针功能。 |
| **代理监控** | `sing-box` | 监控和管理代理流量，提供流量捕获的入口点。 |
| **流量捕获** | `tcpdump`/`tshark` | 在网关的关键路径上捕获原始网络数据包（PCAP）。 |
| **日志与审计** | `SQLite` | 作为核心数据存储层，记录 HTTP 事件、AI 切片、和链接等审计数据。 |
| **代理代理监控** | `codex-lan-agent` | 监控内部应用和代理服务的行为。 |

---

## 🔒 审计原则与数据流 (Audit Principles & Data Flow)

### 1. 审计链条 (Audit Chain)

系统关注的最小数据链是：`PCAP` $\rightarrow$ `SQLite` (http_events / ai_slices / ai_slice_links) $\rightarrow$ `18080 Edge UI`。

### 2. 核心验证规则 (Core Rule)

*   **循环回环流量 (Loopback):** 如果客户端访问的是本地主机 IP (`127.0.0.1:8095`)，该流量不会经过 KVM 网关的 NIC 路径。因此，这种流量必须使用本地主机证据进行验证。
*   **网络审计流量 (LAN IP):** 必须使用机器的实际 LAN IP (`LAN_IP:8095`)，才能被 KVM 网关捕获和审计。

### 3. 强制测试标记 (Mandatory Marker)
为了确保审计的可追踪性，所有文本载荷（Text Payload）必须包含一个**唯一标记**。

### 4. 证据层级 (Evidence Layers)
系统设计了四层证据验证体系：

*   **Layer 1: Transport Evidence (传输证据):** 基于 PCAP/tcpdump 捕获的 IP、端口、协议等基础网络信息。
*   **Layer 2: Body/Form Evidence (载荷证据):** 确认 HTTP 请求中是否包含 `multipart/form-data`、`image MIME` 和 `text marker`。
*   **Layer 3: Gateway SQLite Evidence (数据库证据):** 确认网关审计链是否成功记录了 `http_events`、`ai_slices` 和 `ai_slice_links` 三个关键行。
*   **Layer 4: UI/Operator Evidence (操作员界面证据):** 确认 `18080 UI` 表中事件和切片的数量是否相应增加。

---

## 📊 当前系统状态与审计结论 (Current Status & Audit Findings)

*(根据《审计汇报结论稿》节选)*

### 1. 服务状态 (Service Status)
当前所有核心服务均处于 `active (running)` 状态：`dnsmasq`, `sing-box`, `nftables`, `codex-gateway-capture`, `codex-lan-agent`。

### 2. 关键网络配置
*   **监听端口:** 22/tcp (SSH), 53/tcp, 53/udp (dnsmasq), 2080/tcp (sing-box inbound), 18080/tcp (codex-lan-agent)。
*   **DNS 配置:** `dnsmasq` 正在执行本地解析和污染阻断规则 ，并使用 DNS 健康探针。

### 3. 流量命中分析
*   **主要链路:** 当前主要的被命中链路下的 **UDP/**。
*   **阻断效果:** 旧的 UDP链路纳入 DNS 黑洞和 `nftables` 精准阻断。
*   **附加通道:** 附加加密通道已被成功识别并纳入阻断策略。

### 4. 后续建议
继续以“现网结果 / 证据 / 异常”的口径进行审计汇报，重点监控 `nftables` 计数器和 `pcap` 文件的变化，确保系统在网络环境变化时保持稳定和有效性。

---<img width="1763" height="5995" alt="image" src="https://github.com/user-attachments/assets/2e757b42-a53f-4b55-9c32-7fcc94cffff3" />

