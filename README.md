# One Piece of Sea: 5G Core Orchestration Lab

**Palavras chave:**
- 5G
- Orquestração / Orchestration
- Kubernetes
- Kubeadm

Thiago Guimarães Tavares   
thiagogmta@ifto.edu.br

## Status do Projeto

> :construction: Projeto em construção ainda não funcional :construction:
> 
> Concluído: Construção do Cluster
> Em andamento: Criação do Volume persistente; Criação do namespace e aplicação dos charts para o fre5gc

## Descrição

Com certeza você já deve ter ouvido falar das redes de 5ª geração. Next Generation Networking (NGN). Para que as redes de 5ª geração funcionem são empregadas diversas tecnologias, dentre elas as funções virtualizadas (VNFs). Essas funções virtualizadas compõem o núcleo da rede. Cada VNF tem seu papel como: estabelecimento da conexão; autenticação; gerenciamento de sessão etc.

Uma das questões em relação às redes de 5ª geração é sobre a orquestração dessas funções do núcleo. É necessári um sistema que possa gerir esse núcleo escalando essa funções de acordo com a demanda da rede.

E nesse mar de informações referente ao tema esse repositório hospeda "um pedaço de instruções". Nesse repositório você encontra os arquivos necessário para criação de uma infraestrutura capaz de orquestrar o núcleo da rede 5g. O objetivo é criar um ambiente de testes que sirva de laboratório para experimentos diversos.

**A infraestrutura está organizada em:**

3 Máquinas virtuais virtuais criadas através do **Vagrant** e provisionadas via **Ansible** que irá garanntir os pacotes necessários para o funcionamento.

- Vm1 - k8s-master - Este será o Master Node do Cluster
- Vm2 - k8s-node1 - Este será o Worker Node 1 do Cluster
- Vm3 - k8s-node2 - Este será o Worker Node 1 do Cluster

> **Vagrant** é uma ferramenta que automatiza a criação de máquinas virtuais. O vagrant irá criar três VM's com ubuntu server.
> 
> **Ansible** é uma ferramenta de automação de infraestrutura. Através dessa ferramenta será automatizado o gerenciamento de pacotes, instalação e configuração dos softwares necessários em cada VM.

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

Cada VM consome 2gb de memória RAM 

## Pré-requisitos

As ferramentas a seguir devem estar instaladas em sua máquina:

- [Vagrant](https://developer.hashicorp.com/vagrant/downloads)
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)
- [Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads)

> :warning: Utilizou-se para este projeto o Virtualbox na versão 6.1 (a versão 7.0 apresentou instabilidade com o Vagrant. Pode ter sido uma questão pontual, mas fica aqui registrado).

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

**Verifique se o módulo gtp5g está ativo:**
```bash
$ dmesg | grep gtp

[  118.524085] [gtp5g] gtp5g_init: Gtp5g Module initialization Ver: 0.7.1
[  118.524192] [gtp5g] gtp5g_init: 5G GTP module loaded
```

Caso não esteja:
```bash
$ cd gtp5g
$ sudo make
$ sudo make install
```


Enjoy! Seu cluster está pronto para receber aplicações de teste.

## Instalação do Núcleo do 5G
> :construction: Etapa em desenvolvimennto :construction:

## Tratamento de Erros

**Endereçamento:**
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

**Erro ao subir o ambiente**
Caso ao executar o comando `$ vagrant up` pela primeira vez:
- Apenas o node master seja criado ou
- Apenas o node master e 1 worker

Execute o comando: `$vagrant halt` para finalizar as VMs e volte a executar o comando `$ vagrant up`.

Caso continue reportando erro destrua a infra `$ vagrant destroy` e crie novamente `$ vagrant up`.

## Comentários e Observações

- A partir da versão 1.20 do Kubernetes o dockershin foi descontinuado e definitivamente removido na versão 1.24.
- Este projeto utiliza a versão 1.23 do kubernetes adotando o **Containerd** em detrimento do **Docker**.
- Para este repositório está sendo utilizando [Local Path Provisioner](https://github.com/rancher/local-path-provisioner) para armazenamento local (local storage).
  - Por padrão dos charts helm para implantação do núcleo no cluster é utilizado Persistent Volume. O mongodb busca um Persistent Volume. Entretanto, por algum motivo, nesta infraestrutura o mongo não estava se conectando ao volume. Foi utilizado local path como medida paleativa. Dessa forma o mongo se comunica com o local storage.
  - Posteriormente essa feature será revistada.

## Referências

