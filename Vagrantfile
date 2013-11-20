# -*- mode: ruby -*-
# vi: set ft=ruby :

options = {
  :nodes => 2,
}

CONF_FILE = Pathname.new("vagrant-overrides.conf")

override_options = CONF_FILE.exist? ? Hash[*File.read(CONF_FILE).split(/[= \n]+/)] : {}
override_options.each { |key, value| options[key.to_sym] = value unless key.start_with?("#") }

Vagrant.configure("2") do |cluster|
  # Ensure latest version of Chef is installed.
  cluster.omnibus.chef_version = :latest

  # Utilize the Berkshelf plugin to resolve cookbook dependencies.
  cluster.berkshelf.enabled = true

  (1..options[:nodes].to_i).each do |index|
    cluster.vm.define "riak#{index}".to_sym do |config|
      # Configure the VM and operating system.
      config.vm.box = 'centos'
      config.vm.provider :aws do |aws,override|
        aws.access_key_id = "XXXXXXXXXXXXXXXXXXXXXXX"
        aws.secret_access_key = "YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY"
        aws.instance_type = "m1.medium"
        # Optional
        aws.keypair_name = "riak"
        aws.ami = "emi-XXYXYXYX"
        override.ssh.username ="root"
        override.ssh.private_key_path ="/path/to/my/riak-priv-key.pem"
        # Optional
        aws.security_groups = ["default"]
        aws.region = "eucalyptus"
        aws.endpoint = "http://<my-clc-ip>:8773/services/Eucalyptus"
        aws.tags = {
                Name: "riak-cs-#{index}",
        }
      end

      # Provision using Chef.
      config.vm.provision :chef_solo do |chef|
        chef.roles_path = "roles"

        if config.vm.box =~ /ubuntu/
          chef.add_recipe "apt"
        else
          chef.add_recipe "yum"
          chef.add_recipe "yum::epel"
        end

        chef.add_role "base"
        chef.add_role "riak"
        chef.add_role "riak_cs"
        chef.add_role "stanchion" if index == 1

        if !ENV["RIAK_CS_CREATE_ADMIN_USER"].nil? && index == 1
          chef.add_recipe "riak-cs::control"
          chef.add_recipe "riak-cs-create-admin-user"
        end

        chef.json = {
          "riak" => {
            "args" => {
              "+S" => 1,
            },
          },
          "riak_cs" => {
            "args" => {
              "+S" => 1,
            },
            "config" => {
              "riak_cs" => {
                "anonymous_user_creation" => ((ENV["RIAK_CS_CREATE_ADMIN_USER"].nil? || index != 1) ? false : true),
              }
            }
          },
          "stanchion" => {
            "args" => {
              "+S" => 1,
            }
          },
          "riak_cs_control" => {
            "args" => {
              "+S" => 1,
            }
          }
        }
      end
    end
  end
end
