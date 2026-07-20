# CNH Sem Segredo Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Construir uma SPA/PWA completa, mobile-first e sem token para uma jornada real de preparação teórica da CNH em 30 dias.

**Architecture:** A aplicação usará React Context + reducer como store persistida, páginas internas trocadas por `activeView`, serviços puros para regras de negócio e dados estáticos validados em build/teste. Componentes não acessarão `localStorage`; fluxos pesados serão lazy-loaded; todas as estatísticas serão derivadas de atividade real.

**Tech Stack:** React 19, Vite, JavaScript/JSX, Tailwind CSS 4, Lucide React, vite-plugin-pwa, Vitest, React Testing Library, ESLint.

## Global Constraints

- Usar somente `.js` e `.jsx`; não criar TypeScript, Next.js, React Router, backend, banco, analytics ou requisições externas.
- Abrir e permanecer na raiz do domínio; alternar telas somente por estado.
- Não implementar token, autenticação ou expiração de acesso.
- Persistir sob a chave única `cnhSemSegredoApp_v1` e tolerar armazenamento indisponível ou corrompido.
- Manter a paleta `#071A2D`, `#0E3557`, `#FFC928`, `#FFFFFF`, `#F4F6F8`, `#64748B`, `#22C55E`, `#EF4444`, `#F59E0B`.
- Criar pelo menos 180 questões originais com distribuição 35/25/35/35/20/20/10 e explicação em todas.
- Não inventar regras legais específicas; registrar revisão necessária em `CONTENT_REVIEW.md`.
- Não considerar pronto sem lint, testes, build e QA visual/funcional em navegador.

---

### Task 1: Conceito visual aprovado e inventário de interface

**Files:**
- Create: `docs/design/cnh-home-concept.png`
- Create: `docs/design/cnh-mobile-states.png`
- Create: `docs/design/design-inventory.md`

**Interfaces:**
- Consumes: `docs/superpowers/specs/2026-07-20-cnh-sem-segredo-design.md`
- Produces: tokens, tipografia, anatomia do shell, home mobile/desktop e estados de aula/simulado usados por todas as tarefas visuais.

- [ ] **Step 1: Gerar o conceito da tela inicial completa**

Usar o skill `imagegen` para criar uma tela mobile 390×844 com azul-marinho, branco e amarelo; uma missão dominante; progresso real zerado; navegação inferior; tipografia segura; sem mockup de celular, sem texto inventado e sem aparência de dashboard corporativo.

- [ ] **Step 2: Gerar estados complementares**

Criar uma prancha coerente com onboarding, plano, aula e simulado, preservando exatamente tokens, raio, sombra, tipografia e iconografia da tela inicial.

- [ ] **Step 3: Obter aprovação visual do usuário**

Exibir os dois conceitos e iterar até aprovação explícita. Nenhum código visual começa antes dessa aprovação.

- [ ] **Step 4: Registrar inventário de implementação**

Documentar em `docs/design/design-inventory.md`: copy permitida no primeiro viewport, tokens, escala tipográfica, espaçamentos, raios, sombras, ícones Lucide, container model, estados interativos, comportamento responsivo e lista de desvios intencionais (inicialmente vazia).

- [ ] **Step 5: Commit**

```bash
git add docs/design
git commit -m "docs: approve CNH visual direction"
```

### Task 2: Scaffold, ferramentas e shell testável

**Files:**
- Create: `package.json`
- Create: `vite.config.js`
- Create: `eslint.config.js`
- Create: `index.html`
- Create: `.gitignore`
- Create: `vercel.json`
- Create: `src/main.jsx`
- Create: `src/App.jsx`
- Create: `src/styles/index.css`
- Create: `src/tests/setup.js`
- Test: `src/tests/App.test.jsx`

**Interfaces:**
- Consumes: inventário visual da Task 1.
- Produces: scripts `dev`, `build`, `preview`, `lint`, `test`, `test:run`; ambiente jsdom; aplicação renderizável.

- [ ] **Step 1: Escrever o primeiro teste de renderização**

