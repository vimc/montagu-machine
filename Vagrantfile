# Older versions of vagrant can't start the ubuntu bentobox with
# private networking
Vagrant.require_version ">= 1.8.2"

domain = 'localdomain'
box = "bento/ubuntu-16.04"
ram_size_mb = '16384'
data_disk_size_gb = 900

permanent = [
  {
    :hostname => 'uat',
    :ip => '192.168.81.11',
    :port => 10443,
    :dbport => 15432,
    :autostart => true
  },
  {
    :hostname => 'science',
    :ip => '192.168.81.12',
    :port => 11443,
    :dbport => 5432,
    :autostart => true
  },
  {
    :hostname => 'restore-test',
    :ip => '192.168.81.13',
    :port => 20443,
    :dbport => 25432,
    :autostart => false
  }
]

Vagrant.configure(2) do |config|
  config.vm.box = box
  config.vm.synced_folder 'staging/shared', '/vagrant'

  # Configure a second disk
  config.persistent_storage.enabled = true
  config.persistent_storage.mountname = "data"
  config.persistent_storage.filesystem = "ext4"
  config.persistent_storage.mountpoint = "/mnt/data"
  config.persistent_storage.size = data_disk_size_gb * 1024

  config.vm.provision :shell do |shell|
    shell.path = 'provision/setup-docker'
    shell.args = ['/mnt/data', 'vagrant']
    shell.env = {"DOCKER_LARGE_DISK" => "/mnt/data"}
  end
  config.vm.provision :shell do |shell|
    shell.path = 'provision/setup-pip'
  end
  config.vm.provision :shell do |shell|
    shell.path = 'provision/setup-vault'
  end
  config.vm.provision :shell do |shell|
    shell.path = 'provision/setup-machine-metrics'
  end

  permanent.each do |machine|
    config.vm.define machine[:hostname], autostart: machine[:autostart] do |machine_config|
      machine_config.vm.provider :virtualbox do | vbox |
        vbox.gui = false
        vbox.memory = ram_size_mb
      end
      machine_config.persistent_storage.location =
        "staging/disk/#{machine[:hostname]}.vdi"
      machine_config.vm.network :private_network, ip: machine[:ip]
      machine_config.vm.network "forwarded_port", guest: machine[:port],
                                host: machine[:port]
      machine_config.vm.network "forwarded_port", guest: 5432,
                                host: machine[:dbport]
      machine_config.vm.provision :shell do |shell|
        shell.path = 'provision/setup-hostname'
        shell.args = machine[:hostname]
      end
    end
  end
end