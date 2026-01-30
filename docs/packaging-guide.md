# Veracode Packaging Guide

This guide covers the `veracode package` command for preparing applications for Static Analysis (SAST) scans.

## Table of Contents

- [Overview](#overview)
- [When to Use Packaging](#when-to-use-packaging)
- [Package vs. Manual Upload](#package-vs-manual-upload)
- [Command Reference](#command-reference)
- [Language-Specific Examples](#language-specific-examples)
- [Environment Variables](#environment-variables)
- [Limitations](#limitations)
- [Troubleshooting](#troubleshooting)

---

## Overview

The `veracode package` command automates the creation of scannable artifacts from your source code. It handles:

- **Auto-detection** of build tools and dependencies
- **Compilation** of code into scannable artifacts
- **Packaging** into proper archive format
- **Validation** of artifact completeness

### Supported Scan Types

- ✅ **Upload and Scan** (Policy Scan)
- ✅ **Pipeline Scan**
- ❌ **SCA Agent-based Scan** (NOT supported - use SCA Agent directly)

---

## When to Use Packaging

**Use `veracode package` when:**
- You have source code that needs compilation
- You want automated build process
- You need consistent packaging across environments
- You're integrating into CI/CD pipelines

**Don't use `veracode package` when:**
- You already have pre-built artifacts (JAR, DLL, etc.)
- Running SCA scans (use SCA Agent instead)
- Uploading pre-packaged archives

---

## Package vs. Manual Upload

| Aspect | veracode package | Manual Build + Upload |
|--------|------------------|----------------------|
| **Automation** | Fully automated | Manual steps required |
| **Consistency** | Same process every time | Varies by developer |
| **CI/CD Integration** | Easy integration | Requires custom scripts |
| **Build Control** | Limited customization | Full control |
| **Best For** | Standard builds | Complex/custom builds |

---

## Command Reference

### Basic Syntax

```bash
veracode package --source <path-or-url> --trust [flags]
```

### Flags

| Flag | Short | Required | Description |
|------|-------|----------|-------------|
| `--source` | `-s` | ✅ Yes | Source code location (local path or Git URL) |
| `--trust` | `-a` | ✅ Yes | Acknowledge source is trustworthy (first use) |
| `--output` | `-o` | No | Output directory (default: current directory) |
| `--type` | `-t` | No | Source type: `directory` or `repo` (default: `directory`) |
| `--verbose` | `-v` | No | Show detailed build output |
| `--strict` | | No | Return exit code 4 on build failures |
| `--help` | `-h` | No | Show command help |

### Package Discover Command

**Run this FIRST** to detect toolchains and generate configuration:

```bash
veracode package discover --source <path>
```

**What it does:**
- Detects build tools (MSBuild, Maven, etc.)
- Identifies required upgrades
- Generates `veracode.yml` configuration
- Shows detected modules and dependencies

**Currently Supports:**
- MSBuild (.NET projects)
- Maven (Java projects)

---

## Language-Specific Examples

### Java (Maven)

```bash
# Discover build configuration
veracode package discover --source ./my-java-app

# Package with default Maven goals
veracode package \
  --source ./my-java-app \
  --output ./artifacts \
  --trust

# Package with custom Maven arguments
export SRCCLR_CUSTOM_MAVEN_COMMAND="clean install -DskipTests -P production"
veracode package --source ./my-java-app --output ./artifacts --trust
```

### Java (Gradle)

```bash
# Package Gradle project
veracode package \
  --source ./my-gradle-app \
  --output ./artifacts \
  --trust \
  --verbose
```

### .NET (C#)

```bash
# Discover MSBuild projects
veracode package discover --source ./my-dotnet-app

# Package with specific configuration
export SRCCLR_MSVC_CONFIGURATION=Release
export SRCCLR_MSVC_PLATFORM=x64
veracode package \
  --source ./my-dotnet-app \
  --output ./artifacts \
  --trust
```

### Python

```bash
# Package Python project
veracode package \
  --source ./my-python-app \
  --output ./artifacts \
  --trust

# For projects without package manager (generates "no-pm" artifact)
veracode package \
  --source ./my-simple-python-app \
  --output ./artifacts \
  --trust
```

### JavaScript/Node.js

```bash
# Package Node.js application
veracode package \
  --source ./my-node-app \
  --output ./artifacts \
  --trust
```

### Go

```bash
# Enable CGO code generation
export VERACODE_PACKAGE_GOLANG_GENERATE=true

# Package Go application (excludes nested modules)
veracode package \
  --source ./my-go-app \
  --output ./artifacts \
  --trust
```

### iOS/Swift

```bash
# Package iOS project with custom scheme
export SRCCLR_IOS_SCHEME=MyApp
export SRCCLR_IOS_DESTINATION=iOS
export SRCCLR_IOS_CONFIGURATION=Release

veracode package \
  --source ./my-ios-app \
  --output ./artifacts \
  --trust
```

### C/C++ (Make)

```bash
# Configure Make build
export SRCCLR_MAKE_TARGETS="all"
export SRCCLR_MAKE_JOBS=4
export SRCCLR_MAKE_FILES="Makefile"

veracode package \
  --source ./my-cpp-app \
  --output ./artifacts \
  --trust
```

### Android

```bash
# Package Android application
veracode package \
  --source ./my-android-app \
  --output ./artifacts \
  --trust
```

### Git Repository Packaging

```bash
# Package from public GitHub repository
veracode package \
  --source https://github.com/veracode/verademo \
  --type repo \
  --output ./artifacts \
  --trust

# Package from specific branch
veracode package \
  --source https://github.com/myorg/myapp \
  --type repo \
  --output ./artifacts \
  --trust
```

---

## Environment Variables

### General

| Variable | Purpose | Example |
|----------|---------|---------|
| `SRCCLR_CUSTOM_MAVEN_COMMAND` | Custom Maven arguments | `"clean install -DskipTests"` |
| `VERACODE_PACKAGE_GOLANG_GENERATE` | Enable Go CGO generation | `true` |

### iOS Projects

| Variable | Purpose | Values |
|----------|---------|--------|
| `SRCCLR_IOS_SCHEME` | Build scheme | Custom scheme name |
| `SRCCLR_IOS_DESTINATION` | Target platform | `iOS`, `tvOS`, `watchOS`, `visionOS` |
| `SRCCLR_IOS_CONFIGURATION` | Build type | `Debug`, `Release` |

### C/C++ Projects

| Variable | Purpose | Example |
|----------|---------|---------|
| `SRCCLR_MAKE_TARGETS` | Make goals to execute | `"all test"` |
| `SRCCLR_MAKE_JOBS` | Concurrent jobs | `4` |
| `SRCCLR_MAKE_FILES` | Makefile names | `"Makefile GNUmakefile"` |
| `SRCCLR_MAKE_CLEAN_TARGET` | Clean target | `"clean"` |
| `SRCCLR_MSVC_PREPROCESS` | Preprocessed source vs debug | `true`/`false` |
| `SRCCLR_MSVC_CONFIGURATION` | Build configuration | `Release`, `Debug` |
| `SRCCLR_MSVC_PLATFORM` | Target platform | `x64`, `x86`, `ARM64` |
| `SRCCLR_TEMP_PREPROCESS_DIR` | Temp directory | `/tmp/preprocess` |
| `SRCCLR_SEARCH_CPP_SUBDIRECTORIES` | Search subdirs | `true`/`false` |
| `SRCCLR_MAKE_WINDOWS` | Enable Make on Windows | `true` |

### Other Languages

| Variable | Purpose | Example |
|----------|---------|---------|
| `SRCCLR_BITBAKE_PACKAGE_NAMES` | BitBake recipes | `"recipe1 recipe2"` |

---

## Limitations

### General Limitations

1. **SCA Agent-based Scan NOT Supported**
   - Use SCA Agent (`ci.sh`) for dependency scanning
   - `veracode package` is for SAST only

2. **Repository Packaging Behavior**
   - Clones to temporary location
   - Deletes after packaging
   - No persistent local copy

3. **Discover Command Limitations**
   - Currently supports **MSBuild (.NET)** and **Maven** only
   - Other languages detected but not configured by discover

4. **Archive Requirements**
   - Must not be password-protected
   - No obfuscated binaries
   - No installer packages (RPM, InstallShield)
   - UTF-8 encoding required
   - Archives of archives not supported (except JAR/WAR/EAR)

5. **File Size Limits**
   - Check Veracode documentation for current limits
   - Varies by license tier

### Language-Specific Limitations

#### Go
- Nested Go modules are excluded from packaging
- Must enable CGO generation explicitly if needed

#### Python/JavaScript/PHP
- Projects without package managers generate "no-pm" artifacts
- Includes all code files

#### C/C++
- Requires debug symbols (PDB files) for proper results
- Must compile successfully before packaging

#### .NET
- Requires MSBuild in PATH
- Must target supported .NET versions

---

## Troubleshooting

### Package Command Not Found

```bash
# Ensure CLI is installed
curl -fsS https://tools.veracode.com/veracode-cli/install | sh

# Add to PATH
export PATH="$HOME/.veracode-cli:$PATH"

# Verify installation
veracode --version
```

### Build Failures

```bash
# Use --verbose to see detailed output
veracode package \
  --source ./my-app \
  --output ./artifacts \
  --trust \
  --verbose

# Use --strict to get exit code 4 on failures
veracode package \
  --source ./my-app \
  --output ./artifacts \
  --trust \
  --strict
```

### No Artifacts Generated

**Check:**
1. ✅ Source path is correct
2. ✅ Build tools installed (Maven, Gradle, MSBuild, etc.)
3. ✅ Project compiles successfully locally
4. ✅ Required dependencies available
5. ✅ Run `veracode package discover` first

### Discover Shows No Toolchains

**Reasons:**
- Discover currently only supports MSBuild and Maven
- Your language may not be supported by discover yet
- Still safe to run `veracode package` directly

### Custom Build Requirements

**If you need custom build steps:**
1. Build manually first
2. Package the artifacts
3. Upload directly (don't use `veracode package`)

**Example:**
```bash
# Build manually with custom steps
./custom-build.sh

# Then upload the artifact
veracode static scan \
  --source ./build/output/myapp.jar \
  --results-file results.json
```

### Repository Packaging Issues

**Common problems:**
- Private repositories require authentication (not fully supported)
- Large repositories may timeout
- Submodules may not be included

**Solutions:**
- Clone repository locally first
- Use `--type directory` with local clone
- Ensure proper `.gitmodules` configuration

---

## Best Practices

1. **Always Run Discover First** (for .NET/Maven)
   ```bash
   veracode package discover --source ./my-app
   # Review veracode.yml
   veracode package --source ./my-app --output ./artifacts --trust
   ```

2. **Use Verbose Mode During Setup**
   ```bash
   veracode package --source ./my-app --verbose --trust
   ```

3. **Specify Output Directory**
   ```bash
   veracode package \
     --source ./my-app \
     --output ./build/veracode-artifacts \
     --trust
   ```

4. **Use Environment Variables for Consistency**
   ```bash
   export SRCCLR_CUSTOM_MAVEN_COMMAND="clean install -DskipTests"
   veracode package --source ./my-app --output ./artifacts --trust
   ```

5. **Test Locally Before CI/CD**
   - Verify packaging works on dev machine
   - Check artifact size and contents
   - Validate artifact uploads successfully

6. **Document Custom Configuration**
   - Create `.veracode-config.sh` with environment variables
   - Add to repository for team consistency
   - Include in CI/CD pipeline

---

## Additional Resources

- [Veracode CLI Documentation](https://docs.veracode.com/r/Veracode_CLI)
- [veracode package Command Reference](https://docs.veracode.com/r/veracode_package)
- [About Autopackaging](https://docs.veracode.com/r/About_auto_packaging)
- [Compilation and Packaging](https://docs.veracode.com/r/compilation_packaging)
- [CLI Updates](https://docs.veracode.com/updates/r/Veracode_CLI_Updates)

---

**Note:** This guide is based on Veracode CLI documentation as of January 2026. Always consult the official [Veracode documentation](https://docs.veracode.com/r/Veracode_CLI) for the most current information.
