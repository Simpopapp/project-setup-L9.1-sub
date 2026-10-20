# PRD — projeto 1 (consolidado): PTE Master Hub — plataforma unificada de preparo PTE

> Fonte do escopo: `.opencode/project1` (planejamento consolidado). Roadmap: `.opencode/roadmap-proj1`.
>
> Este PRD é o COMO do trio consolidado. **Substitui** `prd-project1.md … prd-project9.md`: absorve as
> decisões técnicas, o escopo por fase, os critérios de aceite, os veredictos de fonte e o registro de
> gates de cada um, numa numeração única (§0). Não repete o planejamento — confirma o que ele afirma,
> fixa o estado final de cada decisão e registra o que foi superado (§6.2).
>
> Estado de partida (herdado dos PRDs de origem, 10–11/10/2026): projetos 1–9 com todas as fases
> concluídas, exceto um gate de verificação offline em build publicado (§12). Gates de referência:
> vitest 343/343 · tsgo limpo · build OK · lint 0 erros (8 warnings pré-existentes) · axe 0 violações
> graves nas 10 rotas principais. As marcas "✅ confirmado" vêm dos PRDs de origem (checadas no código
> na época); antes de reabrir qualquer fase, reconfirme no código atual — o agente conhece o ambiente
> melhor que este documento.

---

## 0. Como ler este PRD

### 0.1 Convenções (numeração única)

| Símbolo | Significado |
|---|---|
| **Fase N** / **N.M** | Fase e sub-fase da numeração única (1–14). Roadmap, planejamento e PRD usam as mesmas. |
| **DT{n}** | Decisão técnica (§6). Numeração única; a origem fica na coluna "Origem" (ex.: `p9·D11`). |
| **R{n}** | Restrição permanente (§2). |
| **A1–A5 · B1–B6 · C1–C6 · D1–D3** | Códigos de achado do antigo projeto 9, **mantidos** porque nomeiam probes e testes (ex.: "probe B5"). Não confundir com `DT`. |
| **I1–I8** | Invariantes do contrato do glossário (§7.7), usadas por `merge-glossary.mjs` e por `plus-glossary.test.ts`. |
| `pN·FM` / `pN·DK` | Fase M / decisão K do projeto N original — só para rastreio. |

### 0.2 Mapa: numeração antiga → numeração única

| Fase única | Tema | Origem (projeto·fase) |
|---|---|---|
| 1 | Base de dados e catálogo unificado | p1·F0–F1 |
| 2 | Motores de prática A–D (16 tipos) | p1·F2–F5 |
| 3 | Hub, sessões persistentes e SRS em toda a app | p1·F6 · p2·F1–F2 |
| 4 | Diagnóstico de erros, notas e atalhos | p2·F3–F4 |
| 5 | Simulado v1, PWA offline v2 e gráficos de evolução | p2·F5–F6 |
| 6 | Meta, prontidão v1, plano do dia e relatório semanal | p3·F1–F4 |
| 7 | Expansão confiável do banco e novos tipos (19 tipos) | p3·F0+F5 · p4·F0–F4 |
| 8 | Consistência de métricas e SEO dinâmico | p5·F1–F4 |
| 9 | Pack "studie-plus": tradução, áudio de pronúncia, mobile-first, glossário | p6·F0–F5 · p7·F0–F5 · p8·F0–F4 |
| 10 | Integridade do progresso (storage v6) e SRS correto | p9·F0–F2 |
| 11 | Medição honesta, escala 10–90 e prontidão v2 | p9·F3–F4 |
| 12 | Banco rastreável (proveniência, duplicatas, payload) e tipos 22/22 | p9·F5 · F7 |
| 13 | Offline real e simulado fiel | p9·F6 · F8 |
| 14 | Acessibilidade, mobile e fechamento | p9·F9 + gate offline pendente |

### 0.3 Referências legadas (o que fazer com nomes antigos)

Nomes como `roadmap-proj6`, `prd-project9.md` ou "Fase 3 do proj7" ainda aparecem em `AGENTS.md`, comentários
de código e no histórico git. **Não editar esses arquivos por causa disso** (fora do trio). Resolver pela tabela
§0.2: o nome antigo aponta para a Fase única correspondente neste PRD e no roadmap. Os scripts em
`.opencode/proj6-work/`, `proj8-work/`, `proj9-work/` e `.opencode/pte-json/` continuam onde estão.

### 0.4 Precedência (Plan.md)

No escopo (o quê): planejamento > PRD > roadmap. No como: o PRD manda. Dúvida → decidir e registrar aqui
(nunca perguntar ao usuário). Se PRD e roadmap divergirem do planejamento (título, fases, escopo),
tratá-los como desatualizados: refazer o PRD e reabrir o roadmap.

---

## 1. Estado consolidado (o que existe hoje)

### 1.1 Produto

Hub de treino PTE Academic, 100% local (sem auth, nuvem ou multiusuário), com:

- **22 task types** em 4 seções (Speaking, Writing, Reading, Listening) — 19 originais/expandidos + SGD, RTS e
  Describe Image (Fase 12.2); prática individual para todos, simulado cronometrado e sessões híbridas.
- **Treino inteligente:** SRS (Leitner) em toda a app, fila do dia, plano do dia por habilidade, diagnóstico
  de erros, notas, atalhos, histórico de sessões, relatório semanal.
- **Medição honesta:** subscores rotulados, estimativa de treino 10–90 com nível de confiança, prontidão por
  desempenho recente (não por tamanho do banco), pronúncia "não avaliada".
- **Integridade:** `AppState` v6 com snapshot SRS por questão, quarentena de estado ilegível, import com
  prévia/mesclar/desfazer, aviso preventivo de quota.
- **Offline/PWA:** service worker versionado (só em PROD), "Baixar para offline", aviso de nova versão.
- **Pack "studie-plus"** (desligado por padrão): tradução por palavra (glossário), tradução por frase, áudio de
  pronúncia do inglês — separado do banco nativo e do conteúdo oficial.
- **Acessibilidade:** axe limpo, teclado completo nos motores, voz TTS realmente usada exibida.

### 1.2 Evolução das versões de dados

| Versão | Entrou em | Mudança |
|---|---|---|
| v1 | (original) | Só Read Aloud (148 questões) |
| v2 | Fase 1.3 | Métricas por tipo; 16 tipos |
| v3 | Fase 3.2 | `notes`, `sessions`, `draft`; `Attempt.tags` aditivo |
| v4 | Fase 6.1 | `Settings.examDate`, `Settings.targetScore` |
| v5 | Fase 9.1 | `Settings.plus { wordTranslation, phraseTranslation, plusAudio }` |
| v6 | Fase 10.2 | `questions` (snapshot SRS), `daily`, log truncável; quarentena; undo |

### 1.3 Banco (referência histórica — a UI sempre deriva do manifesto, DT31)

- Inventário inicial: 2.032 questões / 16 tipos (Fase 1). Após a Fase 7.2: 2.164 / 19. Fim da Fase 7.3:
  **2.536 no manifesto** / 19 tipos (speaking 1.003 · writing 122 · reading 243 · listening 1.168).
- Fase 12.2 acrescentou SGD 13 · RTS 63 · Describe Image 6 (soma 82 → conferir o total atual no manifesto;
  este PRD não o fixa, DT31).
- Contagem por tipo no fim da Fase 7.3 (19 tipos): wfd 934 · repeat_sentence 586 · read_aloud 195 ·
  answer_short 141 · summarize_spoken_text 115 · mcq_single 106 · lst_fib 96 · retell_lecture 81 · swt 79 ·
  read_fib 62 · reorder_paragraphs 57 · write_essay 43 · mcq_multi 18 · lst_mcq 6 · lst_summary 5 ·
  lst_hcs 3 · lst_hiw 3 · lst_smw 3 · lst_mcq_multi 3 (soma = 2.536).
- Ambiguidade herdada: os projetos 4–5 registraram "5.072 questões nos JSONs" (= 2 × 2.536) contra 2.536 no
  manifesto. Origem da diferença não foi investigada nos PRDs de origem; o **manifesto é a fonte canônica**
  de qualquer número exibido (DT31). Ver §12.3.

---

## 2. Restrições permanentes (consolidadas e sem repetição)

| # | Restrição |
|---|---|
| R1 | **Sem auth, sem nuvem, sem multiusuário.** 100% localStorage (IndexedDB só se medido > 3,5 MB, DT7). Backup JSON versionado é o envelope; campo novo entra com migração pura e testável, sem perda. |
| R2 | **TTS/STT pela Web Speech API.** Nenhum serviço ou biblioteca paga de voz. Sotaque do TTS vem de `settings.accent`, velocidade de `modelRate`. |
| R3 | **Zero dependências novas de runtime.** Ferramentas de verificação (Playwright, axe, wrangler) ficam fora do `package.json` de runtime. |
| R4 | **Código puro separado de UI.** Lógica nova vive em `src/lib/pte/*`, `src/lib/plus/*` ou `src/lib/*`, como funções puras testadas antes da UI; rotas e componentes só orquestram. |
| R5 | **Performance do banco grande.** Listas e navegação usam só `manifest.ts`; conteúdo completo sob demanda via `catalog.ts` (import dinâmico por tipo). |
| R6 | **Gates por fase:** `bunx vitest run` verde (contagem só cresce) · `tsgo --noEmit` limpo · `bun run build` OK · `bun run lint` 0 erros · preview :8080 verificado por Playwright nas rotas tocadas · sem `console.error` novo. |
| R7 | **Dados do pack separados do banco nativo.** `src/data/plus/` nunca se mistura a `src/data/pte/`; carregado por `import()` dinâmico só quando o recurso está ativo. |
| R8 | **Áudio do pack ≠ áudio nativo; tradução do pack ≠ texto do banco.** O áudio do pack fala sempre o **inglês original**; tradução é só texto; nunca "áudio da tradução". |
| R9 | **Pack 100% desligado por padrão**, com rótulo explícito "recurso extra de estudo, não faz parte do PTE". Integrações só por tap/click explícito (nada abre em hover/scroll). |
| R10 | **Sem números fixos na UI/SEO** quando existir constante derivada do manifesto (`TOTAL_ALL_QUESTIONS`, `sectionCounts`, `TYPE_META`). |
| R11 | **Não editar arquivos gerados** (`src/routeTree.gen.ts`); nenhum segredo escrito/ecoado; história git intacta. |
| R12 | **Sem correção por IA/LLM e sem prometer nota oficial.** Toda estimativa é "estimativa de treino". Não apagar questões sem registro. |
| R13 | **Dados novos:** fonte verificável, acessível sem login, com gabarito/transcript, **sem conteúdo gerado por IA**, dedupe contra o banco e entre fontes (§7). |

---

## 3. Modelo de dados final

### 3.1 AppState v6 (`src/lib/storage.ts`)

```ts
interface QuestionSnapshot {          // 1 por canonicalId — fonte de verdade do SRS (DT7)
  attempts: number;                   // total, inclui tentativas já truncadas do log
  bestScore: number | null;
  lastScore: number | null;
  lastAttemptAt: number;
  strongAttempts: number;
  box: number;                        // 0–5 (Leitner)
  dueAt: number;
}
interface DailyAggregate { attempts: number; scored: number; sum: number } // média = sum/scored
interface AppState {
  version: 6;
  attempts: Attempt[];                // LOG truncável (teto 2.000) — só auditoria (DT8)
  questions: Record<string, QuestionSnapshot>;
  daily: Record<string, DailyAggregate>;   // chave "YYYY-MM-DD" local (DT12)
  dailyCutoff: string;                // data mais antiga retida (180 dias)
  bookmarks: string[];
  notes: Record<string, QuestionNote>;     // QuestionNote { text, updatedAt }
  sessions: SessionRecord[];          // histórico, mais recente primeiro, teto 100
  draft: SessionDraft | null;         // sessão retomável
  settings: Settings;
}
```

- `Attempt` (aditivo desde v3): `tags?: string[]` — rótulos de erro (`missed:palavra`, `trap`, categoria de fala,
  `"exam"`). `metrics` continua numérico por contrato (DT13).
- `SessionRecord { id, startedAt, finishedAt, kind: "srs"|"mixed"|"errors"|"due"|"exam", label, results }`;
  `SessionDraft { kind, label, queue, index, startedAt, results }`.

### 3.2 Settings v6

