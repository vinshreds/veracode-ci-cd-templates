# Veracode Authentication Guide

This guide covers various methods for securely authenticating with Veracode in your CI/CD pipelines.

## Table of Contents

1. [API Credentials Overview](#api-credentials-overview)
2. [Credential Storage Best Practices](#credential-storage-best-practices)
3. [Platform-Specific Configuration](#platform-specific-configuration)
4. [Alternative Authentication Methods](#alternative-authentication-methods)
5. [Security Considerations](#security-considerations)

## API Credentials Overview

Veracode API credentials consist of two components:

- **API ID**: A unique identifier (UUID format)
- **API Key**: A secret key (64-character string)

### Generating Credentials

1. Log into [Veracode Platform](https://analysiscenter.veracode.com/)
2. Navigate to **Account** → **My Profile** → **API Credentials**
3. Click **Generate API Credentials**
4. Securely store both the API ID and API Key immediately

**Warning**: The API Key is only displayed once. Store it securely.

## Credential Storage Best Practices

### General Guidelines

- ✅ **DO**: Use your CI/CD platform's secrets management
- ✅ **DO**: Rotate credentials periodically
- ✅ **DO**: Use different credentials for different environments
- ✅ **DO**: Limit credential scope to minimum required permissions
- ❌ **DON'T**: Hardcode credentials in YAML files
- ❌ **DON'T**: Commit credentials to version control
- ❌ **DON'T**: Share credentials across teams unnecessarily
- ❌ **DON'T**: Log or echo credentials in pipeline output

### Environment-Specific Credentials

Consider using different Veracode API credentials for:
- Development pipelines
- Staging/QA pipelines
- Production pipelines

This provides better audit trails and limits blast radius if credentials are compromised.

## Platform-Specific Configuration

### Azure Pipelines

#### Option 1: Variable Groups (Recommended)

```yaml
variables:
  - group: 'veracode-credentials'

steps:
  - task: Bash@3
    inputs:
      targetType: 'inline'
      script: |
        echo "Using API ID: $(VERACODE_API_ID)"
        # Run Veracode scan
    env:
      VERACODE_API_ID: $(VERACODE_API_ID)
      VERACODE_API_KEY: $(VERACODE_API_KEY)
```

Setup:
1. Pipelines → Library → Variable groups
2. Create group named `veracode-credentials`
3. Add `VERACODE_API_ID` and `VERACODE_API_KEY`
4. Lock the API Key variable

#### Option 2: Pipeline Variables

```yaml
variables:
  VERACODE_API_ID: $(veracode-api-id)
  VERACODE_API_KEY: $(veracode-api-key)
```

Setup:
1. Edit Pipeline → Variables
2. Add variables with the lock icon for secrets

### GitHub Actions

#### Using Repository Secrets

```yaml
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - name: Veracode Scan
        env:
          VERACODE_API_ID: ${{ secrets.VERACODE_API_ID }}
          VERACODE_API_KEY: ${{ secrets.VERACODE_API_KEY }}
        run: |
          # Your scan command
```

Setup:
1. Repository Settings → Secrets and variables → Actions
2. New repository secret
3. Add both credentials

#### Using Environment Secrets (for different environments)

```yaml
jobs:
  production-scan:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Veracode Scan
        env:
          VERACODE_API_ID: ${{ secrets.VERACODE_API_ID }}
          VERACODE_API_KEY: ${{ secrets.VERACODE_API_KEY }}
        run: |
          # Your scan command
```

Setup:
1. Repository Settings → Environments → New environment
2. Add environment-specific secrets

### GitLab CI

#### Using CI/CD Variables

```yaml
veracode-scan:
  script:
    - echo "Running scan with API ID $VERACODE_API_ID"
    # Scan commands here
  variables:
    VERACODE_API_ID: $VERACODE_API_ID
    VERACODE_API_KEY: $VERACODE_API_KEY
```

Setup:
1. Settings → CI/CD → Variables → Expand
2. Add variable
   - Key: `VERACODE_API_ID`
   - Value: Your API ID
   - Flags: Protected, Masked
3. Repeat for `VERACODE_API_KEY`

#### Group-Level Variables (for multiple projects)

1. Group Settings → CI/CD → Variables
2. Add variables at group level
3. All projects in group inherit these variables

### Bitbucket Pipelines

#### Repository Variables

```yaml
pipelines:
  default:
    - step:
        name: Veracode Scan
        script:
          - echo "Scanning with Veracode"
          # Scan commands using $VERACODE_API_ID and $VERACODE_API_KEY
```

Setup:
1. Repository settings → Pipelines → Repository variables
2. Add variable
   - Name: `VERACODE_API_ID`
   - Value: Your API ID
   - Secured: No
3. Add variable
   - Name: `VERACODE_API_KEY`
   - Value: Your API Key
   - Secured: Yes

#### Workspace Variables (for multiple repositories)

1. Workspace settings → Workspace variables
2. Variables available to all repositories in workspace

## Alternative Authentication Methods

### Veracode Credentials File

Some Veracode tools support a credentials file (`~/.veracode/credentials`):

```ini
[default]
veracode_api_key_id = your_api_id
veracode_api_key_secret = your_api_key
```

**Usage in CI/CD**:

```yaml
- name: Setup Veracode credentials
  run: |
    mkdir -p ~/.veracode
    echo "[default]" > ~/.veracode/credentials
    echo "veracode_api_key_id = ${{ secrets.VERACODE_API_ID }}" >> ~/.veracode/credentials
    echo "veracode_api_key_secret = ${{ secrets.VERACODE_API_KEY }}" >> ~/.veracode/credentials
    chmod 600 ~/.veracode/credentials
```

### Environment Variables

Most Veracode CLI tools recognize these environment variables:

```bash
export VERACODE_API_KEY_ID="your_api_id"
export VERACODE_API_KEY_SECRET="your_api_key"
```

## Security Considerations

### Credential Rotation

Rotate your Veracode API credentials regularly:

1. Generate new credentials in Veracode Platform
2. Update CI/CD secrets
3. Test pipelines with new credentials
4. Revoke old credentials

### Access Control

- Limit who can view/edit CI/CD secrets
- Use role-based access control (RBAC)
- Audit credential access regularly
- Enable MFA on Veracode account

### Monitoring

- Monitor Veracode API usage
- Set up alerts for authentication failures
- Review API access logs periodically
- Track which pipelines use which credentials

### Compliance

Ensure your credential management meets:
- Your organization's security policies
- Industry compliance requirements (SOC 2, ISO 27001, etc.)
- Data residency requirements

## Troubleshooting

### Invalid Credentials

**Symptoms**: "401 Unauthorized" or "Authentication failed"

**Checklist**:
- ✓ API ID is correct (UUID format)
- ✓ API Key is correct (64 characters)
- ✓ No extra spaces or newlines in secrets
- ✓ Credentials haven't been revoked
- ✓ API access is enabled on Veracode account

### Credentials Not Found

**Symptoms**: Empty environment variables or "credentials not set"

**Checklist**:
- ✓ Secrets are defined in correct scope (repo/environment/group)
- ✓ Variable names match exactly (case-sensitive)
- ✓ Pipeline has permission to access secrets
- ✓ Environment protection rules aren't blocking access

### Permission Errors

**Symptoms**: "403 Forbidden" or "Insufficient permissions"

**Solutions**:
- Verify API credentials have required Veracode roles
- Check application-level permissions
- Ensure team/business unit access is configured

## Resources

- [Veracode API Documentation](https://docs.veracode.com/r/Veracode_APIs)
- [API Credentials Management](https://docs.veracode.com/r/c_api_credentials3)
- [Veracode Security Best Practices](https://docs.veracode.com/r/Veracode_Best_Practices)

---

**Security Reminder**: Never commit credentials to version control, even in private repositories.
