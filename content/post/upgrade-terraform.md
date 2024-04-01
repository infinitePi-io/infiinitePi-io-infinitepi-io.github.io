---
author: "Satish tripathi"
date: 2024-03-28
linktitle: "Streamlining Your Infrastructure: A Guide to Upgrading Terraform state files from Version 0.12.31 to 1.3.6"
title: "Streamlining Your Infrastructure: A Guide to Upgrading Terraform state files from Version 0.12.31 to 1.3.6"
---


**Why This Blog?**
The frustration of not finding a proper guide on the internet prompted the creation of this blog. Countless hours were spent researching, experimenting, and compiling the most effective and efficient upgrade process. We understand the importance of a smooth transition, and we've taken great care to ensure that every step is explained in a way that is easy to understand and follow.

**Why it's Important to Upgrade the Terraform version:**
- I will mention a few of the points here:
- New Features and Enhancements.
- Bug Fixes and Security Updates
- Compatibility with new Provider Versions
- Community Support and Documentation
- LTS support.
- Have you ever noticed that when you run terraform plan, Do your AWS resources get tagged even if those tags are already present? This behavior can occur due to the AWS provider version. To ensure compatibility with the latest provider version, it is important to upgrade your Terraform version.

**Steps to Upgrade the Terraform Version:**
*What assumptions are?*
  State files are parked in S3 or any other storage, but in this blog, I assume that the state file is in the S3 bucket.
  You have a backend file in the Terraform working directory.
  You have `**/*.terraform.tfstate* `in your .gitignore file.
  *Get the state from the Remote Storage:*
  The backend file looks something like the below:

  ![Backend-file](/image.png)

  Login into AWS Account where the resources live using the access key and secret key on the CLI. Keep in mind that Terraform 0.12.31 doesn't support the AWS SSO login method.
*Switch the terraform version to 0.12.31:*
I use tfenv to manage the terraform versions:
```terraform
  tfenv use 0.12.31
```

Run the below command to Initializatize terraform:
```terraform
  terraform init
```

Once Initialization is completed, remove the backend file.
```bash
  rm backend.tf
```
Now move the state locally by running the below command:
```terraform
terraform init
```
The output will be something like the below:
![Alt text](/image-1.png)

Type yes to move the state locally. Now you should have a file name terraform.tfstate in your Terraform project directory.

Create a directory to backup the state file during the upgrade process.
```bash
  mkdir .backup
  cp terraform.tfstate .backup/terraform.tfstate.0.12.31
  git add -A .
  git commit -am "0.12.31 checkpoint"
```
*Upgrade to 0.13.7*
Change the `required_version` in provider.tf or any other file like backend.tf
In our infrastructure, we keep the terraform version in provider.tf file.
Switch the terraform version to 0.13.7 by running the below command:
```terraform
  tfenv use 0.13.7
```
Run the below command to perform the 0.13.7 upgrade:
```bash
find . -type f -not -path './.terraform/*' -not -path './.git/*' -name '*.tf' \
  | xargs -n1 dirname | uniq | xargs -n1 terraform 0.13upgrade -yes
```
the above command is provided by hashicorop.
*Upgrade the state by running the below command:*

```bash
  rm -rf .terraform*
  terraform init
  # this should have no changes
  terraform apply
  # the apply should not ask for confirmation and have 0 resource changes
  head terraform.tfstate
  # should look something along the lines of the following
```
![Alt text](/image-2.png)

*Backup the state & Clear git tree:*
```bash
  cp terraform.tfstate .backup/terraform.tfstate.0.13.7
  git commit -am "0.13.7 checkpoint"
```

*Upgrade to 0.14.11*
Switch to 0.14.11
```terraform
  tfenv use 0.14.11
```

Change the required_version to 0.14.11

![New-3](/image-3.png)

Run the below command:
```bash
  rm -rf .terraform*
  terraform init
  terraform plan
  # this should have no changes, if it does, proceed to fix the issues and re-run the commands in the block above.
```
*The plan should have no changes. Then only processed to run an apply.*
```terraform
  terraform apply
  # the apply should not ask for confirmation and have 0 resource changes
  head terraform.tfstate
```
# should look something along the lines of the following

![Alt text](/image-4.png)

*Upgrade to 1.3.6:*
change the required_version in the provider.tf or backend.tf file.
Switch to 1.3.6:
```terraform
  tfenv use 1.3.6
```
Now upgrade the state:
```bash
  rm -rf .terraform*
  terraform init
  terraform plan
  # this should have no changes
  terraform apply
  # the apply should not ask for confirmation and have 0 resource changes
  head terraform.tfstate
  # should look something along the lines of the following
```
![Alt text](/image-5.png)

*Backup the state & Clear git tree:*

```bash
  cp terraform.tfstate .backup/terraform.tfstate.1.3.6
  git commit -am "1.3.6 checkpoint"
  # re-create the state backend.tf file and run the below command:
  terraform init
  type yes to move the state remote storage.
```
![Alt text](/image-6.png)

Congratulations! You have successfully upgraded your state file.


Note: It is important to exercise caution when handling your state file and avoid pushing it to public repositories like GitHub. Your state file may contain sensitive and confidential information, such as resource IDs, credentials, and other details that could potentially compromise the security of your infrastructure.
Acknowledgments: Phil's expertise and deep understanding of Terraform allowed us to craft a reliable and detailed resource for upgrading Terraform from version 0.12.31 to 1.3.6. His patient guidance and meticulous attention to detail ensured the accuracy and clarity of the steps outlined in this guide.