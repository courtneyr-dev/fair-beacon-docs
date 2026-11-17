# FAIR Beacon Documentation

Developer documentation for the **FAIR DID/Beacon ecosystem** — a decentralized WordPress plugin distribution system using Decentralized Identifiers (DIDs).

> 🚀 **This project will be merged to the [@fairpm](https://github.com/fairpm) GitHub organization for [https://fair.pm](https://fair.pm)**

## What is FAIR?

**FAIR** (Federated and Independent Repositories) enables developers to distribute their plugins independently using decentralized identifiers (DIDs). With FAIR, developers can:

- **Issue DIDs** for their plugins using cryptographic keys
- **Serve package metadata** through beacon endpoints
- **Verify authenticity** with cryptographic proof that updates come from the original developer
- **Distribute packages** without relying on any single centralized platform

## Documentation

| Guide | Description |
|-------|-------------|
| [Quick Start Guide](docs/01-quick-start-guide.md) | Get FAIR Beacon running and issue your first DID in 30 minutes |
| [Concepts & Architecture](docs/02-concepts-architecture.md) | Understand the two-document architecture and ecosystem design |
| [Git Updater Integration](docs/03-git-updater-integration.md) | Configure Git Updater for optimal FAIR Beacon operation |
| [Installing Plugins via FAIR](docs/04-installing-plugins-via-fair.md) | End-user guide for installing DID-verified plugins |
| [DID Lifecycle Management](docs/05-did-lifecycle-management.md) | Manage DIDs over time: sync, rotate keys, and revocation |
| [Troubleshooting Guide](docs/06-troubleshooting-guide.md) | Diagnose and resolve common issues |
| [Security Considerations](docs/07-security-considerations.md) | Understand the trust model and security implications |
| [Plugin Header Reference](docs/08-plugin-header-reference.md) | Quick reference for required and optional plugin headers |

## How FAIR Works

FAIR uses a **two-document architecture**:

1. **DID Document (Trust Layer)** — Stored in the PLC directory, establishes cryptographic identity with public keys and service endpoints
2. **Package Metadata (Information Layer)** — Served by beacons, provides current plugin information for discovery and installation

### Ecosystem Components

- **FAIR Beacon** — WordPress plugin for developers to issue DIDs and serve plugin metadata
- **Aggregator** (e.g., Aspire Cloud) — Indexes DIDs and provides search/discovery for end users
- **FAIR Plugin** — WordPress plugin for end users to install DID-verified plugins
- **PLC Directory** — Public registry of DID documents at [plc.directory](https://plc.directory)
- **Git Updater** — Provides GitHub release data to FAIR Beacon

## Quick Start

### For Plugin Developers (Beacon Operators)

1. Install **FAIR Beacon** and **Git Updater** on your WordPress site
2. Add required headers to your plugin:
   ```php
   /**
    * Plugin Name: Your Plugin Name
    * GitHub Plugin URI: username/repository
    * Primary Branch: main
    */
   ```
3. Create a DID via **FAIR Beacon → Create new PLC DID**
4. Add the DID to your plugin header:
   ```php
   Plugin ID: did:plc:7iza6de2dwap2sbkpluginid
   ```
5. Your beacon now serves verified plugin metadata!

### For End Users

1. Install the **FAIR plugin** on your WordPress site
2. Search for plugins via the FAIR interface
3. Install with cryptographic DID verification

## Key Features

- ✅ **Permissionless** — Anyone can issue DIDs without approval
- ✅ **Cryptographic Verification** — Trust established via public key cryptography
- ✅ **Developer Control** — Maintain control over releases and distribution
- ✅ **Federated Discovery** — Multiple aggregators can index the same DIDs
- ✅ **Resilient** — No single point of failure for distribution

## Security Model

FAIR verifies:
- ✓ Plugin comes from the DID holder
- ✓ Service endpoint matches the DID document
- ✓ Metadata is served from verified endpoints

FAIR does **not** verify:
- ✗ Plugin quality or security
- ✗ Real-world identity of developers
- ✗ Whether code is malicious

Users must evaluate developer reputation and plugin trustworthiness. See [Security Considerations](docs/07-security-considerations.md) for details.

## Contributing

Contributions to the FAIR ecosystem are welcome! This documentation repository is part of the broader FAIR project that will be hosted at [@fairpm](https://github.com/fairpm).

## Related Projects

- [FAIR Beacon](https://github.com/developer-ecosystem-engineering/fair-beacon) — WordPress plugin for issuing DIDs
- [Git Updater](https://git-updater.com/) — GitHub integration for WordPress plugins

## License

This documentation is part of the FAIR ecosystem. See the individual project repositories for license information.

---

**Learn more at [https://fair.pm](https://fair.pm)**
