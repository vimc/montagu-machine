# Support machine
support.montagu.dide.ic.ac.uk

Host fingerprint (ECDSA): `SHA256:etgLOXa8brU/0SsfYfoFwsNW3ljbkLPd3okPJlICN6A`

## Services

* [TeamCity](https://github.com/vimc/montagu-ci) 
  (via vagrant, installed as a systemd service as `montagu-ci`)
* [Staging](https://github.com/vimc/montagu-machine/tree/master/staging)
  (via vagrant, installed as a systemd service as `montagu-staging`)
* [Vault](https://github.com/vimc/montagu-vault) (as Docker container, control from `~montagu/vault`)
* [Montagu-Monitor](https://github.com/vimc/montagu-monitor) (as Docker container, control from `~montagu/monitor`)
* [YouTrack-Integration](https://github.com/reside-ic/youtrack-integration) (as Docker container, control from `~montagu/youtrack-integration`)


## Users

* The `vagrant` user is needed to control the virtual machines - `science`, `uat`, `latest`, and the CI system.
* The `montagu` user is used to control services that are dockerised (though these can also be controlled from fresh clones in any user's directory)

The passwords for these users are in the vault at `/secret/support/passwords/<user>` (stored under the `value` key), but these should be unnecessary for almost all actions.

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
All services except the Vault and the YouTrack webhook restart automatically on boot. So you need to:

1. Start Vault: [Instructions](https://github.com/vimc/montagu-vault#restarting-andor-restoring-the-vault)
1. Unseal the Vault: [Instructions](https://github.com/vimc/montagu-vault#unsealing-the-vault)
1. As the montagu user start the YouTrack webhook: `~montagu/youtrack-integration/run --use-vault`

You can check things are working by:

1. TeamCity: This is in the process of being deprecated and all access is via logging in using super user token which you need to retrieve from the startup logs:
   1. ssh onto support as vagrant user and go to `montagu-ci` dir
   1. `vagrant ssh montagu-ci-server` to ssh into VM
   1. Find the auth token `sudo grep -ahnr "Super user authentication token" /opt/Teamcity/logs/teamcity-server.log`
   1. Update the token in the vault `vault write secret/teamcity/su token=<new-token>`
   1. Going to http://teamcity.montagu.dide.ic.ac.uk:8111 and checking 
   that the page loads, you can login as a super user and that 3 agents are connected.
1. Staging: Check that montagu and orderly web are running on the science & uat instances (URL and ports can be read from [montagu settings json](https://github.com/vimc/montagu/tree/master/settings)) by logging in and accessing the reporting portal.
1. Vault: If you were able to unseal it, then it's up
1. Monitor: Go to http://support.montagu.dide.ic.ac.uk:9090/ see [deployment on support docs for details if not up](https://github.com/vimc/montagu-monitor#deployment-on-support)

