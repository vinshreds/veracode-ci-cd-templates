# Veracode Authentication Guide

This guide covers various methods for securely authenticating with Veracode in your CI/CD pipelines.

## Table of Contents

1. [API Credentials Overview](#api-credentials-overview)
2. [SCA Agent Token (SRCCLR_API_TOKEN)](#sca-agent-token-srcclr_api_token)
3. [Credential Storage Best Practices](#credential-storage-best-practices)
4. [Platform-Specific Configuration](#platform-specific-configuration)
5. [Alternative Authentication Methods](#alternative-authentication-methods)
6. [Security Considerations](#security-considerations)

## API Credentials Overview

Veracode uses different authentication methods depending on the scanning type:

### Static Analysis API Credentials

For **Static Analysis** (Pipeline Scan, Policy Scan, CLI operations), Veracode API credentials consist of two components:

- **API ID**: A unique identifier (UUID format)
- **API Key**: A secret key (64-character string)

#### Generating API Credentials

1. Log into [Veracode Platform](https://analysiscenter.veracode.com/)
2. Navigate to **Account** → **My Profile** → **API Credentials**
3. Click **Generate API Credentials**
4. Securely store both the API ID and API Key immediately

**Warning**: The API Key is only displayed once. Store it securely.

## SCA Agent Token (SRCCLR_API_TOKEN)

For **Software Composition Analysis** (SCA Agent-based scans), you need a different authentication token.

### What is SRCCLR_API_TOKEN?

The SRCCLR_API_TOKEN is an agent authentication token that acts as a password to connect your CI/CD pipeline to Veracode during SCA scans. This token is separate from the Veracode API credentials used for static analysis.

### Obtaining Your SCA Agent Token

1. Log into [Veracode Platform](https://analysiscenter.veracode.com/)
2. Navigate to **Scans & Analysis** → **Software Composition Analysis**
3. Select **Agent-Based Scan**
4. Select a **Workspace** (or create one if needed)
5. Select **Agents** → **Actions** → **Create**
6. Select any option from the Integration Options section
7. Click **Create Agent & Generate Token**
8. **Copy the token value immediately** - it will only be displayed once

**Critical**: If you close the page without copying the token, it disappears permanently and you must regenerate it.

### Regenerating SCA Tokens

If your token is compromised or lost:

1. Navigate to **Scans & Analysis** → **Software Composition Analysis** → **Agent-Based Scan**
2. Select the **Agents** page (workspace or organization level)
3. Select the agent
4. Click **Regenerate Token**
5. Copy the newly displayed token

**Warning**: Regenerating invalidates the old token. Any CI/CD pipelines using the old token will fail.

### Required Permissions

- **Organization-level agent**: Security Lead role
- **Workspace agent**: Security Lead, Workspace Administrator, Workspace Editor, or Submitter role

### Regional Configuration

For non-commercial regions, set additional environment variables alongside SRCCLR_API_TOKEN:

- **European Region**: `SRCCLR_REGION=ER`
- **US Federal Region**: `SRCCLR_REGION=FED`

### Security Best Practices for SCA Tokens

- ✅ **DO**: Store as a secret in your CI/CD platform
- ✅ **DO**: Copy the token immediately when generated
- ✅ **DO**: Regenerate if compromise is suspected
- ✅ **DO**: Use workspace-level tokens when possible (limited scope)
- ❌ **DON'T**: Share tokens across teams unnecessarily
- ❌ **DON'T**: Commit tokens to version control
- ❌ **DON'T**: Log or echo tokens in pipeline output
- ❌ **DON'T**: Use organization-level tokens unless required

**Security Note**: Token compromise is serious. With workspace agents, attackers can taint scan data. With organization agents, they can scan into any accessible workspace.

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

**For Static Analysis:**
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

**For SCA Scans:**
```yaml
variables:
  - group: 'veracode-sca-credentials'

steps:
  - task: Bash@3
    inputs:
      targetType: 'inline'
      script: |
        curl -sSL https://download.sourceclear.com/ci.sh -o ci.sh
        chmod +x ci.sh
        ./ci.sh scan --app my-app
    env:
      SRCCLR_API_TOKEN: $(SRCCLR_API_TOKEN)
```

Setup:
1. Pipelines → Library → Variable groups
2. Create group named `veracode-sca-credentials`
3. Add `SRCCLR_API_TOKEN` (obtained from Veracode Platform → SCA → Agent-Based Scan)
4. Lock the token variable (mark as secret)

#### Option 2: Pipeline Variables

**For Static Analysis:**
```yaml
variables:
  VERACODE_API_ID: $(veracode-api-id)
  VERACODE_API_KEY: $(veracode-api-key)
```

**For SCA Scans:**
```yaml
variables:
  SRCCLR_API_TOKEN: $(srcclr-api-token)
```

Setup:
1. Edit Pipeline → Variables
2. Add variables with the lock icon for secrets

### GitHub Actions

#### Using Repository Secrets

**For Static Analysis:**
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
3. Add `VERACODE_API_ID` and `VERACODE_API_KEY`

**For SCA Scans:**
```yaml
jobs:
  sca-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Veracode SCA Scan
        env:
          SRCCLR_API_TOKEN: ${{ secrets.SRCCLR_API_TOKEN }}
        run: |
          curl -sSL https://download.sourceclear.com/ci.sh -o ci.sh
          chmod +x ci.sh
          ./ci.sh scan --app my-app
```

Setup:
1. Repository Settings → Secrets and variables → Actions
2. New repository secret
3. Add `SRCCLR_API_TOKEN` (obtained from Veracode Platform → SCA → Agent-Based Scan → Create Agent)

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

**For Static Analysis:**
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

**For SCA Scans:**
```yaml
sca-scan:
  script:
    - curl -sSL https://download.sourceclear.com/ci.sh -o ci.sh
    - chmod +x ci.sh
    - ./ci.sh scan --app my-app
  variables:
    SRCCLR_API_TOKEN: $SRCCLR_API_TOKEN
```

Setup:
1. Settings → CI/CD → Variables → Expand
2. Add variable
   - Key: `SRCCLR_API_TOKEN`
   - Value: Your SCA agent token (from Veracode Platform → SCA → Agent-Based Scan)
   - Flags: Protected, Masked

#### Group-Level Variables (for multiple projects)

1. Group Settings → CI/CD → Variables
2. Add variables at group level (`VERACODE_API_ID`, `VERACODE_API_KEY`, `SRCCLR_API_TOKEN`)
3. All projects in group inherit these variables

### Bitbucket Pipelines

#### Repository Variables

**For Static Analysis:**
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

**For SCA Scans:**
```yaml
pipelines:
  default:
    - step:
        name: SCA Scan
        script:
          - curl -sSL https://download.sourceclear.com/ci.sh -o ci.sh
          - chmod +x ci.sh
          - ./ci.sh scan --app my-app
```

Setup:
1. Repository settings → Pipelines → Repository variables
2. Add variable
   - Name: `SRCCLR_API_TOKEN`
   - Value: Your SCA agent token (from Veracode Platform → SCA → Agent-Based Scan)
   - Secured: Yes

#### Workspace Variables (for multiple repositories)

1. Workspace settings → Workspace variables
2. Variables available to all repositories in workspace
3. Add `VERACODE_API_ID`, `VERACODE_API_KEY`, and `SRCCLR_API_TOKEN` as needed

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

### Static Analysis API Documentation
- [Veracode API Documentation](https://docs.veracode.com/r/Veracode_APIs)
- [API Credentials Management](https://docs.veracode.com/r/c_api_credentials3)

### SCA Agent Token Documentation
- [Manage Agents and Scans](https://docs.veracode.com/r/Manage_agents_and_scans)
- [Create Veracode SCA Agents](https://docs.veracode.com/r/Create_Veracode_SCA_Agents)
- [Regenerate SCA Agent Tokens](https://docs.veracode.com/r/Regenerate_Veracode_SCA_Agent_Tokens)
- [SCA Agent Environment Variables](https://docs.veracode.com/r/Veracode_SCA_Agent_Environment_Variables)
- [Integrate SCA with CI Projects](https://docs.veracode.com/r/Integrate_Veracode_SCA_Agent_Based_Scanning_with_Your_CI_Projects)

### General Resources
- [Veracode Security Best Practices](https://docs.veracode.com/r/Veracode_Best_Practices)
- [Agent-Based Scans](https://docs.veracode.com/r/Agent_Based_Scans)

---

**Security Reminder**: Never commit credentials or tokens to version control, even in private repositories. Always use your CI/CD platform's secrets management.
