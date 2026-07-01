# PainelObras.pbip — leia-me

Projeto Power BI de exemplo (formato **PBIP**: modelo semântico em **TMDL** + relatório em **PBIR**), com dados **fictícios**, implementando a reorganização proposta em `docs/proposta-redesign-painel-obras.md`.

## Como abrir

1. Baixe/clone o repositório inteiro (os três itens abaixo precisam estar juntos, na mesma pasta):
   - `PainelObras.pbip`
   - `PainelObras.Report/`
   - `PainelObras.SemanticModel/`
2. Dê duplo clique em `PainelObras.pbip`. O Power BI Desktop precisa ter as opções de visualização em pré-visualização **"Power BI Project (.pbip) save option"** e **"TMDL"** habilitadas (Arquivo → Opções e Configurações → Opções → Versões prévias de recursos). Em versões recentes do Desktop (2024+) isso já vem habilitado por padrão.
3. O modelo semântico usa dados 100% embutidos (via Power Query `#table(...)`, sem conexão externa) — não deve pedir nenhuma credencial ao abrir.

## Importante: este projeto foi gerado sem acesso ao Power BI Desktop

Todo o JSON/TMDL foi escrito manualmente seguindo a especificação do formato PBIP/PBIR/TMDL. Numa primeira versão isso causou um erro real ao abrir (`Cannot find file 'version.json'`) — faltava um arquivo obrigatório (`PainelObras.Report/definition/version.json`) que as versões mais recentes do Power BI Desktop passaram a exigir. Esse arquivo foi adicionado, e nessa correção também validamos a estrutura dos visuais (título, ações de botão) contra projetos PBIP reais publicados publicamente, então a maior parte das incertezas da primeira versão foi resolvida:

- **Ações de navegação dos botões já estão configuradas** (Home ⇄ Visão Geral, abas "Orçamento Anual (PAN)" ⇄ "Ciclo de Vida", botões "‹ Voltar"). Se algum botão específico não navegar, é um ajuste de ~10s: Formatar botão → Ação → Tipo: Navegação de página → Destino.
- **Títulos dos gráficos/cards** foram movidos para a propriedade correta (`visualContainerObjects`), confirmada em exemplos reais.

Se ainda assim algo não abrir de primeira, seria útil relatar a mensagem de erro completa (igual à anterior) para eu corrigir — o Power BI é bastante estrito com esse formato e pequenas divergências de schema podem aparecer em versões futuras do Desktop.

### Correção de tipos de dados (erros "SUM não pode ser usada em string" / comparação string vs. integer)

Uma versão anterior gerava as tabelas com `#table(...)` **sem tipar as colunas**. No Power Query, um `#table` sem tipo entrega tudo como `any`, que o modelo importa como **texto** — por isso `SUM` sobre valores numéricos e comparações como `Ano = 2026` davam erro nas páginas de Visão Projeto (a Visão Geral funcionava porque usava sobretudo contagens e colunas de texto). Corrigido: todas as tabelas agora usam `#table(type table [Coluna = tipo], {...})`, tipando explicitamente cada coluna (`Int64.Type`, `number`, `date`, `text`) para casar com o `dataType` declarado no TMDL.

### Passos manuais recomendados (melhorias, não bloqueiam a abertura)

1. **Sincronizar o slicer "Selecionar Obra"** entre as duas páginas de Visão Projeto (View → Sincronizar Segmentações de Dados → marcar as duas páginas), para que trocar de obra numa aba reflita na outra automaticamente.
2. **Capa (Home) com HTML:** o modelo já inclui a medida **`HTML Capa`** (tabela `Medidas`), que retorna o HTML da capa com: um **banner ilustrado** (SVG de canteiro de obras embutido em base64, sem arquivo externo); 4 KPIs do portfólio (Total de obras, Valor do portfólio, Orçamento planejado no ano, Orçamento realizado no ano); uma **barra de execução do orçamento no ano** (realizado ÷ planejado acumulado até o mês corrente, com **cor condicional** verde/âmbar/vermelho); uma **mini-distribuição "Obras por fase"** (barra empilhada colorida com as cores da `DimFase` + legenda); um selo **"Dados até [mês]/[ano]" com dot pulsante** (animação CSS); **hover** nos cards; e uma barra de **atalhos de navegação**. Todos os números vêm do modelo e **respondem a filtros**. Para usá-la: Inserir → Mais Visuais → AppSource → instalar **"HTML Content"** (Daniel Marsh-Patrick) → arrastar a medida `HTML Capa` para o campo *Values* → aumentar o visual para ocupar a página. O plano de fundo antigo (`vHomeHero`) e o textbox `vHomeWelcomeText` podem ser removidos depois. Prévia em `docs/capa-preview.html`.
   - *Notas técnicas:* larguras em CSS usam `SUBSTITUTE(FORMAT(...),",",".")` para forçar ponto decimal (o modelo é pt-BR, senão a vírgula quebraria o `width:%`); a barra de fases é gerada com `CONCATENATEX` sobre `DimFase` ordenado por `Ordem`; o mês de referência ("dados até") vem do último mês com realizado (`MAX(FatoRealizadoMensal[Data])`), não de `TODAY()`.
   - **Navegação dos atalhos:** o visual HTML Content roda num iframe isolado, então os "botões" desenhados no HTML **não trocam de página do relatório sozinhos** (links HTML só abriam URL externa). Para torná-los funcionais, sobreponha 3 **botões nativos transparentes** (Inserir → Botões → Em branco; preenchimento e borda com transparência 100%; Ação → Navegação de página) exatamente sobre cada chip do canto superior direito, apontando para `PaginaVisaoGeral`, `PaginaProjetoPAN` e `PaginaProjetoCicloVida`. É a técnica padrão para navegação a partir de visuais HTML.
