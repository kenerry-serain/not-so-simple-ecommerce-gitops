# Not So Simple Ecommerce GitOps

Este é o repositório utilizado dentro do curso para gerenciar todos os manifestos
da aplicação `not-so-simple-ecommerce`.

Este repositório utiliza helm charts para empacotar os microserviços e kustomize
para gerenciar os arquivos de infraestrutura e compartilhados entre todos os charts.

---

## 🛠️ Configuração e Execução

### 1. Build e Push das Imagens Docker

Faça o build das imagens docker e envie para seus respectivos repositórios no ECR - se já não o fez.
Lembre-se que os Dockerfiles não estão situados neste repositório, mas no repositório not-so-simple-ecommerce.

📌 **Repositório:** [not-so-simple-ecommerce](https://github.com/kenerry-serain/not-so-simple-ecommerce)

```bash
cd ./not-so-simple-ecommerce

# Order
docker build -f src/services/NotSoSimpleEcommerce.Order/Dockerfile -t <YOUR_ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com/nsse/production/order .
docker push <YOUR_ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com/nsse/production/order

# Main
docker build -f src/services/NotSoSimpleEcommerce.Main/Dockerfile -t <YOUR_ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com/nsse/production/main .
docker push <YOUR_ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com/nsse/production/main

# Identity Server
docker build -f src/services/NotSoSimpleEcommerce.IdentityServer/Dockerfile -t <YOUR_ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com/nsse/production/identity-server .
docker push <YOUR_ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com/nsse/production/identity-server

# Health Checker
docker build -f src/services/NotSoSimpleEcommerce.HealthChecker/Dockerfile -t <YOUR_ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com/nsse/production/health-checker .
docker push <YOUR_ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com/nsse/production/health-checker

# Notificator
docker build -f src/workers/NotSoSimpleEcommerce. -t <YOUR_ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com/nsse/production/notificator .
docker push <YOUR_ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com/nsse/production/notificator

# Invoice Generator
docker build -f src/workers/NotSoSimpleEcommerce.InvoiceGenerator/Dockerfile -t <YOUR_ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com/nsse/production/invoice-generator .
docker push <YOUR_ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com/nsse/production/invoice-generator
```

**Atenção:** Substitua `<YOUR_ACCOUNT>` pela sua conta AWS.

---

### 2. Atualização do Image Pull Secret

Atualize o `image pull secret` do ECR, para que os PODs consigam baixar as imagens dos repositórios privados do ECR
de dentro das instâncias do Cluster.

```bash
cd ./not-so-simple-ecommerce-gitops
kubectl create secret docker-registry ecr-image-pull-credentials \
    --docker-username=AWS \
    --docker-password=$(aws ecr get-login-password --region us-east-1) \
    --docker-server=<YOUR_ACCOUNT>.dkr.ecr.us-east-1.amazonaws.com --dry-run=client \
    -o yaml > production/secrets/ecr-image-pull-credentials.yml
```

**Atenção:** Substitua `<YOUR_ACCOUNT>` pela sua conta AWS.

---

### 3. Configuração Kube Config

```bash
aws ssm start-session --target <ANY_MASTER_INSTANCE_ID>
sudo su
cat /etc/kubernetes/admin.conf
```

Copie o resultado do cat, para o arquivo /etc/kubernetes/admin.conf na sua máquina local e lembre-se
de substituir o DNS do NLB por 127.0.0.1 e também adicionar o apontamento do endereço 127.0.0.1 para o 
DNS do NLB no arquivo hosts da sua máquina.

---

### 4. Teste da Conexão com o Cluster Kubernetes

Para executar os manifestos deste repositório no Cluster Kubernetes a partir da sua máquina local, 
primeiramente é necessário abrir um túnel com algum nó master mapeando localmente o kube-apiserver que estará 
rodando na porta 6443 do nó localmente na mesma porta. Edite o arquivo `/etc/kubernetes/admin.conf` do passo anterior
na sua máquina, substituindo o DNS do NLB por `127.0.0.1` e adicione o apontamento do endereço 127.0.0.1 para o 
DNS do NLB no arquivo hosts da sua máquina e só então, abra o túnel. 

```bash
aws ssm start-session \
    --target <ANY_MASTER_INSTANCE_ID> \
    --document-name AWS-StartPortForwardingSession \
    --parameters 'portNumber=6443,localPortNumber=6443'
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl get nodes
```

📌 **Observação:** Se precisar revisar o processo, consulte a aula `Aula 33-Acesso Local e Port Forwarding` do módulo 06.

---

### 5. Aplicação dos Manifestos no Cluster Kubernetes

Execute o comando abaixo para aplicar os manifestos no Cluster Kubernetes:

```bash
kustomize build --enable-helm | kubectl apply -f -
```
