# Upload to GitHub Release

Upload plugin assets to a GitHub release with optional retry logic for Release Drafter coordination.

## Features

- Upload files to existing GitHub releases
- Optional retry logic to wait for Release Drafter to create the release
- Configurable retry attempts and intervals
- Supports glob patterns for file matching
- Returns success status as output

## Usage

### Basic Usage

Upload files to an existing release:

```yaml
- uses: perdives/wp-plugin-actions/upload-to-release@v1
  with:
    tag: 'v1.2.3'
    files: 'my-plugin-1.2.3.zip my-plugin-1.2.3.zip.sha256'
```

### With Release Drafter

When using Release Drafter, enable retry logic to wait for the draft release to be created:

```yaml
- uses: perdives/wp-plugin-actions/upload-to-release@v1
  with:
    tag: 'v1.2.3'
    files: 'my-plugin-*.zip my-plugin-*.sha256'
    wait_for_release: true
    max_retries: 12
    retry_interval: 5
```

### Complete Workflow Example

```yaml
name: Build and Release

on:
  push:
    branches: [release]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Get version
        id: version
        uses: perdives/wp-plugin-actions/get-plugin-version@v1
        with:
          plugin-file: 'my-plugin.php'

      - name: Build plugin
        id: build
        uses: perdives/wp-plugin-actions/build-plugin@v1
        with:
          plugin-file: 'my-plugin.php'
          generate-checksum: true

      - name: Upload to release
        uses: perdives/wp-plugin-actions/upload-to-release@v1
        with:
          tag: ${{ steps.version.outputs.tag }}
          files: ${{ steps.build.outputs.zip_file }} ${{ steps.build.outputs.checksum_file }}
          wait_for_release: true
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `tag` | Yes | - | Release tag name (e.g., `v1.2.3`) |
| `files` | Yes | - | Files to upload (space-separated, supports globs) |
| `wait_for_release` | No | `false` | Wait for release to exist before uploading |
| `max_retries` | No | `12` | Maximum retry attempts when waiting |
| `retry_interval` | No | `5` | Seconds between retry attempts |
| `github_token` | No | `${{ github.token }}` | GitHub token for authentication |

## Outputs

| Output | Description |
|--------|-------------|
| `uploaded` | `true` if files uploaded successfully, `false` otherwise |

## When to Use `wait_for_release`

Enable `wait_for_release: true` when:

- Using Release Drafter that creates releases asynchronously
- Multiple workflows run in parallel
- The release might not exist immediately when upload starts

The action will retry up to `max_retries` times, waiting `retry_interval` seconds between attempts.

## File Patterns

The `files` input supports:

- **Exact paths**: `my-plugin-1.2.3.zip`
- **Glob patterns**: `my-plugin-*.zip *.sha256`
- **Multiple files**: Space-separated list

## Permissions

Requires `contents: write` permission:

```yaml
permissions:
  contents: write
```

## Error Handling

The action will fail if:

- Release doesn't exist and `wait_for_release` is `false`
- Release not found after `max_retries` attempts
- Upload fails for any reason

## License

MIT
