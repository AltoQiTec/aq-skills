# AQ Skills

Coleção de **Agent Skills** para o Claude que trabalham sobre os softwares da **AltoQi** por meio dos seus servidores **MCP**. Cada skill ensina o Claude a executar uma tarefa de engenharia ou de gestão de ponta a ponta: auditar um projeto contra a norma, gerar um relatório, conciliar o orçamento com o modelo, distribuir tarefas para a equipe e assim por diante.

São **95 skills**, escritas em português (pt-BR), organizadas por produto:

| Pasta | Produto / MCP | Área | Skills |
|---|---|---|---|
| [`skills/arquivos/`](skills/arquivos) | AltoQi Visus Collab | Compatibilização BIM, apontamentos BCF | 8 |
| [`skills/builder/`](skills/builder) | AltoQi Builder | Instalações prediais (elétrica, hidráulica, esgoto, gás, PCI, SPDA) | 18 |
| [`skills/eberick/`](skills/eberick) | AltoQi Eberick | Projeto estrutural em concreto armado | 19 |
| [`skills/fluxos-integrados/`](skills/fluxos-integrados) | Dois ou mais MCPs | Fluxos que cruzam produtos | 8 |
| [`skills/orçamento/`](skills/orçamento) | AltoQi Visus (Orçamento / Custos) | Orçamento, quantificação BIM e planejamento 4D | 16 |
| [`skills/processos/`](skills/processos) | AltoQi Visus Processos (Workflow) | Gestão de tarefas, horas e portfólio | 26 |

## Pré-requisitos

- Um ambiente do Claude com suporte a Skills: claude.ai, Claude Desktop (Cowork) ou Claude Code.
- O **conector MCP** do produto AltoQi que a skill usa, conectado e autenticado.
- Para skills do Eberick, do Builder e do Visus: o **projeto aberto** no software, já que a maioria lê o projeto ativo.
- Para os **fluxos integrados**: todos os MCPs envolvidos conectados ao mesmo tempo (veja a coluna "MCPs" na tabela).

## Instalação

### claude.ai e Claude Desktop

1. Baixe o arquivo `.skill` desejado desta repo.
2. Nas configurações do Claude, abra a seção de **Skills** e use a opção de enviar (upload) uma skill.
3. Selecione o arquivo `.skill`. A skill passa a ser acionada automaticamente quando o pedido combinar com a descrição dela.

### Claude Code

Um arquivo `.skill` é um `.zip` com uma pasta `<nome-da-skill>/SKILL.md` dentro. Basta extrair em `~/.claude/skills/` (todas as sessões) ou em `.claude/skills/` dentro de um projeto (só nele). Os comandos abaixo rodam a partir da raiz desta repo:

```bash
git clone https://github.com/AltoQiTec/aq-skills.git && cd aq-skills
```

Crie a pasta de destino antes, porque o `unzip` não cria `~/.claude/` quando ela ainda não existe:

```bash
mkdir -p ~/.claude/skills
```

Para instalar uma skill:

```bash
unzip -o skills/builder/builder-mapa-de-erros.skill -d ~/.claude/skills/
```

Para instalar todas de uma vez:

```bash
for f in skills/*/*.skill; do unzip -o "$f" -d ~/.claude/skills/; done
```

Os dois arquivos que não estão no formato `.skill`:

```bash
mkdir -p ~/.claude/skills/alocacao-de-recursos && cp skills/processos/processos-alocacao-de-recursos.md ~/.claude/skills/alocacao-de-recursos/SKILL.md
```

```bash
unzip -o skills/eberick/eberick-relatorio-avaliacao-do-projeto.zip 'skills/*' -d /tmp/atp && cp -R /tmp/atp/skills/eberick-relatorio-atp ~/.claude/skills/
```

Depois de instalar, abra uma sessão nova do Claude Code. Cada skill também vira um comando: digite `/` e procure pelo nome, por exemplo `/builder-mapa-de-erros`. No Claude Code o nome vem da pasta, então a skill de ATP aparece como `/eberick-relatorio-atp`.

## Modo de operação

Cada skill declara na descrição se só lê dados ou se altera algo no software:

