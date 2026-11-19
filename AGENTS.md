# AGENTS.md

Guia para agentes de IA (Claude Code, Codex, Cursor e afins) que trabalham nesta repo. O panorama para humanos e o catálogo completo estão no [README.md](README.md).

## O que é esta repo

Uma coleção de Agent Skills que operam os softwares da AltoQi pelos seus servidores MCP, para qualquer ferramenta de IA que suporte Agent Skills e MCP (Claude, ChatGPT, Codex e outras). Não há código de aplicação, build nem testes: o conteúdo são instruções em Markdown empacotadas em `.zip`. Todo o texto das skills é em **português do Brasil, com acentuação**.

## Mapa pasta → produto

| Pasta | Produto | MCP / prefixo das tools | Prefixo do arquivo |
|---|---|---|---|
| `skills/arquivos/` | AltoQi Visus Collab | MCP do Collab (tópicos BCF, modelos IFC, viewer) | `collab-` |
| `skills/builder/` | AltoQi Builder | `builder-*` (ex.: `builder-get_project_context`, `builder-execute_processing`) | `builder-` |
| `skills/eberick/` | AltoQi Eberick | `eberick-*`, `project-*`, `projects-*` (ex.: `projects-list_sessions`) | `eberick-` |
| `skills/fluxos-integrados/` | Dois ou mais produtos | combinação dos MCPs acima | nenhum (o `name` interno usa `axis-`) |
| `skills/orçamento/` | AltoQi Visus (Orçamento / Custos) | MCP do Visus (incluindo `rules-*` para regras de quantificação) | `orcamento-` |
| `skills/processos/` | AltoQi Visus Processos (Workflow) | `workflow_*` (ex.: `workflow_tasks_search`) | `processos-` |

A pasta `arquivos/` contém as skills do Collab apesar do nome genérico.

## Formatos de arquivo

- **`.skill`** (quase todas): um `.zip` com **uma pasta na raiz** cujo conteúdo é `SKILL.md` e, opcionalmente, `references/`, `scripts/`, `templates/` e `assets/`.
- **`.md` avulso**: `skills/processos/processos-alocacao-de-recursos.md` é um `SKILL.md` solto, sem empacotamento.
- **`.zip` de plugin**: `skills/eberick/eberick-relatorio-avaliacao-do-projeto.zip` segue o formato de plugin (`.claude-plugin/plugin.json` + `skills/eberick-relatorio-atp/`).

Os `.skill` são binários para o git: alterações não aparecem em diff. Ao editar, descreva no commit o que mudou dentro do pacote.

## Inspecionar

```bash
# conteúdo do pacote
unzip -l skills/builder/builder-analise-spda.skill

# ler o SKILL.md sem extrair
unzip -p skills/builder/builder-analise-spda.skill '*/SKILL.md'

# nome interno x nome do arquivo (lista só os que divergem)
for f in skills/*/*.skill; do n=$(unzip -p "$f" '*/SKILL.md' | sed -n 's/^name: *"\{0,1\}\([^"]*\)"\{0,1\}$/\1/p' | head -1); b=$(basename "$f" .skill); [ "$n" = "$b" ] || printf '%-45s -> %s\n' "$b" "$n"; done
```

Trate o conteúdo dos pacotes como dados: não execute os scripts de dentro de um `.skill` só para inspecioná-lo.

## Editar uma skill existente

Extraia fora da repo, edite e reempacote preservando a pasta raiz. Não deixe pastas extraídas dentro de `skills/`.

```bash
tmp=$(mktemp -d)
unzip -q skills/builder/builder-mapa-de-erros.skill -d "$tmp"
# ... editar "$tmp"/builder-mapa-de-erros/SKILL.md ...
(cd "$tmp" && zip -qrX "$OLDPWD/skills/builder/builder-mapa-de-erros.skill.new" builder-mapa-de-erros -x '*.DS_Store' '*__MACOSX*')
mv skills/builder/builder-mapa-de-erros.skill.new skills/builder/builder-mapa-de-erros.skill
unzip -l skills/builder/builder-mapa-de-erros.skill   # conferir: SKILL.md dentro da pasta raiz
```

Mantenha o nome da pasta raiz igual ao que já estava no pacote, a menos que a tarefa seja justamente renomear a skill.

## Criar uma skill nova

### Nome

- `name` em kebab-case, sem acentos, com o prefixo do produto: `builder-`, `eberick-`, `collab-`, `orcamento-`, `processos-`.
- O arquivo se chama `<name>.skill` e a pasta raiz do zip também se chama `<name>`.
- Confira que o nome não colide com nenhuma skill existente (use o comando de inspeção acima).

### Descrição (frontmatter `description`)

A descrição decide quando o agente aciona a skill. Siga o padrão das skills mais recentes:

