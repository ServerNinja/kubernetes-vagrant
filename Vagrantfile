defaultRouter = "192.168.1.254"

dictNodeInfo = {
  'node0' => {
      'ip'       => "192.168.1.60",
      'mac'      => "08002799F0D4",
      'hostname' => "k8s-master-1",
      'role'     => "master"
  },
  'node1' => {
      'ip'       => "192.168.1.61",
      'mac'      => "080027F69436",
      'hostname' => "k8s-worker-1",
      'role'     => "worker"
  },
  'node2' => { 
      'ip'       => "192.168.1.62",
      'mac'      => "080027D8A361",
      'hostname' => "k8s-worker-2",
      'role'     => "worker"
  },
}

bridgeOrder = [
  "en0: Wi-Fi (Wireless)",
]

ansibleWorkerGroup = []
ansibleMasterGroup = []

Vagrant.configure(2) do |config|
  #Define the number of nodes to spin up
  N = dictNodeInfo.keys.length()

  #Iterate over nodes
  (1..N).each do |node_id|
    nid = (node_id - 1)
    nodename = "node#{nid}"
    nodeInfo = dictNodeInfo[nodename]

    config.vm.define nodeInfo['hostname'] do |node|

      if nodeInfo['role'] == "worker"
        ansibleWorkerGroup.append(nodeInfo['hostname'])
      elsif nodeInfo['role'] == "master"
        ansibleMasterGroup.append(nodeInfo['hostname'])
      end
     
      node.vm.box = "bento/ubuntu-20.04"
      node.vm.provider "virtualbox" do |vb|
        vb.memory = "4096"
      end
      node.vm.hostname = nodeInfo['hostname']

      node.vm.network :public_network,
        ip: nodeInfo['ip'],
        :mac => nodeInfo['mac'],
        bridge: bridgeOrder,
        hostname: true

      node.vm.provision :shell,
        :inline => "ip route delete default 2>&1 >/dev/null || true; ip route add default via #{defaultRouter}"

      if node_id == N
        node.vm.provision "ansible" do |ansible|
          ansible.limit    = "all"
          ansible.verbose  = "vv"
          ansible.playbook = "provisioning/playbook.yml"
          ansible.groups   = {
            "workers" => ansibleWorkerGroup,
            "masters" => ansibleMasterGroup
          }
        end
      end

    end
  end
end
