![k8s-cluster](/img/cluster.png)

# One Piece of Sea: 5G Core Orchestration Lab

Bem vido a documentação de utilização do OPS-free5g-Lab.

## Pré-requisitos

- 6gb de Memória Ram
- 25GB de espaço em disco

As ferramentas a seguir devem estar instaladas em sua máquina:

- [Oracle VirtualBox](https://www.virtualbox.org/wiki/Downloads)
```bash
cd
wget https://download.virtualbox.org/virtualbox/6.1.42/virtualbox-6.1_6.1.42-155177~Ubuntu~jammy_amd64.deb
sudo dpkg -i virtualbox-6.1_6.1.42-155177~Ubuntu~jammy_amd64.deb
```

- [Vagrant](https://developer.hashicorp.com/vagrant/downloads)
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant
```

- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)
```bash
sudo apt-add-repository ppa:ansible/ansible
sudo apt update
sudo apt install ansible
```

> Caso retorne erro de dependências desencontradas corrija com: 
> sudo apt --fix-broken install

## Implantação do Ambiente

**Clone este repositório**
```bash
git clone https://github.com/thiagogmta/ops-free5gc-lab.git
```

**Iniciando o ambiente**

Acesse o diretório do repositório e crie a infraestrutura com:

```bash
cd ops-free5gc-lab
vagrant up
```

Ao final do deploy teremos a seguinte mensagem:

```bash
PLAY RECAP *********************************************************************
k8s-node2                  : ok=16   changed=15   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
```

Três máquinas virtuais serão criadas no Virtualbox.

![Vagrant Up](/img/vagrant.png)

Você pode verificar as Máquinas criadas com:

```bash
vagrant status
```

![Vagrant status](/img/vagrantstatus.png)

**Acessando o Ambiente**

Para acessar o ambiente basta utilizar o comando a seguir seguido do nome da VM que quer acessar.

```bash
vagrant ssh k8s-master
```

Será feito acesso a VM via SSH

![Vagrant SSH](/img/sshmaster.png)

**Verificando o cluster**

Para verificar os nós do cluster utilize:

```bash
kubectl get nodes
```

![kubectl get nodes](/img/getnodes.png)

**Verifique se o módulo gtp5g está ativo (opicional):**
```bash
dmesg | grep gtp
```

![gtp5g](/img/gtp5g.png)

Caso não esteja execute:
```bash
cd gtp5g
make clean && make
sudo make install
```

## Deploy do Núcleo do 5G

Os comandos a seguir serão executados no **Master Node**. 

### Volume e Storage Class

**Criando um Volume**

```bash
mkdir /home/vagrant/kubedata
kubectl apply -f /ops-free5gc-lab/volume/persistentVolume.yaml
```

**Criando um Storage Class**

Utilizaremos um deploy da Rancher para facilitar o processo:

```bash
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.23/deploy/local-path-storage.yaml
```

**Definindo o StorageClass como default**

```bash
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

### Deploy do Núcleo

Com o ambiente pronto iremos inserir os charts helm para o deploy do núcleo.

```bash
cd ops-free5gc-lab/charts
helm install core free5gc
```

Podemos verificar a criação do núcleo com o comando `$ kubectl get pods`. Inicialmente pode levar alguns instantes até que todos os pods estabilizem.

![kubectl get pods](/img/getpods.png)

### Acessando a Interface Web

Você pode acessar a interface web diretamente do seu host de usuário através do navegador. Para isso utilize em seu navegador o endereço IP: ***192.168.50.10:30500*** (esse é o endereço do Master Node que encontra-se no vagrantfile).

O usuário padrão é *admin* e a senha é *free5gc*.

![webui](/img/webui.png)

Vá ao menu lateral Subscribers > New Subscriber > Submit (com todos os valores padrão).

![subscribers](/img/subscribers.png)

## My5G RANTester

[My5G RANTester](https://github.com/my5G/my5G-RANTester) é uma ferramenta para emular planos de controle e dados do UE (equipamento de usuário) e gNB (estação base 5G).

**Implantação**

```bash
cd ops-free5gc-lab/charts
helm install ran rantester
```

Verifique o estado do com:

```bash
kubectl get pods
```

![rantester](/img/rantester.png)

**Executando o RANTester**


## Monitoramento



## Tratamento de Erros

**Endereçamento:**
Caso ocorra o seguinte erro de endereçamento na criação das VM`s:

![The IP addres configured for the host-only](/img/errorede.png)

Faça os procedimentos a seguir em seu host:

```bash
sudo su
mkdir /etc/vbox/
cd /etc/vbox/
echo '* 0.0.0.0/0 ::/0' > /etc/vbox/networks.conf
chmod 644 /etc/vbox/networks.conf
```

**Erro ao subir o ambiente**
Caso ao executar o comando `$ vagrant up` pela primeira vez:
- Apenas o node master seja criado ou
- Apenas o node master e 1 worker

Execute o comando: `$vagrant halt` para finalizar as VMs e volte a executar o comando `$ vagrant up`.

Caso continue reportando erro destrua a infra `$ vagrant destroy` e crie novamente `$ vagrant up`.

## Comentários e Observações

- A função n3iwf está desabilitada.
- Utilizou-se para este projeto o Virtualbox na versão 6.1 (a versão 7.0 apresentou instabilidade com o Vagrant. Pode ter sido uma questão pontual, mas fica aqui registrado).
- A partir da versão 1.20 do Kubernetes o dockershin foi descontinuado e definitivamente removido na versão 1.24.
- Este projeto utiliza a versão 1.23 do kubernetes adotando o **Containerd** em detrimento do **Docker**.
- Para este repositório está sendo utilizando [Local Path Provisioner](https://github.com/rancher/local-path-provisioner) para armazenamento local (local storage).
  - Por padrão dos charts helm para implantação do núcleo no cluster é utilizado Persistent Volume. O mongodb busca um Persistent Volume. Entretanto, por algum motivo, nesta infraestrutura o mongo não estava se conectando ao volume. Foi utilizado local path como medida paleativa. Dessa forma o mongo se comunica com o local storage.
  - Posteriormente essa feature será revistada.