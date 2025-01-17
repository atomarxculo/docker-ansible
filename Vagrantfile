def get_ip(index = 1)
  $ip_range.sub('xx', (index).to_s)
end

$max_nodes = 2
$ip_range = '10.1.10.2xx'
$all_nodes = Array.new($max_nodes).fill { |i| "#{get_ip(i + 1)}" }

$ansible_groups = {
  "docker_node" => ["docker-node[1:#{$max_nodes}]"],
  "all:vars" => {
    "vagrant_docker_node_ips" => $all_nodes,
    "vagrant_loadbalancer_ip" => "#{get_ip(0)}"
  }
}

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/focal64"

  (1..$max_nodes).each do |i|
    config.vm.define "docker-node#{i}" do |node|
      node_ip_address = "#{get_ip(i)}"
      node.vm.network "private_network", ip: node_ip_address
      node.vm.hostname = "docker-node#{i}"
    end
  end

  config.vm.define "loadbalancer" do |lb|
    node_ip_address = "#{get_ip(0)}"
    lb.vm.network "private_network", ip: node_ip_address
    lb.vm.hostname = "loadbalancer"
  end

  config.vm.provision "traefik", type: "ansible", run: "never" do |ansible|
    ansible.playbook = "playbook-traefik.yml"
    ansible.groups = $ansible_groups
  end

  # Vault requires a running Consul cluster with an elected leader
  # Nomad in turn requires tokens from Vault
  # This playbook also includes the load balancer and DNS configuration
  config.vm.provision "docker", type: "ansible", run: "never" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.groups = $ansible_groups
  end

  # Increase memory for Parallels Desktop
  config.vm.provider "parallels" do |p, o|
    p.memory = "1024"
  end

  # Increase memory for Virtualbox
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end

  # Increase memory for VMware
  ["vmware_fusion", "vmware_workstation"].each do |p|
    config.vm.provider p do |v|
      v.vmx["memsize"] = "1024"
    end
  end
end