```jsx
import { render, screen } from '@testing-library/react';
import App from '../App';

it('exibe a marca durante a inicialização', () => {
  render(<App />);
  expect(screen.getByText('CNH Sem Segredo')).toBeInTheDocument();
});
```

- [ ] **Step 2: Criar configuração mínima e instalar dependências**

Configurar React, Vite, Tailwind, Lucide, PWA, ESLint, Vitest, jsdom e Testing Library. `index.html` deve conter viewport, theme-color e `<meta name="robots" content="noindex, nofollow">`.

- [ ] **Step 3: Executar o teste e confirmar a falha**

Run: `npm run test:run -- src/tests/App.test.jsx`
Expected: FAIL porque `App.jsx` ainda não existe.

- [ ] **Step 4: Implementar splash mínima e tokens CSS**

```jsx
export default function App() {
  return <main aria-busy="true"><h1>CNH Sem Segredo</h1></main>;
}
```

- [ ] **Step 5: Verificar teste e lint**

Run: `npm run test:run -- src/tests/App.test.jsx && npm run lint`
Expected: PASS e zero erros.

- [ ] **Step 6: Commit**

```bash
git add package.json package-lock.json vite.config.js eslint.config.js index.html .gitignore vercel.json src
git commit -m "chore: scaffold React Vite application"
```

### Task 3: Modelo de estado, armazenamento e importação

**Files:**
- Create: `src/store/defaultState.js`
- Create: `src/store/AppProvider.jsx`
- Create: `src/store/appReducer.js`
- Create: `src/hooks/useApp.js`
- Create: `src/hooks/useStorage.js`
- Create: `src/services/storageService.js`
- Create: `src/services/importExportService.js`
- Create: `src/utils/schema.js`
- Test: `src/tests/storageService.test.js`
- Test: `src/tests/importExportService.test.js`

**Interfaces:**
- Produces: `loadState(storage?)`, `saveState(state, storage?)`, `patchState(patch)`, `resetProgress()`, `exportProgress(state)`, `validateImport(value)`, `importProgress(value)` e `useApp()`.

- [ ] **Step 1: Escrever testes de estado padrão e corrupção**

```js
it('recupera o estado padrão quando o JSON está corrompido', () => {
  const storage = { getItem: () => '{ruim', setItem: vi.fn() };
  expect(loadState(storage)).toMatchObject({ version: 1, unlockedFullPlan: false });
});

it('reinicia progresso sem perder preferências de instalação', () => {
  const reset = createResetState({ preferences: { installDismissed: true } });
  expect(reset.preferences.installDismissed).toBe(true);
});
```

- [ ] **Step 2: Rodar e confirmar falha**

Run: `npm run test:run -- src/tests/storageService.test.js`
Expected: FAIL por exports ausentes.

- [ ] **Step 3: Implementar schema versionado e fallback seguro**

Criar `DEFAULT_STATE` com todos os campos da especificação. Capturar falhas de `getItem`/`setItem`, migrar objetos sem `version`, preencher campos ausentes e nunca lançar erro técnico para componentes.

- [ ] **Step 4: Implementar exportação e importação confirmável**

`exportProgress` deve retornar JSON com `version` e sem campos desconhecidos. `validateImport` deve rejeitar JSON inválido, versão futura, arrays/objetos de formato incorreto e retornar `{ valid, data, error }`.

- [ ] **Step 5: Criar provider com reducer e persistência centralizada**

O reducer deve aceitar `HYDRATE`, `PATCH_PROFILE`, `SET_VIEW`, `COMPLETE_ONBOARDING`, `RECORD_ANSWER`, `SAVE_SIMULATION`, `COMPLETE_DAY`, `RATE_FLASHCARD`, `TOGGLE_FAVORITE`, `UNLOCK_ACHIEVEMENT`, `IMPORT_STATE` e `RESET_PROGRESS`.

- [ ] **Step 6: Verificar testes**

Run: `npm run test:run -- src/tests/storageService.test.js src/tests/importExportService.test.js`
Expected: PASS.

- [ ] **Step 7: Commit**

