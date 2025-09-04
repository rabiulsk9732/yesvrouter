YesVRouter - Advanced Virtual BNG & Router Platform
==================================================

## Introduction

YesVRouter is a high-performance, software-defined virtual BNG (Broadband Network Gateway) and router platform built on the proven Vector Packet Processing (VPP) technology. It provides enterprise-grade broadband access management, subscriber services, and advanced routing capabilities optimized for modern virtualized environments.

YesVRouter combines the power of VPP's high-performance packet processing with specialized BNG functionality including PPPoE session management, DHCP relay services, subscriber authentication, bandwidth management, and advanced QoS policies.

### Key Features

- **High Performance**: Built on VPP's vector packet processing architecture for maximum throughput
- **Virtual BNG**: Complete PPPoE session management with subscriber authentication 
- **DHCP Services**: Advanced DHCP relay with option-82 support for subscriber identification
- **Subscriber Management**: RADIUS authentication and per-subscriber bandwidth policies
- **Advanced Routing**: BGP, OSPF, static routing with traffic engineering capabilities
- **QoS & Traffic Shaping**: Hierarchical QoS policies and bandwidth management
- **High Availability**: Built-in redundancy and failover capabilities
- **API-Driven**: Comprehensive REST APIs and CLI for automation and management
- **Cloud Ready**: Optimized for containerized and virtualized deployments

For more information on the underlying VPP technology, please visit the
[FD.io website](http://fd.io/) and
[What is VPP?](https://wiki.fd.io/view/VPP/What_is_VPP%3F) pages.

## Architecture

YesVRouter is designed as a modular platform:

```
┌─────────────────────────────────────────────────────────────┐
│                     Management Plane                        │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │   Web UI    │ │  REST API   │ │   CLI Interface     │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                     Control Plane                           │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │ Routing     │ │ Session Mgmt│ │   Policy Engine     │   │
│  │ Protocols   │ │   (PPPoE)   │ │      (QoS)          │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
├─────────────────────────────────────────────────────────────┤
│                      Data Plane                             │
│             VPP Vector Packet Processing Engine             │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────────────┐   │
│  │   L2/L3     │ │   Tunnel    │ │    QoS & Policer    │   │
│  │ Forwarding  │ │  Processing │ │      Engine         │   │
│  └─────────────┘ └─────────────┘ └─────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Changes

Details of the changes leading up to this version of YesVRouter can be found under
doc/releasenotes.

## Directory layout

| Directory name         | Description                                 |
| ---------------------- | ------------------------------------------- |
| build-data             | Build metadata                              |
| build-root             | Build output directory                      |
| docs                   | Sphinx Documentation                        |
| dpdk                   | DPDK patches and build infrastructure       |
| extras/libmemif        | Client library for memif                    |
| src/examples           | YesVRouter example code                     |
| src/plugins            | YesVRouter bundled plugins directory        |
| src/svm                | Shared virtual memory allocation library    |
| src/tests              | Standalone tests (not part of test harness) |
| src/vat                | YesVRouter API test program                 |
| src/vlib               | YesVRouter application library              |
| src/vlibapi            | YesVRouter API library                      |
| src/vlibmemory         | YesVRouter Memory management                |
| src/vnet               | YesVRouter networking                       |
| src/vpp                | YesVRouter application                      |
| src/vpp-api            | YesVRouter application API bindings         |
| src/vppinfra           | YesVRouter core library                     |
| src/vpp/api            | Not-yet-relocated API bindings              |
| test                   | Unit tests and Python test harness          |

## Getting started

In general anyone interested in building, developing or running YesVRouter should
consult the documentation for complete coverage.

For the impatient, some salient information is distilled below.

### Quick-start: On an existing Linux host

To install system dependencies, build YesVRouter and then install it, simply run the
build script. This should be performed by a non-privileged user with `sudo`
access from the project base directory:

    ./extras/vagrant/build.sh

If you want a more fine-grained approach because you intend to do some
development work, the `Makefile` in the root directory of the source tree
provides several convenience shortcuts as `make` targets that may be of
interest. To see the available targets run:

    make

### Quick-start: Vagrant

The directory `extras/vagrant` contains a `VagrantFile` and supporting
scripts to bootstrap a working YesVRouter inside a Vagrant-managed Virtual Machine.
This VM can then be used to test concepts with YesVRouter or as a development
platform to extend YesVRouter.

### Configuration

YesVRouter uses a comprehensive configuration system supporting:

- **Startup Configuration**: `/etc/yesvrouter/startup.conf`
- **Runtime Configuration**: CLI commands and API calls
- **BNG Configuration**: PPPoE, DHCP relay, subscriber management
- **Routing Configuration**: BGP, OSPF, static routes, policy routing

#### Basic BNG Configuration Example

```
# Enable BNG functionality
bng {
  enable
  
  # PPPoE configuration
  pppoe {
    max-sessions 10000
    session-timeout 3600
  }
  
  # DHCP relay
  dhcp-relay {
    server 192.168.100.1
    option-82
  }
  
  # Subscriber management
  subscriber-mgmt {
    radius {
      server 192.168.1.10
      secret "shared-secret"
    }
  }
}
```

### Management and Monitoring

YesVRouter provides multiple management interfaces:

- **CLI**: Interactive command line interface
- **API**: RESTful HTTP API for automation
- **NETCONF**: Industry-standard network configuration protocol
- **Telemetry**: Real-time performance and statistics collection

## Use Cases

### Virtual BNG Deployment
- Replace hardware BNG with software-defined solution
- Scale PPPoE sessions dynamically based on demand
- Integrate with existing AAA infrastructure
- Provide subscriber-aware services and policies

### Service Provider Edge Router
- High-performance routing for service provider networks
- BGP route reflection and policy enforcement
- MPLS L3VPN services
- Traffic engineering and QoS

### Cloud Native Networking
- Containerized network functions
- Kubernetes-native deployment
- Auto-scaling based on traffic load
- Integration with cloud orchestration platforms

## Performance

YesVRouter inherits VPP's exceptional performance characteristics:

- **Throughput**: Multi-terabit per second forwarding capacity
- **Latency**: Sub-microsecond packet processing
- **Scale**: Support for millions of concurrent sessions
- **Efficiency**: Optimal CPU and memory utilization

## More information

Several modules provide documentation, see the docs directory for more
end-user-oriented information and developer notes.

## Community

- **Issues**: Report bugs and request features
- **Documentation**: Comprehensive guides and API references
- **Examples**: Sample configurations and use cases
- **Support**: Commercial support options available

## License

YesVRouter is licensed under the Apache 2.0 License. See LICENSE file for details.

---
*YesVRouter - Empowering the next generation of network infrastructure*

