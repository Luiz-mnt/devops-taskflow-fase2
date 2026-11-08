# Configuração e segurança do laboratório

Esta documentação descreve a configuração implementada, os controles dos
containers e os gates do pipeline. Os resultados de segurança locais são
medições do momento do scan; não representam ausência permanente de riscos.

## Variáveis e responsabilidade

| Variável | Padrão ou regra | Uso |
|---|---|---|
| `IMAGE_REF` | Obrigatória; tag explícita ou `imagem@sha256:...` válido | Imagem da API; no CD, utiliza o digest publicado |
| `IMAGE_PULL_POLICY` | `always`; aceita `always` ou `never` | Script faz pull no modo normal; `never` exige todas as imagens já disponíveis localmente |
| `TASKFLOW_RESOURCE_PREFIX` | `taskflow`, até 40 caracteres | Prefixo dos nomes da rede/volume externos e nome padrão do projeto Compose |
| `TF_VAR_resource_prefix` | `taskflow` | Variável Terraform; deve corresponder ao prefixo consumido pelo Compose |
| `TF_VAR_docker_host` | `null` no Terraform; no WSL usa `unix:///var/run/docker.sock` | Endpoint do provider Docker |
| `DOCKER_HOST` | Socket Docker do ambiente | Endpoint utilizado pela CLI Docker |
| `COMPOSE_PROJECT_NAME` | Opcional; sobrescreve o nome padrão do projeto | Isola nomes de containers, sem alterar os nomes dos recursos externos |
| `API_PORT` | `8000` | Porta HTTP local da API, sempre publicada em 127.0.0.1 pelo Compose |
| `PROMETHEUS_PORT` | `9090` | Porta HTTP local do Prometheus, também em 127.0.0.1 |
| `TASKFLOW_DB_PATH` | `data/taskflow.db` fora do container | Banco da API; Compose fixa `/data/taskflow.db` no volume persistente |
| `DEPLOY_TIMEOUT` | `120`, inteiro de 1 a 600 segundos | Prazo do `compose up --wait` |
| `SMOKE_TIMEOUT` | `90`, inteiro de 1 a 600 segundos | Prazo de cada espera do smoke |
| `API_URL` | `http://127.0.0.1:${API_PORT}` | Endereço da API usado pelo smoke |
| `PROMETHEUS_URL` | `http://127.0.0.1:${PROMETHEUS_PORT}` | Endereço do monitoramento usado pelo smoke |

O prefixo aceita letras minúsculas, números e hífens internos, começando por
letra. Os nomes externos são `${TASKFLOW_RESOURCE_PREFIX}-net` e
`${TASKFLOW_RESOURCE_PREFIX}-data`. Terraform é seu único proprietário; Compose
os declara `external: true`. Não existem declarações concorrentes de criação
desses recursos no Compose.

O arquivo [`.env.example`](../.env.example) contém exemplos sem segredos.
Compose lê `.env` para interpolação, mas os scripts Bash leem as variáveis do
processo. Exporte explicitamente as variáveis do deploy/smoke no shell. A API
executada diretamente em venv também não carrega `.env` automaticamente.

Exemplo de configuração do laboratório existente, no Bash da distribuição WSL:

```bash
export DOCKER_HOST=unix:///var/run/docker.sock
export TF_VAR_docker_host=unix:///var/run/docker.sock
export TF_VAR_resource_prefix=taskflow
export TASKFLOW_RESOURCE_PREFIX=taskflow
export API_PORT=8000
export PROMETHEUS_PORT=9090
terraform -chdir=infra plan -detailed-exitcode
```

O estado local da fase 1 foi copiado explicitamente para a pasta de trabalho da
fase 2 para manter a propriedade dos recursos existentes. A comparação retornou
exit 0, sem mudanças. **O estado ativo é o da fase 2**: não execute applies
simultâneos das duas pastas para a mesma rede/volume. Estado, planos, `.env` e
bancos não são publicados no Git.

## Containers e dados