- **Leitura**: consulta o MCP e devolve análise, relatório ou arquivo (HTML, Word, Excel, PDF). Não muda nada no projeto.
- **Grava**: altera o projeto ou a base (troca peças, reprocessa, cria tarefas, lança custos). Toda skill que grava mostra uma prévia e pede confirmação explícita antes.
- **Interativo**: muda só o estado da tela (visão ativa, seleção no 3D), sem alterar dados.

## Catálogo

### Visus Collab ([`skills/arquivos/`](skills/arquivos))

| Skill | O que faz | Modo |
|---|---|---|
| `collab-comparar-obras` | Compara várias obras lado a lado: apontamentos abertos, vencidos, taxa de resolução, disciplina e prioridade | Leitura |
| `collab-inventario-modelos` | Lista os modelos IFC por disciplina e revisão, aponta disciplinas faltando e revisões desatualizadas | Leitura |
| `collab-pauta-reuniao` | Pauta da reunião de compatibilização com snapshot, histórico, responsável e link do viewer de cada apontamento | Leitura |
| `collab-pendencias-por-projetista` | Pendências de um projetista, empresa ou disciplina em todas as obras, com mensagem de cobrança pronta | Leitura |
| `collab-pontos-quentes` | Onde os conflitos se concentram: por tipo, por par de disciplinas e por local | Leitura |
| `collab-qualidade-apontamentos` | Audita os apontamentos: sem responsável, sem prazo, descrição vaga, sem imagem, duplicados | Leitura |
| `collab-registro-decisoes` | Registro das decisões de projeto tiradas das conversas dos apontamentos | Leitura |
| `collab-tempo-de-resposta` | Tempo até a primeira resposta e até a resolução, por disciplina e responsável | Leitura |

### Builder ([`skills/builder/`](skills/builder))

| Skill | O que faz | Modo |
|---|---|---|
| `builder-analise-spda` | Analisa o SPDA pela NBR 5419 (partes 1 a 4): nível de proteção, captação, descidas, aterramento, DPS | Leitura |
| `builder-auditoria-gas` | Audita a instalação de gás (GLP ou GN) pela NBR 15526 | Leitura |
| `builder-auditoria-incendio` | Audita o PCI: prumadas, reserva técnica, hidrantes e itens das ITs do Corpo de Bombeiros | Leitura |
| `builder-balanceamento-fases` | Carga por fase e desequilíbrio de cada quadro; opcionalmente reprocessa com balanceamento automático | **Grava** |
| `builder-conferir-premissas` | Confere as premissas de cálculo contra o padrão do escritório ou o caderno do cliente | Leitura |
| `builder-corrigir-dimensionamento` | Localiza elementos com erro, propõe a peça certa do cadastro, aplica a troca e reprocessa | **Grava** |
| `builder-diagrama-unifilar` | Desenha o unifilar esquemático de um quadro em HTML/SVG | Leitura |
| `builder-enxugar-superdimensionados` | Reduz condutos superdimensionados para a menor peça que atende | **Grava** |
| `builder-mapa-de-erros` | Varredura de erros e avisos em todas as disciplinas e pavimentos, em mapa disciplina × pavimento | Leitura |
| `builder-memorial-calculo` | Memorial de cálculo de um sistema ou trecho (barrilete, coluna, alimentador, quadro) | Leitura |
| `builder-memorial-descritivo` | Memorial descritivo guiado por disciplina, entregue em .docx ou .pdf | Leitura |
| `builder-nbr5410-analise` | Auditoria completa da elétrica pela NBR 5410 (e NBR 16690), com relatório HTML | Leitura |
| `builder-nbr5626-analise` | Auditoria da hidráulica (água fria e quente) pela NBR 5626:2020 | Leitura |
| `builder-nbr8160-nbr17076-analise` | Auditoria do esgoto sanitário pela NBR 8160 e NBR 17076:2024 | Leitura |
| `builder-quadro-de-cargas-exportar` | Exporta o quadro de cargas para Excel, um quadro por aba | Leitura |
| `builder-quantitativo-pecas` | Quantitativo das peças modeladas por disciplina, pavimento e tipo | Leitura |
| `builder-resumo-executivo` | One-pager das instalações com semáforo por disciplina | Leitura |
| `builder-trocar-linha-de-pecas` | Troca em lote a linha ou o material de um sistema (ex.: PVC para PPR) e reprocessa | **Grava** |

