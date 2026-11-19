# CLAUDE.md

As instruções do projeto ficam no AGENTS.md, compartilhado com outros agentes. Leia-o antes de qualquer alteração:

@AGENTS.md

## Notas específicas do Claude Code

- Para testar uma skill localmente, extraia o pacote em `.claude/skills/` de uma pasta de teste fora desta repo (ou em `~/.claude/skills/`) e abra uma sessão nova; não extraia dentro de `skills/`.
- As skills dependem dos MCPs da AltoQi. Sem o conector do produto ativo na sessão, só é possível revisar o texto, não executar a skill.
- Ao terminar de editar pacotes, rode o comando de inspeção do AGENTS.md para confirmar que cada `.skill` ainda tem `SKILL.md` dentro da pasta raiz.
