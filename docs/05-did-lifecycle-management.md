# DID Lifecycle Management

Manage your plugin DIDs over time, including updates, key rotation, and revocation.

## Overview

Once you've created a DID for your plugin, ongoing management ensures your beacon continues serving accurate metadata and maintains cryptographic integrity.

This guide covers:

- Syncing DIDs after beacon changes
- Updating plugin metadata
- Rotating keys for security
- Revoking DIDs when necessary
- Understanding the PLC audit log

## DID Operations

### Understanding DID Immutability

The DID identifier itself (e.g., `did:plc:abc123xyz`) never changes. However, the DID document—the data associated with that identifier—can be updated through signed operations.

**Mutable Fields**:
- Service endpoints (where your beacon serves metadata)
- Verification keys (public keys used for signatures)
- Also Known As (alternative identifiers)

**Immutable Fields**:
- DID identifier
- Creation timestamp
- Audit log of all operations

All updates to a DID document are recorded in the PLC directory's audit log, providing a complete history of changes.

## Syncing Your DID

Syncing pushes your current DID document to the PLC directory. You'll need to sync after certain beacon configuration changes.

### When to Sync

**Initial Creation**: Automatically synced when you create a DID via FAIR Beacon.

**Service Endpoint Changes**: If your beacon's URL changes (e.g., moving to a new domain).

**Key Rotation**: After generating new verification keys.

**Recovery Operations**: If the PLC directory's copy is out of sync with your beacon's local copy.

### How to Sync

1. Go to **FAIR Beacon** in WordPress admin
2. Find your plugin in the list
3. Click **Sync DID**
4. Wait for confirmation that the sync completed successfully

FAIR Beacon will:

1. Read the DID document from your local database
2. Sign the document with your private key
3. Submit the signed document to the PLC directory
4. Verify the update was recorded

If syncing fails, check:

- Your beacon has internet connectivity
- The PLC directory service is operational (plc.directory)
- Your private keys are intact in the WordPress database

## Updating Plugin Metadata

Plugin metadata (version, description, changelog, etc.) is served dynamically by your beacon and doesn't require DID operations.

### Automatic Updates

When you publish a new release on GitHub:

1. Git Updater detects the new release (within a few minutes to hours)
2. Your beacon's service endpoint automatically serves the updated metadata
3. Aggregators refresh their cache (on their next polling cycle)
4. End users see the new version when checking for updates

No action is required on your part beyond publishing the GitHub release.

### Manual Refresh

If you need to force a refresh:

1. **Git Updater Cache**: Deactivate and reactivate Git Updater
2. **Beacon Cache**: Some beacons may cache metadata; check FAIR Beacon documentation for cache-clearing options
3. **Aggregator Cache**: Wait for the aggregator's next polling cycle (varies by aggregator)

### Changing Plugin Metadata

To change metadata fields like description or author:

1. Update your plugin's main PHP file headers
2. Commit and push to GitHub
3. Git Updater will pick up changes automatically

For version updates, ensure you also create a corresponding GitHub release.

## Key Rotation

Rotating your verification keys is a security best practice, especially if you suspect a key has been compromised.

### Why Rotate Keys

**Compromise**: If your private key is exposed, rotate immediately to invalidate it.

**Precautionary**: Rotate periodically (e.g., annually) as a security measure.

**Team Changes**: Rotate when team members with key access leave.

**Cryptographic Upgrades**: Future-proof by adopting newer cryptographic algorithms when available.

### How to Rotate Keys

⚠️ **Warning**: Key rotation is a sensitive operation. Ensure you have backups and understand the implications before proceeding.

**Current Status**: As of December 2025, FAIR Beacon does not yet provide a user-friendly key rotation interface. Key rotation must be performed via manual database operations or API calls.

**Planned Feature**: Future versions of FAIR Beacon will include a "Rotate Keys" button in the admin interface.

### Manual Key Rotation Process

If you need to rotate keys before the UI is available:

