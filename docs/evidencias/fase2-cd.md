# CI/CD completo da fase 2

Execução real: https://github.com/Luiz-mnt/devops-taskflow-fase2/actions/runs/42520819347
Commit: `78fe0c236a426ec035a0a2d9daea218f083316a6`. Tag: `fase-2`.
Repositório independente: https://github.com/Luiz-mnt/devops-taskflow-fase2

Os seis jobs foram aprovados, incluindo publicação, deploy temporário, coleta e limpeza.
Imagem: `ghcr.io/luiz-mnt/devops-taskflow-fase2@sha256:d25cad8350c946d5a3c1f646c1309be2fd1b6d82d1e7f83bd9924d90d8c4708d`.
Config ID aprovado, publicado e conferido localmente: `sha256:e7dc805de61db984319ecef80b03afdb954900b5ae936b39bf7be4eee69edb8f`.

38 testes aprovados, cobertura de linhas/branches 100%; pytest 1.22 s;
JUnit 1.23 s. Pip-audit: 50 pacotes, zero vulnerabilidades identificadas.
Trivy: zero HIGH/CRITICAL corrigíveis; 40 HIGH e 2 CRITICAL sem correção informada.
Essa política não equivale a ausência de vulnerabilidades.

Em `ci/42520819347/` estão os metadados, log integral e relatórios de testes, segurança,
infraestrutura, publicação e deploy. O deploy confirmou saúde, CRUD, métricas e Prometheus UP.
Rede e volume dos runners foram removidos; o laboratório local foi preservado.
O pacote é público e o pull anônimo foi testado com configuração Docker vazia.

A tag congela o código validado. `main` reúne os registros produzidos depois da execução.
As provas históricas de PR/falha controlada continuam vinculadas ao repositório original;
esse PR não foi integrado. O repositório, main e tag fase-1 originais não foram modificados.
