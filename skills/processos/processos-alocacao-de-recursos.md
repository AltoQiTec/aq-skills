---
name: alocacao-de-recursos
description: >-
  Gera um painel de alocação de recursos ("workload board") interativo, em UM
  arquivo HTML autossuficiente, a partir de dados reais do MCP do Workflow
  (Visus Workflow / AltoQi). Use SEMPRE que o usuário pedir "alocação de
  recurso(s)", um painel/board de carga de trabalho, "workload", "quadro de
  carga da equipe", "painel de alocação por semana", "visualizar quem está
  sobrecarregado/ocioso num calendário", "board estilo ClickUp da equipe", ou
  quiser simular realocação de tarefas arrastando cards. Dispare mesmo que o usuário não diga "HTML" ou
  "workload" explicitamente, desde que ele queira VISUALIZAR carga de trabalho
  por colaborador ao longo de um período. A skill coleta sozinha o período, os
  colaboradores e os limites de tarefas/semana quando o usuário não os informa.
---

# Alocação de Recurso (Visus Workflow / AltoQi)

Este skill monta um painel de carga de trabalho interativo a partir de dados
reais do MCP do Workflow. A entrega é **UM arquivo HTML autossuficiente** (CSS e
JS embutidos; a única dependência externa permitida é a fonte **Lato** via
Google Fonts).

O ponto central: o painel é **exploratório**. O usuário arrasta tarefas entre
pessoas para simular realocações e ver o impacto — nada disso altera o Workflow
real.

---

## PASSO 0 — Coletar os parâmetros (faça isto ANTES de qualquer coisa)

Três parâmetros dependem de quem está usando o skill. Antes de buscar dados,
verifique se o usuário já os informou na mensagem. Para os que faltarem, **use a
ferramenta `AskUserQuestion`** para perguntar — não presuma valores.

