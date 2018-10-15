# Support machine
support.montagu.dide.ic.ac.uk

Host fingerprint (ECDSA): `SHA256:etgLOXa8brU/0SsfYfoFwsNW3ljbkLPd3okPJlICN6A`

## Services
* [TeamCity](https://github.com/vimc/montagu-ci) 
  (via vagrant, installed as a systemd service as `montagu-ci`)
* [Staging](https://github.com/vimc/montagu/tree/master/staging)
  (via vagrant, installed as a systemd service as `montagu-staging`)
* [Docker registry](https://github.com/vimc/montagu-registry) 
  (as Docker container)
* [Vault](https://github.com/vimc/montagu-vault) (as Docker container)
* [Montagu-Monitor](https://github.com/vimc/montagu-monitor)

## Upgrading and rebooting
### Graceful shutdown
Everything will shutdown gracefully on host system shutdown. The Vagrant VMs
are installed as a service and Docker will automatically stop containers on 
shutdown.

### Upgrade
```
sudo apt-get update
sudo apt-get upgrade
sudo reboot now   # if needed
```

### Resume
All services exept the Vault restart automatically on boot. So you need to:

1. Start Vault: [Instructions](https://github.com/vimc/montagu-vault#restarting-andor-restoring-the-vault)
1. Unseal the Vault: [Instructions](https://github.com/vimc/montagu-vault#unsealing-the-vault)

You can check things are working by:

1. TeamCity: Going to http://teamcity.montagu.dide.ic.ac.uk:8111 and checking 
   that the page loads and that 3 agents are connected.
1. Registry: Running `docker pull docker.montagu.dide.ic.ac.uk:5000/montagu-db:master`
1. Staging: Browsing to the instances: [URLs](https://github.com/vimc/montagu/blob/master/staging/README.md#access-the-stage-instances)
1. Vault: If you were able to unseal it, then it's up
1. Monitor: Go to http://support.montagu.dide.ic.ac.uk:9090/
