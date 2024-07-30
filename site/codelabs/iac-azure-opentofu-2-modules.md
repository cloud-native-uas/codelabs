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
* Azure account
* GitLab account
* OpenTofu installed on your machine
* Git installed on your machine

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

## Step 2: Create a Simple Application in an Autoscaled VM

### Create the OpenTofu Configuration
* Create a new directory for your OpenTofu configuration.
* Create a file named `provider.tf` with the provider configuration:

```hcl
provider "azurerm" {
  features {}
}
```

* Afterward, create another file called `network.tf` containing the network configuration:

```
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

```

Furthermore, add a third configuration, that contains the configuration of the virtual machine and storage bucket for this configuration. Let's call this `main.tf`.

```
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

* Inspect your configuration and afterwards, destroy this configuration again (`tofu destroy`).

## Step 2: Build a Module for the Application

### Create the Module
* Create a new directory in the lab-2 directory, named modules. Furthermore, add another folder named "my-app" in it.
* Afterward, move all of the previously created files (except the provider.tf) into this new folder.
* Replace all static configuration in this configuration by variables and create the corresponding variable definitions. as an example, the variable for the storage account name could look like this:

```hcl
variable "storage_account_name" {
  type = string
}
```

* Afterward, create a folder called `terraform` in the lab-2 folder, and add a configuration according to your variables there. This might look as follows:

```hcl
module "application" {
  source              = "./modules/my-app"
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

* Copy the `provider.tf` file into this folder.
* Then, try this out to check if everything is working as intended

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