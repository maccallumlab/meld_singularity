resize = <<-SCRIPT
sudo parted /dev/sda resizepart 1 100%
sudo pvresize /dev/sda1
sudo lvextend -l +100%FREE /dev/vagrant-vg/root
sudo resize2fs /dev/vagrant-vg/root
SCRIPT

Vagrant.configure("2") do |config|
  config.vm.define "singularity" do |h|
    h.vm.box = "sylabs/singularity-3.7-ubuntu-bionic64"
    h.vm.provider :virtualbox
  end
  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 2
  end
  config.disksize.size = "100GB"
  config.vm.provision :shell, :inline=>resize
end