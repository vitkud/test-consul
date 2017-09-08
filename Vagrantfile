# -*- mode: ruby -*-
# vi: set ft=ruby :

$script = <<SCRIPT

echo "Installing dependencies ..."
sudo apt-get update
sudo apt-get install -y unzip curl jq

echo "Determining Consul version to install ..."
CHECKPOINT_URL="https://checkpoint-api.hashicorp.com/v1/check"
if [ -z "$CONSUL_DEMO_VERSION" ]; then
    CONSUL_DEMO_VERSION=$(curl -s "${CHECKPOINT_URL}"/consul | jq .current_version | tr -d '"')
fi

echo "Fetching Consul version ${CONSUL_DEMO_VERSION} ..."
cd /tmp/
curl -s https://releases.hashicorp.com/consul/${CONSUL_DEMO_VERSION}/consul_${CONSUL_DEMO_VERSION}_linux_amd64.zip -o consul.zip

echo "Installing Consul version ${CONSUL_DEMO_VERSION} ..."
unzip consul.zip
sudo chmod +x consul
sudo mv consul /usr/bin/consul

sudo mkdir /etc/consul.d
sudo chmod a+w /etc/consul.d

SCRIPT

# Specify a Consul version
CONSUL_DEMO_VERSION = ENV['CONSUL_DEMO_VERSION']

# Specify a custom Vagrant box for the demo
DEMO_BOX_NAME = ENV['DEMO_BOX_NAME'] || "debian/jessie64"

# Vagrantfile API/syntax version.
# NB: Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
	config.vm.box = DEMO_BOX_NAME

	config.vm.provision "shell",
			inline: $script,
			env: {'CONSUL_DEMO_VERSION' => CONSUL_DEMO_VERSION}

	(1..5).each do |i|
		config.vm.define "n#{i}" do |node|
			node.vm.hostname = "n#{i}"
			node.vm.network "private_network", ip: "172.20.20.#{10+i-1}"
			node.vm.provision "shell", run: "always", inline: "nohup consul agent -server -bootstrap-expect=2 -bind=172.20.20.#{10+i-1} -data-dir=/tmp/consul -config-dir=/etc/consul.d -enable-script-checks=true >>consul.log 2>&1 &"
			#if i > 1 then
			#	node.vm.provision "shell", run: "always", inline: "consul join 172.20.20.10"
			#end
		end
	end

end
