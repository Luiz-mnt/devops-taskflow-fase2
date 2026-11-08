# Roteiro de demonstração — fase 2

Abra Fase 2/devops-taskflow, cópia local usada para publicar a fase 2 no
[repositório separado](https://github.com/Luiz-mnt/devops-taskflow-fase2).
A fase 1 pode ser consultada pela
[tag fase-1 original](https://github.com/Luiz-mnt/devops-taskflow/tree/fase-1).
Utilize dados fictícios e não mostre credenciais ao gravar capturas. Registre
o [CD aprovado](https://github.com/Luiz-mnt/devops-taskflow-fase2/actions/runs/42520819347) e o laboratório pelos registros preservados.
O [código congelado na tag fase-2](https://github.com/Luiz-mnt/devops-taskflow-fase2/tree/fase-2)
corresponde à tag fase-2; os
[documentos e evidências atuais](https://github.com/Luiz-mnt/devops-taskflow-fase2/tree/main/docs)
ficam em main, pois foram consolidados depois da execução. Não espere
encontrar registros posteriores dentro do commit de código validado.

## 1. Histórico, PR e gates

1. Mostre o planejamento e o código da tag fase-1 no repositório original.
2. Mostre o [PR #1 original](https://github.com/Luiz-mnt/devops-taskflow/pull/1),
   a [falha intencional 42489956044](https://github.com/Luiz-mnt/devops-taskflow/actions/runs/42489956044)
   e o bloqueio do CI, mantendo esse PR sem merge.
3. Mostre a remoção do teste controlado e o
   [CI corrigido 42511228720](https://github.com/Luiz-mnt/devops-taskflow/actions/runs/42511228720):
   qualidade/testes, Terraform e build/scan aprovados; publicação omitida em PR.
4. Mostre o [run 42520819347](https://github.com/Luiz-mnt/devops-taskflow-fase2/actions/runs/42520819347) e os seis jobs aprovados:
   qualidade/testes, Terraform, provisionamento, build/scan, publicação e
   deploy. Identifique o commit `78fe0c236a426ec035a0a2d9daea218f083316a6`.
5. Abra os [fatos e artefatos atuais](https://github.com/Luiz-mnt/devops-taskflow-fase2/tree/main/docs/evidencias/ci/42520819347)
   para mostrar 38 testes, 100% de cobertura, pytest 1,22 s, pip-audit 50/0
   e o inventário da imagem com zero HIGH/CRITICAL corrigíveis e 42 sem correção.
6. Mostre o digest publicado e o deploy temporário aprovado. As provas foram
   coletadas antes da limpeza; o runner não é um servidor permanente.

## 2. Laboratório e configuração

No PowerShell, dentro da raiz do projeto:

```powershell
wsl -d TaskFlowDevOps -u root -- bash scripts/start-lab-wsl.sh
wsl -d TaskFlowDevOps -u root -- bash
```

No Bash, confira o estado existente sem excluir dados:

```bash
export TF_VAR_docker_host=unix:///var/run/docker.sock
export TF_VAR_resource_prefix=taskflow
export TASKFLOW_RESOURCE_PREFIX=taskflow
terraform -chdir=infra plan -detailed-exitcode
docker network inspect taskflow-net
docker volume inspect taskflow-data
```

O estado ativo é o da fase 2. Não aplique simultaneamente a cópia antiga da
fase 1 para os mesmos recursos. Nunca use destroy para demonstrar persistência.

## 3. Deploy pela imagem publicada

O pacote está público e o pull anônimo foi validado no laboratório. Para
reproduzir a implantação, use o digest real do run 42520819347:

```bash
export IMAGE_REF=ghcr.io/luiz-mnt/devops-taskflow-fase2@sha256:d25cad8350c946d5a3c1f646c1309be2fd1b6d82d1e7f83bd9924d90d8c4708d
docker pull "$IMAGE_REF"
bash scripts/deploy.sh
bash scripts/smoke.sh
docker compose ps
docker compose logs --tail 30 api
```

Pacotes públicos permitem pull sem login. Se o pacote for privado, autentique
pelo procedimento oficial com read:packages, no terminal, sem incluir token
em arquivos ou na conversa. O deploy faz pull por digest e não reconstrói.

Alternativa inicial sem registry: docker build --tag taskflow-local:teste .;
defina IMAGE_REF=taskflow-local:teste, baixe Prometheus uma vez e use
IMAGE_PULL_POLICY=never bash scripts/deploy.sh. Identifique essa prova como local.

## 4. Métricas, logs e persistência

Abra http://127.0.0.1:8000/docs e crie/conclua uma tarefa fictícia. Mostre
/health, /metrics e os logs JSON, sem conteúdo da tarefa. No Prometheus,
http://127.0.0.1:9090/targets deve mostrar o alvo taskflow-api UP; consulte
up{job="taskflow-api"}. Mostre que IDs não aparecem nos rótulos das rotas.

Guarde o id/título/status de uma tarefa da listagem, recrie a API com
docker compose up -d --no-deps --force-recreate --wait api e consulte a listagem
novamente. Confirme que o container mudou e a mesma tarefa permaneceu. O smoke
sozinho cria uma tarefa nova e não substitui esse teste de persistência.

A demonstração registrada preservou a tarefa id 5 após a troca real do
container, com a imagem GHCR e UID/GID 10001. Mostre
[persistence-result.json](https://github.com/Luiz-mnt/devops-taskflow-fase2/blob/main/docs/evidencias/fase2-local/imagem-publicada/persistence-result.json),
os logs de smoke antes/depois, os logs JSON, Prometheus `up=1` e o plano
Terraform sem mudanças na mesma pasta de evidências.

## 5. Rollback compatível e encerramento

A demonstração abaixo foi executada entre os dois digests reais publicados.
Para reproduzir, implante o digest seguinte, confirme saúde/CRUD/Prometheus e
execute scripts/rollback.sh com o digest anterior compatível. Confirme também
a preservação da tarefa original. Rollback de imagem não reverte o banco.

Para encerrar containers locais preservando os dados, use docker compose down.
Rede e volume são externos e continuam sob Terraform. A limpeza automática
com destroy aparece somente no runner efêmero, depois da coleta das provas.

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
