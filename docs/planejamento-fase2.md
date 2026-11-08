# Planejamento da fase 2 — Entrega Contínua, Monitoramento e Segurança

Aluno: Luiz Eduardo Santos Monteiro. Projeto: TaskFlow API, DevOps na Prática.

Esta fase evolui a TaskFlow API no repositório separado
[devops-taskflow-fase2](https://github.com/Luiz-mnt/devops-taskflow-fase2).
O [repositório original](https://github.com/Luiz-mnt/devops-taskflow), sua main,
o PR #1 e a [tag fase-1](https://github.com/Luiz-mnt/devops-taskflow/tree/fase-1)
permanecem preservados. O CD foi executado na main do repositório da fase 2,
sem integrar o PR original, e seus seis jobs foram aprovados no
[run 42520819347](https://github.com/Luiz-mnt/devops-taskflow-fase2/actions/runs/42520819347). O pacote e os documentos finais da fase 1
continuam separados dos arquivos desta fase.

## Objetivo e requisitos

Entregar uma imagem verificável da API, implantar containers de forma
reproduzível e demonstrar testes, métricas, logs e controles de segurança.
O documento final deve incluir o pipeline de CD, o fluxograma completo,
a demonstração prática, resultados observados e melhorias futuras.

| Requisito | Implementação planejada | Verificação necessária |
|---|---|---|
| CI expandido para CD | GitHub Actions com dependências entre jobs | Execução real aprovada, vinculada ao commit |
| Containerização | Dockerfile, imagem Linux amd64, processo sem root | Build, inspeção de usuário e healthcheck |
| Orquestração e deploy | Compose com API e Prometheus; scripts Bash | Saúde, CRUD, persistência e alvo UP |
| Infraestrutura como código | Rede/volume Terraform consumidos como external | Apply ou estado existente, inspect e plan sem mudanças |
| Segurança | Dependências fixadas, pip-audit, Trivy, permissões mínimas | Relatórios reais; gates bloqueiam falhas |
| Monitoramento e logging | Métricas HTTP normalizadas e logs JSON | Consulta Prometheus e registro HTTP sem conteúdo de tarefas |
| Gerenciamento de configurações | Variáveis documentadas e imagem por digest | Mesmo prefixo Terraform/Compose; sem segredos no Git |
| Relatório e fluxograma | Documento no modelo da instituição e PDF | Links funcionais, resultados reais, ausência de campos por preencher |

## Arquitetura e responsabilidade

O laboratório persistente utiliza Ubuntu/WSL2 no computador do estudante.
Terraform é o único proprietário da rede e do volume. Compose consome esses
recursos externos e administra os containers da API e do Prometheus.
API e Prometheus devem publicar portas somente em 127.0.0.1 por padrão.

O estado Terraform existente é usado para dar continuidade aos recursos da
fase 1. O estado ativo passa a ser o da pasta de trabalho da fase 2; não execute
applies simultâneos a partir das duas pastas. Estados e planos não são publicados
nem incluídos no pacote de código. O volume de dados local deve ser preservado.

O deploy no GitHub Actions é uma demonstração temporária em um runner Linux.
Usa prefixo exclusivo, baixa a imagem pelo digest publicado e testa o fluxo.
Os logs e relatórios são coletados antes da limpeza. A limpeza automática
se restringe aos containers, à rede e ao volume dessa execução remota.

## Fluxo e critérios de aprovação

Pull request executa qualidade, testes, validação IaC e build/scan sem publicar.
Push em main no repositório da fase 2 publica somente depois da aprovação dos gates. A imagem aprovada
é transferida para a publicação sem reconstrução. O digest resultante identifica
o artefato que o deploy consome.

Falhas de lint, testes, cobertura abaixo de 80%, auditoria de dependências,
vulnerabilidades HIGH/CRITICAL corrigíveis da imagem ou smoke tests impedem
a aprovação do fluxo correspondente. Relatórios devem ser preservados também
em falhas. Vulnerabilidades sem correção são registradas separadamente;
qualquer exceção exige motivo, escopo e validade explícitos.

A falha controlada já demonstrou o bloqueio no PR #1 do repositório original:
execução 42489956044. A remoção do teste intencional foi validada na execução
42511228720; ambas permanecem como evidências históricas nesse repositório.
Rollback usa uma imagem anterior compatível quando disponível e não reverte
alterações no banco. O CD aprovado está registrado em
[facts.json](evidencias/ci/42520819347/facts.json), com 38 testes e 100%
de cobertura. O [laboratório com a imagem publicada](evidencias/fase2-local/imagem-publicada/)
confirmou pull anônimo, deploy, CRUD, Prometheus up=1, preservação da tarefa
id 5 e plan sem mudanças. Demonstrado entre dois digests publicados compatíveis: smoke aprovado antes/depois e tarefa id 5 preservada. Rollback da imagem não reverte o banco.

O código validado é o commit `78fe0c236a426ec035a0a2d9daea218f083316a6`,
preservado na [tag fase-2](https://github.com/Luiz-mnt/devops-taskflow-fase2/tree/fase-2).
Evidências e documentos consolidados depois da
execução são mantidos na main, sem atribuí-los ao conteúdo anterior do commit.

## Limites e melhorias futuras

O ambiente é acadêmico, de host único, sem serviço público permanente, SLA ou
alta disponibilidade. SQLite atende ao laboratório com um processo da API.
Evoluções possíveis incluem PostgreSQL, backup com restauração testada,
TLS/autenticação, alertas, implantação permanente e aprovação de ambientes.

Fontes técnicas: [GitHub Container Registry](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry),
[visibilidade de pacotes](https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility),
[Compose](https://docs.docker.com/reference/compose-file/services/),
[Prometheus](https://prometheus.io/docs/introduction/overview/),
[pip-audit](https://github.com/pypa/pip-audit) e
[Trivy](https://trivy.dev/docs/latest/).

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
