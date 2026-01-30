# Veracode CI/CD Templates

A comprehensive collection of reusable YAML templates for integrating Veracode security scanning into your CI/CD pipelines across multiple platforms.

## Overview

This repository provides ready-to-use templates for integrating Veracode's security scanning tools into your continuous integration and deployment workflows. Each template is designed as a modular "block" that can be copied, customized, and integrated into your existing pipelines.

## Supported Platforms

- **Azure Pipelines** - Microsoft Azure DevOps
- **GitHub Actions** - GitHub's native CI/CD
- **GitLab CI** - GitLab's continuous integration
- **Bitbucket Pipelines** - Atlassian Bitbucket CI/CD

## Veracode Scanning Types

### Pipeline Scan (SAST)
Fast static analysis security testing integrated directly into your pipeline. Provides quick feedback on security vulnerabilities in your code.

### SCA (Software Composition Analysis)
Identifies open-source vulnerabilities and license risks in your dependencies.

### Policy Scan
Comprehensive security scanning with policy compliance checks. Uploads your application (typically JAR files) to Veracode for thorough analysis.

### Packaging
Automated application packaging using the Veracode CLI `package` command. Auto-detects build tools, compiles code, and creates scannable artifacts for Static Analysis.

### CLI Operations
Templates for various Veracode CLI commands including:
- Results retrieval
- Scan status checks
- Report generation
- Custom scanning operations

## Quick Start

1. **Choose your CI/CD platform** - Navigate to the appropriate directory
2. **Select a template** - Pick the Veracode scanning type you need
3. **Configure credentials** - Set up Veracode API credentials in your CI/CD platform
4. **Copy and customize** - Integrate the template blocks into your workflow
5. **Run and verify** - Test your pipeline and review scan results

## Repository Structure

```
veracode-ci-cd-templates/
├── azure-pipelines/       # Azure Pipelines templates
├── .github/workflows/     # GitHub Actions templates
├── gitlab/                # GitLab CI templates
├── bitbucket/             # Bitbucket Pipelines templates
├── docs/                  # Additional documentation
│   ├── setup-guide.md     # Initial setup instructions
│   ├── authentication.md  # Credential configuration
│   └── best-practices.md  # Security scanning tips
└── README.md              # This file
```

## Prerequisites

- Veracode account with API access
- Veracode API ID and API Key
- CI/CD platform account (Azure DevOps, GitHub, GitLab, or Bitbucket)
- Application artifact(s) to scan (JAR, WAR, ZIP, etc.)

## Authentication

All templates require Veracode API credentials:
- **API ID** - Your Veracode API identifier
- **API Key** - Your Veracode API secret key

Store these securely in your CI/CD platform's secrets management:
- Azure Pipelines: Pipeline Variables (secure)
- GitHub Actions: Repository Secrets
- GitLab CI: CI/CD Variables (masked)
- Bitbucket: Repository Variables (secured)

See [docs/authentication.md](docs/authentication.md) for detailed setup instructions.

## Documentation

- [Setup Guide](docs/setup-guide.md) - Get started with Veracode integration
- [Authentication](docs/authentication.md) - Configure API credentials
- [Packaging Guide](docs/packaging-guide.md) - Using veracode package command
- [Best Practices](docs/best-practices.md) - Tips for effective security scanning

## Usage Notes

- Templates are designed to be **modular** - copy only what you need
- All templates include comments explaining configuration options
- Customize scan parameters based on your application type and requirements
- Review Veracode documentation for detailed scanning options

## Contributing

Contributions are welcome! If you have improvements or additional templates:
1. Fork this repository
2. Create a feature branch
3. Add your templates with documentation
4. Submit a pull request

## Resources

- [Veracode Documentation](https://docs.veracode.com/)
- [Veracode CLI Documentation](https://docs.veracode.com/r/Veracode_CLI)
- [Veracode API Documentation](https://docs.veracode.com/r/c_about_veracode_apis)

## License

MIT License - Feel free to use these templates in your projects.

## Support

For Veracode-specific questions, consult the [Veracode Help Center](https://help.veracode.com/).

For issues with these templates, please open a GitHub issue.

---

**Note**: These templates are provided as examples. Always review and test templates in a non-production environment before deploying to production pipelines.