Campos: `prepSeconds`, `dailyGoal`, `modelRate`, `accent` (`en-AU` padrão, `en-US`, `en-GB`), `examDate`
(ISO `YYYY-MM-DD` | `null`), `targetScore` (10–90; padrão 65; opções de UI 50/58/65/79),
`plus { wordTranslation, phraseTranslation, plusAudio }` (tudo `false`), filtro "só fontes verificadas"
(padrão **desligado**, DT26), lembrete de backup (data do último backup) e flag do aviso de microfone.
Os nomes exatos dos três últimos campos seguem `storage.ts` (os PRDs de origem os descrevem sem fixar o nome).
`normalizeSettings` rejeita data inválida, alvo fora de 10–90 e flags não-booleanas (→ `false`).

### 3.3 Cadeia de migração (pura, em `migrateLegacyState`)

| De → Para | Regra |
|---|---|
| v1 → v2 | Migração pura: `Attempt` passa a carregar `taskType` + `metrics`; métricas por tipo derivadas. |
| v2 → v3 | `notes = {}`, `sessions = []`, `draft = null`. |
| v3 → v4 | `examDate = null`, `targetScore = 65` quando ausentes. |
| v4 → v5 | `plus` desligado quando ausente. |
| v5 → v6 | **Com o log completo**: replay cronológico semeia `questions` e `daily` **antes** de qualquer corte; só então trunca o log (2.000) e retém 180 dias de `daily`; funde duplicatas por `canonicalId` (DT27). Reexecução v6→v6 (`srsRules: 2`) recalcula caixas/`dueAt` pelas regras novas do SRS (DT11) sem perder tentativas. |

- Backup: envelope `version: 6`; `importBackup` aceita 2–6 (migra). Export produz v6.
- Estado ilegível → quarentena (`pte-master-hub-quarantine-<ts>`), nunca sobrescreve; versão futura (v7+) →
  somente-leitura com aviso, preservada byte a byte (DT9).
- Import: prévia → **Substituir | Mesclar** → cópia automática em `pte-master-hub-undo` → **Desfazer** (DT10).
- Teto do histórico de sessões: `MAX_SESSIONS = 100`. `MAX_ATTEMPTS` deixou de ser 5.000 e vale só para o log.

### 3.4 Dados do banco

```
.opencode/pte-json/      # fonte de manutenção: um JSON por task type
src/data/pte/*.json      # cópia consumida pelo app (import dinâmico por tipo, R5)
src/lib/pte/manifest.ts  # metadados leves + TOTAL_ALL_QUESTIONS, sectionCounts, TYPE_META, selos
manifest.json            # gerado por scripts/build-manifest.mjs
```

- Ids estáveis `<fonte>-<tipo>-NN`; campo `src` = domínio da fonte. `canonicalId` no manifesto/catálogo (DT27).
- `wfd.json` slim (DT28): contrato `id/answer/source/word_count`; `text`/`transcript`/`audio_text` derivados no catálogo.
- Selos de proveniência: `verificada` · `comunitária` · `ia` (mapa estático em `manifest.ts`, §7.3).

### 3.5 Dados do pack (fora do envelope de backup e do banco nativo)

```
src/data/plus/
  glossary.json            # { [palavraMinúscula]: { literal, contextual? } }   EN → pt-BR
  phrases/{task_type}.json # { [qid]: { t: string[] } } — t[i] = tradução da i-ésima frase
```

Frases alinhadas **por índice** de `splitSentences` (nunca fuzzy, DT37). Cobertura de frases: read_aloud,
reorder_paragraphs, mcq_single, mcq_multi, read_fib, wfd, swt e write_essay (3.933 frases, 100%); tipos só-áudio
ficam de fora por design (a transcrição é gabarito, DT39).

---

## 4. Arquitetura: motores e módulos

### 4.1 Motores de prática (despachante por `taskType` em `/practice`)

| Motor | Tipos | Interação | Núcleo puro |
|---|---|---|---|
| **A — Voz & acústica** | read_aloud, repeat_sentence, retell_lecture, answer_short, + SGD, RTS, describe_image | Prep adaptativo (RA 35–40 s · RS 3 s · RL 10 s · ASQ 3 s · RTS 10 s + 40 s · DI 25 s + 40 s) → gravação → alinhamento | `scoring.ts` (LCS: hit/missed/substituted/moved), `pte/speaking.ts` |
| **B — Escrita & ditado** | wfd, swt, summarize_spoken_text, write_essay | Digitação com contador e zona segura; cronômetros (SST 10:00, essay 20:00) | `pte/dictation.ts`, `pte/summary.ts`, `pte/essay.ts` |
| **C — Áudio interativo & lacunas** | lst_fib, lst_hiw, lst_smw, lst_hcs, lst_summary, lst_mcq, lst_mcq_multi | Áudio TTS de reprodução única; lacunas inline; clique em palavras | `pte/dictation.ts`, `pte/mcq.ts` |
| **D — Leitura** | mcq_single, mcq_multi, reorder_paragraphs, read_fib | Passage, rádio/checkbox com saldo líquido, pares adjacentes, dropdowns inline | `pte/mcq.ts`, `pte/reorder.ts`, `pte/fibDropdown.ts` |

Componentes-chave: `PracticeShell` (cabeçalho, chip de revisão, notas, atalhos, `TtsVoiceNote`), `ScorePanel`
(subscores), `PassageText`, `ExamRunner`. `practice.tsx` é despachante; os motores recebem o subtipo discriminado.

### 4.2 Módulos puros por tema

| Tema | Módulos |
|---|---|
| Dados | `pte/manifest.ts`, `pte/catalog.ts`, `pte/types.ts`, `pte/skillMatrix.ts` |
| Persistência | `storage.ts` (v6), `dates.ts` (chaves de dia local) |
| Sessão/SRS | `session.ts` (`buildQueue`, modos mixed/due/errors, `startMixedSession`, draft) |
| Diagnóstico | `pte/diagnostics.ts` (`tagAttempt`, `diagnoseAttempts`, `errorBreakdown`, `topMissedWords`, `errorTrend`, `worstTopics`) |
| Meta/prontidão | `pte/goal.ts`, `pte/estimate.ts`, `pte/readiness.ts`, `pte/studyPlan.ts`, `pte/report.ts` |
| Exame | `pte/exam.ts` (`BLUEPRINT`, `EXAM_PLAN`, `buildExamQueue`, `examTiming`, `gradeExamSummary`) |
| Medição | `scoring.ts`, `pte/templateDetector.ts` |
| Pack | `plus/settings.ts`, `plus/glossary.ts`, `plus/words.ts`, `plus/sentences.ts`, `plus/phrases.ts`, `plus/plusAudio.ts` |
| Voz | `useSpeechSynthesis.ts` (`speak(text, rate, langOverride?)`, `pickVoice`), `useSpeechRecognition.ts` |

---

## 5. Escopo por fase (o COMO) — Fases 1 a 14

Gates de cada fase = R6; o registro numérico (vitest por marco etc.) fica no roadmap. Cada fase abaixo descreve o que foi entregue
(estado final) e o critério de aceite que o verificou.

### Fase 1 — Base de dados e catálogo unificado (p1·F0–F1) ✅

**Objetivo:** sair de um app de Read Aloud (148 questões) para uma base única de 16 task types / 2.032 questões.

- **1.1 Dados:** 16 JSONs consolidados em `.opencode/pte-json/`; `.opencode/pte.json` virou manifesto de
  redirecionamento; cópia em `src/data/pte/`.
- **1.2 Catálogo:** `src/lib/pte/` com `Question` discriminada por `task_type`; busca por `section`,
  `task_type`, `topic`, `id`; `answer` de FIB normalizado de string JSON → `string[]` (DT1).
- **1.3 Storage v2:** `computeTypeStats`, `attemptedToday`, `recentAverage`, `computeStreak`, `averageWpm`;
  migração pura v1→v2; backup versionado (DT2).
- **1.4 Hub:** Home = dashboard por seção; TopNav (Início/Falar/Escrever/Ler/Ouvir/Sessão/Progresso/Ajustes);
  rotas `/speaking|writing|reading|listening` com `SectionPage` (cards por tipo, filtros busca/tipo/status).

**Aceite:** 31/31 testes; tsgo, build, lint limpos; preview verificado (home, 4 seções, `/practice`);
`FilterBar.tsx` sem uso removido.

### Fase 2 — Motores de prática A–D (p1·F2–F5) ✅

- **2.1 Ditado e lacunas (B/C):** WFD (TTS 1×, digitação, +1/palavra, mapa acertos/omissões/extras,
  normalização de casing/pontuação), FIB `lst_fib` (inputs inline, Tab, correção tolerante a casing), HIW
  (texto clicável; acertos − falsos, mín. 0). Lógica em `pte/dictation.ts`. `practice.tsx` reescrito como
  despachante; `ReadAloudPractice` extraído sem mudar comportamento. **Correção:** `useSpeechSynthesis` passou
  a definir `supported` em `useEffect` (mismatch de hidratação deixava botões mortos).
- **2.2 Escrita (B):** `pte/summary.ts` (`gradeSwt`/`gradeSst`): SWT — validador de frase única (1 ponto final
  terminal), zona 5–75 palavras, keywords por frequência, conectivos; SST — TTS 1× → 10:00 regressivo →
  auto-submit, zona 50–70, cobertura semântica contra a transcrição.
- **2.3 Fala (A):** Read Aloud (prep 35 s, gravação ≤ 40 s, medidor de mic via `AnalyserNode`, transcrição
  interim, alinhamento por sequência, proxies de fluência, replay via `MediaRecorder`); Repeat Sentence
  (áudio → 3 s → gravação → alinhamento com transcript); Retell Lecture (10 s + 40 s → keywords + fluência,
  notas de referência no resultado); Answer Short (matching exato/sinônimos, ex.: "autumn or fall").
- **2.4 Leitura e áudio interativo (C/D):** `McqPractice` genérico para 6 tipos (`pte/mcq.ts`: rádio/checkbox,
  saldo líquido +1/−1 mín. 0, painel de traps e evidência); HCS/Summary/SMW com player de reprodução única;
  `lst_summary` reutiliza o motor SST (`SstPractice` aceita `SstQuestion | SummaryChoiceQuestion`).
  **Correção de dado:** `lst_smw` `pte-lst_sm-40c6f847` tinha `answer` vazio → 0. `TYPE_META.ready` = true
  para os 16 tipos.

**Aceite:** cada tipo abre seu motor (nunca "Em breve"); 75/75 testes; mapa do ditado e saldo líquido do MCQ
cobertos por testes puros; preview verificado por tipo.

### Fase 3 — Hub, sessões persistentes e SRS em toda a app (p1·F6 · p2·F1–F2) ✅

**Objetivo:** o aluno retoma onde parou, vê a agenda em qualquer página e tem uma "fila do dia".

- **3.1 Sessões híbridas (p1·F6):** fila mista entre tipos, shuffle inteligente (fracas primeiro + menos
  recentes + intercalação), barra "3/10", resumo com média por task type; filtros (seção, tipo, tamanho,
  score, tentativas, nunca-tentadas) com paginação de 50 + "Mostrar mais"; dashboard 360° (barras e % iniciadas
  por seção na Home; `/progress` com os tipos e score unificado).
- **3.2 Persistência de sessão (p2·F1, v3):** `session.ts` grava o `draft` a cada resposta, restaura no load,
  `endSession()` arquiva `SessionRecord` (≥ 1 resultado) e limpa o draft; `/session` mostra banner
  "Retomar sessão de N" / "Descartar" e o histórico das últimas 10 sessões. **Correção:** `getServerSnapshot`
  devolvia estado vazio (bug de hidratação SSR) — notas e draft passaram a sobreviver ao reload.
- **3.3 SRS global (p2·F2):** o planejamento original afirmava que `/practice` não alimentava a agenda; o PRD
  de origem **corrigiu**: todo motor já chamava `recordAttempt`; faltava **visibilidade**. Entregue: `buildQueue`
  modo `"due"` (só vencidas, pior último score primeiro) + `countDueByType`; `DueNotice`/chip no `PracticeShell`
  ("Revisar hoje" / "Revisão em N dias"); badge "N vencidas hoje" por seção na Home e contagens nos filtros;
  botão **"Fila do dia (N)"** em `/session` (desabilitado com N = 0). Leitner com intervalos `[0,1,3,7,14,30]`.
  (As regras de promoção foram corrigidas na Fase 10.3.)

