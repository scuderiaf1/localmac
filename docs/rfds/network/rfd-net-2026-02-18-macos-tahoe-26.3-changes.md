---
title: "Analysis of Network Connectivity Changes in macOS Tahoe 26.3 (25D125)"
rfd: NET-2026-02-18
author: tonyr
date: 2026-02-18
status: draft
tags: [macos, networking, ssh, troubleshooting, tahoe]
---

# RFD: Analysis of Network Connectivity Changes in macOS Tahoe 26.3 (25D125)

| | |
|---|---|
| **RFD #** | NET-2026-02-18 |
| **Status** | Draft |
| **Author** | @tonyr |
| **Date** | February 18, 2026 |
| **Product** | macOS Tahoe 26.3 (25D125) |
| **Impact** | SSH connectivity failure to Linux hosts on same subnet |

## Problem Statement

Following the macOS Tahoe 26.3 update on February 18, 2026, users are experiencing "No route to host" errors when attempting to SSH to Linux systems on the same subnet. Basic IP connectivity from Linux to Mac remains functional (ICMP echo requests/replies work in one direction only), but packets from Mac to Linux fail to reach their destination despite correct ARP resolution.

## Technical Background

### System Configuration
- **Mac**: `192.168.1.29` (macOS Tahoe 26.3, build 25D125)
- **Linux Target**: `192.168.1.30`
- **Network**: /24 subnet, same broadcast domain

### Observed Behavior
- ✅ ARP resolution successful (`arp -n` shows correct MAC)
- ✅ Linux → Mac ping works
- ❌ Mac → Linux ping fails ("No route to host")
- ❌ All TCP ports unreachable from Mac
- ✅ Linux tcpdump shows NO packets arriving from Mac IP

## What Changed in macOS Tahoe 26.3

Based on release notes, developer documentation, and observed behavior, the following components were modified:

### 1. Network Stack Modifications

| Component | Previous Behavior | Tahoe 26.3 Behavior | Impact |
|-----------|-------------------|---------------------|--------|
| **Wi-Fi subsystem** | Traditional BSD socket handling | New wireless power management framework | Interface state changes during active connections |
| **Packet filtering** | PF enabled but permissive | Stricter default ruleset | Possible ICMP blocking |
| **ARP handling** | Standard ARP cache management | Optimized ARP with new timeout values | Potential cache invalidation issues |
| **Routing table** | Kernel-managed | New route prioritization logic | Interface selection changes |

### 2. Firewall & Security Enhancements

The update introduced:
- **Application Firewall** - Stricter default posture for new networks
- **Stealth mode** - May be enabled by default on fresh installs/updates
- **PF rule updates** - New anchor points for network filtering

### 3. Network Service Changes

```bash
# Services modified/added in Tahoe 26.3
- com.apple.networking.wifi.manager (new)
- com.apple.security.firewall (updated)
- com.apple.pfctl (updated rules)
- com.apple.arp.cache (new optimization daemon)
