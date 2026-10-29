# Weather App — Especificação de Produto

> Fonte única da verdade para o desenvolvimento do Weather App.
> Entrada: [`specs/discovery.md`](./discovery.md). Saída seguinte:
> `plans/weather-app-plan.md` (Module 04).
>
> Rastreabilidade: FR = Functional Requirement · US = User Story ·
> AC = Acceptance Criteria · NFR = Non-Functional Requirement ·
> EC = Edge Case. Referências a D-xx, OQ-xx, T-xx e P-xx apontam para o
> discovery.

---

## 1. Overview

O **Weather App** é uma aplicação web gratuita, sem cadastro e sem backend
próprio, que permite a qualquer pessoa buscar uma cidade e consultar o **clima
atual** e a **previsão de 5 dias (hoje + 4)**, com alternância entre **Celsius
e Fahrenheit** e experiência **mobile-first**.

- **Problema:** o usuário precisa decidir rapidamente "levo guarda-chuva?",
  "que roupa ponho na mala?" — agora e nos próximos dias, em qualquer cidade.
- **Personas:** Camila (consulta diária rápida, mobile), Ricardo (viajante,
  busca intensa e previsão de 5 dias, usa °F) e Seu Antônio (uso ocasional,
  precisa de clareza e acessibilidade). Detalhes no discovery.
- **Fonte de dados:** API pública **Open-Meteo** (geocoding + forecast), sem
  API key (D-01).
- **Entrega:** SPA estática (React + Vite + Tailwind), publicada no GitHub
  Pages. UI em **pt-BR** (D-05).

**Critério de sucesso da v1:** o fluxo completo *buscar cidade → ver clima
atual → ver previsão de 5 dias → alternar °C/°F* funciona nos navegadores
suportados, em viewport de 320 px a desktop, com os **quatro** estados de
interface (inicial, carregando, erro, vazio) tratados e os critérios de
aceite desta spec verificados por testes automatizados.

---

## 2. Functional Requirements

| ID | Requisito | Prioridade | Origem |
| --- | --- | :---: | --- |
| FR-01 | O usuário pode **buscar uma cidade por nome**, submetendo o termo (botão ou Enter). Validação: após `trim()`, o termo deve ter **≥ 2 caracteres**; o campo limita a entrada a **100 caracteres** (`maxLength`). | P0 | RF-01 |
| FR-02 | O sistema exibe a **lista de resultados do geocoding** (até 5), cada um com **nome, região/estado e país**, para desambiguação de homônimos. A lista é exibida **mesmo com um único resultado** — o usuário sempre confirma a seleção; nunca há seleção automática. | P0 | RF-01, P1 |
| FR-03 | Ao **selecionar um resultado**, o sistema **fecha a lista** (o campo mantém o termo digitado) e carrega os dados de clima daquela cidade. | P0 | RF-01 |
| FR-04 | O sistema exibe o **clima atual** da cidade selecionada com exatamente os campos meteorológicos: temperatura, sensação térmica, condição do tempo (ícone decorativo + texto em pt-BR), umidade relativa e velocidade do vento (km/h) — além do horário de referência (FR-09). | P0 | RF-02, OQ-3 |
| FR-05 | O sistema exibe a **previsão de 5 dias = hoje + 4 seguintes**, cada dia com: nome do dia, temperatura mínima, máxima e condição (ícone + texto). Granularidade diária (sem horária). | P0 | RF-03, D-02 |
| FR-06 | O usuário pode **alternar entre °C e °F**; a troca converte **todas** as temperaturas exibidas (atual, sensação, mín/máx dos 5 dias) **sem nova requisição** à API. Padrão: Celsius. | P0 | RF-04, D-03 |
| FR-07 | O sistema trata e exibe os estados: **inicial** (sem cidade, com convite à busca), **carregando** (indicador visível), **erro** (mensagem clara em pt-BR + ação "Tentar novamente") e **vazio** (busca sem resultados, com orientação). Durante uma nova busca, o clima da cidade anterior **permanece visível** até a nova seleção. | P0 | RF-06 |
| FR-08 | O sistema mantém **permanentemente visíveis** na tela a cidade selecionada (nome + país) e a unidade de temperatura ativa. | P1 | RF-07 |
| FR-09 | O sistema exibe o **horário de referência dos dados** ("Atualizado às HH:MM") junto ao clima atual, usando o horário retornado pela API (`current_weather.time`) no fuso da cidade. | P1 | P5, OQ-14 |

