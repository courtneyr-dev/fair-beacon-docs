# Concepts and Architecture

Understand the structure and design principles behind FAIR's decentralized plugin distribution system.

## Overview

FAIR (Federated API for Integrity and Resilience) uses a two-document architecture to separate trust establishment from metadata distribution. This separation enables flexible aggregation while maintaining cryptographic verification of plugin authenticity.

## The Two-Document Architecture

FAIR's architecture consists of two distinct types of documents that serve different purposes in the ecosystem.

### 1. DID Document (Trust Layer)

**Purpose**: Establishes cryptographic identity and trust anchors.

**Location**: Stored in the PLC (Public Ledger of Credentials) directory at `plc.directory`.

**Contents**:
- DID identifier (e.g., `did:plc:abc123xyz`)
- Public keys for signature verification
- Service endpoints pointing to beacon locations
- Rotation keys for DID updates

**Immutability**: Once created, the DID itself never changes. Updates to the DID document (like rotating keys or changing service endpoints) are signed operations that create a new version in the audit log.

**Example DID Document**:
```json
{
  "@context": ["https://www.w3.org/ns/did/v1"],
  "id": "did:plc:abc123xyz",
  "alsoKnownAs": ["at://your-site.com"],
  "verificationMethod": [{
    "id": "did:plc:abc123xyz#atproto",
    "type": "Multikey",
    "controller": "did:plc:abc123xyz",
    "publicKeyMultibase": "zQ3s..."
  }],
  "service": [{
    "id": "#atproto_pds",
    "type": "AtprotoPersonalDataServer",
    "serviceEndpoint": "https://your-site.com/fair-beacon/your-plugin/"
  }]
}
```

### 2. Plugin Metadata (Information Layer)

**Purpose**: Provides current plugin information for discovery and installation.

**Location**: Served by beacons at the service endpoint URL specified in the DID document.

**Contents**:
- Plugin name, description, author
- Current version and download URL
- Changelog and compatibility information
- Optional: screenshots, banners, icons

**Mutability**: This document changes frequently as plugins are updated. The beacon dynamically generates this metadata from Git Updater's data.

**Example Plugin Metadata**:
```json
{
  "name": "Your Plugin Name",
  "slug": "your-plugin-slug",
  "version": "1.2.0",
  "author": "Your Name",
  "download_link": "https://github.com/user/repo/releases/download/1.2.0/plugin.zip",
  "last_updated": "2025-12-01T10:00:00Z",
  "requires": "6.0",
  "tested": "6.4",
  "sections": {
    "description": "Plugin description...",
    "changelog": "Version 1.2.0: New features..."
  }
}
```

### Why Two Documents?

**Separation of Concerns**: Trust (DID) and information (metadata) have different lifecycles and security requirements.

**Flexible Aggregation**: Aggregators can cache and serve metadata without touching the trust layer.

**Efficient Updates**: Plugin updates don't require DID operations, only metadata refreshes.

**Verifiable Chain**: The DID document provides the cryptographic anchor; the metadata is served from a verified endpoint.

## Ecosystem Components

FAIR consists of several interacting components, each with a specific role.

### Beacon (FAIR Beacon Plugin)

**Role**: DID issuer and metadata publisher for plugin developers.

**Responsibilities**:
- Generate DIDs for plugins
- Sync DIDs to the PLC directory
- Serve plugin metadata at service endpoints
- Integrate with Git Updater for release data

**Who Runs It**: Plugin developers who want to distribute their work via FAIR.

**Location**: Installed on the developer's WordPress site.

### Aggregator (e.g., Aspire Cloud)

**Role**: Discovery and metadata caching for end users.

**Responsibilities**:
- Index DIDs from the PLC directory
- Fetch and cache plugin metadata from beacons
- Provide search and browse interfaces
- Serve aggregated metadata to FAIR plugin installations

**Who Runs It**: Third parties providing discovery services (can be multiple aggregators).

**Location**: Separate infrastructure (not WordPress).

### FAIR Plugin (End User)

**Role**: Plugin installer with DID verification.

**Responsibilities**:
- Query aggregators for available plugins
- Verify DIDs and cryptographic signatures
- Install and update plugins from verified sources
- Display security information to users

**Who Runs It**: WordPress site administrators who want to install plugins via FAIR.

**Location**: Installed on the end user's WordPress site.

### PLC Directory

**Role**: Public registry of DID documents.

**Responsibilities**:
- Store DID documents
- Serve DID documents for verification
- Maintain audit logs of DID operations

**Who Runs It**: Currently centralized (plc.directory), with plans for federation.

**Location**: External service (plc.directory).

### Git Updater

**Role**: GitHub integration layer.

**Responsibilities**:
- Read GitHub releases and metadata
- Provide version information to FAIR Beacon
- Serve download URLs for release assets
- Handle pre-built vs. source release logic

**Who Runs It**: Plugin developers (same site as FAIR Beacon).

