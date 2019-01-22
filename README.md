# montagu-machine

What's in here?

- [`docs`](docs): documentation about different machines, how they were built, how to restore them
- [`provision`](provision) scripts to be used when provisioning machines
- [`Vagrantfile`](Vagrantfile) a [vagrant](https://www.vagrantup.com/) configuration for our staging machines

### Tasks

Upgrade vagrant on a machine

```
sudo dpkg --remove vagrant
sudo ./provision/setup-vagrant
```
