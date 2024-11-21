- [Inputs](#inputs)
- [Setup](#setup)
  - [Deploy Key](#deploy-key)
    - [GitHub](#github)
    - [Azure DevOps](#azure-devops)
    - [GitLab](#gitlab)
  - [Devbox](#devbox)
    - [Instalar Devbox](#instalar-devbox)
    - [Baixar repositório](#baixar-repositório)
- [Iniciar sessão](#iniciar-sessão)
  - [Kubeconfig](#kubeconfig)
  - [Grafana](#grafana)
  - [Argo CD](#argo-cd)
    - [Webhook](#webhook)
  - [Kubeseal](#kubeseal)
    - [Charts](#charts)
- [Operação](#operação)
  - [Criar novo blueprint](#criar-novo-blueprint)
    - [Catálogo de planos](#catálogo-de-planos)
    - [Recomendações para produção](#recomendações-para-produção)
    - [Segredos via kubeseal](#segredos-via-kubeseal)
  - [Criar novo chart](#criar-novo-chart)
    - [Values embutidos](#values-embutidos)
  - [Grafana](#grafana-1)
  - [Slack](#slack)
  - [Horários de backups e reboots](#horários-de-backups-e-reboots)

## Inputs
Preencha o form em https://forms.gle/kDXiZ9zpygpkcPbTA
## Setup
### Deploy Key
No repositório que você criou para _blueprints_ e _charts_, adicione o _deploy key_ informado da seguinte forma:
#### GitHub
- Clique em _Settings_, _Deploy keys_, _Add deploy key_
- Em _Title_ e _Key_ coloque os dados recebidos
- Manter desativado _Allow write access_ e adicionar.
#### Azure DevOps
- Clique no ícone _User settings_, _SSH public keys_
- Em _Description_ e _Key data_ coloque os dados recebidos
#### GitLab
- Acesse _Settings_, _Repository_
- Em _Deploy keys_, selecione _Add new key_
- Complete os campos, mantendo desativado _Grant write permissions..._
### Devbox
Para operar o _cluster_  uma série de ferramentas de linha de comando precisa ser instalada. O _Devbox_ permite uma instalação rápida e sem intereferir no funcionamento dos demais programas do seu computador. Funciona em _Windows (WSL)_, _Mac OS_ e _Linux_.

#### Instalar Devbox
```bash
curl -fsSL https://get.jetify.com/devbox | bash
```

#### Baixar repositório
Crie um _fork_ do respositório https://github.com/lwsa-idp/demo-workloads.git onde você definirá seus _workloads_ baseados no exemplo.
```bash
git clone <seu repo>
```

## Iniciar sessão

Para iniciar uma sessão, mude para o diretório do repositório, onde se encontra o arquivo _devbox.json_, e carregar o ambiente com:

```bash
devbox shell
```

### Kubeconfig
### Grafana
### Argo CD
#### Webhook
O _Argo CD_ sincroniza automaticamente _charts_ e _blueprints_ a cada 3 minutos, ou manualmente via botão _Refresh_ dentro de cada _Application_.

Opcionalmente, você pode configurar um _webhook_ de forma que, a cada _push_ no repositório, é disparada uma sincronização. Para isto, crie o _webhook_ informado, lembrando de alterar _Content type_ para `application/json`.

Mais informações: https://argo-cd.readthedocs.io/en/stable/operator-manual/webhook/
### Kubeseal
Usamos a ferramenta _Sealed Secrets_ para permitir a publicação de segredos de forma segura nos repositórios de _blueprints_ e _charts_.

Você receberá um arquivo `cert.pem` contendo um certificado para criptogarfar e publicar seus segredos. Para isto:
- Salve o certificado recebido em `~/cert.pem`.
- Digitar: `(secret=$(read -s; echo $REPLY); echo -n "$secret" | kubeseal --raw --name <nome do segredo> --namespace <namespace> --cert ~/cert.pem)`

Por exemplo:
```
(secret=$(read -s; echo $REPLY); echo -n "$secret" | kubeseal --raw --name segredo --namespace portal --cert ~/cert.pem)
```
E preencher o segredo a ser criptografado. O valor obtido (sem o `%` final, que só demarca o fim da string), deve ser copiado para `blueprint.yaml`.

Dessa forma o segredo é cadastrado criptografado no _blueprint_ conforme exemplo:

```yaml
secrets:
  - name: segredo
    namespace: portal
    data:
      chave: <valor criptografado acima>
```

__Nota__: Como o _namespace_ faz parte do valor criptografado, é necessário incluí-lo no _blueprint_ para que fique explícito. A aplicação falhará se o _blueprint_ for carregado num _namespace_ diferente daquele para o qual o segredo foi destinado. Esta é uma medida de segurança para evitar que o segredo possa ser descriptografado num _namespace_ diferente. Note também que o nome fornecido no parâmetro `--name` precisa ser igual à da entrada `name` do _blueprint_.

#### Charts

Digitar: `(secret=$(read -s; echo $REPLY); echo -n "$secret" | kubectl create secret generic mysecret --namespace=giba --dry-run=client --from-file=foo=/dev/stdin -o yaml | kubeseal --cert ~/cert.pem)`

(substitua `giba`, `mysecret` e `foo` pelo _namespace_, segredo e chave respectivos)

## Operação
### Criar novo blueprint
#### Catálogo de planos
#### Recomendações para produção
Replicas
#### Segredos via kubeseal
### Criar novo chart
#### Values embutidos
### Grafana
- cluster
- namespaces (sizing)
- CVEs
- Network policies
### Slack
Alarmes que podem ser ignorados
### Horários de backups e reboots
