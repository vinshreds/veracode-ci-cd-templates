# Azure Pipelines - Veracode Integration Templates

This directory contains Azure Pipelines YAML templates for integrating Veracode security scanning into your CI/CD workflows.

## Available Templates

- **pipeline-scan.yml** - Veracode Pipeline Scan (SAST) integration
- **sca-scan.yml** - Software Composition Analysis (SCA) scanning
- **policy-scan.yml** - Policy compliance scanning with JAR uploads
- **cli-examples.yml** - Various Veracode CLI operations

## Prerequisites

1. Veracode API credentials (API ID and API Key)
2. Azure DevOps service connection or pipeline variables configured
3. Veracode CLI or appropriate scanning tools

## Quick Start

1. Copy the desired template to your repository
2. Configure the required variables in your Azure Pipeline
3. Customize the template blocks for your project needs
4. Commit and run your pipeline

## Authentication

Store your Veracode credentials as Azure Pipeline variables:
- `VERACODE_API_ID` - Your Veracode API ID
- `VERACODE_API_KEY` - Your Veracode API Key (mark as secret)

See the main documentation for detailed setup instructions.
