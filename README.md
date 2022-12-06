# 5G - 

Orquestração
5G
Kubernetes
Kubeadm

Thiago Guimarães Tavares   
thiagogmta@ifto.edu.br

## Status do Projeto

> :construction: Projeto em construção ainda não funcional :construction:

## Descrição

Provavelmente você já deve ter ouvido falar das redes de 5ª geração. Para que as redes de 5ª geração funcionem são empregadas diversas tecnologias dentre elas funções virtualizadas (VNFs) que compõem o núcleo da rede. Cada VNF tem sua função como: estabelecimento da conexão; autenticação; gerenciamento de sessão etc.

Uma das questões em relação as redes de 5ª geração é sobre a orquestração dessas funções do núcleo. Esse repositório hospeda os arquivos necessário para criação de uma infraestrutura capaz de orquestrar o núcleo da rede.

O objetivo é criar um ambiente de testes que sirva de laboratório para experimentos diversos.

**A infraestrutura está organizada em:**

3 Máquinas virtuais virtuais criadas através do **Vagrant** e provisionadas via **Ansible** para que possuam os pacotes necessários para o funcionamento.

- Vm1 - k8s-master - Este será o Master Node do Cluster
- Vm2 - k8s-node1 - Este será o Worker Node 1 do Cluster
- Vm3 - k8s-node2 - Este será o Worker Node 1 do Cluster

> **Vagrant** é uma ferramenta que automatiza a criação de máquinas virtuais. O vagrant irá criar três VM`s com ubuntu server:
> **Ansible** é uma ferramenta de automação de infraestrutura. Através dessa ferramenta será automatizado o gerenciamento de pacotes, instalação e configuração dos softwares necessários.

**Worker Nodes**
Caso necesside de mais (ou menos) worker nodes. Altere a quantidade da variável 'N' em Vagrantfile.
```bash
N = 2
```

## Características do Cluster

Ubuntu Focal64 20.04 Kernel 5.4.0-11-generic

- ContainerD
- Kubeadm v1.23
- Kubelet
- Kubectl
- mongoDB v3.6.8

Cada VM consome 2gb de memória RAM

## Pré-requisitos

As ferramentas a seguir devem estar instaladas em sua máquina:

- [Vagrant](https://developer.hashicorp.com/vagrant/downloads)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)
- [Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads)

> Utilizou-se para este projeto o Virtualbox na versão 6.1 (a versão 7.0 apresentou instabilidade com o Vagrant. Pode ter sido uma questão pontual, mas fica aqui registrado).

## Utilização

Clone este repositório:

```bash
git clone https://github.com/thiagogmta/k8s-containerd.git
```

**Iniciando o ambiente com Vagrant**

Acesse o diretório do repo e crie a infraestrutura com:

```bash
$ cd free5g-kubeadm
$ vagrant up
```

Ao final do deploy teremos a seguinte mensagem:

```bash
PLAY RECAP *********************************************************************
k8s-node2                  : ok=16   changed=15   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Três máquinas virtuais serão criadas no Virtualbox.

![Vagrant Up](/img/vagrantup.png)

Você pode verificar as Máquinas criadas com:

```bash
$ vagrant status

Current machine states:

k8s-master                running (virtualbox)
k8s-node1                 running (virtualbox)
k8s-node2                 running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

**Acessando o Ambiente**

Para acessar o ambiente basta utilizar o comando a seguir seguido do nome da VM que quer acessar.

```bash
$ vagrant ssh k8s-master
```

Será feito acesso a VM via SSH

![Vagrant SSH](/img/vagrantssh.png)

**Verificando o cluster**

Para verificar os nós do cluster utilize:

```bash
$ kubectl get nodes

NAME         STATUS   ROLES                  AGE     VERSION
k8s-master   Ready    control-plane,master   12m     v1.23.0
node1        Ready    <none>                 9m52s   v1.23.0
node2        Ready    <none>                 6m35s   v1.23.0
```

Enjoy! Seu cluster está pronto para receber aplicações de teste.

## Instalação do Núcleo do 5G
```bash

```

## Tratamento de Erros

Caso ocorra o seguinte erro de endereçamento na criação das VM`s:

```bash
The IP address configured for the host-only network is not within the allowed ranges. Please update the address used to be within the allowed ranges and run the command again.

Address: 192.168.10.10 Ranges: 192.168.56.0/21
```

Faça os procedimentos a seguir em seu host e tente executar o Vagrantfile novamente.

```bash
$ sudo su
$ mkdir /etc/vbox/
$ cd /etc/vbox/
$ echo '* 0.0.0.0/0 ::/0' > /etc/vbox/networks.conf
$ chmod 644 /etc/vbox/networks.conf
```

## Comentários

- A partir da versão 1.20 do Kubernetes o dockershin foi descontinuado e definitivamente removido na versão 1.24.
- Este projeto utiliza a versão 1.23 do kubernetes adotando o **Containerd** em detrimento do **Docker**.

## Referências

Esse repoistório é um fork do repositório de Lorenz Vanthillo disponível em [Vagrant-ansible-kubernetes](https://github.com/lvthillo/vagrant-ansible-kubernetes).