# serverless-sam-deployment-tools-2025

Tools to automate AWS SAM deployments, fix Globals/config errors, troubleshoot API Gateway method issues.

## Overview

This repository provides a collection of tools and scripts designed to streamline AWS SAM (Serverless Application Model) deployments and resolve common configuration issues. The tools help developers automate deployment workflows, manage global configuration changes, and troubleshoot API Gateway integration problems that frequently arise in serverless architectures.

### Key Features

- **Automated SAM Deployment Pipeline**: Scripts to orchestrate build, package, and deploy stages
- **Globals Configuration Management**: Tools to detect and fix breaking changes in SAM template Globals sections
- **API Gateway Method Resolution**: Utilities to identify and resolve duplicate method errors
- **Comprehensive Troubleshooting Guides**: Step-by-step documentation for common SAM deployment issues
- **Configuration Validation**: Pre-deployment checks to catch errors before they reach AWS

## Example Fixes

### Globals Change Management

When modifying the `Globals` section in SAM templates, certain changes can cause deployment failures. This tool helps manage these transitions:

```yaml
# Before (original Globals)
Globals:
  Function:
    Timeout: 30
    Runtime: python3.9
    
# After (modified Globals)
Globals:
  Function:
    Timeout: 60
    Runtime: python3.11
    MemorySize: 512
```

**Common Issues:**
- Runtime version changes requiring code compatibility updates
- Timeout modifications affecting downstream services
- Memory size adjustments impacting Lambda pricing

**Tool Solution:**
```bash
# Validate Globals changes before deployment
python scripts/validate_globals.py --template template.yaml --previous-version v1.0.0

# Apply Globals changes with rollback support
python scripts/apply_globals.py --template template.yaml --rollback-on-error
```

### Duplicate Method Error Resolution

API Gateway duplicate method errors occur when the same HTTP method is defined multiple times for a resource:

**Error Message:**
```
An error occurred (BadRequestException) when calling the CreateDeployment operation: 
Active stages pointing to this deployment must be moved or deleted
```

**Common Causes:**
- Multiple Lambda functions mapping to the same API path and method
- Conflicting event definitions in SAM template
- API Gateway resource policy conflicts

**Detection Tool:**
```bash
# Scan SAM template for duplicate API methods
python scripts/detect_duplicate_methods.py --template template.yaml

# Output example:
# WARNING: Duplicate method detected:
#   Resource: /users
#   Method: POST
#   Functions: CreateUserFunction, RegisterUserFunction
```

**Resolution Steps:**
1. Identify conflicting resources using the detection tool
2. Consolidate or differentiate API paths
3. Update SAM template event definitions
4. Redeploy with `--force` flag if necessary

```bash
# Force deployment after fixing conflicts
sam deploy --template template.yaml --force-upload --no-fail-on-empty-changeset
```

## Troubleshooting Guides

### 1. Deployment Hangs or Times Out

**Symptoms:**
- CloudFormation stack shows `UPDATE_IN_PROGRESS` indefinitely
- Lambda functions not updating despite successful build

**Solutions:**
- Check CloudWatch Logs for Lambda timeout issues
- Verify IAM permissions for CloudFormation service role
- Increase timeout values in SAM template
- Use `sam deploy --no-execute-changeset` to review changes first

### 2. Globals Section Not Applied

**Symptoms:**
- Individual function properties override Globals unexpectedly
- Runtime or environment variables not inherited

**Solutions:**
- Ensure Globals section is at root level of template
- Verify indentation in YAML (use spaces, not tabs)
- Check for function-level overrides that supersede Globals
- Run validation: `sam validate --template template.yaml`

### 3. API Gateway Integration Errors

**Symptoms:**
- 502 Bad Gateway errors
- Lambda function not invoked
- CORS preflight failures

**Solutions:**
- Verify Lambda integration permissions
- Check API Gateway method response configuration
- Enable CORS in Globals or per-function events
- Review Lambda proxy integration settings

