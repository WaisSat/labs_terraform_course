# Student Setup Guide

This guide explains how to set up your fork of the Terraform course repository and configure automated grading.

## Overview

In this course, you will:
1. Fork the main course repository to your own GitHub account
2. Complete labs in your fork's `week-XX/lab-YY/student-work/` directories
3. Create Pull Requests **within your own fork** to trigger automated grading
4. Deploy infrastructure to **your own AWS account**

## Initial Setup

### Step 1: Fork the Repository

1. Navigate to https://github.com/shart-cloud/labs_terraform_course
2. Click the **Fork** button in the top-right corner
3. Select your personal GitHub account as the destination
4. Wait for the fork to complete

You should now have: `https://github.com/YOUR-USERNAME/labs_terraform_course`

### Step 2: Clone Your Fork

```bash
git clone https://github.com/YOUR-USERNAME/labs_terraform_course.git
cd labs_terraform_course
```

### Step 3: Set Up GitHub Actions Secrets

The automated grading system needs access to your AWS account and Infracost. You must add these as **GitHub Secrets** in your fork.

**Choose your preferred method:**

#### Method A: Using GitHub Web Interface (Recommended - Most Reliable)

The web interface works in all environments, including GitHub Codespaces.

**Quick version**:

##### 3.1 Get Your AWS Credentials

