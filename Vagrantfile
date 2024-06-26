NETWORK = "192.168.1.0"
GW = "192.168.1.1"
PREF = "24"
DNS = "1.1.1.1"

# Specify minimum Vagrant version and Vagrant API version
Vagrant.require_version ">= 1.6.0"
VAGRANTFILE_API_VERSION = "2"

# Require YAML module
require 'yaml'

# Read YAML file with box details
servers = YAML.load_file('servers.yaml')

# Create boxes
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.provider "hyperv" do |h, override|
    h.enable_virtualization_extensions = false
    h.linked_clone = true
    override.vm.synced_folder ".", "/vagrant", disabled: true
  end
  # Iterate through entries in YAML file
  servers.each do |servers|
    config.vm.define servers["name"] do |srv|
      srv.vm.box = servers["box"]
      vhostname = servers["name"]
      # First connect to Default Switch network
      ##vm_exists = system("powershell.exe", "-Command", "Get-VM -VMNAME vhostname -ErrorAction SilentlyContinue | out-null")
      ##if !vm_exists
      ##  #srv.vm.network "private_network", bridge: "Default Switch"
	  ##	srv.vm.network "public_network", bridge: "Lan"
      ##end
	  # srv.vm.network "private_network", ip: value['ip']
      #if value['ip'] !=''
       srv.vm.hostname = servers["name"]
	 ip = system("powershell.exe", "-Command", "Get-VM -VMNAME vhostname  -ErrorAction SilentlyContinue | Get-VMNetworkAdapter| select IPAddresses -ErrorAction SilentlyContinue")
	 if if 'ip' !=''
        srv.vm.provision "shell", inline: "echo NO IP ADDRESS"
        srv.vm.network :public_network, bridge:'Lan'
      else        
        srv.vm.network :public_network, ip:value['ip'] ,bridge:'Lan'
        srv.vm.provision "shell", inline: "echo IP FOUND FOR"
      end
	  end
      srv.vm.provider "hyperv" do |hv|
        hv.memory = servers["ram"]
        hv.cpus = servers["cpu"]
        hv.vmname = servers["name"]
      end
      # Configure static ip script
      srv.vm.provision "shell" do |sh|
        sh.path = "./.scripts/configure-static-ip.sh"
		sh.args = [servers["ip"], GW, PREF, DNS]
		
		#srv.vm.provision "shell", inline: "sudo hostnamectl set-hostname vhostname"
      end
      # Install apps script
         srv.vm.provision :shell, path: servers["script"] 
	     srv.vm.provision :reload	
	     srv.vm.provision :shell, path: "./.scripts/postscript.sh"
	  end
end
 end