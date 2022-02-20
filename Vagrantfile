# -*- mode: ruby -*-
#
#
#
# The script requires:
# ruby
# vagrant
# zip/unzip
#

SUBNET="192.168.33"
DOMAIN="vm.local"
SERVERNAME="ardi-linux"
SERVER_IP="#{SUBNET}.10"

$bootstrapScript = <<-SCRIPT
 yum install -y yum-utils
 yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
 yum install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
 yum install -y docker-ce docker-ce-cli
 yum install -y git vim net-tools
 yum install -y git-core zlib zlib-devel gcc-c++ patch readline readline-devel libyaml-devel libffi-devel openssl-devel make bzip2 autoconf automake libtool bison curl sqlite-devel

 # Install Docker compose
 curl -L "https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
 chmod +x /usr/local/bin/docker-compose

 usermod -aG docker vagrant
 systemctl enable docker
 systemctl start docker
SCRIPT

$installNginxScript = <<-SCRIPT
  yum install -y epel-release
  sudo yum install -y nginx
  systemctl enable nginx
  systemctl start nginx
  systemctl status nginx
SCRIPT

$dockerLoginScript = <<-SCRIPT
  newgrp -
  export DOCKER_USER=#{ENV['DOCKER_USER']}
  export DOCKER_PASSWORD=#{ENV['DOCKER_PASSWORD']}
  docker login -u "$DOCKER_USER" -p "$DOCKER_PASSWORD"
SCRIPT

$installRbenvScript = <<-SCRIPT
  git clone https://github.com/rbenv/rbenv.git ~/.rbenv
  git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
  echo 'export PATH="\$HOME/.rbenv/bin:\$PATH"' >> ~/.bash_profile
  echo 'eval "\$(rbenv init -)"' >> ~/.bash_profile
SCRIPT

$installPythonScript = <<-SCRIPT
  yum install -y python3
  which pip3
SCRIPT

$installTestServer = <<-SCRIPT
  set -x
  mkdir -p workspace/server
  cd workspace/server
  python3 -m venv venv
  . venv/bin/activate
  pip install fastapi uvicorn gunicorn
SCRIPT

$startTestServer = <<-SCRIPT
  cp -a /tmp/systemd/system/test_server.service /etc/systemd/system/test_server.service  
  systemctl start test_server
  systemctl enable test_server
SCRIPT

Vagrant.configure("2") do |config|
  ## Box
  config.vm.box = "centos/7"

  ## Network
  config.vm.hostname = "#{SERVERNAME}.#{DOMAIN}"
  config.vm.network :forwarded_port, guest: 80, host: 8080, id: "nginx", auto_correct: true
  config.vm.network :forwarded_port, guest: 8000, host: 8000, id: "api", auto_correct: true
  config.vm.network "private_network", ip: "#{SERVER_IP}" 
  
  config.vm.provision "file", source: "./test-server/systemd/test_server.service", destination: "/tmp/systemd/system/test_server.service"
  config.vm.provision "file", source: "./test-server/main.py", destination: "/home/vagrant/workspace/server/main.py"
  
  ## Provision
  config.vm.provision "shell", inline: $bootstrapScript
  config.vm.provision "shell", inline: $installNginxScript
  config.vm.provision "shell", inline: $installRbenvScript, privileged: false
  config.vm.provision "shell", inline: $dockerLoginScript, privileged: false
  config.vm.provision "shell", inline: $installTestServer, privileged: false
  config.vm.provision "shell", inline: $startTestServer
  
  
  ## Hardware
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
    vb.cpus = "2"
  end
end
