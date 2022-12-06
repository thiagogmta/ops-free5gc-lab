# Definição da imagem linux base:
IMAGE_NAME = "bento/ubuntu-20.04"

# Quantidade de worker nodes (caso precise de mais workers altere a quantidade)
N = 2

# Definições gerais de memória e processamento para cada Node
Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
    end
    
    # Definições do Master Node (nome, endereçamento e arquivo de playbook)
    config.vm.define "k8s-master" do |master|
        master.vm.box = IMAGE_NAME
        master.vm.network "private_network", ip: "192.168.50.10" 
        master.vm.hostname = "k8s-master"
         master.vm.provision "ansible" do |ansible|
             ansible.playbook = "k8s-setup/master-playbook.yml"
         end
    end

    # Definições dos Worker Nodes
    (1..N).each do |i|
        config.vm.define "k8s-node#{i}" do |node|
            node.vm.box = IMAGE_NAME
            node.vm.network "private_network", ip: "192.168.50.#{i + 10}"
            node.vm.hostname = "node#{i}"
            node.vm.provision "ansible" do |ansible|
                ansible.playbook = "k8s-setup/node-playbook.yml"
            end
        end
    end
end