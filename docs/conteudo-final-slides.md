# Conteúdo final dos slides — Fase 2

## Slide 1

Luiz Eduardo Santos Monteiro

TaskFlow API: entrega, monitoramento e segurança

Entrega da Fase 2

## Slide 3

Expansão do CI para CI/CD

CI valida código, testes e IaC; build produz a imagem; auditorias aprovadas liberam a publicação no GHCR. PR valida sem publicar; main permite entrega após os gates.

Artefato rastreável

Commit 78fe0c236a42. A imagem aprovada é publicada e implantada pelo digest: sha256:d25cad8350c946d5a3c1f646c1309be2fd1b6d82d1e7f83bd9924d90d8c4708d.

Deploy automatizado

Runner hospedado provisiona rede/volume com prefixo exclusivo, executa Compose pelo digest e aplica smoke tests. Coleta provas e limpa somente os recursos efêmeros.

Execução e escopo

6/6 jobs aprovados: qualidade, Terraform, provisionamento, build/scan, publicação e deploy temporário. O ambiente do runner é temporário; o laboratório local mantém seus dados. https://github.com/Luiz-mnt/devops-taskflow-fase2/actions/runs/42520819347

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 4

Falha controlada — gate de qualidade

Recorte da captura original. Execução 42489956044: qualidade/testes falhou com exit 1; Terraform passou e o fluxo não avançou. Captura real de um PR, sem integração em main.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 5

CI/CD completo — seis jobs aprovados

Recorte da captura original. Execução real 42520819347 no novo repositório: validação, build/scan, publicação GHCR e deploy pelo digest aprovados. O ambiente temporário foi limpo após os testes.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 6

CD — smoke e observabilidade aprovados

Registro real do runner: saúde SQLite, criar/listar/concluir, alvo Prometheus UP e consulta up=1, executados após baixar a imagem pelo digest publicado.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 8

Dockerfile e imagem

https://github.com/Luiz-mnt/devops-taskflow-fase2/blob/fase-2/Dockerfile. Imagem: ghcr.io/luiz-mnt/devops-taskflow-fase2@sha256:d25cad8350c946d5a3c1f646c1309be2fd1b6d82d1e7f83bd9924d90d8c4708d.

Container da API

Base Python enxuta, dependências fixadas, usuário não-root, diretório /data gravável e healthcheck sem depender de curl.

Persistência e configuração

SQLite no volume externo taskflow-data; caminho por ambiente. .dockerignore exclui bancos, venv e segredos. Terraform é o único dono da rede e do volume.

Serviços coordenados

Compose conecta API e Prometheus à rede externa taskflow-net. Portas 8000 e 9090 em 127.0.0.1; recursos locais são preservados ao recriar containers.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 9

GHCR — imagem publicada e pública

Recorte da captura original. Página real do pacote, com visibilidade Public e tag vinculada ao commit 78fe0c2. Pull anônimo confirmado separadamente no laboratório.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 10

Scripts de deploy e validação

scripts/deploy.sh, scripts/smoke.sh e scripts/rollback.sh; compose.yaml configura os serviços. Links e comandos completos no README.

Fluxo de implantação

Provisionar a infraestrutura local; informar IMAGE_REF pelo digest; executar pull e compose up -d --wait com prazo limitado. Build local é um modo explícito de desenvolvimento.

Testes e persistência

Smoke: CD pelo digest aprovado: saúde SQLite, criar/listar/concluir, métricas e Prometheus UP; laboratório pela mesma imagem também aprovado.. Persistência: Imagem GHCR: tarefa id 5 preservada após recriar a API; container mudou; UID/GID 10001 confirmado..

Reversão

Demonstrado entre dois digests publicados compatíveis: smoke aprovado antes/depois e tarefa id 5 preservada. Rollback da imagem não reverte o banco.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 12

Planejamento e aplicação na fase 1

Concluída: 32 testes e 100% de linhas/branches; CI e IaC local/remoto aprovados.

CI e infraestrutura

Referência histórica validada: fase-1 / 30b49c595e93. GitHub Actions integrado aos testes; Terraform criou rede e volume, com plano posterior sem alterações.

Evolução preservada

A fase 2 usa um repositório separado. Main, tag fase-1 e PR #1 originais permanecem preservados; as métricas históricas mantêm sua referência.

Rastreabilidade

https://github.com/Luiz-mnt/devops-taskflow/actions/runs/42485447573

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 13

Entrega e deploy na fase 2

fase-2; commit 78fe0c236a42. Imagem imutável: sha256:d25cad8350c946d5a3c1f646c1309be2fd1b6d82d1e7f83bd9924d90d8c4708d.

Operação local

Smoke: Imagem pública pelo digest: saúde/CRUD/métricas aprovados antes e após recriar a API.. Monitoramento: Prometheus UP e consulta up=1 na imagem publicada..

Segurança e permissões