1. **Generate New Keypair**: Use a tool like OpenSSL or a library to generate a new keypair
2. **Update Database**: Replace the old keys in the WordPress database (table: `wp_fair_beacon_dids`)
3. **Update DID Document**: Modify the DID document to reference the new public key
4. **Sign and Submit**: Sign the updated DID document with the new private key and submit to PLC directory
5. **Sync**: Verify the update via FAIR Beacon's sync function

⚠️ This process requires familiarity with the PLC directory API and DID document structure. Incorrect execution can render your DID unusable.

### Recovery Keys

The PLC directory supports rotation keys (separate from verification keys) that can be used to recover a DID if the primary keys are lost. 

**Current Status**: FAIR Beacon does not yet expose rotation key management in the UI. This feature is planned for future releases.

## Revoking a DID

If you need to permanently disable a DID (e.g., discontinuing a plugin), you can revoke it.

### When to Revoke

**Plugin Discontinued**: You're no longer maintaining the plugin.

**Security Compromise**: The plugin's integrity has been compromised and you want to prevent further installations.

**Ownership Transfer**: Transferring the plugin to another developer (they should issue their own DID).

**Rebranding**: Starting fresh with a new identity (though updating metadata is usually preferable).

### How to Revoke

⚠️ **Warning**: Revocation is permanent and cannot be undone. Users with the plugin installed will no longer receive updates.

**Current Status**: FAIR Beacon does not yet provide a revocation interface. Revocation must be performed via manual PLC directory API calls.

**Workaround**: Remove or modify the service endpoint in your DID document to point to a non-functional URL. This effectively breaks the chain of trust without formal revocation.

1. Go to **FAIR Beacon → Your Plugin**
2. Click **Edit DID** (if available) or manually edit the database
3. Change the service endpoint URL to `https://revoked.example.com/` or similar
4. Sync the DID
5. Aggregators will be unable to fetch metadata, effectively disabling the DID

### Informing Users

After revoking a DID:

- Update your plugin's README or website to inform users
- Consider adding a notice in the plugin's admin interface
- Provide migration instructions if applicable (e.g., "Use this alternative plugin")

Users who have already installed your plugin will see update failures. Clear communication helps them understand why.

## Understanding the PLC Audit Log

Every operation on a DID is recorded in the PLC directory's audit log. This provides transparency and enables verification of a DID's history.

### What's Recorded

- **Creation**: When the DID was first created
- **Updates**: Every change to the DID document (service endpoints, keys, etc.)
- **Signatures**: Cryptographic signatures proving each operation was authorized by the keyholder

### Viewing the Audit Log

The PLC directory provides an API endpoint for viewing a DID's history:

```
GET https://plc.directory/{did}/log
```

Example:

```
GET https://plc.directory/did:plc:abc123xyz/log
```

Returns a JSON array of all operations, ordered chronologically.

### What the Audit Log Reveals

**Creation Date**: When the DID was first issued.

**Update Frequency**: How often the DID document has been changed.

**Service Endpoint History**: All URLs that have served as service endpoints over time.

**Key Rotation Events**: When verification keys were changed.

Anyone can view the audit log for any DID. This transparency is a feature, not a bug—it allows users to verify a DID's history and detect suspicious changes.

## Backup and Recovery

Protecting your DID requires backing up critical data.

### What to Back Up

**Private Keys**: The most critical component. Without these, you cannot update or manage your DID.

**DID Document**: A copy of your current DID document (service endpoints, public keys, etc.).

**Database Records**: Backup the `wp_fair_beacon_dids` table in your WordPress database.

**WordPress Site**: Full site backup (standard WordPress backup practices apply).

### Where to Store Backups

- **Encrypted Archive**: Use GPG or similar to encrypt private keys before storing
- **Offline Storage**: Keep a copy on an encrypted USB drive or hardware security module
- **Password Manager**: Securely store keys in a password manager with strong encryption
- **Multiple Locations**: Store copies in multiple secure locations (encrypted cloud storage, physical media)

⚠️ **Never** store private keys in:
- Unencrypted files on your computer
- Plain text in notes apps
- Email or messaging services
- Public or shared cloud storage

### Recovery Scenarios

**Scenario 1: WordPress Site Crash**

