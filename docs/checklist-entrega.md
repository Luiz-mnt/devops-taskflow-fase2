# Checklist de entrega — fase 2 com CD e laboratório verificados

A fase 1 permanece no [repositório original](https://github.com/Luiz-mnt/devops-taskflow),
com sua main, tag fase-1 e PR #1 preservados. A fase 2 foi publicada em
[devops-taskflow-fase2](https://github.com/Luiz-mnt/devops-taskflow-fase2).
O [CI/CD 42520819347](https://github.com/Luiz-mnt/devops-taskflow-fase2/actions/runs/42520819347) aprovou os seis jobs no commit
`78fe0c236a426ec035a0a2d9daea218f083316a6`. O deploy local consumiu a mesma imagem por digest.

| Requisito | Arquivo / evidência | Situação |
|---|---|---|
| Expandir CI para CD | .github/workflows/ci.yml; evidencias/ci/42520819347/facts.json | Concluído: seis jobs aprovados, incluindo publicação e deploy |
| Dockerfile não-root | Dockerfile; imagem-publicada/runtime-user.txt | Concluído: build aprovado e processo UID/GID 10001 |
| Orquestração e scripts de deploy | compose.yaml; scripts/*.sh; logs do runner e locais | Concluído: deploy e smoke pelo digest sem rebuild |
| Monitoramento e logs | app/observability.py; monitoring/; registros do runner e locais | Concluído: saúde SQLite, métricas, JSON e Prometheus UP/up=1 |
| Testes integrados | CI/CD 42520819347 e relatórios preservados | Concluído: 38 testes, 100% de linhas/branches; pytest 1,22 s |
| Segurança | pip-audit; Trivy; gate e relatórios do CD | Concluído: 50 pacotes/zero avisos pip-audit; zero HIGH/CRITICAL corrigíveis; 42 sem correção registrados |
| Infraestrutura e persistência | infra/; evidencias/fase2-local/imagem-publicada/ | Concluído: plan sem mudanças; tarefa id 5 preservada ao recriar API |
| Publicação por digest | GHCR; job publish; image.json do run | Concluído: artefato aprovado publicado e Config ID conferido sem rebuild |
| Acesso à imagem | imagem-publicada/pull-anonimo.log | Concluído: pacote público e pull anônimo real aprovado |
| Deploy por digest | job deploy-demo e imagem-publicada/deploy.log | Concluído: runner efêmero e laboratório local aprovados |
| Rollback, se houver duas versões compatíveis | scripts/rollback.sh | Concluído: dois digests compatíveis, smoke e persistência após reversão |
| Falha controlada | PR #1 original: execuções 42489956044 e 42511228720 | Concluído: bloqueio demonstrado; teste intencional removido e correção aprovada |
| Fluxograma completo | docs/fluxograma.mmd; Mermaid/SVG/PNG e ampliações do pacote | Concluído: Mermaid/SVG/PNG e ampliações representam todo o fluxo |
| Relatório das fases 1/2, resultados e melhorias | docs/relatorio-fase2.md | Concluído: resultados históricos, CD completo, laboratório, limites e melhorias registrados |
| PDF no template | entrega-fase2/ e validação do pacote | Concluído: PDF final de 26 páginas, template preservado, 114 campos conferidos e 15 links validados |
| Referência fase-2 | [Tag fase-2](https://github.com/Luiz-mnt/devops-taskflow-fase2/tree/fase-2), commit 78fe0c236a426ec035a0a2d9daea218f083316a6 | Concluído: tag publicada no código verificado; evidências posteriores documentadas em main |

Imagem validada: `ghcr.io/luiz-mnt/devops-taskflow-fase2@sha256:d25cad8350c946d5a3c1f646c1309be2fd1b6d82d1e7f83bd9924d90d8c4708d`.

O enunciado exige CD, containers/scripts, fluxograma, demonstração e PDF
com resultados e melhorias. O guia acrescenta tag, persistência, falha
controlada e rollback se houver duas versões compatíveis. A publicação natural
das evidências gerou uma segunda imagem; a compatibilidade foi conferida e o rollback demonstrado. O laboratório
local preserva os dados; a limpeza automática ficou restrita ao runner.
