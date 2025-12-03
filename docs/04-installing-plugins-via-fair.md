# Installing Plugins via FAIR

End-user guide for installing DID-verified plugins using the FAIR plugin.

## Introduction

FAIR provides a way to install WordPress plugins directly from developers' DIDs, with cryptographic verification of authenticity. This guide walks through the installation process from an end-user perspective.

### What You Need

- Admin access to a WordPress site
- The FAIR plugin installed and activated

### What FAIR Provides

- Search and discovery of DID-verified plugins
- Cryptographic verification that plugins come from their claimed sources
- Direct installation from developers' distribution channels

### What FAIR Doesn't Provide

FAIR verifies that a plugin comes from a specific DID holder, but does not:

- Vet plugin quality or security
- Verify developer identity beyond DID ownership
- Provide centralized code review

Users must still evaluate plugin trustworthiness based on developer reputation, community feedback, and their own security requirements.

## Installing the FAIR Plugin

### Download and Install

1. Obtain the FAIR plugin zip file (contact the FAIR team for the current distribution method)
2. In WordPress admin, go to **Plugins → Add New → Upload Plugin**
3. Select the FAIR plugin zip and click **Install Now**
4. Click **Activate Plugin**

After activation, you should see a **FAIR** menu item in your WordPress sidebar.

### Verifying Installation

Navigate to the FAIR plugin page:

1. Click **FAIR** in the WordPress admin sidebar
2. You should see a search interface for plugins

If the page displays errors or doesn't load, try:

- Deactivating and reactivating the plugin
- Checking your PHP error log for messages
- Verifying that your WordPress version is supported (6.0+)

## Searching for Plugins

FAIR uses aggregators to provide plugin search and discovery.

### How Search Works

1. You enter a search query in the FAIR interface
2. FAIR queries an aggregator (currently Aspire Cloud)
3. The aggregator returns matching plugins with their DIDs
4. FAIR displays results with verification information

### Search Tips

**Be Specific**: More specific queries return more relevant results. For example, "SEO optimization" is better than just "SEO".

**Check Multiple Queries**: Try different phrasings if you don't find what you're looking for initially.

**Use Developer Names**: If you know who develops the plugin, include their name in the query.

### Interpreting Results

Each search result shows:

- **Plugin Name**: The plugin's display name
- **Description**: Short description of functionality
- **Developer**: Who issued the DID for this plugin
- **Version**: Current version number
- **DID**: The decentralized identifier (e.g., `did:plc:abc123xyz`)

## Installing a Plugin

Once you've found a plugin to install, the process is straightforward.

### Installation Steps

1. Click on the plugin in search results to view details
2. Review the plugin information:
   - Description and features
   - Version and compatibility
   - Developer information
   - DID verification status
3. Click **Install**
4. FAIR performs verification:
   - Retrieves DID document from PLC directory
   - Verifies service endpoint
   - Fetches metadata from beacon
   - Downloads plugin from verified source
5. Upon successful verification and download, click **Activate**

### Understanding Verification

During installation, FAIR verifies:

**DID Resolution**: The DID exists in the PLC directory and resolves correctly.

**Service Endpoint**: The plugin metadata comes from the service endpoint specified in the DID document.

**Metadata Consistency**: The plugin information matches what the aggregator reported.

If any verification step fails, FAIR will display an error and refuse to install the plugin.

## Managing Installed Plugins

After installation, FAIR-installed plugins appear alongside plugins from other sources.

### Viewing Installed FAIR Plugins

Go to **Plugins → Installed Plugins** in WordPress admin. FAIR-installed plugins display normally, but include DID information in their details.

### Updating FAIR Plugins

Updates work similarly to traditional plugins:

1. When a new version is available, WordPress shows an update notification
2. Click **Update Now** for the plugin
3. FAIR re-verifies the DID before downloading the update
4. The new version is installed and activated

FAIR checks for updates by querying the beacon's service endpoint directly, ensuring you get the latest version from the original source.

### Uninstalling

Uninstall FAIR plugins the same way as any WordPress plugin:

1. Go to **Plugins → Installed Plugins**
2. Deactivate the plugin
3. Click **Delete**
4. Confirm deletion

The DID remains in the PLC directory, but the plugin is removed from your site.

## Security Considerations

FAIR provides cryptographic verification, but users must still exercise judgment.

### What FAIR Verifies

✓ The plugin comes from the DID holder
✓ The service endpoint matches the DID document
✓ The download URL is served by the verified beacon

### What FAIR Does NOT Verify

✗ Plugin quality or security
✗ Real-world identity of the developer
✗ Whether the plugin is malicious or safe
✗ Whether the developer controls the GitHub repository

