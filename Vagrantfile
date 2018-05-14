# -*- mode: ruby -*-
# vi: set ft=ruby :
required_plugins = %w( vagrant-proxyconf vagrant-vbguest vagrant-reload )
required_plugins.each do |plugin|
    system "vagrant plugin install #{plugin}" unless Vagrant.has_plugin? plugin
end
Vagrant.configure("2") do |config|
    config.vm.box = "cxtlabs/vagrant-ubuntu-16.04-mate"

    config.vm.box_check_update = true
    config.vbguest.auto_update = true

    home = "/home/vagrant/"

    config.vm.synced_folder File.expand_path("."), "/vagrant", disabled: true
    config.vm.synced_folder "/development", "#{home}development"

    # share git config
    config.vm.provision "file", source: File.expand_path("~") + "/.gitconfig", destination: "#{home}.gitconfig"

    # Expose the running application to the host
    config.vm.network "forwarded_port", guest: 8080, host: 8080
    config.vm.network "forwarded_port", guest: 3500, host: 3500

    config.vm.provision "shell", inline: <<-SHELL
        # change keyboard configuration to 'US'
        sudo sed -i -e 's/XKBLAYOUT=.*/XKBLAYOUT="us"/' /etc/default/keyboard
        # set the timezone
        sudo timedatectl set-timezone Australia/Perth
        # install some software
        sudo apt-get update
        sudo apt-get install htop libnss3 chromium-browser -y
    SHELL

    config.vm.provider "virtualbox" do |vb|
        vb.gui = true
        vb.customize ["modifyvm", :id, "--monitorcount", "1"]
        vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
        vb.customize ["modifyvm", :id, "--draganddrop", "bidirectional"]
        vb.memory = 4096
        vb.cpus = 4
    end

    config.vm.provision :reload
end
