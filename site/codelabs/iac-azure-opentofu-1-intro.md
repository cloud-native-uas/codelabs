summary: Infrastructure-as-Code (2) - Installing a Microservice Application with OpenTofu
id: iac-azure-opentofu-1-intro
categories: opentofu
tags: ktc-cnen, iac
status: Published
authors: Thomas Schuetz

# Infrastructure-as-Code (1) - First Steps with OpenTofu on Azure

<!-- ------------------------ -->

## Introduction
In this lab, we will introduce you to Infrastructure-as-Code (IaC) with OpenTofu. Therefore, we will take a step-by-step approach to create one resource after another and introduce you to the concepts of OpenTofu. We will create a Virtual Machine on Azure and install an application on it using Cloud-Init.

### What You’ll Learn
In this lab you will:
* Get a first introduction to Infrastructure-as-Code (IaC) with OpenTofu
* Create your first resources on Azure with OpenTofu
* Learn how to connect your resources and data sources in OpenTofu
* Automate the installation of an application on this Virtual Machine with Cloud-Init

### Prerequisites
* Azure Account
* OpenTofu installed on your machine
* Azure CLI installed on your machine

## Step 1: Verify the Azure Account
You need to have an Azure Account to follow this lab. If you don't have one yet, you can create one [here](https://azure.microsoft.com/en-us/free/).