Se o usuário já tiver informado tudo na chamada (ex.: "monta o workload de
junho/2026 pra Maria e João, limite 3 e 2"), pule direto para a Parte 1.

Colete, nesta ordem:

### 0.1 — Período
Se o usuário não disse o período, pergunte qual mês/intervalo ele quer analisar
(ex.: "junho/2026"). Aceite tanto um mês inteiro quanto um intervalo de datas.

### 0.2 — Colaboradores a incluir
Se o usuário não listou as pessoas, pergunte **quais colaboradores incluir**.
**Não presuma nem sugira nomes por conta própria** — os nomes devem vir sempre
do usuário ou do MCP do Workflow.

Duas formas de perguntar, em ordem de preferência:

1. **Com dados do MCP:** se possível, use `workflow_users_search` para listar os
   usuários ativos do Workflow e ofereça esses nomes reais como opções. Como
   `AskUserQuestion` é de múltipla escolha, use `multiSelect: true` para o
   usuário marcar quem entra; quem não estiver na lista pode ser adicionado pelo
   campo livre ("Other").
2. **Sem dados do MCP** (ou se a lista for grande demais): peça em texto livre
   que o usuário informe os nomes dos colaboradores que quer no painel.

### 0.3 — Limite de tarefas/semana por colaborador
Se o usuário não informou os limites, **sempre pergunte** o limite semanal de
cada pessoa selecionada (número de tarefas ativas por semana que caracteriza a
capacidade daquela pessoa). Faça isso depois de saber quem entra, para perguntar
apenas pelos colaboradores relevantes.

Boas práticas ao perguntar os limites:
- Se forem poucas pessoas, uma pergunta com opções de valores (ex.: 2, 3, 4, 5)
  por colaborador funciona bem; o usuário pode digitar outro valor em "Other".
- Confirme os valores coletados em texto antes de gerar (ex.: uma linha por
  pessoa: "Maria: 3/sem, João: 4/sem…"), para evitar mal-entendidos.
- **Não use nenhum limite como default silencioso** — os limites variam por
  equipe e devem sempre ser informados/confirmados pelo usuário.

Só avance quando **período + colaboradores + limites** estiverem definidos.

---

## PARTE 1 — Dados (MCP do Workflow)

1. Para cada colaborador selecionado, use `workflow_users_search` para obter o
   `user_id`. **Mostre os IDs encontrados** ao usuário.
2. Use `workflow_tasks_search` filtrando por `assignee_ids` (todos os IDs) e
   `date_start` dentro do período. Campos relevantes: `task_id`, `name`,
   `stage`, `assignees`, `date_start`, `date_end`, `building`. **Liste as
   tarefas** encontradas.
3. Para cada pessoa e cada semana do período (semanas ISO, segunda–domingo),
   conte as tarefas **ATIVAS** — aquelas cujo intervalo (início→término) tem
   sobreposição com a semana. Compare com o limite da pessoa e classifique:
   - **ocioso** (carga < limite)
   - **no limite** (carga = limite)
   - **sobrecarregado** (carga > limite)

   O **status geral no mês** de cada pessoa usa o **pico semanal** (a semana de
   maior carga).
4. Cada card de tarefa deve abrir a tarefa real em nova aba pela URL:
   `https://workflow.altoqivisus.com.br/tasks/{task_id}`

Antes de finalizar, **valide os cálculos**: recontagem por semana, confirmação
de que o status do mês bate com o pico semanal, e coerência das somas dos KPIs.

---

## PARTE 2 — Design System (Visus Workflow)

**Estética:** leve e arejada, muitos espaços em branco, cantos arredondados,
sombras muito suaves, **teal** como cor de marca.

**Tipografia:** fonte "Lato" (Google Fonts), pesos 400/700/900. Números e
rótulos de dados em fonte monoespaçada do sistema. Títulos em peso 900.

**Paleta** — use exatamente estes hex como variáveis CSS:

```css
:root{
  --bg:#F3F4F6;      /* fundo da página, cinza-claro frio */
  --card:#FFFFFF;    /* cards */
  --ink:#3A3D42;     /* texto/títulos, cinza-grafite */
  --ink-2:#6E7378;   /* rótulos */
  --ink-3:#A0A4AB;   /* texto secundário */
  --line:#ECEEF1;    /* bordas */
  --teal:#2EB89C;    /* primária */
  --teal-d:#249C84;
  --teal-l:#D6F1EA;
  --teal-xl:#ECF9F5;
  --amber:#E0A52E;   /* no limite */
  --amber-l:#FBF1DA;
  --coral:#E0654E;   /* sobrecarga */
  --coral-l:#FBE7E1;
  --empty:#B6BBC1;   /* sem alocação */
  --track:#E8EAED;   /* trilho de barras */
}
```

Sombra dos cards: `0 1px 2px rgba(46,52,60,.04), 0 6px 20px rgba(46,52,60,.05)`
Raio: cards ~14px, KPIs ~12px, botões/pills ~8–999px.

**Mapeamento status → cor:** ocioso = teal · no limite = âmbar ·
sobrecarregado = coral · sem alocação = cinza.

**Barra superior do app (chrome):** à esquerda um cubo roxo arredondado com "Wf"
(gradiente `#7d63c9→#5b41a8`), selo pequeno "ALTO QI" e wordmark "Visus
Workflow" ("Visus" leve, "Workflow" 900). À direita um chip de conta: "Você está
na conta / PRODUTO" e um avatar circular teal com um triângulo branco. Abaixo,
breadcrumb "Painéis" e título "Workload — Colaboradores".

---

## PARTE 3 — Layout e componentes

**A) KPIs no topo** (4 cards brancos): ícone de linha teal no topo, rótulo
cinza, número grande peso 900. KPIs: "Tarefas alocadas", "Sobrecargas",
"Pessoas ociosas", "Utilização (%)". Recalculam conforme o escopo.

**B) Card principal** com:
- Título "Carga da equipe · [período]" e, à direita, um **DONUT** pequeno "Carga
  da equipe" mostrando ociosos / no limite / sobrecarga (recalcula).