```bash
git add src/store src/hooks src/services src/utils src/tests
git commit -m "feat: add resilient local state storage"
```

### Task 4: Regras puras de progresso, estatísticas, sequência e conquistas

**Files:**
- Create: `src/services/progressService.js`
- Create: `src/services/statisticsService.js`
- Create: `src/services/achievementService.js`
- Create: `src/hooks/useProgress.js`
- Create: `src/hooks/useStatistics.js`
- Create: `src/hooks/useStudyStreak.js`
- Create: `src/hooks/useAchievements.js`
- Create: `src/data/achievements/index.js`
- Test: `src/tests/progressService.test.js`
- Test: `src/tests/statisticsService.test.js`
- Test: `src/tests/achievementService.test.js`

**Interfaces:**
- Produces: `completeDay(state, dayId)`, `getPlanPercentage(state)`, `isDayUnlocked(state, dayId)`, `calculateStatistics(state)`, `calculateStreak(activityDates, today)`, `evaluateAchievements(state)`.

- [ ] **Step 1: Escrever testes da regra crítica do Dia 7**

```js
it('libera dias 8 a 30 somente ao concluir o Dia 7', () => {
  const state = completeDay(DEFAULT_STATE, 7);
  expect(state.unlockedFullPlan).toBe(true);
  expect(isDayUnlocked(state, 30)).toBe(true);
});

it('não exige os dias 1 a 6 para liberar o plano', () => {
  expect(completeDay(DEFAULT_STATE, 7).studyProgress.completedDays).toEqual([7]);
});
```

- [ ] **Step 2: Escrever testes de porcentagem e sequência**

Cobrir zero dias, 30 dias, duplicidade de datas, hoje/ontem e interrupção.

- [ ] **Step 3: Implementar funções puras sem mutação**

Registrar uma data ativa somente uma vez por dia. Estatísticas devem retornar `null` para matéria forte/fraca sem respostas e jamais criar números de demonstração.

- [ ] **Step 4: Implementar 13 conquistas declarativas**

Cada definição terá `id`, `name`, `description`, `icon`, `target` e `predicate(state, stats)`. `evaluateAchievements` preservará datas já registradas.

- [ ] **Step 5: Verificar testes e commit**

Run: `npm run test:run -- src/tests/progressService.test.js src/tests/statisticsService.test.js src/tests/achievementService.test.js`
Expected: PASS.

```bash
git add src/services src/hooks src/data/achievements src/tests
git commit -m "feat: calculate progress streaks and achievements"
```

### Task 5: Dados dos 30 dias e conteúdo das aulas

**Files:**
- Create: `src/data/days/index.js`
- Create: `src/data/days/week1.js`
- Create: `src/data/days/week2.js`
- Create: `src/data/days/week3.js`
- Create: `src/data/days/week4.js`
- Create: `CONTENT_REVIEW.md`
- Test: `src/tests/daysData.test.js`

**Interfaces:**
- Produces: `studyDays` com exatamente 30 objetos `{ id, week, title, category, duration, icon, steps, content, questionCount }` e `getDayById(id)`.

- [ ] **Step 1: Escrever validador de dias no teste**

```js
it('possui 30 dias completos e IDs únicos', () => {
  expect(studyDays).toHaveLength(30);
  expect(new Set(studyDays.map(day => day.id)).size).toBe(30);
  studyDays.forEach(day => expect(day.steps.length).toBeGreaterThanOrEqual(3));
});
```

- [ ] **Step 2: Confirmar falha e criar dados semana a semana**

Run: `npm run test:run -- src/tests/daysData.test.js`
Expected: FAIL antes dos arquivos.

Cada aula deve usar blocos `paragraph`, `list`, `comparison`, `remember`, `warning`, `exam`, `checklist` ou `activity`, com textos curtos e sem artigos/valores/prazos legais não validados.

- [ ] **Step 3: Registrar revisão editorial**

`CONTENT_REVIEW.md` deve listar formato de prova por estado, atualizações do CTB, competências institucionais, representações de placas e parâmetros de simulados que precisam de fonte oficial.