**Aceite:** reload com sessão em andamento devolve fila e índice; sessão concluída entra no histórico com média e
limpa o draft; migrações v1/v2→v3 testadas; "Fila do dia" só contém `dueAt ≤ now` (teste puro de `buildQueue`);
108/108 testes.

### Fase 4 — Diagnóstico de erros, notas e atalhos (p2·F3–F4) ✅

- **4.1 Diagnóstico (DT13–DT14):** `Attempt.tags` — WFD/FIB: até 5 palavras (`missed:palavra`); MCQ: `trap` do
  conteúdo + alternativa errada; fala: `fluencia` (WPM fora de 120–160), `conteudo` (alinhamento < 70),
  `pronuncia` quando existir. `pte/diagnostics.ts` agrega (`errorBreakdown`: conteúdo/fluência/pronúncia/
  omissão/extra/trap; `topMissedWords`; `errorTrend` só de tentativas com score < 70; `worstTopics`).
  `/progress` → "Padrões de erro" (barras, chips de palavras que levam a `/practice?q=`, tópicos que mais custam)
  com estado vazio elegante. `ERROR_SCORE = 70` segue como corte único de "tentativa fraca".
- **4.2 Notas e atalhos:** `upsertNote(qid, text)` (trim, máx. 2.000 caracteres, apaga se vazio); painel de nota
  no `PracticeShell` (tecla **N**, debounce 400 ms, indicador "salva"); atalhos `←`/`→` (ou `J`/`K`), `B`
  (favoritar), `N`, `?` (overlay), todos **ignorados com foco em input/textarea**.

**Aceite:** testes puros de `diagnostics.ts` (breakdown, ranking, tópicos); WFD errado grava `missed:*` visível
no backup; nota sobrevive ao reload; atalhos não disparam durante o ditado (Playwright); 125/125 testes.

### Fase 5 — Simulado v1, PWA offline v2 e gráficos de evolução (p2·F5–F6) ✅

- **5.1 Simulado (DT15):** `pte/exam.ts` (`EXAM_PLAN`, `buildExamQueue` estratificada, `examTiming`,
  `gradeExamSummary`); rota própria `/exam`: escolha (4 seções + integral, tempo total) → `ExamRunner`
  (cronômetro regressivo por questão, auto-submit, sem retry, reutilizando os motores) → resumo comparativo com
  a média histórica de prática. Tentativas entram no `Attempt` normal com `tags: ["exam"]`; sessão registrada
  com `kind: "exam"`; link "Simulado" na TopNav. Tempos iniciais: RA ~40 s · RS ~15 s · RL 10+40 s · ASQ ~10 s ·
  SWT/SST 10 min · MCQs ~2 min · WFD ~25 s · FIB ~90 s (**superados pelo blueprint oficial na Fase 13.2**).
- **5.2 PWA offline v2 (DT16):** `public/sw.js` v2 — cache `pte-master-hub-v2`, precache tolerante do shell,
  cache-first em `/assets/*` (já inclui os chunks de JSON, que o Vite empacota com hash), network-first em
  navegação com fallback `/`, limpeza de caches antigos no `activate`.
- **5.3 Evolução (DT17):** `/progress` — comparação 7 d × 7 d anteriores (`periodComparison`, delta colorido) e
  evolução semanal por tipo (`weeklyTypeAverages`, 6 semanas, top 6 tipos), mantendo o gráfico SVG manual de 14
  dias (`dailyAverages`, fonte única).

**Aceite:** `exam.ts` testado (planos, fila estratificada, tempos); fluxo `/exam` completo no preview; **offline
E2E verificado** — build de produção (nitro/cloudflare) servido com `wrangler dev` (porta 4173,
`wrangler.preview.jsonc` com `main=dist/server/index.mjs` e `assets=dist/client`), Playwright com SW ativo,
41 entradas em cache, `context.set_offline` + reload → Home e `/progress` renderizam do cache, 0 erros.

### Fase 6 — Meta, prontidão v1, plano do dia e relatório semanal (p3·F1–F4) ✅

**Objetivo:** fechar o ciclo de preparo — o aluno declara data e alvo, o app monta o plano e mede prontidão.
(Prontidão, pesos e comparação de alvo desta fase foram **reescritos na Fase 11** — ver DT19–DT20, §6.2.)

- **6.1 Meta (settings v4, DT18):** `Settings.examDate`/`targetScore`; `normalizeSettings` valida data (regex +
  `Date` válida) e alvo 10–90; `pte/goal.ts` (`daysUntil` — 0 = hoje, null/passado = null; `targetVerdict`;
  `countdownLabel`); `/settings` seção "Prova" (date picker nativo + alvo 50/58/65/79); Home: "Faltam N dias
  (alvo X)" / "A prova é hoje!", link para `/settings`; sem data nada muda.
- **6.2 Prontidão v1 (DT19, superada):** `pte/readiness.ts` (`readinessBySection`, `weakestSection`) com
  40% cobertura + 40% score 7 d + 20% estabilidade (nulls redistribuem peso); níveis Iniciando < 40 · Em
  progresso < 65 · Quase pronto < 85 · Pronto ≥ 85; `/progress#readiness` e chip por seção na Home.
- **6.3 Plano do dia (DT20, pesos superados):** `pte/studyPlan.ts` (`buildDailyPlan`, `planSummary`):
  distribui `dailyGoal` (mín. 4 para distribuir) — vencidas primeiro, rateadas por seção; novas priorizam pior
  média recente e maior estoque; nunca excede o estoque; card "Plano de hoje" + "Iniciar plano" =
  `startMixedSession(counts)` (DT22); `/session` → "Plano de hoje (N)" quando N ≥ 4. Pesos iniciais
  Speaking 30 / Listening 30 / Reading 20 / Writing 20 (`EXAM_WEIGHTS`, removido na Fase 11.2).
- **6.4 Relatório semanal (DT21):** `pte/report.ts` (`weeklyReport`, `renderReportMarkdown`,
  `renderReportJson`) reaproveitando readiness, `periodComparison`, `diagnoseAttempts`, `computeStreak`;
  `/progress` com preview colapsado, "Copiar relatório" (clipboard com fallback), baixar `.md`/`.json`;
  envelope de backup inalterado; estado vazio claro.

**Aceite:** `daysUntil`/`targetVerdict`/readiness/plano/relatório cobertos por testes puros (readiness: nulls,
pesos redistribuídos; plano: proporção, prioridade de vencidas, teto por estoque, meta < 4 e 0); backup v4
reimporta 1:1 e v3 ganha defaults; preview verificado (clipboard e downloads); 171/171 testes.

### Fase 7 — Expansão confiável do banco e novos tipos (p3·F0+F5 · p4·F0–F4) ✅

**Objetivo:** mais questões **úteis, reais e não duplicadas** e os tipos de exame que faltavam, sem tocar em
UI/motores fora do necessário. Governança de fonte e dedupe: §7.1–§7.2.

- **7.1 Gate de fonte e dedupe (p3·F0):** varredura online por fontes de Describe Image, Write Essay,
  Re-order e Reading FIB; veredito registrado (§7.4); só entra conteúdo com probabilidade muito alta de ser
  real, acessível sem login e com gabarito; dedupe contra os itens existentes antes de entrar em `src/data/pte/`.
- **7.2 Três tipos novos (p3·F5):** `write_essay` 38 (band9pte.com.au), `reorder_paragraphs` 50 (goarno.io),
  `read_fib` 50 (goarno.io, dropdown) — motores `WriteEssayPractice` (20:00), `ReorderPractice` (pares
  adjacentes), `FibDropdownPractice` (dropdowns inline); correção pura `pte/essay.ts`, `reorder.ts`,
  `fibDropdown.ts`; dispatcher, `EXAM_PLAN`/`EXAM_SECONDS` e widgets de simulado (Essay/Reorder/FibDropdown
  `ExamWidget`); Home e seções atualizadas. **Describe Image ficou bloqueado** (TypoGrammar exige e-mail; PDF
  da Gurully com ~10 imagens reais) — retomado e resolvido na Fase 12.2. Banco: 2.032 → 2.164 / 19 tipos.
- **7.3 Expansão escalonada (p4):** escopo único — só dados, manifesto e testes de integridade. Ordem pelos
  tipos mais escassos (prioridade e metas em §7.2). Fase P0: `lst_hcs`, `lst_hiw`, `lst_smw`, `lst_mcq_multi`,
  `lst_summary`, `lst_mcq`, `answer_short`, `mcq_multi` (goarno.io 7 páginas "Essential Questions" = 400
  candidatos; answer_short também de pteacademy.in; mcq_* de pteacademy.in). P1: `swt`, `retell_lecture`,
  `write_essay`, `mcq_single`, `read_fib`, `reorder_paragraphs`. P2: `lst_fib`, `summarize_spoken_text`,
  `read_aloud` (+ RS/WFD só se 100% novos — nenhum entrou). Cada fase: extração + normalização no schema
  existente → dedupe + relatório → integração nos JSONs + `build-manifest.mjs` + `bank-integrity`.
- **7.4 Fechamento do banco:** contagem final por tipo (§1.3) e por fonte (§7.3): **2.536** no manifesto / 19 tipos.
  Tipos sem fonte aprovada ficam com o volume atual, registrados como "sem fonte confiável" — nunca
  preenchidos com conteúdo duvidoso.

**Aceite:** veredito com URLs e contagens; relatório de dedupe (encontrados, inválidos, duplicatas,
integrados) por tipo e fonte; `bank-integrity` verde; prática de cada tipo tocado abre no preview sem erros
de console; motores novos com testes puros (+10); 181/181 testes.

### Fase 8 — Consistência de métricas e SEO dinâmico (p5·F1–F4) ✅

**Objetivo:** como a Fase 7 foi data-only, a UI ainda anunciava números antigos. Corrigir a instância **e a
categoria do bug** (número fixo em texto).

- **8.1 Sincronização:** `src/routes/index.tsx` — `head()`, parágrafo do hero, `<h1>` e rodapé derivam de
  `TOTAL_ALL_QUESTIONS`/`sectionCounts`/`TYPE_META` com `toLocaleString("pt-BR")`; lista de tipos "não
  cobertos" gerada de `TYPE_META` (Describe Image permaneceu como aviso manual até a Fase 12.2 — reconferir que
  o aviso saiu, DT31); `listening.tsx`, `__root.tsx` e `progress.tsx` idem. Auditoria das demais rotas públicas.
- **8.2 Prevenção:** `src/test/metrics-sync.test.ts` — (a) `TOTAL_ALL_QUESTIONS === allMeta.length`; (b) soma
  de `sectionCounts` = total; (c) varredura de `src/routes/*.tsx` falha com strings proibidas
  (`"2.164"`, `"1.165"`, `"16 tipos"`, `"16 task types"`, `"2.000 questões"`).
- **8.3 Offline E2E herdado (p3→p4→p5):** `/tmp/browser/test_offline.py` (fora do repo): carga normal (SW ativo)
  → `context.set_offline(True)` → Home, `/listening`, `/practice?q=<wfd>` renderizam do cache; produção via
  wrangler (SW só registra em PROD).
- **8.4 Fechamento:** gates globais; `AGENTS.md` com estado; monitor avisado.

**Aceite:** nenhuma string proibida em `src/routes/`; Home e Listening exibem os números do manifesto iguais
nos cards, cabeçalho e rodapé; navegação offline sem falhas e console limpo; 195/195 testes (181 + 14).

### Fase 9 — Pack "studie-plus": tradução, áudio de pronúncia, mobile-first e glossário (p6·F0–F5 · p7·F0–F5 · p8·F0–F4) ✅

**Contexto e regra-mãe.** O pack é um **recurso extra de estudo** (ativável/desativável e configurável em
Ajustes), explicitamente fora da replicação do PTE. Duas funções **independentes**: **tradução** (saber o que
o texto SIGNIFICA — texto pt-BR na tela) e **áudio de pronúncia** (saber COMO PRONUNCIAR o texto em inglês —
fala do original, nunca da tradução). R7–R9 aplicam-se integralmente.

