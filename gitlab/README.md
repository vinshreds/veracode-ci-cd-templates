# GitLab CI - Veracode Integration Templates

This directory contains GitLab CI YAML templates for integrating Veracode security scanning into your CI/CD pipelines.

## Available Templates

- **pipeline-scan.yml** - Veracode Pipeline Scan (SAST) integration
- **sca-scan.yml** - Software Composition Analysis (SCA) scanning
- **policy-scan.yml** - Policy compliance scanning with JAR uploads
- **cli-examples.yml** - Various Veracode CLI operations

## Prerequisites

1. Veracode API credentials (API ID and API Key)
2. GitLab CI/CD variables configured
3. Veracode CLI or appropriate scanning tools

## Quick Start

1. Copy the desired template content to your `.gitlab-ci.yml` file
2. Configure the required CI/CD variables in your GitLab project
3. Customize the job blocks for your project needs
4. Commit and push to trigger the pipeline

## Authentication

Store your Veracode credentials as GitLab CI/CD variables:
- `VERACODE_API_ID` - Your Veracode API ID
- `VERACODE_API_KEY` - Your Veracode API Key (mark as masked and protected)

Navigate to: Project Settings → CI/CD → Variables → Add variable

See the main documentation for detailed setup instructions.