3. ~~Cores de fase no donut~~ **Já aplicado**: o donut da Visão Geral tem cores fixas por fase (via seletores de categoria no PBIR), idênticas às da coluna `DimFase[Cor]` usada pela capa HTML. A paleta de fases foi validada para daltonismo e contraste (violeta → teal → terracota para pré-execução; azul = Execução, laranja = Encerramento, verde = Concluído, vermelho = Cancelado).
4. Revise os títulos dinâmicos: o card "Custo Linha de Base" mostra o valor numérico; logo abaixo, um segundo card (`vCicloLabelVersao`) mostra o texto "Custo Linha de Base (Rev X – dd/mm/aaaa)" como legenda — se preferir, dá para consolidar isso num título dinâmico do próprio card.

## Design system do relatório (tema)

O relatório usa um tema customizado (`PainelObras.Report/StaticResources/RegisteredResources/PainelObrasTheme.json`) que dá o acabamento em todas as páginas sem formatação manual por visual:

- **Cards/visuais**: fundo branco, canto arredondado (10px), borda sutil e sombra leve, sobre página cinza-claro `#EEF1F5` — o padrão "cartões flutuando" de dashboards profissionais.
- **Tipografia**: Segoe UI; títulos de visual em Segoe UI Semibold navy; callouts (KPIs) 24pt navy.
- **Tabelas e matriz**: cabeçalho navy com texto branco, zebra sutil, grid horizontal fino.
- **Faixa de cabeçalho navy** em todas as páginas internas (mesma identidade da capa), com título/breadcrumb à esquerda e navegação (abas/botões) à direita.
- **Slicers em dropdown** numa linha horizontal sob o cabeçalho (padrão de UX para filtros).
- **Paleta validada** (contraste, daltonismo — ΔE entre pares adjacentes, banda de luminosidade): série principal `#1565C0`, série de referência/planejado `#5B9BD5`; fases conforme `DimFase[Cor]`. O gráfico Planejado vs. Realizado usa rótulos de dados visíveis (obrigatório porque o azul-claro do Planejado fica abaixo de 3:1 de contraste — os rótulos são o "relief").

Para trocar as cores da marca: edite o tema JSON e a coluna `Cor` da `DimFase` (a capa HTML herda automaticamente).

## O que está implementado

- 4 páginas: Home, Visão Geral, Visão Projeto – Orçamento Anual (PAN), Visão Projeto – Ciclo de Vida.
- Navegação em abas (páginas físicas) entre PAN e Ciclo de Vida, conforme a Seção 5 da proposta.
- Modelo estrela: `DimObra`, `DimCalendario`, `DimFase`, `FatoRealizadoMensal`, `FatoPlanejadoPAN`, `FatoCustoDetalhado`, mais uma tabela `Medidas` com ~20 medidas DAX.
- 12 obras fictícias (regiões Norte/Sul/Leste/Centro-Oeste; contratantes PetroMax Óleo & Gás, Metrovia Urban Infra, Hidro Sul Energia; todas as 7 fases representadas).
- Histórico mensal realizado (curva em S) desde o início de cada obra até 07/2026, PAN 2026 mês a mês, e breakdown de custo por Elemento → Tipo → Grupo → Classe.
- Justificativas de desvio fake em ~12% dos meses com valor realizado, para popular o tooltip do gráfico de linha.

## Dados fictícios — como foram gerados

Os dados foram gerados por um script Python (curva de avanço físico-financeiro em formato S, com ruído aleatório) para simular obras em fases diferentes com graus de execução plausíveis. Não representam nenhuma obra real. A obra `OBR-002 — Terminal Portuário Leste` tem o histórico mais longo (início em 01/2024) e é uma boa escolha para explorar o gráfico de ciclo de vida.
