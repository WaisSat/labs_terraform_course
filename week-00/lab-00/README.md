# Lab 0: Getting Started

## Objective

Set up your development environment, configure AWS credentials, install required tools, and deploy a simple S3 bucket to verify your setup.

## Estimated Time

3.5-4.5 hours

## Prerequisites

- Personal AWS account (or AWS Academy sandbox access)
- GitHub account
- Command-line terminal access

## Tasks

### Part 1: Install Required Tools (30 minutes)

#### 1.1 Install Terraform

**Important**: You need Terraform 1.9.0 or later for S3 native state locking support.

**macOS (using Homebrew):**
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

**Linux:**
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

**Windows (using Chocolatey):**
```powershell
choco install terraform
```

Verify installation (ensure version is 1.9.0 or later):
```bash
terraform version
```

#### 1.2 Install AWS CLI

Follow the [official AWS CLI installation guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) for your operating system.

Verify installation:
```bash
aws --version
```

#### 1.3 Install Infracost

**macOS/Linux:**
```bash
curl -fsSL https://raw.githubusercontent.com/infracost/infracost/master/scripts/install.sh | sh
```

**Windows:**
```powershell
choco install infracost
```

Register for Infracost API key:
```bash
infracost configure set api_key YOUR_API_KEY
```

#### 1.4 Install VS Code and Extensions (Recommended)

