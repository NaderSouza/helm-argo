# Helm + ArgoCD Demo

Este repositório reúne um exemplo completo de GitOps para publicar uma aplicação estática no Kubernetes utilizando três abordagens diferentes: manifests YAML "puros", um chart Helm e (como referência) uma estrutura Kustomize. Ele foi organizado para ser usado com o Argo CD, exibindo como a ferramenta pode acompanhar cada fonte de configuração.

## Visão geral

- **app/**: código-fonte de uma página estática (HTML/CSS) com Dockerfile baseado em Nginx que pode ser empacotada e enviada para um registro de imagens.
- **gitops/**: manifests Kubernetes tradicionais (`Deployment` e `Service`) que publicam a imagem `naderhs/page-nhs-kube:latest` no namespace `nhs-demo`.
- **helm/**: chart Helm minimalista (`nginx-chart`) parametrizando um deployment Nginx com ênfase em disponibilidade (réplicas, probes, etc.).
- **argocd/**: definições de `Application` do Argo CD que apontam para cada estratégia (manifests, Helm e Kustomize), habilitando sincronização automática, _pruning_ e criação de namespace.
- **k8s/**: configuração de cluster Kind com um nó de controle e um nó worker, pronta para executar workloads e ingressos locais.
- **kustomize/**: exemplo de overlays para ambientes dev/prod usando Kustomize.
- **monitoramento:**
  - **Prometheus, Grafana e OpenTelemetry Collector** instalados via Helm e gerenciados pelo ArgoCD, monitorando o cluster e aplicações.

## O que foi feito recentemente

- Adição dos manifests do ArgoCD para Prometheus, Grafana e OpenTelemetry Collector em `argocd/`.
- Configuração do Prometheus e Grafana para monitorar o cluster local e integrar com o ArgoCD.
- Configuração do OpenTelemetry Collector para coletar métricas do Kubernetes e exportar para o Prometheus.
- Ajuste dos manifests para uso correto das versões dos charts e sintaxe YAML.
- Passo a passo para acesso aos dashboards e verificação das métricas do cluster.

## Preparando o ambiente local

1. **Criar o cluster Kind (opcional, testes locais)**

   ```bash
   kind create cluster --config k8s/kind-config.yaml
   ```

2. **Construir e publicar a imagem da aplicação**

   ```bash
   cd app
   docker build -t <seu-registro>/perfil-static:latest .
   docker push <seu-registro>/perfil-static:latest
   ```

3. **Atualizar a referência da imagem**
   - `gitops/deployment.yaml`: ajustar `image:` para apontar para a imagem publicada.
   - `helm/nginx-chart/values.yaml`: alterar `image.repository` e `image.tag` conforme necessário.

## Instalando o ArgoCD no Cluster

```
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
kubectl create namespace argocd
helm install argocd argo/argo-cd --namespace argocd
```

## Registrando aplicações no Argo CD

```bash
kubectl apply -f argocd/basic-application.yaml
kubectl apply -f helm/nginx-chart/helm-application.yaml
kubectl apply -f argocd/kustomize-application.yaml
kubectl apply -f prometheus/prometheus-application.yaml
kubectl apply -f grafana/grafana-application.yaml
kubectl apply -f otel/opentelemetry-application.yaml
```

Cada manifest cria uma instância de `Application`:

- `basic-application.yaml`: sincroniza a pasta `gitops/`, entregando os manifests estáticos no namespace `nhs-demo`.
- `helm-application.yaml`: renderiza o chart de `helm/` no namespace `nhs-helm-demo`, usando `values.yaml` como arquivo de valores.
- `kustomize-application.yaml`: referencia uma base Kustomize em `kustomize/overlays/dev` (adicione os arquivos correspondentes se desejar utilizar este fluxo).
- `prometheus-application.yaml`: instala o Prometheus no namespace `monitoring` via Helm.
- `grafana-application.yaml`: instala o Grafana no namespace `monitoring` via Helm, já integrado ao Prometheus.
- `opentelemetry-application.yaml`: instala o OpenTelemetry Collector para coletar métricas do cluster e enviar ao Prometheus.

Todos os manifests habilitam `automated.prune` e `automated.selfHeal`, permitindo que o Argo CD converja automaticamente para o estado definido no Git.

## Como acessar os dashboards de monitoramento

1. Descubra as portas NodePort dos serviços:
   ```bash
   kubectl get svc -n monitoring
   ```
2. Acesse via browser:
   - Prometheus: `kubectl port-forward svc/prometheus-server -n monitoring 9090:80`
   - Grafana: `kubectl port-forward svc/grafana -n monitoring 3000:80`
   - OpenTelemetry Collector: normalmente não tem UI, mas pode expor métricas em `/metrics`.
3. No Grafana, importe dashboards prontos para Kubernetes (exemplo: ID 6417).

## Próximos passos

- Criar um pipeline CI/CD que construa e publique a imagem de `app/` e atualize os manifests.
- Adicionar diretórios Kustomize (`kustomize/base` e `kustomize/overlays`) para completar o exemplo de Kustomize.
- Configurar testes automatizados (ex.: lint de Helm ou validação de manifests) executados no pipeline.

## Requisitos

- Docker ou outra ferramenta compatível para build de containers.
- Kubectl, Kind e Helm instalados localmente para testes.
- Um cluster Kubernetes com Argo CD quando for realizar o fluxo GitOps completo.
