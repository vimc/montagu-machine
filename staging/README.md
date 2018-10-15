# staging machines

This holds scripts and data for interacting with the staging machines, in addition to the basic provisioning and vagrant bits that are at the root of the repository.

* `scripts/write-ssh-shortcuts`: writes out files `ssh-science` and `ssh-uat` in the home directory for easy access to the staging machines from the host

## Setting up on a new machine

Run all commands from the root of this repository, not from within the `staging` directory.

First, install all dependencies

```
sudo ./provision/setup-vagrant
```

## To set up the staging VMs to run as a systemd service

```
sudo ./provision/staging/setup-start-on-boot
```

The username you should provide is probably `vagrant`

The VMs will then start automatically on boot, and shutdown gracefully when the
host system shuts down.

### Managing the service

* To start `sudo systemctl start montagu-staging.service`
* To stop `sudo systemctl stop montagu-staging.service`
* To uninstall the service (and disable auto start on boot):
  `sudo ./scripts/staging/remove-start-on-boot`

### Service logging

```
systemctl status montagu-staging         # short status, run as ordinary user
sudo journalctl --unit montagu-staging   # full log, needs root
```
