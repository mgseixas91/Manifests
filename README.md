# Manifests Kubernetes - Aplicações App1 e App2

Este repositório contém os manifests Kubernetes das aplicações `app1` e `app2`. Toda a estrutura está organizada em diretórios separados para cada aplicação.

### Estrutura de Diretórios

```
manifests/
└── k8s/
    ├── app1/
    │   ├── namespace.yaml
    │   ├── ingress.yaml
    │   ├── deployment.yaml
    │   └── service.yaml
    └── app2/
        ├── ingress.yaml
        ├── deployment.yaml
        └── service.yaml
```
Para esse cenário atual com aplicações básicas usar Deployment puro é mais direto e mais leve e atende perfeitamente, por esse motivo não foi utilizado helm.
---

## Organização e Separação dos Manifests

Os manifests estão **separados do repositório das aplicações** por uma questão de boas práticas. Essa abordagem traz os seguintes benefícios:

- Permite **manutenção independente** dos artefatos Kubernetes sem acionar uma pipeline de build ou gerar novas tags da imagem.
- Evita misturar mudanças de infraestrutura com código de aplicação.
- Facilita a adoção de estratégias **GitOps**, como o uso do ArgoCD, para entrega contínua baseada em controle de versão.

---

## app1

### namespace.yaml

Cria o namespace `poc-sensedia`. Este arquivo está presente **somente no app1**, pois o mesmo namespace será reutilizado pelo `app2`.

### ingress.yaml

Responsável por expor a aplicação externamente via Load Balancer. Esse manifest é considerado **base**, pois define a rota principal que será usada para acessar as aplicações.

- Uso de `ingressClassName: nginx-poc-sensedia` para direcionar o tráfego ao **Ingress Controller** correto, instalado via Helm.
- Define regras para redirecionar as requisições ao `app1` por padrão.
- O `pathType: Prefix` garante roteamento correto mesmo com subcaminhos.

---

### deployment.yaml

Define o deployment da aplicação `app1`.

- Imagem usada: `577125335739.dkr.ecr.us-east-1.amazonaws.com/app1:latest`
- Porta exposta: `5000`
- Configuração de recursos:
  - **requests**: define o mínimo necessário de recursos para agendamento.
  - **limits**: evita que a aplicação consuma recursos excessivos.

Essa configuração é adequada para aplicações simples em ambientes de POC.

---

### service.yaml

Define um `Service` do tipo **ClusterIP** para expor a aplicação internamente no cluster. Isso é suficiente pois o tráfego externo será roteado via o `Ingress`.

---

## app2

A estrutura é semelhante à do `app1`, mas com um diferencial: **estratégia canário**.

### Canary Deployment

No `Ingress` do `app2`, foi adicionada uma regra com base em **HTTP headers** para que a aplicação seja acessada apenas com um cabeçalho específico:

```yaml
nginx.ingress.kubernetes.io/canary: "true"
nginx.ingress.kubernetes.io/canary-by-header: "app2"
nginx.ingress.kubernetes.io/canary-by-header-value: "true"
```

### Como funciona?

- Por padrão, a URL `/` redireciona para o `app1`.
- Para acessar o `app2`, é necessário enviar um cabeçalho:

```bash
curl -H "app2: true" http://34.230.150.251
```

### Por que não funciona via navegador?

Browsers não permitem adicionar headers personalizados facilmente. Por isso, `app2` **só responde via curl** (ou ferramentas que permitam enviar headers HTTP).
---

Caso fosse necessário acessar **ambas as aplicações via web**, a recomendação seria:
- Criar **subpaths** no `Ingress`, como `/app1` e `/app2`, ou
- Utilizar subdomínios, como `app1.suaempresa.com` e `app2.suaempresa.com`.

---

