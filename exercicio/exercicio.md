<img width=100% src="https://capsule-render.vercel.app/api?type=waving&color=296999&height=120&section=header"/>

# devops-hro | Projeto Kubernetes
### ___________________________


## Prerequisitos
- 4 máquinas virtuais com 2/4 processadores e 6/8 gb de memória ram
- 1 domínio
- Sistema operacional Ubuntu 22.04 LTS

- Domínio usado pelo instrutor do curso é: hro.dev.br

```sh
ssh -i rancherserver.pem ubuntu<ip> - RancherServer
ssh -i k8s-1.pem ubuntu@<ip>        - k8s-1         - HOST B
sh -i k8s-2 ubuntu@<ip>             - k8s-2         - HOST C
ssh -i k8s-3 ubuntu@<ip>            - k8s-3         - HOST D
```
### Instalação do Docker
```sh
 
sudo su
apt update && apt upgrade -y && \
curl https://releases.rancher.com/install-docker/24.0.sh | sh && \
systemctl enable docker && \
usermod -aG docker ubuntu
docker --version
```

# Rancher - Single Node

### Instalar Rancher - Single Node
Nesse exercício iremos instalar o Rancher 2.8.2 versão single node. Isso significa que o Rancher e todos seus componentes estão em um container. 

Entrar no host A, que será usado para hospedar o Rancher Server. Iremos verficar se não tem nenhum container rodando ou parado, e depois iremos instalar o Rancher.
```sh
$ docker ps -a
$ sudo docker run -d --privileged --name rancher --restart=unless-stopped -v /opt/rancher:/var/lib/rancher  -p 80:80 -p 443:443 rancher/rancher:v2.8.2
```
Com o Rancher já rodando, irei adicionar a entrada de cada DNS para o IP de cada máquina.

```sh
$ rancher.<dominio> = IP do host A
```
Kubernetes

### Criar cluster Kubernetes

Nesse exercício iremos criar um cluster Kubernetes. Após criar o cluster, iremos instalar o kubectl no host A, e iremos usar para interagir com o cluster.

Seguir as instruções na aula para fazer o deployment do cluster.
Após fazer a configuração, o Rancher irá exibir um comando de docker run, para adicionar os host's.

Adicionar o host B e host C. 

Pegar o seu comando no seu rancher.
```sh
$curl -fL https://rancher.hro.dev.br/system-agent-install.sh | sudo  sh -s - --server https://rancher.hro.dev.br --label 'cattle.io/os=linux' --token c2ntbc2xtgs4xwqjq7md5k85hksjnzqxm8g2pfqzl5vnvf48nbw86j --ca-checksum 43e0199867976f8417fab1b5fb7ce7c5af66bce2349c21d3bb396c9e406c0f28 --etcd --controlplane --worker --node-name k8s-1
```
Será um cluster com 3 nós.
Navegar pelo Rancher e ver os painéis e funcionalidades.

Kubectl

### Instalar kubectl no host A

Agora iremos instalar o kubectl, que é a CLI do kubernetes. Através do kubectl é que iremos interagir com o cluster.
```sh
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gnupg
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg # allow unprivileged APT programs to read this keyring
```
###  Observação:
Em versões anteriores ao Debian 12 e Ubuntu 22.04, a pasta /etc/apt/keyringsnão existe por padrão e deve ser criada antes do comando curl.
Com o kubectl instalado, pegar as credenciais de acesso no Rancher e configurar o kubectl.
```sh
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list   # helps tools such as command-not-found to work correctly
sudo apt-get update
sudo apt-get install -y kubectl
```
##### Configuração do .kube/config
```sh
mkdir ~/.kube
vim ~/.kube/config
chmod 0600 ~/.kube/config
kubectl get nodes
```

##### Verifique os nós
```sh
$ kubectl get nodes
```

DNS

### Traefik - DNS

*.rancher.hro.dev.br

O Traefik é a aplicação que iremos usar como ingress. Ele irá ficar escutando pelas entradas de DNS que o cluster deve responder. Ele possui um dashboard de  monitoramento e com um resumo de todas as entradas que estão no cluster.

```sh
$ kubectl apply -f https://raw.githubusercontent.com/hroliveira/devops-hro/main/exercicios/traefik/v2.9/traefik-rbac.yaml
$ kubectl apply -f https://raw.githubusercontent.com/hroliveira/devops-hro/main/exercicios/traefik/v2.9/traefikv2-1-ds.yaml

```
Para verificar se o os serviços estão radondo 
```sh
$ kubectl --namespace=kube-system get pods
```

<div align="center">
<img src="https://github.com/hroliveira/devops-hro/blob/main/img/traefik.png">
</div>

Agora iremos configurar o DNS pelo qual o Traefik irá responder. No arquivo ui.yml, localizar a url, e fazer a alteração. Após a alteração feita, iremos rodar o comando abaixo para aplicar o deployment no cluster.
```sh
$ cd devops-hro/exercicios/
$ kubectl apply -f ui.yml
```



<img width=100% src="https://capsule-render.vercel.app/api?type=waving&color=296999&height=120&section=footer"/>