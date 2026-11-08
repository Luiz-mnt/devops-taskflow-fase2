# TaskFlow API — DevOps na Prática, fase 2

Aluno: **Luiz Eduardo Santos Monteiro**. Projeto acadêmico da PUCRS.

Repositório público da fase 2: https://github.com/Luiz-mnt/devops-taskflow-fase2

A fase 2 dá continuidade ao projeto em um repositório separado. O
[repositório original da fase 1](https://github.com/Luiz-mnt/devops-taskflow),
sua main, a tag fase-1 e o PR #1 permanecem preservados. Os links de
execuções anteriores continuam apontando para o repositório em que ocorreram.

**Fase 1 concluída e verificada:** 32 testes locais e 32 testes no GitHub Actions,
cobertura de linhas/branches 100%, Ruff e Terraform aprovados. Rede e volume
provisionados no WSL2 local; no CI, provisionamento e limpeza efêmeros aprovados.
[CI aprovado](https://github.com/Luiz-mnt/devops-taskflow/actions/runs/42485447573) · [Tag fase-1](https://github.com/Luiz-mnt/devops-taskflow/tree/fase-1).

API de tarefas com FastAPI e SQLite, testes automatizados, integração contínua no
GitHub Actions e infraestrutura Docker descrita em Terraform. A fase 1 cobre
planejamento, CI e IaC. Containers da aplicação, Prometheus e entrega contínua
pertencem à fase 2 e não são necessários para executar esta API no ambiente virtual.

## Situação da fase 2 — entrega em repositório separado

A implementação foi validada localmente e no
[PR #1 histórico](https://github.com/Luiz-mnt/devops-taskflow/pull/1):
38 testes, 100% de linhas/branches, auditoria e build/scan aprovados na
execução 42511228720. O último CI desse PR, 42513263161, também aprovou
qualidade, Terraform e build/scan; publicação e deploy foram omitidos no PR.

A conclusão do CD será feita na main de devops-taskflow-fase2, sem integrar
o PR no repositório original. A publicação GHCR, o deploy automatizado, o
digest e a tag fase-2 permanecem pendentes de comprovação neste relatório.
A apresentação existente é uma prévia até a consolidação dos resultados
dessa execução e do PDF final.

[Workflow da fase 2](https://github.com/Luiz-mnt/devops-taskflow-fase2/blob/main/.github/workflows/ci.yml) ·
[Actions da fase 2](https://github.com/Luiz-mnt/devops-taskflow-fase2/actions).

[Relatório factual](docs/relatorio-fase2.md) ·
[Checklist da fase 2](docs/checklist-entrega.md) ·
[CI histórico com relatórios](https://github.com/Luiz-mnt/devops-taskflow/actions/runs/42511228720) ·
[Último CI do PR original](https://github.com/Luiz-mnt/devops-taskflow/actions/runs/42513263161).

## Documentação e comprovação

- [Planejamento e requisitos](docs/planejamento-fase1.md)
- [Ambiente realmente identificado](docs/ambiente.md)
- [Checklist de entrega](docs/checklist-entrega.md)
- [Testes e cobertura](docs/evidencias/fase1-testes.md)
- [Terraform e provisionamento](docs/evidencias/fase1-infra.md)
- [Repositório e execução real do CI](docs/evidencias/fase1-github.md)
- [Resumo verificável em JSON](docs/evidencias/resumo.json)
- [Procedimento de demonstração](docs/roteiro-demonstracao.md)

## Executar no Windows (PowerShell)

Instale Python 3.12 ou utilize um Python 3.12 portátil. Os comandos abaixo partem
da raiz deste repositório; não é preciso ativar o ambiente virtual.

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt
$env:TASKFLOW_DB_PATH = 'data/taskflow.db'
.\.venv\Scripts\python.exe -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

Abra http://127.0.0.1:8000/docs para o contrato OpenAPI interativo e
http://127.0.0.1:8000/health para verificar acesso ao banco.
O arquivo `.env.example` documenta a configuração; a API lê a variável do processo
e não carrega `.env` automaticamente.

No ambiente já preparado nesta pasta, `.venv` usa o Python 3.12 portátil em
`../../Fase 1/.tools/python/`. Basta executar os comandos `.venv` acima; `py -3.12`
documenta a recriação em uma máquina com esse Python registrado no launcher.

## Executar no Linux/WSL

```bash
python3.12 -m venv .venv-linux
.venv-linux/bin/python -m pip install -r requirements-dev.txt
export TASKFLOW_DB_PATH=data/taskflow.db
.venv-linux/bin/python -m uvicorn app.main:app --host 127.0.0.1 --port 8000
```

O nome `.venv-linux` preserva o ambiente `.venv` do Windows ao compartilhar a
mesma pasta pelo WSL. Ambientes virtuais não são intercambiáveis entre os sistemas.

## Contrato HTTP

| Método e rota | Entrada | Resultado |
|---|---|---|
| `POST /tasks` | `{"title":"Estudar CI"}` | 201, tarefa com `id`, título e `status: pending` |
| `GET /tasks` | — | 200, lista ordenada por ID |
| `PATCH /tasks/{id}` | `{"status":"completed"}` | 200, tarefa concluída; repetição é idempotente |
| `GET /health` | — | 200 com banco acessível; 503 se consulta ao banco falhar |
| `GET /metrics` | — | 200, métricas HTTP no formato Prometheus |

Títulos aceitam de 1 a 200 caracteres após remover espaços nas extremidades.
Entradas inválidas retornam 422. Tarefa inexistente retorna 404. A base SQLite
permanece no caminho configurado entre reinícios e é excluída do Git. Cada teste
utiliza banco temporário próprio. `GET /metrics` expõe métricas HTTP para o Prometheus.

## Lint, testes e relatórios

```powershell
.\.venv\Scripts\python.exe -m ruff check .
.\.venv\Scripts\python.exe -m ruff format --check .
.\.venv\Scripts\python.exe -m pytest
```

No Linux, substitua `.\.venv\Scripts\python.exe` por `.venv-linux/bin/python`.
O `pyproject.toml` exige cobertura mínima de **80%** com medição de branches.
O pytest gera `reports/junit.xml`, `reports/coverage.xml` e `reports/htmlcov/index.html`.
O CI publica esses relatórios inclusive quando a etapa de testes falha.
Dependências e ferramentas de desenvolvimento estão fixadas nos arquivos
`requirements*.txt`; os arquivos `.in` registram as dependências diretas.

## Infraestrutura como Código

O laboratório desta entrega foi executado na distribuição WSL2 **TaskFlowDevOps**,
com Docker Engine instalado diretamente no Ubuntu. Para reproduzir neste
computador, consulte [o procedimento WSL](docs/ambiente-wsl.md). Os comandos
PowerShell abaixo são uma alternativa para instalações com Docker Desktop.

É necessário Terraform na versão de `infra/versions.tf` e Docker Engine acessível
para provisionar. No Docker Desktop, use Linux containers. O Terraform é o único
responsável pela rede bridge e pelo volume; Compose os consome como recursos
externos. A API em container persiste em `/data`; a execução opcional em ambiente
virtual utiliza o caminho local configurado por TASKFLOW_DB_PATH.

```powershell
# Windows com Docker Desktop aberto:
$env:TF_VAR_docker_host = 'npipe:////./pipe/docker_engine'
docker version
terraform -chdir=infra init -lockfile=readonly
terraform -chdir=infra fmt -check -recursive
terraform -chdir=infra validate
terraform -chdir=infra plan -out=local.tfplan
terraform -chdir=infra apply local.tfplan
docker network inspect taskflow-net
docker volume inspect taskflow-data
terraform -chdir=infra plan -detailed-exitcode
```

No WSL/Linux, use `export TF_VAR_docker_host=unix:///var/run/docker.sock` e os mesmos
comandos. O último `plan` deve encerrar com código **0** (sem alterações); **2**
significa alterações e **1** significa erro. Estado, planos, `.terraform/`, bancos e
segredos não são versionados. O `.terraform.lock.hcl` é versionado.

`terraform init -backend=false` + `terraform validate` validam dependências e a
configuração, **sem criar infraestrutura**. Somente `plan`/`apply` com Docker
acessível comprovam provisionamento. O CI está configurado para executar uma
demonstração em runner efêmero, com prefixo exclusivo e limpeza desses recursos.
A demonstração da fase 1 foi aprovada na execução histórica 42485447573,
incluindo inspeções, plano sem alterações e limpeza dos recursos efêmeros.
O provisionamento da nova execução de CD da fase 2 será registrado
separadamente. O laboratório WSL2 local foi comprovado e permanece preservado.

Não há `destroy` automático do laboratório local. Alterar o prefixo pode planejar
a substituição do volume: revise o plano antes de aplicar para preservar dados.

## Pipeline

O workflow `.github/workflows/ci.yml` deste repositório da fase 2 executa
em push e pull request para `main`, além de acionamento manual.
Os jobs verificam Ruff, testes com cobertura mínima de 80%, auditoria das
dependências de produção/desenvolvimento e formatação/validação Terraform.
Build/scan constrói uma imagem Linux amd64 e inventaria suas vulnerabilidades.
O gate bloqueia HIGH/CRITICAL com correção disponível. O inventário registra
também vulnerabilidades sem correção e das demais severidades.

Pull requests validam e não publicam. Após aprovação de build/scan e da prova
IaC, main publica no GHCR exatamente a imagem aprovada, sem rebuild, usando
tag sha-commit e digest real. Um job dependente baixa esse digest, provisiona
rede/volume exclusivos, implanta API/Prometheus, executa smoke e coleta provas.
A limpeza em always remove somente os recursos efêmeros do runner.

Somente o job de publicação recebe packages: write; o deploy usa packages: read.
Login utiliza GITHUB_TOKEN fornecido pelo GitHub Actions. Não há token pessoal
no código. O laboratório local é independente e seus dados são preservados.
Os registros da fase 1 continuam em docs/evidencias/ci/42485447573/.

O token tem `contents: read`, as actions são fixadas por SHA e há limites de tempo.
O resultado observado e os links da execução estão na documentação de evidências.

## Limites e evolução

O projeto é um laboratório de host único: SQLite e rede/volume Docker locais.
Não oferece autenticação, alta disponibilidade, nuvem gerenciada ou implantação
pública da API. A fase 2 acrescenta containers, métricas e entrega contínua.
O checklist diferencia requisitos do enunciado e verificações adicionais do guia.

Fontes: [FastAPI](https://fastapi.tiangolo.com/),
[GitHub Actions para Python](https://docs.github.com/en/actions/tutorials/build-and-test-code/python),
[Terraform com Docker](https://developer.hashicorp.com/terraform/tutorials/docker-get-started/docker-build)
e [Docker Desktop no Windows](https://docs.docker.com/desktop/setup/install/windows-install/).

## Fase 2 — construir e implantar localmente

No Ubuntu/WSL com Docker Linux, Compose, Terraform, Python 3 e este diretório:

```bash
bash scripts/start-lab-wsl.sh  # somente na distro dedicada TaskFlowDevOps
export TF_VAR_resource_prefix=taskflow
export TF_VAR_docker_host=unix:///var/run/docker.sock
export TASKFLOW_RESOURCE_PREFIX=taskflow
terraform -chdir=infra init -lockfile=readonly
terraform -chdir=infra plan -out=local.tfplan
# Revise o plano; preserve o estado e o volume existentes.
terraform -chdir=infra apply local.tfplan
docker build --platform linux/amd64 --tag taskflow-local:teste .
export IMAGE_REF=taskflow-local:teste
docker compose pull prometheus
IMAGE_PULL_POLICY=never bash scripts/deploy.sh
bash scripts/smoke.sh
```

API: http://127.0.0.1:8000/docs; métricas: http://127.0.0.1:8000/metrics;
Prometheus: http://127.0.0.1:9090. Consulte up{job="taskflow-api"} e
taskflow_http_requests_total. Logs HTTP JSON usam rota normalizada, status e
duração, sem títulos, IDs, corpo, headers ou query string. Métodos desconhecidos
e rotas não encontradas usam rótulos fixos para limitar cardinalidade.

```bash
docker compose logs --follow api
docker compose ps
docker compose restart api
docker compose up -d --no-deps --force-recreate --wait api
bash scripts/smoke.sh
docker compose down   # preserva rede e volume externos; não use terraform destroy
```

O volume SQLite persiste entre recriações da API. Prometheus usa armazenamento
temporário de até 256 MB e retenção de seis horas; suas séries reiniciam quando
o container é recriado. A API executa como UID/GID 10001, com filesystem raiz
somente leitura, capabilities removidas e sem socket Docker montado.

## Fase 2 — imagem publicada, segurança e rollback

Use o digest real registrado em docs/evidencias/ e nos artefatos do workflow.
Para pacote público, docker pull funciona sem login. Se privado, autentique
pelo fluxo oficial do registry com read:packages; não coloque tokens no Git.
A visibilidade do repositório não determina automaticamente a do pacote.

```bash
export IMAGE_REF=ghcr.io/luiz-mnt/devops-taskflow-fase2@sha256:DIGEST_REAL_DO_RELATORIO
docker pull "$IMAGE_REF"
bash scripts/deploy.sh
bash scripts/smoke.sh
# Quando houver um digest anterior compatível:
bash scripts/rollback.sh ghcr.io/luiz-mnt/devops-taskflow-fase2@sha256:DIGEST_ANTERIOR
```

Substitua os nomes em maiúsculas pelos digests medidos no relatório. O caminho
de deploy não faz build. Rollback reimplanta uma imagem e preserva o banco;
ele não reverte esquema nem restaura dados. Revise compatibilidade antes de usar.

Auditoria de dependências: python -m pip_audit --strict --no-deps --disable-pip
-r requirements-dev.txt. A política de imagem mantém o inventário completo e
bloqueia HIGH/CRITICAL corrigíveis. Vulnerabilidades sem correção não são
tratadas como ausência de risco: constam do relatório e exigem acompanhamento.
Não há supressão geral de falhas ou lista de CVEs ignoradas.

[Planejamento da fase 2](docs/planejamento-fase2.md) ·
[Contrato HTTP](docs/contrato-api.md).
