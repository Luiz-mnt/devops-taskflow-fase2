# Evidências locais da fase 2

As medições foram feitas no computador do estudante em 08/11/2026. A pasta
inicial de trabalho foi copiada para Fase 2; os testes e o deploy foram conferidos
novamente a partir da pasta final. As execuções ocorreram no workspace antes
da publicação do código da fase 2; não são atribuídas ao SHA da demonstração de
falha controlada nem a um runner GitHub.

- `inicial/`: primeiro build/deploy, smoke e prova de persistência; a tarefa id 2
  continuou no SQLite após recriar a API. Containers diferentes e UID 10001
  foram verificados. Imagem local taskflow-local:phase2-inicial.
- `python-pasta-final/`: Python 3.12.16 no Windows, 38 testes aprovados, 100% de
  linhas (131/131) e branches (8/8). Pytest 5,55 s; JUnit 5,558 s. O tempo de
  processo inclui startup e é outra medição. Ruff e pip check também aprovados.
- `pasta-final/`: deploy/smoke e plan Terraform a partir de Fase 2. O mesmo volume
  taskflow-data e rede taskflow-net permanecem sob responsabilidade Terraform.
- `security/`: auditoria de dependências e inventário da imagem local inicial.

O primeiro teste Windows registrou 2,64 s; a verificação posterior na pasta final
registrou 5,55 s. Não representam comparação controlada de desempenho. Os tempos
do runner serão apresentados separadamente com o commit e URL de sua execução.

O estado Terraform foi copiado explicitamente da fase 1 para dar continuidade
aos recursos. O estado ativo passa a ser o da fase 2; não execute applies
simultâneos nas duas pastas. Não foi feito destroy do laboratório local.

## Validação da imagem pública GHCR

Em `imagem-publicada/`, pull anônimo com DOCKER_CONFIG vazio, deploy pelo digest,
smoke, saúde, métricas, logs, alvo UP/consulta up=1 e persistência após recriar a API.
A rede e o volume existentes foram preservados; plan final sem mudanças (exit 0).
As medições antigas deste diretório continuam identificadas como locais/iniciais.