- **9.1 Fundação (settings v5, DT35):** `Settings.plus { wordTranslation, phraseTranslation, plusAudio }` (off);
  `/settings` → seção "Studie Plus (extra de estudo)" com 3 toggles + texto de descorrelação + nota de que
  nada do pack entra em score/histórico; `plus/settings.ts` (`normalizePlusSettings`, `plusEnabled`,
  `PLUS_TOGGLES`); migração v4→v5, backup v5 aceita 2/3/4.
- **9.2 Tradução por palavra (DT36, DT38):** `plus/glossary.ts` (`lookupWord`: lowercase, sem pontuação externa,
  `’`→`'`; miss → `null` → UI mostra "Tradução não disponível", nunca tradução errada — **sem lematização**),
  `plus/words.ts` (`splitWords`); `PlusWordText.tsx` — palavra vira alvo de toque ≥ 44 px, **balão fixo
  portallado em `document.body`** (elimina HTML inválido em `<p>`; abre só por tap; fecha por toque fora/✕/Esc;
  nunca em scroll/hover); glossário por `import()` dinâmico só com o recurso ativo. Integrado em Read Aloud
  (só sem modo manual e sem resultado — o modo manual tem prioridade), Re-order, MCQ (mcq/mcq_multi/lst_mcq;
  `lst_smw` fica plain por ter lacuna), Read FIB (trechos entre lacunas, inline), Write Essay (enunciado), SWT
  (texto-fonte), SST/lst_summary (transcrição no resultado). **Motores só-áudio** (repeat, retell, answer_short,
  wfd, lst_fib, lst_hiw, lst_hcs) ficam de fora por design.
- **9.3 Tradução por frase (DT37):** `plus/sentences.ts` (`splitSentences`, `sentenceCount`), `plus/phrases.ts`
  (`lookupPhrase` → `{literal, contextual}` | null); `PlusPhraseCard.tsx` — card colapsado "Tradução da
  frase ▾" que expande **só por tap**; estado vazio honesto. Integrado em Read Aloud, MCQ, Re-order, Read FIB
  e WFD (no resultado). Cobertura: 1.372 questões / 3.098 frases (fase inicial) + 835 frases SWT/Write Essay
  = **3.933 frases, 100%**, 0 sem casamento. Dados gerados por subagentes (DT40). **Correção de dados:** lotes
  out-025↔026 trocados e ciclo 031→033→037→034→035→032 reescritos; out-029 regerado; merge refeito com
  alinhamento verificado (3.098/3.098).
- **9.4 Áudio de pronúncia (DT39, DT41) — inclui a correção do antigo projeto 7.** *Erro original (p6·F3):*
  o card de frase ganhara um botão "ouvir tradução" que falava **pt-BR** (`ptVoice`, `lang="pt-BR"`) — aluno
  brasileiro não precisa ouvir português; o áudio existe para ensinar o inglês. *Estado final:* (a)
  `plus/plusAudio.ts` — matriz campo × tipo, **nunca duplica áudio nativo**, `transcript` sempre `false`
  (gabarito falado não vaza), campos fora da matriz `false`; (b) `PlusSpeakButton.tsx` — Web Speech,
  `rate = modelRate`, cancela ao trocar de questão/unmount, variante compacta 44×44; integrado em 11 motores
  (WriteEssay, Swt, Mcq, Reorder, FibDropdown, Wfd — explicação, `answer` excluído —, Sst, RepeatSentence,
  AnswerShort, Hiw, Fib); (c) **zero fala pt-BR**: sem `ptVoice`, sem "ouvir tradução", sem `langOverride`;
  (d) **independência total dos toggles** (2×2×2 = 8 estados válidos): `PlusPhraseCard` com props `enabled`
  (tradução) e `audioEnabled`; 🔊 por frase **original em inglês**, na própria linha, fora da expansão; modo
  só-áudio = lista de frases não expansível; `PlusWordText` com `audioEnabled` — palavra tappable quando
  `wordTranslation || plusAudio`, balão com tradução só se `wordTranslation`, 🔊 da palavra em inglês ao lado
  do termo; (e) sem voz compatível o botão não renderiza (`supported`).
- **9.5 Mobile-first (p6·F4):** auditoria Playwright 390×844 nos 16 motores + home — zero overflow, zero
  pageerrors; alvos ≥ 44 px (`button.tsx`, TopNav, PracticeShell, Hiw/Reorder/Fib/FibDropdown/ReadAloud);
  rodapé do Read Aloud com wrap real ("Iniciar preparação" não extrapola); `PlusWordText` portallado.
- **9.6 Glossário por palavra via subagentes (p8):** pendência herdada do p6·F1 (AI Gateway sem saldo, HTTP 402,
  glossário `{}`). **Método revisado (DT40):** subagentes read-only em lotes, **zero AI Gateway**. Contrato e
  invariantes em §7.7; pipeline em §7.8. Resultado: **5.776 / 6.000 candidatas (96,3%)**, 30/30 lotes, I1–I6 =
  0 descartes; teste `plus-glossary.test.ts` (10 testes); verificação mobile do balão ("complex → complexo").
- **9.7 Fechamento:** textos de Ajustes e `AGENTS.md` com a regra correta; registros históricos do p6 anotados
  (não apagados); gates e monitor avisado em cada subprojeto.

**Aceite (verificado):** pack off ⇒ nenhuma mudança nas rotas de prática; testes de integração (17) cobrem os
4 cenários de toggle, a fala observada via stub de `speechSynthesis` (`text` = inglês original, `lang` nunca pt),
varredura de fonte sem `"pt-BR"`/`"ouvir tradução"` e integridade dos 8 tipos de frases; 258/258 testes;
Playwright mobile sem overflow, console limpo; traduções geradas e matriz `plusAudio.ts` intactas na correção.

### Premissa e achados das Fases 10–14 (antigo projeto 9)

**Premissa do conjunto.** Estas fases corrigem o que o app **mostrava errado ao aluno**, o que **podia apagar
seu progresso** e o que **não treinava a prova real**. Escopo só do app (dados, lógica, UX). Princípios, nesta
ordem: (1) não perder nem distorcer o progresso; (2) medir com honestidade; (3) alinhar à prova real;
(4) entrega rápida e offline de verdade. Os achados confirmados no código (A1–D3) estão na tabela abaixo.

**Achados (códigos mantidos; todos ✅ confirmados no código antes de agir):**

| Cód. | Achado | Tratado em |
|---|---|---|
| A1 | `MAX_ATTEMPTS=5000` apagava o SRS: `box`/`dueAt`/dominada derivavam do log; ao cortar as tentativas antigas a questão voltava a "nunca tentada" | 10.2 |
| A2 | Estado ilegível ou de versão desconhecida virava vazio e era **sobrescrito** na próxima gravação (`load()` catch → vazio; `migrateLegacyState` versão não listada → vazio) | 10.2 |
| A3 | `importBackup` substituía tudo, sem prévia, sem confirmação, sem mesclar, sem desfazer | 10.2 |
| A4 | Persistência gravava o estado inteiro a cada mutação, sem aviso **preventivo** de quota, sem `navigator.storage.persist()`, sem lembrete de backup (Safari/iOS pode limpar armazenamento de sites pouco usados — conhecimento de plataforma) | 10.2 |
| A5 | `computeStreak`/`dailyAverages` andavam em passos de 24 h (`t -= DAY`); com horário de verão o dia podia ser pulado/duplicado | 10.2 |
| B1 | Escalas misturadas: Ajustes pede meta 10–90, Home comparava com média em % ("média 72% / alvo 65") | 11.2 |
| B2 | Prontidão media o **tamanho do banco** (40% = cobertura; Listening 1.168 itens, 934 só de WFD → nunca sobe); estabilidade `1 − σ/100` quase não discrimina | 11.2 |
| B3 | Pesos 30/30/20/20 sem fonte; modelo "1 tarefa → 1 seção" (na prova uma tarefa pontua mais de uma habilidade) | 11.2 |
| B4 | Score de fala ignorava `extras` e `repetitions`; pronúncia não medida (a API não expõe); reconhecedores "corrigem" para palavras prováveis → score inflado | 11.1 |
| B5 | SRS inflável: toda tentativa ≥ 80 subia a caixa, mesmo fora do prazo; qualquer falha zerava; limiar 80 para todos os tipos (probe: 5 acertos em 5 min → caixa 5, revisão em 30 dias) | 10.3 |
| B6 | Ajustes dizia "Nenhuma conta, nenhum servidor", mas no Chrome o reconhecimento de voz costuma rodar em serviço do navegador | 11.1 |
| C1 | Prova tem 22 task types desde 07/08/2025; app cobria 19 (faltavam SGD, RTS, Describe Image); `manifest.webmanifest` dizia "16 task types" | 12.2 · 13.1 |
| C2 | Proveniência invisível: 647 `ai-generated-prediction` + 200 `pteracai-seed`; UI nunca mostrava `.source` | 12.1 |
| C3 | 28 passagens de SWT duplicadas (`pte-swt-*` × `goarno-swt-NN`; similaridade 0,88); progresso por `qid` dividia estatística e SRS | 12.1 |
| C4 | Bancos rasos (3–6 itens): `lst_hcs`, `lst_hiw`, `lst_smw`, `lst_mcq_multi`, `lst_summary`, `lst_mcq` → decoreba | 12.1 |
| C5 | "Simulado" era um mini-simulado (Speaking = 8 questões ~7 min vs. prova ~2 h/22 tasks; tempos sem fonte) | 13.2 |
| C6 | TTS do navegador sem informar qual voz/sotaque foi usado (fallback por prefixo) | 14.1 |
| D1 | Offline só do que já foi visitado; `sw.js` pré-cacheava 5 rotas; sem aviso de nova versão (chunks: `wfd` 522 KB, `summarize_spoken_text` 435 KB, `glossary` 373 KB) | 13.1 |
| D2 | Payload inflado: WFD com `text = transcript = audio_text = answer` em 934/934 (≈ 51% dos bytes repetidos) | 12.1 |
| D3 | Acessibilidade não auditada (~49 `aria-`/`role=` fora de `components/ui`, sem verificação automática) | 14.1 |

### Fase 10 — Integridade do progresso (storage v6) e SRS correto (p9·F0–F2) ✅

**Objetivo:** o progresso do aluno nunca mais é perdido, truncado ou sobrescrito em silêncio, e a caixa do SRS
representa memória de longo prazo — não volume de prática.

- **10.1 Fontes oficiais (p9·F0):** confirmação na Pearson (Score Guide, páginas de formato, press release de
  10/07/2025) — registro e matriz em §7.5 e `src/lib/pte/skillMatrix.ts`. Fontes secundárias só corroboram;
  divergência → prevalece a oficial; o não confirmável vira decisão conservadora rotulada.
- **10.2 Storage v6 (A1–A5; DT7–DT10, DT12):** schema em §3.1. `recordAttempt()` atualiza o snapshot e o
  agregado diário **incrementalmente**, anexa ao log e **só então** trunca. Migração v5→v6 semeia antes de
  cortar (§3.3). **Escrita resiliente:** quarentena de payload ilegível, versão futura somente-leitura, escrita
  **coalescida** (N mutações no mesmo tick = 1 `setItem`, flush em `visibilitychange`/`beforeunload`), estimativa
  de bytes com banner "armazenamento quase cheio — baixe um backup" a partir de ~80% do teto prático (~4 MB),
  `navigator.storage.persist()` no 1º gesto significativo, lembrete de backup em `/progress` quando o último
  tem > 14 dias. **Datas:** `streak`/`dailyAverages` iteram **chaves de dia local** via `new Date(y, m, d)`
  (helpers puros em `dates.ts`). **Import (`settings.tsx`)**: 2 passos — prévia (contagens, período, favoritos,
  sessões, comparação com o atual) → **Substituir | Mesclar** (união de snapshots com re-replay dos logs;
  favoritos/notas unem; settings = os do arquivo, com prévia) → cópia automática (`pte-master-hub-undo`) +
  **Desfazer**.
