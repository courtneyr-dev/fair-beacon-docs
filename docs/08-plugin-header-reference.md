# Plugin Header Reference

Quick reference for required and optional plugin headers when using FAIR Beacon.

## Required Headers

These headers must be present in your plugin's main PHP file for FAIR Beacon and Git Updater to function correctly.

### Plugin Name

```php
Plugin Name: Your Plugin Name
```

**Purpose**: Human-readable name of your plugin.

**Format**: Plain text, maximum 64 characters recommended.

**Example**: `Plugin Name: FAIR Beacon Documentation`

### Version

```php
Version: 1.0.0
```

**Purpose**: Current version number of your plugin.

**Format**: Semantic versioning (MAJOR.MINOR.PATCH) recommended.

**Example**: `Version: 2.3.1`

**Note**: This should match your GitHub release tag for consistency.

### GitHub Plugin URI

```php
GitHub Plugin URI: username/repository
```

**Purpose**: Tells Git Updater where to find your plugin's GitHub repository.

**Format**: `username/repository` (not a full URL).

**Example**: `GitHub Plugin URI: developer-ecosystem-engineering/fair-beacon`

**Critical**: Must match your actual GitHub username and repository name exactly. Case-sensitive.

**Common Mistakes**:
- ❌ `GitHub Plugin URI: https://github.com/username/repository`
- ❌ `GitHub Plugin URI: github.com/username/repository`
- ✅ `GitHub Plugin URI: username/repository`

### Primary Branch

```php
Primary Branch: main
```

**Purpose**: Specifies the default branch for source checkouts.

**Format**: Branch name (typically `main` or `master`).

**Example**: `Primary Branch: main`

**Note**: This should match your repository's default branch. GitHub now defaults to `main` for new repositories, but older repositories may still use `master`.

### Plugin ID (DID)

```php
Plugin ID: did:plc:abc123xyz
```

**Purpose**: Associates your plugin with a specific DID for verification.

**Format**: Full DID identifier starting with `did:plc:`.

**Example**: `Plugin ID: did:plc:7w3p5hnmzxu2ywny44vkzpfu`

**When to Add**: Add this header after creating your DID via FAIR Beacon. It's not needed during initial plugin development.

## Optional Headers

These headers provide additional metadata or control specific behaviors.

### Description

```php
Description: A brief description of what your plugin does.
```

**Purpose**: Short description of plugin functionality.

**Format**: Plain text, maximum 140 characters recommended.

**Example**: `Description: Enables decentralized plugin distribution via DIDs.`

### Author

```php
Author: Your Name or Organization
```

**Purpose**: Credits the plugin author.

**Format**: Plain text.

**Example**: `Author: Developer Ecosystem Engineering`

### Author URI

```php
Author URI: https://example.com
```

**Purpose**: Link to the author's website.

**Format**: Full URL with protocol.

**Example**: `Author URI: https://fair.pm`

### Plugin URI

```php
Plugin URI: https://example.com/plugin
```

**Purpose**: Link to the plugin's homepage or documentation.

**Format**: Full URL with protocol.

**Example**: `Plugin URI: https://github.com/developer-ecosystem-engineering/fair-beacon`

### Requires at least

```php
Requires at least: 6.0
```

**Purpose**: Minimum WordPress version required.

**Format**: WordPress version number (MAJOR.MINOR).

**Example**: `Requires at least: 6.0`

### Tested up to

```php
Tested up to: 6.4
```

**Purpose**: Highest WordPress version the plugin has been tested with.

**Format**: WordPress version number (MAJOR.MINOR).

**Example**: `Tested up to: 6.4`

### Requires PHP

```php
Requires PHP: 7.4
```

**Purpose**: Minimum PHP version required.

**Format**: PHP version number (MAJOR.MINOR).

**Example**: `Requires PHP: 7.4`

### Release Asset

```php
Release Asset: true
```

**Purpose**: Tells Git Updater to prefer pre-built release assets over building from source.

**Format**: Boolean (`true` or `false`).

**Example**: `Release Asset: true`

**Recommendation**: Set to `true` for production plugins with build processes. This ensures users download optimized, pre-built assets rather than raw source code.

### License

```php
License: GPL v2 or later
```

