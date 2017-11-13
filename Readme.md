# Start configuration up
docker-compose up -d

# Prepare vault for CLI
alias vault='docker exec -it --privileged vault.server vault "$@"'
export VAULT_ADDR=http://127.0.0.1:8200

# Init vault
vault init -address=${VAULT_ADDR} > keys.txt

# Vault unseal (using 3 keys)
vault unseal -address=${VAULT_ADDR} $(grep 'Key 1:' keys.txt | awk '{print $NF}')
vault unseal -address=${VAULT_ADDR} $(grep 'Key 2:' keys.txt | awk '{print $NF}')
vault unseal -address=${VAULT_ADDR} $(grep 'Key 3:' keys.txt | awk '{print $NF}')

# Check vault status
vault status -address=${VAULT_ADDR}

# Login (as root)
vault auth -address=${VAULT_ADDR} $(grep 'Token:' keys.txt | awk '{print $NF}')

# Mount ssh & create role
vault mount -address=${VAULT_ADDR} ssh
vault write -address=${VAULT_ADDR} ssh/roles/ssh_otp key_type=otp default_user=itdays cidr_list=0.0.0.0/0

# Create one OTP
vault write -address=${VAULT_ADDR} ssh/creds/ssh_otp ip=192.168.1.1


# Credits 
- https://www.katacoda.com/courses/docker-production/vault-secrets
- http://pcarion.com/2017/04/30/A-consul-a-vault-and-a-docker-walk-into-a-bar..html