- **10.3 SRS v2 (B5; DT11):** `applyAttemptToSnapshot` é o **único ponto** de passagem de caixa.
  *Promoção:* `score ≥ PASS_SCORE(tipo)` **e** questão vencida (`ts ≥ dueAt` anterior) → `box + 1` (teto 5);
  prática antecipada registra e **não promove**. *Lapso:* `score < PASS_SCORE(tipo)` → `box = max(box − 1, 0)`,
  `dueAt` volta para hoje + intervalo da nova caixa. `PASS_SCORE_BY_TYPE`: ditados (WFD, lst_fib) 85 ·
  Read Aloud/Repeat Sentence 80 · retenção de conteúdo (SST, retell, resumos, ensaio) 70 · MCQ/FIB/reordenar
  75. `dueAt` = `ts` da tentativa qualificante + `SRS_INTERVAL_DAYS[box]`. *Dominada:* `isMasteredStat` com
  gate de vencidas (critérios por tipo; a tentativa forte qualificada foi feita vencida **ou** as duas últimas
  fortes — evita "dominada por maratona de 5 min"). Fila do dia, plano e "revisões vencidas" leem `dueAt` do
  snapshot sem mudar assinatura pública. Migração recalcula caixas do histórico em replay (`srsRules: 2`); log
  truncado abaixo do snapshot mantém a caixa antiga — **nada é inventado**.

**Aceite (probes viram testes verdes):** A1 (5.100 tentativas, caixa 3 na 1ª questão → snapshot intacto, log
≤ 2.000) · A2 (corrupto → quarentena + estado limpo; `version: 7` → somente-leitura, preservado byte a byte
após mutação de settings) · round-trip v5→v6→export→import igual campo a campo · import-mescla (v5 antigo sobre
v6 novo, nada do atual se perde) · streak com offset mudando (A5) · escrita coalescida · B5 (prática seguida →
caixa 1, não 5; lapso derruba 1; atrasada promove; antecipada não promove; 1 caso por faixa de `PASS_SCORE`;
gate anti-maratona; migração idempotente). Gates: 266/266 (F1) → 285/285 (F2); Playwright export → prévia →
mesclar → undo em `/settings`, console limpo.

### Fase 11 — Medição honesta, escala 10–90 e prontidão v2 (p9·F3–F4) ✅

