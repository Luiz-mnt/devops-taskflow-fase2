# Checklist de entrega — fase 2 em repositório separado

A fase 1 está concluída no [repositório original](https://github.com/Luiz-mnt/devops-taskflow)
e preservada na [tag fase-1](https://github.com/Luiz-mnt/devops-taskflow/tree/fase-1).
A fase 2 será concluída em [devops-taskflow-fase2](https://github.com/Luiz-mnt/devops-taskflow-fase2),
sem alterar a main original nem integrar o PR #1. As provas locais e do PR
histórico estão registradas; o CD no novo repositório e o PDF final ainda
precisam de resultados consolidados. Este checklist não declara a entrega pronta.

| Requisito | Arquivo / evidência | Situação |
|---|---|---|
| Expandir CI para CD | .github/workflows/ci.yml | Implementado; aguarda comprovação do CD em main no repositório da fase 2 |
| Dockerfile não-root | Dockerfile | Build e deploy locais aprovados |
| Orquestração e scripts de deploy | compose.yaml e scripts/*.sh | Smoke local aprovado |
| Monitoramento e logs | app/observability.py, monitoring/, docs/observabilidade.md | UP=1 e JSON locais confirmados |
| Testes integrados | tests/ e reports/ | 38 locais e 38 no PR histórico; 100% de linhas/branches |
| Segurança | pip-audit, Trivy e gate obrigatório | PR histórico: auditoria e scan aprovados; 42 HIGH/CRITICAL sem correção registrados |
| Infraestrutura e persistência | infra/, volume externo | Plan local exit 0; tarefa preservada após recriar API |
| Publicação por digest | GHCR e job publish | Pendente de comprovação no repositório da fase 2 |
| Deploy por digest | scripts/deploy.sh e job deploy-demo | Pendente de validação com imagem publicada |
| Rollback, quando houver duas versões compatíveis | scripts/rollback.sh | Script preparado; demonstração condicionada à disponibilidade de dois digests compatíveis |
| Falha controlada | PR #1 original: execuções 42489956044 e 42511228720 | Bloqueio demonstrado; teste intencional removido e correção aprovada |
| Fluxograma completo | docs/fluxograma.mmd e exportações em entrega-fase2/previa-validacao/ | Fonte, SVG, PNG, visão geral e cinco ampliações já preparados; revisar com o resultado final do CD |
| Relatório das fases 1/2, resultados e melhorias | docs/relatorio-fase2.md | Texto e resultados históricos preparados; incorporar os fatos do novo CD |
| PDF final no template | entrega-fase2/ | Prévia de 24 páginas disponível; versão final pendente |
| Tag fase-2 | Commit verificado no repositório da fase 2 | Pendente após validação completa |

O enunciado requer o pipeline de CD, os containers/scripts, o fluxograma,
a demonstração e o relatório final em PDF com resultados e melhorias.
O guia acrescenta tag, persistência, falha controlada e rollback quando houver
duas versões compatíveis. A ausência de uma segunda imagem não será substituída
por evidência inventada. Não há limpeza automática de dados locais; o deploy
no runner é temporário.
