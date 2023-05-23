#Openssl #Encryption #Decryption #Secrets

Create password secured script which stores inline encrypted variables and provide them as environment variables once executed with correct password.

Sources:
* [OWASP Cheatsheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) for getting the recommendated parameters
* [Openssl rand doc](https://www.openssl.org/docs/man1.1.1/man1/rand.html) to create random password

```bash
if [ -z "$1" ]; then  
	echo "Please provide the filename on where to store the encrypted key."  
	exit 1
else  
	echo "Shell script will be stored here: $1"  
fi

# Create 16 CSPRNG bytes base64 encoded as a passwort to be able to decrypt it later
KEY=$(openssl rand -base64 16)

# Encrypt variable and store it as a base64 value
# The parameters are taken from the OWASP Cheatsheet link above
ENCRYPTED_VALUE=$(echo -n "MySuperSecretPassword!" | openssl enc -aes-256-cbc -salt -pbkdf2 -iter 600000 | openssl enc -base64 -k $KEY)

# Create script 
cat > ./$1 << EOF  
if [ -z "$1" ]; then  
	echo "Please provide the password to decrypt the encrypted values."  
	exit 1
fi

export DECRYPT_VALUE=\$(echo "$ENCRYPTED_VALUE" | openssl enc -d -base64 | openssl enc -d -aes-256-cbc -salt -pbkdf2 -iter 600000 -k $1)
EOF

echo "Key: $KEY"
```

## How to use this script

1. Replace `MySuperSecretPassword!` with the actual secret to encrypt.
2. Store the script e.g. as  `encrypt_secret.sh`
3. Execute script e.g. with `sh encrypt_secret.sh export_secret.sh`
4. Store output key value on a secure place
5. Execute generated script to export secret e.g. `sh export_secret.sh [Key from step 4.]` 