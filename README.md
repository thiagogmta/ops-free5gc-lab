![k8s-cluster](/img/cluster.png)

# OPS - 5G Core Orchestration Lab

Para que as redes de 5ª geração funcionem são empregadas diversas tecnologias, dentre elas as funções virtualizadas (VNFs). Uma das questões em relação às redes de 5ª geração é sobre a orquestração dessas funções do núcleo. É necessário um sistema que possa gerir esse núcleo escalando as funções de acordo com a demanda da rede.

Nesse mar de informações esse repositório hospeda "um pedaço de instruções". Nesse repositório você encontra os arquivos necessários para criação de uma infraestrutura capaz de orquestrar o núcleo da rede 5g. O objetivo é criar um ambiente de testes que sirva de laboratório para experimentos diversos.

**Palavras chave:**
- 5G
- Orquestração / Orchestration
- Kubernetes
- Kubeadm

Thiago Guimarães Tavares   
thiagogmta@ifto.edu.br

> **Status do Projeto**
> Concluído: O Cluster está iniciando com suas funções estáveis.
> Em andamento: Testes com My5G-RanTester

## Requisitos

- VirtualBox 6.1
- Vagrant
- Ansible
- 6gb de Memória Ram
- 25GB de espaço em disco

## Arquitetura

**A infraestrutura está organizada em:**

3 Máquinas virtuais virtuais criadas através do **Vagrant** e provisionadas via **Ansible** que irá garanntir os pacotes necessários para o funcionamento.

- Vm1 - k8s-master - Master Node do Cluster
- Vm2 - k8s-node1 - Worker Node 1
- Vm3 - k8s-node2 - Worker Node 2

> **Vagrant** é uma ferramenta que automatiza a criação de máquinas virtuais. O vagrant irá criar três VM's com ubuntu server.
> 
> **Ansible** é uma ferramenta de automação de infraestrutura. Através dessa ferramenta será automatizado o gerenciamento de pacotes, instalação e configuração dos softwares necessários em cada VM.

**Worker Nodes**
Caso necesside de mais (ou menos) worker nodes. Altere a quantidade da variável 'N' em Vagrantfile.
```bash
N = 2
```

**Características do Cluster:**

Ubuntu Focal64 20.04 Kernel 5.4.0-11-generic

- ContainerD
- Kubeadm v1.23
- Kubelet
- Kubectl

Cada VM consome 2gb de memória RAM 

## Documentação

Consulte este [link](doc.md) para documentação básica de utilização deste repositório.

## Referências e Reconhecimento

Agradeço os reponsáveis pelos seguintes repositórios que serviram de base para este projeto.

- [Towards 5Gs Helm](https://github.com/Orange-OpenSource/towards5gs-helm)
- [5G All in One Helm](https://github.com/zanattabruno/5G-all-in-one-helm)
- [5G Core Network Slicing](https://github.com/fhgrings/5g-core-network-slicing)
- [My5G Ran Tester](https://github.com/my5G/my5G-RANTester/wiki)
- [Vagrant Ansible Kubernetes](https://github.com/lvthillo/vagrant-ansible-kubernetes)