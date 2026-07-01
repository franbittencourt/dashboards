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

Todo o JSON/TMDL foi escrito manualmente seguindo a especificação do formato PBIP/PBIR/TMDL, mas **não pôde ser aberto e validado visualmente** antes da entrega (o ambiente onde foi gerado é Linux, sem o Power BI Desktop instalado). Ou seja: a estrutura, os relacionamentos, as medidas DAX e os dados foram todos verificados programaticamente (JSON válido, sintaxe TMDL consistente), mas alguns detalhes finos de formatação de visual podem precisar de um pequeno ajuste manual na primeira abertura. Veja a lista abaixo — nenhum desses pontos deve impedir o arquivo de abrir, mas fique de olho neles.

### Passos manuais recomendados após abrir (dão o "acabamento" final)

1. **Ações dos botões de navegação.** Os botões (Home → Visão Geral, Visão Geral → Home, abas "Orçamento Anual (PAN)" ⇄ "Ciclo de Vida", botões "‹ Voltar") foram criados com aparência de aba/CTA prontos (cor, texto, formato), mas a **ação de clique** (Format → Botão → Ação → Tipo: Navegação de página → Destino) é o item de maior incerteza deste arquivo e pode precisar ser configurada manualmente para cada botão (é um ajuste de ~10 segundos por botão no painel Formatar). Mapa de destinos:
   - Home `vHomeButtonEnter` → página **Visão Geral**
   - Visão Geral `vGeralBtnHome` → página **Home**
   - Visão Projeto (ambas as páginas) `vHdrBtnVoltar` → página **Visão Geral**
   - `vHdrTabPAN` → página **Visão Projeto - Orçamento Anual (PAN)**
   - `vHdrTabCiclo` → página **Visão Projeto - Ciclo de Vida**
2. **Sincronizar o slicer "Selecionar Obra"** entre as duas páginas de Visão Projeto (View → Sincronizar Segmentações de Dados → marcar as duas páginas), para que trocar de obra numa aba reflita na outra automaticamente.
3. **Capa (Home):** o plano de fundo está como um retângulo azul-marinho sólido (`vHomeHero`), no lugar da imagem de capa da empreiteira — troque por Formatar página → Fundo do Canvas → Imagem com a foto real. O visual **"HTML Content"** (custom visual do AppSource) mencionado na proposta como bloco de boas-vindas rico **não está incluído no arquivo** — como ele depende de um pacote binário externo que não pôde ser embutido/validado aqui, foi substituído por um textbox nativo equivalente (`vHomeWelcomeText`). Para evoluir para o visual real: Inserir → Mais Visuais → AppSource → procurar "HTML Content" → instalar → recriar o card usando as medidas de `Medidas` como texto dinâmico.
4. **Cores de fase no donut/badges:** a tabela `DimFase` já tem uma coluna `Cor` (hex por fase) pensada para colorir consistentemente o donut da Visão Geral e qualquer badge de fase — como o JSON de "cores de dados" por categoria é um dos formatos mais sensíveis a erro de sintaxe, essa amarração de cor **não foi aplicada automaticamente** no arquivo; é rápido de fazer manualmente (Formatar visual → Cores de dados → por categoria) usando os hex da tabela como referência.
5. Revise os títulos dinâmicos: o card "Custo Linha de Base" mostra o valor numérico; logo abaixo, um segundo card (`vCicloLabelVersao`) mostra o texto "Custo Linha de Base (Rev X – dd/mm/aaaa)" como legenda — se preferir, uma vez confirmado que a formatação abre bem, dá para consolidar isso num título dinâmico do próprio card.

## O que está implementado

- 4 páginas: Home, Visão Geral, Visão Projeto – Orçamento Anual (PAN), Visão Projeto – Ciclo de Vida.
- Navegação em abas (páginas físicas) entre PAN e Ciclo de Vida, conforme a Seção 5 da proposta.
- Modelo estrela: `DimObra`, `DimCalendario`, `DimFase`, `FatoRealizadoMensal`, `FatoPlanejadoPAN`, `FatoCustoDetalhado`, mais uma tabela `Medidas` com ~20 medidas DAX.
- 12 obras fictícias (regiões Norte/Sul/Leste/Centro-Oeste; contratantes PetroMax Óleo & Gás, Metrovia Urban Infra, Hidro Sul Energia; todas as 7 fases representadas).
- Histórico mensal realizado (curva em S) desde o início de cada obra até 07/2026, PAN 2026 mês a mês, e breakdown de custo por Elemento → Tipo → Grupo → Classe.
- Justificativas de desvio fake em ~12% dos meses com valor realizado, para popular o tooltip do gráfico de linha.

## Dados fictícios — como foram gerados

Os dados foram gerados por um script Python (curva de avanço físico-financeiro em formato S, com ruído aleatório) para simular obras em fases diferentes com graus de execução plausíveis. Não representam nenhuma obra real. A obra `OBR-002 — Terminal Portuário Leste` tem o histórico mais longo (início em 01/2024) e é uma boa escolha para explorar o gráfico de ciclo de vida.
