---
layout: post
title: Error setting billing account with terraform on Google Cloud Platform
date: 2020-03-22 14:16
categories: [GCP, gcloud, terraform]
---

## GCP project creation with Terraform

Terraform allows creating project in Google Cloud Platform. Example configuration:

```yaml
provider "google" {
  project = var.project_id
  region = var.region
}

resource "google_project" "test-project" {
  name       = var.project_name
  project_id = var.project_id
  org_id     = var.org_id
  billing_account = var.billing_account
}

```

Please note that `billing_account` is assigned to that project.

Normal terrafrom flow that creates and deletes the project.

```shell
$ terraform init
$ terraform plan
$ terraform apply
$ terraform destroy
```

## Error setting billing account


I was testing different terraform configurations and creating and destroying playground projects when I got this error message:

```shell
$ pawel@pawel:~/Repo/services/environments/test$ terraform apply -auto-approve
google_project.test-project: Refreshing state... [id=projects/test-project]
google_project.test-project: Modifying... [id=projects/test-project]

Error: Error setting billing account "010DDB-F8DB5E-2C64A7" for project "projects/test-project": googleapi: Error 400: Precondition check failed., failedPrecondition

  on main.tf line 7, in resource "google_project" "test-project":
   7: resource "google_project" "test-project" {
```

I tried to use `gcloud` for setting a billing account but got similar and more descriptive error:


```
pawel@pawel:~/Repo/services/environments/test$ gcloud beta billing projects link test-project --billing-account XXXXXX-XXXXXX-XXXXXX
ERROR: (gcloud.beta.billing.projects.link) FAILED_PRECONDITION: Precondition check failed.
- '@type': type.googleapis.com/google.rpc.QuotaFailure
  violations:
  - description: 'Cloud billing quota exceeded: https://support.google.com/code/contact/billing_quota_increase'
    subject: billingAccounts/XXXXXX-XXXXXX-XXXXXX
```


## Solution

It seems that deleted projects hold onto the quota of max projects linked into a billing account.
I think it applies only to projects that can be still recovered (30 days after deletion).
Solution is to restore the projects, unlink billing accounts and delete them again:

```shell
$ gcloud projects undeleted your-project-id
$ gcloud beta billing projects unlink your-project-id
$ gcloud projects deleted your-project-id
```