### Eberick ([`skills/eberick/`](skills/eberick))

| Skill | O que faz | Modo |
|---|---|---|
| `eberick-agente-revisao` | Revisão técnica assistida: investiga resultados, erros e avisos, consulta a Ajuda oficial e gera relatório HTML | Leitura |
| `eberick-analise-combinacoes` | Combinações de ações (ELU/ELS), coeficientes γf e ψ e imperfeições globais pela NBR 6118:2023 | Leitura |
| `eberick-analise-vento` | Ações do vento, deslocamentos (H/1700), conforto e frequência natural (NBR 6123) | Leitura |
| `eberick-auditoria-padrao-escritorio` | Audita o projeto contra o padrão interno do escritório | Leitura |
| `eberick-cargas-para-geotecnico` | Planilha Excel de cargas nas fundações por pilar, para o consultor geotécnico | Leitura |
| `eberick-checklist-conformidade` | Checklist normativo em Word preenchido com os dados do Eberick | Leitura |
| `eberick-classe-consequencia` | Enquadra o projeto em CC1, CC2 ou CC3 (NBR 6118:2026) e diz se a ATP é exigida | Leitura |
| `eberick-durabilidade` | Classe de agressividade, fck e cobrimentos contra a NBR 6118 e o ambiente real da obra | Leitura |
| `eberick-estabilidade-global` | γz, P-Δ, imperfeições e deslocamentos por vento; nós fixos ou móveis | Leitura |
| `eberick-explicar-elemento` | Explica como um pilar, viga, laje, sapata ou bloco foi dimensionado e o que governou o resultado | Leitura |
| `eberick-flechas-lajes` | Lajes com flecha acima do limite, por pavimento | Leitura |
| `eberick-lancamento-estrutural` | Lança pilares e vigas no pavimento ativo a partir de comandos ou de uma planta anexada | **Grava** |
| `eberick-linha-de-base` | Salva um retrato dos indicadores do projeto e compara revisões | Leitura |
| `eberick-relatorio-avaliacao-do-projeto` | Relatório formal de Avaliação Técnica de Projeto (ATP) em Word, com parecer final (pacote de plugin `.zip`) | Leitura |
| `eberick-relatorio-fundacoes-eberick` | Relatório visual de fundações (cargas, momentos, mapa de esforços) em HTML | Leitura |
| `eberick-relatorio-materiais` | Relatório Word de consumo e eficiência de materiais, com benchmarks | Leitura |
| `eberick-resumo-executivo` | One-pager estrutural em Word com indicadores e semáforos | Leitura |
| `eberick-taxa-compressao-fundacoes` | Uniformização da taxa de compressão (ni) de sapatas e blocos | Leitura |
| `eberick-taxa-compressao-pilares` | Taxa de compressão (ni); a versão atual cobre só o pavimento de fundação | Leitura |

### Fluxos integrados ([`skills/fluxos-integrados/`](skills/fluxos-integrados))

| Skill | MCPs | O que faz | Modo |
|---|---|---|---|
| `apontamentos-para-tarefas` | Collab → Processos | Cada apontamento aberto vira uma tarefa com responsável, prazo e link do viewer, sem duplicar | **Grava** |
| `conciliar-estrutura-orcamento` | Eberick × Visus | Concilia concreto, aço e fôrma da estrutura com as quantidades do orçamento | Leitura |
| `conciliar-instalacoes-orcamento` | Builder × Visus | Concilia peças e comprimentos das instalações com o orçamento | Leitura |
| `cronograma-para-tarefas` | Visus → Processos | Leva as atividades do cronograma 4D para tarefas de acompanhamento e medição | **Grava** |
| `erros-para-tarefas` | Builder → Processos | Agrupa os erros de dimensionamento e cria tarefas de correção por disciplina e pavimento | **Grava** |
| `gate-de-emissao` | Eberick + Builder + Collab + Processos | Parecer de emissão: pronto, pronto com ressalvas ou não pronto | Leitura |
| `interfaces-estrutura-instalacoes` | Eberick × Builder | Confere níveis, reservatórios e itens de interface entre estrutura e instalações | Leitura |
| `investigar-apontamento` | Collab + Eberick/Builder | Busca no projeto a resposta técnica de um apontamento e redige a resposta | Leitura |

