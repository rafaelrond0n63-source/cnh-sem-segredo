# CNH Sem Segredo — Design da Aplicação

## Objetivo

Construir uma SPA mobile-first em React, Vite e JavaScript para uma jornada de preparação teórica da CNH em 30 dias. O produto deve funcionar como aplicativo instalado, armazenar todo o progresso localmente, operar offline após o primeiro carregamento e abrir sempre na raiz do domínio.

O aplicativo é uma ferramenta complementar, não possui vínculo oficial com o DETRAN e não promete aprovação.

## Decisão de acesso

A primeira versão não terá token, autenticação nem prazo de expiração. O aplicativo abre diretamente no onboarding no primeiro acesso e na tela inicial nos acessos seguintes.

Não serão criados `AccessGuard`, `AccessNotFound`, `AccessExpired`, `accessService`, `useAccess` ou `accessConfig`. A separação entre inicialização, persistência e interface permitirá incluir autenticação real futuramente sem reestruturar os recursos de estudo.

## Arquitetura

A aplicação será uma SPA sem React Router. `App.jsx` coordenará inicialização e composição, enquanto um estado `activeView` trocará as telas sem alterar a URL ou recarregar a página.

O código será dividido por responsabilidade:

- componentes de layout e controles reutilizáveis;
- páginas internas para os principais fluxos;
- store/contexto React para estado e ações da aplicação;
- hooks para consumo de estado e comportamentos de interface;
- serviços puros para persistência, progresso, quizzes, simulados, estatísticas, conquistas e importação/exportação;
- dados estáticos separados para os 30 dias, questões, placas, conquistas e mensagens;
- utilitários puros e validáveis isoladamente.

Telas pesadas serão carregadas com `React.lazy` e `Suspense`. Nenhuma regra de negócio relevante ficará presa aos componentes visuais.

## Experiência e identidade visual

O produto adotará azul-escuro `#071A2D`, azul intermediário `#0E3557`, amarelo `#FFC928`, branco, cinzas neutros e cores semânticas de sucesso, erro e atenção. A direção visual será premium, adulta, limpa e confiante, com cantos moderadamente arredondados, sombras discretas, áreas de toque grandes e hierarquia tipográfica clara.

No celular, a navegação principal será uma barra inferior fixa com Início, Plano, Simulados, Placas e Progresso. No desktop, será uma barra lateral compacta. Flashcards, Erros, Conquistas e Configurações serão acessos secundários.

O primeiro viewport da tela inicial priorizará uma única missão seguinte, progresso real e uma ação principal. Cards estatísticos e atalhos aparecerão abaixo, evitando aparência de painel administrativo. O modo escuro manterá a mesma hierarquia e contraste. Animações serão curtas e respeitarão `prefers-reduced-motion`.

## Inicialização e onboarding

Uma splash curta será exibida somente enquanto os dados são carregados e validados. No primeiro acesso, o usuário fará onboarding em cinco etapas: introdução, nome, categoria, data opcional da prova e principal dificuldade.

Campos serão validados antes de avançar e datas passadas serão recusadas. Ao concluir, os dados serão persistidos e a experiência abrirá na tela inicial. Reiniciar o progresso permitirá refazer o onboarding.

## Estado e persistência

Todo o estado persistente usará a chave versionada `cnhSemSegredoApp_v1`. Componentes não acessarão `localStorage` diretamente.

`storageService` fornecerá carregamento, salvamento, atualizações parciais, restauração segura, migração, validação, exportação, importação e reinicialização. Falhas de armazenamento ou JSON corrompido resultarão em valores padrão e uma mensagem amigável, sem travar a aplicação.

O estado conterá perfil, preferências, progresso diário, respostas, erros, simulados, conquistas, placas favoritas, flashcards, datas de atividade, estatísticas e o desbloqueio integral do plano. A exportação nunca incluirá segredos — esta versão não terá nenhum token — e a importação só substituirá dados após validação e confirmação.

## Plano, aulas e desbloqueio

Os 30 dias serão organizados em quatro semanas. Cada dia terá título, matéria, duração estimada, etapas, conteúdo em blocos curtos e atividade avaliativa. O progresso interno será salvo por etapa e abrir uma aula não a concluirá.

Os dias 1 a 7 estarão disponíveis em qualquer ordem. Os dias 8 a 30 permanecerão bloqueados até todas as etapas obrigatórias do Dia 7 serem concluídas. O desbloqueio será persistido e acompanhado de celebração. Concluir o Dia 30 abrirá a experiência de conclusão com estatísticas reais e próximos passos.

