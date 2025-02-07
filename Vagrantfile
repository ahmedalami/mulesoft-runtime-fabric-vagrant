BOX = "bento/centos-8.4"

CONTROLLER_CPU = 2
CONTROLLER_MEMORY = 8192

WORKERS = 2
WORKER_CPU = 2
WORKER_MEMORY = 15360

RTF_NETWORK_PREFIX="192.168.56"
NODE_NAME_PREFIX = "rtf"

Vagrant.configure("2") do |config|

	config.vm.provision "shell", inline: $set_environment_variables, run: 'always'
	config.vm.provision "shell", inline: $ensure_init_script, run: 'always'

	config.vm.disk :disk, size: "20GB", name: "docker_storage"

	config.vm.define "controller" do |controller|
		controller.vm.box = BOX
		controller.vm.hostname = "#{NODE_NAME_PREFIX}-controller"
		controller.vm.network "private_network", ip: "#{RTF_NETWORK_PREFIX}.10"
		controller.vm.provider "virtualbox" do |vb|
			vb.cpus = CONTROLLER_CPU
			vb.memory = CONTROLLER_MEMORY
		end

		controller.vm.disk :disk, size: "5GB", name: "etcd_storage"

		controller.vm.provision "shell", inline: $ensure_controller_env, run: 'always', args: ["#{RTF_NETWORK_PREFIX}.10", "#{RTF_ACTIVATION_DATA}", "#{RTF_MULE_LICENSE}"]

		controller.vm.network "forwarded_port", guest: 32009, host: 32009
    end

    (1..WORKERS).each do |id|
    	config.vm.define "worker-#{id}" do |worker|
	    	worker.vm.box = BOX
			worker.vm.hostname = "#{NODE_NAME_PREFIX}-worker-#{id}"
			worker.vm.network "private_network", ip: "#{RTF_NETWORK_PREFIX}.1#{id}"
			worker.vm.provider "virtualbox" do |vb|
				vb.cpus = WORKER_CPU
				vb.memory = WORKER_MEMORY
	    	end

	    	worker.vm.provision "shell", inline: $ensure_worker_env, run: 'always', args: ["#{RTF_NETWORK_PREFIX}.1#{id}", "#{RTF_NETWORK_PREFIX}.10"]
    	end
	end
end


$set_environment_variables = <<SCRIPT
tee "/etc/profile.d/rtf-generator-environment.sh" > "/dev/null" <<EOF
export RTF_CONTROLLER_IPS=#{RTF_NETWORK_PREFIX}.10
export RTF_WORKER_IPS="#{RTF_NETWORK_PREFIX}.11 #{RTF_NETWORK_PREFIX}.12"
export RTF_ACTIVATION_DATA=#{RTF_ACTIVATION_DATA}
export RTF_MULE_LICENSE=#{RTF_MULE_LICENSE}
export RTF_SERVICE_UID=vagrant
export RTF_SERVICE_GID=vagrant
EOF
SCRIPT

$ensure_init_script = <<SCRIPT
yum install unzip -y
curl https://runtime-fabric-eu.s3.amazonaws.com/install-scripts/rtf-install-scripts-20220112-f8b0e44.zip -s --output rtf-install-scripts.zip
unzip -o rtf-install-scripts.zip -d rtf-install-scripts
mkdir -p /opt/anypoint/runtimefabric && cp ./rtf-install-scripts/scripts/init.sh /opt/anypoint/runtimefabric/init.sh && chmod +x /opt/anypoint/runtimefabric/init.sh
chown -R vagrant:vagrant /opt/anypoint/runtimefabric
SCRIPT

$ensure_controller_env = <<SCRIPT
cat > /opt/anypoint/runtimefabric/env <<EOF 
	RTF_PRIVATE_IP=${1}
	RTF_NODE_ROLE=controller_node 
	RTF_INSTALL_ROLE=leader 
	RTF_INSTALL_PACKAGE_URL= 
	RTF_ETCD_DEVICE=/dev/sdc 
	RTF_DOCKER_DEVICE=/dev/sdb 
	RTF_TOKEN='my-cluster-token' 
	RTF_NAME='runtime-fabric' 
	RTF_ACTIVATION_DATA='${2}' 
	RTF_MULE_LICENSE='${3}' 
	RTF_HTTP_PROXY='' 
	RTF_NO_PROXY='' 
	RTF_MONITORING_PROXY='' 
	RTF_SERVICE_UID='1000' 
	RTF_SERVICE_GID='1000' 
	POD_NETWORK_CIDR='10.244.0.0/16' 
	SERVICE_CIDR='10.100.0.0/16' 
	DISABLE_SELINUX='false' 
EOF
SCRIPT

$ensure_worker_env = <<SCRIPT
cat > /opt/anypoint/runtimefabric/env <<EOF 
	RTF_PRIVATE_IP=${1}
	RTF_NODE_ROLE=worker_node 
	RTF_INSTALL_ROLE=joiner 
	RTF_DOCKER_DEVICE=/dev/sdb 
	RTF_TOKEN='my-cluster-token' 
	RTF_INSTALLER_IP=${2}
	RTF_HTTP_PROXY='' 
	RTF_NO_PROXY='' 
	RTF_MONITORING_PROXY='' 
	RTF_SERVICE_UID='1000' 
	RTF_SERVICE_GID='1000' 
	POD_NETWORK_CIDR='10.244.0.0/16' 
	SERVICE_CIDR='10.100.0.0/16' 
	DISABLE_SELINUX='false' 
EOF
SCRIPT


$install_rtfctl = <<SCRIPT
curl -L https://anypoint.mulesoft.com/runtimefabric/api/download/rtfctl/latest -o rtfctl
sudo chmod +x rtfctl
SCRIPT

