# -*- mode: ruby -*-
# vi: set ft=ruby :

current_dir = File.dirname(__FILE__)

# Import configs from YAML file.
yml = YAML.load_file "#{current_dir}/config/config.yml"

# Use 1) ENV variable, 2) YAML config file, then 3) Default
box     = ENV['box']     ||= yml['box']     ||= "lucid64"
project = ENV['project'] ||= yml['project']

# Write project (or lack thereof) to YAML config file
yml['project'] = project
File.open("#{current_dir}/config/config.yml", 'w') { |f| YAML.dump(yml, f) }

Vagrant::Config.run do |config|
  config.vm.define "ariadne"

  # Mash of box names and urls
  baseboxes = {
    "hardy64" => "https://myplanet.box.com/shared/static/pkmp7lus3ojvd4vlusuj.box",
    "lucid64" => "http://files.vagrantup.com/lucid64.box"
  }

  config.vm.box = box
  config.vm.box_url = baseboxes[box]

  config.vm.network :hostonly, "33.33.33.10"

  # Forward keys and ssh configs into VM
  # Caveats: https://github.com/mitchellh/vagrant/issues/105
  config.ssh.forward_agent = true

  # Use vagrant-dns plugin to configure DNS server
  config.dns.tld = "dev"
  config.dns.patterns = [ /^.*#{project}.dev$/ ]

  # Share directories to speed up boot and general site code directory
  config.vm.share_folder "apt-cache", "/var/cache/apt/archives", "#{current_dir}/tmp/apt/cache", :nfs => true
  config.vm.share_folder "gem-cache", "/opt/ruby/lib/ruby/gems/1.8/cache", "#{current_dir}/tmp/ruby/1.9.1/cache", :nfs => true
  config.vm.share_folder "html", "/mnt/www/html", "#{current_dir}/data/html", :nfs => true

  config.vm.forward_port 80, 8080
  config.vm.forward_port 3306, 3306

  # Update Chef in VM to specific version before running provisioner
  config.vm.provision :shell do |shell|
    shell.path = "config/upgrade_chef.sh"
    shell.args = "0.10.8" # Chef version
  end

  config.vm.provision :chef_solo do |chef|
    chef.roles_path = "roles"

    chef.cookbooks_path = [ "cookbooks", "cookbooks-override", "cookbooks-projects" ]

    # Set up basic environment
    chef.add_role "ariadne"

    # Add recipe for example site if no project set.
    project_recipe = project.empty? ? "ariadne::example" : project
    chef.add_recipe project_recipe

    chef.json = {
      "mysql" => {
        "server_debian_password" => "root",
        "server_root_password"   => "root",
        "server_repl_password"   => "root",
        "allow_remote_root" => true,
        "bind_address" => "0.0.0.0",
      },
      "ariadne" => {
        "project" => project
      }
    }
  end
end
