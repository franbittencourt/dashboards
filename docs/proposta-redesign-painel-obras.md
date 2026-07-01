# Proposta de Redesign — Painel de Acompanhamento de Obras

## 1. Diagnóstico rápido

A página **Visão Projeto** hoje mistura duas perguntas de negócio diferentes:

| | Visão Orçamentária (PAN) | Visão Financeira / Ciclo de Vida |
|---|---|---|
| Pergunta | "Estamos dentro do orçamento previsto **para este ano**?" | "Como está o projeto em relação ao **orçamento total aprovado**?" |
| Referência | Plano Anual (PAN) do ano corrente | Linha de Base do projeto (aprovada, versão X) |
| Horizonte | 12 meses (ano fiscal) | Vida inteira do projeto (multi-anual) |
| Granularidade | Mês vs. mês, mesmo ano | Acumulado desde o marco zero do projeto |
| Usuário típico | Controladoria / orçamento anual | Gestor de obra / diretoria de projetos |

Colocar os dois na mesma tela força o usuário a decodificar, visual a visual, "de qual orçamento estamos falando aqui?" — isso é a causa raiz da confusão relatada. A solução não é um visual melhor, é **separar o contexto antes que o usuário veja o primeiro número**.

## 2. Nova estrutura de páginas

```
Home  →  Visão Geral  →  Visão Projeto
                              ├── (aba) Orçamento Anual (PAN)
                              └── (aba) Ciclo de Vida do Projeto
```

`Visão Projeto` deixa de ser uma página única e passa a ser um **par de páginas físicas do Power BI** que compartilham o mesmo filtro de obra selecionada, com uma barra de navegação estilo abas no topo. Nomes propostos (visíveis na UI, não os nomes técnicos das páginas):

- **"Orçamento Anual (PAN)"** — ícone de calendário/ano.
- **"Ciclo de Vida do Projeto"** — ícone de linha do tempo/infinito.

Por que esses nomes e não "Visão Orçamentária" / "Visão Financeira": para o gestor de obra e o controller, "PAN" já é o termo do processo interno (não precisa reexplicar), e "Ciclo de Vida do Projeto" comunica horizonte multi-anual sem jargão contábil. Evite nomes como "Visão 1 / Visão 2" — não são autoexplicativos e obrigam o usuário a abrir as duas para saber a diferença.

Cabeçalho comum às duas abas (mesma posição, mesmo conteúdo, não duplicado visualmente — ver seção 5):
- Breadcrumb: `Visão Geral › [Código da Obra] — [Nome do Projeto]`
- Badge de fase atual da obra (cor consistente com o donut da Visão Geral)
- Os dois botões de aba

## 3. Alocação dos visuais existentes

### Aba "Orçamento Anual (PAN)" — mantém filtro fixo no ano corrente
- Card **"Planejado PAN [ano]"**
- Gráfico de barras clusterizado mês a mês: Planejado (PAN) vs. Realizado, ano corrente
- **Novo:** card **"Realizado no Ano [ano]"** — hoje esse número só existe implícito nas barras; como card, dá o mesmo nível de destaque que o Planejado PAN e permite comparação direta card-a-card.
- **Novo:** card **"% do PAN Realizado no Ano"** — `Realizado Ano / Planejado PAN Ano`. Note que este é **diferente** do "Percentual Financeiro Realizado" da outra aba (que usa a Linha de Base como denominador). Por isso ele não deve ser copiado da outra aba — é uma métrica distinta com o mesmo nome “percentual”, e por isso o rótulo deve deixar claro: "% do PAN Anual Realizado" ≠ "% da Linha de Base Realizado".

### Aba "Ciclo de Vida do Projeto" — sem filtro de ano, todo o histórico
- Card **"Custo Linha de Base ([versão] – [data])"**
- Card **"Valor Realizado Acumulado"**
- Card **"Percentual Financeiro Realizado"**
- Gráfico de linha temporal (histórico completo, todos os meses/anos) com tooltip de Realizado/Planejado/justificativa
- Tabela pivot: Elemento da obra → Tipo de custo → Grupo de custo → Classe de custo

### Visuais que **não** devem ser duplicados entre as abas
- **Card de percentual**: cada aba tem o seu, mas com fórmulas e rótulos diferentes (ver acima). Não é o "mesmo card" replicado — são dois indicadores diferentes que merecem cada um sua aba.
- **Tabela pivot de custos**: pertence só ao Ciclo de Vida. Ela detalha *como o dinheiro total foi gasto*, uma pergunta de escopo/execução acumulada, sem sentido "por ano" nesse nível de detalhe (o time de obra normalmente não fecha o breakdown de classe de custo por ano-fiscal, e sim por elemento construtivo, do início ao fim). Se a controladoria futuramente quiser esse detalhamento por ano, é um visual novo e explícito na aba PAN — não uma cópia da pivot existente.
- **Gráfico de linha temporal completo**: fica só no Ciclo de Vida. Na aba PAN o gráfico de barras clusterizado (12 meses) já responde "mês a mês do ano" com mais clareza que uma linha de 30+ meses cortada por um filtro de ano.

