# Veracode CLI Command Reference

Complete reference of all Veracode CLI commands used in this repository's templates, verified against official documentation (January 2026).

## Table of Contents

- [Installation](#installation)
- [Authentication](#authentication)
- [Package Commands](#package-commands)
- [Static Analysis Commands](#static-analysis-commands)
- [SBOM Command](#sbom-command)
- [SCA Agent (NOT Veracode CLI)](#important-sca-is-not-part-of-veracode-cli)
- [Policy Commands](#policy-commands)
- [Sandbox Commands](#sandbox-commands)
- [Utility Commands](#utility-commands)
- [Output Formats](#output-formats)
- [Exit Codes](#exit-codes)

---

## Installation

### Install Veracode CLI

```bash
# Linux/macOS - Bash
curl -fsS https://tools.veracode.com/veracode-cli/install | sh
export PATH="$HOME/.veracode-cli:$PATH"

# Windows - PowerShell
Invoke-WebRequest -Uri "https://tools.veracode.com/veracode-cli/install" -OutFile install.ps1
.\install.ps1
$env:PATH += ";$env:USERPROFILE\.veracode-cli"
```

### Verify Installation

```bash
veracode --version
```

**Documentation:** [Install the Veracode CLI](https://docs.veracode.com/r/Install_the_Veracode_CLI)

---

## Authentication

### Environment Variables (Recommended for CI/CD)

```bash
export VERACODE_API_KEY_ID="your-api-id"
export VERACODE_API_KEY_SECRET="your-api-key"
```

### Test Authentication

```bash
veracode whoami
veracode whoami --format json
```

**Documentation:** [Veracode CLI Authentication](https://docs.veracode.com/r/Veracode_CLI)

---

## Package Commands

Auto-package applications for Static Analysis scans.

### veracode package discover

Detect build toolchains and generate configuration.

```bash
veracode package discover \
  --source <path> \
  [--verbose]
```

**Supported Build Tools:**
- MSBuild (.NET)
- Maven (Java)

**Outputs:** Creates `veracode.yml` configuration file

### veracode package

Create scannable artifacts from source code.

```bash
veracode package \
  --source <path-or-url> \
  --trust \
  [--output <directory>] \
  [--type <directory|repo>] \
  [--verbose] \
  [--strict] \
  [--help]
```

**Flags:**

| Flag | Short | Required | Description |
|------|-------|----------|-------------|
| `--source` | `-s` | ✅ | Source code location (local path or Git URL) |
| `--trust` | `-a` | ✅ | Acknowledge source is trustworthy |
| `--output` | `-o` | No | Output directory (default: current dir) |
| `--type` | `-t` | No | Source type: `directory` or `repo` (default: `directory`) |
| `--verbose` | `-v` | No | Show detailed output |
| `--strict` | | No | Return exit code 4 on build failures |
| `--help` | `-h` | No | Show help |

**Environment Variables:**

| Variable | Purpose | Example |
|----------|---------|---------|
| `SRCCLR_CUSTOM_MAVEN_COMMAND` | Custom Maven arguments | `"clean install -DskipTests"` |
| `VERACODE_PACKAGE_GOLANG_GENERATE` | Enable Go CGO generation | `true` |
| `SRCCLR_MSVC_CONFIGURATION` | .NET build config | `Release`, `Debug` |
| `SRCCLR_MSVC_PLATFORM` | .NET platform | `x64`, `x86`, `ARM64` |
| `SRCCLR_IOS_SCHEME` | iOS build scheme | Custom scheme name |
| `SRCCLR_IOS_DESTINATION` | iOS platform | `iOS`, `tvOS`, `watchOS` |
| `SRCCLR_IOS_CONFIGURATION` | iOS build type | `Debug`, `Release` |
| `SRCCLR_MAKE_TARGETS` | Make goals | `"all test"` |
| `SRCCLR_MAKE_JOBS` | Concurrent jobs | `4` |
| `SRCCLR_MAKE_FILES` | Makefile names | `"Makefile"` |
| `SRCCLR_BITBAKE_PACKAGE_NAMES` | BitBake recipes | `"recipe1 recipe2"` |

**Examples:**

```bash
# Local directory
veracode package --source ./my-app --output ./artifacts --trust

# Custom Maven command
export SRCCLR_CUSTOM_MAVEN_COMMAND="clean install -DskipTests"
veracode package --source ./java-app --output ./artifacts --trust

# .NET with Release configuration
export SRCCLR_MSVC_CONFIGURATION=Release
export SRCCLR_MSVC_PLATFORM=x64
veracode package --source ./dotnet-app --output ./artifacts --trust

# From Git repository
veracode package \
  --source https://github.com/veracode/verademo \
  --type repo \
  --output ./artifacts \
  --trust
```

**Supported Languages:** 25+ including Java, .NET, JavaScript, Python, Go, iOS, Android, C/C++

**Limitations:**
- SCA Agent-based Scan NOT supported
- Repository packaging uses temporary clone
- Archives must not be password-protected

**Documentation:** [veracode package](https://docs.veracode.com/r/veracode_package)

---

## Static Analysis Commands

### veracode static scan

Upload and scan for Policy Scan (comprehensive SAST).

```bash
veracode static scan \
  --source <file-or-path> \
  --app <app-name> \
  [--create-app] \
  [--sandbox-name <name>] \
  [--results-file <path>] \
  [--version <version>]
```

**Flags:**
- `--source` - Application file or package to scan
- `--app` - Application profile name
- `--create-app` - Create app profile if doesn't exist
- `--sandbox-name` - Scan in specific sandbox
- `--results-file` - Save results to file
- `--version` - Build version name

### veracode static scan info

Check scan status.

```bash
veracode static scan info \
  --app <app-name> \
  [--sandbox <name>]
```

### veracode static scan results

Download scan results.

```bash
veracode static scan results \
  --app <app-name> \
  --format <xml|json|csv> \
  --output <file>
```

### veracode static pipeline-scan

Fast Pipeline Scan for quick feedback.

```bash
veracode static pipeline-scan \
  --file <artifact> \
  [--fail-on-severity <levels>] \
  [--baseline-file <path>] \
  [--json-output-file <path>]
```

**Severity Levels:** `Very High`, `High`, `Medium`, `Low`, `Very Low`, `Informational`

### veracode static findings

Get detailed findings/flaws.

```bash
veracode static findings \
  --app <app-name> \
  --format <json|csv|table> \
  [--output <file>]
```

### veracode static flaw annotate

Annotate or mitigate flaws.

```bash
veracode static flaw annotate \
  --app <app-name> \
  --flaw-id <id> \
  --action <action> \
  --comment <text>
```

**Actions:** `Mitigated`, `False Positive`, `Approved Mitigation`

### veracode static app list

List all applications.

```bash
veracode static app list \
  --format <table|json>
```

### veracode static app info

Get application details.

```bash
veracode static app info \
  --app <app-name> \
  [--format json]
```

### veracode static app create

Create application profile.

```bash
veracode static app create \
  --name <app-name> \
  [--business-unit <name>] \
  [--criticality <level>]
```

**Criticality Levels:** `Very High`, `High`, `Medium`, `Low`, `Very Low`

### veracode static app delete

Delete application profile.

```bash
veracode static app delete \
  --app <app-name>
```

**Documentation:** [Static Analysis Commands](https://docs.veracode.com/r/CLI_Reference)

---

## SBOM Command

### veracode sbom

Generate Software Bill of Materials (SBOM) from containers, repositories, archives, or directories.

**⚠️ Note:** This is different from the SCA Agent's `./ci.sh sbom` command. The Veracode CLI `veracode sbom` can generate SBOMs from various sources, not just SCA scans.

```bash
veracode sbom \
  --type <image|repo|archive|directory> \
  --source <path-or-url> \
  [--format <format>] \
  [--output <file>]
```

**Flags:**

| Flag | Short | Description |
|------|-------|-------------|
| `--type` | | **Required**: Target type (`image`, `repo`, `archive`, `directory`) |
| `--source` | `-s` | **Required**: SBOM source location |
| `--format` | `-f` | Output format (see below) |
| `--output` | `-o` | Output file path (default: stdout) |
| `--help` | `-h` | Show help |

**Supported Formats:**
- `cyclonedx-json` (default)
- `cyclonedx-xml`
- `spdx-json`
- `spdx-tag-value`
- `github` (GitHub dependency snapshot)
- `table`
- `text`

**Supported Container Images:**
- Alpine Linux
- Amazon Linux
- CentOS
- Debian
- GitLab BusyBox
- Distroless
- Oracle Linux
- RHEL
- Ubuntu

**Examples:**

```bash
# Container image
veracode sbom --source alpine:latest --type image

# Local directory
veracode sbom --source ./my-app --type directory --format cyclonedx-json

# Git repository
veracode sbom --source https://github.com/veracode/veracode-sca --type repo

# Archive with custom output
veracode sbom \
  --source myapp.zip \
  --type archive \
  --format spdx-json \
  --output sbom.json
```

**Limitations:**
- Limited SBOM output for Gradle projects that haven't been built

**Documentation:** [veracode sbom](https://docs.veracode.com/r/veracode_sbom)

---

## IMPORTANT: SCA Is NOT Part of Veracode CLI

**⚠️ The Veracode CLI does NOT have `veracode sca` commands.**

Software Composition Analysis (SCA) uses the **SCA Agent** (`ci.sh`), which is a **separate tool** from the Veracode CLI.

### SCA Agent (ci.sh)

Download and use the SCA Agent for dependency scanning:

```bash
# Download SCA Agent
curl -sSL https://download.sourceclear.com/ci.sh -o ci.sh
chmod +x ci.sh

# Set API token
export SRCCLR_API_TOKEN="your-sca-token"

# Run SCA scan
./ci.sh scan \
  --app <app-name> \
  [--allow-dirty] \
  [--maven|--gradle|--npm|--yarn|--pip|--go] \
  [--threshold-cvss <score>] \
  [--threshold-cve-severity <level>] \
  [--json] \
  [--recursive]

# Generate SBOM
./ci.sh sbom \
  --format <cyclonedx|spdx> \
  --output <file>
```

**SCA Agent Parameters:**

| Parameter | Description |
|-----------|-------------|
| `--app` | Application name |
| `--allow-dirty` | Allow uncommitted changes |
| `--maven` | Scan Maven project |
| `--gradle` | Scan Gradle project |
| `--npm` | Scan npm project |
| `--yarn` | Scan Yarn project |
| `--pip` | Scan Python pip project |
| `--go` | Scan Go modules |
| `--ruby` | Scan Ruby gems |
| `--nuget` | Scan .NET NuGet |
| `--threshold-cvss` | Min CVSS score to fail (0.0-10.0) |
| `--threshold-cve-severity` | Min severity (`low`, `medium`, `high`, `critical`) |
| `--json` | Output JSON results |
| `--recursive` | Scan subdirectories |
| `--no-upload` | Don't upload to platform |
| `--quick` | Quick scan mode |

**SBOM Formats:**
- CycloneDX 1.6
- SPDX 2.3

**Key Differences:**
- ❌ `veracode sca scan` - Does NOT exist
- ❌ `veracode sca results` - Does NOT exist
- ❌ `veracode sca vulnerabilities` - Does NOT exist
- ✅ `./ci.sh scan` - Correct SCA Agent command
- ✅ `./ci.sh sbom` - Correct SCA Agent command

**Documentation:** [SCA Agent Documentation](https://docs.veracode.com/r/c_sc_what_is)

---

## Policy Commands

### veracode static policy

Check policy compliance.

```bash
veracode static policy \
  --app <app-name> \
  --format <json|xml> \
  --output <file>
```

**Policy Status Values:**
- `Pass` - Meets policy requirements
- `Did Not Pass` - Fails policy
- `Not Assessed` - No policy applied

### veracode policy

Download security policy.

```bash
veracode policy \
  --name <policy-name> \
  --output <file>
```

**Documentation:** [Policy Commands](https://docs.veracode.com/r/CLI_Reference)

---

## Sandbox Commands

### veracode static sandbox list

List sandboxes for an application.

```bash
veracode static sandbox list \
  --app <app-name>
```

### veracode static sandbox create

Create a new sandbox.

```bash
veracode static sandbox create \
  --app <app-name> \
  --name <sandbox-name>
```

### veracode static sandbox results

Get sandbox scan results.

```bash
veracode static sandbox results \
  --app <app-name> \
  --sandbox <sandbox-name> \
  --format <json|xml>
```

### veracode static sandbox promote

Promote sandbox scan to policy.

```bash
veracode static sandbox promote \
  --app <app-name> \
  --sandbox <sandbox-name>
```

### veracode static sandbox delete

Delete a sandbox.

```bash
veracode static sandbox delete \
  --app <app-name> \
  --sandbox <sandbox-name>
```

---

## Utility Commands

### veracode whoami

Display current user information.

```bash
veracode whoami
veracode whoami --format json
```

### veracode --version

Display CLI version.

```bash
veracode --version
```

### veracode --help

Show help for CLI or specific command.

```bash
veracode --help
veracode <command> --help
```

### veracode configure

Configure CLI settings.

```bash
veracode configure
```

### veracode cache

Clear CLI cache.

```bash
veracode cache clear
```

### veracode configure api

Make custom API calls.

```bash
veracode configure api \
  --endpoint <api-path> \
  --method <GET|POST|PUT|DELETE>
```

---

## Output Formats

Most commands support multiple output formats:

| Format | Description | Use Case |
|--------|-------------|----------|
| `table` | Human-readable table | Terminal display, debugging |
| `json` | Structured JSON | Parsing, automation, CI/CD |
| `xml` | XML format | Detailed reports, legacy tools |
| `csv` | Comma-separated | Spreadsheets, reporting |

**Example:**
```bash
# Table format (default)
veracode static app list --format table

# JSON for parsing
veracode static app list --format json | jq '.applications'

# CSV for Excel
veracode static findings --app my-app --format csv --output findings.csv
```

---

## Exit Codes

| Code | Meaning | Usage |
|------|---------|-------|
| `0` | Success | Operation completed successfully |
| `1` | General error | Command failed |
| `4` | Build failure | Package command build failed (with `--strict`) |
| Other | Specific errors | See command documentation |

**Example:**
```bash
veracode package --source . --output artifacts --trust --strict || {
  EXIT_CODE=$?
  if [ $EXIT_CODE -eq 4 ]; then
    echo "Build failure detected"
  fi
  exit $EXIT_CODE
}
```

---

## Common Usage Patterns

### Complete Workflow: Package → Upload → Check Results

```bash
# 1. Package application
veracode package --source . --output artifacts --trust --verbose

# 2. Upload for Policy Scan
ARTIFACT=$(find artifacts -name "*.zip" -o -name "*.jar" | head -1)
veracode static scan \
  --source "$ARTIFACT" \
  --app my-app \
  --create-app \
  --results-file results.json

# 3. Check policy compliance
veracode static policy \
  --app my-app \
  --format json \
  --output policy.json

# 4. Parse result
COMPLIANCE=$(jq -r '.policy_compliance_status' policy.json)
if [[ "$COMPLIANCE" == "Pass" ]]; then
  echo "✓ Passed policy"
else
  echo "✗ Failed policy"
  exit 1
fi
```

### Pipeline Scan with Baseline

```bash
# First scan - create baseline
veracode static pipeline-scan \
  --file target/myapp.jar \
  --json-output-file baseline.json

# Subsequent scans - compare to baseline
veracode static pipeline-scan \
  --file target/myapp.jar \
  --baseline-file baseline.json \
  --json-output-file results.json
```

### SCA with Threshold

```bash
# Fail build on CVSS >= 7.0
./ci.sh scan \
  --app my-app \
  --threshold-cvss 7.0 \
  --threshold-cve-severity high \
  --json > sca-results.json
```

---

## Documentation Sources

All commands verified against official Veracode documentation (January 2026):

- **Main CLI Docs:** https://docs.veracode.com/r/Veracode_CLI
- **CLI Reference:** https://docs.veracode.com/r/CLI_Reference
- **Package Command:** https://docs.veracode.com/r/veracode_package
- **Autopackaging:** https://docs.veracode.com/r/About_auto_packaging
- **CLI Updates:** https://docs.veracode.com/updates/r/Veracode_CLI_Updates
- **SCA Agent:** https://docs.veracode.com/r/c_sc_what_is
- **API Documentation:** https://docs.veracode.com/r/c_about_veracode_apis

---

## Notes

- Commands shown use Linux/macOS syntax - adjust for Windows PowerShell
- Always use `--verbose` when debugging
- Store API credentials securely (environment variables, CI/CD secrets)
- Test locally before CI/CD integration
- Check official documentation for latest updates

---

**Last Updated:** January 2026
**Source:** Official Veracode CLI Documentation
