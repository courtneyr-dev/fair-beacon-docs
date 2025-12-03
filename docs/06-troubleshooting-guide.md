# Troubleshooting Guide

Diagnose and resolve common issues with FAIR Beacon, Git Updater, and the FAIR plugin ecosystem.

## Known Issues

This section documents known issues with current workarounds. These may be fixed in future releases.

### Issue: Slug Modification During Installation

**Symptom**: When installing a plugin via FAIR, WordPress modifies the plugin slug by appending a random number. For example, `pods` becomes `pods-121903`.

**Cause**: WordPress detects a slug conflict with an existing plugin or directory in `wp-content/plugins/` and automatically renames the new plugin to avoid collision.

**Impact**:

- Dependency resolution fails (dependencies look for the original slug)
- Updates may not work correctly
- Plugin functionality may break if it relies on slug-based paths

**Workaround**:

1. Before installing via FAIR, check for conflicting plugins or directories
2. If a conflict exists, remove or rename the conflicting item
3. Reinstall the plugin via FAIR
4. The plugin should now install with the correct slug

**Status**: Under investigation. Future releases may handle slug conflicts more gracefully.

### Issue: Modal Activation Button Not Responding

**Symptom**: After installing a plugin via the FAIR modal dialog, clicking the "Activate" button does nothing. The button appears but doesn't trigger activation.

**Cause**: JavaScript event handler not properly attached to the activation button in the modal context.

**Workaround**:

1. Close the modal
2. Go to **Plugins → Installed Plugins**
3. Locate the newly installed plugin
4. Click "Activate" from the plugins list page
5. The plugin activates normally

**Status**: Bug confirmed. Fix planned for next release.

### Issue: Security Contact Field Not Available

**Symptom**: FAIR Beacon's DID management interface mentions a "security contact" field, but no UI element exists to enter this information.

**Cause**: Feature partially implemented. Backend supports security contacts, but UI was not completed.

**Workaround**: None currently available. Security contact cannot be set via the UI.

**Status**: Low priority. Will be addressed in future release.

## FAIR Beacon Issues

Problems related to running a beacon and issuing DIDs.

### Plugin Not Detected by FAIR Beacon

**Symptom**: Your plugin doesn't appear in the FAIR Beacon plugin list, even though it's activated and has the correct headers.

**Check**:

1. **Git Updater is Activated**: FAIR Beacon relies on Git Updater to detect plugins. Ensure Git Updater is installed and activated.

2. **GitHub Plugin URI Header**: Verify the header in your plugin's main PHP file:
   ```php
   GitHub Plugin URI: username/repository
   ```
   - Format must be `username/repository` (not a full URL)
   - No `https://` or `github.com/` prefix
   - Use the actual GitHub username and repository name

3. **Primary Branch Header**: Ensure you've specified the primary branch:
   ```php
   Primary Branch: main
   ```
   - Use `main` or `master` depending on your repository's default branch

4. **Plugin is Activated**: The plugin must be activated in WordPress. FAIR Beacon only lists activated plugins.

5. **Refresh the Page**: Git Updater may cache plugin lists. Refresh the FAIR Beacon admin page.

**Solution**:

- Correct the headers and save the plugin file
- Deactivate and reactivate the plugin
- Deactivate and reactivate Git Updater (clears cache)
- Refresh the FAIR Beacon page

### DID Creation Fails

**Symptom**: Clicking "Create new PLC DID" results in an error or timeout.

**Check**:

1. **Internet Connectivity**: The beacon must be able to reach `plc.directory`. Test with:
   ```bash
   curl https://plc.directory/
   ```

2. **PHP Errors**: Check your WordPress debug log for PHP errors during DID creation:
   ```php
   // In wp-config.php
   define('WP_DEBUG', true);
   define('WP_DEBUG_LOG', true);
   ```
   Errors will appear in `wp-content/debug.log`.

3. **Database Permissions**: Ensure WordPress can write to the database. DID data is stored in custom tables.

4. **PLC Directory Status**: Verify the PLC directory service is operational. Check community channels for service status updates.

**Solution**:

- Resolve connectivity issues (firewall, hosting restrictions, etc.)
- Fix any PHP errors revealed in the debug log
- Try again after confirming PLC directory is operational

### Service Endpoint Not Serving Metadata

**Symptom**: Visiting your beacon's service endpoint URL (e.g., `https://your-site.com/fair-beacon/your-plugin-slug/`) returns a 404 error or blank page.

**Check**:

1. **Rewrite Rules**: FAIR Beacon uses WordPress rewrite rules. Flush rewrite rules:
   - Go to **Settings → Permalinks**
   - Click "Save Changes" without modifying anything
   - Test the service endpoint URL again