- [ ] **Step 4: Testar e commit**

Run: `npm run test:run -- src/tests/daysData.test.js`
Expected: PASS.

```bash
git add src/data/days CONTENT_REVIEW.md src/tests/daysData.test.js
git commit -m "feat: add complete 30-day curriculum"
```

### Task 6: Banco de 180 questões e motores de avaliação

**Files:**
- Create: `src/data/questions/legislation.js`
- Create: `src/data/questions/trafficSystem.js`
- Create: `src/data/questions/signaling.js`
- Create: `src/data/questions/defensiveDriving.js`
- Create: `src/data/questions/firstAid.js`
- Create: `src/data/questions/mechanics.js`
- Create: `src/data/questions/environment.js`
- Create: `src/data/questions/index.js`
- Create: `src/utils/validateQuestions.js`
- Create: `src/services/quizService.js`
- Create: `src/services/simulationService.js`
- Create: `src/hooks/useQuiz.js`
- Create: `src/hooks/useSimulation.js`
- Test: `src/tests/questionsData.test.js`
- Test: `src/tests/quizService.test.js`
- Test: `src/tests/simulationService.test.js`

**Interfaces:**
- Produces: `questions`, `validateQuestions(list)`, `checkAnswer(question, selectedIndex)`, `createSimulation(options, pool, rng)`, `calculateSimulationResult(simulation, answers)`.

- [ ] **Step 1: Escrever testes de distribuição e schema**

```js
expect(questions).toHaveLength(180);
expect(countByCategory(questions)).toEqual({
  legislation: 35, trafficSystem: 25, signaling: 35,
  defensiveDriving: 35, firstAid: 20, mechanics: 20, environment: 10,
});
expect(validateQuestions(questions)).toEqual([]);
```

- [ ] **Step 2: Criar questões originais categoria por categoria**

Cada objeto seguirá `{ id, category, topic, difficulty, statement, alternatives, correctAnswer, explanation, tip, dayId, optionalImage, tags }`. Nenhum arquivo usará geração em runtime ou alternativas repetidas.

- [ ] **Step 3: Implementar validador executado em desenvolvimento**

Validar ID duplicado, enunciado, categoria, quatro alternativas, índice correto, explicação e dificuldade. Em `main.jsx`, lançar erro descritivo somente quando `import.meta.env.DEV` e houver falha.

- [ ] **Step 4: Escrever e implementar motores puros**

Testar confirmação obrigatória em aprendizagem, ausência de correção no simulado, seleção por categoria/dificuldade, RNG injetável, resultado por matéria, classificação e histórico de erros.

- [ ] **Step 5: Verificar e commit**

Run: `npm run test:run -- src/tests/questionsData.test.js src/tests/quizService.test.js src/tests/simulationService.test.js`
Expected: PASS com 180 questões válidas.

```bash
git add src/data/questions src/utils src/services src/hooks src/tests
git commit -m "feat: add validated question bank and assessment engines"
```

### Task 7: Shell, onboarding e tela inicial

**Files:**
- Create: `src/components/common/SplashScreen.jsx`
- Create: `src/components/common/ProgressBar.jsx`
- Create: `src/components/common/EmptyState.jsx`
- Create: `src/components/common/ErrorBoundary.jsx`
- Create: `src/components/layout/AppShell.jsx`
- Create: `src/components/layout/AppHeader.jsx`
- Create: `src/components/layout/BottomNavigation.jsx`
- Create: `src/components/layout/DesktopSidebar.jsx`
- Create: `src/pages/Onboarding.jsx`
- Create: `src/pages/HomeView.jsx`
- Test: `src/tests/onboarding.integration.test.jsx`
- Test: `src/tests/navigation.test.jsx`

**Interfaces:**
- Consumes: `useApp`, `studyDays`, estatísticas e inventário visual.
- Produces: shell responsivo e fluxo onboarding → home.

- [ ] **Step 1: Escrever integração do onboarding**

