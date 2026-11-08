# Relatório do projeto — fases 1 e 2 verificadas

Aluno: Luiz Eduardo Santos Monteiro. Disciplina: DevOps na Prática, PUCRS.
Repositório da fase 2: https://github.com/Luiz-mnt/devops-taskflow-fase2

Este relatório registra a implementação e as provas observadas. A conclusão
da fase 2 foi autorizada em repositório separado, preservando a main, a tag
fase-1 e o PR #1 do [repositório original](https://github.com/Luiz-mnt/devops-taskflow).
O [CI/CD 42520819347](https://github.com/Luiz-mnt/devops-taskflow-fase2/actions/runs/42520819347) aprovou os seis jobs em 08/11/2026,
no commit `78fe0c236a426ec035a0a2d9daea218f083316a6`. A imagem foi publicada no GHCR e
implantada pelo digest no runner e no laboratório local. As provas do CD
e do laboratório estão preservadas no projeto. O código validado é a
referência da tag fase-2; a main consolida os documentos e as evidências
produzidos após essa execução.

## Projeto e fase 1

A TaskFlow API cria, lista e conclui tarefas com FastAPI e SQLite. A fase 1
definiu objetivos, requisitos, contrato HTTP, infraestrutura e critérios de CI.
GitHub Actions passou a verificar Ruff, testes com cobertura mínima de 80% e
Terraform. Rede bridge e volume Docker foram realmente provisionados no WSL2;
inspeções e um segundo plan sem alterações confirmaram o estado do laboratório.

A implementação da fase 1 permanece no commit
30b49c595e931f1ea6c2e122ba4ca25ead004b48 e na
[tag fase-1 do repositório original](https://github.com/Luiz-mnt/devops-taskflow/tree/fase-1).
A [execução histórica 42485447573](https://github.com/Luiz-mnt/devops-taskflow/actions/runs/42485447573)
aprovou os três jobs, 32 testes e 100% de linhas/branches.
O PDF, PPTX e pacote finais dessa fase foram preservados na pasta Fase 1.

## Expansão da fase 2

O workflow liga qualidade/testes/auditoria e Terraform ao build/scan. A imagem
Linux amd64 é construída uma única vez e salva antes da análise. Trivy examina
esse arquivo; checksum e Config ID independente vinculam o artefato aprovado à
publicação, sem reconstrução. O job de publicação aceita somente push/manual
na main do repositório da fase 2 e depende dos gates e da prova de
provisionamento efêmero. As etapas de publicação e deploy foram executadas
e aprovadas no run 42520819347.

GHCR recebe tag sha-commit e disponibiliza o digest real. O job dependente
deploy-demo utiliza o digest, cria rede e volume em um runner independente,
executa Compose sem build e testa saúde, CRUD e Prometheus. Coleta logs,
inspeções e métricas antes da limpeza, inclusive em falha. A limpeza se limita
ao ambiente temporário desse job; o laboratório local continua persistente.

O Dockerfile usa Python com digest fixado, UID/GID 10001 e healthcheck que
consulta a API e o SQLite. Compose administra API e Prometheus e referencia
rede/volume como external. O estado Terraform foi copiado explicitamente para
a pasta da fase 2, que passa a ser o estado ativo. Não há proprietários
concorrentes desses recursos no Compose nem destroy automático dos dados locais.

## Observabilidade, testes e configurações

O endpoint /metrics expõe contador HTTP e histograma por rota normalizada.
IDs, query strings, títulos e headers não são labels. Rotas não encontradas
e métodos não usuais usam valores fixos, limitando cardinalidade. Cada factory
de aplicação mantém registro isolado para testes.

Prometheus coleta api:8000/metrics a cada cinco segundos. O smoke confirma o
alvo UP e consulta up=1, além de testar o CRUD. Logs HTTP em stdout são JSON
com instante UTC, método, rota, status e duração, sem conteúdo das tarefas.
Logs de inicialização/erro do servidor podem aparecer em stderr.

Configurações ficam nas variáveis documentadas e em monitoring/prometheus.yml.
API e Prometheus publicam portas somente em 127.0.0.1 por padrão. O banco fica
no volume externo /data; o histórico do Prometheus é temporário, em tmpfs de
256 MiB e retenção temporal de seis horas. Limites de memória/CPU e políticas
de reinício estão definidos. Não há socket Docker montado nos containers.

## Resultados do CD e da imagem publicada

A execução real de push em main no novo repositório aprovou qualidade/testes,
Terraform, provisionamento efêmero, build/scan, publicação GHCR e deploy
temporário. O runner coletou provas antes de remover seus próprios recursos.

| Verificação | Origem | Resultado |
|---|---|---|
| CI/CD completo | Run 42520819347, commit 78fe0c2 | Seis jobs aprovados |
| Testes e cobertura | Runner Linux, Python 3.12.16 | 38 aprovados; 131/131 linhas; 8/8 branches; pytest 1,22 s; JUnit 1,230 s |
| Auditoria de dependências | Pip-audit do CD | 50 pacotes; zero vulnerabilidades identificadas |
| Scan de imagem | Trivy e gate obrigatório do CD | Zero HIGH/CRITICAL corrigíveis; 40 HIGH e 2 CRITICAL sem correção |
| Publicação | GHCR | Mesma imagem aprovada, checksum/Config ID conferidos sem rebuild; digest registrado |
| Deploy temporário | Runner independente | Saúde SQLite, CRUD, métricas e Prometheus UP/up=1 aprovados; limpeza aprovada |
| Pull local | WSL2, pacote público | Download anônimo por digest aprovado |
| Deploy local | Imagem publicada, sem rebuild | API e Prometheus saudáveis; CRUD e up=1 aprovados |
| Persistência local | Recriação do container API | Container mudou e tarefa id 5 permaneceu; UID/GID 10001 |
| Infraestrutura local | Terraform, estado ativo da fase 2 | Plan sem mudanças; rede e volume preservados |
| Rollback | Duas imagens publicadas compatíveis | Demonstrado: smoke aprovado e tarefa id 5 preservada |

Imagem validada: `ghcr.io/luiz-mnt/devops-taskflow-fase2@sha256:d25cad8350c946d5a3c1f646c1309be2fd1b6d82d1e7f83bd9924d90d8c4708d`.

O Config ID `sha256:e7dc805de61db984319ecef80b03afdb954900b5ae936b39bf7be4eee69edb8f`
vincula a configuração da imagem analisada à publicada. Ele é diferente do
digest do manifesto do registry e não deve ser confundido com esse digest.
As provas estão em [facts.json](evidencias/ci/42520819347/facts.json), nos
artefatos preservados do run e nos
[registros locais da imagem publicada](evidencias/fase2-local/imagem-publicada/).

## Resultados locais e históricos do PR original

As execuções de PR desta seção pertencem a Luiz-mnt/devops-taskflow. Elas
comprovam os gates anteriores à separação e não representam publicação
ou deploy no novo repositório. O
[CI 42511228720](https://github.com/Luiz-mnt/devops-taskflow/actions/runs/42511228720)
fornece os relatórios medidos abaixo. O
[último CI do PR, 42513263161](https://github.com/Luiz-mnt/devops-taskflow/actions/runs/42513263161),
também aprovou os três jobs de validação; provisionamento, publicação e
deploy foram omitidos nesse evento.

| Verificação | Origem | Resultado |
|---|---|---|
| Testes e cobertura | Windows/Python 3.12.16, pasta final Fase 2 | 38 aprovados; 131/131 linhas e 8/8 branches; pytest 5,55 s |
| Testes e cobertura | PR 42511228720, Linux/Python 3.12.16 | 38 aprovados; 100%; pytest 1,29 s; JUnit 1,305 s |
| Ruff, formatação, pip check | Ambiente local final | Aprovados |
| Auditoria de dependências | Local Windows | 51 pacotes; zero vulnerabilidades identificadas |
| Auditoria de dependências | PR Linux | 50 pacotes; zero vulnerabilidades identificadas |
| Scan de imagem | Local e PR | Zero HIGH/CRITICAL corrigíveis; 40 HIGH e 2 CRITICAL sem correção informada |
| Deploy local | WSL2/Docker a partir da pasta Fase 2 | API e Prometheus saudáveis; CRUD e UP=1 aprovados |
| Persistência | Recriação real do container API | Tarefa id 2 preservada; UID 10001 confirmado |
| Infraestrutura | Estado ativo da fase 2 | Rede/volume existentes; plan sem mudanças, exit 0 |
| Falha controlada | PR original, execução 42489956044 | 1 falha intencional e 32 testes aprovados; CI bloqueou; teste removido e correção validada |
| PR corrigido | Execução 42511228720 | Qualidade, Terraform e build/scan aprovados; publicação/CD omitidos corretamente em PR |

O pacote colorama, condicionado ao Windows, explica a diferença 51/50 entre
as auditorias. Tempos local e remoto são medições distintas e não sustentam
uma comparação controlada de desempenho. O aviso Starlette sobre o TestClient
httpx permanece visível, sem supressão; não reprovou os testes.

## Segurança e análise dos resultados

Pip-audit bloqueia falhas de auditoria das dependências fixadas. Trivy inventaria
todas as severidades; o gate obrigatório bloqueia HIGH/CRITICAL com correção
disponível. Zero bloqueantes não significa zero vulnerabilidades: os 42 achados
sem correção são explicitamente registrados e devem ser acompanhados.

O token global tem contents: read. Somente publicação recebe packages: write;
deploy recebe packages: read e usa o GITHUB_TOKEN do Actions. Actions e bases
estão fixadas por SHA/digest. Containers usam filesystem raiz somente leitura,
capabilities removidas e no-new-privileges. Não há autenticação/TLS porque o
ambiente é um laboratório local em loopback, sem implantação pública permanente.

As provas confirmam testes reproduzíveis, bloqueio de falhas, integridade
da imagem, publicação e deploy pelo mesmo digest. O ciclo remoto
main → GHCR → deploy foi concluído no repositório devops-taskflow-fase2.
O laboratório consumiu essa imagem e preservou os dados ao recriar a API.
Nenhum desses resultados dependeu de alteração ou merge no repositório original.

## Fluxograma, demonstração e melhorias

A fonte docs/fluxograma.mmd representa CI, decisão de aprovação/reprovação,
build/scan, publicação, provisionamento/deploy temporário, smoke/observabilidade,
coleta/limpeza e laboratório persistente. O documento inclui a visão geral e
ampliações para legibilidade, preservando a fonte e a exportação completas.
O roteiro reproduzível está em docs/roteiro-demonstracao.md.

Melhorias futuras: backups com restauração testada; migração para PostgreSQL;
alertas e armazenamento durável do Prometheus; HTTPS/autenticação para um
ambiente exposto; atualização contínua de bases e triagem de vulnerabilidades;
aprovação de ambientes e implantação permanente. Rollback de imagem não
equivale a restauração de banco, nem garante compatibilidade entre esquemas.

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
