---
title: "Introduction to Terraform"
description: "Get started with terraform, learn basics, and write your first terraform file."
lead: "After almost seven (7) years in development, 11,000 pull requests, and 100 million downloads, Hashicorp Terraform has finally reached general availbility. Terraform has become a tool of choice for many DevOps teams for infrastructure provisioning and configuration. In this post we will review terraform basics and write our frist terraform script."
date: 2021-07-02T11:00:00-04:00
lastmod: 2021-07-02T11:00:00-04:00
draft: false
images: ["terraform-logo.png"]
weight: 50
contributors: ["Faheem Memon"]
---

{{< img src="terraform-logo.png" alt="Introduction to Terraform" class="wide" >}}

Cloud vendors provide their own scripting languages for infrastructure management, such as AWS's Cloud Formation or Azure's Resource Manager Templates, but scripting with those can easily get complicated, brittle, and hard to maintain. Terraform uses the simple yet powerful HashiCorp Configuration Language (HCL) which can be scaled easily to an entire enterprise without the side-effects. Terraform has been embraced across the industry and has also been gaining favor with major cloud providers. Check out Azure's [Bicep project](https://docs.microsoft.com/en-us/azure/azure-resource-manager/bicep/overview) or Google Cloud's recent [announcement](https://cloud.google.com/blog/products/management-tools/private-catalog-for-google-cloud-marketplace-supports-terraform) to support Terraform for Marketplace Private Catalogs.

Terraform not only works across cloud, but its vast ecosystem of plugins also covers on-premises virtualization systems or private clouds such as [vSphere](https://registry.terraform.io/providers/hashicorp/vsphere/latest), [Nutanix](https://registry.terraform.io/providers/nutanix/nutanix/latest), [Cisco ACI](https://registry.terraform.io/providers/CiscoDevNet/aci/latest), [HPE OneView](https://registry.terraform.io/providers/HewlettPackard/oneview/latest) and others. You get more milage out of Terraform compared to other tooling. It also works with common shared services such as [F5 BigIP](https://registry.terraform.io/providers/F5Networks/bigip/latest) load-balancers, [Infoblox](https://registry.terraform.io/providers/infobloxopen/infoblox/latest) Grid, Microsoft [Active Directory](https://registry.terraform.io/providers/hashicorp/ad/latest), and others. Additionally, it can be extended through its [plugin SDK](https://github.com/hashicorp/terraform-plugin-sdk) if needed. Remeber to check out the [Terraform Registry](https://registry.terraform.io/) and colloborate with existing open-source community before you build your own plugins.

In essence, Terraform manages the entire life-cycle of your infrastructure. It does not, however, replace your VM configuration management tooling. Once your infrastructure is up and running, you can switch over to your configuration management software and apply your configurations, recipes, or playbooks. It integrates nicely with push or pull-based configuration tools such as Ansible, Chef, Puppet, or others. Alternatively, you don't need any configuration management if you design your infrastructure to be immutable and be replaced instead of in-place updates.

Let's take Terraform for a spin and write some HCL.

## Installation

Terraform comes as a single binary that can be installed on a variety of operating systems and CPU architectures. Since I work on a Mac, I am downloading the `darwin` binary.

[Download Terraform](https://www.terraform.io/downloads.html)

```bash
wget https://releases.hashicorp.com/terraform/1.0.0/terraform_1.0.0_darwin_amd64.zip
unzip terraform*
sudo mv terraform /usr/local/bin
terraform --version

  Terraform v1.0.0
  on darwin_amd64
```

## Getting Started

Terraform HFCL files have `.tf` file extension. Let's create our first file, `main.tf`:

```tf
resource "random_password" "admin_password" {
  length           = 16
  special          = true
  override_special = "_%@"
}
```

And we are done! The code above will create a random password. We use the following three commands to run this code.

### 1. Initialize

Before we run our script, we have to initialize the "backend" and download the referenced plugins.  Let's look into these quickly:

* *Backend* is where terraform stores it state. After every code run, Terraform stores the configuration and the result in a state file. State can be stored locally or in remote backends. When working with a real environment, you have to share, version, backup, and secure your terraform state and remote backends are ideal. However, since we have not configured any remote backend, the terraform will create the state locally.

* We are using the [`random_password`](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password) *plugin* to create our password. This plugin is provided by HashiCorp and is available through their public registry.

```bash
$ terraform init
Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/random...
- Installing hashicorp/random v3.1.0...
- Installed hashicorp/random v3.1.0 (self-signed, key ID 34365D9472D7468F)

Partner and community providers are signed by their developers.
If you''d like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

The logs from the `init` command are pretty descriptive. Let's see what files are created in our directory.

```bash
$ ls -a
.terraform          .terraform.lock.hcl main.tf
```

`.terraform` is the folder where the plugins are downloaded. This folder is auto-generated and contains the binary files, it should not be included in the version control system.

### 2. Plan

The second step is to list the changes the script is going to make. Since this is our first time running this code, this step is not as helpful. However, it will be beneficial when you make changes to any existing infrastructure.

```bash
$ terrraform plan

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # random_password.admin_password will be created
  + resource "random_password" "admin_password" {
      + id               = (known after apply)
      + length           = 16
      + lower            = true
      + min_lower        = 0
      + min_numeric      = 0
      + min_special      = 0
      + min_upper        = 0
      + number           = true
      + override_special = "_%@"
      + result           = (sensitive value)
      + special          = true
      + upper            = true
    }

Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

We are adding one new resource, perfect. So let's apply these changes now.

### 3. Apply

```bash
$ terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # random_password.admin_password will be created
  + resource "random_password" "admin_password" {
      + id               = (known after apply)
      + length           = 16
      + lower            = true
      + min_lower        = 0
      + min_numeric      = 0
      + min_special      = 0
      + min_upper        = 0
      + number           = true
      + override_special = "_%@"
      + result           = (sensitive value)
      + special          = true
      + upper            = true
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

random_password.admin_password: Creating...
random_password.admin_password: Creation complete after 0s [id=none]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

Terraform `apply` command displays the planned changes one more time and prompts before applying these changes. We have successfully created the random password.

But wait, where is the password we just created? The password is created and stored in the terraform state. You wouldn't want to print the password to the screen, would you? Now, you can safely reference the password when creating another resource, such as a Virtual Machine, or store it in HashiCorp Vault by extending the HCL script.

For our exercise, however, let's look at the state file terraform created for us.

```bash
$ cat terraform.tfstate
{
  "version": 4,
  "terraform_version": "0.14.6",
  "serial": 1,
  "lineage": "77fd4a2d-351a-1700-e3d4-d74ebc9aabd8",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "random_password",
      "name": "admin_password",
      "provider": "provider[\"registry.terraform.io/hashicorp/random\"]",
      "instances": [
        {
          "schema_version": 0,
          "attributes": {
            "id": "none",
            "keepers": null,
            "length": 16,
            "lower": true,
            "min_lower": 0,
            "min_numeric": 0,
            "min_special": 0,
            "min_upper": 0,
            "number": true,
            "override_special": "_%@",
            "result": "DM4Skgcigxp@y3RX",
            "special": true,
            "upper": true
          },
          "sensitive_attributes": [],
          "private": "bnVsbA=="
        }
      ]
    }
  ]
}
```

You can see that the password is stored in the state as plain text. That is why you need to keep the state in a secure place. For added encryption, Terraform supports several remote backends that provide encryption at rest.

That's it for this post, in our next post we will create a virtual machine in AWS with some basic configuration. Stay tuned.
