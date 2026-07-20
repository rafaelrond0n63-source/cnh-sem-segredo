# CNH Sem Segredo — Inventário Visual

## Referências aprovadas

- `cnh-home-concept.png`: tela inicial mobile e referência principal do shell.
- `cnh-mobile-states.png`: onboarding, plano, aula e simulado.

Os conceitos foram aprovados em 20 de julho de 2026. O texto renderizado nas imagens é referência de hierarquia; a copy code-native da especificação prevalece quando a imagem contiver variações ou erros tipográficos.

## Direção

Produto educacional adulto, premium e confiável. Densidade baixa a média, superfícies claras, cabeçalhos azul-marinho, amarelo restrito a ações/progresso e uma única área dominante por tela. Sem aparência infantil, institucional ou de painel corporativo.

## Tokens

```css
--navy-950: #071a2d;
--navy-800: #0e3557;
--yellow-500: #ffc928;
--white: #ffffff;
--gray-50: #f4f6f8;
--slate-500: #64748b;
--green-500: #22c55e;
--red-500: #ef4444;
--orange-500: #f59e0b;
--radius-sm: 12px;
--radius-md: 16px;
--radius-lg: 20px;
--shadow-card: 0 8px 24px rgb(7 26 45 / 10%);
--shadow-float: 0 16px 36px rgb(7 26 45 / 16%);
--motion-fast: 160ms;
--motion-base: 240ms;
```

O fundo claro é cinza frio `#F4F6F8`, nunca creme. Superfícies de leitura são branco verdadeiro. O modo escuro usa `#071A2D` como fundo e variações de `#0E3557` como superfícies.

## Tipografia

- Família: `Inter`, `ui-sans-serif`, `system-ui`, `-apple-system`, `BlinkMacSystemFont`, `Segoe UI`, sans-serif. Não carregar fonte externa; `Inter` só será usada se instalada no sistema.
- Display mobile: 32/38, peso 800.
- Título de tela: 28/34, peso 800.
- Título de card: 20/26, peso 750.
- Corpo: 16/24, peso 400–500.
- Label e controles: 14/20, peso 650–750.
- Caption: 13/18, peso 500.
- Botão principal: 15/20, peso 800, caixa alta apenas nas ações principais já fornecidas.

## Espaçamento e containers

- Escala base: 4, 8, 12, 16, 20, 24, 32 e 40 px.
- Gutter mobile: 20 px; 16 px em 320 px.
- Gutter desktop: 32 px, conteúdo máximo de 1180 px.
- Cards: raio 16–20 px, borda fria discreta ou sombra, nunca ambos em excesso.
- Listas de plano e questões: linhas abertas ou uma superfície contínua; não converter tudo em grid de cards.
- Botões e campos: mínimo 48 px de altura; alternativas de simulado, mínimo 56 px.
- Barra inferior: conteúdo + `env(safe-area-inset-bottom)`; reservar espaço no main.

## Anatomia por tela

### Shell

Mobile: cabeçalho simples, conteúdo rolável e navegação inferior fixa com cinco itens. Desktop: sidebar compacta fixa e área de conteúdo; ações secundárias no header. Ícones são Lucide outline, stroke 2, 20–24 px; selecionado em amarelo sobre azul-marinho.

### Início

Ordem: marca/header, saudação, missão dominante, progresso geral/métricas, recomendação, dois atalhos visíveis e continuação. O card de missão usa fundo azul-marinho, motivo discreto de estrada, copy branca e botão amarelo.

### Onboarding

Cabeçalho azul-marinho com voltar, indicador `Etapa X de 5` e linha amarela. Conteúdo branco com uma pergunta por tela, campo/opções grandes e botão amarelo no fim da viewport.

### Plano

Cabeçalho azul-marinho e lista por semana. Cada dia é uma linha com status, duração e chevron. Bloqueados têm contraste reduzido, cadeado e explicação; continuam legíveis.

### Aula

Cabeçalho azul-marinho com título e tracker de cinco etapas. Conteúdo em blocos curtos sobre branco. Caixas pedagógicas têm borda/amarelo suave sem fundo saturado. A ação principal fica após o conteúdo, não flutuante.

### Simulado

Cabeçalho azul-marinho com título, timer e progresso. Questão e alternativas sobre branco. Marcar para revisão é secundário outline; próxima é amarela; voltar é outline. Navegador compacto em faixa azul-marinho.

## Famílias de componentes

- `Button`: primary, secondary, ghost, danger; loading e disabled.
- `ProgressBar`: linear, compact e step tracker.
- `Surface`: card padrão, mission e educational callout.
- `NavigationItem`: mobile e desktop, selected/hover/focus.
- `Status`: available, in-progress, completed e locked; usar texto + ícone, não cor isolada.
- `FormControl`: input, radio-card, checkbox e date.
- `QuestionOption`: idle, selected, correct, incorrect, disabled.
- `Modal`: confirmação, desbloqueio e erro recuperável.
- `EmptyState`: ícone, título/texto curto e uma ação.

## Copy permitida no primeiro viewport

- `CNH Sem Segredo`
- `Plano de Aprovação em 30 Dias`
- `Olá, [nome] 👋`
- `Continue avançando rumo à sua aprovação.`
- `Dia 1 — Conhecendo a prova teórica` ou missão real seguinte
- `Tempo estimado: [X] minutos`
- `COMEÇAR AGORA` ou `CONTINUAR MISSÃO`
- `[X] de 30 dias`
- `Questões respondidas`
- `Acertos`
- `Melhor simulado`
- `Sequência`
- `Responda algumas questões para receber recomendações personalizadas.`
- Navegação: `Início`, `Plano`, `Simulados`, `Placas`, `Progresso`

Não adicionar eyebrow, badge, prova social, promessa de aprovação, estatística simulada, token ou validade.

## Estados e interação

- Hover só complementa; toda ação funciona por toque e teclado.
- Foco: outline amarelo de 3 px com offset de 2 px em superfícies escuras e azul-marinho em superfícies claras.
- Feedback de resposta usa ícone, texto e cor.
- Mudanças relevantes usam `aria-live="polite"`.
- Animações: fade/translate de 4–8 px, 160–240 ms; desativar transformações sob `prefers-reduced-motion`.

## Responsividade

- 320–430 px: uma coluna, navegação inferior e métricas 2×2 quando quatro colunas não couberem.
- 768–1023 px: conteúdo central até 760 px, barra inferior ou sidebar conforme espaço real.
- 1024 px+: sidebar de 240 px, main central até 1180 px; home pode usar coluna principal e rail de progresso, sem alterar a ordem semântica.

## Assets

Ícones funcionais serão Lucide. Placas serão SVGs locais validados. O motivo de estrada do card principal será SVG/CSS discreto, sem rasterização da interface. Nenhum texto de interface virá das imagens de conceito.

## Desvios intencionais

- O conceito da home mostra `0 de 1 etapas`; a implementação usará o número real de etapas do dia.
- O conceito do plano mostra títulos ilustrativos diferentes da grade oficial; a implementação usará os 30 títulos aprovados na especificação.
- O conceito da aula combina o Dia 4 com placas de advertência; a implementação exibirá o conteúdo correspondente ao dia selecionado.
- O conceito do simulado contém uma questão apenas como composição visual; o banco final usará questões originais validadas e não copiará o enunciado da imagem.
