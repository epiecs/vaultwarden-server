# Vaultwarden setup
A collection of docker containers that helps to setup an internal vaultwarden server with automatic backups.

Makes use of the following containers:
* [vaultwarden](https://github.com/dani-garcia/vaultwarden)
* [nginx](https://github.com/nginxinc/docker-nginx) as https proxy
* [bw backup](https://github.com/Bruceforce/bitwarden_rs-backup) for automated backups


All bitwarden data is saved in `data/vaultwarden` and all backups are saved in `backups`.

## Setup

1. Install docker, docker-compose and git
   
2. Create a user to run the docker containers and pull the git repo
```console
    sudo mkdir /opt/vaultwarden
    sudo useradd vaultwarden
    sudo passwd vaultwarden
    sudo usermod -aG docker vaultwarden
    sudo usermod -d /opt/vaultwarden vaultwarden
    sudo usermod -s /bin/bash vaultwarden
    cd /opt/vaultwarden

    sudo su vaultwarden
    git clone https://github.com/epiecs/vaultwarden-server.git .
```

3. Generate an admin key with the command `openssl rand -base64 48` and place it in `config/vaultwarden/vaultwarden.env`. **You can use the provided key for quick tests but please generate a new key and keep it secure.**
   
4. Check the backup settings in `config/backup/bw_backup.env`. The default settings run a daily backup with a retention of 30 days and appends the timestamp to each backup. Refer to the [documentation](https://github.com/Bruceforce/bitwarden_rs-backup) if you need to edit the values.

## Certificates

Bitwarden requires ssl to function properly. You can choose to generate self signed certificates or to generate a private key and csr and have your internal ca sign them.

If you do not have a private key generate one using 

`openssl genrsa -out config/ssl/private/vaultwarden.key 3072`

### Using a self signed certificate

Use the following command to generate a self signed certificate using your private key. Make sure that you fill in the correct FQDN. The certificate is valid for **3 years**.

`openssl req -x509 -nodes -days 1095 -key config/ssl/private/vaultwarden.key -out config/ssl/certs/vaultwarden.crt`

You can verify the certificate with

`openssl x509 -in config/ssl/certs/vaultwarden.crt -text -noout`

### Creating a CSR

When you want to create a CSR for signing use the following command. 

`openssl req -new -sha256 -key config/ssl/private/vaultwarden.key -out config/ssl/requests/vaultwarden.csr`

You will need to answer a few questions. Make sure that you use the correct FQDN of your server.

The CSR can be found in `config/ssl/requests/vaultwarden.csr`


You can verify the CSR with `openssl req -in config/ssl/requests/vaultwarden.csr -noout -text`

Have the CSR signed by your CA and put the certificate in the following location: `config/ssl/certs/vaultwarden.crt`


## Starting the services

1. `docker-compose pull`
2. `docker-compose up -d`

## Updating
TODO
https://stackoverflow.com/questions/31466428/how-to-restart-a-single-container-with-docker-compose/39501539#39501539

docker compose prune -f?