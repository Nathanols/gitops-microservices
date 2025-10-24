# 🚀 Projeto GitOps com ArgoCD e Minikube (Compass Uol)

Este projeto demonstra uma implementação prática de **GitOps**, utilizando:

- **Minikube** como cluster Kubernetes local.
- **ArgoCD** para automação de deploys.
- **GitHub** como fonte de verdade (repositório público com manifests YAML).
- **Online Boutique** como aplicação de exemplo.

---

## 📚 Objetivo

Executar um conjunto de microserviços (Online Boutique) em Kubernetes local usando Minikube, controlado por GitOps com ArgoCD, a partir de um repositório público no GitHub.  

Ao final, o frontend da aplicação estará acessível via `kubectl port-forward`.

---

## 🛠 Pré-requisitos

- Docker instalado e funcionando.
- WSL (Ubuntu) ou Linux/macOS.
- Git instalado.
- Conta no GitHub.
- Navegador web.

---

## Estrutura do repositório

gitops-microservices/  
  └── k8s/  
  └── online-boutique.yaml


- `online-boutique.yaml` → Contém todos os manifests Kubernetes do projeto.

---

## 📝 Passo a passo detalhado

### 1️ - Criar o repositório GitHub

1. Fork do repositório oficial:  
   [GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo)
2. Criar repositório público no GitHub: `gitops-microservices`.
3. Comandos para clonar, copiar e enviar para GitHub:

```bash
git clone https://github.com/SEU_USUARIO/gitops-microservices.git
cd gitops-microservices
mkdir -p k8s
cp /caminho/do/fork/microservices-demo/release/kubernetes-manifests.yaml k8s/online-boutique.yaml
git add .
git commit -m "Adicionar manifest Kubernetes Online Boutique"
git push origin main
```

### 2 - Criar cluster Kubernetes local com Minikube

Verifique Docker:
```bash
docker --version
```

Inicie Minikube (ajuste memória conforme seu sistema):
```bash
minikube start --driver=docker --nodes=1 --memory=2048 --cpus=2
```

Verifique status do cluster:
```bash
minikube status
kubectl get nodes
```

### 3 - Instalar ArgoCD no cluster

Criar namespace:
```bash
kubectl create namespace argocd
```

Aplicar manifests do ArgoCD:
```bash
kubectl apply -n argocd -f install.yaml
```

⚠️ Se houver erro 429 ao baixar do GitHub, baixe o arquivo manualmente e aplique localmente.

Verificar pods:
```bash
kubectl get pods -n argocd
```

<img width="1111" height="293" alt="Image" src="https://github.com/user-attachments/assets/567f1c3c-b5c7-4c0a-832c-58b056bfb311" />

Port-forward para acessar Web UI (navegador):
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Web UI (navegador): https://localhost:8080

Usuário: admin  
Senha inicial: XXXXXXXXX
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

### 4️ - Instalar ArgoCD CLI (WSL/Linux)

Verificar arquitetura:
```bash
uname -m
```

- x86_64 → argocd-linux-amd64
- aarch64 → argocd-linux-arm64

Baixar, tornar executável e mover para /usr/local/bin:
```bash
cd ~
curl -LO https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd-linux-amd64
sudo mv argocd-linux-amd64 /usr/local/bin/argocd
argocd version
```

<img width="686" height="576" alt="Image" src="https://github.com/user-attachments/assets/bf4aa451-ea10-47b2-9b06-d1593f6133c9" />


### 5️ - Criar e sincronizar aplicação GitOps

Login no ArgoCD CLI:
```bash
argocd login localhost:8080 \
  --username admin \
  --password $(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d) \
  --insecure
```

Criar aplicação apontando para GitHub:
```bash
argocd app create online-boutique \
  --repo https://github.com/SEU_USUARIO/gitops-microservices.git \
  --path k8s \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

Sincronizar aplicação:
```bash
argocd app sync online-boutique
kubectl get pods
```

<img width="954" height="503" alt="Image" src="https://github.com/user-attachments/assets/5259f3dc-826d-4451-8f2f-2a63d4e92f50" />

<img width="1910" height="1027" alt="Image" src="https://github.com/user-attachments/assets/730d9f84-5628-4149-802b-e7e6ccf71e07" />

### 6️ - Acessar frontend da aplicação

Port-forward do frontend (evitando conflito de portas):
```bash
kubectl port-forward svc/frontend 8081:80
```

Acesse no navegador:
```bash
http://localhost:8081
```

<img width="1906" height="1032" alt="Image" src="https://github.com/user-attachments/assets/5a61efae-2b20-46e9-8070-2435672e158b" />

### ✅ Resultado Final

- Repositório GitHub com manifests YAML.
- ArgoCD rodando no Minikube.
- Aplicação online-boutique criada e sincronizada via GitOps.
- Frontend acessível via port-forward.

### 📝 Observações

- Cada alteração no repositório Git (git push) é automaticamente refletida no ArgoCD ao sincronizar.
- Pode-se customizar manifests YAML (ex: alterar réplicas de pods).
- Portas do localhost podem ser ajustadas para evitar conflitos.
