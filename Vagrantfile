$script = <<-SCRIPT
echo "Waiting for apt to finish"
sudo systemd-run --property="After=apt-daily.service apt-daily-upgrade.service" --wait /bin/true

echo "Updating components"
apt-get install -y build-essential
apt-get install -y gcc

echo "Downloading CUDA"
wget -q https://developer.download.nvidia.com/compute/cuda/11.5.0/local_installers/cuda_11.5.0_495.29.05_linux.run -O cuda.run

echo "Installing CUDA"
sudo sh cuda.run --silent --toolkit
grep cuda /etc/profile || cat >> /etc/profile <<EOF
export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_HOME=/usr/local/cuda
EOF
source /etc/profile

echo "Done"
SCRIPT


Vagrant.configure("2") do |config|
  config.vm.box = "sylabs/singularity-3.7-ubuntu-bionic64"
  config.vm.disk :disk, size: "100GB", primary: true
  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 4
  end

end