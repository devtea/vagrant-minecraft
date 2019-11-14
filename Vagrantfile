# -*- mode: ruby -*-
# vi: set ft=ruby :
require 'yaml'

###########################
# Load local config files #
###########################
project_path = File.dirname(__FILE__)
# Load the default config supplied with the repository
config = YAML.load_file(File.join(project_path, 'config.yaml'))

# To make managmenet of ip addresses easier in vagrant (because there is no dhcp
# support out of the box), Static ips will be written to a hosts file for salt
# There is another append block inside the server build loop to add individual
# host IPs.
File.open(File.join(project_path, "salt/files/hostsfile"), "w") do |hostsfile|
  hostsfile.puts("# Salt managed hostfile for vagrant")
  hostsfile.puts("127.0.0.1     localhost localhost.localdomain localhost4 localhost4.localdomain4")
  hostsfile.puts("::1           localhost localhost.localdomain localhost6 localhost6.localdomain6")
end


###########################
# Configure vagrant hosts #
###########################
Vagrant.configure("2") do |vagrant_config|
  #
  # Global settings
  #
  vagrant_config.vm.synced_folder ".", "/vagrant", type: "virtualbox"

  ############################################
  #           Server build loop              #
  ############################################
  config["hosts"].each do | host, host_properties_list |
    vagrant_config.vm.define host, primary: true do |guest|
      guest.vm.hostname = host
      guest.vm.box = "archlinux/archlinux"
      if host_properties_list.key?("portmap")
        host_properties_list["portmap"].each do |portmap|
          guest.vm.network "forwarded_port", guest: portmap["guest"], host: portmap["host"]
        end
      end
      # To make managmenet of ip addresses easier in vagrant (because there is no dhcp
      # support out of the box), Static ips will be written to a hosts file for salt
      # There is block for the header of the file above.
      File.open(File.join(project_path, "salt/files/hostsfile"), "a") do |hostsfile|
        hostsfile.puts("#{host_properties_list["ip"]} #{host}")
      end

      guest.vm.network "private_network", ip: host_properties_list["ip"]
      guest.vm.provider "virtualbox" do |v|
        v.cpus = host_properties_list["cpu"]
        v.memory = host_properties_list["ram"]
      end
      guest.vm.provision :salt, :run => 'always' do |salt|
        salt.masterless = true
        salt.install_type = "stable"
        salt.version = config["salt_version"]
        salt.run_highstate = config["run_highstate_on_up"]
        salt.verbose = config["verbose_salt_output"]
        salt.minion_config = "salt/minion"
        salt.grains_config = "salt/grains/#{host}.yaml"
      end
    end
  end
end

