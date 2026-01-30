# Socket Tier 1 Reachability Analysis

Reachability analysis identifies which vulnerabilities are actually reachable from your application code, reducing noise and helping prioritize fixes.

## Requirements

- Socket enterprise plan
- API token with the following scopes:
  - `uploaded-artifacts:create`
  - `full-scans:create`
  - `full-scans:list`
  - `repo:create`

## Setup

### 1. Add API Token to GitHub Secrets

1. Create token at [Socket Dashboard](https://socket.dev/dashboard) > Organization Settings > API Tokens
2. In GitHub: **Settings** > **Secrets and variables** > **Actions** > **New repository secret**
3. Name: `SOCKET_SECURITY_API_KEY`, Value: your token

### 2. Create Workflow

Add `.github/workflows/socket-security-scan.yml`:

```yaml
name: Socket Security Scan

on:
  schedule:
    - cron: "0 6 * * 1"  # Weekly Monday 6 AM UTC
  workflow_dispatch:

jobs:
  socket-security:
    runs-on: ubuntu-latest
    timeout-minutes: 60
    permissions:
      contents: read

    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - uses: actions/setup-node@v4
        with:
          node-version: "20"

      - uses: astral-sh/setup-uv@v4

      - name: Install Socket CLI
        run: pip install socketsecurity --upgrade

      - name: Run Socket Security Scan
        env:
          SOCKET_SECURITY_API_KEY: ${{ secrets.SOCKET_SECURITY_API_KEY }}
        run: |
          socketcli \
            --target-path "$GITHUB_WORKSPACE" \
            --repo "${GITHUB_REPOSITORY#*/}" \
            --reach \
            --reach-memory-limit 16384 \
            --reach-timeout 3600
```

### 3. Run

Go to **Actions** > **Socket Security Scan** > **Run workflow**

## Reachability Options

| Flag | Default | Description |
|------|---------|-------------|
| `--reach` | off | Enable reachability analysis |
| `--reach-memory-limit` | 8192 | Memory limit (MB) |
| `--reach-timeout` | 1800 | Timeout (seconds) |

## Supported Ecosystems

npm, PyPI, Maven, Go modules

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Multiple repositories found" | Use unique `--repo` name |
| API unauthorized | Verify token scopes |
| Timeout | Increase `--reach-timeout` |

## Links

- [Socket Dashboard](https://socket.dev/dashboard)
- [Socket Docs](https://docs.socket.dev)
- [Socket CLI](https://github.com/SocketDev/socket-python-cli)
