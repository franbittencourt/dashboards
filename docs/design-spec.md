# Spec de Design — Painel de Acompanhamento de Obras

Especificação visual do painel, fiel ao que está no PBIR (`PainelObras.Report`). Serve para (a) importar os mockups no Figma e (b) ajustar/reproduzir a formatação no Power BI. Mockups vetoriais em [`docs/spec/`](spec/) — SVG, editáveis no Figma (colar direto no canvas).

- `spec/1-home.svg`
- `spec/2-visao-geral.svg`
- `spec/3-projeto-pan.svg`
- `spec/4-projeto-ciclo.svg`

> Os mockups são desenhados **nas coordenadas exatas** de cada visual no relatório (canvas 1280×720). Um retângulo no SVG = um visual no Power BI, mesma posição e tamanho.

---

## 1. Tokens de design

A maior parte disto já está no tema `PainelObras.Report/StaticResources/RegisteredResources/PainelObrasTheme.json` — o Figma é para desenhar, o tema é o que aplica no Power BI.

### Cores

| Token | Hex | Uso |
|---|---|---|
| Navy (marca) | `#10243E` | Faixa de cabeçalho, hero da capa, cabeçalho de tabela, títulos de visual |
| Navy suave | `#1B3A5F` | Botões secundários sobre a faixa |
| Contorno faixa | `#46648C` | Borda de botões/cards sobre o navy |
| Página (fundo) | `#EEF1F5` | Fundo do canvas (cinza-claro) |
| Card | `#FFFFFF` | Fundo dos visuais |
| Borda card | `#E3E7EC` | Contorno dos cards |
| Tinta título | `#10243E` | Títulos de visual, valores de KPI |
| Tinta corpo | `#1F2937` | Texto de tabela/valores |
| Tinta rótulo | `#5A6472` | Rótulos, categorias, eixos |
| Tinta eixo fraco | `#8A94A0` | Números de eixo |
| Grade | `#EDF0F4` | Linhas de grade horizontais |
| Zebra | `#F5F7FA` | Linha alternada de tabela |
| **Azul (destaque)** | `#1E88E5` | Botão ativo, aba ativa, linha principal |
| **Azul escuro** | `#1565C0` | Série "Realizado", curva do ciclo de vida |
| **Azul claro** | `#5B9BD5` | Série "Planejado (PAN)" |
| Bom | `#43A047` | % acima da meta, concluído |
| Atenção | `#EF6C00` | Obras em atenção, encerramento |
| Crítico | `#E53935` | % abaixo, cancelado |

### Cores por fase (categórico — `DimFase[Cor]`, validado p/ daltonismo)

| Fase | Hex | | Fase | Hex |
|---|---|---|---|---|
| Planejamento | `#9575CD` (violeta) | | Encerramento | `#EF6C00` (laranja) |
| Pré-Contratação | `#00897B` (teal) | | Concluído | `#43A047` (verde) |
| Contratação | `#A0522D` (terracota) | | Cancelado | `#E53935` (vermelho) |
| Execução | `#1E88E5` (azul) | | | |

### Tipografia
- Família: **Segoe UI** (títulos em **Segoe UI Semibold**).
- Título de visual: 12px semibold, navy `#10243E`.
- Valor de KPI (callout): 24–26px, peso 800.
- Rótulo de KPI: 11px, `#5A6472`, MAIÚSCULAS, letter-spacing 0.4.
- Texto de tabela/eixo: 10px / 9px.

### Forma e elevação
- Raio dos cards: **10px**. Botões: **8px**. Chips/badges: 6px.
- Sombra: `y+2, blur 10, #90A4AE @ 16% opacidade` (sutil, "cartão flutuando").
- Espaçamento base: **24px** de margem externa; **12–16px** de gutter entre cards.

---

## 2. Sistema de layout

- **Canvas:** 1280 × 720 (16:9), "Ajustar à página".
- **Faixa de cabeçalho:** navy, altura **64px** (páginas internas) / **300px** (hero da capa), largura total (x=0).
- **Margem lateral de conteúdo:** 24px (x começa em 24, largura útil 1232).
- **Linha de KPIs:** cards de 296×112, gutter de 16px (24 → 336 → 648 → 960).
- **Filtros:** slicers em dropdown, 250×52, numa linha horizontal sob a faixa (y=80).

---

## 3. Inventário por página (coordenadas reais)

Cada linha é um visual do relatório. `x,y,w,h` em px no canvas 1280×720.

