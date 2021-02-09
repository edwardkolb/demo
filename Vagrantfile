
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
# Config VM Hostname
  config.vm.hostname = "ci-server"
  config.vm.box_check_update = false
# Config public IP address to establish connection with Jenkins web-interface
  config.vm.network "public_network", ip: "192.168.100.100"
# Config timezone
  if Vagrant.has_plugin?("vagrant-timezone")
    config.timezone.value = "Europe/Minsk"
  config.vm.provider "virtualbox" do |vb|
# By default Vagrant provides 512Mb of RAM, so I changed it to 1024Mb
     vb.memory = "1024"
# Vagrant VM's automatic naming is some kind of crazy, so I decided to change it as well
     vb.name = "Sudoku"
     end
  end

# Config VM provisioning by Ansible
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
  end
 
end