- **11.1 Fala e escrita (B4, B6; DT32–DT34):**
  - *Score de fala:* `score = round(max(0, credit − 0,25·extras − 0,1·repetições) / total · 100)`, piso 0,
    `gradable` inalterado, constantes nomeadas e testadas (`scoring.ts → gradePassage`).
  - *Subscores no `ScorePanel`:* **Conteúdo** (score atual) · **Fluência** (proxy rotulado: pausas longas +
    veredito de ritmo; `fluencyProxy`) · **Pronúncia "não avaliada"** (nenhum proxy disfarçado — a Web Speech
    API não expõe pronúncia nem `confidence` confiável). Aviso fixo: o reconhecedor pode "corrigir" a fala,
    então o score tende a ficar otimista. Aplicado também ao `ReadAloudPractice` (layout próprio).
  - *Textos honestos:* Ajustes → "Tudo é salvo neste navegador, sem conta. O reconhecimento de voz pode usar um
    serviço do próprio navegador (ex.: Chrome)." Na 1ª vez que um recurso abre o reconhecimento, uma frase
    explica o uso do áudio (flag em settings, não por questão).
  - *Detector local de template (`pte/templateDetector.ts`, sem IA):* taxa de 4-gramas repetidos, inícios de
    sentença repetidos, sobreposição com lista curta e pública de frases-modelo genéricas ("In today's
    world…", "It is undeniable that…"), com guarda anti-falso-positivo. Saída `{ likelyTemplate, reasons }` →
    aviso rotulado em Essay/SWT/SST ("a Pearson pode enviar para revisão humana"); **nunca muda o score**.
- **11.2 Escala, alvo e prontidão (B1–B3; DT20, DT23, DT19):**
  - *Matriz e pesos:* `skillMatrix.ts` (22 linhas, matriz oficial tarefa×habilidade, §7.5) deriva
    `SKILL_WEIGHTS` por contagem de contribuições — Speaking **20** · Listening **40** · Reading **27** · Writing
    **13** (6/12/8/4 de 30 contribuições, §7.5) —, rotulados aproximação (pesos numéricos reais são proprietários). `EXAM_WEIGHTS` removido; `studyPlan.ts` e o simulado usam a matriz.
  - *Estimativa:* `estimate.ts → estimateSkill(pct)` mapeia % interna → 10–90 (10 = 0%, 90 = 100%, linear,
    arredondado), sempre rotulada **"estimativa de treino"** e com nível de confiança. `goal.ts → targetVerdict`
    compara estimativa × alvo **na mesma escala**; Home e `/progress` mostram "Estimativa de treino: 62 ·
    Alvo: 65". Nenhuma frase mistura % com 10–90 (teste varre strings proibidas).
  - *Prontidão v2 (`readiness.ts` reescrito):* núcleo = desempenho recente ponderado (últimas N tentativas com
    score por seção, pesos exponenciais por recência) + tendência (recente × anterior) + **confiança** (< 10
    tentativas com score → "confiança baixa" explícito). Cobertura e variedade (tipos distintos
    tentados/total) viram secundárias e só **reduzem** o índice quando absurdamente baixas (nunca mais 40%).
    Itens `ia` pesam 0,7 (DT26). Níveis Iniciando…Pronto mantidos sobre o índice novo. Relatório semanal usa os
    mesmos componentes.

**Aceite:** casos anotados de scoring (extras/repetições derrubam o score de forma explicável); detector com
positiva real, negativa real e falsos-positivos comuns; subscores visíveis em 390×844; nenhum texto diz "nenhum
servidor" sobre reconhecimento; `estimate()` monotônica; pesos = soma da matriz; n < 10 → "confiança baixa";
IA/seed pesa 0,7; nenhuma tela mistura % com 10–90. Gates: 299/299 (F3) → 324/324 (F4).

### Fase 12 — Banco rastreável (proveniência, duplicatas, payload) e tipos 22/22 (p9·F5 · F7) ✅

- **12.1 Banco (C2–C4, D2; DT26–DT28):**
  - *Proveniência:* `QuestionMeta.source` classificado em 3 selos (§7.3), chip discreto na prática e nas
    listas. Ajustes → **"Só fontes verificadas"**, **desligado por padrão** (revisão de execução: WFD 934 e
    Repeat Sentence 586 não têm nenhum item verificado; ligado por padrão, as duas tarefas centrais sairiam
    das filas). Quando ativo afeta fila SRS, plano e simulado; **as listas sempre mostram tudo, com selo**.
  - *Dedupe:* `.opencode/proj9-work/dedupe.mjs` (prefixo de 200 caracteres normalizados + razão de trigramas ≥
    0,85, todos os tipos textuais) → `canonical-map.json` (`qid → canonicalId`, o id mais antigo vence).
    `canonicalId` entra no manifesto/catálogo (`canonicalIdOf`); snapshot indexa por `canonicalId`;
    `mergeSnapshots` funde (máximos/últimos, contagens somadas, `box = max`) **sem perda**; migração v6 funde
    `questions`/`bookmarks`/`notes`. Duplicatas ficam no banco com selo "repetida" e **fora das filas**
    (nenhum qid é apagado sem registro). 28 pares de SWT tratados.
  - *Payload:* `wfd.json` com campo único (`answer`) e `dictation.ts`/catálogo derivando o texto; bundles
    medidos antes/depois — **3.583 KB → 1.612 KB (−55%)**, chunk `wfd` 166–174 KB. Selo **"banco pequeno"**
    (≤ 6 itens) nas listas e no plano ("treine, mas não confie na cobertura") — sem criar conteúdo.
- **12.2 Tipos 22/22 (C1; DT29, DT30):** por tipo, integrar com fonte verificável ou "em breve" com motivo.
  Integrados: **SGD 13** (12 EdKnot + 1 Pearson Test Tips; áudio multi-voz via TTS com pitch/rate por falante),
  **RTS 63** (12 EdKnot + 50 GoArno + 1 Pearson; 10 s de preparo + 40 s de fala — a fonte oficial omite o
  preparo e o EdKnot diz 20 s; ambiguidade documentada), **Describe Image 6** (gráficos Our World in Data,
  CC BY 4.0, atribuição renderizada no motor; 25 s de preparo + 40 s de fala; `alt` acessível + citação).
  `pearsonpte.com` e `ourworldindata.org` entram em `VERIFIED_SOURCES`. Motores `SgdPractice`,
  `RespondSituationPractice`, `DescribeImagePractice` (reaproveitam Retell Lecture/SST); `SpeechWidget` do
  exame estendido aos 3 tipos; catálogo e contagens derivam do manifesto (zero número fixo).

**Aceite:** zero quase-duplicatas no relatório do script; progresso dos 28 pares de SWT fundido sem perda;
selos por fonte (1 caso por selo); filtro "só verificadas" afeta fila/plano/simulado; contrato do `wfd.json`
e derivação no catálogo testados; `bank-integrity` valida forma/conteúdo dos 3 bancos novos; catálogo 22/22.
Gates: 338/338 (F5) → 341/341 (F7); Playwright 390×844 com selos e toggle de fontes, console limpo.

### Fase 13 — Offline real e simulado fiel (p9·F6 · F8) ✅ (um gate pendente → 14.2)

- **13.1 Offline real (D1; DT16):** `sw.js` registrado só em PROD, pré-cache das 10 rotas; **"Baixar para
  offline"** em Ajustes (importa os 19 bancos → chunks `/assets/` cacheados, visita rotas) com contagem de
  progresso; cache versionado por `VERSION` em `sw.js` (migra `/assets/` entre versões); aviso **"Nova versão
  — Recarregar"** via `SKIP_WAITING`; descrição do `manifest.webmanifest` derivada do banco em
  `scripts/build-manifest.mjs` (acaba com o "16 task types" defasado). **Decisão de execução:** download dirigido
  **pelo cliente** em vez de lista injetada no build (o build Nitro/Cloudflare não expõe a pasta do cliente de
  forma estável); `scripts/gen-precache.mjs` previsto no PRD de origem não foi o caminho adotado.
- **13.2 Simulado fiel (C5; DT15):** `ExamId` ganhou `"mini"`; `full` agora é o **BLUEPRINT oficial** em
  `exam.ts` (59 questões, ~2 h 01 min, ordem fixa Speaking & Writing → Reading → Listening, contagens = meio dos
  intervalos oficiais); o antigo "completo" virou **"Mini-simulado"** (rótulo obrigatório na escolha). Omissão
  explícita: FIB drag&drop de Reading (`BLUEPRINT_OMISSIONS`, exibida na tela de escolha). `ExamSummary.bySkill`
  traz a média por habilidade (matriz `TASK_SKILLS`) comparada à prática livre e a simulados anteriores; fila
  respeita o estoque do banco. Tempos não confirmáveis oficialmente (Repeat Sentence, Answer Short) rotulados
  "aproximados".

**Aceite:** ordem/tempos conferidos item a item com a fonte oficial; rótulos mini × completo; resultado por
habilidade contra o alvo e o histórico; Playwright E2E do simulado de **Reading** (`/exam`): 7/7 questões
interagidas + auto-submit por cronômetro (`page.clock.run_for`), resumo com média por seção/habilidade/tipo e
"Pontos Fracos", console limpo. 343/343 testes. **Não cumprido ainda (→ 14.2):** Playwright offline em build
publicado — um tipo **nunca visitado** funcionando após "Baixar para offline" e a atualização do SW (SW desligado no dev).

### Fase 14 — Acessibilidade, mobile e fechamento (p9·F9 + pendência) 🔶 em aberto: 14.2

- **14.1 Acessibilidade e voz (D3, C6; DT44) ✅:** axe 4.10.2 (runner de teste, sem dependência de runtime) —
  **0 violações** nas 10 rotas principais. Contraste: `--muted-foreground` do tema claro escurecido para
  `oklch(0.526)` (4,34 → 4,89:1 sobre `bg-secondary`); link ativo do TopNav legível (antes `!text-primary-foreground`
  sobre `bg-secondary` = 1,04:1); `(tecla N)` sem `/70`; chips de prática legíveis. Landmarks/headings: barras de
  ação viraram `<footer>`, título de questão virou `<h1>` (PracticeShell + ReadAloud), cards de seção h3 → h2.
  Teclado: Esc fecha notas (update funcional no listener), `aria-pressed` no ★, Tab com ring visível, Espaço marca
  MCQ, Enter inicia simulado, setas/B/N/`?` OK. TTS: `TtsVoiceNote` no shell e no Read Aloud mostra a voz do
  sistema usada (verificada pós-playback). Revisão 390×844: 9 rotas sem overflow horizontal, console limpo.
- **14.2 Fechamento e verificação offline de produção (aberto):**
  1. Build de produção servido com `wrangler dev` (porta 4173, `wrangler.preview.jsonc`); Playwright com SW ativo.
  2. **Tipo nunca visitado** (ex.: um dos 3 tipos de Fase 12.2) → "Baixar para offline" → `context.set_offline`
     → `/practice?q=<id>` renderiza do cache.
  3. **Motores novos offline** (herdado da Fase 7: `write_essay`, `reorder_paragraphs`, `read_fib`; + SGD/RTS/DI):
     cada prática abre offline sem erro de console.
  4. **Atualização do SW:** subir `VERSION` → aviso "Nova versão — Recarregar" aparece e `SKIP_WAITING` aplica.
  5. Gates globais; `AGENTS.md` (estado atual): conferir se já registra as Fases 10–14 (o roadmap de origem não
     registrava essa atualização) e atualizar se faltar; marcar o roadmap; avisar o monitor com **"remix finalizado (+N da etapa do roadmap que foi efetuada)"**; informar o
     usuário (sem `ask`).
  6. Concluído tudo: planejar as próximas melhorias em **novo** `project2` (Plan.md), ativar o opencode, informar o monitor.

**Aceite:** itens 1–4 com screenshots em `/tmp/browser/` (nunca no worktree) e console limpo; 343/343 testes
(ou mais) · tsgo · build · lint 0 erros mantidos.

---

## 6. Decisões técnicas (o COMO, numeração única)

### 6.1 Decisões vigentes

| # | Decisão (estado final) | Por quê | Origem |
|---|---|---|---|
| DT1 | `Question` discriminada por `task_type`; listas usam só `manifest.ts`; conteúdo completo sob demanda por `catalog.ts` (import dinâmico por tipo) | Home leve com milhares de itens; cada motor recebe o subtipo que entende | p1 · p2·§1.6 |
| DT2 | Backup JSON versionado como **envelope único**; campos novos entram com migração pura e aditiva; import aceita versões anteriores | Zero perda; migração testável | p1–p9 |
| DT3 | Sessão = `draft` persistido a cada resposta + histórico de `SessionRecord` (teto 100) | Retomada simples; localStorage não cresce sem limite | p2·D3 |
| DT4 | `practice.tsx` é despachante por `taskType`; correção em módulos puros de `src/lib/pte/*`; motores novos reaproveitam os existentes (Retell Lecture/SST) quando o formato permite | Menos superfície de bug; testes antes da UI | p1 · p9·F7 |
| DT5 | SRS Leitner de 6 caixas (0–5), intervalos `[0,1,3,7,14,30]` dias; "Fila do dia" = `buildQueue` modo `"due"` (só vencidas, pior score primeiro) | Agenda visível em qualquer página | p2·F2 |
| DT6 | Reconhecimento de fala com `en-AU` padrão (`en-US`/`en-GB` opcionais); medidor de mic; replay via `MediaRecorder` | Sotaque do exame; feedback imediato | p1 |
| DT7 | **Snapshot SRS por questão** em localStorage, atualizado incrementalmente; IndexedDB só se a medição mostrar payload > 3,5 MB após dedupe/payload (medir, não apostar) | A1: SRS não pode depender de um log truncável; payload por questão ~120 B × 2.536 ≈ 300 KB | p9·D1 |
| DT8 | Log de tentativas mantido só para auditoria; teto 2.000 aplicado **depois** de semear o snapshot; SRS/estatística nunca derivam do log | Truncar nunca tem efeito colateral | p9·D2 |
| DT9 | Estado ilegível → **quarentena** (cópia do payload bruto, estado limpo); versão futura (v7+) → somente-leitura com aviso; nunca sobrescrever | A2: perda silenciosa é o pior erro | p9·D8 |
| DT10 | Import = prévia obrigatória + **substituir \| mesclar** + cópia automática (`pte-master-hub-undo`) + **desfazer** | A3: backup antigo não pode destruir progresso novo | p9·D9 |
| DT11 | SRS v2: caixa só avança se a questão estava **vencida**; lapso rebaixa **1** caixa (piso 0); `PASS_SCORE` **por tipo** (ditados 85 · RA/RS 80 · retenção 70 · MCQ/FIB/reordenar 75); "dominada" com gate de vencidas | B5: caixa = memória de longo prazo, não volume | p9·D11–D12 |
| DT12 | Datas por **chave de dia local** (`new Date(y,m,d)`), nunca passo fixo de 24 h | A5: horário de verão | p9 |
| DT13 | Erros em `Attempt.tags?: string[]` (não em `metrics`, que é numérica por contrato) | Palavras erradas precisam de campo textual | p2·D4 |
| DT14 | Diagnóstico = funções puras agregando attempts existentes; UI só renderiza | Testável sem UI | p2·D5 |
| DT15 | Exame em rota própria `/exam` reutilizando os motores via `ExamRunner`; **blueprint oficial** (~2 h, 59 q) × **mini-simulado** rotulados; omissões declaradas | `/session` é treino; exame tem semântica e tempos próprios | p2·D6 · p9·D15 |
| DT16 | PWA: `sw.js` versionado, pré-cache de rotas, cache-first em `/assets/*`, network-first em navegação; **download offline dirigido pelo cliente**; aviso de nova versão via `SKIP_WAITING`; sem service-worker generator (zero deps) | D1: offline de verdade; build Nitro/Cloudflare não expõe a pasta do cliente de forma estável | p2·D7 · p9·D14 |
| DT17 | Gráficos continuam SVG manual | Padrão do projeto; zero lib de chart | p2·D8 |
| DT18 | `Settings.examDate` + `targetScore` aditivos; alvo padrão 65; opções 50/58/65/79 | Alvo 65 = pontuação corrente mais pedida | p3·D1 |
| DT19 | **Prontidão v2:** desempenho recente ponderado + tendência + confiança (n mínimo 10); cobertura/variedade secundárias e só reduzem quando absurdamente baixas; itens `ia` pesam 0,7 | B2: prontidão mede habilidade, não o tamanho do banco | p9 (supera p3·D3) |
| DT20 | **Pesos por habilidade derivados da matriz oficial** tarefa×habilidade (`skillMatrix.ts`): Speaking 20 · Listening 40 · Reading 27 · Writing 13, rotulados aproximação | B3: uma tarefa pontua mais de uma habilidade | p9·D10 (supera p3·D2) |
| DT21 | Relatório semanal gerado sob demanda; envelope de backup inalterado | Derivável a qualquer momento | p3·D4 · D7 |
| DT22 | "Iniciar plano" = `startMixedSession(counts)` | Reusa sessão/histórico/draft | p3·D5 |
| DT23 | Estimativa de treino 10–90 por habilidade (linear, arredondada), rotulada e com nível de confiança; alvo comparado **na mesma escala**; nunca "previsão oficial" | B1: nenhuma tela mistura % com 10–90 | p9·D3 |
| DT24 | **Gate de fonte** (§7.1) + **gate de dedupe** (§7.2) antes de qualquer dado entrar | Só conteúdo de probabilidade muito alta de ser real; zero duplicatas | p3·D6 · p4·D3 |
| DT25 | Dados novos: sem IA, sem paráfrase "de memória", sem conteúdo de baixa probabilidade; tipo sem fonte aprovada fica "em breve" com motivo | Qualidade acima de volume | p4·D4 |
| DT26 | Itens `ia`/seed **não são apagados**: selo de proveniência, filtro "só verificadas" (**padrão desligado**), peso 0,7 na prontidão | WFD e RS não têm item verificado; ligado por padrão, tiraria as duas tarefas centrais das filas | p9·D4 (revisado na execução) |
| DT27 | Quase-duplicatas fundidas por **`canonicalId`** (id mais antigo vence); ambos os qids ficam no banco; progresso/SRS/estatística indexam pelo canônico; migração funde sem perda | C3: estatística dividida entre cópias | p9·D7 |
| DT28 | `wfd.json` slim (campo único `answer`; texto derivado no catálogo); outros tipos só se medido ganho > 15% | D2: ≈ 51% dos bytes eram repetição | p9 |
| DT29 | Describe Image só com imagens acessíveis (alt) e licença clara por imagem; senão "em breve" com motivo | Honestidade do 22/22 | p9·D6 |
| DT30 | SGD com áudio multi-voz via TTS (turnos com vozes distintas); RTS 10 s + 40 s; DI 25 s + 40 s | Formato oficial; timing de RTS com ambiguidade documentada | p9·F7 |
| DT31 | Todo número de banco/tipos na UI e no SEO deriva de `TOTAL_ALL_QUESTIONS`, `sectionCounts`, `TYPE_META` (`toLocaleString("pt-BR")`); `metrics-sync.test.ts` falha com strings proibidas | Fecha a **categoria** do bug, não só a instância | p5·D1–D5 |
| DT32 | Score de fala = `credit − 0,25·extras − 0,1·repetições` (piso 0) sobre o total; constantes nomeadas | B4: o que sobra conta | p9 |
| DT33 | Pronúncia = **"não avaliada"** (nenhum proxy disfarçado); Fluência = proxy rotulado | A Web Speech API não expõe pronúncia | p9·D5 |
| DT34 | Detector de template **local** (n-gramas, inícios repetidos, frases-modelo); aviso rotulado, **nunca** muda o score; textos honestos sobre servidor de voz e microfone | Revisão humana de "template" é fato oficial; sem IA (R12) | p9·D13 |
| DT35 | Dados do pack em `src/data/plus/`, `import()` dinâmico só quando ativo; settings v5 aditivo, tudo `false`; rótulo de descorrelação na UI | R7–R9 | p6·D1 D2 D8 |
| DT36 | Glossário: lookup **exato** (lowercase, sem pontuação), miss → estado vazio honesto; **sem lematização/pluralização**; cada forma do corpus é uma chave | Tradução errada é pior que ausente | p6·D4 · p8·D8 |
| DT37 | Frases alinhadas por **índice** de `splitSentences` (mesma função da UI), nunca fuzzy | Alinhamento determinístico e testável | p6·D5 |
| DT38 | Palavra e frase só por **tap/click explícito**; balão portallado fecha por toque fora/✕/Esc; nada em hover/scroll | Regra mobile do planejamento | p6·D6 |
| DT39 | **Áudio do pack = pronúncia do inglês original** (`settings.accent` + `modelRate`), zero fala pt-BR, independente dos toggles de tradução; matriz `plusAudio.ts` nunca duplica áudio nativo e nunca vaza `transcript` | O erro do p6 acoplou duas funções; pt-BR não ensina pronúncia de inglês | p6·D7 · p7·D1–D3 D5–D6 |
| DT40 | Dados do pack gerados **dev-time por subagentes read-only em lotes** (o agente principal grava e valida); **zero AI Gateway** (`generate-*.mjs` ficam dormentes) | Gateway sem saldo (HTTP 402); método já provado em 3.933 frases | p6 (revisado) · p8·D1–D7 |
| DT41 | `PlusPhraseCard`/`PlusWordText` com props separadas `enabled` (tradução) e `audioEnabled`; motores só repassam props (mudança mecânica) | Independência real dos toggles sem tocar lógica dos motores | p7·D4 |
| DT42 | Glossário: contrato fixo, invariantes I1–I8 checadas por script **e** por teste (`plus-glossary.test.ts`) | O dado vira parte da suite | p8·D9 |
| DT43 | Verificação offline sempre em **build de produção** (SW só registra em PROD) servido por `wrangler dev`; scripts e screenshots fora do worktree (`/tmp/browser/`) | Dev não tem SW; worktree limpo | p2·F6 · p5·D6 |
| DT44 | Acessibilidade verificada com axe (runner de teste, sem dependência de runtime); voz TTS realmente usada exibida (`TtsVoiceNote`) | D3, C6 | p9·F9 |

### 6.2 Decisões superadas (mantidas para rastreio — não reaplicar)

| Decisão antiga | Superada por | Motivo |
|---|---|---|
| "SRS derivado de `attempts`, sem campo próprio" (p2·D2) | DT7–DT8 | Cortar o log apagava o SRS (A1) |
| `MAX_ATTEMPTS = 5.000` como teto de tudo | DT8 (2.000, só log) | Idem |
| Prontidão = 40% cobertura + 40% score 7 d + 20% estabilidade (p3·D3) | DT19 | Media o tamanho do banco (B2) |
| Pesos do plano 30/30/20/20 sem fonte (p3·D2, `EXAM_WEIGHTS`) | DT20 | Matriz oficial (B3) |
| `targetVerdict` sobre média em % vs. alvo 10–90 | DT23 | Escalas misturadas (B1) |
| Limiar único `PASS_SCORE = 80`; falha zera a caixa | DT11 | SRS inflável (B5) |
| Import substitui tudo sem prévia | DT10 | A3 |
| Estado ilegível vira vazio e é sobrescrito | DT9 | A2 |
| "Voz pt-BR para a tradução" no card de frase (p6·F3) | DT39 | Erro lógico: áudio ≠ áudio da tradução |
| Traduções geradas via AI Gateway (p6·D3) | DT40 | Sem saldo; subagentes provaram o método |
| Simulado "completo" de 8 questões por seção (p2·F5) | DT15 (blueprint + mini) | C5 |
| Describe Image "em breve" por falta de fonte (p3→p5) | DT29 + Fase 12.2 | Fonte CC BY 4.0 encontrada |
| Fonte de contagem: números fixos nos textos (p1–p4) | DT31 | Divergência 2.164 × 2.536 |

---

## 7. Governança de dados e fontes

### 7.1 Gate de fonte (DT24 — endurecido no p4)

Uma fonte só é **APROVADA** se cumprir **todos**: HTTP 200, acessível sem login/e-mail/pagamento · resposta/
gabarito (e transcript para listening) explícitos · formato fiel ao exame (enunciado, opções, contagem de
palavras plausível) · volume ≥ 10 itens úteis por tipo (fontes menores só se agregarem tipo escasso) · sinal de
"real/prediction de prova" (sites de prep reconhecidos: GoArno, Band9PTE, Gurully, PTE Tutorials, E2Language,
blogs abertos de APEUni/PTE Magic etc.). **REPROVADA** se exigir cadastro, tiver conteúdo gerado por IA/aleatório,
não tiver gabarito, for listening sem transcript ou tiver item quebrado/incompleto. **Toda reprovação é
registrada.** Itens de listening só entram se o motor atual os usa (transcript para TTS). Para provar
"probabilidade muito alta de ser real", o agente planeja métodos adaptativos de verificação (varredura online,
inspeção de blocos de resposta, checagem de acesso direto) antes de aprovar.

### 7.2 Gate de dedupe

- Normalização: lowercase, remoção de pontuação/espaços extras, Unicode NFKC.
- Duplicata exata: hash do texto normalizado (enunciado/passagem/transcript).
- Quase-duplicata: similaridade ≥ **0,85** (Jaccard de tokens, ratio difflib ou razão de trigramas) contra
  **todos** os itens do mesmo tipo — existentes **e** novos de outras fontes — → descartada.
- Listening/WFD/RS comparam transcript; MCQ compara passagem + pergunta; textuais usam prefixo de 200 caracteres
  normalizados.
- Relatório por tipo e fonte: encontrados · inválidos · duplicatas · integrados. Integração só via JSON +
  `build-manifest.mjs` + `bank-integrity`; ids `<fonte>-<tipo>-NN`; `src` = domínio da fonte.
- Prioridade da varredura (metas mínimas do p4): P0 `lst_hcs`/`lst_hiw`/`lst_smw`/`lst_mcq_multi`/`lst_summary`/
  `lst_mcq` (3–6 itens → +20 cada) e `answer_short`/`mcq_multi` (8 → +30); P1 `swt`/`retell_lecture`/
  `write_essay` (+20) e `mcq_single`/`read_fib`/`reorder_paragraphs` (+30); P2 `lst_fib`/`summarize_spoken_text`/
  `read_aloud` (+30); `repeat_sentence`/`wfd` só se fonte nova e 100% nova.

### 7.3 Proveniência (selos, Fase 12.1)

| Selo | Fontes (campo `source`/`src`) |
|---|---|
| `verificada` | goarno.io · pteacademy.in · band9pte.com.au · stormedugo.com · edknot.com · 79score.com · languageacademy.com.au · pearsonpte.com · ourworldindata.org (as duas últimas via `VERIFIED_SOURCES`, Fase 12.2) |
| `comunitária` | anki-cards |
| `ia` | ai-generated-prediction · pte2-prediction · pteracai-seed |

Contagem por fonte no início da Fase 12 (manifesto = 2.536): pte2-prediction 1.081 · ai-generated-prediction 647
· goarno.io 294 · pteracai-seed 200 · anki-cards 98 · pteacademy.in 107 · stormedugo.com 47 · band9pte.com.au 38 ·
edknot.com 19 · languageacademy.com.au 3 · 79score.com 2. WFD (934) e Repeat Sentence (586) não têm item `verificada`.

### 7.4 Veredito de fontes (consolidado de p3·§4.1, p4·§6 e p9·F7)

| Tipo | Fonte | Volume | Evidência | Status |
|---|---|---|---|---|
| reorder_paragraphs | goarno.io (50 re-order examples) | 50 | 35 blocos "Answer:" inspecionados | APROVADA (Fase 7.2) |
| read_fib (dropdown) | goarno.io | 50 | passagens + opções + respostas | APROVADA (7.2) |
| write_essay | band9pte.com.au (`pte-essay-topics`) | 38 | prompts + essays modelo band 9 | APROVADA (7.2) |
| lst_* (7 tipos) | goarno.io (7 páginas "Essential Questions") | 50/tipo (400 cand.) | HTTP 200, gabarito + transcript | APROVADA (7.3 P0) |
| answer_short | goarno.io (asq) | 50 | Q/A explícitos | APROVADA (P0) |
| answer_short | pteacademy.in (PDF 100 ASQ) | 90 integradas | PDF, chave consolidada | APROVADA (P0) |
| mcq_single | goarno.io (rmcq_single) | 48 | passage + a)–d) + Answer | APROVADA (P1) |
| mcq_single · mcq_multi | pteacademy.in (PDF) | 10 + 10 | gabarito "ANSWERS" / pares A,E | APROVADA (P1) |
| swt | goarno.io | 50 | Example N + modelo | APROVADA (P1) |
| retell_lecture | goarno.io | 50 | Transcript + Sample Answer | APROVADA (P1) |
| read_aloud | stormedugo.com | 47 | passagens 25–120 palavras | APROVADA (P2) |
| read_fib · reorder | edknot.com | 12 · 7 (de 12 blocos) | JSON-LD+HTML / "Correct order" | APROVADA (P1) |
| write_essay | 79score.com · languageacademy.com.au | 2 · 3 (com sample real; demais sem sample → rejeitados) | essays completos | APROVADA (parcial) |
| SGD · RTS | edknot.com · goarno.io · Pearson Test Tips | 13 · 63 | fontes verificáveis | APROVADA (12.2) |
| describe_image | Our World in Data (CC BY 4.0) + Pearson Test Tips | 6 | imagens acessíveis, atribuição renderizada | APROVADA (12.2) |
| describe_image | typogrammar.com (100 PDF) | 0 | exige e-mail | REPROVADA |
| describe_image | gurully.com (PDF "50") | 0 | ~10 imagens reais, não 50 | REPROVADA |
| mcq_multi | edknot.com | 0 | chave JS-gated | REPROVADA |
| mcq_multi | LeapScholar · PTE Magic · 7esl · testsucceed | 0 | login/paywall ou 1–3 itens | REPROVADAS |
| read_aloud | ptenepal.com | 0 | bloqueia o sandbox (43 bytes) | REPROVADA |
| reorder · read_fib | EdKnot extras | 0 | chave JS-gated | REPROVADAS |

