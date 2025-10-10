# Helm + ArgoCD Demo

Este repositório reúne um exemplo completo de GitOps para publicar uma aplicação estática no Kubernetes utilizando três abordagens diferentes: manifests YAML "puros", um chart Helm e (como referência) uma estrutura Kustomize. Ele foi organizado para ser usado com o Argo CD, exibindo como a ferramenta pode acompanhar cada fonte de configuração.

## Visão geral

- **app/**: código-fonte de uma página estática (HTML/CSS) com Dockerfile baseado em Nginx que pode ser empacotada e enviada para um registro de imagens.
- **gitops/**: manifests Kubernetes tradicionais (`Deployment` e `Service`) que publicam a imagem `naderhs/page-nhs-kube:latest` no namespace `nhs-demo`.
- **helm/**: chart Helm minimalista (`nginx-chart`) parametrizando um deployment Nginx com ênfase em disponibilidade (réplicas, probes, etc.).
- **argocd/**: definições de `Application` do Argo CD que apontam para cada estratégia (manifests, Helm e Kustomize), habilitando sincronização automática, _pruning_ e criação de namespace.
- **k8s/**: configuração de cluster Kind com um nó de controle e um nó worker, pronta para executar workloads e ingressos locais.

O fluxo esperado é:

1. Desenvolver ou ajustar a aplicação estática em `app/` e gerar uma imagem container.
2. Publicar essa imagem em um registro e referenciá-la nos manifests ou valores Helm.
3. Registrar o repositório no Argo CD utilizando um dos manifests da pasta `argocd/`.
4. O Argo CD criará o namespace de destino, aplicará os manifests e manterá o estado desejado através de sincronização automática.

## Preparando o ambiente local

1. **Criar o cluster KinD (opcional, para testes locais)**

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

## Registrando aplicações no Argo CD

Com o Argo CD já instalado no cluster, aplique um dos manifests a seguir:

```bash
kubectl apply -f argocd/basic-application.yaml
kubectl apply -f argocd/helm-application.yaml
kubectl apply -f argocd/kustomize-application.yaml
```

Cada manifest cria uma instância de `Application`:

- `basic-application.yaml`: sincroniza a pasta `gitops/`, entregando os manifests estáticos no namespace `nhs-demo`.
- `helm-application.yaml`: renderiza o chart de `helm/` no namespace `nhs-helm-demo`, usando `values.yaml` como arquivo de valores.
- `kustomize-application.yaml`: referencia uma base Kustomize em `kustomize/overlays/dev` (adicione os arquivos correspondentes se desejar utilizar este fluxo).

Todos os manifests habilitam `automated.prune` e `automated.selfHeal`, permitindo que o Argo CD converja automaticamente para o estado definido no Git.

## Próximos passos sugeridos

- Criar um pipeline CI/CD que construa e publique a imagem de `app/` e atualize os manifests.
- Adicionar diretórios Kustomize (`kustomize/base` e `kustomize/overlays`) para completar o exemplo de Kustomize.
- Configurar testes automatizados (ex.: lint de Helm ou validação de manifests) executados no pipeline.

## Requisitos

- Docker ou outra ferramenta compatível para build de containers.
- Kubectl, Kind e Helm instalados localmente para testes.
- Um cluster Kubernetes com Argo CD quando for realizar o fluxo GitOps completo.

## Licença

Este projeto não define uma licença explícita; ajuste conforme a necessidade do seu ambiente.