### Home (`PaginaHome`)
| Visual | Tipo | x | y | w | h |
|---|---|---|---|---|---|
| vHomeHero | Retângulo (navy) | 0 | 0 | 1280 | 300 |
| vHomeTitle / Subtitle | Caixa de texto | 60 | 56 / 118 | 900 | 60 |
| vHomeCardObrasAtivas | Cartão | 60 | 340 | 270 | 140 |
| vHomeCardOrcamento | Cartão | 350 | 340 | 270 | 140 |
| vHomeCardPctExecucao | Cartão | 640 | 340 | 270 | 140 |
| vHomeCardAtencao | Cartão | 930 | 340 | 270 | 140 |
| vHomeButtonEnter | Botão → Visão Geral | 60 | 610 | 300 | 56 |

> Alternativa: substituir a página inteira pelo visual **HTML Content** com a medida `HTML Capa` (capa rica com banner, KPIs e barras).

### Visão Geral (`PaginaVisaoGeral`)
| Visual | Tipo | x | y | w | h |
|---|---|---|---|---|---|
| vBand + título | Retângulo navy + texto | 0 | 0 | 1280 | 64 |
| vBtnHome | Botão → Home | 1146 | 14 | 110 | 36 |
| vSlicerArea / Regiao / Fase | Slicer (dropdown) | 24 / 290 / 556 | 80 | 250 | 52 |
| vGeralDonutFase | Rosca (cores por fase) | 24 | 148 | 430 | 280 |
| vGeralTabelaArea | Tabela | 470 | 148 | 786 | 280 |
| vGeralTabelaResumo | Tabela | 24 | 444 | 1232 | 260 |

### Visão Projeto — Orçamento Anual PAN (`PaginaProjetoPAN`)
| Visual | Tipo | x | y | w | h |
|---|---|---|---|---|---|
| Cabeçalho (faixa + breadcrumb + abas) | — | 0 | 0 | 1280 | 64 |
| vPanCardPlanejado / Realizado / Pct / Saldo | Cartão | 24 / 336 / 648 / 960 | 84 | 296 | 112 |
| vPanChartColunas | Colunas agrupadas | 24 | 216 | 1232 | 488 |

> Filtro de página fixo em **Ano = 2026**. Séries: Planejado `#5B9BD5`, Realizado `#1565C0`. Rótulos de dados visíveis, eixo em milhões.

### Visão Projeto — Ciclo de Vida (`PaginaProjetoCicloVida`)
| Visual | Tipo | x | y | w | h |
|---|---|---|---|---|---|
| Cabeçalho (faixa + breadcrumb + abas) | — | 0 | 0 | 1280 | 64 |
| vCicloCardLinhaBase (+ legenda versão) | Cartão | 24 | 84 | 296 | 112 |
| vCicloCardRealizado | Cartão | 336 | 84 | 296 | 112 |
| vCicloCardPct | Cartão | 648 | 84 | 296 | 112 |
| vCicloNota | Caixa de texto | 960 | 84 | 296 | 112 |
| vCicloLineChart | Linha (histórico completo) | 24 | 244 | 1232 | 240 |
| vCicloMatrix | Matriz (Elemento→Tipo→Grupo→Classe) | 24 | 496 | 1232 | 208 |

---

## 4. Como aplicar/ajustar no Power BI

1. **Tema:** Exibição → Temas → Procurar temas → `PainelObrasTheme.json`. Isso já dá cards, bordas, sombra, fundo cinza, cabeçalho de tabela navy e tooltips. **Ajuste a cor da marca aqui** (`dataColors`, `tableAccent`).
2. **Faixa navy:** um visual Retângulo (Inserir → Formas → Retângulo) 1280×64 no topo, preenchimento `#10243E`, z-order atrás.
3. **Donut colorido por fase:** Formatar → Cores dos dados → por categoria, casando com a tabela de fases acima (mesmos hex de `DimFase[Cor]`).
4. **Colunas Planejado vs. Realizado:** Formatar → Colunas → Cores por série (`#5B9BD5` / `#1565C0`); ative Rótulos de dados; eixo Y → Unidades de exibição = Milhões.
5. **Slicers:** Formatar → estilo do slicer = "Lista suspensa" (dropdown).
6. **Navegação (abas/botões):** cada botão → Ação → Navegação de página (destinos em `docs/pbip-leia-me.md`).

> Espaçamentos e coordenadas: use o painel **Formatar → Geral → Propriedades** de cada visual e digite os `x/y/largura/altura` da tabela acima para bater exatamente com o mockup.
