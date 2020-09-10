# Loading Settings from settings.yml file
require 'yaml'
settings = YAML.load_file("settings.yml")

dictNodeInfo = settings['node_info']
defaultRouter = settings['default_router']
bridgeOrder = settings['net_bridge_order']
kubeSettings = settings['kubernetes'] || {}

# Some default settings for the K8s cluster...
kube_cni = kubeSettings['kube_cni'] || "calico"
kube_podnet_cidr = kubeSettings['kube_podnet_cidr'] || "172.16.0.0/16"
kube_service_cidr = kubeSettings['kube_service_cidr'] || "10.96.0.0/12"

ansibleWorkerGroup = []
ansibleMasterGroup = []

Vagrant.configure(2) do |config|
  #Define the number of nodes to spin up
  N = dictNodeInfo.keys.length()
  counter = 0

  dictNodeInfo.each_key do |hostname|
    counter+=1
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

      node.vm.provision :shell,
        :inline => "ip route delete default 2>&1 >/dev/null || true; ip route add default via #{defaultRouter}"

      if counter == N
        node.vm.provision "ansible" do |ansible|
          ansible.limit    = "all"
          ansible.verbose  = "vv"
          ansible.playbook = "provisioning/playbook.yml"

          ansible.groups = {
            "all:vars" => {
              "deploy_user" => "vagrant",

              "kube_cni" => kube_cni,
              "kube_podnet_cidr" => kube_podnet_cidr,
              "kube_serice_cidr" => kube_service_cidr

            },
            "workers" => ansibleWorkerGroup,
            "masters" => ansibleMasterGroup
          }
        end
      end

    end
  end
end
