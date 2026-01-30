# Veracode Integration Setup Guide

This guide walks you through setting up Veracode security scanning in your CI/CD pipelines.

## Table of Contents

1. [Veracode Account Setup](#veracode-account-setup)
2. [API Credentials](#api-credentials)
3. [CI/CD Platform Configuration](#cicd-platform-configuration)
4. [First Scan](#first-scan)
5. [Troubleshooting](#troubleshooting)

## Veracode Account Setup

### Prerequisites

1. Active Veracode account
2. Access to Veracode Platform
3. Appropriate scanning permissions

### Getting Started

1. Log into the [Veracode Platform](https://analysiscenter.veracode.com/)
2. Navigate to your account settings
3. Ensure you have API access enabled

## API Credentials

### Generating API Credentials

1. Log into Veracode Platform
2. Go to **Account** → **API Credentials**
3. Click **Generate API Credentials**
4. Save your **API ID** and **API Key** securely

**Important**: Store these credentials securely - you won't be able to view the API Key again.

### Credential Format

```
API ID: xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
API Key: yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy
```

## CI/CD Platform Configuration

### Azure Pipelines

1. Navigate to your Azure DevOps project
2. Go to **Pipelines** → **Library** → **Variable groups**
3. Create a new variable group or select an existing one
4. Add variables:
   - `VERACODE_API_ID` = your API ID
   - `VERACODE_API_KEY` = your API Key (click the lock icon to make it secret)
5. Save the variable group
6. Link the variable group to your pipeline

### GitHub Actions

1. Navigate to your GitHub repository
2. Go to **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret**
4. Add secrets:
   - `VERACODE_API_ID` = your API ID
   - `VERACODE_API_KEY` = your API Key
5. Save each secret

### GitLab CI

1. Navigate to your GitLab project
2. Go to **Settings** → **CI/CD** → **Variables**
3. Click **Add variable**
4. Add variables:
   - `VERACODE_API_ID` = your API ID (protected and masked)
   - `VERACODE_API_KEY` = your API Key (protected and masked)
5. Save each variable

### Bitbucket Pipelines

1. Navigate to your Bitbucket repository
2. Go to **Repository settings** → **Pipelines** → **Repository variables**
3. Click **Add variable**
4. Add variables:
   - `VERACODE_API_ID` = your API ID
   - `VERACODE_API_KEY` = your API Key (check "Secured")
5. Save each variable

## First Scan

### Choose Your Scan Type

1. **Pipeline Scan** (fastest) - Quick SAST feedback in minutes
2. **SCA Scan** - Identify vulnerable dependencies
3. **Policy Scan** - Comprehensive analysis with policy compliance

### Using a Template

1. Choose your CI/CD platform directory
2. Select the appropriate template file
3. Copy the template content
4. Paste into your pipeline configuration
5. Customize the following:
   - Application name
   - Build artifact path
   - Scan parameters
6. Commit and run your pipeline

### Example: GitHub Actions Pipeline Scan

```yaml
name: Veracode Pipeline Scan

on: [push]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Build application
        run: |
          # Your build commands here
          mvn clean package

      - name: Veracode Pipeline Scan
        uses: veracode/Veracode-pipeline-scan-action@v1.0.0
        with:
          vid: ${{ secrets.VERACODE_API_ID }}
          vkey: ${{ secrets.VERACODE_API_KEY }}
          file: target/myapp.jar
```

## Troubleshooting

### Common Issues

#### Authentication Errors

**Problem**: "Authentication failed" or "Invalid credentials"

**Solutions**:
- Verify API ID and API Key are correct
- Check that credentials are properly stored in CI/CD secrets
- Ensure API credentials haven't expired
- Verify you have API access enabled in Veracode

#### File Not Found

**Problem**: "Could not find file to scan"

**Solutions**:
- Verify the build artifact path is correct
- Ensure the build step completes successfully before scanning
- Check file permissions
- Verify the artifact exists in the expected location

#### Scan Failures

**Problem**: Scan starts but fails to complete

**Solutions**:
- Check Veracode Platform for detailed error messages
- Verify application is in a supported format (JAR, WAR, ZIP, etc.)
- Ensure file size is within Veracode limits
- Review scan logs for specific errors

### Getting Help

- **Veracode Support**: [Veracode Help Center](https://help.veracode.com/)
- **Documentation**: [Veracode Docs](https://docs.veracode.com/)
- **Community**: [Veracode Community](https://community.veracode.com/)

## Next Steps

1. Review [authentication.md](authentication.md) for advanced credential management
2. Read [best-practices.md](best-practices.md) for optimization tips
3. Customize templates for your specific use case
4. Set up scan result notifications
5. Integrate scan results into your workflow

---

**Need more help?** Check the platform-specific README files in each template directory.
