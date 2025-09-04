# YesVRouter CLI Guide

## Overview
YesVRouter provides a powerful command-line interface for managing your virtual BNG and router.

## Quick Start
- Start CLI engine: `yesvcli`
- All commands designed to be simple and intuitive
- Tab completion and help available

## Basic Commands
```bash
# Start YesVRouter CLI
yesvcli

# Show system status
yesvrouter# show version
yesvrouter# show interfaces
yesvrouter# show subscribers

# Basic configuration
yesvrouter# configure
yesvrouter(config)# interface GigabitEthernet0/8/0
yesvrouter(config-if)# ip address 192.168.1.1/24
yesvrouter(config-if)# no shutdown
```

## BNG Operations
```bash
# PPPoE sessions
yesvrouter# show pppoe sessions
yesvrouter# show pppoe statistics

# IPoE subscribers  
yesvrouter# show subscribers
yesvrouter# show dhcp bindings

# RADIUS status
yesvrouter# show radius statistics
```

## Advanced Features
```bash
# NAT operations
yesvrouter# show nat44 sessions
yesvrouter# show nat44 pools

# QoS status
yesvrouter# show qos policies
yesvrouter# show qos statistics

# Security
yesvrouter# show access-lists
yesvrouter# show security sessions
```

## Monitoring
```bash
# System health
yesvrouter# show system resources
yesvrouter# show logs
yesvrouter# show alerts

# Performance
yesvrouter# show interface counters
yesvrouter# show cpu utilization
yesvrouter# show memory usage
```

For complete command reference, see FINAL_TASK_LIST.md Phase 7.3