### 4. Stack Rollback on Deploy

**Symptoms:**
- Stack enters `UPDATE_ROLLBACK_IN_PROGRESS` state
- New resources fail to create

**Solutions:**
- Check CloudFormation events for specific error messages
- Verify resource limits (Lambda concurrent executions, API Gateway quotas)
- Ensure S3 bucket for deployment artifacts exists and is accessible
- Use `sam logs` to debug Lambda initialization errors

### 5. Environment Variable Issues

**Symptoms:**
- Lambda functions cannot access expected environment variables
- Secrets or configuration values missing

**Solutions:**
- Verify environment variables defined in Globals or function properties
- Check SSM Parameter Store or Secrets Manager permissions
- Use `sam local invoke` to test locally
- Validate parameter resolution: `sam build && sam deploy --parameter-overrides`

## References to AWS SAM Issues (October 2025)

Based on reported issues and community discussions from October 2025:

### Issue #3847: Globals Runtime Changes Breaking Deployments
- **Status**: Resolved in SAM CLI v1.102.0
- **Impact**: Runtime version changes in Globals section caused all functions to redeploy
- **Workaround**: Use per-function runtime specification during transition period
- **Fix**: SAM CLI now supports gradual runtime migration

### Issue #3852: API Gateway Duplicate Method Detection
- **Status**: Under investigation
- **Impact**: SAM does not validate duplicate HTTP methods before deployment
- **Workaround**: Use custom pre-deployment validation scripts (included in this repo)
- **Expected Fix**: SAM CLI v1.105.0 (planned for November 2025)

### Issue #3861: CloudFormation Stack Update Timeout
- **Status**: Acknowledged
- **Impact**: Large SAM applications (50+ Lambda functions) timeout during stack updates
- **Workaround**: Split applications into multiple stacks using nested templates
- **Investigation**: AWS CloudFormation team investigating root cause

### Issue #3868: Globals Environment Variables Override
- **Status**: Resolved in SAM CLI v1.103.0
- **Impact**: Function-level environment variables not properly merging with Globals
- **Fix**: Environment variable inheritance logic corrected
- **Migration**: Update SAM CLI to latest version

### Issue #3875: API Gateway CORS Configuration Inconsistency
- **Status**: Open
- **Impact**: CORS settings in Globals not applied to all API Gateway methods
- **Workaround**: Define CORS explicitly in each API event
- **Tracking**: Community PR in review

## Installation

```bash
# Clone the repository
git clone https://github.com/sandeepkatakam21/serverless-sam-deployment-tools-2025.git
cd serverless-sam-deployment-tools-2025

# Install dependencies
pip install -r requirements.txt

# Run setup script
python setup.py install
```

## Usage

### Basic Deployment Validation

```bash
# Validate SAM template
python scripts/validate_template.py --template template.yaml

# Check for common issues
python scripts/pre_deploy_check.py --template template.yaml --verbose
```

### Automated Deployment

```bash
# Build and deploy with automated checks
python scripts/auto_deploy.py --template template.yaml --stack-name my-stack --region us-east-1
```

### Troubleshooting Commands

```bash
# Analyze failed deployment
python scripts/analyze_failure.py --stack-name my-stack --region us-east-1

# Generate troubleshooting report
python scripts/generate_report.py --stack-name my-stack --output report.html
```

## Contributing

Contributions are welcome! Please feel free to submit issues or pull requests.

## License

MIT License - see LICENSE file for details

## Additional Resources

- [AWS SAM Official Documentation](https://docs.aws.amazon.com/serverless-application-model/)
- [AWS SAM GitHub Repository](https://github.com/aws/serverless-application-model)
- [AWS SAM CLI GitHub Issues](https://github.com/aws/aws-sam-cli/issues)
- [API Gateway Developer Guide](https://docs.aws.amazon.com/apigateway/)
- [CloudFormation Best Practices](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/best-practices.html)