On your lab environment, the Azure CLI should be installed. If you are not running this lab from a classroom environment, you can follow the instructions [here](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

After you have installed the Azure CLI, you can verify the installation by running the following command:

```bash
az --version
```

If you see the version of the Azure CLI, you have successfully installed the Azure CLI.

After you have installed the Azure CLI, you can log in to your Azure Account by running the following command:

```bash
az login --use-device-code
```

As our lab environment has no browser, we use the device code to authenticate. Therefore, open the URL in your browser and enter the code shown on the screen. After you have successfully authenticated, you should see your subscriptions. Choose the subscription you want to use for this lab.

## Step 2: Install OpenTofu
OpenTofu is a tool that allows you to create Infrastructure-as-Code (IaC) with a simple and easy-to-understand syntax. Like the Azure CLI, this should be installed on your machine. If you haven't installed OpenTofu yet, you can follow the instructions [here](https://opentofu.org/docs/intro/install/).

After you have installed OpenTofu, you can verify the installation by running the following command:

```bash
tofu --version
```

If you see the version of OpenTofu, you have successfully installed OpenTofu.

## Learning: Infrastructure-as-Code (IaC) and OpenTofu

### Infrastructure-as-Code (IaC)
There are many steps to get infrastructure working when doing this manually This would be fairly enough
to try out some new things, but in the real world you might want to do this in a more reproducible and scalable way.

This is where Infrastructure-as-Code comes into play. When you are provisioning 100s of instances, you might not want to
click through the consoles for every one you are provisioning. Infrastructure-as-Code (IaC) gives you the possibility to
configure your instances:

- As code
- In a declarative way
- Scalable
- With development best practices in mind.

Let's take a closer look on all of this.

_"Infrastructure-as-Code is an approach to infrastructure-automation based on practices from software development. It emphasizes
consistent, repeatable routines for provisioning and changing systems and their configuration. You make changes to code, then
use automation to test and apply those changes to your systems. (Kief Morris(2020), Infrastructure-as-Code, 2nd Edition)"_

When we're talking about Infrastructure-as-Code, we are no more creating our Infrastructure via the UI (ClickOps). Instead,
we are writing Code, mainly in a declarative way to describe the state, we want our Infrastructure to be in after it is applied.

Declarative configuration means that you express your intention (desired state), and a tool takes care of transitioning
the managed objects to this state. Using imperative language, you would tell the tooling what it should do. You can distinguish
between these two styles using the following statements:

- Imperative: Dear Tool, execute command A, B and C
- Declarative: Dear Tool, please transition my infrastructure to exactly this state

Please note, that each declarative tool, will have some kind of imperative logic in it to transition to the state.

When we're provisioning Infrastructure-as-Code, we often have the possibility to modularize components, as well as having mechanisms
in there, which help us to create multiple, similar objects. These are practices, we know from typical software development.
As we are using practices from Software Development, we should also treat our Infrastructure as such. Therefore, the code should
be versioned, there should be automatic tests in place and pull requests should be on the order of the day.

There are many tools out there for Infrastructure-as-Code like:

- OpenTofu/Terraform
- Pulumi
- Crossplane

Which tool you use, often depends on your own preferences and your use-cases. In this lab, we will take a closer look on OpenTofu.

Further Information / Videos:

- DevOps with Nana: Infrastructure-as-Code [https://www.youtube.com/watch?v=POPP2WTJ8es](https://www.youtube.com/watch?v=POPP2WTJ8es)

### OpenTofu
Before we will start with our first configuration, we will put some light on OpenTofu.

OpenTofu is an Infrastructure-as-Code tool forked from Terraform which was initially developed by Hashicorp. It uses a declarative language to describe infrastructure.
The language used is the "Hashicorp Configuration Language (HCL)". It is designed to work with many cloud providers. Nevertheless,
it is not really cloud-agnostic as knowledge about the individual providers is needed when writing the code.

To start using OpenTofu, we should be familiar with the following terms:

- **State:** The current configuration of our managed resources. The state can be stored in the local filesystem (terraform.tfstate),
  or remote backends, as S3 Buckets or consul
- **Provider:** Implements the commands needed to communicate with the cloud provider. e.g. an Exoscale provider knows how
  to talk to the Exoscale API and how to get from one state to another.
- **Resources:** The Objects you are creating and managing with OpenTofu. As an example, a virtual machine is a resource.
- **Data Sources:** Describe objects we can query (!= write) in OpenTofu. In amazon, the AMIs (Amazon Machine Images / Templates)
  are implemented as datasources you can query
- **Variables:** Give us the possibility to parametrize terraform configurations.

For this lab, we will use a local state, Azure as provider and a Compute resource to create our Virtual Machine.

## Step 3: Create a new OpenTofu Configuration
Normally, you start a new OpenTofu configuration by creating a new directory and a new file in it. The file should have the
extension `.tf`. In this file, you can describe your resources. Note, that you can use multiple files in one configuration, 
OpenTofu will merge them together.

To start with our configuration, we will create a new directory and a new file in it. The file should have the name `provider.tf`.

### OpenTofu Provider Configuration
The objects you are creating in OpenTofu are managed by a provider. The provider is responsible for understanding API interactions
and exposing resources. In our case, we will use the Azure provider. These providers are implemented as plugins and hosted on the
OpenTofu or Terraform Registry. You can find the Azure provider [here](https://registry.terraform.io/providers/hashicorp/azurerm/latest).

After you have created the file, you can add the following content to it:

```hcl
provider "azurerm" {
  features {}
}
```

In this block, you see the typical structure of a hcl configuration. The first word is the type of the object you are creating, the second word is the type of the object you are creating. The block is enclosed in curly braces. The provider block is a special block in OpenTofu, as it is responsible for the communication with the cloud provider. In typical HCL configuration blocks, you would also give the block a name, but in the provider block, this is not necessary.

Additionally, it absolutely makes sense to pin the version of the provider you are using. This can be done by adding and additional block to the configuration:

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "2.0.0"
    }
  }
}
```

In this block, you see the `required_providers` block. In this block, you can define the providers you are using in your configuration. The `source` attribute is the source of the provider, the `version` attribute is the version of the provider you are using. This is a good practice to ensure that your configuration is working with the provider you are using and is reproducible.

After you have created the provider configuration, you can verify the configuration by running the following command:

```bash
tofu init
```

If you see the message `OpenTofu has been successfully initialized`, you have successfully created the provider configuration.

## Step 4: Create OpenTofu Resources and Variables
In this step, we will first create the objects needed to create a Virtual Machine on Azure. Therefore, we will create a new file in the same directory as the `provider.tf` file. The file should have the name `main.tf`.

**Resources**: In OpenTofu, managed objects you can directly create and configure are called resources.
**Variables**: Variables give us the possibility to parametrize our configuration.

### Resource Group
The first thing we need to create a Virtual Machine is a resource group. The resource group is a logical container for resources deployed on Azure. Therefore, we will create a new resource group in Azure. To do this, you can add the following content to the `main.tf` file:

```hcl
resource "azurerm_resource_group" "example" {
  name     = "${var.prefix}-resources"
  location = "West Europe"
}
```

As described before, the first word is the type of the object you are creating, the second word is the name of the object you are creating and the third one gives the resource a name. The block is enclosed in curly braces. In this block, you see the `name` attribute, which is the name of the resource group. The `location` attribute is the location of the resource group. The location is a string and must be one of the supported Azure locations. You can find a list of the supported locations [here](https://azure.microsoft.com/en-us/global-infrastructure/locations/).

In this block, you see the `${var.prefix}` syntax. This is a variable reference. You can define variables in OpenTofu by adding a `variables` block to the configuration and it is a good practice to define variables in a separate file. You can find more information about variables [here](https://www.terraform.io/docs/language/values/variables.html).

```hcl
variable "prefix" {
  type    = string
  default = "opentofu"
}
```

Please note, that you can define the type of the variable and a default value. The default value is used when no value is given to the variable. In your case, please add your name to the `prefix` variable.

### Validate - Plan - Apply
After you have created the configuration for the resource group, you can verify the configuration by running the following command:

```bash
tofu validate
```

If you see the message `Success! The configuration is valid.`, your configuration is at least syntactically correct.

After that you can take a look at the execution plan by running the following command:

```bash
tofu plan
```

If you see the plan of the resources that will be created, your configuration is correct.

Finally, you can apply the configuration by running the following command:

```bash
tofu apply
```

After this command has finished, you should see the resource group in the Azure Portal.

<aside class="positive">
You have successfully created your first resources on Azure with OpenTofu. In the next step, we will create a Virtual Network and a Virtual Machine.
</aside>

## Step 5: Connecting Resources
In this step, we will create a Virtual Network and a Virtual Machine. The Virtual Network is a logical isolation of the Azure resources in a network. The Virtual Machine is a resource that you can use to run your applications. Like in networking, you can also connect resources in OpenTofu as we will see in this step.

### Virtual Network
The first thing we need to create a Virtual Machine is a Virtual Network. Therefore, we will create a new Virtual Network in Azure. To do this, you can add the following content to the `main.tf` file:

```hcl
resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
}
```

In this block, you see the `azurerm_virtual_network` resource. The `name` attribute is the name of the Virtual Network. The `address_space` attribute is the address space of the Virtual Network. The `location` attribute is the location of the Virtual Network. The `resource_group_name` attribute is the name of the resource group the Virtual Network is in. In the location and resource_group_name attributes, you see the `azurerm_resource_group.example.location` and `azurerm_resource_group.example.name` syntax. This is a reference to the resource group we have created before and follows the pattern `<type>.<resource_name>.<attribute>`.

Like after each change, you can validate, plan and apply the configuration by running the following commands:

```bash
tofu validate
tofu plan
tofu apply
```

After this command has finished, you should see the Virtual Network in the Azure Portal.

The virtual machine also needs a subnet and a network interface to be connected to the Virtual Network. Therefore, we will create a subnet in the Virtual Network. To do this, you can add the following content to the `main.tf` file:

```hcl
# Create Subnet
resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

