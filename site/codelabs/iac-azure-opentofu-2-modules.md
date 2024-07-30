summary: Infrastructure-as-Code (2) - Modularity and Automation with OpenTofu
id: iac-azure-opentofu-2-modules
categories: opentofu
tags: ktc-cnen, iac
status: Published
authors: Thomas Schuetz

# Infrastructure-as-Code (2) - Modularity and Automation with OpenTofu

<!-- ------------------------ -->

## Summary
In this lab, you will learn how to create a simple application in an autoscaled VM that utilizes data from a storage bucket using OpenTofu. You will also learn how to build a module for this application, store the state in GitLab, and create a pipeline in GitLab that automatically plans and executes the OpenTofu configuration after approval on push.

## Introduction
In this lab, we will:
* Create a simple application in an autoscaled VM
* Utilize data from a storage bucket
* Build a module for the application
* Store the state in GitLab
* Create a GitLab pipeline for automatic planning and execution

### Prerequisites
* GitLab account
* OpenTofu installed on your machine
* Git installed on your machine

## Step 1: Create a Simple Application in an Autoscaled VM

### Create the OpenTofu Configuration
1. Create a new directory for your OpenTofu configuration.
2. Create a file named `main.tf` with the following content:

```hcl
provider "azurerm" {
  features {}
}

resource "azurerm_resource_group" "example" {
  name     = "example-resources"
  location = "West Europe"
}

resource "azurerm_virtual_network" "main" {
  name                = "example-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.example.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_network_interface" "main" {
  name                = "example-nic"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name

  ip_configuration {
    name                          = "testconfiguration1"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_virtual_machine" "main" {
  name                  = "example-vm"
  location              = azurerm_resource_group.example.location
  resource_group_name   = azurerm_resource_group.example.name
  network_interface_ids = [azurerm_network_interface.main.id]
  vm_size               = "Standard_DS1_v2"

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

resource "azurerm_storage_account" "example" {
  name                     = "examplestorageacct"
  resource_group_name      = azurerm_resource_group.example.name
  location                 = azurerm_resource_group.example.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

resource "azurerm_storage_container" "example" {
  name                  = "content"
  storage_account_name  = azurerm_storage_account.example.name
  container_access_type = "private"
}
```

### Initialize and Apply the Configuration
1. Initialize the configuration:
    ```bash
    tofu init
    ```
2. Apply the configuration:
    ```bash
    tofu apply
    ```

## Step 2: Build a Module for the Application

### Create the Module
1. Create a new directory named `modules/application`.
2. Create a file named `main.tf` in the `modules/application` directory with the following content:

```hcl
resource "azurerm_virtual_machine" "main" {
  name                  = var.vm_name
  location              = var.location
  resource_group_name   = var.resource_group_name
  network_interface_ids = [var.network_interface_id]
  vm_size               = var.vm_size

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
    admin_username = var.admin_username
    admin_password = var.admin_password
  }

  os_profile_linux_config {
    disable_password_authentication = false
  }

  tags = var.tags
}
```

3. Create a file named `variables.tf` in the `modules/application` directory with the following content:

```hcl
variable "vm_name" {
  type = string
}

variable "location" {
  type = string
}

variable "resource_group_name" {
  type = string
}

variable "network_interface_id" {
  type = string
}

variable "vm_size" {
  type = string
}

variable "admin_username" {
  type = string
}

variable "admin_password" {
  type = string
}

variable "tags" {
  type = map(string)
}
```

4. Update the `main.tf` file in the root directory to use the module:

```hcl
module "application" {
  source              = "./modules/application"
  vm_name             = "example-vm"
  location            = azurerm_resource_group.example.location
  resource_group_name = azurerm_resource_group.example.name
  network_interface_id = azurerm_network_interface.main.id
  vm_size             = "Standard_DS1_v2"
  admin_username      = "testadmin"
  admin_password      = "Password1234!"
  tags                = {
    environment = "staging"
  }
}
```

## Step 3: Store the State in GitLab

### Configure GitLab
1. Create a new repository in GitLab.
2. Add the following `.gitlab-ci.yml` file to the root of your repository:

```yaml
stages:
  - validate
  - plan
  - apply

validate:
  stage: validate
  script:
    - tofu validate

plan:
  stage: plan
  script:
    - tofu plan -out=tfplan
  artifacts:
    paths:
      - tfplan

apply:
  stage: apply
  script:
    - tofu apply tfplan
  when: manual
```

## Step 4: Create a GitLab Pipeline

### Create the Pipeline
1. Push your changes to GitLab:
    ```bash
    git add .
    git commit -m "Initial commit"
    git push origin main
    ```
2. Go to your GitLab repository and navigate to CI/CD > Pipelines.
3. You should see the pipeline running. The `apply` stage will wait for manual approval.

### Approve and Execute the Pipeline
1. Click on the `apply` stage and approve it to execute the OpenTofu configuration.

<aside class="positive">
Congrats! You have successfully created a simple application in an autoscaled VM, utilized data from a storage bucket, built a module for the application, stored the state in GitLab, and created a GitLab pipeline for automatic planning and execution.
</aside>