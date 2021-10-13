# Loading Settings from settings.yml file
require 'yaml'
settings = YAML.load_file("settings.yml")

system("
    if [ '#{ARGV[0]}' = 'up' ] || [ '#{ARGV[0]}' = 'provision' ]; then
      ansible-galaxy collection install community.kubernetes -p provisioning/ansible_collections/
    fi
")

dictNodeInfo = settings['node_info']
defaultRouter = settings['default_router']
bridgeOrder = settings['net_bridge_order']
kubeSettings = settings['kubernetes'] || {}
nameServers = settings['nameservers']

# Some default settings for the K8s cluster...
kube_cni = kubeSettings['kube_cni'] || "calico"
kube_podnet_cidr = kubeSettings['kube_podnet_cidr'] || "172.16.0.0/16"
kube_service_cidr = kubeSettings['kube_service_cidr'] || "10.96.0.0/12"

ansibleWorkerGroup = []
ansibleMasterGroup = []

# VM Provision Shell Script (override default router to eth1)
$script = <<-SCRIPT
ip route delete default
ip route add default via #{defaultRouter}
cat <<EOF>/etc/netplan/99-route-overrides.yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4-overrides:
        use-routes: false
        use-dns: false
    eth1:
      gateway4: #{defaultRouter}
      nameservers: 
        addresses: #{nameServers}
EOF
SCRIPT

Vagrant.configure(2) do |config|
  #Define the number of nodes to spin up

  dictNodeInfo.each_with_index do |(hostname), index|
    nodeInfo = dictNodeInfo[hostname]

    config.vm.define hostname do |node|
      if nodeInfo['role'] == "worker"
        ansibleWorkerGroup.append(hostname)
      elsif nodeInfo['role'] == "master"
        ansibleMasterGroup.append(hostname)
      end
     
      node.vm.box = "bento/ubuntu-20.04"
      node.vm.provider "virtualbox" do |vb|
        vb.memory = nodeInfo['memory'] || 4096
        vb.cpus = nodeInfo['cpus'] || 1
      end
      node.vm.hostname = hostname

      node.vm.network :public_network,
        ip: nodeInfo['ip'],
        :mac => nodeInfo['mac'] || nil,
        bridge: bridgeOrder,
        hostname: true

      node.vm.provision :shell, :inline => $script

      # Make sure to run ansible provisioner last
      if index == dictNodeInfo.keys.length() - 1
        node.vm.provision "ansible" do |ansible|
          ansible.limit    = "all"
          ansible.verbose  = "vv"
          ansible.playbook = "provisioning/playbook.yml"
          ansible.galaxy_role_file = "provisioning/requirements.yaml"
          ansible.config_file = "provisioning/ansible.cfg"

          ansible.groups = {
            "all:vars" => {
              "deploy_user" => "vagrant",
              "kube_vagrant" => settings.to_json,
              "primary_interface" => "eth1"
            },
            "workers" => ansibleWorkerGroup,
            "masters" => ansibleMasterGroup
          }
        end
      end
    end
  end
end