| Controle | API | Prometheus |
|---|---|---|
| Usuário | UID/GID `10001:10001`, sem root | UID/GID `65534:65534`, sem root |
| Sistema de arquivos raiz | Somente leitura | Somente leitura |
| Escrita necessária | Volume externo em `/data`; `/tmp` em tmpfs de 16 MiB | `/prometheus` em tmpfs de 256 MiB; `/tmp` de 16 MiB |
| Capabilities | Todas removidas por `cap_drop: ALL` | Todas removidas por `cap_drop: ALL` |
| Elevação de privilégios | `no-new-privileges:true` | `no-new-privileges:true` |
| Limite de recursos | 256 MiB de memória e 1 CPU | 512 MiB de memória e 1 CPU |
| Portas publicadas | `127.0.0.1:8000` por padrão | `127.0.0.1:9090` por padrão |
| Reinício | `unless-stopped` | `unless-stopped` |

O Dockerfile prepara `/data` com proprietário 10001 e permissões 0750. No teste
local com o volume do laboratório, a API executou sem root e conseguiu gravar o
SQLite. O volume preserva o banco ao recriar o container. Não há `destroy` ou
remoção de volume automático no deploy local. O rollback reimplanta uma imagem
anterior compatível e **não reverte banco ou migração de esquema**.

O Prometheus tem retenção temporal de 6 horas e armazenamento temporário em
memória. Não substitui backup ou armazenamento durável. Nenhum container recebe
o socket Docker; o daemon é utilizado pelos scripts no host e pelo runner CI.
No WSL local, o socket do daemon é Unix, sem listener TCP configurado.

Não há autenticação ou TLS na API/Prometheus, pois são serviços HTTP de
laboratório publicados em loopback. Os processos escutam suas portas dentro da
rede Docker para comunicarem entre si. Esse modelo não constitui implantação
pública segura. Também não há backup de produção nem restauração automatizada.

## Integridade e permissões do pipeline