### Visus Orçamento ([`skills/orçamento/`](skills/orçamento))

| Skill | O que faz | Modo |
|---|---|---|
| `orcamento-alerta-estouros` | Varre as atividades em busca de estouros de custo e desvios, com ranking por gravidade | Leitura |
| `orcamento-analise-quantitativos` | Quantidades por frente e serviço e índices como concreto e aço por m² | Leitura |
| `orcamento-atualizar-dados-obra` | Atualiza o cadastro da obra (cliente, área, BDI, CUB) mostrando antes e depois | **Grava** |
| `orcamento-auditoria-padrao` | Audita o orçamento contra o padrão interno do escritório | Leitura |
| `orcamento-avanco-fisico-financeiro` | Valor agregado (EVM): CPI, SPI, EAC e VAC | Leitura |
| `orcamento-curva-s` | Curva S, desembolso por período e pico de gasto | Leitura |
| `orcamento-gerenciar-calendario` | Feriados, pontos facultativos e jornadas reduzidas do planejamento | **Grava** |
| `orcamento-lancar-avanco-fisico` | Registra o percentual de avanço físico das atividades 4D | **Grava** |
| `orcamento-lancar-custo-executado` | Registra o custo executado das atividades 4D | **Grava** |
| `orcamento-localizar-no-modelo` | Destaca e enquadra no 3D os elementos de uma frente da EAP ou atividade | Interativo |
| `orcamento-navegar-visoes` | Lista, troca e analisa as visões do projeto | Interativo |
| `orcamento-pronto-para-licitacao` | Checklist antes da licitação: itens sem preço, rastreabilidade BIM, BDI, data-base | Leitura |
| `orçamento-painel-financeiro` | Painel HTML com custo com e sem BDI, custo por m², EAP e curva ABC | Leitura |
| `orçamento-regra-quantificacao` | Cria e edita regras de quantificação do modelo IFC | **Grava** |
| `orçamento-replanejamento` | Simula ou aplica mudanças no cronograma 4D e mostra o impacto em cascata | **Grava** |
| `orçamento-revisar-regras` | Audita as regras de quantificação: entidades sem regra, quantidades zeradas, duplicidade | Leitura |

### Visus Processos / Workflow ([`skills/processos/`](skills/processos))

| Skill | O que faz | Modo |
|---|---|---|
| `processos-acuracia-de-estimativa` | Horas estimadas × realizadas por tipo de tarefa, fase, entrega e pessoa | Leitura |
| `processos-alocacao-de-recursos` | Board interativo de carga de trabalho por pessoa e semana, em HTML (arquivo `.md`) | Leitura |
| `processos-analise-de-gargalo` | Onde o fluxo trava e qual a causa raiz dos atrasos | Leitura |
| `processos-apontamento-horas` | Cronômetro e lançamento de horas a partir de uma descrição livre do dia | **Grava** |
| `processos-controle-de-escopo` | Horas fora do escopo por empreendimento e pedido de aditivo; marca tarefas | **Grava** |
| `processos-criar-tarefas-em-lote` | Transforma ata, lista, escopo ou planilha em tarefas, com pré-visualização | **Grava** |
| `processos-encerrar-empreendimento` | Checklist de pendências e encerramento de planejadores, processos e empreendimento | **Grava** |
| `processos-follow-up-comercial` | Tarefas comerciais ligadas a oportunidades: abertas, vencidas, paradas | Leitura |
| `processos-horas-por-cliente` | Horas por cliente e tipo de entrega, para cobrança e rentabilidade | Leitura |
| `processos-horas-por-colaborador` | Timesheet por pessoa e projeto para o fechamento mensal | Leitura |
| `processos-montar-empreendimento` | Cria empreendimento, planejadores, processos, fases e equipe a partir de templates | **Grava** |
| `processos-padronizar-processos` | Compara processos com um modelo e alinha as divergências | **Grava** |
| `processos-projetos-sem-gestao` | Empreendimentos sem dono e tarefas atrasadas sem responsável | Leitura |
| `processos-qualidade-do-cadastro` | Tarefas sem responsável, prazo, estimativa, fase ou etiqueta | Leitura |
| `processos-redistribuir-tarefas` | Repassa em lote as tarefas de uma pessoa conforme a carga da equipe | **Grava** |
| `processos-relatorio-empreendimento` | % concluído, % atrasado e horas por empreendimento | Leitura |
| `processos-relatorio-planejadores` | Planejadores ativos com fases e datas de todos os empreendimentos | Leitura |
| `processos-reprogramar-prazos` | Desloca em lote as datas quando um marco atrasa ou antecipa | **Grava** |
| `processos-retrabalho-ranking` | Ranking de retrabalho por pessoa | Leitura |
| `processos-risco-de-portfolio` | Empreendimentos classificados por risco de atraso (crítico, alto, moderado) | Leitura |
| `processos-tarefas-em-atraso` | Tarefas vencidas agrupadas por responsável | Leitura |
| `processos-tarefas-paradas-em-revisao` | Fila de revisão e tarefas paradas há dias | Leitura |
| `processos-tarefas-por-responsavel` | Tarefas de uma pessoa num período | Leitura |
| `processos-tendencia-de-atraso` | Análise preditiva dos projetos que tendem a atrasar | Leitura |
| `processos-to-do-diario` | O que vence hoje e amanhã, para a daily | Leitura |
| `processos-to-do-semanal` | Tarefas da semana, para o planning | Leitura |