```jsx
it('salva o perfil e abre o início após cinco etapas', async () => {
  render(<App />);
  await user.click(screen.getByRole('button', { name: /começar meu plano/i }));
  await user.type(screen.getByLabelText(/primeiro nome/i), 'Rafa');
  // selecionar categoria, data opcional e dificuldade
  expect(await screen.findByText('Olá, Rafa 👋')).toBeInTheDocument();
});
```

- [ ] **Step 2: Implementar onboarding validado**

Etapas, voltar/continuar, barra, nome obrigatório, data mínima hoje, preparo curto ligado a estado real e conclusão persistida.

- [ ] **Step 3: Implementar shell fiel ao conceito**

Bottom nav em `<nav aria-label="Navegação principal">`, safe-area e cinco itens; sidebar desktop com mesmos destinos; atalhos secundários no cabeçalho. Navegação chama `setActiveView` sem mudar URL.

- [ ] **Step 4: Implementar home sem dados fictícios**

Missão seguinte, progresso, quatro métricas reais, contagem para a prova, recomendação por matéria, atalhos e mensagem motivacional determinística por data.

- [ ] **Step 5: Testar em jsdom e navegador; commit**

Run: `npm run test:run -- src/tests/onboarding.integration.test.jsx src/tests/navigation.test.jsx`
Expected: PASS.

```bash
git add src/components src/pages src/tests src/App.jsx
git commit -m "feat: add onboarding app shell and home"
```

### Task 8: Plano, aula, quiz e desbloqueio persistente

**Files:**
- Create: `src/pages/StudyPlanView.jsx`
- Create: `src/pages/LessonView.jsx`
- Create: `src/pages/CompletionView.jsx`
- Create: `src/components/lessons/DailyLessonCard.jsx`
- Create: `src/components/lessons/LessonProgress.jsx`
- Create: `src/components/lessons/ContentBlock.jsx`
- Create: `src/components/lessons/TipBox.jsx`
- Create: `src/components/lessons/UnlockCelebration.jsx`
- Create: `src/components/quiz/QuestionCard.jsx`
- Create: `src/components/quiz/QuizEngine.jsx`
- Test: `src/tests/day7Unlock.integration.test.jsx`

**Interfaces:**
- Consumes: dias, questões, store e serviços de progresso/quiz.
- Produces: plano completo, retomada por etapa, conclusão e desbloqueio.

- [ ] **Step 1: Escrever integração crítica de desbloqueio**

Montar provider com onboarding concluído, abrir Dia 7, completar conteúdo e simulado, confirmar `unlockedFullPlan`, desmontar/remontar e verificar Dia 30 disponível.

- [ ] **Step 2: Implementar plano em quatro semanas**

Cards devem calcular `available`, `in-progress`, `completed` ou `locked`; dias 1–7 independentes; dias 8–30 esmaecidos e não clicáveis até desbloqueio.

- [ ] **Step 3: Implementar aula por etapas**

Renderizar os tipos de bloco declarativos, checklist e quiz; salvar o índice exato após uma ação concluída; nunca concluir no simples acesso.

- [ ] **Step 4: Implementar QuizEngine acessível**

Alternativas serão radios grandes; confirmação separada; feedback com `aria-live`; explicação e dica somente depois de confirmar.

- [ ] **Step 5: Implementar celebração e conclusão**

Ao completar Dia 7, abrir modal de desbloqueio. Ao completar Dia 30, exibir métricas reais e quatro ações solicitadas.

- [ ] **Step 6: Testar e commit**

Run: `npm run test:run -- src/tests/day7Unlock.integration.test.jsx`
Expected: PASS incluindo remount.

```bash
git add src/pages src/components/lessons src/components/quiz src/tests
git commit -m "feat: add study plan lessons and persistent unlock"
```

### Task 9: Simulados, resultados e revisão dos erros

**Files:**
- Create: `src/pages/SimulationsView.jsx`
- Create: `src/pages/SimulationView.jsx`
- Create: `src/pages/SimulationResult.jsx`
- Create: `src/pages/MistakesView.jsx`
- Create: `src/components/simulations/SimulationEngine.jsx`
- Create: `src/components/simulations/SimulationTimer.jsx`
- Create: `src/components/simulations/QuestionNavigator.jsx`
- Test: `src/tests/simulationFlow.integration.test.jsx`
- Test: `src/tests/mistakes.test.js`

