# Script to create structure
#Terraform #TerraformInAction #Architecture

Example of file structure we are gonna create. File description in the script below.
But we will create another module calles `virtualmachines` which will do the provisioning of the VMs. The `root` part will only contain input, output and loading the modules.

The general idea behind it
![[TerraformInAction_Folder_Structure.png]]

### Overview of file structure

```plantuml
map "**root [./]**" as root {
  main.tf  => Entry point
  variables.tf => Input variables to\ncustomize deployment
  outputs.tf => Deployment output variables
  providers.tf => Provider declarations
  versions.tf => Provider version locking
}

map "**keys**" as keys {
  public_ssh.key => Public key to deploy
}

map "**modules/virtual_machines**" as vms {
  main.tf  => Entry point
  variables.tf => Input variables to\ncustomize deployment
  outputs.tf => Deployment output variables
  versions.tf => Provider version locking
}

map "**modules/network**" as networks {
  ... => ...
}

map "**modules/databases**" as databases {
  ... => ...
}

root::main --> vms: loads in main.tf
root::main ..> networks: loads in main.tf
root::main ..> databases: loads in main.tf
```

## Script to create initial Terraform folder structure
Creates basic file structure and configuration

```bash
# modules directory where the actual code is same structure as this root
mkdir modules
# Something we always need is a module for provisioning vms
mkdir modules/virtual_machines
touch modules/virtual_machines/main.tf
touch modules/virtual_machines/variables.tf
touch modules/virtual_machines/outputs.tf
touch modules/virtual_machines/providers.tf
touch modules/virtual_machines/versions.tf

# Stores ssh public and optionally private keys
mkdir keys

# **variables.tf** Input variables to customize the deployments
tee -a ./variables.tf << END
variable "project_name" {
  description = "The project to use for unique resource naming in combination with environment"
  default     = "#TODO: name is e.g. shop"
  type        = string
}

variable "environment" {
  description = "The environment to use for unique resource naming in combination with environment"
  type        = string
  default     = "development"
  validation {
    condition     = contains(["test", "development", "staging", "production"], var.environment)
    error_message = "Valid values for var: test_variable are ('test', 'development', 'staging', 'production')."
  }
}

variable "region" {
  description = "Region identifier to use per provider"
  default     = "#TODO: Provider specific regsion identifier e.g. DE1"
  type        = string
}
END

# **main.tf** Entry point for Terraform
tee -a ./main.tf << END
# Loads our main module to provisioning vms
module "virtual_machines" {
  source       = "./modules/virtual_machines"
  project_name = var.project_name
  environment  = var.environment
}
END

# **output.tf** Deployment output variables
tee -a ./output.tf << END
# List created vms
output "servers" {
  sensitive = true
  value     = module.virtual_machines.server
}
END

# **providers.tf** - Terraform Provider declarations
tee -a ./providers.tf << END
#TODO: Provider configuration file https://registry.terraform.io/browse/providers. Provider examples see below.
END

# **versions.tf** - Terraform Provider declarations
tee -a ./versions.tf << END
#TODO: Provider lib and its versions https://registry.terraform.io/browse/providers. Provider examples see below.
END
```
After the above script is executed you can delete it.
Select one of the below example provider configurations to setup a VM.

## Example configuration
* [[Terraform OVH Cloud VM]]