# Fechamento da fase 1 — concluído em 08/11/2026

A pendência de execução remota foi encerrada. [O CI real](https://github.com/Luiz-mnt/devops-taskflow/actions/runs/42485447573) aprovou os
três jobs no commit `30b49c595e931f1ea6c2e122ba4ca25ead004b48`. Depois da aprovação foi criada a
[tag fase-1](https://github.com/Luiz-mnt/devops-taskflow/tree/fase-1), sem mover outra referência.

32 testes e cobertura de linhas/branches de 100% foram medidos no runner. O
Terraform remoto validou os scripts, criou dois recursos, inspecionou-os e
confirmou o segundo plano sem alterações. A limpeza removeu somente os recursos
efêmeros do runner. O laboratório local permanece preservado.

Os documentos finais em main consolidam as evidências depois da execução; a tag
mantém o commit verificado da implementação. Não mova a tag para acompanhar
atualizações documentais. A fase 2 dá continuidade à API no repositório
separado [devops-taskflow-fase2](https://github.com/Luiz-mnt/devops-taskflow-fase2),
preservando a main, a tag e o PR #1 originais. Sua própria referência será
criada depois da validação da fase 2.

As evidências permanentes estão em `docs/evidencias/ci/42485447573/`. O documento
da disciplina é o PDF/PPTX final em `../entrega-fase1/`, com os links da execução,
tag e scripts. O [checklist](checklist-entrega.md) mapeia todos os requisitos.

Para regenerar a apresentação local após uma revisão documental legítima:

```powershell
python entrega-fase1/ferramentas/gerar_entrega.py
```

Execute esse comando na pasta Fase 1, com Pillow/PyMuPDF disponíveis. O conteúdo
é derivado do resumo verificável; resultados não devem ser alterados manualmente
para representar uma execução que não ocorreu.
