# Automate Terraform with GitHub Actions

This repo is a companion repo to the [Automate Terraform with GitHub Actions tutorial](https://developer.hashicorp.com/terraform/tutorials/automation/github-actions).

GitHub Actions add continuous integration to GitHub repositories to automate your software builds, tests, and deployments. Automating Terraform with CI/CD enforces configuration best practices, promotes collaboration, and automates the Terraform workflow.

HashiCorp provides GitHub Actions that integrate with the Terraform Cloud API. These actions let you create your own custom CI/CD workflows to meet the needs of your organization.

<img src = "https://github.com/maveric-coder/Terraform/blob/main/files/content/assets.png" >

The workflow will:

1. Generate a plan for every commit to a pull request branch, which you can review in Terraform Cloud.
Apply the configuration when you update the main branch.
2. After configuring the GitHub Action, you will create and merge a pull request to test the workflow.

Terraform Cloud's built-in support for GitHub webhooks can accomplish this generic workflow. 

## Prerequisits
* A GitHub account
* A Terraform Cloud account
* An AWS account


## Set up Terraform Cloud

First, create a new Terraform Cloud workspace.
<br>Go to the *Create a new Workspace page* 
<img src = "https://github.com/maveric-coder/Terraform/blob/main/files/content/1.png" >

Select *API-driven workflow*.

<img src = "https://github.com/maveric-coder/Terraform/blob/main/files/content/2.png" >

Name your workspace *Terraform_Prod* or a new name and click Create workspace.

Now, find the AWS credentials you want to use for the workspace, or create a new key pair in the IAM console. 

<img src = "https://github.com/maveric-coder/Terraform/blob/main/files/content/aws_tokens.png" >


Then, add the following as Environment Variables for your learn-terraform-github-actions workspace.

<img src = "https://github.com/maveric-coder/Terraform/blob/main/files/content/3.png" >

|Type|	Variable name|	Description|	Sensitive|
|-----|-----|-----|--|
|Environment variable|	AWS_ACCESS_KEY_ID|	The access key ID from your AWS key pair|	No|
|Environment variable|	AWS_SECRET_ACCESS_KEY|	The secret access key from your AWS key pair|	Yes|


Terraform Cloud will use these credentials to authenticate to AWS.

Finally, go to the Tokens page in your Terraform Cloud User Settings.

<img src = "https://github.com/maveric-coder/Terraform/blob/main/files/content/4.png" >


Click on Create an API token, enter GitHub Actions for the Description, then click Generate token. And copy the token.

<img src = "https://github.com/maveric-coder/Terraform/blob/main/files/content/5.png" >


## Set up a GitHub repository

Navigate to the [Learn Terraform GitHub Actions](https://github.com/hashicorp-education/learn-terraform-github-actions) template repository.

Select Use this template, then select Create a new repository. In our case we already have the repository so no need create a new repository.

In the repository, navigate to the Settings page. Open the Secrets and variables menu, then select Actions.
<img src = "https://github.com/maveric-coder/Terraform/blob/main/files/content/6.png" >

Now, select New repository secret. Create a secret named TF_API_TOKEN, setting the Terraform Cloud API token you created in the previous step as the value.
<img src = "https://github.com/maveric-coder/Terraform/blob/main/files/content/7.png" >
Now, clone the repository.
```sh
git clone git@github.com:maveric-coder/terraform-github-actions.git
```

## Review Actions workflows
There are several files in your local repository.

* `main.tf` contains Terraform configuration to deploy a publicly accessible EC2 instance.
* `.github/workflows/terraform-plan.yml` defines the Actions workflow that runs Terraform plan.
* `.github/workflows/terraform-apply.yml` defines the Actions workflow that runs Terraform apply.


## Review Terraform plan workflow
In your editor, open `.github/workflows/terraform-plan.yml`.
<br>The configuration states that this workflow should only run on pull requests. It also defines environment variables used by the workflow.
```yml
env:
  TF_CLOUD_ORGANIZATION: "YOUR-ORGANIZATION-HERE"
  TF_API_TOKEN: "${{ secrets.TF_API_TOKEN }}"
  TF_WORKSPACE: "YOUR-WORKSPACE-HERE"
  CONFIG_DIRECTORY: "./"
## ...
```
Replace the values of `TF_CLOUD_ORGANIZATION: "YOUR-ORGANIZATION-HERE"` and `TF_WORKSPACE: "YOUR-WORKSPACE-HERE"` from Terraform cloud.

Now, open `.github/workflows/terraform-apply.yml`.
<br>Replace the values of `TF_CLOUD_ORGANIZATION: "YOUR-ORGANIZATION-HERE"` and `TF_WORKSPACE: "YOUR-WORKSPACE-HERE"`.

The workflow defines several steps.

Checkout checks out the current configuration. Uses defines the action/Docker image to run that specific step. The checkout step uses GitHub's actions/checkout@v3 action.

```yml
.github/workflows/terraform-apply.yml
## ...
- name: Checkout
  uses: actions/checkout@v3
## ...
```
Upload Configuration uploads the Terraform configuration to Terraform Cloud.
```yml
.github/workflows/terraform-apply.yml
## ...
- name: Upload Configuration
  uses: hashicorp/tfc-workflows-github/actions/upload-configuration@v1.0.0
  id: apply-upload
  with:
    workspace: ${{ env.TF_WORKSPACE }}
    directory: ${{ env.CONFIG_DIRECTORY }}
## ...
```
Create Apply Run creates a Terraform apply run using the configuration uploaded in the previous step.
```yml
.github/workflows/terraform-apply.yml
## ...
- name: Create Apply Run
  uses: hashicorp/tfc-workflows-github/actions/create-run@v1.0.0
  id: apply-run
  with:
    workspace: ${{ env.TF_WORKSPACE }}
    configuration_version: ${{ steps.apply-upload.outputs.configuration_version_id }}
## ...
```
Apply confirms and applies the run.
```yml
.github/workflows/terraform-apply.yml
## ...
- name: Apply
  uses: hashicorp/tfc-workflows-github/actions/apply-run@v1.0.0
  if: fromJSON(steps.apply-run.outputs.payload).data.attributes.actions.IsConfirmable
  id: apply
  with:
    run: ${{ steps.apply-run.outputs.run_id }}
    comment: "Apply Run from GitHub Actions CI ${{ github.sha }}"
```

## Create pull request
First, see all the existing branches if 'update-tfc-org' exists, wither delete it or creata new brnach with name 'update-tfc-org2'
```sh
git branch
```
Creata a new branch and checkout to it.
```sh
git checkout -b 'update-tfc-org'
```
Now commit the org name changes you made to the workflow files.
```sh
git add .github/workflows
```

Commit these changes with a message.
```sh
git commit -m 'Use our Terraform Cloud organization'
```

Push these changes.
```sh
git push origin update-tfc-org
```

Head to Github and, open a pull request from the update-tfc-org branch. From the base drop-down, choose the main branch.
<img src = "https://github.com/maveric-coder/Terraform/blob/main/files/content/8.png" >

Go to actions tab in GitHub and observe terraform execution. 
<br>As well head to Terraform cloud and see the created resources
<img src = "https://github.com/maveric-coder/Terraform/blob/main/files/content/9.png" >