# Create Network Interface
resource "azurerm_network_interface" "main" {
  name                = "${var.prefix}-nic"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name

  ip_configuration {
    name                          = "testconfiguration1"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}
```

In this block, you see the `azurerm_subnet` resource. The `name` attribute is the name of the subnet. The `resource_group_name` attribute is the name of the resource group the subnet is in. The `virtual_network_name` attribute is the name of the Virtual Network the subnet is in. The `address_prefixes` attribute is the address prefix of the subnet.

### Virtual Machine
The last thing we need to create a Virtual Machine is a Virtual Machine. Therefore, we will create a new Virtual Machine in Azure. To do this, you can add the following content to the `main.tf` file:

```hcl
# Create Virtual Machine
resource "azurerm_virtual_machine" "main" {
  name                  = "${var.prefix}-vm"
  location              = azurerm_resource_group.example.location
  resource_group_name   = azurerm_resource_group.example.name
  network_interface_ids = [azurerm_network_interface.main.id]
  vm_size               = "Standard_DS1_v2"

  # Uncomment this line to delete the OS disk automatically when deleting the VM
  # delete_os_disk_on_termination = true

  # Uncomment this line to delete the data disks automatically when deleting the VM
  # delete_data_disks_on_termination = true

  storage_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  storage_os_disk {
    name              = "myosdisk1"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }
  os_profile {
    computer_name  = "hostname"
    admin_username = "testadmin"
    admin_password = "Password1234!"
  }
  os_profile_linux_config {
    disable_password_authentication = false
  }
  tags = {
    environment = "staging"
  }
}
```

In this block, you see the `azurerm_virtual_machine` resource. The `name` attribute is the name of the Virtual Machine. The `location` attribute is the location of the Virtual Machine. The `resource_group_name` attribute is the name of the resource group the Virtual Machine is in. The `network_interface_ids` attribute is the id of the network interface the Virtual Machine is connected to. The `vm_size` attribute is the size of the Virtual Machine. The `storage_image_reference` block is the image of the Virtual Machine. The `storage_os_disk` block is the OS disk of the Virtual Machine. The `os_profile` block is the profile of the Virtual Machine. The `os_profile_linux_config` block is the Linux configuration of the Virtual Machine. The `tags` attribute is the tags of the Virtual Machine.

Finally, you can validate, plan and apply the configuration by running the following commands:

```bash
tofu validate
tofu plan
tofu apply
```

After this command has finished, you should see the Virtual Machine in the Azure Portal.

<aside class="positive">
You have successfully created a Virtual Network and a Virtual Machine on Azure with OpenTofu. In the next step, we will make some changes on the configuration to make this a bit prettier.
</aside>

## Step 6: Dealing with Sensitive Data
In the first version of our configuration, we have created a Virtual Machine Object with a static password.

In the first version of our configuration, we have used a static password to log in to the Virtual Machine. This is not a good practice as the password is stored in the configuration in plain text. To avoid this, we will use a variable for the password and hand it over to OpenTofu as an environment variable. To do this, you can add the following content to the `variables.tf` file:

```hcl
variable "admin_username" {
  type    = string
  default = "testadmin"
}

variable "admin_password" {
  type    = string
  sensitive = true
}
```

In this block, you see the `admin_username` and `admin_password` variables. The `admin_username` variable is the username of the Virtual Machine. The `admin_password` variable is the password of the Virtual Machine. The `sensitive` attribute is set to `true` for the `admin_password` variable. This means that the value of the variable is not shown in the output of OpenTofu.

After you have created the variables, you have to change the relevant things in the virtual machine block of the `main.tf` file:

```hcl
  os_profile {
    computer_name  = "hostname"
    admin_username = var.admin_username
    admin_password = var.admin_password
  }
```

If you have changed the configuration, you can validate, plan and apply the configuration by running the same commands as before. In the apply step, you will be asked for the value of the `admin_password` variable. After you have entered the value, the configuration will be applied. If you want to pass over the value of the `admin_password` as an environment variable, you can either use the `-var` flag or create an environment variable with the name `TF_VAR_admin_password`.

## Step 7: Using Data Sources to Query Data
In the first version of our configuration, we have created the Virtual Network by ourselves. In some environments, these objects are already there and you just want to query them. In OpenTofu, you can use data sources to query objects. In this step, we will use a data source to query the Virtual Network. To do this, you can add the following content to the `main.tf` file:

```hcl
# Query Virtual Network
data "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  resource_group_name = "your-resource-group-name"
}
```

The data queried from the data source can also be used to query other objects. In this case, we will use the data queried from the Virtual Network data source to query the subnet. To do this, you can add the following content to the `main.tf` file:

```hcl
# Query Subnet
data "azurerm_subnet" "internal" {
  name                 = "<your subnet name>"
  virtual_network_name = data.azurerm_virtual_network.main.name
  resource_group_name  = "your-resource-group-name"
}
```

With the queried data, you can create the network interface. To do this, you can add the following content to the `main.tf` file:

```hcl
# Create Network Interface
resource "azurerm_network_interface" "main" {
  name                = "${var.prefix}-nic"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name

  ip_configuration {
    name                          = "testconfiguration1"
    subnet_id                     = data.azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}
```

As you can see, the `subnet_id` attribute is set to `data.azurerm_subnet.internal.id`. This is a reference to the id of the subnet queried from the data source.

Finally, you can validate, plan and apply the configuration by running the same commands as before.

<aside class="positive">
You have successfully created a Virtual Network and a Virtual Machine on Azure with OpenTofu. Now you should be able to log in to the Virtual Machine with the password you have entered in the apply step. In the next step, we will automate the installation of an application on the Virtual Machine with Cloud-Init.
</aside>

## Step 8: Automate the Installation of an Application with Cloud-Init
In the first version of our configuration, we have created a Virtual Machine with a static password. In the second version of our configuration, we have used a variable for the password. In this version, we will automate the installation of an application on the Virtual Machine with Cloud-Init. Cloud-Init is a multi-distribution package that handles early initialization of a cloud instance. It is supported across all major public cloud providers and private cloud platforms. To use Cloud-Init, you have to add the `custom_data` attribute to the `os_profile` block of the Virtual Machine. To do this, you can add the following content to the `main.tf` file:

```hcl
  os_profile {
    computer_name  = "hostname"
    admin_username = var.admin_username
    admin_password = var.admin_password
    custom_data    = file("cloud-init.txt")
  }
```

In this block, you see the `custom_data` attribute. The `custom_data` attribute is the custom data of the Virtual Machine. The `file` function reads the content of the `cloud-init.txt` file and passes it to the `custom_data` attribute.

After you have added the `custom_data` attribute, you have to create the `cloud-init.txt` file in the same directory as the `main.tf` file. The `cloud-init.txt` file should have the following content:

```bash
#cloud-config
package_upgrade: true
packages:
  - nginx
runcmd:
  - cd /var/www/html
  - echo "Hello, World!" > index.html
  - systemctl start nginx
```

In this file, you see the `#cloud-config` header. This header is required for Cloud-Init. The `package_upgrade` attribute is set to `true`. This means that the packages on the Virtual Machine will be upgraded. The `packages` attribute is a list of packages that will be installed on the Virtual Machine. In this case, the `nginx` package will be installed. The `runcmd` attribute is a list of commands that will be executed on the Virtual Machine. In this case, the current directory will be changed to `/var/www/html`, the `Hello, World!` string will be written to the `index.html` file and the `nginx` service will be started.

Finally, you can validate, plan and apply the configuration by running the same commands as before.

<aside class="positive">
You have successfully automated the installation of an application on the Virtual Machine with Cloud-Init. Now you should be able to see the `Hello, World!` string in the `index.html` file on the Virtual Machine. Congratulations! You have completed the lab. In the next step, we will summarize the lab and give you some further information.
</aside>

## Summary
In this lab, we have introduced you to Infrastructure-as-Code (IaC) with OpenTofu. Therefore, we have created a Virtual Machine on Azure and installed an application on it with Cloud-Init. We have taken a step-by-step approach to create one resource after another and introduced you to the concepts of OpenTofu. We have created a Virtual Machine on Azure and installed an application on it with Cloud-Init. We have learned how to connect resources and data sources in OpenTofu. We have automated the installation of an application on the Virtual Machine with Cloud-Init.

### Further Information
* [OpenTofu Documentation](https://opentofu.org/docs/intro/)
* [Azure Documentation](https://docs.microsoft.com/en-us/azure/)
* [Cloud-Init Documentation](https://cloudinit.readthedocs.io/en/latest/)
* [Terraform Documentation](https://www.terraform.io/docs/index.html)
* [Hashicorp Configuration Language (HCL) Documentation](https://www.terraform.io/docs/language/syntax/configuration.html)
* [Azure CLI Documentation](https://docs.microsoft.com/en-us/cli/azure/)
* [Azure Locations](https://azure.microsoft.com/en-us/global-infrastructure/locations/)


