# Lab 0 Submission Checklist

## Required Deliverables

### 1. Screenshots

Submit the following screenshots:

- [ ] `terraform version` output showing installed version (1.9.0+)
- [ ] `aws sts get-caller-identity` output showing AWS account details
- [ ] `terraform output` from common/billing-setup directory showing budget configuration
- [ ] SNS email confirmation showing budget alert subscription
- [ ] Infracost output showing cost breakdown for S3 bucket
- [ ] AWS Console showing deployed S3 bucket

### 2. Code Requirements

Your `student-work/` directory must contain:

- [ ] `main.tf` with properly configured S3 bucket resource
- [ ] `outputs.tf` with bucket name and ARN outputs
- [ ] `backend.tf` with S3 remote state configuration
- [ ] `.gitignore` that prevents committing state files
- [ ] All code passes `terraform fmt -check`
- [ ] All code passes `terraform validate`
- [ ] S3 bucket name includes your student ID or GitHub username
- [ ] All required tags present:
  - Name
  - Environment
  - ManagedBy
  - Student
  - AutoTeardown

### 3. Infrastructure Verification

- [ ] S3 bucket successfully created in AWS
- [ ] Bucket versioning is enabled
- [ ] Bucket encryption is enabled
- [ ] State storage bucket created in S3 (`terraform-state-YOUR-ACCOUNT-ID`)
- [ ] State successfully migrated to S3 (no local `terraform.tfstate` committed)
- [ ] Backend configuration includes `use_lockfile = true`
- [ ] No hardcoded credentials in code
- [ ] No state files or `.terraform/` directory in Git
- [ ] Estimated monthly cost is under $5

### 4. Pull Request

- [ ] PR title follows format: "Week 0 Lab 0 - [Your Name]"
- [ ] PR description filled out completely
- [ ] All checklist items in PR template completed
- [ ] Cost estimate included in PR description

### 5. Cost Management

- [ ] Billing budget deployed via Terraform from `common/billing-setup/` directory
- [ ] Screenshot of `terraform output` from common/billing-setup showing budget details
- [ ] Budget threshold set between $10-20
- [ ] Email notifications enabled and confirmed (SNS subscription confirmed)
- [ ] Infracost report generated and reviewed for S3 resources
- [ ] `terraform.tfvars` created in common/billing-setup (should NOT be committed to Git)

### 6. Documentation

Answer these questions in your PR description:

1. What is the estimated monthly cost of your S3 bucket (from Infracost)?
2. What AWS region did you deploy to and why?
3. What is the name of your state storage bucket?
4. What budget limit did you set in your Terraform billing configuration?
5. Why is remote state important for infrastructure management?
6. Did you encounter any issues during setup? How did you resolve them?
7. How long did it take to complete this lab?

## Grading Rubric (100 points)

### Code Quality (20 points)
- Terraform fmt passes (4 pts)
- Terraform validate passes (4 pts)
- No hardcoded credentials (4 pts)
- Proper naming conventions (4 pts)
- All required tags present (4 pts)

### Functionality (30 points)
- S3 bucket successfully deployed (10 pts)
- Versioning enabled (5 pts)
- Encryption enabled (5 pts)
- Outputs defined correctly (5 pts)
- Backend configured correctly (5 pts)

### State Management (15 points)
- Remote state bucket created (5 pts)
- State migrated to S3 successfully (5 pts)
- No state files committed to Git (3 pts)
- `.gitignore` properly configured (2 pts)

### Cost Management (20 points)
- Billing budget deployed via Terraform (10 pts)
- SNS subscription confirmed (5 pts)
- Infracost report generated (5 pts)

### Documentation (10 points)
- Screenshots provided (5 pts)
- Questions answered (5 pts)

### Submission Format (5 points)
- PR properly formatted (3 pts)
- All checklist items completed (2 pts)

## Submission Instructions

1. Complete all tasks in the lab
2. Take required screenshots
3. Commit your code to your forked repository
4. Create a pull request with the proper format
5. Fill out the PR template completely
6. Wait for automated grading workflow to run
7. Respond to any feedback from the instructor

## Due Date

[To be specified by instructor]

## Late Submission Policy

[To be specified by instructor]

## Getting Help

If you're stuck:

1. Review the lab README.md thoroughly
2. Check the troubleshooting section
3. Search common errors online
4. Ask in the course discussion forum
5. Attend office hours
6. Email the instructor as a last resort

## After Submission

After your lab is graded:

- Review feedback carefully
- Run `terraform destroy` to clean up resources
- Or wait for auto-teardown after 8 hours
- Keep a copy of your work for reference

## Academic Integrity

- You may discuss concepts with classmates
- You may search online for Terraform syntax help
- You must write your own code
- You must not copy code from other students
- Proper attribution required for any external resources used

## Questions?

If anything is unclear, ask in the discussion forum or during office hours before the due date.