1. Log in to your AWS account
2. Create an IAM user for Terraform (if you haven't already):
   - Navigate to IAM → Users → Create User
   - Username: `terraform-student`
   - Attach policy: `AdministratorAccess` (for learning purposes)
   - Create access key for "Command Line Interface (CLI)"
   - **Save the Access Key ID and Secret Access Key**

##### 3.2 Get Your Infracost API Key

1. Register for a free Infracost account:
   ```bash
   infracost auth login
   ```
   
2. Get your API key:
   ```bash
   infracost configure get api_key
   ```

##### 3.3 Add Secrets via Web Interface

1. Go to your fork: `https://github.com/YOUR-USERNAME/labs_terraform_course`
2. Click **Settings** (in the top navigation bar of YOUR fork)
3. In the left sidebar, expand **Secrets and variables** → Click **Actions**
4. Click **New repository secret** button (green button on the right)
5. Add three secrets one at a time:
   
   **Secret 1:**
   - **Name**: `AWS_ACCESS_KEY_ID`  
   - **Value**: Your AWS access key ID
   - Click **Add secret**
   
   **Secret 2:**
   - **Name**: `AWS_SECRET_ACCESS_KEY`  
   - **Value**: Your AWS secret access key
   - Click **Add secret**
   
   **Secret 3:**
   - **Name**: `INFRACOST_API_KEY`  
   - **Value**: Your Infracost API key
   - Click **Add secret**

6. Verify all three secrets are listed (you won't see the values, just the names)

⚠️ **IMPORTANT**: Never commit these credentials to your code!

#### Method B: Using GitHub CLI (Alternative - For Local Development)

The GitHub CLI can be faster but requires proper authentication scopes. This may not work in GitHub Codespaces due to token limitations.

**Complete guide**: See [GITHUB_CLI_SETUP.md](GITHUB_CLI_SETUP.md)

**Quick version**:

```bash
# Install gh CLI (if not already installed)
# macOS: brew install gh
# Linux/Windows: See https://cli.github.com

# Login to GitHub with workflow scope
gh auth login --scopes repo,workflow

# Navigate to your fork
cd labs_terraform_course

# Run the automated setup script
./scripts/setup-secrets.sh

# Or set secrets manually
gh secret set AWS_ACCESS_KEY_ID -R YOUR-USERNAME/labs_terraform_course
gh secret set AWS_SECRET_ACCESS_KEY -R YOUR-USERNAME/labs_terraform_course
gh secret set INFRACOST_API_KEY -R YOUR-USERNAME/labs_terraform_course

# Verify
gh secret list -R YOUR-USERNAME/labs_terraform_course
```

**Troubleshooting:**
- **"HTTP 403: Resource not accessible"** - Use Method A (Web Interface) instead. Codespaces tokens don't have `workflow` scope.
- **"Multiple remotes detected"** - Use the `-R` flag with your repo: `gh secret set SECRET_NAME -R YOUR-USERNAME/labs_terraform_course`

### Step 4: Configure AWS CLI Locally

For local development and testing:

```bash
aws configure
```

Enter:
- AWS Access Key ID: (your access key)
- AWS Secret Access Key: (your secret key)
- Default region: `us-east-1`
- Default output format: `json`

Verify:
```bash
aws sts get-caller-identity
```

## Working on Labs

### Lab Workflow

For each lab, follow this process:

#### 1. Create a Branch (Recommended)

```bash
git checkout -b week-00-lab-00
```

Or work directly on `main` if you prefer.

#### 2. Navigate to Lab Directory

```bash
cd week-00/lab-00/student-work
```

#### 3. Complete Your Work

Write your Terraform code following the lab instructions.

#### 4. Test Locally

```bash
# Format your code
terraform fmt

# Initialize
terraform init

# Validate
terraform validate

# Check cost estimate
infracost breakdown --path .

# Review the plan
terraform plan

# Deploy (to your AWS account)
terraform apply
```

#### 5. Commit and Push

```bash
git add .
git commit -m "Complete Week 0 Lab 0"
git push origin week-00-lab-00
```

(Or `git push origin main` if working on main branch)

#### 6. Create Pull Request **In Your Fork**

This is the key difference from typical workflows!

1. Go to **your fork** on GitHub: `https://github.com/YOUR-USERNAME/labs_terraform_course`
2. Click **Pull requests** → **New pull request**
3. **IMPORTANT**: Both base and compare should be in YOUR fork:
   - Base repository: `YOUR-USERNAME/labs_terraform_course` base: `main`
   - Head repository: `YOUR-USERNAME/labs_terraform_course` compare: `week-00-lab-00`
4. Title your PR: `Week 0 Lab 0 - Your Name`
5. Fill out the PR template
6. Click **Create pull request**

#### 7. Wait for Automated Grading

Within a few minutes, the GitHub Actions workflow will:
- ✅ Check code formatting
- ✅ Validate Terraform configuration
- ✅ Run lab-specific tests
- ✅ Generate cost estimates
- ✅ Perform security scanning
- ✅ Calculate your grade (0-100 points)
- ✅ Post results as a comment on your PR

#### 8. Review Your Grade

The grading bot will post a detailed comment showing:
- **Final Grade**: Points and letter grade
- **Breakdown by Category**:
  - Code Quality (25 points)
  - Functionality (30 points)
  - Cost Management (20 points)
  - Security (15 points)
  - Documentation (10 points)
- **Specific Issues**: What passed and what needs fixing

#### 9. Improve Your Grade (If Needed)

If you scored less than desired:

1. Review the feedback in the PR comment
2. Fix the issues in your code
3. Commit and push the changes:
   ```bash
   git add .
   git commit -m "Fix formatting issues"
   git push origin week-00-lab-00
   ```
4. The grading workflow will automatically re-run
5. The bot will update its comment with your new grade

#### 10. Submit to Instructor

Once you're satisfied with your grade:

1. **Option A**: Tag your instructor in a PR comment:
   ```
   @shart-cloud Ready for final review! Grade: 95/100
   ```

2. **Option B**: Submit your PR link via your course LMS/form

3. **Option C**: Your instructor may have a list of student forks and will check them directly

#### 11. Clean Up Resources

After grading is complete:

```bash
cd week-00/lab-00/student-work
terraform destroy
```

Or wait for the auto-teardown action (8 hours).

**IMPORTANT**: Always clean up to avoid unexpected AWS charges!

## Grading Breakdown

### Code Quality (25 points)
- Terraform formatting (`terraform fmt`) - 5 pts
- Terraform validation - 5 pts
- No hardcoded credentials - 5 pts
- Naming conventions - 5 pts
- Terraform version requirement - 5 pts

### Functionality (30 points)
- Lab-specific requirements - 20 pts
- Outputs defined correctly - 5 pts
- Resources in plan - 5 pts

### Cost Management (20 points)
- Infracost analysis - 5 pts
- Within budget limits - 10 pts
- AutoTeardown tag present - 5 pts

### Security (15 points)
- Security scan (Checkov) - 15 pts
  - 0 issues: 15 pts
  - 1-3 issues: 10 pts
  - 4-5 issues: 5 pts
  - 6+ issues: 0 pts

### Documentation (10 points)
- Code comments - 5 pts
- README.md - 5 pts

### Letter Grades
- **A**: 90-100 points
- **B**: 80-89 points
- **C**: 70-79 points
- **D**: 60-69 points
- **F**: Below 60 points

## Troubleshooting

### GitHub Actions Not Running

**Problem**: PR created but no grading workflow runs.

**Solutions**:
- Ensure you have GitHub Actions enabled in your fork (Settings → Actions)
- Check that your PR modifies files in `week-*/lab-*/student-work/`
- Verify the workflow file exists: `.github/workflows/grading.yml`

### AWS Authentication Errors in Actions

**Problem**: Workflow fails with AWS authentication errors.

**Solutions**:
- Verify you added `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` as repository secrets
- Check secret names are exact (case-sensitive)
- Ensure your IAM user has proper permissions
- Test credentials locally: `aws sts get-caller-identity`

### Infracost Errors

**Problem**: Cost estimation fails.

**Solutions**:
- Verify you added `INFRACOST_API_KEY` as a repository secret
- Test locally: `infracost configure get api_key`
- Re-authenticate: `infracost auth login`

### Terraform Plan Fails

**Problem**: `terraform plan` fails in the workflow.

**Solutions**:
- Test locally first: `terraform init && terraform plan`
- Check for syntax errors: `terraform validate`
- Ensure all required variables are defined
- Review the workflow logs for specific errors

### Wrong PR Base

**Problem**: Created PR against the original repository instead of your fork.

**Solution**:
- Close the incorrect PR
- Create a new PR with base = your fork's main branch
- Make sure both "base" and "compare" dropdowns show your username

### Grading Comment Not Posted

**Problem**: Workflow runs but no comment appears.

**Solutions**:
- Check workflow logs for errors
- Ensure your fork has Issues enabled (Settings → General → Features → Issues)
- Verify the workflow has `pull-requests: write` permission

## Tips for Success

### Before Submitting
- ✅ Run `terraform fmt` on all files
- ✅ Run `terraform validate` to check for errors
- ✅ Test `terraform plan` locally
- ✅ Run `infracost breakdown` to check costs
- ✅ Review your code for hardcoded credentials
- ✅ Ensure all required tags are present
- ✅ Add comments to explain your code
- ✅ Test deployment in your AWS account

### Cost Management
- 💰 Always run Infracost before applying
- 💰 Use smallest instance types (t3.micro, t3.nano)
- 💰 Destroy resources when done
- 💰 Monitor your AWS billing dashboard
- 💰 Keep billing alerts active (set up in Week 0)

### Security
- 🔒 Never commit AWS credentials
- 🔒 Never commit `.tfstate` files
- 🔒 Use `.gitignore` properly
- 🔒 Enable encryption on resources
- 🔒 Review Checkov findings

### Getting Help
1. Check lab README for troubleshooting
2. Review your PR comments for specific errors
3. Test locally before pushing
4. Ask questions in course discussion forum
5. Attend office hours
6. Tag instructor in PR if stuck

## FAQ

**Q: Do I need to merge my PR?**
A: Not required for grading, but you can merge it to keep your main branch updated.

**Q: Can I resubmit if I get a low grade?**
A: Yes! Just push new commits to your PR branch and it will re-grade automatically.

**Q: Will the workflow deploy to AWS automatically?**
A: No, the workflow only validates your code. You deploy manually from your local machine.

**Q: How do I know what grade I need?**
A: Check your lab's SUBMISSION.md for specific requirements and grading criteria.

**Q: Can I work on multiple labs at once?**
A: Yes, create separate branches for each lab and submit separate PRs.

**Q: Do I need to fork again for each lab?**
A: No, use the same fork for all labs. Just work in different `week-XX/lab-YY` directories.

**Q: What if I accidentally push sensitive data?**
A: Immediately delete the commit, rotate your AWS credentials, and force-push to remove it from history.

---

**Ready to start?** Head over to [Week 0, Lab 0](week-00/lab-00/README.md) for your first lab!
