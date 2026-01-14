NODE_ROLES = ["server-0", "agent-0","agent-1"]
NODE_BOXES = ["bento/ubuntu-20.04", "bento/ubuntu-20.04", "bento/ubuntu-20.04"]
NODE_CPUS = 2
NODE_MEMORY = 2048
NETWORK_PREFIX = "192.168.56"

def setup_vm(vm, role, node_num)
  vm.box = NODE_BOXES[node_num]
  vm.hostname = "#{role}"
  
  node_ip = "#{NETWORK_PREFIX}.#{10 + node_num}"
  vm.network "private_network", ip: node_ip, net_mask: "255.255.255.0"
  
  vm.provision "ansible", run: 'once' do |ansible|
    ansible.compatibility_mode = "2.0"
    ansible.playbook = "playbooks/site.yml"
    ansible.groups = {
      "server" => NODE_ROLES.grep(/^server/),
      "agent" => NODE_ROLES.grep(/^agent/),
      "k3s_cluster:children" => ["server", "agent"],
    }

    ansible.extra_vars = {
      k3s_version: "v1.31.12+k3s1",
      api_endpoint: "#{NETWORK_PREFIX}.10",
      
      token: "myvagrant",

      extra_server_args: "--node-ip #{node_ip} --flannel-iface eth1", 
      extra_agent_args: "--node-ip #{node_ip} --flannel-iface eth1",

    }
  end
end

Vagrant.configure("2") do |config|

  config.vm.boot_timeout = 600
  
  # disable sync folder
  config.vm.synced_folder '.', '/vagrant', disabled: true
  
  # disable serial port (error in wsl2 x vbox)
  config.vm.provider "virtualbox" do |vb|
    vb.cpus = NODE_CPUS
    vb.memory = NODE_MEMORY
    vb.customize [ "modifyvm", :id, "--uartmode1", "disconnected" ]
  end


  NODE_ROLES.each_with_index do |name, i|
    config.vm.define name do |node|
      setup_vm(node.vm, name, i)
    end
  end
end