### 7.5 Registro oficial Pearson (Fase 10.1, confirmado em 11/10/2026)

Fontes: Score Guide do PTE Academic (`pearsonpte.com/content/dam/ELL/pte/pearsonpte/pdfs/pte-academic-pdfs/
PTE-Academic-Test-Taker-Score-Guide.pdf`), páginas oficiais de formato (`/pte-academic/test-format/
speaking-writing|reading|listening`) e press release de 10/07/2025.

- **22 question types** (21 pontuados + Personal Introduction, não pontuada), válidos para provas a partir de
  **07/08/2025**. Não existe mudança "novembro 2025" (data secundária incorreta).
- **Partes e ordem fixa:** Speaking & Writing 76–84 min (9 tipos) → Reading 23–30 min (5 tipos) → Listening
  31–39 min (8 tipos); total ≈ 2 h. Escala 10–90 para o geral e cada habilidade; o geral **não** é média das 4.
- **Matriz tarefa×habilidade (Score Guide):** Read Aloud → S · Repeat Sentence → L+S · Describe Image → S ·
  Retell Lecture → L+S · Answer Short → L · Summarize Group Discussion → L+S · Respond to a Situation → S ·
  Summarize Written Text → R+W · Write Essay → W · FIB dropdown → R · MCQ multi (leitura) → R · Re-order → R ·
  FIB drag&drop → R (a página de Reading sugere +W; o Score Guide lista só R — ambiguidade registrada, tratado
  como R) · MCQ single (leitura) → R · Summarize Spoken Text → L+W · MCQ multi (listening) → L · FIB type-in → L ·
  Highlight Correct Summary → L+R · MCQ single (listening) → L · Select Missing Word → L · Highlight Incorrect
  Words → L+R · Write From Dictation → L+W. Contribuições: S 6 · L 12 · R 8 · W 4 (de 30).
  **Correção vs. planejamento original:** Read Aloud e Describe Image são **só Speaking** (o exemplo "Read Aloud →
  Speaking+Reading" não bate com a matriz oficial).
- **Revisão humana de conteúdo** (template): Describe Image, Retell Lecture, Respond to a Situation, Summarize
  Group Discussion, Summarize Written Text, Write Essay e Summarize Spoken Text; white paper de Test Integrity
  (nov/2024) trata de respostas-template. Usado só para o rótulo do detector (DT34).
- **Não confirmável oficialmente (decisão conservadora):** pesos numéricos exatos por tarefa (proprietários →
  aproximação rotulada); segundos exatos de preparo/resposta de Repeat Sentence e Answer Short → tempos do
  simulado rotulados "aproximados"; dual-skill do FIB drag&drop.

### 7.6 Tipos sem fonte e bancos rasos

Se um tipo não atingir fonte aprovada: fica com o volume atual, registrado como "sem fonte confiável" — nunca
preenchido com conteúdo de baixa probabilidade. Tipos com ≤ 6 itens recebem selo "banco pequeno". O que for
integrado traz `src` rastreável; nada é apagado sem registro (R12).

### 7.7 Contrato do glossário e invariantes (Fase 9.6, DT36/DT42)

`src/data/plus/glossary.json` — `Record<palavraMinúscula, { literal: string; contextual?: string }>`, chaves
ordenadas alfabeticamente (diff estável). `literal`: 1–3 palavras pt-BR minúsculas, sem ponto final; `contextual`:
opcional, ≤ 80 caracteres, só se acrescentar informação (colocação, falso amigo, uso acadêmico).

| # | Invariante |
|---|---|
| I1 | Toda chave pertence ao lote de entrada correspondente (sem chave inventada). |
| I2 | Chave minúscula, casa `^[a-z]+(-[a-z]+)*$`. |
| I3 | `literal` string não vazia, `trim() === literal`, sem `\n`. |
| I4 | `literal` com 1–3 palavras e ≤ 60 caracteres. |
| I5 | `contextual`, se presente: não vazia, ≤ 80 caracteres, `!== literal`. |
| I6 | Nenhuma tradução é exatamente a palavra-chave inglesa (salvo sigla idêntica explicitamente aceita). |
| I7 | Zero chaves duplicadas entre lotes; conflito → mantém a 1ª (ordem por nº do lote). |
| I8 | Cobertura = nº de chaves ÷ 6.000 ≥ 0,95 (meta operacional ≥ 0,97). |

**Qualidade linguística:** pt-BR neutro, sem gíria; sentido mais frequente no domínio PTE (`research` →
"pesquisa"); palavras funcionais recebem equivalente direto (`the` → "o, a"); termos técnicos com equivalente
consagrado; omitir nomes próprios/siglas/fragmentos sem tradução segura (**omitir, não inventar**, DT25/DT36);
nada de inglês na resposta; sem pontuação final.

### 7.8 Pipeline de produção por subagentes (DT40)

1. **Fatiar (determinístico, agente principal):** `extract-words.mjs` (em `.opencode/proj6-work/`) varreu
   `src/data/pte/*.json` (campos `text|content|transcript|prompt|passage|option|answer|model|sample|summary|
   explanation|template`), tokenizou (`[A-Za-z][A-Za-z'’-]*`, minúsculas, ≥ 2 letras, ≤ 30 caracteres) e gravou as
   **6.000** mais frequentes de 12.060 únicas em `out/words.json` (`{w, n}`); `proj8-work/split-words.mjs` fatia em 30
   lotes de 200 (`in/batch-000..029.json`). Para frases: `extract-sentences.mjs` (lotes de ~80, `in-NNN`).
2. **Traduzir (subagentes read-only):** um brief autônomo por lote (EN → pt-BR no domínio PTE Academic; entrada
   literal; saída = **só JSON** no contrato, sem cercas/markdown; omitir sem tradução segura; não editar arquivos);
   ondas de até 4 em paralelo. O agente principal grava `out/glossary-batches/batch-NNN.json` e **verifica no
   disco**. O monitor/OpenCode **não** é subagente (só monitora).
3. **Validar e mesclar (`merge-glossary.mjs` / `merge-phrases.mjs`, determinísticos e resumíveis):** lote sem
   saída válida fica pendente; valida I1–I6 por item; nunca sobrescreve chave preenchida; ordena chaves;
   imprime `RELATÓRIO: entradas · lotes_ok k/30 · pendentes · descartes · cobertura`.
4. **Resumibilidade:** reexecutar só relança lotes pendentes; lotes válidos são reaproveitados do disco.
5. **Resultado registrado:** glossário 5.776/6.000 (96,3%) · frases 3.933/3.933 (100%).

---

## 8. Testes (matriz mínima obrigatória por área)

| Área | Arquivo(s) / tipo | O que cobre |
|---|---|---|
| Dados do banco | `bank-integrity.test.ts` | Forma/conteúdo de todos os bancos (inclui SGD, RTS, DI); ids únicos; manifesto coerente |
| Ditado/lacunas | `dictation` (puro) | Mapa acertos/omissões/extras, normalização, FIB tolerante a casing, HIW acertos − falsos |
| Escrita | `summary` (puro) | SWT frase única/5–75/keywords; SST zona 50–70/cobertura |
| MCQ e correção | `mcq` (9), `essay`/`reorder`/`fibDropdown` (10) | Saldo líquido, traps, pares adjacentes, dropdowns |
| Sessão/SRS | `session-persist.test.ts`, `srs-v2.test.ts` (23) | Draft/retomada, `buildQueue` modo `"due"`; probe B5, lapso, atraso, antecipada, `PASS_SCORE` por tipo, anti-maratona, migração idempotente |
| Storage v6 | `storage-v6.test.ts` (8) | Probes A1/A2, round-trip v5→v6→export→import, mescla + undo, DST (A5), escrita coalescida |
| Diagnóstico | `diagnostics` (puro) | Breakdown, ranking de palavras, tópicos, estado vazio |
| Exame | `exam`/`exam-blueprint` | Planos, fila estratificada, `BLUEPRINT` conferido com a fonte oficial, rótulos mini × completo, `bySkill` |
| Meta/prontidão | `goal`, `estimate` (novo), `readiness` (reescrito), `studyPlan`, `report` | `daysUntil`, `targetVerdict` na mesma escala, `estimateSkill` monotônica, n < 10 → "confiança baixa", pesos = soma da matriz, IA 0,7 |
| Medição | `fase3.test.ts` (14) | Extras/repetições anotados, piso 0, `fluencyProxy`, detector de template com guarda anti-falso-positivo, copy honesta |
| Banco rastreável | `fase5-banco.test.ts` (14) | Selos por fonte, 28 pares canônicos, `mergeSnapshots` sem perda, migração funde `questions`/`bookmarks`/`notes`, filtro nas filas, contrato do `wfd.json` |
| Métricas | `metrics-sync.test.ts` (14) | Total = manifesto; soma de seções = total; strings proibidas em `src/routes/` |
| Pack | `plus-settings` (6), `plus-words`, `plus-phrases`, `plus-audio`, `plus-glossary` (10), `plus-integration.test.tsx` (17) | Normalização, lookup, alinhamento por índice, matriz de áudio, invariantes I2–I6, cobertura ≥ 95%, os 4 cenários de toggle, fala observada (nunca pt), varredura sem `"pt-BR"`/`"ouvir tradução"` |
| Acessibilidade | axe (runner) + scripts Playwright | 0 violações graves nas 10 rotas; teclado nos motores; 390×844 sem overflow |

Regras: função pura primeiro, teste antes da UI; a contagem da suíte **só cresce**; flake de import dinâmico pesado
resolve-se com timeout (`session-persist.test.ts` usa 30 s), nunca removendo teste.

---

## 9. Gates e ferramentas (R6)

1. `bunx vitest run` — suíte completa verde.
2. `bunx tsgo --noEmit` — zero erros.
3. `bun run build` — zero erros.
4. `bun run lint` — 0 erros (os 8 warnings pré-existentes são tolerados).
5. Preview `:8080` — rotas da fase verificadas por Playwright (390×844 quando a fase toca UI; sem `full_page`).
6. Sem `console.error`/pageerror novo nas rotas tocadas.
7. Nenhuma dependência nova de runtime; nenhum arquivo gerado editado (`src/routeTree.gen.ts`); nenhum segredo
   escrito ou ecoado.
8. Fases que tocam offline/PWA: build de produção via `wrangler dev` (DT43). Scripts/screenshots em `/tmp/browser/`.

Ferramentas: para tarefas de complexidade média/alta o agente consulta `.opencode/TOOLS.md` e
`.opencode/metodologia-skills` e usa as melhores tools/skills para cada tipo de trabalho (Plan.md). Subagentes
só onde o PRD prevê (DT40) e sempre read-only.

---

## 10. Fora de escopo (não fazer)

- Auth, contas, nuvem, multiusuário, sync entre dispositivos, pagamento, Lovable Cloud ou qualquer backend.
- Correção por IA/LLM; prometer nota oficial; alterar a prova para algo que ela não é.
- Bibliotecas novas de runtime (gráfico, estado, voz, tradução, NLP, i18n, dicionários); serviço de tradução ou
  de voz online em runtime — o app continua 100% offline após carregar.
- Reativar o AI Gateway ou executar `generate-*.mjs`.
- Lematização, stemming, fuzzy matching ou pluralização automática no glossário.
- Notificações push, lembretes por e-mail, calendário externo.
- Conteúdo de baixa probabilidade de ser real, itens gerados por IA em dados **novos**, ou preencher bancos rasos
  "de memória".
- Refactors cosméticos fora do que cada fase exige; mudar a estrutura de rotas sem necessidade.
- Misturar tradução/áudio do pack com o banco nativo ou com áudios/textos nativos das questões.

---

## 11. Riscos e mitigações

| Risco | Mitigação |
|---|---|
| Snapshot v6 crescer além do localStorage | ~300 KB estimados; dedupe/payload liberam espaço; aviso preventivo de quota; IndexedDB só se medido > 3,5 MB (DT7) |
| Fusão de SRS por duplicata injusta (box alto) | Fusão determinística e testada (`box = max`, contagens somadas) com caso patológico em teste |
| Matriz oficial divergir entre fontes | Prevalece a oficial; divergência registrada em `skillMatrix.ts`; decisão conservadora no simulado |
| Score de fala cair com a fórmula nova | Queda intencional e explicada na UI; constante única recalibra |
| Novos tipos sem fonte verificável | Desfecho "em breve" com motivo — a honestidade é o aceite |
| Subagente devolve JSON com cercas/comentário/truncado | Contrato rígido de saída + validação no merge; lotes de 200; lote inválido é refeito isoladamente |
| Tradução errada/alucinada | Regras linguísticas (§7.7), amostragem, invariante I6, omitir em dúvida |
| Subagente "edita" arquivos | Brief proíbe; o agente principal verifica o disco |
| Número fixo reaparecer na UI/SEO | `metrics-sync.test.ts` (DT31) |
| Regressão de áudio em pt-BR no pack | Testes de varredura e de fala observada (DT39) |
| Desvio entre trio e código (documentos antigos citados em `AGENTS.md`) | §0.3: tabela de equivalência; reconfirmar no código antes de reabrir fase |

---

## 12. Estado final, pendências e ambiguidades herdadas

### 12.1 Pendência única (aberta)

**Fase 14.2** — verificação offline em build **publicado/de produção**: tipo nunca visitado após "Baixar para
offline", atualização do SW (nova versão → aviso), e prática dos motores novos (`write_essay`,
`reorder_paragraphs`, `read_fib`, SGD, RTS, DI) offline. Origem: gate pendente do p9·F6 **mais** o item "Offline
E2E dos novos motores" do p3·F5 (o E2E offline do p5 cobriu Home, `/listening` e `/practice` de WFD, não esses
motores; o cache-first de `/assets/*` os cobre estruturalmente, mas isso não foi exercitado).

