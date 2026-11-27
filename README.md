Probe Action
==

A GitHub Action that runs [probe](https://github.com/mozership/probe) workflows for testing, monitoring, and automation tasks.

Features
--

- **Easy Integration**: Simple setup with your existing GitHub workflows
- **Automatic Download**: Automatically downloads and sets up the probe binary
- **Checksum Verification**: Verifies the downloaded archive against the release's `checksums.txt` (SHA-256) before extracting it
- **Archive Caching**: Caches the probe release archive via `actions/cache` (keyed by resolved version, OS, and arch) to skip re-downloads on subsequent runs; a restored archive is re-verified against the release's `checksums.txt` on every run and re-downloaded if it does not match
- **Linux and macOS Support**: Runs on Ubuntu and macOS runners (x86_64 and ARM64)
- **Flexible Options**: Configurable verbose output and response time display
- **Rich Testing**: Supports HTTP, SSH, Database, Browser, Shell, SMTP, and IMAP actions
- **Rate Limit Safe**: Uses GitHub token for API calls to avoid rate limiting

Usage
--


```yaml
name: API Testing
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run probe tests
        uses: mozership/probe-action@v1.4.1
        with:
          path: 'tests/api-test.yml'

      # Multiple paths example
      - name: Run multiple probe tests
        uses: mozership/probe-action@v1.4.1
        with:
          paths: |
            tests/api-test.yml
            tests/db-test.yml
            tests/integration-test.yml
          options: '--verbose --rt'

      # Working directory example
      - name: Run probe from specific directory
        uses: mozership/probe-action@v1.4.1
        with:
          path: 'workflow.yml'
          workdir: './tests'
          options: '--verbose'
```

Inputs
--

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `path` | Path to the probe workflow YAML file | No* | - |
| `paths` | Multiple paths to workflow YAML files (newline-separated multiline string) | No* | - |
| `version` | Version of probe to use (a release tag such as `v1.38.1`, or `latest`; a bare `1.38.1` is normalized to `v1.38.1`) | No | `latest` |
| `before-run` | Checks to run on each workflow file before running any of them, separated by newlines or commas: `check`, `fmt` | No | - |
| `options` | Command line options for probe when running the workflows (e.g., "--verbose --rt") | No | `` |
| `workdir` | Working directory to change to before running probe | No | `` |
| `install-only` | Install probe and add it to `PATH` without running a workflow (true/false/yes/1) | No | `false` |
| `action-debug` | Enable action debug output (true/false/yes/1) | No | `false` |
| `cache` | Cache the probe binary across runs via `actions/cache` (set to `false` to disable) | No | `true` |
| `github-token` | Token used for GitHub API calls (resolving the latest version) to avoid rate limiting | No | `${{ github.token }}` |

*Either `path` or `paths` must be provided, unless `install-only` is set.

Outputs
--

| Output | Description |
|--------|-------------|
| `probe-path` | Absolute path of the probe binary this action installed |

Checking Workflows Before Running
--

`before-run` checks each workflow file of `path` or `paths` before running any of them:

| Value | Runs |
|-------|------|
| `check` | `probe check <file>`, which finds what is wrong or weak in the workflow without running it |
| `fmt` | `probe fmt --check <file>`, which fails when the file is not laid out the way `probe fmt` writes it, and changes nothing |

The checks run in the order given, and the step fails at the first that fails, before any workflow runs. `options` applies only to running the workflows. `check` and `fmt` need a version of probe that has them, such as `v1.44.0`:

```yaml
- uses: mozership/probe-action@v1.4.1
  with:
    version: v1.44.0
    path: tests/api-test.yml
    before-run: |
      check
      fmt
    options: --report github-summary
```

The values can also be given on one line, as `before-run: check, fmt`. For any other subcommand, use `install-only`, or run `probe` in a later step: it is on `PATH` after the action has run.

Installing Probe Only
--

With `install-only`, the action downloads and verifies probe, adds it to `PATH`, and runs no workflow. Later steps of the job run probe themselves:

```yaml
- uses: mozership/probe-action@v1.4.1
  with:
    install-only: true
    version: v1.38.1

- run: probe check tests/api-test.yml
```

`install-only` cannot be combined with `path`, `paths` or `before-run`. The binary is also on `PATH`, and in the `probe-path` output, after a run that executes workflows.

Sample Probe Workflow
--

Create a probe workflow file (e.g., `tests/api-test.yml`):

```yaml
name: API Health Check
description: Test API endpoints

jobs:
  - name: API Tests
    steps:
      - name: Test API Health
        uses: http
        with:
          get: https://api.example.com/health
        test: "res.code == 200"

      - name: Test API Authentication
        uses: http
        with:
          post: https://api.example.com/auth
          json:
            username: "test"
            password: "secret"
        test: "res.code == 200 && res.body.token != ''"
```

Supported Probe Actions
--

The probe tool supports various built-in actions:

- **HTTP**: REST API testing, authentication, file uploads
- **GRPC**: GRPC testing, authentication
- **Database**: SQL queries and connection testing
- **Shell**: Command execution and script running
- **SSH**: Remote command execution and file operations
- **SMTP**: Email sending functionality
- **IMAP**: Email reading and processing
- **Embedded**: Embedded job execution
- **Browser**: Web UI automation and testing

For detailed probe syntax and examples, see the [probe documentation](https://github.com/mozership/probe).

Platform Support
--

Linux and macOS are supported, from the release archives of probe for each. Windows is not.

- **Linux** (`ubuntu-latest`, `ubuntu-24.04-arm`): x86_64, ARM64
- **macOS** (`macos-latest`): ARM64. On an Intel Mac the x86_64 archive is used, but it is not tested here.

On macOS, probe can run what needs a Mac, such as [probe-ios](https://github.com/mozership/probe-ios) against an iOS simulator.

Debugging
--

Enable verbose output and response times for detailed information:

```yaml
- uses: mozership/probe-action@v1.4.1
  with:
    path: 'tests/debug-test.yml'
    options: '--verbose --rt'
    action-debug: 'true'  # Shows action internal debug information
```

Contributing
--

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

License
--

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