## Estrutura do repositório

```
.
├── README.md
├── AGENTS.md            # guia para agentes de IA que editam esta repo
├── CLAUDE.md            # importa o AGENTS.md (Claude Code)
└── skills/
    ├── arquivos/            # Visus Collab
    ├── builder/             # Builder
    ├── eberick/             # Eberick
    ├── fluxos-integrados/   # cruzam dois ou mais produtos
    ├── orçamento/           # Visus Orçamento / Custos
    └── processos/           # Visus Processos (Workflow)
```

## Anatomia de uma skill

Um arquivo `.skill` é um `.zip` com esta estrutura:

```
builder-analise-spda/
├── SKILL.md              # obrigatório: frontmatter + instruções
└── references/           # opcional: normas, critérios, material de apoio
    └── norma-nbr5419.md
```

Algumas skills também trazem `scripts/` (Python para gerar gráficos e relatórios), `templates/` e `assets/` (HTML base, imagens).

O `SKILL.md` começa com um frontmatter YAML:

```yaml
---
name: builder-balanceamento-fases
description: "Analisa e corrige o balanceamento de fases dos quadros elétricos ... GRAVA no projeto (reprocessamento). Use quando o usuário pedir \"balanceamento de fases\", ... Para auditoria completa da elétrica use builder-nbr5410-analise."
---
```

A `description` é o que o Claude lê para decidir quando acionar a skill. Por isso ela diz o que a skill faz, se lê ou grava, as frases típicas do usuário e qual skill usar nos casos vizinhos.

Para ver o conteúdo de uma skill sem extrair:

```bash
unzip -p skills/builder/builder-analise-spda.skill '*/SKILL.md'
```

## Contribuindo

As convenções de nome, formato, descrição e empacotamento estão em [AGENTS.md](AGENTS.md). Em resumo:

1. Use o prefixo do produto no nome (`builder-`, `eberick-`, `collab-`, `orcamento-`, `processos-`) e mantenha o `name` do frontmatter igual ao nome do arquivo.
2. Na descrição, diga se a skill é **Só leitura** ou **GRAVA**, liste as frases que a acionam e aponte as skills vizinhas.
3. Toda skill que grava precisa mostrar prévia e pedir confirmação antes de alterar o projeto.
4. Empacote como `.zip` com a pasta da skill na raiz e renomeie para `.skill`.
5. Atualize o catálogo deste README.

## Aviso

As skills auxiliam a análise e a documentação, mas não substituem a verificação do engenheiro responsável. Confira os resultados antes de emitir projetos, relatórios ou pareceres.