**Location**: WordPress plugin installed alongside FAIR Beacon.

## Data Flow

Understanding how data flows through the FAIR ecosystem clarifies how the components interact.

### DID Creation Flow

1. Developer installs FAIR Beacon and Git Updater on their WordPress site
2. Developer creates a DID for their plugin via FAIR Beacon UI
3. FAIR Beacon generates keypair and DID document
4. DID document is synced to PLC directory
5. Service endpoint in DID document points to beacon's metadata URL
6. Developer adds DID to plugin header and commits to GitHub

### Plugin Discovery Flow

1. Aggregator periodically scans PLC directory for DIDs
2. For each DID, aggregator fetches plugin metadata from service endpoint
3. Aggregator indexes metadata (name, description, version, etc.)
4. End user searches for plugins via FAIR plugin UI
5. FAIR plugin queries aggregator API
6. Aggregator returns matching plugins with DID identifiers

### Installation Flow

1. User selects a plugin from search results
2. FAIR plugin retrieves DID document from PLC directory
3. FAIR plugin verifies service endpoint matches aggregator's claim
4. FAIR plugin fetches current metadata from beacon's service endpoint
5. FAIR plugin downloads plugin zip from URL in metadata
6. Plugin is installed with DID verification recorded

### Update Flow

1. Developer releases new version on GitHub
2. Git Updater automatically detects the new release
3. Beacon's metadata endpoint serves updated version information
4. Aggregator refreshes cached metadata (on next poll)
5. FAIR plugin checks for updates (queries aggregator)
6. User is notified of available update with DID verification

## Trust Model

FAIR's trust model is based on cryptographic verification rather than centralized authority.

### What FAIR Verifies

**DID Ownership**: The plugin comes from the DID holder (via service endpoint verification).

**Service Endpoint**: The metadata is served from the URL specified in the DID document.

**Consistency**: The DID in the plugin header matches the DID in the PLC directory.

### What FAIR Does NOT Verify

**Plugin Quality**: FAIR provides no assessment of code quality, security, or functionality.

**Developer Identity**: DIDs are pseudonymous; FAIR doesn't verify real-world identity.

**Malicious Intent**: A validly signed DID can still point to malicious code.

**Repository Control**: FAIR doesn't verify that the DID holder controls the GitHub repository.

### User Responsibility

Users must still evaluate:

- Developer reputation
- Plugin reviews and community feedback
- Code audits (for security-critical applications)
- Business continuity (will the developer maintain the plugin?)

FAIR provides the infrastructure for **verifiable distribution**, not a stamp of approval.

## Comparison to Traditional Distribution

### Centralized Plugin Repositories (e.g., WordPress.org)

**Advantages**:
- Manual code review process
- Centralized search and discovery
- Uniform guidelines and standards
- Community ratings and support forums

**Limitations**:
- Single point of failure
- Approval delays
- Platform lock-in
- Limited developer control over distribution

### FAIR's Approach

**Advantages**:
- Permissionless participation (anyone can issue DIDs)
- Developer control over releases and timing
- Resilient (multiple aggregators possible)
- Cryptographic verification of authenticity

**Trade-offs**:
- No centralized code review
- Users must evaluate plugin trustworthiness themselves
- Ecosystem still developing (fewer aggregators currently)
- More complexity for end users

FAIR complements traditional repositories rather than replacing them. Developers can use both channels simultaneously.

## Design Principles

FAIR's architecture follows several key design principles.

### Permissionless Participation

Anyone can run a beacon and issue DIDs without approval from a central authority. This lowers barriers to entry for developers.

### Cryptographic Verification

Trust is established through public key cryptography rather than centralized gatekeepers. DIDs provide verifiable ownership.

### Separation of Trust and Information

The DID document (trust) is separate from plugin metadata (information), allowing flexible caching and aggregation without compromising verification.

### Federated Discovery

Multiple aggregators can index the same DIDs, providing redundancy and preventing any single aggregator from becoming a chokepoint.

### Developer Control

Plugin developers maintain control over their release schedule, distribution channels, and plugin metadata. Beacons serve metadata dynamically from Git Updater data.

## Future Directions

FAIR is under active development. Planned improvements include:

**Aggregator Federation**: Multiple independent aggregators to improve resilience and prevent centralization.

**Enhanced Verification**: Additional signature verification for plugin zip files (beyond service endpoint verification).

**Reputation Signals**: Optional integration with reputation systems (reviews, security audits) while maintaining permissionless issuance.

**Improved Tooling**: Streamlined workflows for DID issuance, key rotation, and plugin metadata management.

**PLC Decentralization**: Plans to federate the PLC directory to eliminate the single centralized component.

## Next Steps

- **[Git Updater Integration](03-git-updater-integration.md)**: Optimize your release packaging for FAIR
- **[DID Lifecycle Management](05-did-lifecycle-management.md)**: Update and manage DIDs over time
- **[Security Considerations](07-security-considerations.md)**: Understand the trust model in depth
