#Terraform #OVH #VirtualMachines

Based on the file/folder structure from [[Terraform projects structure]] we are going over all steps to configure and create `VMS` on `OVH`

This will be the final infrastructure
```plantuml
cloud OVH {
	rectangle DBCluster {
		node dbcluster_vm_1
		node dbcluster_vm_2
		node dbcluster_vm_3
	}

	rectangle WorkerCluster {
		node workercluster_vm_1
		node workercluster_vm_2 
		node workercluster_vm_3
		node workercluster_vm_4
	}
}
```

# OVH and OVH OpenStack credentials

## OVH credentials
Sources:
- https://registry.terraform.io/providers/ovh/ovh/latest/docs

Following things are required from OVH and Openstack provider.
> Besides the `API end-point`, the required keys are the `application_key`, the `application_secret`, and the `consumer_key`. These keys can be generated via the [OVHcloud token generation page](https://api.ovh.com/createToken/?GET=/*&POST=/*&PUT=/*&DELETE=/*). 
><cite>https://registry.terraform.io/providers/ovh/ovh/latest/docs</cite>

Goto the  [OVHcloud token generation page](https://api.ovh.com/createToken/?GET=/*&POST=/*&PUT=/*&DELETE=/*)  login and store the provided credentials. We will later store them in a secure way.

### OVH Openstack credentials

To optain the Openstack credentials from OVH we have to login at [OVH](https://www.ovh.com/auth/) 

![[Pasted image 20230524115729.png]]
Follow the steps above to create a new Openstack user. Don't foger to store the OS_PASSWORD which is only visible after creating the new user. Download the OpenStack RC file which contains all credentials.

### Secure secret management storage

We are going to encrypt **all** the given information we use to configure OpenStack and OVH providers.

Source:
* [[OpenSSL managing credentials]] Simple example and sources and references about the given parameters.

```bash
if [ -z "$1" ]; then
        echo "Please provide the file path to store the exporter."
        exit 1
else
        echo "Shell script will be stored here: $1"
fi

# Create 16 CSPRNG bytes base64 encoded as a passwort to be able to decrypt it later
KEY=$(openssl rand -base64 16)

# Encrypt variable and store it as a base64 value
# The parameters are taken from the OWASP Cheatsheet link above

# Encrypt OVH Provider config/credentials
OS_AUTH_URL=$(echo -n "{{OS_AUTH_URL from OVH Public Cloud openrc.sh}}" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000  -k $KEY | openssl enc -base64)
OS_IDENTITY_API_VERSION=$(echo -n "{{OS_IDENTITY_API_VERSION from OVH Public Cloud openrc.sh}}" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000  -k $KEY | openssl enc -base64)

OS_USER_DOMAIN_NAME=$(echo -n "{{OS_USER_DOMAIN_NAME from OVH Public Cloud openrc.sh}}" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000  -k $KEY | openssl enc -base64)
OS_PROJECT_DOMAIN_NAME=$(echo -n "{{OS_PROJECT_DOMAIN_NAME from OVH Public Cloud openrc.sh}}" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000  -k $KEY | openssl enc -base64)

OS_TENANT_ID=$(echo -n "{{OS_TENANT_ID from OVH Public Cloud openrc.sh}}" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000  -k $KEY | openssl enc -base64)
OS_TENANT_NAME=$(echo -n "{{OS_TENANT_NAME from OVH Public Cloud openrc.sh}}" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000  -k $KEY | openssl enc -base64)

OS_USERNAME=$(echo -n "{{OS_USERNAME from OVH Public Cloud openrc.sh}}" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000  -k $KEY | openssl enc -base64)
OS_PASSWORD=$(echo -n "{{OS_PASSWORD from OVH Public Cloud openrc.sh}}" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000  -k $KEY | openssl enc -base64)

# Encrypt OVH Provider config/credentials
OVH_APPLICATION_KEY=$(echo -n "{{APPLICATION_KEY from OVH API}}" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000  -k $KEY | openssl enc -base64)
OVH_APPLICATION_SECRET=$(echo -n "{{APPLICATION_SECRET from OVH API}}" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000  -k $KEY | openssl enc -base64)
OVH_CONSUMER_KEY=$(echo -n "{{CONSUMER_KEY from OVH API}}" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000  -k $KEY | openssl enc -base64)


# Create script which contains the encrypted confi/credentials
cat > ./$1 << EOF
echo "Enter your decryption key:"
read -s DECRYPT_KEY

# OpenStack Provider config/credentials

export OS_AUTH_URL=\$(echo "$OS_AUTH_URL" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k \$DECRYPT_KEY)
export OS_IDENTITY_API_VERSION=\$(echo "$OS_IDENTITY_API_VERSION" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k \$DECRYPT_KEY)

export OS_USER_DOMAIN_NAME=\$(echo "$OS_USER_DOMAIN_NAME" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k \$DECRYPT_KEY)
export OS_PROJECT_DOMAIN_NAME=\$(echo "$OS_PROJECT_DOMAIN_NAME" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k \$DECRYPT_KEY)

export OS_TENANT_NAME=\$(echo "$OS_TENANT_NAME" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k \$DECRYPT_KEY)
export OS_TENANT_ID=\$(echo "$OS_TENANT_ID" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k \$DECRYPT_KEY)

export OS_USERNAME=\$(echo "$OS_USERNAME" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k \$DECRYPT_KEY)
export OS_PASSWORD=\$(echo "$OS_PASSWORD" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k \$DECRYPT_KEY)

# OVH Provider config/credentials
export OVH_APPLICATION_KEY=\$(echo "$OVH_APPLICATION_KEY" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k \$DECRYPT_KEY)
export OVH_APPLICATION_SECRET=\$(echo "$OVH_APPLICATION_SECRET" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k \$DECRYPT_KEY)
export OVH_CONSUMER_KEY=\$(echo "$OVH_CONSUMER_KEY" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k \$DECRYPT_KEY)

EOF

echo "Decryption key: $KEY"
```

Steps:
1. Replace the `{{Placeholders}}` with the actual keys
2. Execute this script with a parameter to store the actual script. e.g. `sh create_credfile.sh export_ovh_secrets.sh`
3. Save the Key in the stdout on the secure place you will need it to encrypt the values
4. Execute the generated script with e.g. `source export_ovh_secrets.sh` and add the `decryption key` when requested.

To see if all env variables are set correclty execute this (make sure noone is behind you)
```bash
echo "OS_AUTH_URL: $OS_AUTH_URL"
echo "OS_IDENTITY_API_VERSION: $OS_IDENTITY_API_VERSION"
echo "OS_USER_DOMAIN_NAME: $OS_USER_DOMAIN_NAME"
echo "OS_PROJECT_DOMAIN_NAME: $OS_PROJECT_DOMAIN_NAME"

echo "OS_TENANT_NAME: $OS_TENANT_NAME"
echo "OS_TENANT_ID: $OS_TENANT_ID"

echo "OS_USERNAME: $OS_USERNAME"
echo "OS_PASSWORD: $OS_PASSWORD"

echo "OVH_APPLICATION_KEY: $OVH_APPLICATION_KEY"
echo "OVH_APPLICATION_SECRET: $OVH_APPLICATION_SECRET"
echo "OVH_CONSUMER_KEY: $OVH_CONSUMER_KEY"
```

### Input configuration | `modules/virtual_machines/` 
```bash
variable "ssh_keypair" {
  description = "SSH keypair to use for VM instances"
  default     = "../files/keys/public.ssh"
  type        = string
}

# Amount of db server nodes
variable "db_cluster_count {
  type    = number
  default = 3
}

# Amount of worker nodes
variable "worker_node_count" {
  type    = number
  default = 4
}
```

### Application configuration | `main.tf`
Here is the application itself. In our case we will just load the vm

```bash
module "virtual_machines" {
  source       = "./modules/virtual_machines"
  project_name = var.project_name # Declared and initialised in root/variables.tf
  environment  = var.environment # Declared and initialised in root/variables.tf
}
```

### Provider configuration | `providers.tf`
As `OVH` is using internally `OpenStack` for VM management we have to load [OpenStack Provider](https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs) first. We only need those in the root `versions.tf` as it will be interit to the modules.

```bash
# providers.tf
# All configs are provided by environment variables
provider "openstack" {
}

provider "ovh" {
}
```

### Version configuration | `versions.tf` and `modules/virtual_machines/versions.tf`

Here we add the version information of the libs we and our submodules will use. As this is required in the root and all modules this libs is used we must add it in two `version.tf` files.

```bash
# versions.tf and modules/virtual_machines/versions.tf
terraform {
  required_version = ">= 0.15"
  required_providers {
    openstack = {
      source  = "terraform-provider-openstack/openstack"
      version = "~> 1.51.1"
    }
    ovh = {
      source  = "ovh/ovh"
      version = "~> 0.30"
    }
  }
}
```

### VM configuration | `modules/virtual_machines/mail.tf`
The actual module implemtation which is just upload new public key and ramp up the servers.

```bash
# modules/virtual_machines/main.tf

# Creating an SSH key pair resource the public key will be provided to each vm
# Resource reference: https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/compute_keypair_v2
resource "openstack_compute_keypair_v2" "vm_keypair" {
  provider   = openstack.ovh         # Provider name declared in provider.tf
  name       = "vm_keypair"        # Name of the SSH key to use for creation
  public_key = file(var.ssh_keypair) # Path to your previously generated SSH key
}

# Creates a VM with the name Test
# Resource reference: https://registry.terraform.io/providers/terraform-provider-openstack/openstack/latest/docs/resources/compute_instance_v2
# DB Cluster server
resource "openstack_compute_instance_v2" "db_servers" {
  count       = var.db_cluster_count
  name        = "dbcluster_vm_${count.index}"     # Instance name
  provider    = openstack.ovh  # Provider name/alias
  image_name  = "Ubuntu 22.10" # Image name
  flavor_name = "s1-2"         # Instance type name
  # Name of openstack_compute_keypair_v2 resource named keypair_test
  key_pair = openstack_compute_keypair_v2.test_keypair.name
  network {
    name = "Ext-Net" # Adds the network component to reach your instance
  }
}

# Worker nodes
resource "openstack_compute_instance_v2" "worker_servers" {
  count       = var.worker_node_count
  name        = "workercluster_vm_${count.index}"     # Instance name
  provider    = openstack.ovh  # Provider name/alias
  image_name  = "Ubuntu 22.10" # Image name
  flavor_name = "s1-2"         # Instance type name
  # Name of openstack_compute_keypair_v2 resource named keypair_test
  key_pair = openstack_compute_keypair_v2.vm_keypair.name
  network {
    name = "Ext-Net" # Adds the network component to reach your instance
  }
}
```

 ### Other Examples
* [[Terraform OVH Subnets]]
* [[Terraform OVH Databases]]