## Questões e simulados

O banco conterá pelo menos 180 questões originais distribuídas pelas sete categorias definidas. Cada questão terá ID único, quatro alternativas, resposta válida, explicação, dificuldade e associação temática. Um validador de desenvolvimento e testes automatizados impedirão dados incompletos ou duplicados.

O modo aprendizagem exigirá confirmação antes da correção e mostrará explicação imediata. O modo simulado ocultará a correção, permitirá navegar e marcar questões, confirmará a finalização e calculará o resultado somente ao final.

Serão oferecidos simulados rápido, completo, por matéria, difícil e baseado em erros. Resultados, tempo, desempenho por categoria e respostas serão persistidos. Questões erradas manterão histórico mesmo após revisão correta.

Conteúdos regulatórios sujeitos a mudança serão evitados ou listados em `CONTENT_REVIEW.md` para conferência oficial antes do lançamento comercial.

## Placas e flashcards

A biblioteca terá pesquisa, categorias, favoritos, detalhes e flashcards. Apenas placas representadas com segurança serão incluídas; os desenhos serão SVGs locais, sem URLs externas. Itens que exijam validação visual serão documentados em `CONTENT_REVIEW.md`.

Flashcards registrarão “não lembrei”, “quase lembrei” e “lembrei”. Sessões futuras priorizarão dificuldades e itens revisados há mais tempo.

## Estatísticas, sequência e conquistas

Estatísticas serão derivadas exclusivamente da atividade real. Serviços puros calcularão porcentagem do plano, acertos, erros, média, melhor simulado, tempo estudado, desempenho por matéria e sequência de dias ativos.

Gráficos serão construídos com SVG e CSS, sem biblioteca pesada. Estados vazios orientarão a primeira ação sem inventar números. Conquistas serão avaliadas após ações relevantes, persistidas com data e apresentadas com feedback discreto.

## PWA, offline e privacidade

`vite-plugin-pwa` gerará manifest, service worker, cache do shell e recursos locais. Um aviso discreto indicará modo offline. O convite de instalação poderá ser dispensado e essa preferência será persistida.

Não haverá backend, analytics, fontes remotas ou requisições externas. O documento HTML usará `noindex, nofollow`. Dados pessoais e progresso não serão enviados nem registrados no console.

## Acessibilidade e responsividade

Controles usarão semântica, labels, estados ARIA, foco visível e áreas de toque adequadas. Mensagens dinâmicas importantes serão anunciadas a leitores de tela. Modais terão foco controlado e serão utilizáveis por teclado.

A interface será validada em 320, 360, 390 e 430 px, tablet e desktop. A barra inferior respeitará safe-area e nunca cobrirá conteúdo. Não haverá rolagem horizontal, texto cortado ou modal maior que a viewport.

## Tratamento de erros

Um `ErrorBoundary` protegerá a árvore da aplicação. Estados explícitos cobrirão carregamento, armazenamento indisponível, conteúdo ausente, simulado interrompido e importação inválida. Erros técnicos serão convertidos em mensagens acionáveis, com recuperação sempre que possível.

## Estratégia de testes

Vitest e React Testing Library cobrirão:

- persistência, dados corrompidos, migração, importação e exportação;
- progresso diário, porcentagem e desbloqueio após o Dia 7;
- sequência de estudos, estatísticas e conquistas;
- correção de quizzes, resultados de simulados e revisão de erros;
- validação integral das 180 questões;
- onboarding, navegação e persistência entre remounts;
- integração do fluxo onboarding → plano → Dia 7 → desbloqueio persistido.

A validação final incluirá lint, suíte completa, build de produção, teste do service worker e inspeção no navegador em desktop e mobile. O fluxo antigo de token não será testado porque foi removido por decisão do produto.

## Entrega e documentação

O repositório incluirá configuração de Vite, Tailwind CSS, ESLint, Vitest, PWA e Vercel, além de `.gitignore`, README e `CONTENT_REVIEW.md`. O README explicará instalação, desenvolvimento, testes, build, GitHub, Vercel, domínio, atualização de conteúdo e ajustes do período de estudo. Não haverá instruções de token ou prazo de acesso, pois esses recursos não existem nesta versão.

## Critérios de conclusão

A entrega só será considerada concluída quando não houver erros de lint, testes ou build; os principais fluxos estiverem funcionais; os dados persistirem; o desbloqueio estiver correto; a PWA funcionar; e a interface estiver verificada em navegador real nos tamanhos prioritários.
