# Montagu disaster recovery
Montagu depends on several required external services:

* Amazon S3 (backup location)
* GitHub

And several internally hosted services:

* Vault (secret storage)
* Docker registry (built images, ready for deployment)
* TeamCity (continuous integration, builds images from source code)

See [the architecture diagram](diagrams/VIMC%20Architecture.png) for more 
details.

This guide assumes that both the support and production machines have been
destroyed/corrupted and you need to start again, but that the required external
services are unaffected. If only some internal services are broken, you can
skip some steps.

## What do you need
* Two blank Ubuntu servers (16.04 LTS), with DNS mapping one as 
  `support.montagu.dide.ic.ac.uk`.
* A GitHub account that is a member of the "development" team on the VIMC 
  organization. You need a personal access associated with this account. To 
  generate a personal access token, go to GitHub > Settings > Personal Access 
  Tokens. The new token must have the 'user' scope.

## Support machine 

We have to, at a minimum, get the vault and the Docker registry back up. Getting TeamCity up is required before we can start working again, but is not needed to redeploy production.

### 1. Install minimum requirements

As root:

* `apt-get update`
* `apt install git python3-pip -y`
* [Install Docker Community Edition](https://docs.docker.com/engine/installation/)
* [Install vagrant](https://www.vagrantup.com/downloads.html) (for TeamCity)

See [provision.sh in montagu-recovery](https://github.com/vimc/montagu-recovery/blob/master/provision.sh) for further details.

### 2. Restore backup

As root:

* Get the backup code:
  ```
  mkdir /montagu && cd /montagu
  git clone https://github.com/vimc/montagu-backup backup
  cd backup
  ```
* Configure: `mkdir -p /etc/montagu/backup && cp configs/support.montagu/* /etc/montagu/backup/`
* Install: `./setup.sh` (You will be prompted for your GitHub personal access
  token)
* Restore: `./restore.py`. This is straightforward on a blank machine. If there
  are existing corrupted files you may want to use the `--overwrite=true` flag.
* Less urgent, but worth doing now: `./schedule.py`, to get backups going.

### 3. Restore the Vault

Follow the instructions [here](https://github.com/vimc/montagu-vault/blob/master/README.md#restarting-the-vault).

### 4. Restore the Docker Registry

Follow the instructions [here](https://github.com/vimc/montagu-registry#deployment)

### 5. Restore TeamCity

Backups will have been restored by duplicati and can be restored following instructions [here](https://github.com/vimc/montagu-ci#recovery-from-backup)

## Production machine

Then, as root:
```
cd /
git clone https://github.com/vimc/montagu.git montagu
cd /montagu
```

Then follow [these instructions](https://github.com/vimc/montagu/blob/master/README.md).
