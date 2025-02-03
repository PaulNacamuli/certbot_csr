# Created by Paul Nacamuli
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  
    $scriptlinux = <<-SCRIPT
    echo I am provisioning...
    sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
    echo 'vagrant ALL=(ALL) NOPASSWD:ALL' >>/etc/sudoers
    sudo -u vagrant ssh-keygen -b 2048 -t rsa -f /home/vagrant/.ssh/id_rsa -q -N ""
  SCRIPT

    # DHCP server
  config.vm.define "dhcp" do |dhcp |
    dhcp.vm.provider "hyperv" do |v|
      v.vmname = "dhcp"
      v.memory = 512
      v.cpus = 1
      v.enable_virtualization_extensions = true
      v.linked_clone = true
    end
    dhcp.vm.network "public_network",bridge: "NATswitch"
    dhcp.vm.box = "gusztavvargadr/ubuntu-server"
    dhcp.vm.hostname = "dhcp.test"
    dhcp.vm.network :private_network, ip: "10.21.32.11"
    dhcp.vm.provision "shell", inline: $scriptlinux
  end

    # certbot server
    config.vm.define "certbot" do |certbot |
      certbot.vm.provider "hyperv" do |v|
        v.vmname = "certbot"
        v.memory = 512
        v.cpus = 1
        v.enable_virtualization_extensions = true
        v.linked_clone = true
      end
      certbot.vm.network "public_network",bridge: "NATswitch"
      certbot.vm.box = "gusztavvargadr/ubuntu-server"
      certbot.vm.hostname = "certbot.test"
      certbot.vm.synced_folder "C:/git/_linux", "/pc", type: "smb", smb_username: "serviceuser", smb_password: "servicepw"
      certbot.vm.provision "shell", inline: $scriptlinux
    end
end