---

## 3. User Stories

> Formato: **Como [persona], quero [ação] para [valor]**. Personas definidas
> no [discovery](./discovery.md#personas). Cada story referencia os
> requisitos funcionais que a realizam.

### Camila — a Planejadora do Dia a Dia

- **US-01** — Como **Camila**, quero **buscar minha cidade pelo nome e ver o
  clima atual em poucos segundos** para **decidir roupa e transporte antes de
  sair de casa**.
  ↳ Requisitos: **FR-01** (busca), **FR-03** (seleção), **FR-04** (clima atual)
- **US-02** — Como **Camila**, quero **ver um indicador de carregamento
  enquanto os dados chegam** para **saber que o app está funcionando mesmo com
  minha conexão 4G instável**.
  ↳ Requisitos: **FR-07** (estado de loading)
- **US-03** — Como **Camila**, quero **ver a que horas os dados foram
  atualizados** para **confiar na informação antes de tomar minha decisão**.
  ↳ Requisitos: **FR-09** (atualidade dos dados)

### Ricardo — o Viajante Frequente

- **US-04** — Como **Ricardo**, quero **ver uma lista de cidades com estado e
  país quando houver nomes repetidos** para **ter certeza de que estou vendo o
  clima do meu destino, e não de uma cidade homônima**.
  ↳ Requisitos: **FR-02** (desambiguação), **FR-08** (cidade visível)
- **US-05** — Como **Ricardo**, quero **ver a previsão de 5 dias (hoje + 4)
  com mínimas e máximas de cada dia** para **montar a mala e planejar os
  compromissos da viagem**.
  ↳ Requisitos: **FR-05** (previsão de 5 dias)
- **US-06** — Como **Ricardo**, quero **alternar entre Celsius e Fahrenheit
  com um clique e ver todas as temperaturas convertidas** para **usar o app
  naturalmente em viagens aos Estados Unidos**.
  ↳ Requisitos: **FR-06** (toggle °C/°F), **FR-08** (unidade ativa visível)

### Seu Antônio — o Usuário Ocasional

- **US-07** — Como **Seu Antônio**, quero **entender a condição do tempo por
  um ícone e um texto simples em português** para **saber se "vai fazer frio"
  sem depender da ajuda dos meus filhos**.
  ↳ Requisitos: **FR-04** (condição com ícone + texto), **FR-05** (previsão)
- **US-08** — Como **Seu Antônio**, quero **receber uma mensagem clara dizendo
  o que fazer quando algo der errado ou a busca não encontrar nada** para
  **não ficar perdido em uma tela vazia**.
  ↳ Requisitos: **FR-07** (estados de erro e vazio)

### Matriz de cobertura stories × requisitos

| | FR-01 | FR-02 | FR-03 | FR-04 | FR-05 | FR-06 | FR-07 | FR-08 | FR-09 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| US-01 | ✅ | | ✅ | ✅ | | | | | |
| US-02 | | | | | | | ✅ | | |
| US-03 | | | | | | | | | ✅ |
| US-04 | | ✅ | | | | | | ✅ | |
| US-05 | | | | | ✅ | | | | |
| US-06 | | | | | | ✅ | | ✅ | |
| US-07 | | | | ✅ | ✅ | | | | |
| US-08 | | | | | | | ✅ | | |

> Todos os 9 requisitos funcionais estão cobertos por pelo menos uma story —
> nenhum requisito órfão, nenhuma story sem requisito.

### Rastreabilidade US → AC → NFR

> Tabela-mestre para a quebra de tarefas (Module 05) e para os testes
> (Modules 08–09): cada story lista os critérios de aceite que a verificam,
> os NFRs que a restringem e os edge cases associados. Uma tarefa que
> implemente uma story só está "pronta" quando seus ACs passam e os NFRs
> listados são respeitados.

| Story | Acceptance Criteria | NFRs relevantes | Edge Cases |
| --- | --- | --- | --- |
| **US-01** (Camila — buscar e ver clima atual) | AC-01.1…01.5 · AC-03.1, 03.2 · AC-04.1…04.3 | NFR-01 (mobile-first) · NFR-03 (carga) · NFR-04 (resposta/timeout) | EC-01, 02, 09, 11, 15 |
| **US-02** (Camila — feedback de carregamento) | AC-07.2 | NFR-02 (`role="status"`) · NFR-04 · NFR-05 | EC-06 |
| **US-03** (Camila — atualidade dos dados) | AC-09.1, 09.2 | NFR-04 | EC-07 |
| **US-04** (Ricardo — desambiguação de cidades) | AC-02.1…02.4 · AC-08.1 | NFR-02 (teclado, `:focus-visible`) · NFR-01 | EC-03, 04 |
| **US-05** (Ricardo — previsão de 5 dias) | AC-05.1…05.3 | NFR-01 (5 cartões em 320 px) | EC-10, 11, 13 |
| **US-06** (Ricardo — toggle °C/°F) | AC-06.1…06.4 · AC-08.1 | NFR-02 (`aria-pressed`) | EC-11 |
| **US-07** (Seu Antônio — condição clara com ícone + texto) | AC-04.2, 04.3 · AC-05.2 | NFR-02 (contraste ≥ 4.5:1, zoom 200%, ícone decorativo) | EC-08 |
| **US-08** (Seu Antônio — erro e vazio orientativos) | AC-07.1, 07.3, 07.4, 07.5 | NFR-02 (`role="alert"`) · NFR-05 (nunca tela branca) | EC-05, 06, 07, 12, 14, 16 |

**Verificação de completude da rastreabilidade:**

- **ACs órfãos:** nenhum — os 27 cenários (AC-01.1 a AC-09.2) aparecem em
  pelo menos uma linha.
- **ECs órfãos:** nenhum — EC-01…16 estão todos associados a uma story.
- **NFRs não rastreados por story:** NFR-06 (compatibilidade) e NFR-07
  (privacidade) são **transversais** — valem para todas as stories e são
  verificados no nível do app (matriz de navegadores e inspeção de rede),
  não por cenário.

---

## 4. Acceptance Criteria

> Formato **Given / When / Then** (Gherkin), prontos para virar testes
> automatizados (Vitest/Testing Library e Playwright) nos Modules 08–09.
>
> **Convenções globais (valem para todos os cenários):**
> - A API real **nunca** é chamada — respostas mockadas (`page.route` /
>   mock de `fetch`) com os dados citados.
> - Todos os valores numéricos exibidos (temperatura, umidade, vento) são
>   **arredondados para o inteiro mais próximo**.
> - Ícones de condição são **decorativos** (`aria-hidden="true"`); o texto
>   da condição é a informação acessível.
> - Os Edge Cases da seção 6 complementam estes cenários.

### FR-01 — Busca de cidade

```gherkin
AC-01.1 — Busca disparada por submissão
  Given o app está no estado inicial com o campo de busca vazio
  When o usuário digita "São Paulo" e pressiona Enter
  Then exatamente 1 requisição de geocoding é disparada com o termo "São Paulo"
  And o indicador de carregamento fica visível até a resposta

AC-01.2 — Busca disparada pelo botão
  Given o campo de busca contém "Recife"
  When o usuário aciona o botão "Buscar"
  Then exatamente 1 requisição de geocoding é disparada com o termo "Recife"

AC-01.3 — Entrada abaixo do mínimo é bloqueada
  Given o campo de busca contém "a" (1 caractere após trim)
  When o usuário tenta submeter
  Then nenhuma requisição é disparada
  And uma mensagem de validação associada ao input (aria-describedby) informa
      o mínimo de 2 caracteres

AC-01.4 — Entrada vazia ou só espaços é bloqueada
  Given o campo de busca contém "   " (apenas espaços)
  When o usuário tenta submeter
  Then nenhuma requisição é disparada
  And a mensagem de campo obrigatório é exibida de forma acessível

AC-01.5 — Limite de tamanho do campo
  Given o campo de busca tem maxLength=100
  When o usuário tenta inserir 150 caracteres
  Then o valor do campo contém no máximo 100 caracteres
```

### FR-02 — Resultados e desambiguação

```gherkin
AC-02.1 — Lista com dados de desambiguação
  Given o geocoding retorna 5+ resultados para "Springfield"
  When a resposta é renderizada
  Then no máximo 5 opções são exibidas
  And cada opção exibe nome, região/estado e país (ex.: "Springfield,
      Illinois, Estados Unidos")

AC-02.2 — Nenhuma seleção automática
  Given o geocoding retorna 3 resultados para "Santos"
  When a lista é exibida
  Then nenhum forecast é requisitado até o usuário selecionar uma opção

AC-02.3 — Operável por teclado
  Given a lista de resultados está exibida
  When o usuário navega apenas com Tab/setas e pressiona Enter em uma opção
  Then a opção focada é selecionada
  And o elemento focado tem indicador visível (:focus-visible)
  And a ordem do foco segue a ordem visual da lista

AC-02.4 — Resultado único ainda exige confirmação
  Given o geocoding retorna exatamente 1 resultado para "Curitiba"
  When a resposta é renderizada
  Then a lista é exibida com essa única opção (nome, região/estado e país)
  And nenhum forecast é requisitado até o usuário selecioná-la
```

### FR-03 — Seleção de cidade

```gherkin
AC-03.1 — Seleção dispara forecast da coordenada correta
  Given a lista exibe "Santos, São Paulo, Brasil" com latitude -23.96 e
        longitude -46.33
  When o usuário seleciona essa opção
  Then 1 requisição de forecast é disparada com latitude=-23.96 e
       longitude=-46.33
  And o painel de clima exibe "Santos" como cidade ativa

AC-03.2 — Lista fechada após seleção
  Given a lista de resultados está exibida e o campo contém "Santos"
  When o usuário seleciona uma opção
  Then a lista de resultados é fechada
  And o campo de busca mantém o termo "Santos"
```

### FR-04 — Clima atual

```gherkin
AC-04.1 — Composição exata do painel
  Given o forecast mockado retorna temperatura 22.6°C, sensação 24.1°C,
        código WMO 3, umidade 78% e vento 12.4 km/h
  When o painel de clima atual é renderizado
  Then ele exibe exatamente os campos: "23°" (temperatura arredondada),
       sensação "24°", condição "Nublado" com ícone, umidade "78%" e vento
       "12 km/h"
  And o horário de referência (FR-09) é exibido no painel
  And nenhum campo meteorológico adicional (pressão, UV, etc.) é exibido

AC-04.2 — Código WMO conhecido tem texto em pt-BR
  Given o forecast retorna código WMO 95
  When a condição é renderizada
  Then o texto exibido é "Tempestade" (nunca o número 95)

AC-04.3 — Código WMO desconhecido tem fallback
  Given o forecast retorna código WMO 999 (não mapeado)
  When a condição é renderizada
  Then o texto exibido é "Condição desconhecida" com ícone neutro
  And nenhum erro é lançado e a UI não exibe campo em branco
```

### FR-05 — Previsão de 5 dias

```gherkin
AC-05.1 — Exatamente 5 dias começando hoje
  Given o app solicita a previsão com forecast_days=5 e o mock retorna 5
        dias a partir de hoje no fuso da cidade
  When a previsão é renderizada
  Then exatamente 5 itens de dia são exibidos
  And o primeiro item é rotulado "Hoje"
  And os demais exibem o nome do dia da semana em pt-BR (ex.: "quinta")

AC-05.2 — Conteúdo de cada dia
  Given o dia mockado tem mínima 17.2°C, máxima 26.8°C e código WMO 61
  When o item do dia é renderizado
  Then ele exibe mínima "17°", máxima "27°" e condição "Chuva fraca" com ícone
  And a mínima exibida é sempre ≤ máxima

AC-05.3 — Parâmetros da requisição de forecast
  Given a requisição de forecast é disparada
  When os parâmetros são inspecionados
  Then o parâmetro timezone=auto está presente (a API resolve o fuso pela
       coordenada), garantindo que "Hoje" reflita o fuso da cidade
  And o parâmetro forecast_days=5 está presente
```

### FR-06 — Alternância °C/°F

```gherkin
AC-06.1 — Conversão de todas as temperaturas
  Given o clima está exibido em °C com atual 20°, sensação 22°, e dias com
        mín/máx 15°/25°
  When o usuário ativa o toggle °F
  Then todas as temperaturas exibem o valor convertido com sufixo °F:
       atual 68°F, sensação 72°F (71.6 arredondado), mín/máx 59°F/77°F

AC-06.2 — Sem nova requisição
  Given o clima de uma cidade está carregado
  When o usuário alterna °C → °F → °C
  Then zero novas requisições de rede são disparadas

AC-06.3 — Fórmula e arredondamento
  Given uma temperatura interna de 20°C
  When convertida para Fahrenheit
  Then o valor é 68 (F = C × 9/5 + 32, arredondado para o inteiro mais
       próximo; -3.5°C → 26°F)

AC-06.4 — Padrão Celsius
  Given o app acabou de carregar
  When o primeiro clima é exibido
  Then a unidade ativa é °C
```

### FR-07 — Estados da interface

```gherkin
AC-07.1 — Estado inicial
  Given o app foi aberto e nenhuma busca foi feita
  When a tela é renderizada
  Then o texto "Busque uma cidade para ver a previsão" é exibido
  And nenhum painel de clima é renderizado

AC-07.2 — Estado de loading acessível
  Given uma busca ou carga de forecast está em andamento
  When o usuário aguarda
  Then um indicador de carregamento com role="status" (anunciado a leitores
       de tela) permanece visível até a resposta

AC-07.3 — Estado de erro com retry
  Given a requisição de forecast falha (HTTP 500 ou falha de rede)
  When o erro é tratado
  Then um contêiner com role="alert" exibe mensagem em pt-BR iniciada por
       "Não foi possível"
  And um botão "Tentar novamente" está visível
  And ao acioná-lo, a mesma operação é refeita com os mesmos parâmetros
  And em caso de sucesso no retry, o clima é exibido normalmente

AC-07.4 — Estado vazio distinto de erro
  Given o geocoding retorna lista vazia para "Xyzabc123"
  When a resposta é tratada
  Then a mensagem "Nenhuma cidade encontrada" com sugestão de verificar a
       grafia é exibida
  And não há botão "Tentar novamente" (não é um erro recuperável por retry)

AC-07.5 — Painel anterior preservado em nova busca
  Given o clima de "Santos" está exibido
  When o usuário dispara uma nova busca por "Rio de Janeiro"
  Then o painel de "Santos" permanece visível durante o carregamento da lista
  And só é substituído após a seleção de um novo resultado
```

### FR-08 — Contexto visível

```gherkin
AC-08.1 — Cidade e unidade sempre identificáveis
  Given o usuário selecionou "Lisboa, Portugal" e ativou °F
  When a tela de clima é exibida
  Then o texto "Lisboa, Portugal" está visível no painel
  And o toggle indica visualmente (e via aria-pressed/aria-checked) que °F
      está ativo
```

### FR-09 — Atualidade dos dados

```gherkin
AC-09.1 — Horário de referência visível
  Given o forecast mockado tem horário de referência 14:30 (fuso da cidade)
  When o painel de clima atual é renderizado
  Then o texto "Atualizado às 14:30" está visível

AC-09.2 — Dados renovados a cada seleção
  Given o usuário já consultou a cidade A
  When ele busca e seleciona a cidade B
  Then uma nova requisição de forecast é disparada para B (sem reuso de cache
       da sessão anterior de A)
```

---

## 5. Non-Functional Requirements

| ID | Categoria | Requisito (mensurável) |
| --- | --- | --- |
| NFR-01 | Responsividade | Layout funcional e sem scroll horizontal de **320 px** a desktop; mobile-first. |
| NFR-02 | Acessibilidade | Conformidade **WCAG 2.1 AA**: navegação completa por teclado, foco visível, roles/labels semânticos, contraste ≥ 4.5:1 no tema dark, atualização de resultados anunciada via `aria-live`, zoom de texto 200% sem perda de função. |
| NFR-03 | Performance (carga) | Lighthouse (preset mobile, throttling padrão): **Performance ≥ 90** e LCP ≤ 2,5 s; bundle JS inicial ≤ 200 KB gzip. |
| NFR-04 | Performance (interação) | **Timeout de rede de 10 s** (via `AbortController`) com transição para o estado de erro — verificável em teste com mock de atraso. Meta: resposta renderizada em ≤ 3 s com API saudável. |
| NFR-05 | Resiliência | O shell do app **nunca** quebra (tela branca/crash) por falha da API; toda falha degrada para o estado de erro com retry (FR-07). |
| NFR-06 | Compatibilidade | Duas últimas versões de Chrome, Firefox, Safari e Edge, incluindo iOS Safari e Android Chrome. |
| NFR-07 | Privacidade | Todas as chamadas via HTTPS; nenhum dado pessoal coletado ou enviado a terceiros. |

### Constraints (restrições técnicas, não negociáveis nesta versão)

- **C-01** — SPA estática, sem backend próprio e sem autenticação (D-04).
- **C-02** — Dados exclusivamente da Open-Meteo, sem API key (D-01).
- **C-03** — Stack: TypeScript strict, React + Vite, Tailwind CSS; testes com
  Vitest + Testing Library e Playwright; lint com Biome; pnpm.
- **C-04** — Deploy no GitHub Pages (atenção ao `base` path — risco T8).
- **C-05** — UI em pt-BR; identificadores de código em en-US (D-05).

---

## 6. Edge Cases

| ID | Cenário | Comportamento esperado |
| --- | --- | --- |
| EC-01 | Busca vazia ou só espaços | Submissão bloqueada; mensagem acessível de campo obrigatório (AC-01.2). Nenhuma requisição. |
| EC-02 | Busca com < 2 caracteres após `trim()` | Idem EC-01, mensagem de mínimo de caracteres. |
| EC-03 | Cidade inexistente ("Xyzabc123") | Estado **vazio** com orientação (AC-07.4) — não é tratado como erro. |
| EC-04 | Cidades homônimas | Lista de desambiguação com região + país (AC-02.1); nunca selecionar automaticamente a primeira. |
| EC-05 | Falha de rede (offline) | Estado de **erro** com mensagem "verifique sua conexão" + retry (AC-07.3). |
| EC-06 | Timeout (> 10 s sem resposta) | Requisição abortada; estado de erro com retry (NFR-04). |
| EC-07 | Resposta da API malformada ou com campos ausentes | Estado de erro genérico; nunca renderizar `undefined`/`NaN` na UI (risco T2). |
| EC-08 | Código WMO desconhecido | Fallback "Condição desconhecida" + ícone neutro (AC-04.2, risco T6). |
| EC-09 | Buscas consecutivas rápidas (A, depois B) | Apenas o resultado de **B** é exibido; resposta tardia de A é descartada/cancelada (risco T4). |
| EC-10 | Fuso horário da cidade ≠ fuso do usuário | "Hoje" e os nomes dos dias refletem o fuso **da cidade** (parâmetro `timezone` da API — risco T7). |
| EC-11 | Temperaturas negativas e casas decimais | Exibição correta de valores negativos (ex.: −3°); arredondamento consistente para inteiro em toda a UI. |
| EC-12 | Retry após erro | O botão "Tentar novamente" repete a **última** operação (busca ou forecast) com os mesmos parâmetros. |
| EC-13 | Forecast com menos de 5 dias no array diário | **Degradação graciosa:** exibir os dias disponíveis, sem cartões vazios; nunca `undefined` na UI. |
| EC-14 | Clima atual carrega, previsão falha (ou vice-versa) | **Erro localizado** no bloco afetado ("Não foi possível carregar a previsão" + retry do bloco); o bloco saudável permanece (NFR-05). |
| EC-15 | Caracteres especiais na busca (acentos, hífens, emojis, HTML) | Termo enviado com `encodeURIComponent`; acentos/hífens aceitos ("São João del-Rei"); emoji/símbolo segue o fluxo normal (vazio se sem resultados); termo re-exibido sempre como **texto literal**, nunca interpretado como HTML. |
| EC-16 | HTTP 429 (rate limit) ou 5xx da API | Estado de erro padrão (AC-07.3) com mensagem genérica; códigos HTTP **nunca** expostos ao usuário. |

---

## 7. Assumptions

> Herdadas do discovery (A-xx) ou adotadas nesta spec (S-xx) para destravar a
> escrita — validar com o stakeholder; mudanças geram revisão da spec.

- **S-01** — Busca **por submissão** (Enter/botão), sem autocomplete na v1
  (resolve OQ-5 de forma conservadora; autocomplete fica em Out of Scope).
- **S-02** — "Clima atual" tem a composição **fechada** do FR-04 (resolve
  OQ-3); pressão, UV e nascer/pôr do sol ficam fora da v1.
- **S-03** — Previsão com **granularidade diária**; sem dados horários e sem
  probabilidade de chuva na v1 (resolve OQ-2).
- **S-04** — Estado inicial **vazio** com convite à busca (resolve OQ-4);
  "lembrar última cidade" via localStorage é candidata à v1.1.
- **S-05** — A preferência °C/°F **não persiste** entre sessões na v1
  (fecha OQ-8; persistência via localStorage é candidata à v1.1).
- **S-06** — O toggle converte **apenas temperaturas**; vento permanece em
  km/h na v1 (fecha OQ-7).
- **S-07** — "Mobile" = site **responsivo mobile-first**; PWA/offline fora da
  v1 (fecha OQ-9).
- **S-08** — Busca global com `language=pt`; sem tolerância a typo além da
  oferecida pela própria API (fecha OQ-12).
- **S-09** — Dados atualizados **a cada seleção de cidade**; sem polling
  automático (fecha OQ-14, combinada com FR-09).
- **A-02** — Temperaturas armazenadas internamente em **Celsius**; conversão
  apenas na apresentação.
- **A-07** — Navegadores evergreen (formalizada no NFR-06).

---

## 8. Risks

> Registro completo (probabilidade × impacto × mitigação) no
> [discovery](./discovery.md#riscos). Os mais relevantes para esta spec:

| Risco | Tratamento nesta spec |
| --- | --- |
| P1 — Clima da cidade errada (homônimos) | FR-02 + AC-02.1 + EC-04 tornam a desambiguação obrigatória. |
| P2 — Scope creep | Seção 9 (Out of Scope) fechada; toda ideia nova exige revisão da spec. |
| T1 — API indisponível/lenta | NFR-04 (timeout 10 s), NFR-05, AC-07.3, EC-05/06. |
| T4 — Race condition na busca | EC-09 com comportamento verificável. |
| T6 — Código WMO desconhecido | AC-04.2 + EC-08. |
| T7 — Fuso horário | AC-05.1 + EC-10. |
| T9 — Testes flaky | Testes nunca chamam a API real (estratégia detalhada no plano). |

---

## 9. Out of Scope (v1)

Explicitamente **fora** desta versão (fecha OQ-11):

- Autocomplete/sugestões enquanto digita (S-01)
- Favoritos, histórico de buscas e múltiplas cidades simultâneas
- Geolocalização automática do navegador (OQ-16 → fora da v1)
- Alertas de tempo severo, mapas e radar
- Previsão horária, probabilidade de chuva, pressão, UV, nascer/pôr do sol
- PWA, modo offline e notificações
- Persistência de preferências e de última cidade (candidatas à v1.1)
- Outros idiomas além de pt-BR (D-05)
- Autenticação, contas e qualquer backend próprio (D-04)

---

## 10. Open Questions

> Não bloqueiam o plano nem as primeiras tarefas, mas precisam de resposta do
> stakeholder antes do fim do projeto.

1. **Métricas de sucesso do produto** (OQ-17): além do critério técnico da
   seção 1, há meta de negócio (uso, satisfação)? *Proposta:* adotar
   Lighthouse ≥ 90 (performance e a11y) como régua mínima da v1.
2. **Deadline e faseamento** (OQ-18): confirmar se a v1 completa é a entrega
   única ou se há valor em um corte intermediário (busca + clima atual).
3. **Validação das assumptions S-01…S-09** com o stakeholder: qualquer
   rejeição (ex.: querer autocomplete ou persistência na v1) reabre a spec e
   o escopo.