2. **Plugin Activated**: FAIR Beacon must be activated for service endpoints to work.

3. **Correct Slug**: Verify you're using the correct plugin slug in the URL. The slug matches your plugin's directory name in `wp-content/plugins/`.

4. **Server Configuration**: Some server configurations block URLs with certain patterns. Check your server error logs.

**Solution**:

- Flush rewrite rules
- Verify plugin activation
- Check server logs for blocked requests
- Test with a simple plugin slug (avoid special characters)

### Metadata Shows Wrong Version

**Symptom**: Your beacon's service endpoint serves outdated version information, even though you've released a new version on GitHub.

**Check**:

1. **Git Updater Cache**: Git Updater caches release data. Clear the cache:
   - Deactivate Git Updater
   - Reactivate Git Updater
   - Wait a few minutes for Git Updater to refresh data

2. **GitHub Release Published**: Verify the release is published (not a draft) on GitHub.

3. **Version Header**: Ensure the `Version:` header in your plugin file matches the Git release tag:
   ```php
   Version: 1.2.0
   ```

4. **Git Updater Detecting Release**: Check if Git Updater detects the update:
   - Go to **Dashboard → Updates**
   - Your plugin should appear with the new version
   - If not, Git Updater isn't detecting the release

**Solution**:

- Clear Git Updater cache (deactivate/reactivate)
- Ensure version header matches release tag
- Verify GitHub release is published and accessible
- Wait up to an hour for Git Updater to poll GitHub again

## Git Updater Issues

Problems with Git Updater detecting releases or serving metadata.

### Git Updater Not Detecting Releases

**Symptom**: New releases on GitHub don't appear in WordPress updates or FAIR Beacon metadata.

**Check**:

1. **GitHub API Rate Limit**: Git Updater uses GitHub's API, which has rate limits. Without an API token, you're limited to 60 requests/hour.
   - Check if you're hitting rate limits
   - Consider adding a GitHub API token (Git Updater Pro, or manual configuration)

2. **Release vs. Tag**: Ensure you created a GitHub *release*, not just a Git tag. Releases include metadata that Git Updater reads.

3. **Release Asset**: If you set `Release Asset: true`, ensure you attached a zip file to the release. Git Updater won't detect the release if it expects an asset but finds none.

4. **Repository Visibility**: For private repositories, Git Updater requires Pro and API token configuration.

**Solution**:

- Create a proper GitHub release (not just a tag)
- Attach release assets if `Release Asset: true`
- Add a GitHub API token to avoid rate limits
- Wait for Git Updater's next polling cycle (can take an hour)

### Release Asset Not Used

**Symptom**: End users download the source archive instead of your pre-built release asset.

**Check**:

1. **Release Asset Header**: Verify you have this header in your plugin:
   ```php
   Release Asset: true
   ```

2. **Asset Naming**: Git Updater looks for assets with specific naming patterns. Ensure your asset is named clearly (e.g., `plugin-name-1.0.0.zip` or `plugin-name.zip`).

3. **Asset Attached to Release**: Verify the asset is attached to the GitHub release (not just uploaded to the repository).

**Solution**:

- Add `Release Asset: true` header
- Name assets clearly and attach to releases
- Rebuild the release if needed

## FAIR Plugin (End User) Issues

Problems experienced by users installing plugins via FAIR.

### Plugin Not Found in Search

**Symptom**: A DID-verified plugin doesn't appear in FAIR search results.

**Check**:

1. **Aggregator Indexing**: Aggregators poll the PLC directory periodically. Your DID may not have been indexed yet.
   - Wait a few hours and search again
   - Check if the aggregator is operational (community channels)

2. **Service Endpoint Accessibility**: The aggregator must be able to fetch metadata from your beacon. Test:
   ```bash
   curl https://your-site.com/fair-beacon/your-plugin-slug/
   ```
   If this fails, aggregators can't index your plugin.

3. **DID Synced to PLC**: Verify your DID exists in the PLC directory:
   ```bash
   curl https://plc.directory/did:plc:your-did-here
   ```

**Solution**:

- Ensure your service endpoint is publicly accessible
- Sync your DID via FAIR Beacon if needed
- Wait for aggregator to index (can take hours to days depending on polling frequency)

### Installation Verification Fails

**Symptom**: FAIR plugin refuses to install a plugin, displaying a verification error.

**Check**:

1. **DID Resolution**: The DID must resolve correctly in the PLC directory. Test:
   ```bash
   curl https://plc.directory/did:plc:your-did-here
   ```
   Should return a valid DID document.

2. **Service Endpoint Match**: The service endpoint in the DID document must match where the beacon is actually serving metadata.
   - Check the DID document's `service` array
   - Ensure the URL is correct and accessible

