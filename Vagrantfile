# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

# The number of nodes except the master
# (# total deployed VMs = NUM_NODES + 1)
NUM_NODES = 2

$repository_init_script = <<SCRIPT
yum install -y wget expect
cd /etc/yum.repos.d/
wget http://public-repo-1.hortonworks.com/ambari/centos6/1.x/updates/1.4.4.23/ambari.repo
yum install -y ambari-agent
cd /vagrant
python ambari_agent_init.py
ambari-agent start
SCRIPT

$hosts_init_script = <<SCRIPT
while [ "$1" != "" ]; do
  hostname=$1
  shift
  ipaddr=$1
  shift
  echo "$ipaddr $hostname" >> /etc/hosts
done
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  host_ip_list = ["master", "192.168.33.100"]
  (1..NUM_NODES).each do |i|
    host_ip_list.push("node#{i}")
    host_ip_list.push("192.168.33.#{i+100}")
  end
  FileUtils.cp(File.expand_path("~/.vagrant.d/insecure_private_key"), "insecure-host-key")

  # Every Vagrant virtual environment requires a box to build off of.
  config.vm.define :master, primary: true do |master_conf|
    vmname = "master"
    master_conf.vm.host_name = vmname
    master_conf.vm.box = "centos6.5-x86_64"
    master_conf.vm.network "private_network", ip: "192.168.33.100"
    master_conf.vm.network "forwarded_port", guest: 8080, host: 8080
    master_conf.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "4096"]
      vb.customize ["modifyvm", :id, "--name", vmname]
    end
    master_conf.vm.provision "shell", inline: "mkdir -p /root/.ssh; cp /vagrant/insecure-host-key /root/.ssh/id_rsa; chmod 600 /root/.ssh/id_rsa"
    master_conf.vm.provision "shell", inline: $repository_init_script
    master_conf.vm.provision "shell", inline: $hosts_init_script, args: host_ip_list
    # Automatic server setup!
    master_conf.vm.provision "shell", inline: "yum install -y ambari-server"
    master_conf.vm.provision "shell", inline: "expect /vagrant/ambari_server_init.expect"
  end

  (1..NUM_NODES).each do |i|
    vmname = "node#{i}"
    config.vm.define vmname.to_sym do |node_conf|
      node_conf.vm.host_name = vmname
      node_conf.vm.box = "centos6.5-x86_64"
      node_conf.vm.network "private_network", ip: "192.168.33.#{100+i}"
      node_conf.vm.provider :virtualbox do |vb|
        vb.customize ["modifyvm", :id, "--memory", "4096"]
        vb.customize ["modifyvm", :id, "--name", vmname]
      end
      node_conf.vm.provision "shell", inline: $repository_init_script
      node_conf.vm.provision "shell", inline: $hosts_init_script, args: host_ip_list
    end
  end

  # The url from where the 'config.vm.box' box will be fetched if it
  # doesn't already exist on the user's system.
  # config.vm.box_url = "http://domain.com/path/to/above.box"

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network :forwarded_port, guest: 80, host: 8080

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network :public_network

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  # config.ssh.forward_agent = true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider :virtualbox do |vb|
  #   # Don't boot with headless mode
  #   vb.gui = true
  #
  #   # Use VBoxManage to customize the VM. For example to change memory:
  #   vb.customize ["modifyvm", :id, "--memory", "1024"]
  # end
  #
  # View the documentation for the provider you're using for more
  # information on available options.

  # Enable provisioning with Puppet stand alone.  Puppet manifests
  # are contained in a directory path relative to this Vagrantfile.
  # You will need to create the manifests directory and a manifest in
  # the file base.pp in the manifests_path directory.
  #
  # An example Puppet manifest to provision the message of the day:
  #
  # # group { "puppet":
  # #   ensure => "present",
  # # }
  # #
  # # File { owner => 0, group => 0, mode => 0644 }
  # #
  # # file { '/etc/motd':
  # #   content => "Welcome to your Vagrant-built virtual machine!
  # #               Managed by Puppet.\n"
  # # }
  #
  # config.vm.provision :puppet do |puppet|
  #   puppet.manifests_path = "manifests"
  #   puppet.manifest_file  = "site.pp"
  # end

  # Enable provisioning with chef solo, specifying a cookbooks path, roles
  # path, and data_bags path (all relative to this Vagrantfile), and adding
  # some recipes and/or roles.
  #
  # config.vm.provision :chef_solo do |chef|
  #   chef.cookbooks_path = "../my-recipes/cookbooks"
  #   chef.roles_path = "../my-recipes/roles"
  #   chef.data_bags_path = "../my-recipes/data_bags"
  #   chef.add_recipe "mysql"
  #   chef.add_role "web"
  #
  #   # You may also specify custom JSON attributes:
  #   chef.json = { :mysql_password => "foo" }
  # end

  # Enable provisioning with chef server, specifying the chef server URL,
  # and the path to the validation key (relative to this Vagrantfile).
  #
  # The Opscode Platform uses HTTPS. Substitute your organization for
  # ORGNAME in the URL and validation key.
  #
  # If you have your own Chef Server, use the appropriate URL, which may be
  # HTTP instead of HTTPS depending on your configuration. Also change the
  # validation key to validation.pem.
  #
  # config.vm.provision :chef_client do |chef|
  #   chef.chef_server_url = "https://api.opscode.com/organizations/ORGNAME"
  #   chef.validation_key_path = "ORGNAME-validator.pem"
  # end
  #
  # If you're using the Opscode platform, your validator client is
  # ORGNAME-validator, replacing ORGNAME with your organization name.
  #
  # If you have your own Chef Server, the default validation client name is
  # chef-validator, unless you changed the configuration.
  #
  #   chef.validation_client_name = "ORGNAME-validator"
end