**Interfaces:**
- Consumes: simulationService, quizService, questions e store.
- Produces: cinco modos, cronômetro, navegação, revisão e histórico de erros.

- [ ] **Step 1: Testar fluxo sem correção antecipada**

Responder, avançar, voltar, marcar, tentar finalizar com lacunas, confirmar finalização e só então encontrar o resultado.

- [ ] **Step 2: Implementar catálogo de simulados**

Rápido 10; completo 30/40 min; categoria parametrizada; difícil 30 média/difícil; erros com estado vazio quando insuficiente.

- [ ] **Step 3: Implementar execução resiliente**

Persistir sessão ativa a cada resposta; restaurar simulado interrompido; cronômetro baseado em timestamp para não derivar durante aba suspensa; confirmar antes do fim.

- [ ] **Step 4: Implementar resultado e erros**

Resultado por categoria, classificação, tempo, marcadas, revisar/refazer/voltar. Erros acumulam `attempts`, `lastAttemptAt`, `selectedAnswer`, `correctAnswer`, `reviewed` sem apagar histórico.

- [ ] **Step 5: Verificar e commit**

Run: `npm run test:run -- src/tests/simulationFlow.integration.test.jsx src/tests/mistakes.test.js`
Expected: PASS.

```bash
git add src/pages src/components/simulations src/tests
git commit -m "feat: add simulations results and mistake review"
```

### Task 10: Placas e flashcards

**Files:**
- Create: `src/data/plates/index.js`
- Create: `src/assets/plates/*.svg`
- Create: `src/pages/PlatesView.jsx`
- Create: `src/pages/PlateDetails.jsx`
- Create: `src/pages/FlashcardsView.jsx`
- Create: `src/components/plates/PlateCard.jsx`
- Create: `src/components/flashcards/Flashcard.jsx`
- Create: `src/hooks/useFlashcards.js`
- Create: `src/services/flashcardService.js`
- Test: `src/tests/platesData.test.js`
- Test: `src/tests/flashcardService.test.js`

**Interfaces:**
- Produces: `plates`, `searchPlates`, `orderFlashcards(progress, now)` e ações de favorito/avaliação.

- [ ] **Step 1: Testar schema e ordenação**

Validar IDs, categorias, asset local, nome, significado, finalidade, uso e dica. Ordenação deve priorizar `forgot`, `almost`, depois `remembered` por data crescente.

- [ ] **Step 2: Criar conjunto seguro de placas e SVGs locais**

Incluir somente placas conhecidas representadas corretamente. Adicionar qualquer dúvida visual ao `CONTENT_REVIEW.md` e omitir imagem insegura.

- [ ] **Step 3: Implementar biblioteca e detalhes**

Busca normalizada, filtro, grade, favorito e questão relacionada. Navegação interna usa `activeView` + `selectedPlateId`.

- [ ] **Step 4: Implementar flashcards acessíveis**

Virar por botão/teclado, não apenas hover; frente e verso semanticamente anunciáveis; três avaliações persistidas.

- [ ] **Step 5: Testar e commit**

Run: `npm run test:run -- src/tests/platesData.test.js src/tests/flashcardService.test.js`
Expected: PASS.

```bash
git add src/data/plates src/assets/plates src/pages src/components src/hooks src/services src/tests CONTENT_REVIEW.md
git commit -m "feat: add plates library and adaptive flashcards"
```

### Task 11: Progresso, conquistas e configurações

**Files:**
- Create: `src/pages/ProgressView.jsx`
- Create: `src/pages/AchievementsView.jsx`
- Create: `src/pages/SettingsView.jsx`
- Create: `src/components/progress/StatisticsCard.jsx`
- Create: `src/components/progress/PerformanceChart.jsx`
- Create: `src/components/achievements/AchievementCard.jsx`
- Create: `src/components/achievements/AchievementToast.jsx`
- Create: `src/components/common/ConfirmationModal.jsx`
- Create: `src/hooks/useTheme.js`
- Test: `src/tests/settings.integration.test.jsx`