3. **Metadata Availability**: The beacon must be serving metadata. Test the service endpoint URL in your browser.

4. **Network Issues**: Temporary network problems can cause verification failures. Try again.

**Solution**:

- Fix service endpoint URL if incorrect
- Sync DID if service endpoint changed
- Verify beacon is online and accessible
- Retry installation after resolving issues

### Updates Fail After Installation

**Symptom**: Plugin installed successfully via FAIR, but updates don't work.

**Check**:

1. **Slug Mismatch**: If the plugin was installed with a modified slug (e.g., `plugin-123456`), updates may fail because Git Updater looks for the original slug.
   - See "Slug Modification During Installation" above

2. **Service Endpoint Changed**: If the developer moved their beacon to a new domain without updating the DID, updates will fail.

3. **Beacon Offline**: If the developer's beacon is temporarily offline, updates can't be checked.

**Solution**:

- Reinstall with correct slug (remove conflicts first)
- Contact plugin developer if beacon appears to have moved
- Wait and try again if beacon is temporarily offline

## Debugging Techniques

General strategies for diagnosing FAIR ecosystem issues.

### Enable WordPress Debug Logging

Add to `wp-config.php`:

```php
define('WP_DEBUG', true);
define('WP_DEBUG_LOG', true);
define('WP_DEBUG_DISPLAY', false);
```

Errors will be logged to `wp-content/debug.log`.

### Check Service Endpoint Manually

Test your beacon's service endpoint:

```bash
curl -v https://your-site.com/fair-beacon/your-plugin-slug/
```

Look for:

- **200 OK**: Service endpoint is working
- **404 Not Found**: Rewrite rules may need flushing
- **500 Internal Server Error**: Check PHP error logs
- Valid JSON response with plugin metadata

### Verify DID Document

Fetch your DID document from PLC directory:

```bash
curl https://plc.directory/did:plc:your-did-here | jq .
```

Check:

- `id` matches your DID
- `service` array includes your beacon's URL
- `verificationMethod` contains public keys

### Test with Simple Plugin

If you're having trouble with a complex plugin, create a minimal test plugin:

```php
<?php
/**
 * Plugin Name: FAIR Test Plugin
 * Description: Minimal plugin for testing FAIR
 * Version: 1.0.0
 * GitHub Plugin URI: username/test-repo
 * Primary Branch: main
 */

// Minimal functionality
```

Issue a DID for this plugin and test the full flow. This helps isolate whether issues are specific to your main plugin or systemic.

### Check Browser Console (End Users)

If FAIR plugin UI isn't working:

1. Open browser developer tools (F12)
2. Go to Console tab
3. Look for JavaScript errors
4. Errors may indicate missing dependencies or conflicts with other plugins

### Inspect Network Requests

Use browser developer tools to inspect network requests:

1. Open browser developer tools (F12)
2. Go to Network tab
3. Perform the action (search, install, etc.)
4. Look for failed requests (red status codes)
5. Inspect request/response details for error messages

## Getting Help

If you've tried the above troubleshooting steps and still have issues:

### For Beacon Operators

1. **Check Documentation**: Review the relevant guides in this documentation
2. **Test Systematically**: Isolate the problem (is it Git Updater? FAIR Beacon? PLC directory?)
3. **Gather Information**:
   - WordPress version
   - PHP version
   - FAIR Beacon version
   - Git Updater version
   - Error messages from debug logs
   - Service endpoint URL
   - DID identifier
4. **Reach Out**: Contact the FAIR community via their communication channels with the above information

### For End Users

1. **Contact Plugin Developer**: Many issues are specific to individual plugins. Reach out to the developer first.
2. **Check Aggregator Status**: Verify the aggregator is operational (ask in community channels)
3. **Gather Information**:
   - WordPress version
   - PHP version
   - FAIR plugin version
   - DID or plugin name you're trying to install
   - Error messages (screenshot or copy from browser console)
4. **Report to FAIR Project**: If the issue seems to be with the FAIR plugin itself, report via FAIR's communication channels

## Reporting Bugs

When reporting bugs, include:

**Environment**:
- WordPress version
- PHP version
- Relevant plugin versions (FAIR Beacon, Git Updater, FAIR plugin)
- Hosting environment (if relevant)

**Steps to Reproduce**:
1. Clear, numbered steps to reproduce the issue
2. Expected behavior
3. Actual behavior

**Evidence**:
- Error messages (exact text, not paraphrased)
- Screenshots (if UI issue)
- Relevant log entries
- Network request details (if available)

**Impact**:
- How does this affect functionality?
- Is there a workaround?
- How critical is the issue?

Clear bug reports help developers diagnose and fix issues faster.
