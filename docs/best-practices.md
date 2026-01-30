# Veracode CI/CD Best Practices

This guide provides recommendations for effective and efficient Veracode security scanning in your CI/CD pipelines.

## Table of Contents

1. [Scan Strategy](#scan-strategy)
2. [Pipeline Integration](#pipeline-integration)
3. [Performance Optimization](#performance-optimization)
4. [Results Management](#results-management)
5. [Security Considerations](#security-considerations)

## Scan Strategy

### Choose the Right Scan Type

#### Pipeline Scan (SAST)
**Best for**: Fast feedback during development

✅ **Use when**:
- Running on every commit or pull request
- Need results in minutes, not hours
- Want to fail builds on critical issues
- Scanning feature branches

❌ **Don't use for**:
- Policy compliance reporting
- Comprehensive security audits
- Official security sign-off

#### SCA (Software Composition Analysis)
**Best for**: Identifying vulnerable dependencies

✅ **Use when**:
- Tracking open-source libraries
- Need license compliance information
- Want to monitor dependency changes
- Running scheduled dependency audits

❌ **Don't use for**:
- First-party code analysis
- Performance testing

#### Policy Scan
**Best for**: Comprehensive compliance scanning

✅ **Use when**:
- Need official security compliance
- Running on release candidates
- Require detailed flaw information
- Need policy violation reports

❌ **Don't use for**:
- Every commit (too slow)
- Feature branch development

### Recommended Workflow

```
Feature Branch    → Pipeline Scan (fast feedback)
      ↓
Pull Request      → Pipeline Scan + SCA
      ↓
Main Branch       → Pipeline Scan + SCA
      ↓
Release Candidate → Policy Scan (comprehensive)
      ↓
Production        → Verified clean scans
```

## Pipeline Integration

### When to Scan

#### Every Commit (Aggressive)
```yaml
on:
  push:
    branches: [ main, develop, feature/* ]
```

**Pros**: Maximum security coverage
**Cons**: Slower pipelines, higher API usage

#### Pull Requests (Balanced)
```yaml
on:
  pull_request:
    types: [ opened, synchronize, reopened ]
```

**Pros**: Catch issues before merge
**Cons**: May miss issues in direct commits

#### Scheduled (Periodic)
```yaml
on:
  schedule:
    - cron: '0 2 * * 1'  # Weekly on Monday at 2 AM
```

**Pros**: Lower pipeline impact
**Cons**: Delayed issue detection

#### Recommended Approach
```yaml
on:
  pull_request:
    types: [ opened, synchronize ]
  push:
    branches: [ main, develop ]
  schedule:
    - cron: '0 2 * * 1'  # Weekly policy scan
```

### Fail Build on Issues

#### Severity-Based Failure

**Azure Pipelines**:
```yaml
- script: |
    # Fail on high/critical vulnerabilities
    veracode-pipeline-scan --fail_on_severity="High,Critical"
  displayName: 'Veracode Pipeline Scan'
```

**GitHub Actions**:
```yaml
- name: Veracode Pipeline Scan
  uses: veracode/veracode-pipeline-scan-action@v1
  with:
    fail_build: true
    severity_threshold: "High"
```

#### CWE-Based Failure

Fail on specific vulnerability types:
```bash
--fail_on_cwe="79,89,90"  # XSS, SQL Injection, LDAP Injection
```

### Artifact Handling

#### Cache Build Artifacts

Save time by caching dependencies:

```yaml
# GitHub Actions
- uses: actions/cache@v3
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
```

#### Upload Scan Artifacts

Preserve scan results:

```yaml
# GitHub Actions
- uses: actions/upload-artifact@v3
  if: always()
  with:
    name: veracode-results
    path: results.json
```

## Performance Optimization

### Optimize Build Process

#### Build Once, Scan Multiple Times

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build application
        run: mvn clean package
      - uses: actions/upload-artifact@v3
        with:
          name: app-jar
          path: target/*.jar

  pipeline-scan:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v3
      - name: Veracode Pipeline Scan
        # Scan the pre-built artifact

  sca-scan:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v3
      - name: Veracode SCA Scan
        # SCA scan the same artifact
```

### Parallel Scanning

Run different scan types in parallel:

```yaml
jobs:
  security-scans:
    strategy:
      matrix:
        scan-type: [pipeline, sca]
    runs-on: ubuntu-latest
    steps:
      - name: Run ${{ matrix.scan-type }} scan
        # Run scans in parallel
```

### Incremental Scanning

Only scan changed code (where supported):

```bash
# Use git diff to determine what changed
git diff --name-only ${{ github.event.before }} ${{ github.sha }}
```

## Results Management

### Store Results

#### Upload to Artifact Storage

```yaml
- uses: actions/upload-artifact@v3
  with:
    name: veracode-scan-results
    path: |
      results.json
      results.pdf
    retention-days: 90
```

#### Commit Results to Repository

```yaml
- name: Commit scan results
  run: |
    git config user.name "Veracode Bot"
    git config user.email "bot@example.com"
    git add security-reports/
    git commit -m "Update Veracode scan results"
    git push
```

### Notifications

#### Slack Notifications

```yaml
- name: Notify Slack
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK }}
    payload: |
      {
        "text": "Veracode scan failed on ${{ github.repository }}"
      }
```

#### Email Notifications

Configure in your CI/CD platform to send emails on scan failures.

### Issue Tracking Integration

#### Create GitHub Issues for Findings

```yaml
- name: Create issue for high severity findings
  if: failure()
  uses: actions/github-script@v6
  with:
    script: |
      github.rest.issues.create({
        owner: context.repo.owner,
        repo: context.repo.repo,
        title: 'High severity security findings detected',
        body: 'Veracode scan found critical issues. Review results.',
        labels: ['security', 'high-priority']
      })
```

#### Jira Integration

Use Veracode's Jira integration or custom scripts to create tickets.

## Security Considerations

### Secure Credential Management

- ✅ Use CI/CD secrets management
- ✅ Rotate credentials regularly (every 90 days)
- ✅ Use separate credentials per environment
- ✅ Enable audit logging
- ❌ Never log credentials
- ❌ Don't commit credentials to git

### Least Privilege

Grant minimum required permissions:
- Read-only API access for scanning
- Write access only for result uploads
- Separate accounts for different teams

### Scan Sensitive Applications

#### Private Repositories Only

For sensitive code:
- Use private repositories
- Restrict pipeline access
- Enable branch protection
- Require code review

#### Data Handling

- Don't include sensitive data in build artifacts
- Sanitize logs and outputs
- Use `.veracodeignore` to exclude sensitive files
- Review what's being uploaded to Veracode

### Compliance

#### Audit Trail

Maintain records of:
- All security scans performed
- Results and findings
- Remediation actions taken
- Policy exceptions granted

#### Reporting

Generate regular reports:
- Weekly scan summaries
- Monthly vulnerability trends
- Quarterly compliance reports
- Annual security audits

## Common Pitfalls

### ❌ Don't: Ignore All Findings

```yaml
# Bad: Never do this
- name: Veracode Scan
  continue-on-error: true  # Ignores all failures
```

### ✅ Do: Selectively Handle Findings

```yaml
# Good: Fail on critical, warn on others
- name: Veracode Scan
  run: veracode-scan --fail_on_severity="Critical"
```

### ❌ Don't: Scan Too Frequently

```yaml
# Bad: Scanning every file change
on: [push, pull_request, fork, watch, star]
```

### ✅ Do: Scan Strategically

```yaml
# Good: Scan on meaningful events
on:
  pull_request:
  push:
    branches: [main]
```

### ❌ Don't: Upload Incomplete Builds

```yaml
# Bad: Scanning before build completes
- name: Veracode Scan
  run: veracode-scan target/app.jar
- name: Build App
  run: mvn package
```

### ✅ Do: Scan After Successful Build

```yaml
# Good: Build first, then scan
- name: Build App
  run: mvn package
- name: Veracode Scan
  if: success()
  run: veracode-scan target/app.jar
```

## Metrics and KPIs

Track these metrics to measure security posture:

- **Scan Coverage**: % of builds scanned
- **Time to Remediation**: Days from finding to fix
- **Vulnerability Density**: Issues per 1000 lines of code
- **False Positive Rate**: % of findings marked as FP
- **Policy Compliance**: % of scans passing policy

## Resources

- [Veracode Documentation](https://docs.veracode.com/)
- [Veracode Community](https://community.veracode.com/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [CWE Top 25](https://cwe.mitre.org/top25/)

---

**Remember**: Security scanning is most effective when it's part of your regular development workflow, not an afterthought.
