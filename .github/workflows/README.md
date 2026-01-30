# GitHub Actions - Veracode Integration Templates

This directory contains GitHub Actions workflow templates for integrating Veracode security scanning into your CI/CD pipelines.

## Available Templates

- **pipeline-scan.yml** - Veracode Pipeline Scan (SAST) integration
- **sca-scan.yml** - Software Composition Analysis (SCA) scanning
- **policy-scan.yml** - Policy compliance scanning with JAR uploads
- **cli-examples.yml** - Various Veracode CLI operations

## Prerequisites

1. Veracode API credentials (API ID and API Key)
2. GitHub repository secrets configured
3. Veracode CLI or appropriate scanning tools

## Quick Start

1. Copy the desired template to `.github/workflows/` in your repository
2. Configure the required secrets in your GitHub repository settings
3. Customize the workflow blocks for your project needs
4. Commit and push to trigger the workflow

## Authentication

Store your Veracode credentials as GitHub Secrets:
- `VERACODE_API_ID` - Your Veracode API ID
- `VERACODE_API_KEY` - Your Veracode API Key

Navigate to: Repository Settings → Secrets and variables → Actions → New repository secret

See the main documentation for detailed setup instructions.