**Purpose**: Specifies the plugin's license.

**Format**: License name or identifier.

**Example**: `License: GPL v2 or later`

### License URI

```php
License URI: https://www.gnu.org/licenses/gpl-2.0.html
```

**Purpose**: Link to the full license text.

**Format**: Full URL with protocol.

**Example**: `License URI: https://www.gnu.org/licenses/gpl-2.0.html`

## Complete Example

Here's a complete plugin header with all recommended fields:

```php
<?php
/**
 * Plugin Name: FAIR Documentation Plugin
 * Plugin URI: https://github.com/example/fair-docs-plugin
 * Description: Demonstrates FAIR plugin distribution with complete headers.
 * Version: 1.2.0
 * Author: Example Developer
 * Author URI: https://example.com
 * License: GPL v2 or later
 * License URI: https://www.gnu.org/licenses/gpl-2.0.html
 * Requires at least: 6.0
 * Tested up to: 6.4
 * Requires PHP: 7.4
 * GitHub Plugin URI: example/fair-docs-plugin
 * Primary Branch: main
 * Release Asset: true
 * Plugin ID: did:plc:7w3p5hnmzxu2ywny44vkzpfu
 */

// Plugin code follows
```

## Header Format Rules

**Comment Block**: Headers must be in a PHP comment block (`/** ... */`) at the top of the main plugin file.

**One Per Line**: Each header must be on its own line.

**Format**: `Header Name: Header Value`

**Case Sensitivity**: Header names are case-insensitive, but values may be case-sensitive (e.g., GitHub usernames).

**Order**: Headers can appear in any order within the comment block.

**Whitespace**: Extra whitespace around the colon is allowed and ignored.

## Updating Headers

When updating headers:

1. **Edit the Main Plugin File**: Open your plugin's main PHP file (the file with the `Plugin Name:` header)
2. **Update the Relevant Headers**: Modify the headers you need to change
3. **Commit and Push**: Commit the changes to your repository
4. **Create a Release** (if changing `Version:`): Create a GitHub release for the new version
5. **Sync DID** (if changing service endpoint): Sync your DID via FAIR Beacon if you changed anything affecting the DID document

### Common Update Scenarios

**Version Bump**:
1. Update `Version:` header
2. Create a GitHub release with the same version number
3. No DID sync needed

**Adding DID After Creation**:
1. Add `Plugin ID:` header with your DID
2. Commit and push
3. No DID sync needed (the DID already exists)

**Changing Primary Branch**:
1. Update `Primary Branch:` header
2. Commit and push
3. No DID sync needed (this only affects Git Updater)

## Validation

To verify your headers are correct:

1. **Activate the Plugin**: Ensure the plugin activates without errors in WordPress
2. **Check FAIR Beacon**: Your plugin should appear in the FAIR Beacon plugin list
3. **Check Git Updater**: Go to **Dashboard → Updates** and verify Git Updater detects your plugin
4. **Test Service Endpoint**: Visit your beacon's service endpoint URL and verify it serves correct metadata

If your plugin doesn't appear in FAIR Beacon or Git Updater, double-check the required headers for typos or formatting issues.

## Troubleshooting Headers

### Plugin Not Detected by FAIR Beacon

**Check**:

- `GitHub Plugin URI` is in `username/repository` format (no URL prefix)
- `Primary Branch` matches your repository's default branch
- Plugin is activated in WordPress

### Git Updater Not Detecting Releases

**Check**:

- `Version` header matches your GitHub release tag
- `Release Asset: true` is set if you're attaching pre-built zips
- GitHub release is published (not a draft)

### Service Endpoint Serves Wrong Metadata

**Check**:

- `Version` header is up to date
- Plugin is activated
- Git Updater cache may need clearing (deactivate/reactivate Git Updater)

See [Troubleshooting Guide](06-troubleshooting-guide.md) for more detailed diagnostics.

## Reference Links

- [WordPress Plugin Header Documentation](https://developer.wordpress.org/plugins/plugin-basics/header-requirements/)
- [Git Updater Documentation](https://git-updater.com/knowledge-base/)
- [Semantic Versioning](https://semver.org/)
- [FAIR Project](https://fair.pm)