- Abas com sublinhado teal: **"Workload"** (padrão) e **"Quadro"**. (Não criar
  aba "Capacidade".)
- Barra de ferramentas: alternador **"Semana / Dia"**; navegação por período
  (botões ‹ ›, rótulo tipo "15 – 21 de junho", botão "Hoje"); aviso "Você
  alterou a alocação · restaurar" quando algo é simulado; botão "Restaurar".

**C) Aba WORKLOAD** (calendário estilo ClickUp):
- Coluna esquerda fixa = colaboradores (avatar com iniciais na cor da pessoa,
  nome, papel) + barra de capacidade (carga/limite da semana, colorida por
  status). Colunas = dias (seg–dom) com "Seg 15 / Ter 16…".
- **LINHA RECOLHIDA POR PADRÃO:** cada dia mostra só uma **FAIXA COLORIDA** de
  status (pílula arredondada ~22px) — sem nenhum texto. Dia sem tarefa ou fim de
  semana = célula hachurada (listras diagonais).
- **AO CLICAR na pessoa**, a linha **EXPANDE** e revela os cards de tarefa
  abaixo das faixas (nome da tarefa + datas + #id). Chevron ▸ que gira ao abrir.
  Cabeçalho "Responsáveis" tem botão ⌄/⌃ para expandir/recolher todos.
- Modo **"Dia"**: mostra um único dia (coluna larga), navegação dia a dia.
- **Cards de tarefa:** cartão branco, borda esquerda colorida; clicáveis (abrem
  a URL da tarefa em nova aba, ícone ↗ no hover); arrastáveis. Clique ≠ arrastar
  (distinga drag de click para não abrir ao arrastar).

**D) Aba QUADRO** (kanban): uma coluna por pessoa + coluna "Backlog". Cabeçalho
da coluna com avatar, mini-barra de carga e status. Cards arrastáveis e
clicáveis (mesma URL).

**E) Seção "Insights & Orientação"** (card separado, 3 colunas, recalcula no
escopo):
- **Capacidade do time:** nº de vagas livres, barra de utilização, frase de
  leitura e o "gargalo" (pessoa de menor folga).
- **Possibilidade de alocação:** ranking de quem tem folga (avatar + chip "+N")
  e destaque "Melhor opção agora".
- **Orientação:** recomendações acionáveis ordenadas por severidade (coral =
  sobrecarga → ex. "Mova 1 tarefa de X para Y"; âmbar = no limite; teal = ok;
  cinza = dica de balanceamento entre semanas).

---

## PARTE 4 — Interatividade

- **Arrastar tarefas** entre pessoas (e para o Backlog) para simular realocação;
  ao soltar, recalcula tudo (faixas, barras, KPIs, donut, insights) e a pessoa
  de destino **expande automaticamente**.
- **Ajustar limite** de cada pessoa com botões −/+.
- **"Restaurar"** volta à alocação e limites originais.
- Tudo em React state ou variáveis JS. **NÃO** usar `localStorage`/
  `sessionStorage`.

---

## PARTE 5 — Entrega

- **Valide a sintaxe do JS** e confira os números do cálculo de cargas antes de
  finalizar (rode o HTML/JS num sandbox ou faça uma verificação programática das
  contagens semanais).
- Entregue o **arquivo HTML pronto** + um **resumo em tabela** (pessoa × semana
  + status), destacando quem está sobrecarregado ou no limite.
- Deixe claro no rodapé do HTML: **as simulações no board são exploratórias e
  NÃO alteram o Workflow.**
- Salve o arquivo na pasta de saída e apresente-o ao usuário com
  `present_files`.

---

## Notas de execução

- Se o MCP do Workflow não estiver disponível ou não retornar dados no período,
  avise o usuário claramente antes de gerar qualquer HTML — não invente tarefas.
- Mantenha nomes de pessoas, tarefas e IDs fiéis ao que o MCP retornou.
- O HTML precisa ser 100% autossuficiente: um único arquivo que abre no
  navegador sem servidor e sem outras dependências além da fonte Lato.
