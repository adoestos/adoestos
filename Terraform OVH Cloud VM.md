#Terraform #OVH #VirtualMachines

Based on the file/folder structure from [[Terraform projects structure]] we are going over all steps to configure and create `VMS` on `OVH`

# OVH OpenStack credentials

Sources:
- https://registry.terraform.io/providers/ovh/ovh/latest/docs

Following things are required from OVH and Openstack provider.
> Besides the `API end-point`, the required keys are the `application_key`, the `application_secret`, and the `consumer_key`. These keys can be generated via the [OVHcloud token generation page](https://api.ovh.com/createToken/?GET=/*&POST=/*&PUT=/*&DELETE=/*). 
><cite>https://registry.terraform.io/providers/ovh/ovh/latest/docs</cite>

Goto the  [OVHcloud token generation page](https://api.ovh.com/createToken/?GET=/*&POST=/*&PUT=/*&DELETE=/*) , login and replace the values with the script below.
This script will store the credentials in a script wich we can use later to makes them available as environment variables for a console session.

### Secure secret management storage

Source:
* [[OpenSSL managing credentials]] Simple example and sources

```bash
if [ -z "$1" ]; then  
	exit 1
else  
	echo "Shell script will be stored here: $1"  
fi

# Create 16 CSPRNG bytes base64 encoded as a passwort to be able to decrypt it later
KEY=$(openssl rand -base64 16)

# Encrypt variable and store it as a base64 value
# The parameters are taken from the OWASP Cheatsheet link above
APPLICATION_KEY=$(echo -n "{{APPLICATION_KEY}}" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000 | openssl enc -base64 -k $KEY)
APPLICATION_SECRET=$(echo -n "{{APPLICATION_SECRET}}" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000 | openssl enc -base64 -k $KEY)
CONSUMER_KEY=$(echo -n "{{CONSUMER_KEY}}" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000 | openssl enc -base64 -k $KEY)

# Create script 
cat > ./$1 << EOF  
echo "Enter your decryption key:"  
read -s DECRYPT_KEY

export APPLICATION_KEY=\$(echo "$APPLICATION_KEY" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k $DECRYPT_KEY)
export APPLICATION_SECRET=\$(echo "$APPLICATION_SECRET" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k $DECRYPT_KEY)
export CONSUMER_KEY=\$(echo "$CONSUMER_KEY" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k $DECRYPT_KEY)

EOF

echo "Decryption key: $KEY"
```

Steps:
1. Replace the `{{Placeholders}}` with the actual keys
2. Execute this script with a parameter to store the actual script.
3. Save the Key in the stdout on the secure place you will need it to encrypt the values
4. Store the generated script
5. Execute the script and add the `decryption key` when requested.

# Provider configuration
TODO

# Version configuration
TODO
