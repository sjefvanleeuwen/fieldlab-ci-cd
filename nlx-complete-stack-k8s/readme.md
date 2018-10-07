# NLX complete stack in kubernetes/minikube

This tutorial functions as a walkthrough in setting up NLX running on virtualbox using Vagrant to get things up and running quickly and effortlessly for development purposes.

# Vagrant

With Vagrant you can distribute development environments on Virtualbox, but can also work with VMWare, Openstack and Docker, AWS or any other provider. 

## Get started

To get started with virtualbox and vagrant, first intall them, if you have not done so, using:

```
$ sudo apt install virtualbox
$ sudo apt install vagrant
```

A vagrant file for setting things up is inclued in the directory of this readme.

'cd' to the directoy and simply execute:

```
$ vagrant up
```

Vagrantfile

```
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  #config.vm.network "forwarded_port", guest: 8443, host: 8443
  #config.vm.network "public_network"
  config.vm.provider "virtualbox" do |vb|
    vb.name = "wigo4it-nlx-ubuntu"
    vb.cpus = 4
    vb.memory = "4096"
  end
  config.vm.provision "shell", inline: $script
end

$script = <<-SCRIPT
sudo apt update
sudo apt upgrade -y
sudo apt install apt-transport-https ca-certificates curl software-properties-common -yaml
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt update
sudo apt install docker-ce -y
sudo usermod -aG docker $USER
sudo docker container run hello-world
#sudo snap install microk8s --beta --classic
sudo snap install kubectl --classic
#sudo apt-get install virtualbox -y
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.30.0/minikube-linux-amd64 && chmod +x minikube && sudo cp minikube /usr/local/bin/ && rm minikube
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/latest/skaffold-linux-amd64 && chmod +x skaffold && sudo mv skaffold /usr/local/bin
sudo snap install go --classic
sudo apt-get install git -y
git clone https://gitlab.com/commonground/nlx.git/
cd nlx
sudo minikube start --vm-driver=none --cpus 4 --memory 4096 --disk-size=100G
sudo snap install helm --classic
helm init
helm install stable/traefik --name traefik --namespace traefik --values helm/traefik-values-minikube.yaml
helm install stable/nginx-ingress --version 0.17.1 --name nginx-ingress --namespace=nginx-ingress --values helm/nginx-ingress-values-minikube.yaml
helm install stable/postgresql --name postgresql --namespace=postgresql --values helm/postgresql-values-minikube.yaml
MINIKUBE_IP=$(minikube ip) skaffold dev --profile minikube

echo "$(minikube ip)                 traefik.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip)            docs.dev.nlx.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip)      certportal.dev.nlx.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip)       directory.dev.nlx.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip)   directory-api.dev.nlx.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip)           txlog.dev.rdw.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip)        irma-api.dev.rdw.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip)     insight-api.dev.rdw.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip)           txlog.dev.brp.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip)        irma-api.dev.brp.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip)     insight-api.dev.brp.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip)       txlog.dev.haarlem.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip)    irma-api.dev.haarlem.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip) insight-api.dev.haarlem.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip)      outway.dev.haarlem.minikube" | sudo tee -a /etc/hosts
echo "$(minikube ip) application.dev.haarlem.minikube" | sudo tee -a /etc/hosts

SCRIPT
```