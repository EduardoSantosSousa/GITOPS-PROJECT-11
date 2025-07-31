## **✅ Resumo: Configuração do Argo CD com Jenkins (CI/CD)**

### **🔧 Pré-requisitos**

- Ter um cluster Kubernetes em funcionamento (ex: Minikube ou GKE)
- Ter o `kubectl` instalado na sua máquina ou VM
- Jenkins instalado e funcionando
- Docker configurado (com sua imagem já no DockerHub)
---
### 📦 1. Instalação do Argo CD

**1.1 Criar Namespace:**

```bash
kubectl create namespace argocd
```

**1.2 Instalar Argo CD via manifest:**

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

**1.3 Verificar se os pods estão prontos:**

```bash
kubectl get all -n argocd
```
---
### 🌐 2. Expor o Argo CD externamente

**2.1 Alterar o tipo do serviço `argocd-server` de `ClusterIP` para `NodePort`:**

```bash
kubectl get all -n argocd
```

→ Mude type: ClusterIP para type: NodePort

**2.2 Obter a porta externa:**

```bash
http://<EXTERNAL-IP>:<NODE-PORT>
```
→ Use a opção "Avançado" se o navegador alertar sobre certificado inseguro.

### 🔐 3 Login no Argo CD

3.1 Usuário padrão: `admin`

3.2 Obter senha:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```
---

### ⚙️ 4. Criar kubeconfig customizado (para Jenkins)

**4.1 Exportar conteúdo dos arquivos de certificados:**

```bash
cat ~/.minikube/ca.crt | base64 -w 0
cat ~/.minikube/client.crt | base64 -w 0
cat ~/.minikube/client.key | base64 -w 0
```

**4.2 Substituir os caminhos no ~/.kube/config pelos dados base64 acima:**

- certificate-authority → certificate-authority-data

- client-certificate → client-certificate-data

- client-key → client-key-data

**4.3 Salvar como arquivo de sistema (não .txt) com o nome kubeconfig usando o git bash:**

```bash
vi kubeconfig
# cole o conteúdo, depois pressione :wq!
```

### 🔐 5. Adicionar kubeconfig como credencial no Jenkins

Vá em **Manage Jenkins** > **Credentials** > **Global** > **Add Credentials**

Tipo: **Secret file**

Faça upload do arquivo **kubeconfig**

Dê um ID: **kubeconfig**

### 🐳 6. Instalar Argo CD e kubectl dentro do container do Jenkins

No `Jenkinsfile`, adicione uma stage:

```bash
stage('Install kubectl and Argo CD CLI') {
    steps {
        sh '''
        curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
        chmod +x ./kubectl
        mv ./kubectl /usr/local/bin/kubectl

        curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
        chmod +x argocd
        mv argocd /usr/local/bin/
        '''
    }
}
```
### 🔁 7. Conectar Jenkins ao Argo CD e sincronizar

7.1 Gerar pipeline syntax no Jenkins com withKubeConfig:

    - Use o endpoint do cluster (via kubectl cluster-info)

    - Selecione a credencial kubeconfig

7.2 Script no Jenkinsfile:

```bash
stage('Sync with Argo CD') {
    steps {
        script {
            withKubeConfig([credentialsId: 'kubeconfig', serverUrl: 'https://<your-k8s-ip>:<port>']) {
                sh '''
                argocd login <ARGOCD-IP>:<PORT> --username admin --password $(kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d) --insecure
                argocd app sync gitops-app
                '''
            }
        }
    }
}

```

### ✅ 8. Pronto!