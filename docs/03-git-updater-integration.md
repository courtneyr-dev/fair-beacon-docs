# Git Updater Integration

Configure Git Updater for optimal operation with FAIR Beacon, including release packaging and automation.

## Overview

FAIR Beacon relies on Git Updater to provide plugin release data. Git Updater reads your GitHub releases and serves metadata to WordPress, which FAIR Beacon then exposes via your DID service endpoint.

This guide covers:

- Git Updater configuration for FAIR
- Release packaging strategies
- Pre-built vs. source releases
- GitHub Actions automation

## Git Updater Basics

Git Updater connects WordPress plugins to Git repositories for updates. For FAIR, it provides the crucial link between your GitHub releases and the metadata served by your beacon.

### What Git Updater Provides to FAIR

**Version Information**: Current version, release date, and changelog.

**Download URLs**: Links to release assets (typically zip files).

**Compatibility Data**: WordPress version requirements and PHP version compatibility.

**Release Assets**: Pre-built plugin zips when available, or dynamically built from source.

### Free vs. Pro

**Free Version**:
- Public GitHub repositories
- Basic release detection
- Automatic updates from GitHub releases

**Pro Version** ($20/year):
- Private repository support
- API token management (higher GitHub API rate limits)
- Bitbucket, GitLab, Gitea support

For FAIR Beacon with public repositories, the free version is sufficient.

## Plugin Header Configuration

Git Updater requires specific headers in your plugin's main PHP file.

### Required Headers for FAIR

```php
/**
 * Plugin Name: Your Plugin Name
 * Plugin URI: https://example.com/your-plugin
 * Description: Your plugin description.
 * Version: 1.0.0
 * Author: Your Name
 * GitHub Plugin URI: username/repository
 * Primary Branch: main
 * Plugin ID: did:plc:abc123xyz
 */
```

**GitHub Plugin URI**: Your GitHub repository in `username/repository` format.

**Primary Branch**: Usually `main` or `master`. This is the default branch for source checkouts.

**Plugin ID**: Your DID (added after creating the DID via FAIR Beacon).

### Optional Headers

```php
/**
 * Requires at least: 6.0
 * Tested up to: 6.4
 * Requires PHP: 7.4
 * Release Asset: true
 */
```

**Release Asset**: When set to `true`, Git Updater will prefer pre-built release assets over building from source. Recommended for FAIR.

## Release Packaging Strategies

How you package your releases determines what end users download when they install your plugin via FAIR.

### Strategy 1: Pre-built Release Assets (Recommended)

**How It Works**:

1. You build a production-ready plugin zip locally or via CI/CD
2. You create a GitHub release and attach the zip as a release asset
3. Git Updater serves the pre-built zip URL to FAIR Beacon
4. End users download the pre-built asset directly

**Advantages**:

- Faster downloads (no build time)
- Consistent builds (same artifact for all users)
- Smaller file sizes (no dev dependencies, source files, or git history)
- Better security (no build process on end user's site)

**Requirements**:

- Add `Release Asset: true` header to your plugin
- Attach a zip file to each GitHub release
- Name the zip file clearly (e.g., `your-plugin-1.0.0.zip`)

### Strategy 2: Source Checkout

**How It Works**:

1. You create a GitHub release (no need to attach a zip)
2. Git Updater dynamically builds a zip from the repository source
3. End users download the dynamically-built source archive

**Advantages**:

- Simpler release process (just tag and release)
- No build step required

**Disadvantages**:

- Larger downloads (includes git metadata and all source files)
- May include development files (.git, tests, etc.)
- Requires WordPress to have zip creation capabilities

**When to Use**: Development or testing environments, or if you have very simple plugins with no build process.

## Creating Release Assets

For production FAIR deployments, pre-built release assets are strongly recommended.

### Manual Process

1. **Build your plugin**:
   ```bash
   # Install production dependencies
   composer install --no-dev --optimize-autoloader
   
   # Build frontend assets if applicable
   npm run build
   ```

2. **Create a clean directory**:
   ```bash
   mkdir -p release
   rsync -av --exclude='.*' --exclude='node_modules'      --exclude='tests' --exclude='release'      . release/your-plugin/
   ```

3. **Create the zip**:
   ```bash
   cd release
   zip -r your-plugin-1.0.0.zip your-plugin/
   ```

4. **Create GitHub release**:
   - Go to your repository's Releases page
   - Click "Draft a new release"
   - Tag version: `1.0.0`
   - Release title: `Version 1.0.0`
   - Description: Changelog
   - Attach `your-plugin-1.0.0.zip`
   - Click "Publish release"

### Automated Process with GitHub Actions

Automate release builds using GitHub Actions whenever you create a release tag.

**Create `.github/workflows/release.yml`**:

```yaml
name: Build Release Asset

on:
  release:
    types: [created]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.0'
      
      - name: Install Composer dependencies
        run: composer install --no-dev --optimize-autoloader
      
      - name: Setup Node.js (if you have frontend assets)
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install npm dependencies and build
        run: |
          npm ci
          npm run build
      
      - name: Create plugin directory
        run: |
          mkdir -p release
          rsync -av --exclude='.*' --exclude='node_modules'             --exclude='tests' --exclude='release'             --exclude='src' --exclude='.github'             . release/${{ github.event.repository.name }}/
      
      - name: Create zip
        run: |
          cd release
          zip -r ${{ github.event.repository.name }}.zip             ${{ github.event.repository.name }}/
      
      - name: Upload release asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ github.event.release.upload_url }}
          asset_path: ./release/${{ github.event.repository.name }}.zip
          asset_name: ${{ github.event.repository.name }}-${{ github.event.release.tag_name }}.zip
          asset_content_type: application/zip
```

**Usage**:

1. Commit this workflow to your repository
2. Create a new release via GitHub UI or CLI
3. GitHub Actions automatically builds and attaches the zip

This ensures every release has a properly built asset without manual intervention.

## Handling Composer Dependencies

If your plugin uses Composer dependencies, you must include them in release assets.

### Production Dependencies Only

```bash
composer install --no-dev --optimize-autoloader
```

This installs only dependencies from `require`, not `require-dev`, and optimizes the autoloader for production.

### Vendor Directory

Ensure your release zip includes the `vendor/` directory. The end user's site won't run Composer, so dependencies must be bundled.

### Autoloading

Make sure your plugin loads Composer's autoloader:

```php
// In your main plugin file
if (file_exists(__DIR__ . '/vendor/autoload.php')) {
    require_once __DIR__ . '/vendor/autoload.php';
}
```

## Versioning and Changelog

Proper versioning and changelog management improves the end user experience.

### Version Numbering

Follow semantic versioning (semver):

- **Major** (1.0.0): Breaking changes
- **Minor** (1.1.0): New features, backward compatible
- **Patch** (1.1.1): Bug fixes, backward compatible

Update the `Version:` header in your main plugin file with each release.

### Changelog Format

Git Updater reads changelogs from:

1. Release notes on GitHub (recommended)
2. `CHANGES.md` or `CHANGELOG.md` in your repository
3. Commit messages (if no release notes)

**Recommended Format** (in GitHub release notes):

```markdown
## Version 1.2.0

### Added
- New feature X
- Support for Y

### Fixed
- Bug causing Z
- Issue with W

### Changed
- Improved performance of A
```

This provides clear information to users about what changed and is automatically served by Git Updater to FAIR Beacon.

## Testing Your Setup

Before relying on FAIR for distribution, verify your Git Updater configuration works correctly.

### Test Release Detection

1. Create a test release on GitHub
2. On your beacon site, activate your plugin
3. Go to **Dashboard → Updates**
4. You should see your plugin listed with the new version available

If your plugin doesn't appear, check:

- `GitHub Plugin URI` header is correct
- Plugin is activated
- Git Updater is activated and functional
- GitHub release is published (not draft)

### Test FAIR Beacon Metadata

After creating a DID for your plugin:

1. Visit your service endpoint: `https://your-site.com/fair-beacon/your-plugin-slug/`
2. Verify the JSON includes current version information
3. Check that `download_link` points to your release asset

The metadata should update automatically when you publish new releases.

## Troubleshooting

### Plugin Not Detected by Git Updater

**Symptom**: Plugin doesn't appear in FAIR Beacon's plugin list.

**Solutions**:

- Verify `GitHub Plugin URI` format is correct (`username/repo`, not a URL)
- Ensure plugin is activated
- Check that Git Updater is activated
- Refresh the page (Git Updater may cache plugin lists)

### Release Asset Not Used

**Symptom**: End users download the source archive instead of pre-built zip.

**Solutions**:

- Add `Release Asset: true` to plugin header
- Ensure release asset is named clearly (include plugin slug)
- Verify asset is attached to release (not just uploaded to repo)

### Metadata Not Updating

**Symptom**: FAIR Beacon serves stale version information after new release.

**Solutions**:

- Wait a few minutes (Git Updater may cache data)
- Deactivate and reactivate Git Updater to clear cache
- Check that the new release is published on GitHub
- Verify the version number in the plugin header matches the release tag

## Best Practices

### Release Checklist

Before publishing a release:

- [ ] Update `Version:` header in main plugin file
- [ ] Update changelog in README or release notes
- [ ] Build production asset with optimized dependencies
- [ ] Test the plugin locally with the built asset
- [ ] Create GitHub release with pre-built zip attached
- [ ] Verify FAIR Beacon serves updated metadata
- [ ] Test installation via FAIR on a separate site

### Security Considerations

- Don't include development dependencies in release assets
- Remove test files, `.git` directory, and other non-production files
- Use `.zipignore` or build scripts to exclude sensitive files
- Verify release assets don't contain secrets or API keys

### Performance Optimization

- Optimize Composer autoloader with `--optimize-autoloader`
- Minimize release asset file size (smaller downloads for users)
- Use production builds for frontend assets (minified JS/CSS)

## Next Steps

- **[DID Lifecycle Management](05-did-lifecycle-management.md)**: Update and manage your DIDs as your beacon evolves
- **[Installing Plugins via FAIR](04-installing-plugins-via-fair.md)**: Understand the end-user experience
- **[Troubleshooting Guide](06-troubleshooting-guide.md)**: Diagnose and fix common issues