pip-audit e scanner de imagem antecedem a publicação. O gate bloqueia HIGH/CRITICAL com correção disponível; vulnerabilidades sem correção permanecem identificadas nos relatórios.

Resultados observados

Ubuntu 24.04 / CPython 3.12.16, CI/CD do novo repositório, 08/11/2026: 38/38 testes aprovados; cobertura 100.0%; pytest 1.22 s.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 14

API local — imagem publicada pelo digest

Recorte da captura original. Interface OpenAPI real da API executada no laboratório a partir da imagem pública GHCR validada pelo CI/CD.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 15

Monitoramento e logging

Prometheus coleta /metrics; /health verifica API e SQLite. Contadores e histogramas usam rotas normalizadas. Logs JSON em stdout registram status, rota e duração sem dados sensíveis.

Segurança e limitações reais

CI/CD: pip-audit 50 pacotes Linux, zero vulnerabilidades identificadas; Trivy zero HIGH/CRITICAL corrigíveis, 40 HIGH e 2 CRITICAL sem correção informada.

Configuração e propriedade

Versões/digest fixados e .env.example sem segredos. Terraform gerencia rede/volume; Compose gerencia containers. Estados e dados locais ficam fora do Git.

Evidências operacionais

Monitoramento: CD e laboratório: alvo taskflow-api UP e consulta up=1 aprovados.. Logs: Logs HTTP JSON reais preservados no CD e no laboratório, com rota normalizada, status e duração..

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 16

Prometheus local — alvo UP

Recorte da captura original. Interface real do Prometheus no laboratório com imagem publicada. A consulta up=1 também foi validada pelo smoke e preservada em JSON.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 17

Laboratório — validação após recriar a API

Registro real de terminal: saúde/CRUD e Prometheus aprovados após recriar a API. A tarefa de persistência permaneceu no volume externo.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 18

Fluxo DevOps — visão geral

Figura 1. Dependências entre as três seções. As próximas páginas ampliam o mesmo fluxo; Mermaid, SVG e PNG integrais acompanham a entrega.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 19

GitHub Actions: CI e aprovação — parte 1/2

Figura 1-CI — parte 1/2. Mesmos nós e arestas do fluxo completo; disposição horizontal por seção. Setas no limite continuam no recorte seguinte ou anterior.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 20

GitHub Actions: CI e aprovação — parte 2/2

Figura 1-CI — parte 2/2. Mesmos nós e arestas do fluxo completo; disposição horizontal por seção. Setas no limite continuam no recorte seguinte ou anterior.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 21

GitHub Actions: entrega e demonstração temporária — parte 1/2

Figura 1-CD — parte 1/2. Mesmos nós e arestas do fluxo completo; disposição horizontal por seção. Setas no limite continuam no recorte seguinte ou anterior.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 22

GitHub Actions: entrega e demonstração temporária — parte 2/2

Figura 1-CD — parte 2/2. Mesmos nós e arestas do fluxo completo; disposição horizontal por seção. Setas no limite continuam no recorte seguinte ou anterior.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 23

Laboratório persistente no WSL2

Figura 1-LOCAL. Mesmos nós e arestas do fluxo completo; disposição horizontal por seção. Contexto de entrada preservado.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 24

Integração — provas históricas

PR no repositório original: https://github.com/Luiz-mnt/devops-taskflow/pull/1. Falha controlada: https://github.com/Luiz-mnt/devops-taskflow/actions/runs/42489956044.

Aplicação e observabilidade

Deploy pelo digest; criação, listagem e conclusão de tarefa; /health; métricas e alvo Prometheus; logs JSON. A sequência e os comandos estão no roteiro de demonstração.

Persistência e recuperação

Imagem GHCR: tarefa id 5 preservada após recriar a API; container mudou; UID/GID 10001 confirmado.. Rollback: Demonstrado entre dois digests publicados compatíveis: smoke aprovado antes/depois e tarefa id 5 preservada. Rollback da imagem não reverte o banco..

Registros da demonstração

Falha controlada e PR corrigido no repositório original; CI/CD completo no separado; pull público, deploy, logs, Prometheus e persistência demonstrados.. Entrega verificada, sem requisito obrigatório pendente.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42

## Slide 25

Resultados medidos

CI/CD: 6 jobs aprovados, 38/38 testes, 100% de linhas/branches, pytest 1.22 s. Publicação e deploy pelo digest concluídos; pull anônimo, Prometheus UP e persistência local aprovados.

Critérios de sucesso

Entrega verificada, sem requisito obrigatório pendente.

Limitações

Laboratório de host único; monitoramento em tmpfs; não há implantação pública permanente. Achados de segurança sem correção são inventariados.

Melhorias futuras

Backup/restauração testados, PostgreSQL, alertas, TLS/autenticação e ambiente permanente com aprovação.

TaskFlow API • Fase 2 • atualizado em 08/11/2026 • código 78fe0c236a42