### Evaluating Trustworthiness

Before installing a plugin via FAIR, consider:

**Developer Reputation**: Do you know the developer? Do they have a track record in the WordPress community?

**Community Feedback**: Are there reviews, discussions, or testimonials about this plugin?

**Source Code**: Is the code publicly available for inspection (e.g., on GitHub)?

**Security Audits**: Has the plugin been audited by security professionals?

**Active Maintenance**: When was the last update? Is the developer responsive to issues?

FAIR removes the need to trust a central repository, but you still need to trust the developer.

## Troubleshooting

### Plugin Not Found

**Symptom**: Your search returns no results for a plugin you know exists.

**Possible Causes**:

- The plugin's DID hasn't been indexed by the aggregator yet
- The plugin slug or name doesn't match your query
- The beacon serving the plugin is temporarily unavailable

**Solutions**:

- Try different search terms
- Wait and try again later (aggregators index periodically)
- Contact the plugin developer to verify the DID is published

### Installation Fails

**Symptom**: FAIR displays an error during installation.

**Possible Causes**:

- DID verification failed (service endpoint mismatch, etc.)
- Beacon is temporarily unavailable
- Download URL is invalid or inaccessible
- Plugin zip file is corrupted

**Solutions**:

- Check the error message for specific details
- Verify your internet connection
- Try again later
- Contact the plugin developer if the issue persists

### Slug Conflicts

**Symptom**: Plugin installs with a modified slug (e.g., `your-plugin-123456` instead of `your-plugin`).

**Cause**: WordPress detects a slug conflict with an existing plugin or directory.

**Impact**: Updates and dependencies may fail due to slug mismatch.

**Workaround**:

- Remove or rename the conflicting plugin/directory
- Reinstall the FAIR plugin
- See [Troubleshooting Guide](06-troubleshooting-guide.md) for details

### Updates Not Detected

**Symptom**: WordPress doesn't show available updates for a FAIR plugin.

**Possible Causes**:

- Beacon's metadata hasn't been updated yet
- Git Updater on the beacon site isn't detecting the new release
- WordPress update check cache hasn't refreshed

**Solutions**:

- Wait a few hours and check again
- Manually check the plugin's GitHub releases page
- Contact the developer if the issue persists

## Advanced Usage

### Installing via DID Directly

If you know a plugin's DID, you can install it directly without searching:

1. Go to **FAIR → Install Plugin**
2. Enter the DID (e.g., `did:plc:abc123xyz`)
3. Click **Fetch Plugin**
4. Review details and click **Install**

This bypasses aggregator search and queries the beacon directly.

### Switching Aggregators

FAIR currently uses Aspire Cloud as its aggregator, but support for multiple aggregators is planned. When available, you'll be able to:

- Configure which aggregator to use
- Query multiple aggregators simultaneously
- Fall back to alternative aggregators if one is unavailable

## Privacy Considerations

### What FAIR Sends to Aggregators

When you search for plugins, FAIR sends:

- Your search query
- Request timestamp and IP address (standard HTTP)

Aggregators may log this information, but FAIR itself doesn't track your searches.

### What FAIR Sends to Beacons

When you install or update a plugin, FAIR sends requests to:

- The PLC directory (to fetch the DID document)
- The beacon's service endpoint (to fetch metadata and download URL)

These requests include:

- The DID being queried
- Your site's IP address (standard HTTP)
- HTTP headers (user agent, etc.)

Beacon operators can see which sites are installing their plugins, similar to how developers can see download statistics from traditional repositories.

### Minimizing Data Sharing

To reduce data sharing:

- Use a VPN if you want to hide your IP address from aggregators and beacons
- Be selective about which plugins you query (aggregators see all searches)
- Consider running your own aggregator (advanced, requires infrastructure)

## Providing Feedback

### Reporting Issues

If you encounter problems with the FAIR plugin:

1. Check the [Troubleshooting Guide](06-troubleshooting-guide.md)
2. Report issues to the FAIR project via their communication channels
3. Include relevant error messages, WordPress version, and PHP version

### Rating Plugins

Currently, FAIR doesn't include a built-in rating or review system. To share feedback about plugins:

- Contact the plugin developer directly (check their website or GitHub)
- Discuss in WordPress community forums
- Write blog posts or reviews

Reputation and review systems may be added to FAIR in the future.

## Next Steps

- **[Concepts and Architecture](02-concepts-architecture.md)**: Understand how FAIR works under the hood
- **[Security Considerations](07-security-considerations.md)**: Learn more about FAIR's trust model
- **[Troubleshooting Guide](06-troubleshooting-guide.md)**: Diagnose common issues
