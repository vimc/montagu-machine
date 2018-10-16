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

## Troubleshooting

### Bringing up vms

If you get a error message like this on bringing up a VM

```
vagrant@fi--didevimc02:~/staging/staging$ vagrant up uat
Bringing machine 'uat' up with 'virtualbox' provider...
==> uat: Checking if box 'bento/ubuntu-16.04' is up to date...
==> uat: A newer version of the box 'bento/ubuntu-16.04' is available! You currently
==> uat: have version '2.3.5'. The latest is version '201708.22.0'. Run
==> uat: `vagrant box update` to update.
==> uat: Fixed port collision for 22 => 2222. Now on port 2203.
==> uat: Clearing any previously set network interfaces...
There was an error while executing `VBoxManage`, a CLI used by Vagrant
for controlling VirtualBox. The command and stderr is shown below.

Command: ["hostonlyif", "create"]

Stderr: 0%...
Progress state: NS_ERROR_FAILURE
VBoxManage: error: Failed to create the host-only adapter
VBoxManage: error: VBoxNetAdpCtl: Error while adding new interface: failed to open /dev/vboxnetctl: No such file or directory
VBoxManage: error: Details: code NS_ERROR_FAILURE (0x80004005), component HostNetworkInterfaceWrap, interface IHostNetworkInterface
VBoxManage: error: Context: "RTEXITCODE handleCreate(HandlerArg*)" at line 94 of file VBoxManageHostonly.cpp
```

The solution is to load the virtualbox module

```
sudo modprobe vboxnetadp
```
