# Older versions of vagrant can't start the ubuntu bentobox with
# private networking
Vagrant.require_version ">= 1.8.2"

domain = 'localdomain'
box = "bento/ubuntu-16.04"
ram_size_mb = '16384'
data_disk_size_gb = 900

vault_config_file = 'staging/shared/vault_config'

if !File.file?(vault_config_file)
  raise "Run ./staging/scripts/vault-prepare first!"
end

vault_config = Hash[File.read(vault_config_file).split("\n").
                     map{|s| s.split('=')}]

permanent = [
  {
    :hostname => 'uat',
    :ip => '192.168.81.11',
    :port => 10443,
    :dbport => 15432,
    :autostart => true,
    :deploylatest => false
  },
  {
    :hostname => 'science',
    :ip => '192.168.81.12',
    :port => 11443,
    :dbport => 5432,
    :autostart => true,
    :deploylatest => false
  },
  {
    :hostname => 'latest',
    :ip => '192.168.81.14',
    :port => 12443,
    :dbport => 35432,
    :metricsport => 9115,
    :autostart => true,
    :deploylatest => true
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
    shell.args = ['vagrant']
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

      # This one depends on the hostname being set so it needs to move
      # here
      machine_config.vm.provision :shell do |shell|
        shell.path = 'provision/setup-montagu'
        shell.env = vault_config
        shell.privileged = false
      end

      if machine[:deploylatest]
         machine_config.vm.provision :shell do |shell|
           shell.path = 'provision/deploy-latest-montagu'
           shell.env = vault_config
           shell.privileged = false
         end
      end
    end
  end
end
