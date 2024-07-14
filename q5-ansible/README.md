## create and encrypt secrets.yml file using ansible vault
```
ansible-vault create vault/secrets.yml
```

## enter credentials to be encrypted
ansible_user: username
ansible_password: password

## running the playbook
```
ansible-playbook -i inventory/hosts.ini playbooks/install_node_exporter.yml --ask-vault-pass
```
enter the password to decrypt
