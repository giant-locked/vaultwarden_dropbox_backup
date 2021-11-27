# Vaultwarden Dropbox Nightly Backup

## About
Run this image alongside your Vaultwarden container for automated nightly (1AM UTC) encrypted backups of your Vaultwarden database to your Dropbox account.

**IMPORTANT: Make sure you have at least one personal device (e.g. laptop) connected to Dropbox and syncing files locally. This will save you in the event Vaultwarden goes down and your Dropbox account credentials are stored in Vaultwarden!!!**

## Setup

### Clone this repo
1. Clone the repo into the Vaultwarden work directory: `git clone git@github.com:giant-locked/vaultwarden_dropbox_backup.git`
2. Cd into repo directory: `cd vaultwarden_dropbox_backup`

### Create Dropbox app
1. Open the following URL in your Browser, and log in using your account: https://www.dropbox.com/developers/apps
2. Click on "Create App", then select "Choose an API: Scoped Access"
3. Choose the type of access you need: "App folder"
4. Enter the "App Name" that you prefer (e.g. MyVaultBackups); must be unique
5. Now, click on the "Create App" button.
6. Now the new configuration is opened, switch to tab "permissions" and check "files.metadata.read/write" and "files.content.read/write"
7. Now, click on the "Submit" button.
8. Once your app is created, you can find your "App key" and "App secret" in the "Settings" tab.

### Build Docker image and run docker-compose
1. From the directory with Dockerfile execute: `docker build -t vw-bcp .`
2. Run the container in interactive mode (`docker run -it vw-bcp`) to create the configuration and follow the steps in the terminal. 
3. Press `Ctrl+P` followed by `Ctrl+Q` to exit interactive mode / detach and keep the container running.
4. From the directory with docker-compose.yml execute: `docker-compose up -d`
#### Note 1: Steps 2, 3 are only needed for the first run to populate the config file with secrets. If you re-create the container with the same config file mount, the container will not need to be run in interactive mode
#### Note 2: If you are using a GUI like Portainer to create the container, you will need to attach to the container. The first input to provide once attached is the App key).

### Check everything is up and running
1. Execute `docker ps -a` to check docker containers
2. Checck your Dropbox app's directory. By default backup will be created upon docker-compose first launch. Then - every night at 1 AM UTC.

## Encryption

- Backups are encrypted (OpenSSL AES256) and zipped (`.tar.gz`) with a passphrase of your choice.
- Encrypting Vaultwarden backups is not required since the data is already encrypted with user master passwords. We've added this for good practice and added obfuscation should your Dropbox account get compromised.
- Pick a secure `BACKUP_ENCRYPTION_KEY`. This is for added protection and will be needed when decrypting your backups.


### Decrypting Backup
`openssl enc -d -aes256 -salt -pbkdf2 -in mybackup.tar.gz | tar xz --strip-components=1 -C my-folder`

### Restoring Backup to BitWarden_RS
Volume mount the decrypted `./bwdata` folder to your bitwarden_rs container. Done!

## Other notes
- Volume mount the `../vw-data` folder your Vaultwarden container uses.
- This image will always run an extra backup on container start (regardless of cron interval) to ensure your setup is working.
- Supports an optional `DELETE_AFTER` environment variable which is used to delete old backups after X many days. This job is executed with each backup cron job run.