## 4. Visuais novos/reformulados sugeridos

| Visual | Aba | Por quê |
|---|---|---|
| Curva S (linha dupla: Planejado acumulado x Realizado acumulado, todo o histórico) | Ciclo de Vida | É o gráfico padrão da indústria de construção para comunicar avanço físico-financeiro a diretoria/cliente. A linha temporal atual mostra só o Realizado acumulado — adicionar a curva do Planejado (mesmo que o PAN só cubra o ano corrente, dá para tornar essa curva contínua compondo baseline original) fecha visualmente a pergunta "estamos acima ou abaixo da curva prevista desde o início?". |
| Waterfall de variação (Linha de Base → Aditivos → Reprogramações → Custo Atual Projetado) | Ciclo de Vida | Muitos projetos de obra têm aditivos contratuais que mudam o valor "aprovado" ao longo da vida. Hoje o painel só mostra a Linha de Base original; sem esse waterfall, uma obra com aditivo aparenta "estourar orçamento" quando na verdade o orçamento foi revisado. |
| Indicador de tendência / projeção de conclusão (ex.: card ou gauge "Custo Final Projetado" com base na velocidade média de queima) | Ciclo de Vida | Responde "se continuar nesse ritmo, vou terminar dentro do aprovado?" — pergunta natural depois de ver % realizado, hoje sem resposta no painel. |
| Card **"Saldo PAN a Realizar no Ano"** (Planejado PAN − Realizado Ano) | Orçamento Anual (PAN) | Complementa o gráfico de barras com o número que a controladoria mais usa em reunião de fechamento mensal. |
| Semáforo de status de prazo (Em dia / Atenção / Atrasada) ao lado do breadcrumb comum | Ambas as abas (cabeçalho compartilhado) | É contexto de leitura rápida que vale nas duas visões e não compete por espaço com os gráficos financeiros — por isso vive no cabeçalho comum, não dentro de uma aba específica. |

## 5. Implementação da navegação (Power BI)

**Escolha: duas páginas físicas + botões estilo aba**, conforme decidido.

