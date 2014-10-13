# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.provider "docker" do |d|
    d.image = "grahamc/jekyll"
    d.ports=["4000:4000"]
    d.cmd = ["serve"] 
  end

  config.vm.synced_folder ".", "/src"

end
