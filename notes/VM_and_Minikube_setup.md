### **🧭 Objetivo da Aula**
Configurar uma VM na Google Cloud com as ferramentas necessárias para deploy/teste com Docker, Minikube e Kubernetes (`kubectl`).

## ✅ Step-by-Step Resumido

1. Acesse o Google Cloud
    
    Acesse: https://console.cloud.google.com/

    Crie ou entre em sua conta.

    No menu, procure por "VM Instances".

---

2. Criação da VM

    2.1 Clique em `Create Instance`.

    2.2 Nome: `gitops-machine`

    2.3 Região sugerida: `us-central1`

    2.4 Máquina:

    - Tipo: `E2-standard-4` (4 vCPU, 16 GB RAM)

    2.5 Disco:

    - OS: `Ubuntu 20.04 LTS` 

    - Espaço: `128 GB`

    2.6 Networking:

    - Ative: `Allow HTTP`, `Allow HTTPS`, `IP forwarding`

    2.7 Clique em **Create**

---    

3. Conectar na VM

    - Após criada, clique em `SSH > Open in browser window`

    - Aguarde abrir o terminal

---    

4. Instalar o Git e Clonar Repositório

```bash
sudo apt update
sudo apt install git -y
git clone https://github.com/SeuUsuario/SeuProjeto.git
cd SeuProjeto
```
---

5. Instalar Docker

Siga os comandos do site oficial: https://docs.docker.com/engine/install/ubuntu/

5.1. Pré-requisitos:

```bash
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release -y
```

5.2. Adicionar repositório Docker:

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] \
  https://download.docker.com/linux/ubuntu focal stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

5.3. Instalar Docker:

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io -y
```

5.4. Verificar instalação:

```bash
sudo docker run hello-world
```

5.5. Evitar uso de sudo no Docker:

```bash
sudo groupadd docker         # pode retornar "already exists"
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world       # sem sudo
```

5.6. Habilitar o serviço Docker:

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

---
6. Instalar Minikube

Site oficial: https://minikube.sigs.k8s.io/docs/start/

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```

**Iniciar Minikube:**

```bash
minikube start
```

---

7. Instalar kubectl (Kubernetes CLI)

7.1. Usando Snap (mais simples em Ubuntu):

```bash
sudo snap install kubectl --classic
```

7.2. Verificar instalação:

```bash
kubectl version --client
```
---

8. Validar Configurações

Verificar status do Minikube:

```bash
minikube status
```

Verificar cluster:

```bash
kubectl get nodes
kubectl cluster-info
```

Verificar container Minikube rodando:

```bash
docker ps
```