If your WordPress site is lost but you have backups:

1. Restore your WordPress site from backup
2. Verify the `wp_fair_beacon_dids` table is intact
3. Sync your DID to ensure PLC directory is current

**Scenario 2: Private Key Loss (No Backup)**

If you've lost your private keys and have no backup:

- Your DID is **permanently inaccessible**
- You cannot update the service endpoint, rotate keys, or perform any operations
- Users can still install the plugin if the current service endpoint is functional
- You'll need to create a new DID with a new identifier

This is why **backing up private keys is critical**.

**Scenario 3: Service Endpoint Domain Lost**

If you lose control of your domain (the service endpoint URL):

1. Acquire a new domain
2. Set up your beacon on the new domain
3. Update the DID document's service endpoint to the new URL
4. Sync the DID

Aggregators and users will start fetching metadata from the new location.

## Monitoring Your DID

Regularly check that your DID is functioning correctly to catch issues early.

### What to Monitor

**Service Endpoint Availability**: Ensure your beacon is online and serving metadata.

```bash
curl https://your-site.com/fair-beacon/your-plugin-slug/
```

Should return valid JSON with current plugin metadata.

**PLC Directory Sync**: Verify your DID document in the PLC directory matches your local configuration.

```bash
curl https://plc.directory/did:plc:abc123xyz
```

Should return your current DID document with correct service endpoint.

**Aggregator Indexing**: Check that aggregators are successfully indexing your plugin.

Search for your plugin via the FAIR plugin UI. If it doesn't appear, aggregators may be having trouble fetching your metadata.

**Git Updater Functionality**: Verify Git Updater is detecting new releases.

Create a test release and check that Git Updater picks it up within a few minutes.

### Automated Monitoring

Consider setting up automated monitoring:

- **Uptime Monitoring**: Use a service like UptimeRobot to monitor your beacon's service endpoint
- **Cron Jobs**: Periodically fetch your DID document and compare to expected values
- **Alerting**: Set up alerts for when your service endpoint returns errors or unexpected data

Early detection of issues allows you to respond before users are affected.

## Migration and Ownership Transfer

If you need to transfer a plugin to another developer, plan carefully.

### Recommended Approach

**Don't Transfer the DID**: DIDs represent cryptographic identity. Transferring private keys is risky and breaks the trust model.

**Instead**:

1. New developer creates their own DID for the plugin
2. New developer updates the `Plugin ID` header to their DID
3. Users transition to the new DID over time

### Communicating the Transition

- Announce the transition in your plugin's changelog
- Publish a notice on your website or repository
- Encourage users to reinstall via the new DID
- Consider supporting both DIDs during a transition period

### What Happens to the Old DID

You can:

- Revoke it to prevent further installations
- Leave it active but update metadata to point to the new developer's version
- Continue serving metadata for existing installations while directing new users to the new DID

## Best Practices

### Regular Maintenance

- **Check Service Endpoint**: Monthly verification that your beacon is serving correct metadata
- **Review Audit Log**: Quarterly review of your DID's audit log for unexpected changes
- **Test Updates**: Before releasing major versions, test the update flow from beacon to end users
- **Backup Keys**: Quarterly verification that your key backups are secure and accessible

### Security Hygiene

- **Limit Key Access**: Only trusted team members should have access to private keys
- **Use Secure Storage**: Encrypt keys at rest and in transit
- **Rotate Periodically**: Annual key rotation as a precautionary measure
- **Monitor for Compromise**: Watch for signs of unauthorized access to your beacon or keys

### Documentation

- **Document Your Setup**: Maintain internal documentation of your beacon configuration
- **Record Key Locations**: Keep a secure record of where keys are backed up
- **Succession Planning**: Ensure multiple team members understand how to manage the DID if you're unavailable

## Next Steps

- **[Security Considerations](07-security-considerations.md)**: Understand the security implications of DID management
- **[Troubleshooting Guide](06-troubleshooting-guide.md)**: Diagnose issues with DIDs and beacons
- **[Plugin Header Reference](08-plugin-header-reference.md)**: Quick reference for required headers
