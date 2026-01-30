# Bitbucket Pipelines - Veracode Integration Templates

This directory contains Bitbucket Pipelines YAML templates for integrating Veracode security scanning into your CI/CD workflows.

## Available Templates

- **pipeline-scan.yml** - Veracode Pipeline Scan (SAST) integration
- **sca-scan.yml** - Software Composition Analysis (SCA) scanning
- **policy-scan.yml** - Policy compliance scanning with JAR uploads
- **cli-examples.yml** - Various Veracode CLI operations

## Prerequisites

1. Veracode API credentials (API ID and API Key)
2. Bitbucket repository variables configured
3. Veracode CLI or appropriate scanning tools

## Quick Start

1. Copy the desired template content to your `bitbucket-pipelines.yml` file
2. Configure the required repository variables in Bitbucket
3. Customize the pipeline blocks for your project needs
4. Commit and push to trigger the pipeline

## Authentication

Store your Veracode credentials as Bitbucket repository variables:
- `VERACODE_API_ID` - Your Veracode API ID
- `VERACODE_API_KEY` - Your Veracode API Key (mark as secured)

Navigate to: Repository Settings → Pipelines → Repository variables → Add variable

See the main documentation for detailed setup instructions.
