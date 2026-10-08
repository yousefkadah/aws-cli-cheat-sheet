# AWS CLI Cheat Sheet

A practical, copy-ready **AWS CLI v2** quick reference: setup, profiles & SSO, querying output, S3, EC2, VPC, SSM, IAM, ECR, ECS, EKS, Lambda, CloudWatch, RDS, DynamoDB, SQS, SNS, Secrets Manager, Route 53, CloudFront, CloudFormation, cost checks, deployments and troubleshooting.

> **Copy buttons:** every command sits in its own fenced code block, so GitHub (and most Markdown viewers) show a **Copy** button next to it.

**Conventions** — replace these example values with your own:

| Placeholder | Meaning |
|---|---|
| `123456789012` | AWS account ID |
| `eu-central-1` | Region |
| `prod`, `dev` | Profile names in `~/.aws/config` |
| `my-bucket`, `my-cluster`, `my-service`, `my-fn` | Resource names |
| `i-0123456789abcdef0`, `sg-…`, `subnet-…`, `vpc-…` | Resource IDs |
| `<something>` | Any other value you must fill in |

---

## Index

1. [Install & Update](#1-install--update)
2. [Help, Discovery & Auto-prompt](#2-help-discovery--auto-prompt)
3. [Configure & Profiles](#3-configure--profiles)
4. [Config & Credentials Files](#4-config--credentials-files)
5. [IAM Identity Center (SSO)](#5-iam-identity-center-sso)
6. [Identity, STS & Assume Role](#6-identity-sts--assume-role)
7. [Global Options, Output & Pagination](#7-global-options-output--pagination)
8. [Filtering Output with --query (JMESPath)](#8-filtering-output-with---query-jmespath)
9. [S3: Files & Sync](#9-s3-files--sync)
10. [S3: Buckets & Settings (s3api)](#10-s3-buckets--settings-s3api)
11. [EC2: Instances](#11-ec2-instances)
12. [EC2: AMIs, Key Pairs, Volumes, Snapshots & Elastic IPs](#12-ec2-amis-key-pairs-volumes-snapshots--elastic-ips)
13. [VPC & Security Groups](#13-vpc--security-groups)
14. [SSM: Session Manager, Run Command & Parameter Store](#14-ssm-session-manager-run-command--parameter-store)
15. [IAM](#15-iam)
16. [ECR (Container Registry)](#16-ecr-container-registry)
17. [ECS](#17-ecs)
18. [EKS](#18-eks)
19. [Lambda](#19-lambda)
20. [CloudWatch Logs](#20-cloudwatch-logs)
21. [CloudWatch Metrics & Alarms](#21-cloudwatch-metrics--alarms)
22. [RDS & Aurora](#22-rds--aurora)
23. [DynamoDB](#23-dynamodb)
24. [SQS](#24-sqs)
25. [SNS](#25-sns)
26. [Secrets Manager](#26-secrets-manager)
27. [KMS](#27-kms)
28. [Route 53](#28-route-53)
29. [CloudFront](#29-cloudfront)
30. [Load Balancers (ELBv2)](#30-load-balancers-elbv2)
31. [Auto Scaling](#31-auto-scaling)
32. [CloudFormation](#32-cloudformation)
33. [ACM Certificates](#33-acm-certificates)
34. [Cost & Billing](#34-cost--billing)
35. [Tags & Resource Discovery](#35-tags--resource-discovery)
36. [Waiters](#36-waiters)
37. [Scripting, Loops & Multi-region](#37-scripting-loops--multi-region)
38. [Troubleshooting](#38-troubleshooting)
39. [Most-used Commands](#39-most-used-commands)
40. [Deployment Quick Flows](#40-deployment-quick-flows)
41. [Official References](#41-official-references)
42. [Safety Notes](#42-safety-notes)
43. [Aliases](#43-aliases)

---

## 1. Install & Update

macOS (official installer):

```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
```

```bash
sudo installer -pkg AWSCLIV2.pkg -target /
```

macOS (Homebrew):

```bash
brew install awscli
```

Linux x86_64:

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```

```bash
unzip awscliv2.zip
```

```bash
sudo ./aws/install
```

Linux ARM (Graviton, Raspberry Pi):

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
```

Update an existing Linux install (download the zip again first):

```bash
sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
```

Windows (PowerShell / CMD):

```powershell
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```

Verify:

```bash
aws --version
```

```bash
which aws
```

Run the CLI from Docker without installing it:

```bash
docker run --rm -it -v ~/.aws:/root/.aws amazon/aws-cli sts get-caller-identity
```

Install the Session Manager plugin (needed for `aws ssm start-session` and `aws ecs execute-command`) on macOS:

```bash
brew install --cask session-manager-plugin
```

```bash
session-manager-plugin
```

---

## 2. Help, Discovery & Auto-prompt

General help:

```bash
aws help
```

Help for a service:

```bash
aws s3 help
```

Help for a command (all parameters + examples):

```bash
aws ec2 describe-instances help
```

Interactive auto-prompt (fuzzy-completes services, commands and parameters):

```bash
aws --cli-auto-prompt
```

```bash
aws ec2 describe-instances --cli-auto-prompt
```

Turn on auto-prompt for partial commands permanently:

```bash
export AWS_CLI_AUTO_PROMPT=on-partial
```

Tab completion — bash (add to `~/.bashrc`):

```bash
complete -C aws_completer aws
```

Tab completion — zsh (add to `~/.zshrc`):

```bash
autoload bashcompinit && bashcompinit
autoload -Uz compinit && compinit
complete -C aws_completer aws
```

Print a JSON skeleton of all parameters for a command:

```bash
aws ec2 run-instances --generate-cli-skeleton > run-instances.json
```

Run a command from a JSON (or YAML) input file:

```bash
aws ec2 run-instances --cli-input-json file://run-instances.json
```

```bash
aws ec2 run-instances --cli-input-yaml file://run-instances.yaml
```

---

## 3. Configure & Profiles

Interactive setup of the `default` profile (access key, secret, region, output):

```bash
aws configure
```

Set up a named profile:

```bash
aws configure --profile prod
```

Show the active configuration **and where each value comes from** (env, config file, SSO…):

```bash
aws configure list
```

```bash
aws configure list --profile prod
```

List all profiles:

```bash
aws configure list-profiles
```

Read / write a single setting:

```bash
aws configure get region --profile prod
```

```bash
aws configure set region eu-central-1 --profile prod
```

```bash
aws configure set output json
```

Use a profile for one command:

```bash
aws s3 ls --profile prod
```

Use a profile for the whole shell session:

```bash
export AWS_PROFILE=prod
```

```bash
unset AWS_PROFILE
```

Override the region for one command:

```bash
aws ec2 describe-instances --region us-east-1
```

Useful environment variables:

| Variable | Purpose |
|---|---|
| `AWS_PROFILE` | Profile to use |
| `AWS_REGION` / `AWS_DEFAULT_REGION` | Region (`AWS_REGION` wins) |
| `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` / `AWS_SESSION_TOKEN` | Static or temporary credentials |
| `AWS_DEFAULT_OUTPUT` | `json`, `text`, `table`, `yaml`, `yaml-stream` |
| `AWS_PAGER` | Pager for output — set to `""` to disable |
| `AWS_CONFIG_FILE` / `AWS_SHARED_CREDENTIALS_FILE` | Custom file locations |
| `AWS_CA_BUNDLE` | Custom CA bundle (corporate proxies) |
| `AWS_MAX_ATTEMPTS` / `AWS_RETRY_MODE` | Retry behaviour (`standard`, `adaptive`) |
| `AWS_CLI_AUTO_PROMPT` | `on` or `on-partial` |

Credential lookup order (first match wins):

1. Command-line options (`--profile`, `--region`)
2. Environment variables (`AWS_ACCESS_KEY_ID`, `AWS_PROFILE`, …)
3. Profile settings: assume-role, web identity, IAM Identity Center (SSO)
4. `~/.aws/credentials`
5. `credential_process` / `~/.aws/config`
6. Container credentials (ECS task role)
7. EC2 instance profile (IMDS)

---

## 4. Config & Credentials Files

| File | Holds |
|---|---|
| `~/.aws/config` | Region, output, SSO, assume-role and other settings. Sections are `[profile name]` |
| `~/.aws/credentials` | Access keys. Sections are `[name]` (no `profile` prefix) |
| `~/.aws/cli/alias` | CLI aliases (see [Aliases](#43-aliases)) |
| `~/.aws/sso/cache/` | Cached SSO tokens |

`~/.aws/credentials` (long-lived keys — prefer SSO or roles when you can):

```ini
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

`~/.aws/config` with an SSO profile, a role-assuming profile and MFA:

```ini
[default]
region = eu-central-1
output = json

[sso-session my-sso]
sso_start_url = https://my-company.awsapps.com/start
sso_region = us-east-1
sso_registration_scopes = sso:account:access

[profile dev]
sso_session = my-sso
sso_account_id = 123456789012
sso_role_name = AdministratorAccess
region = eu-central-1

[profile prod]
role_arn = arn:aws:iam::210987654321:role/Admin
source_profile = default
mfa_serial = arn:aws:iam::123456789012:mfa/my-user
duration_seconds = 3600
region = eu-central-1

[profile localstack]
region = us-east-1
endpoint_url = http://localhost:4566
```

---

## 5. IAM Identity Center (SSO)

Create an SSO profile interactively (opens the browser):

```bash
aws configure sso
```

Create only an `sso-session` block that several profiles can share:

```bash
aws configure sso-session
```

Log in (token is cached in `~/.aws/sso/cache/`):

```bash
aws sso login --profile dev
```

```bash
aws sso login --sso-session my-sso
```

Log in on a machine without a browser (prints a URL + code to open elsewhere):

```bash
aws sso login --profile dev --use-device-code
```

Log out:

```bash
aws sso logout
```

Export the profile's temporary credentials as environment variables (for tools that don't understand SSO profiles):

```bash
eval "$(aws configure export-credentials --profile dev --format env)"
```

---

## 6. Identity, STS & Assume Role

Who am I? (run this first when anything is weird):

```bash
aws sts get-caller-identity
```

Account ID only:

```bash
aws sts get-caller-identity --query Account --output text
```

Assume a role:

```bash
aws sts assume-role --role-arn arn:aws:iam::123456789012:role/Admin --role-session-name cli-session
```

Assume a role and export its credentials into the current shell:

```bash
eval "$(aws sts assume-role --role-arn arn:aws:iam::123456789012:role/Admin --role-session-name cli-session --query 'Credentials.[AccessKeyId,SecretAccessKey,SessionToken]' --output text | awk '{print "export AWS_ACCESS_KEY_ID="$1" AWS_SECRET_ACCESS_KEY="$2" AWS_SESSION_TOKEN="$3}')"
```

…or simply let a profile with `role_arn` do it (see [Config Files](#4-config--credentials-files)) and export it:

```bash
eval "$(aws configure export-credentials --profile prod --format env)"
```

Drop assumed-role credentials:

```bash
unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN
```

Get an MFA session token (for policies that require MFA):

```bash
aws sts get-session-token --serial-number arn:aws:iam::123456789012:mfa/my-user --token-code 123456 --duration-seconds 43200
```

Which account owns this access key?

```bash
aws sts get-access-key-info --access-key-id AKIAIOSFODNN7EXAMPLE
```

Decode an `Encoded authorization failure message` from an `AccessDenied` error:

```bash
aws sts decode-authorization-message --encoded-message <encoded-message> --query DecodedMessage --output text | jq
```

---

## 7. Global Options, Output & Pagination

Output formats:

```bash
aws ec2 describe-instances --output json
```

```bash
aws ec2 describe-instances --output table
```

```bash
aws ec2 describe-instances --output text
```

```bash
aws ec2 describe-instances --output yaml
```

Disable the pager for one command / for good:

```bash
aws ec2 describe-instances --no-cli-pager
```

```bash
export AWS_PAGER=""
```

```bash
aws configure set cli_pager ""
```

Limit how many items are returned (and continue later):

```bash
aws s3api list-objects-v2 --bucket my-bucket --max-items 100
```

```bash
aws s3api list-objects-v2 --bucket my-bucket --max-items 100 --starting-token <NextToken>
```

Smaller pages per API call (avoids timeouts on huge accounts; still returns everything):

```bash
aws ec2 describe-instances --page-size 50
```

Only the first page:

```bash
aws ec2 describe-instances --no-paginate
```

Check permissions without making the change (EC2 only — `DryRunOperation` means it would have succeeded):

```bash
aws ec2 stop-instances --instance-ids i-0123456789abcdef0 --dry-run
```

Read a parameter value from a file (`file://` = text, `fileb://` = binary):

```bash
aws iam put-role-policy --role-name my-role --policy-name s3-read --policy-document file://policy.json
```

Point the CLI at another endpoint (LocalStack, MinIO, VPC endpoint):

```bash
aws s3 ls --endpoint-url http://localhost:4566
```

Increase timeouts for slow or long operations (seconds, `0` = no timeout):

```bash
aws s3 cp big.iso s3://my-bucket/ --cli-read-timeout 0 --cli-connect-timeout 60
```

Full debug log (credentials source, region, endpoint, raw HTTP):

```bash
aws s3 ls --debug
```

---

## 8. Filtering Output with --query (JMESPath)

`--query` runs **client-side** on the response. `--filters` (where a command supports it) runs **server-side** and is faster on big accounts — combine both.

List of values:

```bash
aws ec2 describe-instances --query 'Reservations[].Instances[].InstanceId' --output text
```

Pick fields into a table, including the `Name` tag:

```bash
aws ec2 describe-instances --query 'Reservations[].Instances[].{ID:InstanceId,Name:Tags[?Key==`Name`]|[0].Value,Type:InstanceType,State:State.Name,IP:PrivateIpAddress}' --output table
```

Filter with a condition (backticks are JSON literals):

```bash
aws ec2 describe-instances --query 'Reservations[].Instances[?State.Name==`running`].InstanceId' --output text
```

Server-side filter instead (multiple filters are AND-ed; multiple values are OR-ed):

```bash
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running" "Name=tag:Env,Values=prod,staging"
```

First / last item:

```bash
aws ec2 describe-vpcs --query 'Vpcs[0].VpcId' --output text
```

Sort and take the newest:

```bash
aws ec2 describe-images --owners self --query 'sort_by(Images,&CreationDate)[-1].ImageId' --output text
```

Count:

```bash
aws s3api list-buckets --query 'length(Buckets)'
```

`contains` / `starts_with`:

```bash
aws lambda list-functions --query 'Functions[?contains(FunctionName, `prod`)].FunctionName' --output text
```

```bash
aws s3api list-buckets --query 'Buckets[?starts_with(Name, `logs-`)].Name' --output text
```

Missing / null fields:

```bash
aws ec2 describe-addresses --query 'Addresses[?AssociationId==null].PublicIp' --output text
```

Store a value in a shell variable:

```bash
VPC_ID=$(aws ec2 describe-vpcs --filters Name=is-default,Values=true --query 'Vpcs[0].VpcId' --output text)
```

Use `jq` when JMESPath gets awkward:

```bash
aws ec2 describe-instances --output json | jq -r '.Reservations[].Instances[] | [.InstanceId, .State.Name] | @tsv'
```

---

## 9. S3: Files & Sync

List buckets:

```bash
aws s3 ls
```

List a bucket / prefix:

```bash
aws s3 ls s3://my-bucket/
```

```bash
aws s3 ls s3://my-bucket/uploads/
```

Recursive listing with total size:

```bash
aws s3 ls s3://my-bucket --recursive --human-readable --summarize
```

Create / delete a bucket:

```bash
aws s3 mb s3://my-bucket --region eu-central-1
```

```bash
aws s3 rb s3://my-bucket
```

Upload / download a file:

```bash
aws s3 cp file.txt s3://my-bucket/
```

```bash
aws s3 cp s3://my-bucket/file.txt .
```

Upload / download a directory:

```bash
aws s3 cp ./dist s3://my-bucket/ --recursive
```

```bash
aws s3 cp s3://my-bucket/backups/ ./backups --recursive
```

Copy between buckets (server-side):

```bash
aws s3 cp s3://source-bucket/file.txt s3://target-bucket/file.txt
```

Stream from stdin / to stdout:

```bash
echo "hello" | aws s3 cp - s3://my-bucket/hello.txt
```

```bash
aws s3 cp s3://my-bucket/app.log - | tail -n 50
```

Move / rename:

```bash
aws s3 mv s3://my-bucket/old.txt s3://my-bucket/new.txt
```

Delete a file / a whole prefix:

```bash
aws s3 rm s3://my-bucket/file.txt
```

```bash
aws s3 rm s3://my-bucket/tmp/ --recursive
```

Sync a directory to S3 (only changed files):

```bash
aws s3 sync ./dist s3://my-bucket/
```

Mirror exactly — also delete remote files that no longer exist locally (preview first!):

```bash
aws s3 sync ./dist s3://my-bucket/ --delete --dryrun
```

```bash
aws s3 sync ./dist s3://my-bucket/ --delete
```

Sync S3 → local / S3 → S3:

```bash
aws s3 sync s3://my-bucket/ ./my-bucket-backup
```

```bash
aws s3 sync s3://source-bucket s3://target-bucket
```

Include / exclude patterns (order matters — later rules win):

```bash
aws s3 sync . s3://my-bucket/images/ --exclude "*" --include "*.jpg" --include "*.png"
```

Storage class and encryption:

```bash
aws s3 cp backup.tar.gz s3://my-bucket/ --storage-class STANDARD_IA
```

```bash
aws s3 cp backup.tar.gz s3://my-bucket/ --sse aws:kms --sse-kms-key-id <kms-key-id>
```

Headers for static sites:

```bash
aws s3 cp index.html s3://my-bucket/index.html --cache-control "no-cache" --content-type "text/html"
```

Temporary download link (seconds, max 7 days):

```bash
aws s3 presign s3://my-bucket/report.pdf --expires-in 3600
```

Enable static website hosting:

```bash
aws s3 website s3://my-bucket/ --index-document index.html --error-document error.html
```

Faster transfers for many/large files:

```bash
aws configure set default.s3.max_concurrent_requests 20
```

```bash
aws configure set default.s3.multipart_chunksize 64MB
```

---

## 10. S3: Buckets & Settings (s3api)

Create a bucket (outside `us-east-1` a `LocationConstraint` is required):

```bash
aws s3api create-bucket --bucket my-bucket --region eu-central-1 --create-bucket-configuration LocationConstraint=eu-central-1
```

Does the bucket exist and can I access it?

```bash
aws s3api head-bucket --bucket my-bucket
```

Bucket region:

```bash
aws s3api get-bucket-location --bucket my-bucket
```

Block all public access:

```bash
aws s3api put-public-access-block --bucket my-bucket --public-access-block-configuration BlockPublicAcls=true,IgnorePublicAcls=true,BlockPublicPolicy=true,RestrictPublicBuckets=true
```

```bash
aws s3api get-public-access-block --bucket my-bucket
```

Enable versioning:

```bash
aws s3api put-bucket-versioning --bucket my-bucket --versioning-configuration Status=Enabled
```

```bash
aws s3api get-bucket-versioning --bucket my-bucket
```

Default encryption with a KMS key:

```bash
aws s3api put-bucket-encryption --bucket my-bucket --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"aws:kms","KMSMasterKeyID":"arn:aws:kms:eu-central-1:123456789012:key/<key-id>"},"BucketKeyEnabled":true}]}'
```

Bucket policy:

```bash
aws s3api get-bucket-policy --bucket my-bucket --query Policy --output text | jq
```

```bash
aws s3api put-bucket-policy --bucket my-bucket --policy file://bucket-policy.json
```

Lifecycle rules:

```bash
aws s3api put-bucket-lifecycle-configuration --bucket my-bucket --lifecycle-configuration file://lifecycle.json
```

`lifecycle.json` — move logs to cheaper storage, expire them, clean old versions and failed uploads:

```json
{
  "Rules": [
    {
      "ID": "logs-tiering",
      "Status": "Enabled",
      "Filter": { "Prefix": "logs/" },
      "Transitions": [{ "Days": 30, "StorageClass": "STANDARD_IA" }],
      "Expiration": { "Days": 365 }
    },
    {
      "ID": "cleanup",
      "Status": "Enabled",
      "Filter": {},
      "NoncurrentVersionExpiration": { "NoncurrentDays": 30 },
      "AbortIncompleteMultipartUpload": { "DaysAfterInitiation": 7 }
    }
  ]
}
```

Object metadata:

```bash
aws s3api head-object --bucket my-bucket --key uploads/file.pdf
```

List objects with details:

```bash
aws s3api list-objects-v2 --bucket my-bucket --prefix logs/ --query 'Contents[].{Key:Key,Size:Size,Modified:LastModified}' --output table
```

Ten largest objects:

```bash
aws s3api list-objects-v2 --bucket my-bucket --query 'reverse(sort_by(Contents,&Size))[:10].[Key,Size]' --output table
```

Objects modified after a date:

```bash
aws s3api list-objects-v2 --bucket my-bucket --query "Contents[?LastModified>='2026-01-01'].Key" --output text
```

Versions of an object:

```bash
aws s3api list-object-versions --bucket my-bucket --prefix config.json --query 'Versions[].{Version:VersionId,Latest:IsLatest,Modified:LastModified}' --output table
```

Download an older version:

```bash
aws s3api get-object --bucket my-bucket --key config.json --version-id <version-id> config.json
```

Tags:

```bash
aws s3api put-bucket-tagging --bucket my-bucket --tagging 'TagSet=[{Key=Env,Value=prod},{Key=Project,Value=myapp}]'
```

---

## 11. EC2: Instances

Instances as a readable table:

```bash
aws ec2 describe-instances --query 'Reservations[].Instances[].{ID:InstanceId,Name:Tags[?Key==`Name`]|[0].Value,Type:InstanceType,State:State.Name,PrivateIP:PrivateIpAddress,PublicIP:PublicIpAddress}' --output table
```

Running instances only:

```bash
aws ec2 describe-instances --filters Name=instance-state-name,Values=running --query 'Reservations[].Instances[].InstanceId' --output text
```

By `Name` tag (wildcards allowed):

```bash
aws ec2 describe-instances --filters "Name=tag:Name,Values=web-*"
```

One instance:

```bash
aws ec2 describe-instances --instance-ids i-0123456789abcdef0
```

Public IP of an instance:

```bash
aws ec2 describe-instances --instance-ids i-0123456789abcdef0 --query 'Reservations[].Instances[].PublicIpAddress' --output text
```

Start / stop / reboot / terminate:

```bash
aws ec2 start-instances --instance-ids i-0123456789abcdef0
```

```bash
aws ec2 stop-instances --instance-ids i-0123456789abcdef0
```

```bash
aws ec2 reboot-instances --instance-ids i-0123456789abcdef0
```

```bash
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0
```

Wait until it is running:

```bash
aws ec2 wait instance-running --instance-ids i-0123456789abcdef0
```

Status checks (system + instance):

```bash
aws ec2 describe-instance-status --instance-ids i-0123456789abcdef0 --include-all-instances
```

Launch an instance (IMDSv2 enforced, tagged, with user data):

```bash
aws ec2 run-instances --image-id ami-0123456789abcdef0 --instance-type t3.micro --key-name my-key --subnet-id subnet-0123456789abcdef0 --security-group-ids sg-0123456789abcdef0 --iam-instance-profile Name=my-instance-profile --metadata-options HttpTokens=required --user-data file://user-data.sh --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=web-1},{Key=Env,Value=prod}]' --count 1
```

Change the instance type (instance must be stopped):

```bash
aws ec2 modify-instance-attribute --instance-id i-0123456789abcdef0 --instance-type Value=t3.large
```

Serial console output (boot problems):

```bash
aws ec2 get-console-output --instance-id i-0123456789abcdef0 --latest --output text
```

Add / overwrite tags:

```bash
aws ec2 create-tags --resources i-0123456789abcdef0 --tags Key=Env,Value=prod Key=Owner,Value=platform
```

Instance-type specs:

```bash
aws ec2 describe-instance-types --instance-types t3.micro m7g.large --query 'InstanceTypes[].{Type:InstanceType,vCPU:VCpuInfo.DefaultVCpus,MemMiB:MemoryInfo.SizeInMiB,Arch:ProcessorInfo.SupportedArchitectures[0]}' --output table
```

Regions and availability zones:

```bash
aws ec2 describe-regions --query 'Regions[].RegionName' --output text
```

```bash
aws ec2 describe-availability-zones --query 'AvailabilityZones[].ZoneName' --output text
```

---

## 12. EC2: AMIs, Key Pairs, Volumes, Snapshots & Elastic IPs

Latest Amazon Linux 2023 AMI (via public SSM parameter):

```bash
aws ssm get-parameter --name /aws/service/ami-amazon-linux-latest/al2023-ami-kernel-default-x86_64 --query Parameter.Value --output text
```

Latest Ubuntu 24.04 AMI (Canonical's account):

```bash
aws ec2 describe-images --owners 099720109477 --filters "Name=name,Values=ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*" --query 'sort_by(Images,&CreationDate)[-1].ImageId' --output text
```

Your own AMIs:

```bash
aws ec2 describe-images --owners self --query 'Images[].{ID:ImageId,Name:Name,Created:CreationDate}' --output table
```

Create an AMI from an instance:

```bash
aws ec2 create-image --instance-id i-0123456789abcdef0 --name "web-1-$(date +%Y%m%d)" --no-reboot
```

Deregister an AMI:

```bash
aws ec2 deregister-image --image-id ami-0123456789abcdef0
```

Create a key pair and save the private key:

```bash
aws ec2 create-key-pair --key-name my-key --key-type ed25519 --query KeyMaterial --output text > my-key.pem && chmod 400 my-key.pem
```

Import your existing public key:

```bash
aws ec2 import-key-pair --key-name my-key --public-key-material fileb://~/.ssh/id_ed25519.pub
```

```bash
aws ec2 describe-key-pairs --query 'KeyPairs[].KeyName' --output text
```

```bash
aws ec2 delete-key-pair --key-name my-key
```

Volumes:

```bash
aws ec2 describe-volumes --query 'Volumes[].{ID:VolumeId,Size:Size,Type:VolumeType,State:State,Instance:Attachments[0].InstanceId}' --output table
```

Unattached volumes (you still pay for them):

```bash
aws ec2 describe-volumes --filters Name=status,Values=available --query 'Volumes[].[VolumeId,Size,CreateTime]' --output table
```

Create, attach and resize a volume:

```bash
aws ec2 create-volume --availability-zone eu-central-1a --size 50 --volume-type gp3
```

```bash
aws ec2 attach-volume --volume-id vol-0123456789abcdef0 --instance-id i-0123456789abcdef0 --device /dev/sdf
```

```bash
aws ec2 modify-volume --volume-id vol-0123456789abcdef0 --size 100
```

Snapshots:

```bash
aws ec2 create-snapshot --volume-id vol-0123456789abcdef0 --description "web-1 before upgrade" --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=web-1-pre-upgrade}]'
```

```bash
aws ec2 describe-snapshots --owner-ids self --query 'Snapshots[].{ID:SnapshotId,Volume:VolumeId,Started:StartTime,Size:VolumeSize}' --output table
```

```bash
aws ec2 delete-snapshot --snapshot-id snap-0123456789abcdef0
```

Elastic IPs:

```bash
aws ec2 allocate-address --domain vpc
```

```bash
aws ec2 associate-address --instance-id i-0123456789abcdef0 --allocation-id eipalloc-0123456789abcdef0
```

Unassociated Elastic IPs (billed while idle):

```bash
aws ec2 describe-addresses --query 'Addresses[?AssociationId==null].[PublicIp,AllocationId]' --output table
```

```bash
aws ec2 release-address --allocation-id eipalloc-0123456789abcdef0
```

---

## 13. VPC & Security Groups

VPCs:

```bash
aws ec2 describe-vpcs --query 'Vpcs[].{ID:VpcId,CIDR:CidrBlock,Default:IsDefault,Name:Tags[?Key==`Name`]|[0].Value}' --output table
```

Subnets of a VPC:

```bash
aws ec2 describe-subnets --filters Name=vpc-id,Values=vpc-0123456789abcdef0 --query 'Subnets[].{ID:SubnetId,AZ:AvailabilityZone,CIDR:CidrBlock,PublicIP:MapPublicIpOnLaunch,FreeIPs:AvailableIpAddressCount}' --output table
```

Route tables, internet and NAT gateways:

```bash
aws ec2 describe-route-tables --filters Name=vpc-id,Values=vpc-0123456789abcdef0
```

```bash
aws ec2 describe-internet-gateways --filters Name=attachment.vpc-id,Values=vpc-0123456789abcdef0
```

```bash
aws ec2 describe-nat-gateways --query 'NatGateways[].{ID:NatGatewayId,VPC:VpcId,State:State,Subnet:SubnetId}' --output table
```

Who owns this private IP?

```bash
aws ec2 describe-network-interfaces --filters Name=addresses.private-ip-address,Values=10.0.1.25 --query 'NetworkInterfaces[].{ENI:NetworkInterfaceId,Type:InterfaceType,Desc:Description,Instance:Attachment.InstanceId}'
```

Security groups in a VPC:

```bash
aws ec2 describe-security-groups --filters Name=vpc-id,Values=vpc-0123456789abcdef0 --query 'SecurityGroups[].{ID:GroupId,Name:GroupName}' --output table
```

Rules of one group:

```bash
aws ec2 describe-security-group-rules --filters Name=group-id,Values=sg-0123456789abcdef0 --query 'SecurityGroupRules[].{Egress:IsEgress,Proto:IpProtocol,From:FromPort,To:ToPort,CIDR:CidrIpv4,SG:ReferencedGroupInfo.GroupId}' --output table
```

Create a group:

```bash
aws ec2 create-security-group --group-name web-sg --description "Web servers" --vpc-id vpc-0123456789abcdef0
```

Allow HTTPS from anywhere:

```bash
aws ec2 authorize-security-group-ingress --group-id sg-0123456789abcdef0 --protocol tcp --port 443 --cidr 0.0.0.0/0
```

Allow SSH from **your current IP** only:

```bash
aws ec2 authorize-security-group-ingress --group-id sg-0123456789abcdef0 --protocol tcp --port 22 --cidr "$(curl -s https://checkip.amazonaws.com)/32"
```

Allow traffic from another security group (e.g. app → database):

```bash
aws ec2 authorize-security-group-ingress --group-id sg-0aaaaaaaaaaaaaaaa --protocol tcp --port 5432 --source-group sg-0bbbbbbbbbbbbbbbb
```

Remove a rule:

```bash
aws ec2 revoke-security-group-ingress --group-id sg-0123456789abcdef0 --protocol tcp --port 22 --cidr 203.0.113.10/32
```

Find groups with SSH open to the world:

```bash
aws ec2 describe-security-groups --filters Name=ip-permission.from-port,Values=22 Name=ip-permission.cidr,Values=0.0.0.0/0 --query 'SecurityGroups[].[GroupId,GroupName]' --output table
```

Delete a group:

```bash
aws ec2 delete-security-group --group-id sg-0123456789abcdef0
```

---

## 14. SSM: Session Manager, Run Command & Parameter Store

> Session Manager needs the [Session Manager plugin](#1-install--update) locally, the SSM agent on the instance and the `AmazonSSMManagedInstanceCore` policy on its role. No SSH keys or open port 22 required.

Instances registered with SSM:

```bash
aws ssm describe-instance-information --query 'InstanceInformationList[].{ID:InstanceId,Ping:PingStatus,OS:PlatformName,Agent:AgentVersion}' --output table
```

Open a shell on an instance:

```bash
aws ssm start-session --target i-0123456789abcdef0
```

Forward a port of the instance to your machine (`localhost:8080` → instance `:80`):

```bash
aws ssm start-session --target i-0123456789abcdef0 --document-name AWS-StartPortForwardingSession --parameters '{"portNumber":["80"],"localPortNumber":["8080"]}'
```

Reach a private database **through** an instance (`localhost:15432` → RDS `:5432`):

```bash
aws ssm start-session --target i-0123456789abcdef0 --document-name AWS-StartPortForwardingSessionToRemoteHost --parameters '{"host":["mydb.abcdefghijkl.eu-central-1.rds.amazonaws.com"],"portNumber":["5432"],"localPortNumber":["15432"]}'
```

Run a shell command on instances (by ID or by tag):

```bash
aws ssm send-command --instance-ids i-0123456789abcdef0 --document-name AWS-RunShellScript --parameters 'commands=["uptime","df -h"]' --query Command.CommandId --output text
```

```bash
aws ssm send-command --targets Key=tag:Env,Values=prod --document-name AWS-RunShellScript --parameters 'commands=["sudo systemctl restart nginx"]'
```

Get the command's output:

```bash
aws ssm get-command-invocation --command-id <command-id> --instance-id i-0123456789abcdef0 --query StandardOutputContent --output text
```

**Parameter Store** — create / update:

```bash
aws ssm put-parameter --name /myapp/prod/DB_HOST --value db.internal --type String
```

```bash
aws ssm put-parameter --name /myapp/prod/DB_PASSWORD --value file://db-password.txt --type SecureString --overwrite
```

Read one value (decrypted):

```bash
aws ssm get-parameter --name /myapp/prod/DB_PASSWORD --with-decryption --query Parameter.Value --output text
```

Read a whole path:

```bash
aws ssm get-parameters-by-path --path /myapp/prod/ --recursive --with-decryption --query 'Parameters[].[Name,Value]' --output text
```

Turn a path into a `.env` file:

```bash
aws ssm get-parameters-by-path --path /myapp/prod/ --with-decryption --query 'Parameters[].[Name,Value]' --output text | awk -F'\t' '{n=$1; sub(".*/", "", n); print n"="$2}' > .env
```

Search, history, delete:

```bash
aws ssm describe-parameters --parameter-filters Key=Name,Option=BeginsWith,Values=/myapp/ --query 'Parameters[].Name' --output text
```

```bash
aws ssm get-parameter-history --name /myapp/prod/DB_HOST --query 'Parameters[].[Version,LastModifiedDate,LastModifiedUser]' --output table
```

```bash
aws ssm delete-parameter --name /myapp/prod/DB_HOST
```

---

## 15. IAM

Users:

```bash
aws iam list-users --query 'Users[].[UserName,CreateDate,PasswordLastUsed]' --output table
```

```bash
aws iam create-user --user-name ci-deploy
```

```bash
aws iam add-user-to-group --user-name ci-deploy --group-name deployers
```

Policies attached to a user:

```bash
aws iam list-attached-user-policies --user-name ci-deploy
```

```bash
aws iam attach-user-policy --user-name ci-deploy --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

```bash
aws iam detach-user-policy --user-name ci-deploy --policy-arn arn:aws:iam::aws:policy/ReadOnlyAccess
```

Access keys — list, last used, create, disable, delete:

```bash
aws iam list-access-keys --user-name ci-deploy
```

```bash
aws iam get-access-key-last-used --access-key-id AKIAIOSFODNN7EXAMPLE
```

```bash
aws iam create-access-key --user-name ci-deploy
```

```bash
aws iam update-access-key --user-name ci-deploy --access-key-id AKIAIOSFODNN7EXAMPLE --status Inactive
```

```bash
aws iam delete-access-key --user-name ci-deploy --access-key-id AKIAIOSFODNN7EXAMPLE
```

MFA devices of a user:

```bash
aws iam list-mfa-devices --user-name my-user
```

Roles:

```bash
aws iam list-roles --query 'Roles[].[RoleName,Arn]' --output table
```

```bash
aws iam get-role --role-name my-role
```

Create a role with a trust policy:

```bash
aws iam create-role --role-name my-lambda-role --assume-role-policy-document file://trust.json
```

`trust.json` (lets Lambda assume the role):

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "Service": "lambda.amazonaws.com" },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Attach a managed policy / add an inline policy:

```bash
aws iam attach-role-policy --role-name my-lambda-role --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

```bash
aws iam put-role-policy --role-name my-lambda-role --policy-name s3-read --policy-document file://s3-read.json
```

What is attached to a role?

```bash
aws iam list-attached-role-policies --role-name my-role
```

```bash
aws iam list-role-policies --role-name my-role
```

```bash
aws iam get-role-policy --role-name my-role --policy-name s3-read
```

Customer-managed policies and their current document:

```bash
aws iam list-policies --scope Local --query 'Policies[].[PolicyName,Arn,DefaultVersionId]' --output table
```

```bash
aws iam create-policy --policy-name myapp-s3 --policy-document file://myapp-s3.json
```

```bash
aws iam get-policy-version --policy-arn arn:aws:iam::123456789012:policy/myapp-s3 --version-id v1 --query PolicyVersion.Document
```

Instance profile for EC2:

```bash
aws iam create-instance-profile --instance-profile-name my-instance-profile
```

```bash
aws iam add-role-to-instance-profile --instance-profile-name my-instance-profile --role-name my-ec2-role
```

Will this principal be allowed to do X?

```bash
aws iam simulate-principal-policy --policy-source-arn arn:aws:iam::123456789012:role/my-role --action-names s3:GetObject s3:PutObject --resource-arns "arn:aws:s3:::my-bucket/*" --query 'EvaluationResults[].[EvalActionName,EvalDecision]' --output table
```

Credential report (all users, keys, MFA, last activity) as CSV:

```bash
aws iam generate-credential-report
```

```bash
aws iam get-credential-report --query Content --output text | base64 --decode > credential-report.csv
```

Account alias:

```bash
aws iam list-account-aliases
```

---

## 16. ECR (Container Registry)

Log Docker in to ECR:

```bash
aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 123456789012.dkr.ecr.eu-central-1.amazonaws.com
```

Log in to ECR Public (always `us-east-1`):

```bash
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws
```

Create a repository (scan on push, immutable tags):

```bash
aws ecr create-repository --repository-name myapp --image-scanning-configuration scanOnPush=true --image-tag-mutability IMMUTABLE
```

Repository URIs:

```bash
aws ecr describe-repositories --query 'repositories[].repositoryUri' --output text
```

Tag and push an image:

```bash
docker tag myapp:1.4.0 123456789012.dkr.ecr.eu-central-1.amazonaws.com/myapp:1.4.0
```

```bash
docker push 123456789012.dkr.ecr.eu-central-1.amazonaws.com/myapp:1.4.0
```

List images / last five pushed:

```bash
aws ecr list-images --repository-name myapp
```

```bash
aws ecr describe-images --repository-name myapp --query 'sort_by(imageDetails,&imagePushedAt)[-5:].{Tags:join(`,`,imageTags||`[]`),Pushed:imagePushedAt,Bytes:imageSizeInBytes}' --output table
```

Most recent tag:

```bash
aws ecr describe-images --repository-name myapp --query 'sort_by(imageDetails,&imagePushedAt)[-1].imageTags[0]' --output text
```

Delete an image by tag:

```bash
aws ecr batch-delete-image --repository-name myapp --image-ids imageTag=1.0.0
```

Delete all untagged images:

```bash
aws ecr list-images --repository-name myapp --filter tagStatus=UNTAGGED --query 'imageIds[*]' --output json > untagged.json
```

```bash
aws ecr batch-delete-image --repository-name myapp --image-ids file://untagged.json
```

Lifecycle policy (do it once instead of cleaning up by hand):

```bash
aws ecr put-lifecycle-policy --repository-name myapp --lifecycle-policy-text file://ecr-lifecycle.json
```

`ecr-lifecycle.json`:

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Expire untagged images after 7 days",
      "selection": { "tagStatus": "untagged", "countType": "sinceImagePushed", "countUnit": "days", "countNumber": 7 },
      "action": { "type": "expire" }
    },
    {
      "rulePriority": 2,
      "description": "Keep only the last 30 images",
      "selection": { "tagStatus": "any", "countType": "imageCountMoreThan", "countNumber": 30 },
      "action": { "type": "expire" }
    }
  ]
}
```

Vulnerability scan results:

```bash
aws ecr start-image-scan --repository-name myapp --image-id imageTag=1.4.0
```

```bash
aws ecr wait image-scan-complete --repository-name myapp --image-id imageTag=1.4.0
```

```bash
aws ecr describe-image-scan-findings --repository-name myapp --image-id imageTag=1.4.0 --query 'imageScanFindings.findingSeverityCounts'
```

Delete a repository and all its images:

```bash
aws ecr delete-repository --repository-name myapp --force
```

---

## 17. ECS

Clusters and services:

```bash
aws ecs list-clusters
```

```bash
aws ecs list-services --cluster my-cluster
```

Service health at a glance:

```bash
aws ecs describe-services --cluster my-cluster --services my-service --query 'services[].{Status:status,Desired:desiredCount,Running:runningCount,Pending:pendingCount,TaskDef:taskDefinition}' --output table
```

Deployments in progress:

```bash
aws ecs describe-services --cluster my-cluster --services my-service --query 'services[0].deployments[].{Status:status,Rollout:rolloutState,TaskDef:taskDefinition,Running:runningCount,Desired:desiredCount}' --output table
```

Latest service events (why tasks fail to start/register):

```bash
aws ecs describe-services --cluster my-cluster --services my-service --query 'services[0].events[:10].[createdAt,message]' --output text
```

Redeploy (pull the same tag again, replace all tasks):

```bash
aws ecs update-service --cluster my-cluster --service my-service --force-new-deployment
```

Scale:

```bash
aws ecs update-service --cluster my-cluster --service my-service --desired-count 3
```

Deploy a specific task-definition revision (also how you roll back):

```bash
aws ecs update-service --cluster my-cluster --service my-service --task-definition my-task:42
```

Wait for the deployment to finish:

```bash
aws ecs wait services-stable --cluster my-cluster --services my-service
```

Tasks — running and recently stopped:

```bash
aws ecs list-tasks --cluster my-cluster --service-name my-service
```

```bash
aws ecs list-tasks --cluster my-cluster --desired-status STOPPED
```

Why did a task stop?

```bash
aws ecs describe-tasks --cluster my-cluster --tasks <task-id> --query 'tasks[].{Last:lastStatus,StopCode:stopCode,Reason:stoppedReason,Containers:containers[].{Name:name,Exit:exitCode,Reason:reason}}'
```

Shell into a running container (ECS Exec):

```bash
aws ecs execute-command --cluster my-cluster --task <task-id> --container app --interactive --command "/bin/sh"
```

Enable ECS Exec on a service (task role also needs `ssmmessages:*` permissions):

```bash
aws ecs update-service --cluster my-cluster --service my-service --enable-execute-command --force-new-deployment
```

Task definitions:

```bash
aws ecs list-task-definitions --family-prefix my-task --sort DESC --max-items 5
```

Export the current task definition as a reusable input file:

```bash
aws ecs describe-task-definition --task-definition my-task --query taskDefinition | jq 'del(.taskDefinitionArn,.revision,.status,.requiresAttributes,.compatibilities,.registeredAt,.registeredBy)' > taskdef.json
```

```bash
aws ecs register-task-definition --cli-input-json file://taskdef.json
```

Run a one-off Fargate task (e.g. a database migration):

```bash
aws ecs run-task --cluster my-cluster --launch-type FARGATE --task-definition my-task --network-configuration 'awsvpcConfiguration={subnets=[subnet-0123456789abcdef0],securityGroups=[sg-0123456789abcdef0],assignPublicIp=DISABLED}' --overrides '{"containerOverrides":[{"name":"app","command":["php","artisan","migrate","--force"]}]}'
```

Stop a task:

```bash
aws ecs stop-task --cluster my-cluster --task <task-id> --reason "Manual stop"
```

Logs of a service (awslogs driver):

```bash
aws logs tail /ecs/my-service --follow
```

---

## 18. EKS

Clusters:

```bash
aws eks list-clusters
```

```bash
aws eks describe-cluster --name my-cluster --query 'cluster.{Status:status,Version:version,Endpoint:endpoint,PlatformVersion:platformVersion}'
```

Configure `kubectl`:

```bash
aws eks update-kubeconfig --name my-cluster --region eu-central-1
```

With a short context name and/or through a role:

```bash
aws eks update-kubeconfig --name my-cluster --region eu-central-1 --alias prod --role-arn arn:aws:iam::123456789012:role/eks-admin
```

```bash
kubectl get nodes
```

Managed node groups:

```bash
aws eks list-nodegroups --cluster-name my-cluster
```

```bash
aws eks describe-nodegroup --cluster-name my-cluster --nodegroup-name my-nodes --query 'nodegroup.{Status:status,Scaling:scalingConfig,Types:instanceTypes,Version:version,Release:releaseVersion}'
```

Scale a node group:

```bash
aws eks update-nodegroup-config --cluster-name my-cluster --nodegroup-name my-nodes --scaling-config minSize=2,maxSize=6,desiredSize=3
```

Upgrade a node group to the latest AMI for its Kubernetes version:

```bash
aws eks update-nodegroup-version --cluster-name my-cluster --nodegroup-name my-nodes
```

Upgrade readiness (deprecated APIs, add-on compatibility):

```bash
aws eks list-insights --cluster-name my-cluster
```

Upgrade the control plane:

```bash
aws eks update-cluster-version --name my-cluster --kubernetes-version 1.34
```

Add-ons:

```bash
aws eks list-addons --cluster-name my-cluster
```

```bash
aws eks describe-addon-versions --addon-name vpc-cni --kubernetes-version 1.34 --query 'addons[].addonVersions[].addonVersion' --output text
```

```bash
aws eks update-addon --cluster-name my-cluster --addon-name vpc-cni --addon-version <version> --resolve-conflicts OVERWRITE
```

Grant an IAM role access to the cluster (access entries):

```bash
aws eks list-access-entries --cluster-name my-cluster
```

```bash
aws eks create-access-entry --cluster-name my-cluster --principal-arn arn:aws:iam::123456789012:role/developers
```

```bash
aws eks associate-access-policy --cluster-name my-cluster --principal-arn arn:aws:iam::123456789012:role/developers --policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSViewPolicy --access-scope type=cluster
```

Raw token (what `kubectl` uses under the hood):

```bash
aws eks get-token --cluster-name my-cluster
```

---

## 19. Lambda

Functions:

```bash
aws lambda list-functions --query 'Functions[].{Name:FunctionName,Runtime:Runtime,Memory:MemorySize,Timeout:Timeout,Modified:LastModified}' --output table
```

```bash
aws lambda get-function-configuration --function-name my-fn
```

Invoke with a JSON payload:

```bash
aws lambda invoke --function-name my-fn --cli-binary-format raw-in-base64-out --payload '{"name":"test"}' response.json && cat response.json
```

Invoke and print the last 4 KB of its log:

```bash
aws lambda invoke --function-name my-fn --cli-binary-format raw-in-base64-out --payload '{}' --log-type Tail --query LogResult --output text response.json | base64 --decode
```

Invoke asynchronously:

```bash
aws lambda invoke --function-name my-fn --invocation-type Event --cli-binary-format raw-in-base64-out --payload '{}' response.json
```

Deploy code — zip, S3 or container image:

```bash
zip -r function.zip . -x "*.git*"
```

```bash
aws lambda update-function-code --function-name my-fn --zip-file fileb://function.zip
```

```bash
aws lambda update-function-code --function-name my-fn --s3-bucket my-artifacts --s3-key my-fn/function.zip
```

```bash
aws lambda update-function-code --function-name my-fn --image-uri 123456789012.dkr.ecr.eu-central-1.amazonaws.com/my-fn:1.4.0
```

Wait until the update is applied:

```bash
aws lambda wait function-updated --function-name my-fn
```

Environment variables (**replaces the whole set**):

```bash
aws lambda update-function-configuration --function-name my-fn --environment 'Variables={APP_ENV=production,LOG_LEVEL=info}'
```

Memory and timeout:

```bash
aws lambda update-function-configuration --function-name my-fn --memory-size 1024 --timeout 30
```

Versions and aliases:

```bash
aws lambda publish-version --function-name my-fn --description "1.4.0"
```

```bash
aws lambda create-alias --function-name my-fn --name live --function-version 5
```

```bash
aws lambda update-alias --function-name my-fn --name live --function-version 6
```

Reserved concurrency (`0` = stop all invocations — an emergency off switch):

```bash
aws lambda put-function-concurrency --function-name my-fn --reserved-concurrent-executions 10
```

```bash
aws lambda delete-function-concurrency --function-name my-fn
```

Triggers (SQS, Kinesis, DynamoDB streams) — list and pause:

```bash
aws lambda list-event-source-mappings --function-name my-fn
```

```bash
aws lambda update-event-source-mapping --uuid <mapping-uuid> --no-enabled
```

Function URL:

```bash
aws lambda create-function-url-config --function-name my-fn --auth-type AWS_IAM
```

Logs:

```bash
aws logs tail /aws/lambda/my-fn --follow
```

---

## 20. CloudWatch Logs

Log groups with retention and size:

```bash
aws logs describe-log-groups --query 'logGroups[].{Name:logGroupName,RetentionDays:retentionInDays,Bytes:storedBytes}' --output table
```

```bash
aws logs describe-log-groups --log-group-name-prefix /aws/lambda/ --query 'logGroups[].logGroupName' --output text
```

Tail logs live:

```bash
aws logs tail /aws/lambda/my-fn --follow
```

Last hour, compact format:

```bash
aws logs tail /aws/lambda/my-fn --since 1h --format short
```

Only matching lines:

```bash
aws logs tail /aws/lambda/my-fn --since 2h --filter-pattern ERROR
```

JSON logs — filter on a field:

```bash
aws logs tail /ecs/my-service --since 30m --filter-pattern '{ $.level = "error" }'
```

Newest log streams:

```bash
aws logs describe-log-streams --log-group-name /aws/lambda/my-fn --order-by LastEventTime --descending --max-items 5
```

Search events in the last hour (start time is in milliseconds):

```bash
aws logs filter-log-events --log-group-name /aws/lambda/my-fn --filter-pattern '"Task timed out"' --start-time "$(( $(date +%s) - 3600 ))000"
```

Logs Insights query:

```bash
QUERY_ID=$(aws logs start-query --log-group-name /aws/lambda/my-fn --start-time "$(( $(date +%s) - 3600 ))" --end-time "$(date +%s)" --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc | limit 20' --query queryId --output text)
```

```bash
aws logs get-query-results --query-id "$QUERY_ID"
```

Set retention (log groups keep data forever by default — and you pay for it):

```bash
aws logs put-retention-policy --log-group-name /aws/lambda/my-fn --retention-in-days 30
```

Set 30-day retention on every group that has none:

```bash
for g in $(aws logs describe-log-groups --query 'logGroups[?!retentionInDays].logGroupName' --output text); do aws logs put-retention-policy --log-group-name "$g" --retention-in-days 30; done
```

Create / delete a group:

```bash
aws logs create-log-group --log-group-name /myapp/prod
```

```bash
aws logs delete-log-group --log-group-name /myapp/old
```

---

## 21. CloudWatch Metrics & Alarms

Available metrics:

```bash
aws cloudwatch list-metrics --namespace AWS/EC2 --metric-name CPUUtilization
```

CPU of an instance (5-minute average and maximum):

```bash
aws cloudwatch get-metric-statistics --namespace AWS/EC2 --metric-name CPUUtilization --dimensions Name=InstanceId,Value=i-0123456789abcdef0 --start-time 2026-01-01T00:00:00Z --end-time 2026-01-01T06:00:00Z --period 300 --statistics Average Maximum --output table
```

Alarms currently firing:

```bash
aws cloudwatch describe-alarms --state-value ALARM --query 'MetricAlarms[].[AlarmName,StateReason]' --output table
```

Create a CPU alarm that notifies an SNS topic:

```bash
aws cloudwatch put-metric-alarm --alarm-name web-1-high-cpu --namespace AWS/EC2 --metric-name CPUUtilization --dimensions Name=InstanceId,Value=i-0123456789abcdef0 --statistic Average --period 300 --evaluation-periods 2 --threshold 80 --comparison-operator GreaterThanThreshold --alarm-actions arn:aws:sns:eu-central-1:123456789012:alerts
```

Test the notification path by forcing a state:

```bash
aws cloudwatch set-alarm-state --alarm-name web-1-high-cpu --state-value ALARM --state-reason "Testing notifications"
```

Silence / unsilence alarm actions (maintenance window):

```bash
aws cloudwatch disable-alarm-actions --alarm-names web-1-high-cpu
```

```bash
aws cloudwatch enable-alarm-actions --alarm-names web-1-high-cpu
```

Publish a custom metric:

```bash
aws cloudwatch put-metric-data --namespace MyApp --metric-name QueueBacklog --value 42 --unit Count
```

Delete alarms:

```bash
aws cloudwatch delete-alarms --alarm-names web-1-high-cpu
```

---

## 22. RDS & Aurora

Instances:

```bash
aws rds describe-db-instances --query 'DBInstances[].{ID:DBInstanceIdentifier,Engine:Engine,Version:EngineVersion,Class:DBInstanceClass,Status:DBInstanceStatus,MultiAZ:MultiAZ,Endpoint:Endpoint.Address}' --output table
```

Aurora clusters (writer + reader endpoints):

```bash
aws rds describe-db-clusters --query 'DBClusters[].{ID:DBClusterIdentifier,Engine:Engine,Status:Status,Writer:Endpoint,Reader:ReaderEndpoint}' --output table
```

Start / stop (a stopped instance starts again automatically after 7 days):

```bash
aws rds start-db-instance --db-instance-identifier mydb
```

```bash
aws rds stop-db-instance --db-instance-identifier mydb
```

Reboot, optionally failing over to the standby (Multi-AZ):

```bash
aws rds reboot-db-instance --db-instance-identifier mydb
```

```bash
aws rds reboot-db-instance --db-instance-identifier mydb --force-failover
```

Manual snapshot:

```bash
aws rds create-db-snapshot --db-instance-identifier mydb --db-snapshot-identifier mydb-before-upgrade
```

```bash
aws rds wait db-snapshot-available --db-snapshot-identifier mydb-before-upgrade
```

```bash
aws rds describe-db-snapshots --db-instance-identifier mydb --query 'DBSnapshots[].{ID:DBSnapshotIdentifier,Created:SnapshotCreateTime,Status:Status,Type:SnapshotType}' --output table
```

Restore a snapshot into a **new** instance:

```bash
aws rds restore-db-instance-from-db-snapshot --db-instance-identifier mydb-restored --db-snapshot-identifier mydb-before-upgrade --db-instance-class db.t4g.medium
```

Point-in-time restore into a new instance:

```bash
aws rds restore-db-instance-to-point-in-time --source-db-instance-identifier mydb --target-db-instance-identifier mydb-pitr --restore-time 2026-01-01T10:00:00Z
```

Resize instance / storage:

```bash
aws rds modify-db-instance --db-instance-identifier mydb --db-instance-class db.t4g.large --apply-immediately
```

```bash
aws rds modify-db-instance --db-instance-identifier mydb --allocated-storage 200 --apply-immediately
```

```bash
aws rds wait db-instance-available --db-instance-identifier mydb
```

Turn on deletion protection:

```bash
aws rds modify-db-instance --db-instance-identifier mydb --deletion-protection --apply-immediately
```

Recent events (last 24 h, in minutes):

```bash
aws rds describe-events --source-identifier mydb --source-type db-instance --duration 1440
```

Database log files:

```bash
aws rds describe-db-log-files --db-instance-identifier mydb --query 'DescribeDBLogFiles[].[LogFileName,Size]' --output table
```

```bash
aws rds download-db-log-file-portion --db-instance-identifier mydb --log-file-name error/postgresql.log.2026-01-01-10 --starting-token 0 --output text
```

IAM database authentication token (use it as the password):

```bash
aws rds generate-db-auth-token --hostname mydb.abcdefghijkl.eu-central-1.rds.amazonaws.com --port 5432 --region eu-central-1 --username app_user
```

Pending maintenance and available engine versions:

```bash
aws rds describe-pending-maintenance-actions
```

```bash
aws rds describe-db-engine-versions --engine postgres --query 'DBEngineVersions[].EngineVersion' --output text
```

---

## 23. DynamoDB

Tables:

```bash
aws dynamodb list-tables
```

```bash
aws dynamodb describe-table --table-name Users --query 'Table.{Status:TableStatus,Items:ItemCount,Bytes:TableSizeBytes,Keys:KeySchema}'
```

Get / put / delete an item:

```bash
aws dynamodb get-item --table-name Users --key '{"pk":{"S":"USER#42"}}'
```

```bash
aws dynamodb put-item --table-name Users --item '{"pk":{"S":"USER#42"},"name":{"S":"Alice"},"age":{"N":"30"}}'
```

```bash
aws dynamodb delete-item --table-name Users --key '{"pk":{"S":"USER#42"}}'
```

Update one attribute (`status` is a reserved word, hence `#s`):

```bash
aws dynamodb update-item --table-name Users --key '{"pk":{"S":"USER#42"}}' --update-expression 'SET #s = :s' --expression-attribute-names '{"#s":"status"}' --expression-attribute-values '{":s":{"S":"active"}}' --return-values ALL_NEW
```

Query by partition key:

```bash
aws dynamodb query --table-name Orders --key-condition-expression 'pk = :pk' --expression-attribute-values '{":pk":{"S":"USER#42"}}'
```

Scan (reads the whole table — expensive on big tables):

```bash
aws dynamodb scan --table-name Users --max-items 10
```

```bash
aws dynamodb scan --table-name Users --filter-expression 'age > :a' --expression-attribute-values '{":a":{"N":"25"}}'
```

```bash
aws dynamodb scan --table-name Users --select COUNT
```

Create an on-demand table:

```bash
aws dynamodb create-table --table-name Users --attribute-definitions AttributeName=pk,AttributeType=S --key-schema AttributeName=pk,KeyType=HASH --billing-mode PAY_PER_REQUEST
```

```bash
aws dynamodb wait table-exists --table-name Users
```

TTL, point-in-time recovery, backup:

```bash
aws dynamodb update-time-to-live --table-name Sessions --time-to-live-specification Enabled=true,AttributeName=expires_at
```

```bash
aws dynamodb update-continuous-backups --table-name Users --point-in-time-recovery-specification PointInTimeRecoveryEnabled=true
```

```bash
aws dynamodb create-backup --table-name Users --backup-name users-2026-01-01
```

---

## 24. SQS

Queues:

```bash
aws sqs list-queues
```

```bash
aws sqs list-queues --queue-name-prefix myapp
```

Create a standard / FIFO queue:

```bash
aws sqs create-queue --queue-name my-queue
```

```bash
aws sqs create-queue --queue-name my-queue.fifo --attributes FifoQueue=true,ContentBasedDeduplication=true
```

Queue URL into a variable:

```bash
QUEUE_URL=$(aws sqs get-queue-url --queue-name my-queue --query QueueUrl --output text)
```

Send:

```bash
aws sqs send-message --queue-url "$QUEUE_URL" --message-body '{"job":"test"}'
```

Peek at messages without hiding them from consumers:

```bash
aws sqs receive-message --queue-url "$QUEUE_URL" --max-number-of-messages 10 --wait-time-seconds 5 --visibility-timeout 0
```

Delete a received message:

```bash
aws sqs delete-message --queue-url "$QUEUE_URL" --receipt-handle <receipt-handle>
```

Backlog size:

```bash
aws sqs get-queue-attributes --queue-url "$QUEUE_URL" --attribute-names ApproximateNumberOfMessages ApproximateNumberOfMessagesNotVisible ApproximateNumberOfMessagesDelayed
```

Change the visibility timeout:

```bash
aws sqs set-queue-attributes --queue-url "$QUEUE_URL" --attributes VisibilityTimeout=120
```

Attach a dead-letter queue (after 5 failed receives):

```bash
aws sqs set-queue-attributes --queue-url "$QUEUE_URL" --attributes '{"RedrivePolicy":"{\"deadLetterTargetArn\":\"arn:aws:sqs:eu-central-1:123456789012:my-queue-dlq\",\"maxReceiveCount\":\"5\"}"}'
```

Move messages from the DLQ back to their source queue:

```bash
aws sqs start-message-move-task --source-arn arn:aws:sqs:eu-central-1:123456789012:my-queue-dlq
```

```bash
aws sqs list-message-move-tasks --source-arn arn:aws:sqs:eu-central-1:123456789012:my-queue-dlq
```

Delete **all** messages (once per 60 s):

```bash
aws sqs purge-queue --queue-url "$QUEUE_URL"
```

---

## 25. SNS

Topics:

```bash
aws sns list-topics
```

```bash
aws sns create-topic --name alerts --query TopicArn --output text
```

Subscribe an email address (confirm via the email) or an SQS queue:

```bash
aws sns subscribe --topic-arn arn:aws:sns:eu-central-1:123456789012:alerts --protocol email --notification-endpoint ops@example.com
```

```bash
aws sns subscribe --topic-arn arn:aws:sns:eu-central-1:123456789012:alerts --protocol sqs --notification-endpoint arn:aws:sqs:eu-central-1:123456789012:my-queue
```

```bash
aws sns list-subscriptions-by-topic --topic-arn arn:aws:sns:eu-central-1:123456789012:alerts
```

Publish:

```bash
aws sns publish --topic-arn arn:aws:sns:eu-central-1:123456789012:alerts --subject "Deploy finished" --message "myapp 1.4.0 is live"
```

Unsubscribe / delete:

```bash
aws sns unsubscribe --subscription-arn <subscription-arn>
```

```bash
aws sns delete-topic --topic-arn arn:aws:sns:eu-central-1:123456789012:alerts
```

---

## 26. Secrets Manager

List:

```bash
aws secretsmanager list-secrets --query 'SecretList[].{Name:Name,Changed:LastChangedDate,Rotation:RotationEnabled}' --output table
```

Read a secret / one JSON key of it:

```bash
aws secretsmanager get-secret-value --secret-id prod/myapp/db --query SecretString --output text
```

```bash
aws secretsmanager get-secret-value --secret-id prod/myapp/db --query SecretString --output text | jq -r .password
```

Previous value (after a rotation):

```bash
aws secretsmanager get-secret-value --secret-id prod/myapp/db --version-stage AWSPREVIOUS --query SecretString --output text
```

Create / update from a file (keeps the value out of your shell history):

```bash
aws secretsmanager create-secret --name prod/myapp/db --secret-string file://db-secret.json
```

```bash
aws secretsmanager put-secret-value --secret-id prod/myapp/db --secret-string file://db-secret.json
```

Generate a strong random password:

```bash
aws secretsmanager get-random-password --password-length 32 --exclude-punctuation --query RandomPassword --output text
```

Metadata and rotation:

```bash
aws secretsmanager describe-secret --secret-id prod/myapp/db
```

```bash
aws secretsmanager rotate-secret --secret-id prod/myapp/db
```

Delete with a recovery window / undo the delete:

```bash
aws secretsmanager delete-secret --secret-id prod/myapp/old --recovery-window-in-days 7
```

```bash
aws secretsmanager restore-secret --secret-id prod/myapp/old
```

---

## 27. KMS

Keys and aliases:

```bash
aws kms list-aliases --query 'Aliases[].[AliasName,TargetKeyId]' --output table
```

```bash
aws kms describe-key --key-id alias/my-key
```

Create a key with an alias and yearly rotation:

```bash
aws kms create-key --description "myapp data key" --query KeyMetadata.KeyId --output text
```

```bash
aws kms create-alias --alias-name alias/my-key --target-key-id <key-id>
```

```bash
aws kms enable-key-rotation --key-id <key-id>
```

```bash
aws kms get-key-rotation-status --key-id <key-id>
```

Encrypt / decrypt a small file (up to 4 KB — use envelope encryption for bigger data):

```bash
aws kms encrypt --key-id alias/my-key --plaintext fileb://secret.txt --query CiphertextBlob --output text | base64 --decode > secret.txt.enc
```

```bash
aws kms decrypt --ciphertext-blob fileb://secret.txt.enc --query Plaintext --output text | base64 --decode
```

---

## 28. Route 53

Hosted zones:

```bash
aws route53 list-hosted-zones --query 'HostedZones[].{ID:Id,Name:Name,Private:Config.PrivateZone,Records:ResourceRecordSetCount}' --output table
```

Zone ID by domain:

```bash
aws route53 list-hosted-zones-by-name --dns-name example.com --query 'HostedZones[0].Id' --output text
```

Records:

```bash
aws route53 list-resource-record-sets --hosted-zone-id Z0123456789ABCDEFGHIJ --query "ResourceRecordSets[?Type=='A' || Type=='CNAME'].{Name:Name,Type:Type,Value:ResourceRecords[0].Value,Alias:AliasTarget.DNSName}" --output table
```

Create or update a record:

```bash
aws route53 change-resource-record-sets --hosted-zone-id Z0123456789ABCDEFGHIJ --change-batch file://change-batch.json --query ChangeInfo.Id --output text
```

`change-batch.json` — an `A` record and an alias to a load balancer:

```json
{
  "Comment": "Point app to the new server",
  "Changes": [
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "app.example.com",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [{ "Value": "203.0.113.10" }]
      }
    },
    {
      "Action": "UPSERT",
      "ResourceRecordSet": {
        "Name": "example.com",
        "Type": "A",
        "AliasTarget": {
          "HostedZoneId": "<load-balancer-hosted-zone-id>",
          "DNSName": "my-alb-1234567890.eu-central-1.elb.amazonaws.com",
          "EvaluateTargetHealth": true
        }
      }
    }
  ]
}
```

Wait until the change is live on all Route 53 servers:

```bash
aws route53 wait resource-record-sets-changed --id /change/C0123456789ABCDEFGHIJ
```

What does Route 53 answer for a name?

```bash
aws route53 test-dns-answer --hosted-zone-id Z0123456789ABCDEFGHIJ --record-name app.example.com --record-type A
```

Create a hosted zone:

```bash
aws route53 create-hosted-zone --name example.com --caller-reference "$(date +%s)"
```

---

## 29. CloudFront

Distributions:

```bash
aws cloudfront list-distributions --query 'DistributionList.Items[].{ID:Id,Domain:DomainName,Alias:Aliases.Items[0],Status:Status,Enabled:Enabled}' --output table
```

Find a distribution by its custom domain:

```bash
aws cloudfront list-distributions --query "DistributionList.Items[?Aliases.Items && contains(Aliases.Items, 'www.example.com')].Id" --output text
```

Clear the cache (all paths / specific paths):

```bash
aws cloudfront create-invalidation --distribution-id E1234567890ABC --paths "/*"
```

```bash
aws cloudfront create-invalidation --distribution-id E1234567890ABC --paths "/index.html" "/assets/*"
```

Invalidation status:

```bash
aws cloudfront list-invalidations --distribution-id E1234567890ABC --query 'InvalidationList.Items[:5]' --output table
```

```bash
aws cloudfront wait invalidation-completed --distribution-id E1234567890ABC --id <invalidation-id>
```

Current config (the `ETag` is needed for `update-distribution`):

```bash
aws cloudfront get-distribution-config --id E1234567890ABC > distribution-config.json
```

---

## 30. Load Balancers (ELBv2)

Application / Network load balancers:

```bash
aws elbv2 describe-load-balancers --query 'LoadBalancers[].{Name:LoadBalancerName,Type:Type,Scheme:Scheme,State:State.Code,DNS:DNSName}' --output table
```

Target group ARN by name:

```bash
aws elbv2 describe-target-groups --names my-tg --query 'TargetGroups[0].TargetGroupArn' --output text
```

Are the targets healthy? (the first thing to check on a `502`/`503`):

```bash
aws elbv2 describe-target-health --target-group-arn <target-group-arn> --query 'TargetHealthDescriptions[].{Target:Target.Id,Port:Target.Port,State:TargetHealth.State,Reason:TargetHealth.Reason,Detail:TargetHealth.Description}' --output table
```

Listeners and rules:

```bash
aws elbv2 describe-listeners --load-balancer-arn <load-balancer-arn> --query 'Listeners[].{Port:Port,Protocol:Protocol,ARN:ListenerArn}' --output table
```

```bash
aws elbv2 describe-rules --listener-arn <listener-arn>
```

Register / deregister targets:

```bash
aws elbv2 register-targets --target-group-arn <target-group-arn> --targets Id=i-0123456789abcdef0
```

```bash
aws elbv2 deregister-targets --target-group-arn <target-group-arn> --targets Id=i-0123456789abcdef0
```

Tune health checks and connection draining:

```bash
aws elbv2 modify-target-group --target-group-arn <target-group-arn> --health-check-path /health --health-check-interval-seconds 15 --healthy-threshold-count 2
```

```bash
aws elbv2 modify-target-group-attributes --target-group-arn <target-group-arn> --attributes Key=deregistration_delay.timeout_seconds,Value=30
```

---

## 31. Auto Scaling

Groups at a glance:

```bash
aws autoscaling describe-auto-scaling-groups --query 'AutoScalingGroups[].{Name:AutoScalingGroupName,Min:MinSize,Desired:DesiredCapacity,Max:MaxSize,Instances:length(Instances)}' --output table
```

Change capacity:

```bash
aws autoscaling set-desired-capacity --auto-scaling-group-name my-asg --desired-capacity 4
```

```bash
aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg --min-size 2 --max-size 10
```

Roll all instances (e.g. after a new launch-template version):

```bash
aws autoscaling start-instance-refresh --auto-scaling-group-name my-asg --preferences '{"MinHealthyPercentage":90,"InstanceWarmup":120}'
```

```bash
aws autoscaling describe-instance-refreshes --auto-scaling-group-name my-asg --query 'InstanceRefreshes[0].{Status:Status,Done:PercentageComplete,Reason:StatusReason}'
```

```bash
aws autoscaling cancel-instance-refresh --auto-scaling-group-name my-asg
```

Recent scaling activity (why did it scale?):

```bash
aws autoscaling describe-scaling-activities --auto-scaling-group-name my-asg --max-items 10 --query 'Activities[].[StartTime,StatusCode,Description]' --output table
```

Stop the group replacing an instance while you debug it:

```bash
aws autoscaling suspend-processes --auto-scaling-group-name my-asg --scaling-processes HealthCheck ReplaceUnhealthy
```

```bash
aws autoscaling resume-processes --auto-scaling-group-name my-asg
```

---

## 32. CloudFormation

Stacks:

```bash
aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE UPDATE_ROLLBACK_COMPLETE --query 'StackSummaries[].[StackName,StackStatus,LastUpdatedTime]' --output table
```

```bash
aws cloudformation describe-stacks --stack-name my-stack --query 'Stacks[0].StackStatus' --output text
```

Stack outputs:

```bash
aws cloudformation describe-stacks --stack-name my-stack --query 'Stacks[0].Outputs[].[OutputKey,OutputValue]' --output table
```

Validate a template:

```bash
aws cloudformation validate-template --template-body file://template.yaml
```

Create or update in one command:

```bash
aws cloudformation deploy --template-file template.yaml --stack-name my-stack --parameter-overrides Env=prod InstanceType=t3.small --capabilities CAPABILITY_NAMED_IAM --tags Project=myapp --no-fail-on-empty-changeset
```

Preview the change set without applying it:

```bash
aws cloudformation deploy --template-file template.yaml --stack-name my-stack --capabilities CAPABILITY_NAMED_IAM --no-execute-changeset
```

Upload local code/assets referenced by the template, then deploy the packaged template:

```bash
aws cloudformation package --template-file template.yaml --s3-bucket my-artifacts --output-template-file packaged.yaml
```

Why did it fail?

```bash
aws cloudformation describe-stack-events --stack-name my-stack --query 'StackEvents[?contains(ResourceStatus, `FAILED`)].[Timestamp,LogicalResourceId,ResourceStatusReason]' --output table
```

Resources of a stack:

```bash
aws cloudformation describe-stack-resources --stack-name my-stack --query 'StackResources[].[LogicalResourceId,ResourceType,PhysicalResourceId,ResourceStatus]' --output table
```

Drift detection (manual changes made outside CloudFormation):

```bash
aws cloudformation detect-stack-drift --stack-name my-stack
```

```bash
aws cloudformation describe-stack-resource-drifts --stack-name my-stack --stack-resource-drift-status-filters MODIFIED DELETED
```

Cross-stack exports:

```bash
aws cloudformation list-exports --query 'Exports[].[Name,Value]' --output table
```

Stuck in `UPDATE_ROLLBACK_FAILED`:

```bash
aws cloudformation continue-update-rollback --stack-name my-stack
```

Protect a stack from deletion:

```bash
aws cloudformation update-termination-protection --enable-termination-protection --stack-name my-stack
```

Delete:

```bash
aws cloudformation delete-stack --stack-name my-stack
```

```bash
aws cloudformation wait stack-delete-complete --stack-name my-stack
```

---

## 33. ACM Certificates

Certificates:

```bash
aws acm list-certificates --query 'CertificateSummaryList[].{Domain:DomainName,Status:Status,ARN:CertificateArn}' --output table
```

Request a DNS-validated certificate (for **CloudFront** it must be in `us-east-1`):

```bash
aws acm request-certificate --domain-name example.com --subject-alternative-names "*.example.com" --validation-method DNS --region us-east-1 --query CertificateArn --output text
```

DNS records to create for validation:

```bash
aws acm describe-certificate --certificate-arn <certificate-arn> --query 'Certificate.DomainValidationOptions[].ResourceRecord'
```

```bash
aws acm wait certificate-validated --certificate-arn <certificate-arn>
```

Expiry date and where it is used:

```bash
aws acm describe-certificate --certificate-arn <certificate-arn> --query 'Certificate.{Status:Status,Expires:NotAfter,Renewal:RenewalEligibility,InUseBy:InUseBy}'
```

---

## 34. Cost & Billing

> Each Cost Explorer API request costs **$0.01**. Dates: `Start` is inclusive, `End` is exclusive.

Last month's cost by service:

```bash
aws ce get-cost-and-usage --time-period Start=2026-09-01,End=2026-10-01 --granularity MONTHLY --metrics UnblendedCost --group-by Type=DIMENSION,Key=SERVICE --query 'ResultsByTime[0].Groups[].[Keys[0],Metrics.UnblendedCost.Amount]' --output table
```

Daily total:

```bash
aws ce get-cost-and-usage --time-period Start=2026-10-01,End=2026-10-08 --granularity DAILY --metrics UnblendedCost --query 'ResultsByTime[].[TimePeriod.Start,Total.UnblendedCost.Amount]' --output table
```

Cost by a cost-allocation tag:

```bash
aws ce get-cost-and-usage --time-period Start=2026-09-01,End=2026-10-01 --granularity MONTHLY --metrics UnblendedCost --group-by Type=TAG,Key=Project
```

Forecast to the end of the month (start must be today or later):

```bash
aws ce get-cost-forecast --time-period Start=2026-10-09,End=2026-11-01 --metric UNBLENDED_COST --granularity MONTHLY
```

Budgets:

```bash
aws budgets describe-budgets --account-id 123456789012
```

Free-tier usage:

```bash
aws freetier get-free-tier-usage
```

Quick waste hunt:

```bash
aws ec2 describe-volumes --filters Name=status,Values=available --query 'Volumes[].[VolumeId,Size]' --output text
```

```bash
aws ec2 describe-addresses --query 'Addresses[?AssociationId==null].PublicIp' --output text
```

```bash
aws ec2 describe-instances --filters Name=instance-state-name,Values=stopped --query 'Reservations[].Instances[].InstanceId' --output text
```

```bash
aws ec2 describe-snapshots --owner-ids self --query 'Snapshots[?StartTime<=`2025-01-01`].[SnapshotId,VolumeSize,StartTime]' --output text
```

```bash
aws logs describe-log-groups --query 'logGroups[?!retentionInDays].[logGroupName,storedBytes]' --output text
```

---

## 35. Tags & Resource Discovery

Everything tagged `Project=myapp` (across services):

```bash
aws resourcegroupstaggingapi get-resources --tag-filters Key=Project,Values=myapp --query 'ResourceTagMappingList[].ResourceARN' --output text
```

Only some resource types:

```bash
aws resourcegroupstaggingapi get-resources --resource-type-filters ec2:instance rds:db --tag-filters Key=Env,Values=prod
```

Tag keys and values in use:

```bash
aws resourcegroupstaggingapi get-tag-keys
```

```bash
aws resourcegroupstaggingapi get-tag-values --key Project
```

Tag many resources at once:

```bash
aws resourcegroupstaggingapi tag-resources --resource-arn-list arn:aws:s3:::my-bucket arn:aws:sns:eu-central-1:123456789012:alerts --tags Owner=platform
```

---

## 36. Waiters

Waiters poll until a resource reaches a state, and exit non-zero if it never does — perfect for scripts and CI.

See every waiter of a service:

```bash
aws ec2 wait help
```

Common ones:

```bash
aws ec2 wait instance-running --instance-ids i-0123456789abcdef0
```

```bash
aws ec2 wait instance-status-ok --instance-ids i-0123456789abcdef0
```

```bash
aws ecs wait services-stable --cluster my-cluster --services my-service
```

```bash
aws rds wait db-instance-available --db-instance-identifier mydb
```

```bash
aws lambda wait function-updated --function-name my-fn
```

```bash
aws cloudformation wait stack-create-complete --stack-name my-stack
```

```bash
aws cloudformation wait stack-update-complete --stack-name my-stack
```

```bash
aws dynamodb wait table-exists --table-name Users
```

```bash
aws cloudfront wait invalidation-completed --distribution-id E1234567890ABC --id <invalidation-id>
```

---

## 37. Scripting, Loops & Multi-region

Safe script header:

```bash
#!/usr/bin/env bash
set -euo pipefail
export AWS_PAGER=""
export AWS_REGION="${AWS_REGION:-eu-central-1}"
```

Running instances in **every** region:

```bash
for region in $(aws ec2 describe-regions --query 'Regions[].RegionName' --output text); do echo "== $region"; aws ec2 describe-instances --region "$region" --filters Name=instance-state-name,Values=running --query 'Reservations[].Instances[].[InstanceId,InstanceType]' --output text; done
```

Who am I in every profile?

```bash
for p in $(aws configure list-profiles); do printf '%-20s ' "$p"; aws sts get-caller-identity --profile "$p" --query Arn --output text 2>/dev/null || echo "(no valid credentials)"; done
```

Stop every running instance tagged `Env=dev`:

```bash
aws ec2 describe-instances --filters Name=tag:Env,Values=dev Name=instance-state-name,Values=running --query 'Reservations[].Instances[].InstanceId' --output text | xargs -r aws ec2 stop-instances --instance-ids
```

Empty `--output text` safely (avoid acting on `None`):

```bash
ID=$(aws ec2 describe-vpcs --filters Name=tag:Name,Values=main --query 'Vpcs[0].VpcId' --output text); [ "$ID" = "None" ] && echo "not found" && exit 1
```

Retry on throttling automatically:

```bash
export AWS_RETRY_MODE=adaptive AWS_MAX_ATTEMPTS=10
```

---

## 38. Troubleshooting

Start every investigation with:

```bash
aws sts get-caller-identity
```

```bash
aws configure list
```

See where credentials, region and endpoint come from:

```bash
aws s3 ls --debug 2>&1 | grep -iE 'credential|region|endpoint' | head -n 20
```

| Error | Fix |
|---|---|
| `Unable to locate credentials` | Run `aws configure`, `aws sso login --profile …`, or `export AWS_PROFILE=…` |
| `ExpiredToken` / `The security token included in the request is expired` | Log in again (`aws sso login`) or re-assume the role |
| `The SSO session associated with this profile has expired` | `aws sso login --profile <profile>` |
| `AccessDenied` / `UnauthorizedOperation` | Check identity, region, SCPs and resource policies; try `aws iam simulate-principal-policy` |
| `Encoded authorization failure message` | `aws sts decode-authorization-message --encoded-message …` |
| `Could not connect to the endpoint URL` | Wrong/typo region, service not in that region, or no network/proxy route |
| `Signature expired` / `InvalidSignatureException` | Local clock is off — sync time (`sudo sntp -sS time.apple.com` on macOS, `timedatectl` on Linux) |
| `ThrottlingException` / `Rate exceeded` | `export AWS_RETRY_MODE=adaptive AWS_MAX_ATTEMPTS=10`, reduce parallelism |
| `SSL validation failed` behind a corporate proxy | `export AWS_CA_BUNDLE=/path/to/corporate-ca.pem` |
| S3 `PermanentRedirect` / `301` | Bucket is in another region — `aws s3api get-bucket-location --bucket …` and pass `--region` |
| Output opens in `less` | `export AWS_PAGER=""` or `--no-cli-pager` |
| Lambda `Invalid base64` on `--payload` | Add `--cli-binary-format raw-in-base64-out` (or `aws configure set cli_binary_format raw-in-base64-out`) |
| `SessionManagerPlugin is not found` | Install the Session Manager plugin |
| SSM `TargetNotConnected` | SSM agent offline, instance role lacks `AmazonSSMManagedInstanceCore`, or no route to SSM endpoints |
| ECS Exec `execute command was not enabled` | `aws ecs update-service … --enable-execute-command --force-new-deployment` |
| `kubectl`: `You must be logged in to the server (Unauthorized)` | Re-run `aws eks update-kubeconfig` with the right profile; check the principal has an EKS access entry |

---

## 39. Most-used Commands

| Task | Command |
|---|---|
| Who am I | `aws sts get-caller-identity` |
| Active config | `aws configure list` |
| SSO login | `aws sso login --profile dev` |
| Switch profile | `export AWS_PROFILE=prod` |
| List buckets | `aws s3 ls` |
| Upload / download | `aws s3 cp file.txt s3://my-bucket/` |
| Sync directory | `aws s3 sync ./dist s3://my-bucket/ --delete` |
| Running instances | `aws ec2 describe-instances --filters Name=instance-state-name,Values=running` |
| Shell on instance | `aws ssm start-session --target i-0123456789abcdef0` |
| Read parameter | `aws ssm get-parameter --name /myapp/prod/KEY --with-decryption` |
| Read secret | `aws secretsmanager get-secret-value --secret-id prod/myapp/db` |
| ECR login | `aws ecr get-login-password \| docker login --username AWS --password-stdin <registry>` |
| Redeploy ECS service | `aws ecs update-service --cluster my-cluster --service my-service --force-new-deployment` |
| kubeconfig for EKS | `aws eks update-kubeconfig --name my-cluster` |
| Tail logs | `aws logs tail <log-group> --follow` |
| Invalidate CloudFront | `aws cloudfront create-invalidation --distribution-id <id> --paths "/*"` |
| Deploy CloudFormation | `aws cloudformation deploy --template-file template.yaml --stack-name my-stack` |

---

## 40. Deployment Quick Flows

### A. Docker image → ECR → ECS

Variables:

```bash
REGION=eu-central-1
ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
REGISTRY="$ACCOUNT_ID.dkr.ecr.$REGION.amazonaws.com"
TAG=$(git rev-parse --short HEAD)
```

Log in, build and push (`--platform linux/amd64` matters when building on Apple Silicon for x86 Fargate):

```bash
aws ecr get-login-password --region "$REGION" | docker login --username AWS --password-stdin "$REGISTRY"
```

```bash
docker buildx build --platform linux/amd64 -t "$REGISTRY/myapp:$TAG" --push .
```

Register a new task-definition revision that uses the new image:

```bash
aws ecs describe-task-definition --task-definition myapp --query taskDefinition | jq --arg IMAGE "$REGISTRY/myapp:$TAG" '.containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn,.revision,.status,.requiresAttributes,.compatibilities,.registeredAt,.registeredBy)' > taskdef.json
```

```bash
TASK_DEF_ARN=$(aws ecs register-task-definition --cli-input-json file://taskdef.json --query taskDefinition.taskDefinitionArn --output text)
```

Deploy and wait:

```bash
aws ecs update-service --cluster my-cluster --service myapp --task-definition "$TASK_DEF_ARN"
```

```bash
aws ecs wait services-stable --cluster my-cluster --services myapp
```

Roll back to the previous revision:

```bash
aws ecs update-service --cluster my-cluster --service myapp --task-definition myapp:<previous-revision>
```

### B. Docker image → ECR → EKS

```bash
aws eks update-kubeconfig --name my-cluster --region "$REGION"
```

```bash
kubectl set image deployment/myapp app="$REGISTRY/myapp:$TAG"
```

```bash
kubectl rollout status deployment/myapp
```

```bash
kubectl rollout undo deployment/myapp
```

> Using GitOps (Argo CD / Flux)? Push the new tag to your manifests repository instead of running `kubectl set image`.

### C. Static site → S3 + CloudFront

Long-cache hashed assets, never cache `index.html`, then invalidate it:

```bash
aws s3 sync ./dist s3://my-site-bucket --delete --exclude "index.html" --cache-control "public,max-age=31536000,immutable"
```

```bash
aws s3 cp ./dist/index.html s3://my-site-bucket/index.html --cache-control "no-cache"
```

```bash
aws cloudfront create-invalidation --distribution-id E1234567890ABC --paths "/index.html"
```

### D. Lambda (zip) with a `live` alias

```bash
zip -r function.zip . -x "*.git*"
```

```bash
aws lambda update-function-code --function-name my-fn --zip-file fileb://function.zip
```

```bash
aws lambda wait function-updated --function-name my-fn
```

```bash
VERSION=$(aws lambda publish-version --function-name my-fn --query Version --output text)
```

```bash
aws lambda update-alias --function-name my-fn --name live --function-version "$VERSION"
```

---

## 41. Official References

- AWS CLI v2 command reference — <https://awscli.amazonaws.com/v2/documentation/api/latest/index.html>
- AWS CLI user guide — <https://docs.aws.amazon.com/cli/latest/userguide/>
- Install or update the AWS CLI — <https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html>
- Configuration and credential settings — <https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html>
- Environment variables — <https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html>
- IAM Identity Center (SSO) with the CLI — <https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html>
- Filtering output (`--query`) — <https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-filter.html>
- JMESPath tutorial — <https://jmespath.org/tutorial.html>
- CLI aliases — <https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-alias.html>
- Session Manager plugin — <https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html>
- AWS CLI on GitHub — <https://github.com/aws/aws-cli>

---

## 42. Safety Notes

Before anything destructive, confirm **which account and region** you are in:

```bash
aws sts get-caller-identity --query '[Account,Arn]' --output text && aws configure get region
```

Preview first where the CLI allows it:

```bash
aws s3 sync ./dist s3://my-bucket/ --delete --dryrun
```

```bash
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0 --dry-run
```

Commands that permanently delete data or infrastructure:

```bash
aws s3 rb s3://my-bucket --force
```

```bash
aws s3 rm s3://my-bucket/ --recursive
```

```bash
aws rds delete-db-instance --db-instance-identifier mydb --skip-final-snapshot
```

```bash
aws dynamodb delete-table --table-name Users
```

```bash
aws cloudformation delete-stack --stack-name my-stack
```

```bash
aws ecr delete-repository --repository-name myapp --force
```

```bash
aws secretsmanager delete-secret --secret-id prod/myapp/db --force-delete-without-recovery
```

```bash
aws kms schedule-key-deletion --key-id <key-id> --pending-window-in-days 30
```

```bash
aws sqs purge-queue --queue-url "$QUEUE_URL"
```

Turn on protection for things that must never disappear:

```bash
aws ec2 modify-instance-attribute --instance-id i-0123456789abcdef0 --disable-api-termination
```

```bash
aws rds modify-db-instance --db-instance-identifier mydb --deletion-protection --apply-immediately
```

```bash
aws cloudformation update-termination-protection --enable-termination-protection --stack-name my-stack
```

```bash
aws s3api put-bucket-versioning --bucket my-bucket --versioning-configuration Status=Enabled
```

Rules of thumb:

- Prefer **IAM Identity Center (SSO)** or roles over long-lived access keys; never commit `~/.aws/credentials`, `.env` files or keys to Git.
- Keep production in its own account and profile, and always pass `--profile` / `--region` explicitly in scripts.
- Pass secrets with `file://` instead of typing them on the command line (shell history).
- Data deleted from KMS (after the waiting period) can never be decrypted again.

---

## 43. Aliases

### AWS CLI aliases

Create `~/.aws/cli/alias`:

```ini
[toplevel]
whoami = sts get-caller-identity

regions = ec2 describe-regions --query 'Regions[].RegionName' --output text

running = ec2 describe-instances --filters Name=instance-state-name,Values=running --query 'Reservations[].Instances[].{ID:InstanceId,Name:Tags[?Key==`Name`]|[0].Value,Type:InstanceType,IP:PrivateIpAddress}' --output table

ecr-login =
  !f() {
    region="${1:-$(aws configure get region)}"
    account="$(aws sts get-caller-identity --query Account --output text)"
    aws ecr get-login-password --region "$region" | docker login --username AWS --password-stdin "$account.dkr.ecr.$region.amazonaws.com"
  }; f
```

Use them like built-in commands:

```bash
aws whoami
```

```bash
aws running --profile prod
```

```bash
aws ecr-login eu-central-1
```

### Shell aliases

Add to `~/.bashrc` or `~/.zshrc`:

```bash
alias awsid='aws sts get-caller-identity'
alias awsprofiles='aws configure list-profiles'
alias awslogin='aws sso login'
awsp() { export AWS_PROFILE="$1"; aws sts get-caller-identity --query Arn --output text; }
awsr() { export AWS_REGION="$1"; echo "AWS_REGION=$AWS_REGION"; }
```

---

## Contributing

Found a wrong flag, an outdated command or a missing everyday task? Pull requests are welcome — keep one command per code block and verify it against `aws <service> <command> help`.

## Acknowledgements

Format inspired by [omaralalwi/docker-devops-cheat-sheet](https://github.com/omaralalwi/docker-devops-cheat-sheet).

## License

[MIT](LICENSE)
