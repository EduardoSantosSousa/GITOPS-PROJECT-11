## **🚀 Integração GitHub + Jenkins (via VM na GCP)**

### 🔧 1. Gerar Personal Access Token (PAT) no GitHub

- Vá em: **GitHub** > **Perfil** > **Settings** > **Developer Settings** > **Personal Access Tokens**

- Crie um token do tipo **Classic**

- Dê um nome (ex: `GitHubToken`) e marque as permissões:

    - `repo`

    - `workflow`

    - `admin:org`

    - `admin:public_key`

    - `admin:repo_hook`

    - `admin:org_hook`

- Clique em **Generate token**

- ❗ **Não feche essa aba** — você vai precisar do token depois


## **🔐 2. Adicionar token no Jenkins (Credenciais)**

- Acesse o **Jenkins Dashboard**

- Vá em: **Manage Jenkins** > **Credentials** > **Global** > **Add Credentials**

- Preencha:

    - **Username:** seu usuário do GitHub

    - **Password:** cole o token gerado

    - **ID (opcional)**: `github-token`

    - **Description**: pode ser o mesmo do ID

- Salve

## **🛠️ 3. Criar pipeline no Jenkins**

- No Jenkins Dashboard:

    - Clique em New Item

    - Dê um nome (ex: `gitops-pipeline`)

    - Selecione Pipeline

    - Clique em OK


## **🔗 4. Configurar repositório Git no pipeline**

- Na aba do pipeline criado:

    - Vá até a seção **Pipeline**

    - Em Definition, selecione: `Pipeline script from SCM`

    - Em SCM, selecione: `Git`

    - Em **Repository URL**, cole a URL do seu repositório GitHub

    - Em **Credentials**, selecione o token salvo

    - Em **Branch**, coloque: `main` (ou `master`, se for o seu caso)

    - Caminho do Jenkinsfile: mantenha como `Jenkinsfile`

    - Clique em **Save**

## **📝 5. Criar o Jenkinsfile na VM da GCP**

- Conectado à sua VM (via SSH):

```bash
nano Jenkinsfile
```

- Cole seu pipeline script (exemplo básico):

```bash
pipeline {
    agent any
    stages {
        stage('Checkout Github') {
            steps {
                echo 'Checking out code from GitHub...'
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: 'github-token',
                        url: 'https://github.com/seu-usuario/repositorio.git'
                    ]]
                )
            }
        }
        // Outros stages...
    }
}
```
- Salve com: `Ctrl + O`, depois `Enter` e saia com `Ctrl + X`

## **🔧 6. Configurar Git na VM**

```bash
git config --global user.email "seuemail@exemplo.com"
git config --global user.name "Seu Nome"
```

## **📤 7. Enviar Jenkinsfile para o GitHub**

```bash
git add .
git commit -m "Adiciona Jenkinsfile"
git push origin main
```

Será solicitado seu usuário e token como senha (cole o token como senha no terminal).

## **🚀 9. Executar o pipeline**

- Volte ao Jenkins Dashboard

- Abra seu pipeline (`gitops-pipeline`)

- Clique em **Build Now**

- Veja o console: se tudo estiver correto, você verá um ✅ **build com sucesso**

## **📁 10. Verificar arquivos clonados**

- No Jenkins, vá em:

    - Pipeline > Build > Workspace

    - Confirme se os arquivos do repositório foram copiados

## **✅ Resultado final**

- Você concluiu com sucesso:

    - Geração de token

    - Configuração de credencial

    - Pipeline com `Jenkinsfile` versionado

    - Integração Jenkins + GitHub
    
    - Build de pipeline rodando na VM