O [workflow](../.github/workflows/ci.yml) do repositório separado
[devops-taskflow-fase2](https://github.com/Luiz-mnt/devops-taskflow-fase2)
usa actions fixadas por SHA, limites de tempo e permissão global `contents: read`. Pull requests validam qualidade,
testes, IaC e build/scan, sem publicar pacotes ou acessar segredos de deploy.

O job `publish` recebe `packages: write` e `contents: read`. O job `deploy-demo`
recebe `packages: read` e `contents: read`, permitindo baixar um pacote privado
sem ampliar a permissão dos demais jobs. A autenticação usa `GITHUB_TOKEN`
fornecido pelo GitHub; tokens não são copiados para o repositório ou exemplos.
Um repositório público não torna automaticamente o pacote GHCR público.
Nesta entrega, a visibilidade do pacote foi definida como pública e o
pull anônimo real foi aprovado no laboratório, com registro em
[pull-anonimo.log](evidencias/fase2-local/imagem-publicada/pull-anonimo.log).

A imagem Linux amd64 é construída uma vez e salva antes da análise. O mesmo
arquivo é analisado e transferido como artefato com checksum. A publicação confere o arquivo e o Config ID, sem reconstruir
outra imagem. Depois do push, obtém o digest real, baixa essa referência imutável
e confirma novamente seu Config ID. O deploy usa esse digest e `--no-build`.
As bases Python e Prometheus também estão fixadas por digests consultados.

O runner utiliza prefixo exclusivo e estado Terraform próprio. Evidências são
coletadas antes de `compose down` e `terraform destroy`, inclusive em falha.
Essa limpeza é restrita aos recursos efêmeros do job. O laboratório local é
persistente e independente da vida útil do runner.

## Gates de segurança e resultados medidos

`pip-audit` analisa o arquivo resolvido de dependências de desenvolvimento, que
inclui as dependências de produção. Usa modo estrito, sem correção automática,
sem resolver silenciosamente outras versões e sem lista de vulnerabilidades
ignoradas. Qualquer falha desse comando bloqueia o job de qualidade.

Na auditoria local, **51 pacotes foram avaliados e não houve vulnerabilidade
identificada pelo pip-audit**. Isso descreve a base consultada naquele momento,
sem garantir que novos avisos não serão publicados posteriormente.

Trivy produz um inventário completo da imagem, incluindo pacotes do sistema e
Python, com todas as severidades e achados sem correção. A etapa de inventário
usa exit code 0 para gravar o relatório; em seguida,
[`check_image_security.py`](../scripts/check_image_security.py) aplica o gate
obrigatório, verifica o Config ID e bloqueia toda vulnerabilidade **HIGH/CRITICAL
com versão corrigida disponível**. Falhas operacionais do scanner ou do gate
também interrompem o fluxo. Não há `continue-on-error` para obter aprovação.

A imagem local analisada teve **zero HIGH/CRITICAL corrigíveis**, mas **42
HIGH/CRITICAL sem correção informada: 40 HIGH e 2 CRITICAL**. Portanto, o resultado
do gate não significa zero vulnerabilidades nem risco zero. Os achados sem
correção ficam registrados e precisam ser reavaliados com novas bases e scans.

Os controles de usuário sem root, filesystem somente leitura, capabilities
removidas, ausência de elevação, portas em loopback e isolamento do laboratório
reduzem a exposição; não eliminam a possibilidade de exploração desses achados.
A política distingue explicitamente o que é bloqueado do risco residual aceito
para este laboratório acadêmico. Exceções adicionais precisam de justificativa,
escopo e validade, sem desligar os gates de forma geral.

Os resultados locais iniciais acima permanecem identificados com seu
ambiente. O [CI/CD 42520819347](https://github.com/Luiz-mnt/devops-taskflow-fase2/actions/runs/42520819347) também aprovou os gates:
pip-audit avaliou 50 pacotes Linux sem vulnerabilidades identificadas; Trivy
registrou zero HIGH/CRITICAL corrigíveis e 42 sem correção, com inventário
completo preservado. O
[facts.json](evidencias/ci/42520819347/facts.json) vincula esses resultados ao
commit `78fe0c236a426ec035a0a2d9daea218f083316a6` e à imagem
`ghcr.io/luiz-mnt/devops-taskflow-fase2@sha256:d25cad8350c946d5a3c1f646c1309be2fd1b6d82d1e7f83bd9924d90d8c4708d`.

O deploy remoto e o laboratório consumiram esse digest sem rebuild. Saúde,
CRUD, Prometheus UP/up=1 e coleta de logs passaram. No laboratório, a tarefa
id 5 permaneceu após recriar a API; o plan confirmou ausência de mudanças.
As [provas locais da imagem publicada](evidencias/fase2-local/imagem-publicada/)
documentam pull anônimo, usuário e persistência. As provas históricas de
fase 1 e PR continuam vinculadas ao repositório original. Rollback depende
de imagens compatíveis; a demonstração real e os digests estão registrados ao final deste documento.

## Demonstração real de rollback

A publicação do commit documental `ba6213b` produziu uma segunda imagem real:
`ghcr.io/luiz-mnt/devops-taskflow-fase2@sha256:640782cb89678df2d1d2969bb31a424be03cf5b62d51bafb91a4280e0459cb8b`. O código da aplicação, Dockerfile, scripts e workflow
permaneceram idênticos à implementação validada; a compatibilidade foi conferida
com git diff. Os metadados de revisão e as imagens publicadas são distintos.
Não se trata de uma nova funcionalidade da API.

O laboratório implantou essa imagem, aprovou o smoke e executou rollback para
`ghcr.io/luiz-mnt/devops-taskflow-fase2@sha256:d25cad8350c946d5a3c1f646c1309be2fd1b6d82d1e7f83bd9924d90d8c4708d`. Saúde, CRUD e Prometheus UP/up=1 passaram após a reversão;
a tarefa id 5 continuou no volume. O banco não foi revertido ou removido.
Provas: `docs/evidencias/fase2-local/rollback/`, incluindo os dois digests,
metadados da execução, logs e confirmação de persistência.

## Proteção da main do repositório separado

A main de Luiz-mnt/devops-taskflow-fase2 exige os checks Qualidade e testes,
Terraform e Build e seguranca da imagem, com atualização estrita, histórico
linear e conversas resolvidas. Exclusão da branch e force-push estão proibidos.
Administradores podem consolidar documentos diretamente neste projeto individual;
essa exceção é explícita. A publicação da imagem continua dependendo de todos
os gates do workflow, inclusive em um push administrativo. A configuração foi
aplicada somente no repositório separado; a proteção da fase1 não foi alterada.