- Download [VS Code](https://code.visualstudio.com/)
- Install extensions:
  - HashiCorp Terraform
  - AWS Toolkit

### Part 2: Configure AWS Credentials (30 minutes)

#### 2.1 Create IAM User

1. Log in to AWS Console
2. Navigate to IAM → Users → Create User
3. Create user with programmatic access
4. Attach policy: `AdministratorAccess` (for learning purposes only)
5. Save access key ID and secret access key

#### 2.2 Configure AWS CLI

```bash
aws configure
```

Enter:
- AWS Access Key ID
- AWS Secret Access Key
- Default region: `us-east-1`
- Default output format: `json`

Verify:
```bash
aws sts get-caller-identity
```

### Part 3: Fork and Clone Repository (10 minutes)

#### 3.1 Fork the Repository

1. Navigate to https://github.com/shart-cloud/labs_terraform_course
2. Click "Fork" to create your own copy
3. Clone your fork:

```bash
git clone https://github.com/YOUR-USERNAME/labs_terraform_course.git
cd labs_terraform_course
```

**Note**: Replace `YOUR-USERNAME` with your actual GitHub username.

### Part 4: Set Up Billing Alerts with Terraform (20 minutes)

The billing setup has been pre-configured for you in the `common/billing-setup/` directory.

#### 4.1 Navigate to Billing Setup Directory

```bash
cd common/billing-setup
```

#### 4.2 Create Your Configuration File

Copy the example file and edit it with your information:

```bash
cp terraform.tfvars.example terraform.tfvars
```

Edit `terraform.tfvars` using your preferred text editor:

```hcl
student_name         = "your-github-username"
alert_email          = "your-email@example.com"
monthly_budget_limit = "20"
```

#### 4.3 Deploy Your Billing Budget

```bash
# Initialize Terraform
terraform init

# Review what will be created
terraform plan

# Create the budget
terraform apply
```

Type `yes` when prompted.

#### 4.4 Confirm Email Subscription

**Important**: Check your email for an SNS subscription confirmation message. Click the "Confirm subscription" link to activate budget alerts.

#### 4.5 Verify Budget Creation

```bash
# View the outputs (including next steps)
terraform output

# Optionally verify in AWS Console
aws budgets describe-budgets --account-id $(aws sts get-caller-identity --query Account --output text)
```

### Part 4.5: Understanding the Terraform Workflow (10 minutes)

Before writing code, understand the typical Terraform workflow:

```
Write → Init → Plan → Apply → Verify
  ↑                              ↓
  ← Modify ← Review ← Destroy ←
```

**Core Commands:**

1. **`terraform init`**
   - Downloads provider plugins
   - Initializes backend (state storage)
   - Run once per directory or when providers change

2. **`terraform fmt`**
   - Automatically formats your code to HCL standards
   - Run before committing code
   - Use `-check` flag to verify without modifying

3. **`terraform validate`**
   - Checks syntax and internal consistency
   - Doesn't check if resources can actually be created
   - Runs offline (no API calls)

4. **`terraform plan`**
   - Creates an execution plan
   - Shows what will change: `+` create, `-` destroy, `~` modify
   - Makes API calls to check current state
   - **Always run before apply!**

5. **`terraform apply`**
   - Executes the plan
   - Creates/modifies/deletes real infrastructure
   - Asks for confirmation (type `yes`)
   - Updates state file

6. **`terraform destroy`**
   - Removes all infrastructure
   - Use for cleanup
   - Also asks for confirmation

7. **`terraform output`**
   - Displays output values
   - Can be run anytime after infrastructure is created

**Best Practice Workflow:**
```bash
terraform fmt && terraform validate && terraform plan
```

Run this before every `apply` to catch errors early.

### Part 5: Deploy Test Infrastructure (60 minutes)

Navigate to your student work directory:
```bash
cd week-00/lab-00/student-work
```

In this section, you'll build your first Terraform configuration step-by-step. Instead of copying code all at once, you'll learn what each piece does and why it's needed.

#### 5.1 Understanding HCL (HashiCorp Configuration Language)

Terraform uses HCL to describe infrastructure. HCL is declarative - you describe the desired state, not the steps to get there.

**Key HCL syntax concepts:**
- **Blocks**: Containers for configuration (e.g., `resource`, `provider`)
- **Arguments**: Assign values (e.g., `region = "us-east-1"`)
- **Expressions**: Reference values (e.g., `aws_s3_bucket.test_bucket.id`)
- **Comments**: Use `#` for single line or `/* */` for multi-line

#### 5.2 Step 1: Configure Terraform and AWS Provider

Every Terraform configuration needs two things:
1. **Terraform block**: Specifies required Terraform version and providers
2. **Provider block**: Configures the cloud platform you're using

Create `main.tf` and add:

```hcl
# Terraform block - defines version requirements
terraform {
  required_version = ">= 1.9.0"  # Minimum version needed for S3 native locking
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"  # Where to download the AWS provider
      version = "~> 5.0"         # Use any 5.x version (but not 6.0)
    }
  }
}

# Provider block - configures AWS
provider "aws" {
  region = "us-east-1"  # AWS region where resources will be created
}
```

**What this does:**
- The `terraform` block is metadata about your configuration
- `required_version` ensures team members use compatible Terraform versions
- `required_providers` tells Terraform which plugins to download
- The `provider` block configures authentication and default settings for AWS

**Test it:**
```bash
terraform init
```

You should see Terraform download the AWS provider. This creates a `.terraform` directory with provider plugins.

#### 5.3 Step 2: Create Your First Resource - A Basic S3 Bucket

Now add a **resource block** to `main.tf`. Resources are the core of Terraform - they represent infrastructure objects.

**Resource block syntax:**
```hcl
resource "PROVIDER_TYPE" "LOCAL_NAME" {
  argument1 = value1
  argument2 = value2
}
```

Add this to your `main.tf` (below the provider block):

```hcl
# Resource block - creates an S3 bucket
resource "aws_s3_bucket" "test_bucket" {
  bucket = "terraform-lab-00-YOUR-GITHUB-USERNAME"  # Replace with your GitHub username
}
```

**Understanding this resource:**
- `aws_s3_bucket` - The resource type (from AWS provider)
- `test_bucket` - Local name to reference this resource elsewhere
- `bucket` - The globally unique name for your S3 bucket

**Important:** S3 bucket names must be globally unique across all AWS accounts. Use your GitHub username or student ID to avoid conflicts.

**Test it:**
```bash
terraform fmt      # Format your code
terraform validate # Check for syntax errors
terraform plan     # Preview what will be created
```

The `plan` command shows you what Terraform will do without actually doing it. You should see it wants to create 1 resource.

**Do NOT apply yet** - we need to add required tags first!

#### 5.4 Step 3: Add Required Tags

All resources in this course require specific tags for tracking and cost management. Tags are key-value pairs that help organize and identify resources.

**Understanding Required Tags:**
- `Name`: Human-readable resource identifier
- `Environment`: Deployment context (e.g., "Learning", "Production")
- `ManagedBy`: Shows infrastructure is managed by Terraform
- `Student`: Your GitHub username for resource ownership tracking
- `AutoTeardown`: Set to "8h" to trigger automatic cleanup after 8 hours (prevents unexpected AWS charges)

Update your `aws_s3_bucket` resource to include tags:

```hcl
resource "aws_s3_bucket" "test_bucket" {
  bucket = "terraform-lab-00-YOUR-GITHUB-USERNAME"  # Replace with your GitHub username

  tags = {
    Name         = "Lab 0 Test Bucket"
    Environment  = "Learning"
    ManagedBy    = "Terraform"
    Student      = "your-github-username"  # Replace with your GitHub username
    AutoTeardown = "8h"
  }
}
```

**What's new:**
- The `tags` argument takes a map (key-value pairs) in curly braces
- Each tag is specified as `key = "value"`
- Tags are metadata - they don't affect functionality but help with organization

**Test your changes:**
```bash
terraform fmt
terraform validate
terraform plan
```

#### 5.5 Step 4: Deploy Your Basic S3 Bucket

Now let's actually create the bucket:

```bash
terraform apply
```

Terraform will show you the plan again and ask for confirmation. Type `yes` to proceed.

**What just happened:**
1. Terraform compared your desired state (the code) to actual state (what exists in AWS)
2. It created an execution plan
3. It called AWS APIs to create the bucket
4. It saved the current state to `terraform.tfstate`

**Verify in AWS:**
```bash
aws s3 ls | grep terraform-lab-00
```

Or check the AWS Console: https://s3.console.aws.amazon.com/s3/buckets

#### 5.6 Step 5: Add Versioning (Using a Separate Resource)

In Terraform, S3 bucket features like versioning are configured as separate resources. This follows AWS best practices for fine-grained control.

Add this new resource to `main.tf`:

```hcl
# Enable versioning on the S3 bucket
resource "aws_s3_bucket_versioning" "test_bucket_versioning" {
  bucket = aws_s3_bucket.test_bucket.id  # Reference to our bucket

  versioning_configuration {
    status = "Enabled"
  }
}
```

**Understanding references:**
- `aws_s3_bucket.test_bucket.id` references the bucket we created earlier
- This creates a dependency: Terraform knows it must create the bucket before versioning
- `.id` is an attribute exported by the `aws_s3_bucket` resource

**Nested blocks:**
- `versioning_configuration` is a nested block (a block within a block)
- It groups related settings together

**Apply the change:**
```bash
terraform plan   # See that only versioning will be added
terraform apply
```

Notice Terraform only modifies what changed - it doesn't recreate the bucket.

#### 5.7 Step 6: Add Encryption

Add encryption to protect your data at rest:

```hcl
# Enable server-side encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "test_bucket_encryption" {
  bucket = aws_s3_bucket.test_bucket.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"  # AWS managed encryption
    }
  }
}
```

**Understanding nested blocks:**
- This resource has multiple levels of nesting: `rule` → `apply_server_side_encryption_by_default`
- Each level groups related configuration
- `AES256` uses AWS-managed encryption keys (no extra cost)

**Apply the change:**
```bash
terraform plan
terraform apply
```

#### 5.8 Step 7: Add Outputs

Outputs let you extract information from your infrastructure. They're printed after `apply` and can be queried later.

Create a new file `outputs.tf`:

```hcl
# Output the bucket name
output "bucket_name" {
  description = "Name of the S3 bucket"
  value       = aws_s3_bucket.test_bucket.id
}

# Output the bucket ARN
output "bucket_arn" {
  description = "ARN of the S3 bucket"
  value       = aws_s3_bucket.test_bucket.arn
}
```

**Understanding outputs:**
- `output` blocks expose values from your infrastructure
- `description` documents what the output represents
- `value` can reference resource attributes
- ARN (Amazon Resource Name) is a unique identifier for AWS resources

**Apply and see outputs:**
```bash
terraform apply
```

You should see the outputs printed at the end.

**Query outputs anytime:**
```bash
terraform output
terraform output bucket_name  # Get a specific output
```

#### 5.9 Your Complete Configuration

At this point, your `main.tf` should look like this:

<details>
<summary>Click to see complete main.tf</summary>

```hcl
# Terraform block - defines version requirements
terraform {
  required_version = ">= 1.9.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# Provider block - configures AWS
provider "aws" {
  region = "us-east-1"
}

# Resource block - creates an S3 bucket
resource "aws_s3_bucket" "test_bucket" {
  bucket = "terraform-lab-00-YOUR-GITHUB-USERNAME"  # Replace with your GitHub username

  tags = {
    Name         = "Lab 0 Test Bucket"
    Environment  = "Learning"
    ManagedBy    = "Terraform"
    Student      = "your-github-username"  # Replace with your GitHub username
    AutoTeardown = "8h"
  }
}

# Enable versioning on the S3 bucket
resource "aws_s3_bucket_versioning" "test_bucket_versioning" {
  bucket = aws_s3_bucket.test_bucket.id

  versioning_configuration {
    status = "Enabled"
  }
}

# Enable server-side encryption
resource "aws_s3_bucket_server_side_encryption_configuration" "test_bucket_encryption" {
  bucket = aws_s3_bucket.test_bucket.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}
```
</details>

And your `outputs.tf`:

<details>
<summary>Click to see complete outputs.tf</summary>

```hcl
# Output the bucket name
output "bucket_name" {
  description = "Name of the S3 bucket"
  value       = aws_s3_bucket.test_bucket.id
}

# Output the bucket ARN
output "bucket_arn" {
  description = "ARN of the S3 bucket"
  value       = aws_s3_bucket.test_bucket.arn
}
```
</details>

#### 5.10 Run Infracost

Before considering your infrastructure "done", always check the cost:

```bash
infracost breakdown --path .
```

**Expected cost:** ~$0.50/month for minimal storage

**Understanding the output:**
- S3 storage cost is typically $0.023 per GB/month
- Data transfer out has costs (but transfers in are free)
- Your bucket with no data has minimal cost

#### 5.11 Final Verification

**Verify your configuration is clean:**
```bash
terraform fmt -check    # Should show no changes needed
terraform validate      # Should show "Success!"
```

**Verify in AWS Console:**
1. Navigate to: https://s3.console.aws.amazon.com/s3/buckets
2. Find your bucket (should be named `terraform-lab-00-YOUR-USERNAME`)
3. Click on the bucket name
4. Go to **Properties** tab
5. Scroll down to **Bucket Versioning** - should show "Enabled"
6. Check **Default encryption** - should show "Enabled" with SSE-S3 (AES-256)
7. Go to **Tags** tab - verify all 5 required tags are present

**Verify with AWS CLI:**
```bash
# Check bucket exists
aws s3 ls | grep terraform-lab-00

# Check versioning status
aws s3api get-bucket-versioning --bucket terraform-lab-00-YOUR-USERNAME

# Check encryption
aws s3api get-bucket-encryption --bucket terraform-lab-00-YOUR-USERNAME
```

#### 5.12 Key Terraform Concepts You Just Learned

**1. Infrastructure as Code (IaC)**
- Your infrastructure is defined in version-controlled files
- Changes are reviewable and repeatable
- No more clicking in consoles or running manual scripts

**2. Declarative vs. Imperative**
- Declarative: "I want a bucket with versioning" (what you want)
- Imperative: "Create bucket, then enable versioning" (how to do it)
- Terraform figures out the "how" for you

**3. Resource Dependencies**
- Terraform automatically determines the order to create resources
- When you referenced `aws_s3_bucket.test_bucket.id`, you created an implicit dependency
- Explicit dependencies can be set with `depends_on` (you'll learn this later)

**4. State Management**
- The `terraform.tfstate` file tracks what Terraform created
- **Never manually edit this file**
- It's how Terraform knows what exists vs. what you want
- In future labs, you'll store state remotely in S3

**5. Idempotency**
- Running `terraform apply` multiple times with the same code produces the same result
- If nothing changed in your code, Terraform won't modify anything
- Try it: run `terraform apply` again - it should say "No changes"

**6. Resource Addressing**
- Each resource has an address: `TYPE.NAME` (e.g., `aws_s3_bucket.test_bucket`)
- Use this to reference resources elsewhere in your code
- Also used in commands: `terraform state show aws_s3_bucket.test_bucket`

**Try these commands to explore:**
```bash
# Show the state of a specific resource
terraform state show aws_s3_bucket.test_bucket

# List all resources in state
terraform state list

# Show all outputs
terraform output

# See the state file (but don't edit it!)
cat terraform.tfstate | jq '.resources'  # Requires jq installed
```

### Part 5.5: Set Up Remote State Storage (30 minutes)

**CRITICAL**: Before you submit your work via Git, you need to migrate your state file to remote storage. If you commit and push with only a local state file, you won't be able to manage your infrastructure later!

#### Why Remote State Matters

Right now, your infrastructure state is stored in a local file: `terraform.tfstate`

**Problems with local state:**
- ❌ If you lose the file, Terraform "forgets" what it created
- ❌ Can't manage resources from multiple computers
- ❌ Can't collaborate with team members
- ❌ Easy to accidentally commit sensitive data to Git
- ❌ After you push to GitHub, you can't easily destroy resources from another location

**Solution: Remote state in S3**
- ✅ State stored in AWS S3 bucket
- ✅ Accessible from anywhere with AWS credentials
- ✅ Encrypted and versioned
- ✅ Native locking prevents conflicts (Terraform 1.9+)
- ✅ Never committed to Git

#### Step 1: Create a State Bucket

First, create a dedicated S3 bucket to store your Terraform state files. This bucket will be used for ALL your labs.

**Get your AWS account ID:**
```bash
aws sts get-caller-identity --query Account --output text
```

**Create the state bucket:**
```bash
# Replace YOUR-ACCOUNT-ID with the account ID from above
aws s3 mb s3://terraform-state-YOUR-ACCOUNT-ID --region us-east-1
```

**Example:**
```bash
aws s3 mb s3://terraform-state-123456789012 --region us-east-1
```

**Enable versioning (protects against accidental deletions):**
```bash
aws s3api put-bucket-versioning \
  --bucket terraform-state-YOUR-ACCOUNT-ID \
  --versioning-configuration Status=Enabled
```

**Enable encryption:**
```bash
aws s3api put-bucket-encryption \
  --bucket terraform-state-YOUR-ACCOUNT-ID \
  --server-side-encryption-configuration '{
    "Rules": [{
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "AES256"
      }
    }]
  }'
```

**Verify the bucket was created:**
```bash
aws s3 ls | grep terraform-state
```

#### Step 2: Configure Backend in Your Lab

Now tell Terraform to use this bucket for state storage.

**Create a new file** `backend.tf` in `week-00/lab-00/student-work/`:

```hcl
# Backend configuration for remote state storage
terraform {
  backend "s3" {
    bucket         = "terraform-state-YOUR-ACCOUNT-ID"  # Replace with your bucket name
    key            = "week-00/lab-00/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    use_lockfile   = true  # Native S3 locking (Terraform 1.9+)
  }
}
```

**Understanding the backend block:**
- `bucket` - The S3 bucket you just created
- `key` - Path within the bucket (organizes state files by lab)
- `region` - AWS region where the bucket exists
- `encrypt` - Encrypts state at rest
- `use_lockfile` - Uses S3's native locking to prevent concurrent modifications

**Important:** Replace `YOUR-ACCOUNT-ID` with your actual AWS account ID!

#### Step 3: Migrate State to S3

Now migrate your existing local state to S3:

```bash
# Re-initialize Terraform with the new backend
terraform init -migrate-state
```

**What happens:**
1. Terraform detects you changed from local to S3 backend
2. It asks: "Do you want to copy existing state to the new backend?"
3. Type `yes`
4. Your local state is uploaded to S3
5. Terraform now uses S3 for all operations

**Verify the migration:**
```bash
# Check that state is in S3
aws s3 ls s3://terraform-state-YOUR-ACCOUNT-ID/week-00/lab-00/

# Should show: terraform.tfstate
```

**Test it works:**
```bash
terraform plan
```

Should show "No changes" - Terraform successfully read state from S3!

#### Step 4: Clean Up Local State (Optional)

After successful migration, you can remove the local state files:

```bash
# These are no longer needed
rm terraform.tfstate
rm terraform.tfstate.backup  # If it exists
```

**Important:** Do this ONLY after confirming state is in S3!

#### Step 5: Update .gitignore

Make sure your `.gitignore` prevents accidentally committing state files.

Check if there's a `.gitignore` in your lab directory:
```bash
ls -la week-00/lab-00/student-work/.gitignore
```

If it doesn't exist, create one:
```bash
cat > week-00/lab-00/student-work/.gitignore << 'EOF'
# Terraform files
.terraform/
*.tfstate
*.tfstate.*
*.tfvars
.terraform.lock.hcl

# Sensitive files
terraform.rc
.terraformrc
EOF
```

**Verify state files won't be committed:**
```bash
cd week-00/lab-00/student-work
git status
```

You should NOT see `terraform.tfstate` in the list of files to be committed.

#### Understanding What You Committed

After setting up remote state, your Git repository will contain:
- ✅ `main.tf` - Your infrastructure code
- ✅ `outputs.tf` - Output definitions
- ✅ `backend.tf` - Backend configuration (no secrets)
- ✅ `.gitignore` - Protects sensitive files
- ❌ `terraform.tfstate` - NOT committed (in S3)
- ❌ `.terraform/` - NOT committed (provider plugins)

**This is the correct setup!** Code is in Git, state is in S3.

#### Test Remote State Works

To prove remote state works, try this:

```bash
# See your current directory
pwd

# Move to home directory
cd ~

# Try to run Terraform from a different location
cd ~/git/shart-cloud-gh/terraform-course/week-00/lab-00/student-work
terraform init
terraform plan
```

It should work because state is in S3, not tied to your local directory!

#### Troubleshooting Remote State

**"Error: Failed to get existing workspaces"**
- Double-check your bucket name in `backend.tf`
- Verify the bucket exists: `aws s3 ls | grep terraform-state`
- Check AWS credentials: `aws sts get-caller-identity`

**"Error: Error loading state: AccessDenied"**
- Your IAM user needs S3 permissions
- Verify: `aws s3 ls s3://terraform-state-YOUR-ACCOUNT-ID`

**"Backend initialization required"**
- Run `terraform init` whenever you change backend configuration
- Use `-migrate-state` flag to move existing state

**Want to move back to local state?**
```bash
# Remove or comment out the backend block in backend.tf
# Then run:
terraform init -migrate-state
```

### Part 6: Submit Your Work (20 minutes)

**Before submitting:** Make sure you've completed all steps in Part 5 AND Part 5.5:
- ✅ A working `main.tf` with bucket, versioning, and encryption
- ✅ An `outputs.tf` with bucket name and ARN outputs
- ✅ A `backend.tf` with S3 remote state configured
- ✅ Created state bucket in S3
- ✅ Migrated state to S3 (`terraform init -migrate-state`)
- ✅ Verified state is in S3 (not local)
- ✅ `.gitignore` prevents committing state files
- ✅ A working `main.tf` with bucket, versioning, and encryption
- ✅ An `outputs.tf` with bucket name and ARN outputs
- ✅ Run `terraform fmt` to format your code
- ✅ Run `terraform validate` to check for errors
- ✅ Successfully applied your configuration (`terraform apply`)
- ✅ Verified the bucket exists in AWS Console
- ✅ Run `infracost breakdown --path .` and reviewed costs
- ✅ Created a state storage bucket in S3
- ✅ Configured remote backend in `backend.tf`
- ✅ Migrated state to S3 successfully

#### 6.1 Set Up GitHub Secrets (First Time Only)

Before creating your first PR, you need to configure GitHub Actions secrets in **your fork**:

1. Go to your fork: `https://github.com/YOUR-USERNAME/labs_terraform_course`
2. Navigate to **Settings** → **Secrets and variables** → **Actions**
3. Add three secrets:
   - `AWS_ACCESS_KEY_ID` - Your AWS access key
   - `AWS_SECRET_ACCESS_KEY` - Your AWS secret key  
   - `INFRACOST_API_KEY` - Your Infracost API key (get via `infracost configure get api_key`)

⚠️ **Security Note**: These secrets are only accessible to workflows in YOUR fork, not the main repo.

See [STUDENT_SETUP.md](../../STUDENT_SETUP.md) for detailed instructions.

#### 6.2 Commit and Push Your Work

```bash
# Create a branch (recommended)
git checkout -b week-00-lab-00

# Add your files (main.tf, outputs.tf, backend.tf, .gitignore)
git add week-00/lab-00/student-work/

# Verify state files are NOT being committed
git status

# You should see:
#   main.tf
#   outputs.tf
#   backend.tf
#   .gitignore
# You should NOT see terraform.tfstate or .terraform/

# Commit
git commit -m "Week 0 Lab 0 - Your Name"

# Push to your fork
git push origin week-00-lab-00
```

#### 6.3 Create Pull Request in Your Fork

**IMPORTANT**: Create the PR within YOUR fork, not to the main repository!

1. Go to **your fork** on GitHub
2. Click **Pull requests** → **New pull request**
3. Set both base and compare to YOUR fork:
   - Base: `YOUR-USERNAME/labs_terraform_course` base: `main`
   - Compare: `YOUR-USERNAME/labs_terraform_course` compare: `week-00-lab-00`
4. Title: `Week 0 Lab 0 - [Your Name]`
5. Fill out the PR template
6. Create pull request

#### 6.4 Wait for Automated Grading

The grading workflow will automatically run and:
- ✅ Check code formatting (`terraform fmt`)
- ✅ Validate configuration (`terraform validate`)
- ✅ Run lab-specific tests
- ✅ Generate cost estimates (Infracost)
- ✅ Perform security scanning (Checkov)
- ✅ Calculate your grade (0-100 points)
- ✅ Post detailed results as a PR comment

**Expected grade breakdown**:
- Code Quality: 25 points
- Functionality: 30 points
- Cost Management: 20 points
- Security: 15 points
- Documentation: 10 points

#### 6.5 Review Your Grade and Iterate

1. Check the automated comment on your PR for your grade
2. If you need to improve your score:
   - Fix the issues mentioned in the feedback
   - Commit and push your changes
   - The workflow will automatically re-run and update your grade
3. Once satisfied, tag your instructor: `@shart-cloud` in a PR comment

### Part 7: Cleanup (10 minutes)

After your PR is reviewed and graded:

#### 7.1 Destroy S3 Infrastructure

Because you set up remote state, you can destroy your resources from any location:

```bash
cd week-00/lab-00/student-work

# Initialize (pulls state from S3)
terraform init

# Destroy resources
terraform destroy
```

Type `yes` to confirm.

**Benefits of remote state:**
- You can run this from any computer (as long as you have AWS credentials)
- Even if you cloned your repo to a new machine, Terraform knows what resources exist
- The state in S3 tracks everything you created

**Alternative:** Wait 8 hours for the auto-teardown GitHub Action to destroy resources automatically (based on the `AutoTeardown = "8h"` tag).

#### 7.2 (Optional) Clean Up State Bucket

**Important**: Keep your state bucket active if you have more labs to complete!

The state bucket you created (`terraform-state-YOUR-ACCOUNT-ID`) will be used for ALL labs in this course. Only remove it at the end of the semester:

```bash
# ONLY do this after completing ALL labs and destroying ALL infrastructure

# First, verify all state files are for destroyed resources
aws s3 ls s3://terraform-state-YOUR-ACCOUNT-ID/ --recursive

# Delete all state files
aws s3 rm s3://terraform-state-YOUR-ACCOUNT-ID/ --recursive

# Delete the bucket
aws s3 rb s3://terraform-state-YOUR-ACCOUNT-ID
```

**Cost:** S3 storage for state files costs less than $0.01/month, so keeping it is fine.

#### 7.3 Keep or Remove Billing Budget

**Important**: The billing budget should typically be kept active throughout the course to monitor costs. However, if you need to remove it:

```bash
cd ../../common/billing-setup
terraform destroy
```

**Note**: We recommend keeping the billing budget active for the entire semester.

## Deliverables

See [SUBMISSION.md](SUBMISSION.md) for the complete checklist.

## Troubleshooting

### Common Terraform Errors

**"Error: Invalid block definition"**
- Check your HCL syntax - missing braces `{}` or brackets
- Ensure blocks are properly closed
- Run `terraform fmt` to auto-fix formatting issues

**"Error: Reference to undeclared resource"**
- You're referencing a resource that doesn't exist
- Check spelling: `aws_s3_bucket.test_bucket` (not `test_buckets`)
- Ensure the resource is defined before you reference it

**"Error: Duplicate resource block"**
- You have two resources with the same type and name
- Each resource must have a unique combination of type + name
- Example: You can't have two `resource "aws_s3_bucket" "test_bucket"` blocks

**"Error: Missing required argument"**
- A resource is missing a required field
- Check the Terraform AWS provider documentation
- Example: `aws_s3_bucket` requires the `bucket` argument

### Terraform Init Fails
- Check internet connection
- Verify Terraform is properly installed
- Try removing `.terraform` directory and re-running `terraform init`
- Check that your `terraform` block has correct `required_providers` syntax

### AWS Authentication Errors
- Verify `aws configure` was run correctly
- Check credentials with `aws sts get-caller-identity`
- Ensure IAM user has proper permissions (AdministratorAccess for learning)
- Check that credentials aren't expired (AWS Academy credentials expire)

### S3 Bucket Name Conflicts
**"Error: BucketAlreadyExists" or "Error: InvalidBucketName"**
- S3 bucket names must be globally unique across ALL AWS accounts
- Use your student ID or GitHub username in the bucket name
- Bucket names must be lowercase, no spaces, 3-63 characters
- Valid: `terraform-lab-00-johnsmith`
- Invalid: `Terraform Lab 00`, `lab00`, `my_bucket`

### "Error: error creating S3 bucket ... Access Denied"
- Your AWS credentials don't have S3 permissions
- Verify IAM user has `AmazonS3FullAccess` or `AdministratorAccess`
- Run `aws s3 ls` to test S3 access

### Infracost Errors
- Ensure you've run `infracost auth login` or configured an API key
- Check API key is valid: `infracost configure get api_key`
- If using Infracost for the first time, sign up at https://www.infracost.io/

### State File Issues

**"Error: state snapshot was created by Terraform vX.Y.Z"**
- Your Terraform version is older than what created the state file
- Upgrade Terraform: `brew upgrade terraform` (macOS) or reinstall
- Never downgrade Terraform after creating state

**"Error: acquiring state lock"**
- Another Terraform process is running (maybe in another terminal?)
- Wait for it to finish or find and kill the process
- Check for stale lock files in S3: `aws s3 ls s3://terraform-state-YOUR-ACCOUNT-ID/week-00/lab-00/`
- Last resort: `terraform force-unlock LOCK_ID` (use carefully!)

**"Error: Failed to get existing workspaces: NoSuchBucket"**
- The state bucket doesn't exist
- Check bucket name in `backend.tf` matches what you created
- Verify bucket exists: `aws s3 ls | grep terraform-state`
- Recreate bucket if needed: `aws s3 mb s3://terraform-state-YOUR-ACCOUNT-ID`

**"Error: Backend initialization required, please run 'terraform init'"**
- You changed backend configuration
- Run `terraform init` to reconfigure
- Use `-migrate-state` if moving from local to S3 or vice versa

**"Error: Error loading state: AccessDenied"**
- Your IAM user can't access the state bucket
- Verify: `aws s3 ls s3://terraform-state-YOUR-ACCOUNT-ID/`
- Your IAM user needs S3 read/write permissions

**Accidentally committed state to Git?**
```bash
# Remove from Git history (before pushing!)
git rm --cached terraform.tfstate
git rm --cached terraform.tfstate.backup

# Make sure .gitignore is set up
echo "*.tfstate*" >> .gitignore
git add .gitignore
git commit -m "Remove state files and update .gitignore"
```

**Lost your state file completely?**
- If it's in S3: Just run `terraform init` to download it
- If you deleted the S3 bucket: You'll need to import resources manually or recreate them
- This is why remote state is critical!

### Verify Before Asking for Help

Run these diagnostic commands:
```bash
# Check Terraform version
terraform version

# Check AWS credentials
aws sts get-caller-identity

# Check Terraform syntax
terraform fmt -check
terraform validate

# See detailed error output
TF_LOG=DEBUG terraform plan 2>&1 | less
```

## Learning Outcomes Checklist

After completing this lab, you should be able to:

**Tool Setup:**
- ✅ Install and verify Terraform 1.9.0+, AWS CLI, and Infracost
- ✅ Configure AWS credentials and verify access
- ✅ Set up billing alerts and budgets using Terraform

**Terraform Fundamentals:**
- ✅ Understand HCL syntax (blocks, arguments, expressions)
- ✅ Explain the difference between terraform, provider, resource, and output blocks
- ✅ Write a terraform block with version constraints
- ✅ Configure an AWS provider
- ✅ Create resources with proper syntax
- ✅ Use resource references to create dependencies
- ✅ Add tags to AWS resources
- ✅ Define and use output values

**Terraform Workflow:**
- ✅ Use `terraform init` to initialize a configuration
- ✅ Use `terraform fmt` to format code
- ✅ Use `terraform validate` to check syntax
- ✅ Use `terraform plan` to preview changes
- ✅ Use `terraform apply` to create infrastructure
- ✅ Use `terraform output` to query values
- ✅ Use `terraform destroy` to clean up resources

**State Management:**
- ✅ Understand why remote state is critical
- ✅ Create and configure an S3 bucket for state storage
- ✅ Configure Terraform backend for S3
- ✅ Migrate local state to remote S3 backend
- ✅ Understand state locking with S3 native locking (Terraform 1.9+)
- ✅ Protect state files with .gitignore

**AWS & Cost Management:**
- ✅ Create an S3 bucket with versioning and encryption
- ✅ Run Infracost to estimate infrastructure costs
- ✅ Understand AWS resource tagging for cost tracking

**Development Practices:**
- ✅ Submit work via Git and GitHub pull requests
- ✅ Follow infrastructure as code best practices

## Next Steps

Proceed to Week 1, Lab 1 where you'll deploy more complex infrastructure with multiple resources.

## Support

- Office hours: [Schedule TBD]
- Discussion forum: [Link TBD]
- Email instructor for urgent issues