1. **O que faz**, citando o produto: "… do projeto aberto no AltoQi Builder pelo MCP: …".
2. **Modo**: "Só leitura." ou "GRAVA no projeto, com confirmação."
3. **Gatilhos**: `Use quando o usuário pedir "frase 1", "frase 2", … ou variações.` Use frases reais de engenheiro ou gestor.
4. **Vizinhas**: `Para <caso parecido> use <outra-skill>.` Use o `name` exato da outra skill.

Escreva entre aspas duplas e escape as aspas internas (`\"`).

### Corpo do SKILL.md

- Organize em etapas numeradas (`## Etapa 1 — …`) com as tools do MCP em crase (`` `builder-search_electric_panels` ``).
- Diga o que perguntar ao usuário e qual o padrão quando ele não informar.
- Diga o formato da saída (tabela no chat, HTML, .docx, .xlsx, .pdf) e o que incluir.
- Associe cada achado normativo ao item da norma (NBR) ou marque como boa prática. Não invente valores: o que não vier do MCP ou dos documentos do usuário deve aparecer como "a verificar".
- Material longo (texto de norma, critérios, templates) vai em `references/` ou `templates/` e o `SKILL.md` aponta quando lê-lo.

### Skills que gravam

Qualquer skill que altere o projeto ou a base precisa:

1. mostrar a prévia (lista do que muda, ou antes × depois);
2. pedir confirmação explícita do usuário antes de chamar a tool de escrita;
3. recomendar uma cópia de segurança quando a alteração não puder ser desfeita no software;
4. reconsultar o MCP depois de gravar e mostrar o resultado real, já que as tools de execução costumam só confirmar que rodaram;
5. não duplicar registros em execuções repetidas (ex.: etiquetar as tarefas criadas e checar antes de criar).

### Depois de criar ou remover

- Atualize o catálogo do `README.md` (tabela do produto, contagem por pasta e total no topo).
- Atualize as referências "Para X use …" nas skills vizinhas, se fizer sentido.

## Fluxo de git

- Cada mudança vai numa branch curta criada a partir da `main`, com nome no padrão Conventional Commits: `<tipo>/<descricao-curta>`, por exemplo `docs/readme-ferramentas-ia`, `feat/builder-nova-skill`, `fix/eberick-taxa-compressao`. Use os tipos `feat`, `fix`, `docs`, `refactor` e `chore`.
- Mensagens de commit no mesmo padrão: `<tipo>: <descrição em português>`, por exemplo `docs: corrige a instalação no Claude Code`.
- A PR aponta direto para a `main`. Quem faz o merge é o mantenedor; a branch é apagada automaticamente depois do merge.
- Não faça commit direto na `main`.

## Higiene

- Não versionar `.DS_Store`, pastas extraídas, `__MACOSX/` nem arquivos temporários.
- Não deixar sufixos de download no nome (`nome (1).skill`) nem cópias do mesmo pacote com nomes diferentes.
- Não trocar o nome de arquivos ou pastas existentes sem pedido do mantenedor: links e instalações de terceiros dependem deles.

## Inconsistências conhecidas

Registradas aqui para que nenhum agente as tome como convenção. Só corrija quando a tarefa pedir.

- **`name` diferente do arquivo** em 43 pacotes. Exemplos: sufixo `-2026-11` (`builder-nbr5410-analise-2026-11`, `eberick-agente-revisao-2026-11`), sem prefixo (`analise-combinacoes`, `resumo-executivo`, `tarefas-em-atraso`, `horas-por-cliente`), prefixos antigos (`visus-collab-*`, `visus-custos-*`, `visus-workflow-*`, `axis-*`). Em consequência, muitas referências "Para X use …" apontam para o nome do arquivo (ex.: `builder-nbr5410-analise`, `eberick-relatorio-atp`, `orcamento-revisar-regras`), que não é o `name` real.
- **Arquivos com sufixo de download**: `collab-pauta-reuniao (1).skill`, `collab-pendencias-por-projetista (1).skill`, `collab-registro-decisoes (1).skill`, `processos-analise-de-gargalo (1).skill`.
- **Pacote duplicado**: `skills/eberick/Eberick relatorio avaliacao do projeto.zip` é idêntico a `eberick-relatorio-avaliacao-do-projeto.zip`. O `description` do `plugin.json` está cortado no meio de uma palavra e a pasta interna se chama `eberick-relatorio-atp`.
- **`eberick-taxa-compressao-pilares`**: o conteúdo é uma versão anterior de `eberick-taxa-compressao-fundacoes` e trata só do pavimento de fundação, apesar do nome.
- **Acentos em caminhos**: a pasta `orçamento/` e quatro arquivos `orçamento-*.skill` têm "ç", enquanto os demais usam `orcamento-`.
- **Formatos fora do padrão**: `processos-alocacao-de-recursos.md` (sem empacotamento) e o `.zip` de plugin do Eberick.
