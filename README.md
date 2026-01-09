# WordPress Plugin Actions by Perdives

Modular GitHub Actions for automating WordPress plugin development and releases.

## Available Actions

### 🏷️ get-plugin-version
Extract version number from WordPress plugin file header.

```yaml
- uses: perdives/wp-plugin-actions/get-plugin-version@v0.1
  with:
    plugin_file: 'my-plugin.php'
```

**Outputs:**
- `version`: Plugin version (e.g., `1.2.3`)
- `tag_name`: Git tag with v prefix (e.g., `v1.2.3`)

---

### 📦 setup-wp-cli
Install WP-CLI and the dist-archive package.

```yaml
- uses: perdives/wp-plugin-actions/setup-wp-cli@v0.1
```

---

### 🔨 build-plugin
Build production-ready WordPress plugin zip using WP-CLI dist-archive.

```yaml
- uses: perdives/wp-plugin-actions/build-plugin@v0.1
  with:
    plugin_slug: 'my-plugin'
    version: '1.2.3'  # Optional, will extract from plugin_file if not provided
    plugin_file: 'my-plugin.php'  # Required if version not provided
    create_checksum: 'true'  # Optional, default: true
```

**Outputs:**
- `zip_file`: Path to generated zip
- `checksum_file`: Path to SHA256 checksum

---

### ☁️ upload-to-release
Upload plugin assets to a GitHub release with optional retry logic.

```yaml
- uses: perdives/wp-plugin-actions/upload-to-release@v0.1
  with:
    tag: 'v1.2.3'
    files: 'my-plugin-*.zip my-plugin-*.sha256'
    wait_for_release: true  # Wait for Release Drafter
    max_retries: 12
    retry_interval: 5
```

**Outputs:**
- `uploaded`: `true` if files uploaded successfully

**When to use `wait_for_release`:**
- When using Release Drafter (creates releases asynchronously)
- When multiple workflows run in parallel
- To handle race conditions between build and release creation

---

### ✅ phpcs-check
Run PHP CodeSniffer checks. Automatically sets up PHP and installs dependencies.

```yaml
- uses: perdives/wp-plugin-actions/phpcs-check@v0.1
  with:
    command: 'composer check-cs'  # Optional, default: composer check-cs
    fail_on_error: 'true'  # Optional, default: true
    php_version: '7.4'  # Optional, default: 7.4
    composer_args: '--prefer-dist --no-progress'  # Optional
```

**Outputs:**
- `exit_code`: PHPCS exit code
- `has_errors`: `true` if issues found

**Note:** This action automatically runs `composer install` with the specified args.

---

### 🛡️ plugin-check
Run WordPress Plugin Check. Automatically sets up PHP and installs dependencies.

```yaml
- uses: perdives/wp-plugin-actions/plugin-check@v0.1
  with:
    fail_on_error: 'true'  # Optional, default: true
    php_version: '7.4'  # Optional, default: 7.4
```

**Outputs:**
- `has_errors`: `true` if issues found

**Note:** This action automatically runs `composer install --no-dev --optimize-autoloader`.

## Complete Example Workflow

```yaml
name: Build and Release

on:
  push:
    branches:
      - release

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Get version
        id: version
        uses: perdives/wp-plugin-actions/get-plugin-version@v0.1
        with:
          plugin_file: 'my-plugin.php'

      - name: Build plugin
        uses: perdives/wp-plugin-actions/build-plugin@v0.1
        with:
          plugin_file: 'my-plugin.php'
          generate-checksum: true

      - name: Upload to release
        uses: perdives/wp-plugin-actions/upload-to-release@v0.1
        with:
          tag: ${{ steps.version.outputs.tag }}
          files: 'my-plugin-*.zip my-plugin-*.sha256'
          wait_for_release: true
```

## Quick Links

- 📁 [Example Workflows](.github/workflows/) - Copy-paste examples

## Setup Instructions

### 1. Prerequisites

Your plugin repository needs:
- A main plugin PHP file with `Version:` header
- `.distignore` file (see [templates/](templates/))
- `.github/release-drafter.yml` config (see [templates/](templates/))
- Composer scripts for PHPCS:
  ```json
  {
    "scripts": {
      "check-cs": "phpcs",
      "fix-cs": "phpcbf"
    }
  }
  ```

### 2. Copy Template Files

```bash
# Copy .distignore to your plugin root
cp templates/.distignore /path/to/your-plugin/

# Copy release-drafter config
mkdir -p /path/to/your-plugin/.github
cp templates/release-drafter.yml /path/to/your-plugin/.github/
```

### 3. Create Workflows

See example workflows in [.github/workflows/](.github/workflows/):
- `example-complete-release.yml` - Build and create releases
- `example-pr-checks.yml` - Run checks on pull requests

Copy and customize for your plugin.

### 4. Create Release Branch

```bash
cd /path/to/your-plugin
git checkout -b release
git push -u origin release
```

## Usage Guide

### Making a Release

1. Update version in plugin file
2. Update changelog
3. Create PR from `main` → `release`
4. Checks run automatically
5. Merge PR
6. Release is created with built zip
7. Publish the release

### PR Checks

When you open a PR to `release`, the following checks run:
- PHPCS coding standards
- WordPress Plugin Check

### Deploy to WordPress.org

Combine our actions with the 10up WordPress.org deploy action:

```yaml
- uses: perdives/wp-plugin-actions/build-plugin@v0.1
  with:
    plugin_slug: 'my-plugin'
    version: ${{ steps.version.outputs.version }}

- uses: 10up/action-wordpress-plugin-deploy@stable
  with:
    generate-zip: false  # Already built
  env:
    SVN_USERNAME: ${{ secrets.SVN_USERNAME }}
    SVN_PASSWORD: ${{ secrets.SVN_PASSWORD }}
```

See [WORDPRESS-ORG-DEPLOY.md](WORDPRESS-ORG-DEPLOY.md) for complete guide.

## Repository Structure

```
wp-plugin-actions/
├── get-plugin-version/
│   └── action.yml
├── setup-wp-cli/
│   └── action.yml
├── build-plugin/
│   └── action.yml
├── upload-to-release/
│   ├── action.yml
│   └── README.md
├── phpcs-check/
│   └── action.yml
├── plugin-check/
│   └── action.yml
├── templates/
│   ├── .distignore
│   └── release-drafter.yml
├── .github/workflows/
│   ├── example-complete-release.yml
│   └── example-pr-checks.yml
└── README.md
```

## Versioning

Pin to specific versions for stability:

```yaml
# Recommended: Pin to a specific version
uses: perdives/wp-plugin-actions/build-plugin@v0.1.0

# Or use v0.1 to get latest v0.1.x updates
uses: perdives/wp-plugin-actions/build-plugin@v0.1

# Latest (less stable, for testing)
uses: perdives/wp-plugin-actions/build-plugin@main
```

## Contributing

Contributions welcome! Please open an issue or PR.

## License

GPL v2 or later

## Support

For issues or questions, please [open an issue](https://github.com/perdives/wp-plugin-actions/issues).