### 12.2 Divergências entre documentos de origem — como foram resolvidas

| Divergência | Resolução neste PRD |
|---|---|
| PRD 9 §3.8 dizia "Pendência: Playwright de um simulado de seção"; o roadmap 9 o dá como feito | Roadmap = registro de progresso → fechado (Reading E2E, 7/7) |
| p3·F5 marcava "Offline E2E dos novos motores" `[ ]` e o p4 dizia "executado no p5" | Cobertura parcial do p5 reconhecida; item reaberto em 14.2 |
| Planejamento p9 exemplificava "Read Aloud → Speaking+Reading" | Matriz oficial prevalece: só Speaking (§7.5) |
| PRD 9 previa `scripts/gen-precache.mjs` com lista injetada no build | Adotado download dirigido pelo cliente (DT16); o script não é caminho vigente |
| PRD 9 D4 não fixava o padrão do filtro "só verificadas" | Fixado na execução da Fase 12.1: **desligado** (DT26) |
| PRD 6 previa traduções via AI Gateway e 🔊 pt-BR no card | Superado por DT40 e DT39 (§6.2) |
| Fase 1 dizia 2.032/16; p4 2.164/19; p5/p9 2.536/19; Fase 12.2 +82 | Histórico linear (§1.3); o manifesto é a verdade atual (DT31) |

### 12.3 Ambiguidades registradas (não bloqueiam; confirmar quando tocar na área)

- **5.072 × 2.536:** "JSONs somam 5.072, manifesto 2.536" (p4/p5). 5.072 = 2 × 2.536; origem não investigada.
  Verificar se se trata de duas cópias (`.opencode/pte-json/` e `src/data/pte/`) antes de citar 5.072 em qualquer lugar.
- **Timing de RTS:** fonte oficial omite o preparo; EdKnot diz 20 s; adotado 10 s + 40 s (DT30).
- **Dual-skill do FIB drag&drop:** Reading (Score Guide) × Reading+Writing (página de Reading) — tratado como Reading.
- **Pesos 20/40/27/13:** aproximação por contagem de contribuições, não pesos reais da Pearson.
- **Estimativa 10–90:** mapeamento linear simples; rotulada "estimativa de treino", sem valor preditivo oficial.
- **Safari/iOS:** limpeza de armazenamento de sites pouco usados é conhecimento de plataforma a confirmar em
  dispositivo real; mic real, acessibilidade assistiva e layout mobile em hardware real não foram exercitados nos
  projetos de origem (só Playwright/axe).

### 12.4 Definição de pronto do projeto consolidado

Fases 1–14 com gates verdes + 14.2 cumprida com evidências + critérios globais abaixo verificados + monitor
avisado ("remix finalizado") + próximo planejamento em `project2`.

**Critérios globais (herdados do p9, verificáveis):** (1) A1–A3 e B5 não reproduzem — probes viram testes verdes;
(2) nenhuma tela mistura % com escala 10–90 e toda estimativa traz rótulo e confiança; (3) catálogo com 22 tipos
(ou motivo por tipo); (4) offline: tipo **nunca visitado** funciona após "Baixar para offline"; (5) migração
v5→v6 sem perda (round-trip testado), nenhuma tela pede reset; (6) nenhum número na UI que não derive do
manifesto; (7) pack desligado por padrão e sem fala pt-BR; (8) glossário ≥ 95% das candidatas.
