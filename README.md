# Overview
Este é o repositório que contém os manifestos  para subir as aplicações no Cluster Kubernetes.

# Setup
```bash
$ cd ./production && kubectl apply -k .
```

# Pré Requisitos
1. Imagens docker precisam já estar nos repositórios do ECR.

```bash
$ cd <YOUR_FOLDER>/not-so-simple-ecommerce

# Order
$ docker build -f src/services/NotSoSimpleEcommerce.Order/Dockerfile -t 968225077300.dkr.ecr.us-east-1.amazonaws.com/nsse/production/order .
$ docker push 968225077300.dkr.ecr.us-east-1.amazonaws.com/nsse/production/order

# Main
$ docker build -f src/services/NotSoSimpleEcommerce.Main/Dockerfile -t 968225077300.dkr.ecr.us-east-1.amazonaws.com/nsse/production/main .
$ docker push 968225077300.dkr.ecr.us-east-1.amazonaws.com/nsse/production/main

# Identity Server
$ docker build -f src/services/NotSoSimpleEcommerce.IdentityServer/Dockerfile -t 968225077300.dkr.ecr.us-east-1.amazonaws.com/nsse/production/identity-server .
$ docker push 968225077300.dkr.ecr.us-east-1.amazonaws.com/nsse/production/identity-server

# Health Checker
$ docker build -f src/services/NotSoSimpleEcommerce.HealthChecker/Dockerfile -t 968225077300.dkr.ecr.us-east-1.amazonaws.com/nsse/production/health-checker .
$ docker push 968225077300.dkr.ecr.us-east-1.amazonaws.com/nsse/production/health-checker

# Notificator
$ docker build -f src/workers/NotSoSimpleEcommerce. -t 968225077300.dkr.ecr.us-east-1.amazonaws.com/nsse/production/notificator .
$ docker push 968225077300.dkr.ecr.us-east-1.amazonaws.com/nsse/production/notificator

# Invoice Generator
$ docker build -f src/workers/NotSoSimpleEcommerce.InvoiceGenerator/Dockerfile -t 968225077300.dkr.ecr.us-east-1.amazonaws.com/nsse/production/invoice-generator .
$ docker push 968225077300.dkr.ecr.us-east-1.amazonaws.com/nsse/production/invoice-generator
```
2. Seu secret precisa estar atualizado, para que os PODs consigam baixar as imagens do ECR.

```bash

$ cd cd <YOUR_FOLDER>/not-so-simple-ecommerce-gitops
$ kubectl create secret docker-registry ecr-image-pull-credentials --docker-username=AWS --docker-password=$(aws ecr get-login-password --region us-east-1) --docker-server=968225077300.dkr.ecr.us-east-1.amazonaws.com --dry-run=client -o yaml > production/secrets/ecr-image-pull-credentials.yml
```

3. Para que o  `kubectl apply -k` funcione da sua máquina local, você precisa copiar o arquivo `/etc/kubernetes/admin.conf` que está em qualquer nó master para dentro da sua máquina, subtituir o DNS do NLB por `127.0.0.1`, adicionar o apontamento do endereço 127.0.0.1 para o DNS do NLB no arquivo hosts da sua máquina e abrir corretamente um túnel com o nó master na porta 6443. 

```bash
$ aws ssm start-session --target <ANY_MASTER_INSTANCE_ID> --document-name AWS-StartPortForwardingSession  --parameters 'portNumber=6443,localPortNumber=6443'

$ export KUBECONFIG=/etc/kubernetes/admin.conf

$ kubectl get nodes
```

Observação: Se precisar revisar esse processo na prática, confira novamente aula `Aula 33-Acesso Local e Port Forwarding` do módulo 06.