**Interfaces:**
- Consumes: estatísticas, conquistas, storage e import/export.
- Produces: gráficos reais, gestão de perfil/preferências e recuperação de dados.

- [ ] **Step 1: Testar tema, exportação, importação e reset**

Verificar preferência do sistema no primeiro acesso, persistência de tema, importação só após confirmação e reset preservando somente preferências técnicas necessárias.

- [ ] **Step 2: Implementar progresso e gráficos leves**

SVG/CSS para acertos por matéria, evolução, dias e semanas; estado vazio com CTA; nenhuma biblioteca de charts e nenhum valor fictício.

- [ ] **Step 3: Implementar conquistas**

Grid com progresso/estado/data e toast discreto uma única vez por desbloqueio.

- [ ] **Step 4: Implementar configurações completas**

Editar nome/categoria/data/dificuldade, sons, tema, reduzir animações, exportar, selecionar/importar JSON com preview e reset com modal destrutivo.

- [ ] **Step 5: Testar e commit**

Run: `npm run test:run -- src/tests/settings.integration.test.jsx`
Expected: PASS.

```bash
git add src/pages src/components src/hooks src/tests
git commit -m "feat: add progress achievements and settings"
```

### Task 12: PWA, offline, documentação e QA final

**Files:**
- Create: `public/icons/icon-192.png`
- Create: `public/icons/icon-512.png`
- Create: `public/icons/maskable-512.png`
- Create: `src/components/common/InstallPWA.jsx`
- Create: `src/components/common/OfflineNotice.jsx`
- Create: `src/hooks/usePWAInstall.js`
- Create: `README.md`
- Modify: `vite.config.js`
- Modify: `src/App.jsx`
- Test: `src/tests/pwaUi.test.jsx`

**Interfaces:**
- Produces: manifest, service worker, instalação, shell offline, documentação de GitHub/Vercel e entrega verificada.

- [ ] **Step 1: Escrever teste da UI de instalação/offline**

Simular `beforeinstallprompt`, dispensar e confirmar persistência; disparar `offline`/`online` e verificar aviso discreto.

- [ ] **Step 2: Configurar PWA**

Manifest: nome `CNH Sem Segredo`, curto `CNH 30 Dias`, `display: standalone`, `start_url: /`, cores do projeto e ícones. Cachear shell e assets locais; não criar requisições externas.

- [ ] **Step 3: Escrever README operacional**

Documentar `npm install`, dev, teste, build, preview, criação do GitHub, push, importação na Vercel, domínio, atualização de conteúdo/questões e ausência intencional de token/prazo de acesso.

- [ ] **Step 4: Executar validação automatizada completa**

Run: `npm run lint`
Expected: exit 0, zero erros.

Run: `npm run test:run`
Expected: exit 0, todos os testes passando.

Run: `npm run build`
Expected: exit 0, `dist/` gerado com service worker e manifest.

- [ ] **Step 5: Executar QA no Browser/IAB**

Testar 320, 360, 390, 430 px, tablet e desktop; onboarding; navegação; Dia 7; persistência após reload; quiz; cinco simulados; timer; erros; placas; flashcards; estatísticas; conquistas; tema; export/import/reset; instalação e offline. Inspecionar console em cada fluxo.

- [ ] **Step 6: Comparar implementação com conceitos aprovados**

Capturar home mobile/desktop e estados de aula/simulado; usar `view_image` nos conceitos e renders; registrar em `docs/design/fidelity-ledger.md` pelo menos copy, composição, tipografia, paleta, ícones, espaçamento, responsividade e interações. Corrigir todo desvio não intencional.

- [ ] **Step 7: Verificação final e commit**

Reexecutar `npm run lint && npm run test:run && npm run build` após a última correção e salvar evidência no handoff.

```bash
git add public src vite.config.js README.md docs/design CONTENT_REVIEW.md
git commit -m "feat: complete offline-ready CNH study app"
```
