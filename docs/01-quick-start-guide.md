# FAIR Beacon Quick Start Guide

Get FAIR Beacon running on your WordPress site and issue your first DID within 30 minutes.

## Introduction

### What is FAIR Beacon?

FAIR Beacon is a WordPress plugin that transforms your site into a DID (Decentralized Identifier) issuer for plugins. When you run a beacon, you can:

- Issue DIDs for plugins you maintain
- Serve plugin metadata to aggregators and end users
- Participate in the decentralized FAIR ecosystem

### Why Issue DIDs for Your Plugins?

- **Independence**: Distribute your plugins without relying on any single platform
- **Verification**: Cryptographic proof that updates come from you
- **Resilience**: Your plugins remain available even if centralized repositories have issues
- **Control**: Manage your own release schedule and distribution

### Prerequisites

Before you begin, ensure you have:

- [ ] A WordPress site where you have admin access
- [ ] Your plugin hosted on GitHub with at least one release
- [ ] FTP/SFTP access or the ability to upload plugins via WordPress admin

## Installing FAIR Beacon

FAIR Beacon can be installed two ways. Choose the method that best fits your workflow.

### Option A: Pre-packaged Zip (Recommended)

This method is fastest and requires no additional tools.

1. Download the pre-packaged FAIR Beacon zip file (includes all Composer dependencies)
2. In WordPress admin, go to **Plugins → Add New → Upload Plugin**
3. Select the zip file and click **Install Now**
4. Click **Activate Plugin**

The pre-packaged zip is available from the FAIR team. Check the project's communication channels for the current download link.

### Option B: Installing from GitHub

Use this method if you want to work with the latest development version or customize the plugin.

**Requirements**: Composer installed locally, command line access.

1. Clone the repository:
   ```bash
   git clone https://github.com/developer-ecosystem-engineering/fair-beacon.git
   ```

2. Install dependencies:
   ```bash
   cd fair-beacon
   composer install
   ```

3. Create a zip of the directory:
   ```bash
   cd ..
   zip -r fair-beacon.zip fair-beacon
   ```

4. Upload the zip via **Plugins → Add New → Upload Plugin** in WordPress admin

5. Activate the plugin

### Verifying Installation

After activation, you should see a **FAIR Beacon** menu item in your WordPress admin sidebar. If you don't see it, deactivate and reactivate the plugin, then check for any error messages.

## Installing Git Updater

FAIR Beacon requires Git Updater to function. Git Updater provides the release data and metadata that FAIR Beacon uses to create DID documents.

### Why Git Updater?

Git Updater:

- Reads your plugin's GitHub releases
- Provides version information, changelogs, and download URLs
- Serves pre-built assets when available
- Handles the connection between your GitHub repository and WordPress

### Installation Steps

1. Download Git Updater from [git-updater.com](https://git-updater.com/)
2. Install and activate it like any WordPress plugin
3. No additional configuration is required for basic FAIR Beacon operation

### Free vs. Pro

The free version of Git Updater works with public GitHub repositories. The Pro version ($20/year) adds:

- Private repository support
- GitHub API key management (higher rate limits)
- Additional Git hosting platforms

For most beacon operators using public repositories, the free version is sufficient.

## Preparing Your Plugin

Before you can issue a DID, your plugin needs specific headers in its main PHP file.

### Required Headers

Add these headers to your plugin's main PHP file (the one with the `Plugin Name:` header):

```php
/**
 * Plugin Name: Your Plugin Name
 * Plugin URI: https://example.com/your-plugin
 * Description: Your plugin description.
 * Version: 1.0.0
 * Author: Your Name
 * GitHub Plugin URI: your-username/your-repo-name
 * Primary Branch: main
 */
```

**GitHub Plugin URI**: Your GitHub username and repository name, separated by a slash. This tells Git Updater where to find your plugin.

**Primary Branch**: Usually `main` or `master`, depending on your repository's default branch.

You'll add the **Plugin ID** header after creating your DID in the next section.

## Creating Your First DID

With FAIR Beacon and Git Updater installed, and your plugin headers configured, you're ready to issue a DID.

### Step 1: Navigate to FAIR Beacon

In WordPress admin, click **FAIR Beacon** in the sidebar. You'll see a list of plugins that Git Updater has detected.

### Step 2: Select Your Plugin

Find your plugin in the list. If it doesn't appear:

- Verify the GitHub Plugin URI header is correct
- Ensure the plugin is activated
- Check that Git Updater is activated
- Try refreshing the page

### Step 3: Create the DID

Click **Create new PLC DID** next to your plugin. FAIR Beacon will:

1. Generate a new DID with public and verification keys
2. Sync the DID to the PLC directory
3. Create a service endpoint URL on your beacon site

This process takes a few seconds. When complete, you'll see the DID displayed (e.g., `did:plc:abc123xyz`).

### Step 4: Add the DID to Your Plugin

Copy the DID and add it to your plugin's header:

```php
/**
 * Plugin Name: Your Plugin Name
 * GitHub Plugin URI: your-username/your-repo-name
 * Primary Branch: main
 * Plugin ID: did:plc:abc123xyz
 */
```

Commit this change to your GitHub repository.

## Verifying Your Setup

After creating your DID, verify everything is working correctly.

### Check the PLC Directory

Your DID should now be registered in the PLC directory. The DID document contains:

- Your DID identifier
- Public keys for verification
- Service endpoint URL pointing to your beacon

### View Your Service Endpoint

Your beacon serves plugin metadata at:

```
https://your-site.com/fair-beacon/your-plugin-slug/
```

Visiting this URL should return JSON containing your plugin's metadata, including version information pulled from Git Updater.

### Test the Flow

To fully verify your setup:

1. Wait for an aggregator (like Aspire Cloud) to index your DID
2. On a different WordPress site with the FAIR plugin installed, search for your plugin
3. Your plugin should appear with DID verification

Note: Aggregator indexing time varies. It may take some time for your DID to appear in search results.

## Next Steps

Congratulations! You've issued your first DID. Here's what to explore next:

- **[Concepts and Architecture](02-concepts-architecture.md)**: Understand the two-document architecture and how the FAIR ecosystem works
- **[Git Updater Integration](03-git-updater-integration.md)**: Optimize your release packaging for FAIR
- **[DID Lifecycle Management](05-did-lifecycle-management.md)**: Learn how to update and manage your DIDs over time
- **[Troubleshooting](06-troubleshooting-guide.md)**: Solutions for common issues

## Getting Help

If you encounter issues:

1. Check the [Troubleshooting Guide](06-troubleshooting-guide.md)
2. Verify all prerequisites are met
3. Reach out to the FAIR community channels