Estrutura:
1. Crie as páginas `Visão Projeto - PAN` e `Visão Projeto - Ciclo de Vida` como páginas reais do relatório (não a mesma página com bookmarks).
2. No topo de cada uma, dois botões retângulo lado a lado, ação **"Navegação de página"** apontando um para cada página. O botão da aba ativa fica com preenchimento sólido (cor de destaque) e o botão da aba inativa com preenchimento neutro/outline — simulando visualmente uma tab strip. Isso é conseguido com **dois estados de formatação por botão** (default vs. "on hover"/estado ativo simulado por estar na própria página): a forma mais simples e 100% nativa é usar o mesmo par de botões nas duas páginas, mas com o botão da página atual pré-formatado como "ativo" (cor sólida, sem ação de clique nele mesmo) e o botão da outra página como "inativo" (outline, com ação de navegação).
3. Sincronize os filtros/slicer de obra selecionada entre as duas páginas: use o mesmo campo de filtro (ex.: um "ID do projeto selecionado" vindo de um drill-through ou de um slicer sincronizado via **Sincronização de Segmentação de Dados**, ou — mais robusto — implemente a navegação `Visão Geral → Visão Projeto` como um **drillthrough** por `CodigoObra`, que é automaticamente aplicado nas duas páginas de destino se ambas tiverem o mesmo filtro de drillthrough configurado.
4. Um terceiro botão "‹ Voltar" (ação "Voltar", nativa de drillthrough, ou "Navegação de página" para Visão Geral) fica fixo no cabeçalho comum.

Por que páginas físicas e não bookmarks (registrando a decisão para o time, caso seja revisitada):
- Bookmarks acumulam um "estado escondido" por trás dos botões — todo novo visual adicionado a uma das duas visões precisa ser manualmente incluído/excluído dos bookmarks certos, e é fácil esquecer, gerando bugs sutis (visual "vaza" para a visão errada).
- Páginas físicas aparecem no painel de páginas, são mais fáceis de dar manutenção por outra pessoa da equipe, funcionam corretamente com o app do Power BI Mobile e com paginação/URLs de deep link (`?pageName=...`) — útil se um dia vocês quiserem linkar direto para "a visão PAN da obra X" a partir de um e-mail ou Teams.
- O custo (2 páginas em vez de bookmarks numa página só) é irrelevante em número de páginas do relatório.

## 6. Boas práticas de UX aplicadas

- **Uma pergunta por tela.** Cada aba responde a uma pergunta de negócio (anual vs. vida inteira). Nunca force o usuário a filtrar mentalmente qual card responde a qual pergunta.
- **Rótulos incluem sempre a referência de orçamento.** Nenhum card deve dizer só "Planejado" ou só "%" — sempre "Planejado (PAN 2026)" ou "% da Linha de Base". Ambiguidade de rótulo é a causa nº 1 de desconfiança no dado em painéis financeiros.
- **Cor e forma consistentes para o mesmo conceito em todo o painel.** A cor de cada Fase (Execução, Concluído, etc.) deve ser idêntica no donut da Visão Geral, no badge do cabeçalho de Visão Projeto e em qualquer card de status — reduz carga cognitiva.
- **Navegação sempre visível, nunca mais que 2 cliques de profundidade.** Home → Visão Geral → Visão Projeto (PAN ou Ciclo de Vida) é a árvore inteira. Não adicione uma quarta camada.
- **Cabeçalho fixo com o contexto do filtro atual** (qual obra, qual fase) em toda página de "deep dive" — sem isso, é fácil um gestor analisar a obra errada sem perceber, um erro clássico em painéis multi-projeto.
- **Justificativas de desvio como tooltip, não como coluna permanente** — mantém o gráfico de linha limpo e ainda assim dá contexto sob demanda, exatamente como já implementado; vale reforçar esse padrão nos novos visuais (waterfall de aditivos, por exemplo).
- **Evite duplicar visuais "porque cabe".** Cada visual deve existir em exatamente uma aba, a menos que a métrica seja genuinamente compartilhada por ambas as perguntas de negócio (caso do badge de fase/status de prazo no cabeçalho).

## 7. Redesign da capa (Home)

A capa atual é só uma imagem com um botão — espaço desperdiçado num painel usado recorrentemente (não é uma tela "de primeira visita" apenas). Proposta:

- Mantém a imagem de fundo (identidade visual da empreiteira), mas em camada de fundo, não como único conteúdo.
- **Faixa de KPIs de portfólio** sobrepondo a imagem (parte inferior ou lateral): Obras Ativas, Orçamento Total do Portfólio, % Médio de Execução, Obras em Atenção/Atrasadas. Dá ao usuário recorrente um resumo do estado do portfólio *antes* de entrar em Visão Geral — a capa passa a ter valor informativo, não só estético.
- **Bloco HTML custom (visual "HTML Content" via AppSource)** para um cartão de destaque mais rico que os cards nativos permitem — ex.: um mini-resumo com formatação livre (título, subtítulo, badge de "atualizado em", texto explicativo curto sobre como navegar o painel) que combina texto estático e medidas DAX dinâmicas via placeholders. Use esse visual **com moderação**: ele depende de um custom visual externo (governança/atualização depende do fornecedor no AppSource), então reserve-o para o único bloco onde formatação rica realmente agrega (o card de boas-vindas), e mantenha os KPIs numéricos em cards nativos (mais robustos, sem dependência externa, melhor performance).
- Botão de navegação único e proeminente ("Entrar no Painel →") continua existindo, mas agora abaixo da faixa de KPIs, como call-to-action final, não como único elemento da tela.

## 8. Entregável de exemplo

Foi gerado um projeto Power BI (formato **PBIP**, com modelo semântico em **TMDL** e relatório em **PBIR**) com dados fictícios ilustrando esta estrutura, em `PainelObras/`. Ele implementa:

- As 4 páginas descritas (Home, Visão Geral, Visão Projeto – PAN, Visão Projeto – Ciclo de Vida) com a navegação em abas da seção 5.
- 12 obras fictícias em 4 regiões, 3 áreas contratantes, todas as 7 fases.
- Histórico mensal realizado, PAN 2026 e breakdown de custos por elemento/tipo/grupo/classe.

Ver `PainelObras/README.md` para instruções de abertura, o que foi implementado com fidelidade e quais ajustes manuais finos podem ser necessários na primeira abertura no Power BI Desktop (o ambiente onde este projeto foi gerado não tem o Power BI Desktop instalado para validação visual direta).

## 9. Pontos em aberto para validar com o time

- Confirmar se "PAN" é realmente o termo que os usuários finais reconhecem, ou se é melhor grafar por extenso ("Plano Anual") no rótulo da aba.
- Definir o dono da métrica "Custo Final Projetado" (projeção) — normalmente é controladoria quem valida a metodologia de projeção antes de expor num painel.
- Confirmar com o time de TI/governança se o visual custom "HTML Content" pode ser instalado organizacionalmente (AppSource certificado vs. bloqueado por política).
