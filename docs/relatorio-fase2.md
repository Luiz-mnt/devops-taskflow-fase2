# Relatório do projeto — fase 2 em consolidação

Aluno: Luiz Eduardo Santos Monteiro. Disciplina: DevOps na Prática, PUCRS.
Repositório da fase 2: https://github.com/Luiz-mnt/devops-taskflow-fase2

Este relatório registra a implementação e as provas observadas. A conclusão
da fase 2 foi autorizada em repositório separado, preservando a main, a tag
fase-1 e o PR #1 do [repositório original](https://github.com/Luiz-mnt/devops-taskflow).
A publicação GHCR, o CD em main e a referência fase-2 serão registrados
com as execuções do novo repositório. Seus resultados ainda estão pendentes
de consolidação; a fase 2 não está declarada pronta para submissão.

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
provisionamento efêmero. Essas etapas ainda aguardam resultados observados
neste relatório.

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
| Publicação e CD por digest | Repositório devops-taskflow-fase2 | Resultados ainda pendentes de consolidação |
| Rollback por digest | Quando houver duas imagens compatíveis | Script preparado; demonstração ainda não executada |

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

As provas mostram testes reproduzíveis, bloqueio de falhas, integridade da
imagem e preservação dos dados no laboratório. Ainda é necessário concluir
o ciclo remoto main → GHCR → deploy no repositório devops-taskflow-fase2
e registrar seus resultados antes da entrega. Esse fluxo não depende de
alteração ou merge no repositório original.

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
