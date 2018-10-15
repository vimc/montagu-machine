# Annex machine
annex.montagu.dide.ic.ac.uk

Host key fingerprint (ECDSA): `SHA256:lYJUME+1ATyTvzcFvs8K+aSOBfX/Xv3mpz3kKeYdzM4`

## Services
* Annex database (in docker container)
* Barman
* Cron job to nightly dump data from barman and ferry it to the starport
* Cron job to nightly back up starport to AWS using duplicati
* (Starport - this is just a folder than can be SSH'd to by bb8 from other machines)

## Upgrading and rebooting
```
sudo apt update
sudo apt upgrade
sudo reboot now

sudo su montagu
docker start barman-montagu
docker exec barman-montagu setup-barman --no-initial-backup

```
Annex database will resume automatically. Cron job will continue running as per
usual. Our goal is to have Barman resuming automatically also, but as yet this does not work. See https://vimc.myjetbrains.com/youtrack/issue/VIMC-2116

We couldn't upgrade because VirtualBox was running. If so, you may find this
untested script that should hibernate all VirtualBox VMs that are running. You
could replace "savestate" with "poweroff" if you're sure none of the running VMs
are important.

```
vboxmanage list runningvms \
    | sed -r 's/.*\{(.*)\}/\1/' \
    | xargs -L1 -I {} VBoxManage controlvm {} savestate
```
### Check things are back up and running
* Annex database: `docker exec -it db_annex psql -U vimc -d montagu`
* Barman (as user `montagu`): `barman-montagu status`