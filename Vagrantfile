# Need a recent version for the 'reset' functionality which fixes
# docker permissions for us
Vagrant.require_version ">= 2.2.3"

domain = 'localdomain'
box = "bento/ubuntu-16.04"
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
    :metricsport => 9113,
    :autostart => true,
    :ram_size_mb => '16384'
  },
  {
    :hostname => 'science',
    :ip => '192.168.81.12',
    :port => 11443,
    :dbport => 5432,
    :metricsport => 9114,
    :autostart => true,
    :ram_size_mb => '32768'
  },
  {
    :hostname => 'latest',
    :ip => '192.168.81.14',
    :port => 12443,
    :dbport => 35432,
    :metricsport => 9115,
    :autostart => true,
    :deploymontagu => 'latest',
    :ram_size_mb => '16384'
  },
  {
    :hostname => 'dev',
    :ip => '192.168.81.15',
    :port => 13443,
    :dbport => 45432,
    :metricsport => 9116,
    :autostart => false,
    :deploymontagu => 'branch',
    :ram_size_mb => '16384'
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
    shell.reset = true
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
        vbox.memory = machine[:ram_size_mb]
      end
      machine_config.persistent_storage.location =
        "staging/disk/#{machine[:hostname]}.vdi"
      machine_config.vm.network :private_network, ip: machine[:ip]
      machine_config.vm.network "forwarded_port", guest: machine[:port],
                                host: machine[:port]
      machine_config.vm.network "forwarded_port", guest: 5432,
                                host: machine[:dbport]
      machine_config.vm.network "forwarded_port", guest: 9113,
                                host: machine[:metricsport]
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

      # Should machine provisioning include deploying montagu, either latest or named branch?
      if machine.key?(:deploymontagu) and ['latest','branch'].include? machine[:deploymontagu]
        # Do not continue with deploy if the calling script has set PREVENT_DEPLOY
        if ENV['PREVENT_DEPLOY'] != 'true'
          machine_config.vm.provision :shell do |shell|
            if machine[:deploymontagu] == 'latest'
              shell.args = 'latest'
            end

            if machine[:deploymontagu] == 'branch'
              # Expect this vagrant environment variable to be set to desired branch on host machine
              # (or allow default to latest branch)
              shell.args = [ENV["REF"] || 'latest']
            end

            shell.path = 'staging/scripts/deploy-montagu-branch'
            shell.env = vault_config
            shell.privileged = false
          end
        else
          machine_config.vm.post_up_message = "Montagu has not been installed on the VM because --prevent-deploy was set. " +
              "Any branch parameter has been ignored. Remember to checkout the desired branch before you install Montagu."
        end
      end
    end
  end
end
