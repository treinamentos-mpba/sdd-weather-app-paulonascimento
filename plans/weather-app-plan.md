# Weather App — Plano Técnico

> Entrada: [`specs/weather-app-spec.md`](../specs/weather-app-spec.md) (fonte
> da verdade). Saída seguinte: `tasks/weather-app-tasks.md` (Module 05).
>
> Este plano define **decisões e contratos** — não é código final. Cada
> decisão referencia o requisito (FR/NFR/AC/EC) que a motiva.

---

## 1. Architecture Overview

Arquitetura em camadas, com dependências apontando sempre "para baixo":

```text
┌─────────────────────────────────────────────────────┐
│  components/            (apresentação, sem rede)    │
│  SearchBar · SearchResults · CurrentWeatherCard ·   │
│  ForecastList · UnitToggle · states/ (4 estados)    │
└───────────────▲─────────────────────────────────────┘
                │ props e callbacks
┌───────────────┴─────────────────────────────────────┐
│  hooks/useWeather        (orquestração + estado)    │
└───────────────▲─────────────────────────────────────┘
                │ chama e mapeia
┌───────────────┴─────────────────────────────────────┐
│  services/               (única camada com rede)    │
│  geocoding.ts · forecast.ts · http.ts (timeout)     │
└───────────────▲─────────────────────────────────────┘
                │ usa
┌───────────────┴─────────────────────────────────────┐
│  lib/ (funções puras) · types/ (contratos)          │
│  temperature.ts · wmo.ts · format.ts                │
└─────────────────────────────────────────────────────┘
```

Regras arquiteturais (invariantes, verificáveis em review):

- **R1** — Componentes e hooks **não conhecem URLs** nem `fetch`; toda rede
  vive em `services/` (mitiga risco T2; convenção do projeto).
- **R2** — Temperaturas trafegam **sempre em Celsius** no estado e nos tipos;
  conversão para °F acontece só na renderização, via `lib/temperature.ts`
  (FR-06, AC-06.2, risco T5).
- **R3** — `lib/` contém apenas **funções puras** (sem I/O, sem Date.now
  implícito) — 100% testáveis por unidade.
- **R4** — Os 4 estados de interface (inicial/loading/erro/vazio, FR-07) são
  representados por **tipos de união discriminada**, nunca por booleans
  soltos (`isLoading && !error && ...`).

### 1.1 Responsabilidades por camada e impacto nos testes

| Camada | O que ela sabe fazer | O que ela **não** sabe fazer | Como isso facilita os testes |
| --- | --- | --- | --- |
| **`components/`** (apresentação) | Renderizar dados recebidos via props; emitir intenções do usuário via callbacks (`onSearch`, `onSelect`, `onToggle`) | Fazer rede, conhecer URLs, formatar números brutos, conhecer o estado global | Teste = `render` + props + `getByRole`. Cada componente é uma função **props → DOM**: os 4 estados do FR-07 viram 4 renders isolados, sem mock de rede algum (AC-01.x, 02.x, 04.x, 07.x) |
| **`hooks/useWeather`** (orquestração/estado) | Sequenciar busca→seleção→clima; manter `SearchState`/`WeatherState`; cancelar operações obsoletas; expor `retry` | Saber como a UI desenha o estado; montar URLs (delega ao service) | Testável com `renderHook` injetando services **mockados por injeção de dependência**: dá para dirigir cada transição da união discriminada (R4) e provar o cancelamento de race (EC-09) sem renderizar nenhum componente |
| **`services/`** (acesso a dados) | Única camada que conhece HTTP: montar URLs/params (§6), timeout 10 s (NFR-04), validar na borda (EC-07) e mapear JSON → tipos do domínio | Saber que existe React; manter estado de UI; decidir o que é erro "visível" (retorna `WeatherApiError` tipado; quem decide é o hook) | Teste com `fetch` mockado: assere **parâmetros da URL** (AC-05.3) e cada ramo da taxonomia de erros (EC-05/06/07/16) — sem navegador, sem DOM, em milissegundos |
| **`lib/`** (funções puras) | Conversão °C→°F, mapeamento WMO→texto/ícone, formatação de data/hora | Ler estado, relógio do sistema ou rede: mesma entrada ⇒ mesma saída, sempre | São as funções mais baratas de testar do projeto (função pura = teste de tabela); e carregam a lógica de **domínio mais sensível a bug** — conversão e arredondamento (EC-11, T5) e fallback WMO (EC-08, T6) |

**Por que esta separação (e não, ex., tudo no App):**

1. **A lógica perigosa fica onde o teste é mais barato.** O que mais quebra
   silenciosamente (conversão de unidades, mapeamento WMO, zip dos arrays
   paralelos, timeout) está em `lib/` e `services/` — testes unitários de
   milissegundos cobrem dezenas de casos. Se essa lógica vivesse dentro de
   componentes, exigiria mocks de rede e renders completos para cada caso.
2. **O que precisa de browser fica onde ele é inevitável.** Componentes
   testam só "props → DOM acessível"; E2E testa só a **composição** (fluxos
   feliz/erro/vazio). Resultado: a pirâmide do §9 fica realmente em formato
   de pirâmide — poucos testes lentos e frágeis no topo.
3. **Mudança de API = mudança localizada.** Se a Open-Meteo renomear campos
   (risco T2), só `services/` muda; tipos, hook e UI permanecem. Se o design
   mudar (ex.: lista vira dropdown), só `components/` muda.
4. **Fronteiras verificáveis em review.** R1–R4 viram regras mecânicas de
   import: quem revisa o PR pode verificar "componente importou de
   `services/`?" lendo o diff — sem executar nada (o Review Agent usa isso).

## 2. Tech Stack

| Camada | Escolha | Justificativa |
| --- | --- | --- |
| Linguagem | TypeScript strict | C-03; uniões discriminadas para os estados (R4) |
| UI | React 19 + Vite | C-03; SPA estática para GitHub Pages (C-04) |
| Estilo | Tailwind CSS (tema dark glassmorphism) | C-03; tokens de contraste p/ NFR-02 |
| Estado | **React puro** (`useState`/`useReducer` no hook) | Escopo pequeno; Redux/Zustand/React Query seriam over-engineering (ver §10) |
| Dados | `fetch` nativo + `AbortController` | Sem dependência extra; timeout de 10 s (NFR-04) e cancelamento (EC-09) |
| Testes unit | Vitest + Testing Library | C-03; queries acessíveis reforçam NFR-02 |
| Testes E2E | Playwright com `page.route` | C-03; API real nunca é chamada (risco T9) |
| Lint/format | Biome | C-03 |

**Nenhuma dependência de runtime além de `react` e `react-dom`.**

## 3. Project Structure

```text
src/
├── components/
│   ├── SearchBar.tsx            # form de busca (FR-01)
│   ├── SearchResults.tsx        # lista de desambiguação (FR-02)
│   ├── CurrentWeatherCard.tsx   # clima atual + "Atualizado às" (FR-04, FR-09)
│   ├── ForecastList.tsx         # 5 dias (FR-05)
│   ├── ForecastDayCard.tsx      # item de dia (AC-05.2)
│   ├── UnitToggle.tsx           # °C/°F (FR-06)
│   └── states/
│       ├── InitialState.tsx     # AC-07.1
│       ├── LoadingState.tsx     # AC-07.2 (role="status")
│       ├── ErrorState.tsx       # AC-07.3 (role="alert" + retry)
│       └── EmptyState.tsx       # AC-07.4
├── hooks/
│   └── useWeather.ts            # orquestração: busca → seleção → clima
├── services/
│   ├── http.ts                  # fetchJson com timeout + AbortSignal
│   ├── geocoding.ts             # searchCities()
│   └── forecast.ts              # fetchWeather()
├── lib/
│   ├── temperature.ts           # celsiusToFahrenheit, formatTemp
│   ├── wmo.ts                   # weatherCodeToLabel/Icon (+ fallback EC-08)
│   └── format.ts                # formatTime, formatDayName ("Hoje", pt-BR)
├── types/
│   └── weather.ts               # contratos do §4
├── styles/
│   └── index.css                # Tailwind + tokens do tema
├── App.tsx                      # composição + estado da unidade
└── main.tsx
tests/
├── unit/                        # espelha src/ (lib, services, components, hooks)
└── e2e/                         # fluxos Playwright + fixtures de mock
```

Um componente por arquivo; `components/` nunca importa de `services/`
(somente via props do `App`/hook — R1).

## 4. Data Model

Contratos em `src/types/weather.ts` (valores sempre em Celsius — R2):

```ts
/** Cidade retornada pelo geocoding (FR-02). */
interface City {
  id: number;            // id da Open-Meteo (chave de lista)
  name: string;          // "Santos"
  admin1?: string;       // região/estado — "São Paulo"
  country: string;       // "Brasil"
  latitude: number;
  longitude: number;
}

/** Clima atual — composição fechada do FR-04. */
interface CurrentWeather {
  temperature: number;        // °C
  apparentTemperature: number;// °C (sensação)
  humidity: number;           // %
  windSpeed: number;          // km/h
  weatherCode: number;        // código WMO (mapeado em lib/wmo.ts)
  time: string;               // ISO local da cidade (FR-09)
}

/** Um dia da previsão (FR-05). */
interface ForecastDay {
  date: string;               // "2026-10-29" (fuso da cidade — EC-10)
  tempMin: number;            // °C (invariante: tempMin ≤ tempMax, AC-05.2)
  tempMax: number;            // °C
  weatherCode: number;
}

/** Resposta agregada do forecast. */
interface WeatherData {
  city: City;
  current: CurrentWeather;
  daily: ForecastDay[];       // 0–5 itens (EC-13: pode vir incompleto)
}

type Unit = 'celsius' | 'fahrenheit';

/** Taxonomia de erros dos services (EC-05/06/07/16). */
type WeatherApiErrorKind = 'network' | 'timeout' | 'http' | 'invalid-data';
```

Estados do hook (uniões discriminadas — R4):

```ts
type SearchState =
  | { status: 'idle' }
  | { status: 'searching' }
  | { status: 'results'; cities: City[] }   // exibida mesmo com 1 (AC-02.4)
  | { status: 'empty' }                      // AC-07.4
  | { status: 'error'; retry: () => void };

type WeatherState =
  | { status: 'idle' }                       // AC-07.1
  | { status: 'loading' }
  | { status: 'success'; data: WeatherData }
  | { status: 'error'; retry: () => void };
```

> `SearchState` e `WeatherState` são **independentes** para suportar AC-07.5
> (painel anterior visível durante nova busca) e EC-14 (erro localizado).

## 5. Data Flow

```text
[SearchBar] submit("Santos")
  → valida: trim().length >= 2 (AC-01.3/01.4) — falha não gera requisição
  → useWeather.search(term)
      aborta busca anterior (EC-09) · SearchState: 'searching'
      → services/geocoding.searchCities(term)
      → 0 resultados → 'empty' | n resultados → 'results' (máx. 5)
[SearchResults] select(city)
  → fecha lista, campo mantém termo (AC-03.2)
  → useWeather.select(city)
      aborta forecast anterior (EC-09) · WeatherState: 'loading'
      (painel anterior permanece até aqui — AC-07.5)
      → services/forecast.fetchWeather(city)  // lat/lon, timezone=auto,
                                              // forecast_days=5 (AC-05.3)
      → valida/mapeia resposta → WeatherData (EC-07) → 'success'
[UnitToggle] toggle
  → App: setUnit('fahrenheit')  // só re-render; zero requisições (AC-06.2)
  → componentes formatam via lib/temperature na renderização (R2)
```

Retry (AC-07.3/EC-12): cada estado `error` carrega um closure `retry` que
repete a **mesma operação com os mesmos parâmetros** — sem estado global de
"última operação".

## 6. External APIs (Open-Meteo)

### 6.1 Geocoding — busca de cidades (FR-01/02)

```text
GET https://geocoding-api.open-meteo.com/v1/search
  ?name={termo}&count=5&language=pt&format=json
```

| Parâmetro | Valor | Motivo |
| --- | --- | --- |
| `name` | termo com `encodeURIComponent` | EC-15 (caracteres especiais) |
| `count` | `5` | máximo da lista de desambiguação (AC-02.1) |
| `language` | `pt` | nomes localizados (S-08, D-05) |
| `format` | `json` | formato padrão da resposta |

**Exemplo de resposta (resumido):**

```json
{
  "results": [
    {
      "id": 3449433,
      "name": "Santos",
      "latitude": -23.96083,
      "longitude": -46.33361,
      "country": "Brasil",
      "country_code": "BR",
      "admin1": "São Paulo",
      "timezone": "America/Sao_Paulo",
      "population": 411403
    }
  ],
  "generationtime_ms": 0.7
}
```

> ⚠️ **Sem resultados, a chave `results` é OMITIDA** (não vem `[]`). O
> service deve tratar `results === undefined` como lista vazia → estado
> vazio (EC-03), nunca como `invalid-data`.

**Mapeamento `results[i]` → `City`:**

| Campo da API | Campo do tipo | Observação |
| --- | --- | --- |
| `id` | `City.id` | chave de lista React |
| `name` | `City.name` | localizado (pt) |
| `admin1` | `City.admin1?` | **pode faltar** em vilarejos/territórios |
| `country` | `City.country` | exibido na desambiguação (AC-02.1) |
| `latitude` / `longitude` | `City.latitude` / `City.longitude` | repassados ao forecast (AC-03.1) |

Campos ignorados: `elevation`, `feature_code`, `population`, `timezone`
(o forecast usa `timezone=auto`), `*_id` — não entram no domínio (YAGNI).

### 6.2 Forecast — clima atual + 5 dias (FR-04/05/09)

```text
GET https://api.open-meteo.com/v1/forecast
  ?latitude={lat}&longitude={lon}
  &current=temperature_2m,apparent_temperature,relative_humidity_2m,
           wind_speed_10m,weather_code
  &daily=weather_code,temperature_2m_max,temperature_2m_min
  &timezone=auto&forecast_days=5
```

| Parâmetro | Valor | Motivo |
| --- | --- | --- |
| `latitude`/`longitude` | da `City` selecionada | AC-03.1 |
| `current` | os 5 campos do FR-04 | composição fechada (S-02) |
| `daily` | código + mín/máx | FR-05, granularidade diária (S-03) |
| `timezone` | `auto` | API resolve o fuso pela coordenada → "Hoje" correto (AC-05.3, EC-10) |
| `forecast_days` | `5` | hoje + 4 (D-02, AC-05.3) |

Unidades padrão da API já são **°C** e **km/h** — nenhum parâmetro de
unidade é enviado; conversão °F é local (R2, AC-06.2).

**Exemplo de resposta (resumido):**

```json
{
  "latitude": -23.96,
  "longitude": -46.33,
  "timezone": "America/Sao_Paulo",
  "utc_offset_seconds": -10800,
  "current_units": { "temperature_2m": "°C", "wind_speed_10m": "km/h" },
  "current": {
    "time": "2026-10-29T14:30",
    "temperature_2m": 22.6,
    "apparent_temperature": 24.1,
    "relative_humidity_2m": 78,
    "wind_speed_10m": 12.4,
    "weather_code": 3
  },
  "daily_units": { "temperature_2m_max": "°C" },
  "daily": {
    "time": ["2026-10-29", "2026-10-30", "2026-10-31", "2026-11-01", "2026-11-02"],
    "weather_code": [3, 61, 95, 0, 2],
    "temperature_2m_max": [26.8, 24.3, 22.1, 27.5, 28.0],
    "temperature_2m_min": [17.2, 18.0, 16.4, 17.9, 19.1]
  }
}
```

**Mapeamento `current` → `CurrentWeather`:**

| Campo da API | Campo do tipo | Observação |
| --- | --- | --- |
| `current.temperature_2m` | `temperature` | °C, arredonda só na view |
| `current.apparent_temperature` | `apparentTemperature` | sensação |
| `current.relative_humidity_2m` | `humidity` | % inteiro |
| `current.wind_speed_10m` | `windSpeed` | km/h, não convertido (S-06) |
| `current.weather_code` | `weatherCode` | traduzido em `lib/wmo.ts` (EC-08) |
| `current.time` | `time` | ISO local da cidade → "Atualizado às HH:MM" (FR-09) |

**Mapeamento `daily` → `ForecastDay[]` (zip dos arrays paralelos):**

A API retorna `daily` como **arrays paralelos** indexados por dia; o service
"des-paraleliza" para objetos:

```text
para cada i em daily.time:
  ForecastDay {
    date        = daily.time[i]                 // "YYYY-MM-DD", fuso da cidade
    weatherCode = daily.weather_code[i]
    tempMax     = daily.temperature_2m_max[i]
    tempMin     = daily.temperature_2m_min[i]
  }
```

Regras do zip: iterar pelo **menor** comprimento entre os quatro arrays
(EC-13 — resposta parcial nunca gera `undefined`); `daily.length` final de
0 a 5. `WeatherData.city` vem da `City` selecionada (não da resposta).

**Validação na borda (ambos os services, EC-07):** antes de mapear,
verificar presença e tipo (`typeof === 'number'`/`'string'`) de cada campo
usado; qualquer violação → `WeatherApiError('invalid-data')` → ErrorState.
Sem biblioteca de schema (ver §10).

### 6.3 Contrato dos services

```ts
// services/geocoding.ts
searchCities(term: string, signal?: AbortSignal): Promise<City[]>

// services/forecast.ts
fetchWeather(city: City, signal?: AbortSignal): Promise<WeatherData>

// services/http.ts — único ponto com fetch
fetchJson<T>(url: string, opts: { signal?: AbortSignal }): Promise<T>
// timeout de 10 s combinando AbortSignal.timeout(10_000) com o signal do
// chamador; lança WeatherApiError { kind: WeatherApiErrorKind }
```

## 7. State Management

### 7.1 Onde cada estado vive

| Estado | Onde vive | Por quê |
| --- | --- | --- |
| `SearchState` + `WeatherState` | `useWeather` (único hook, consumido pelo `App`) | Dados de rede e sua orquestração — um único dono evita fontes de verdade duplicadas |
| `unit: Unit` | `useState` no `App` | É estado de **apresentação**, não de dados (R2); não pertence ao hook de dados |
| Nada mais | — | Sem URL sync, sem localStorage, sem Context (S-04/S-05; árvore de ~2 níveis faz props bastarem — §10) |

**Injeção de dependência:** `useWeather` recebe os services como parâmetro
opcional (`useWeather(deps = { searchCities, fetchWeather })`) — produção
usa os reais, testes injetam mocks. Sem isso, testar o hook exigiria mockar
módulos (Vitest `vi.mock`), o que acopla o teste ao sistema de imports.

**Persistência:** nenhuma na v1 (S-04/S-05). O hook expõe uma API que
permitirá adicionar localStorage na v1.1 sem refactor (`initialUnit`).

### 7.2 Estados explícitos (uniões discriminadas — R4)

**Busca (geocoding):**

| Estado | Quando | UI renderizada |
| --- | --- | --- |
| `idle` | Antes da 1ª busca | nada além do campo |
| `searching` | Requisição de geocoding em voo | spinner da lista (o clima anterior permanece — AC-07.5) |
| `results` (≥ 1 cidade) | Resposta OK com itens — inclusive 1 (AC-02.4) | `SearchResults` com nome/região/país |
| `empty` | `results` ausente ou vazio | `EmptyState` sem retry (AC-07.4) |
| `error` (carrega `retry`) | falha/timeout/HTTP/inválida | `ErrorState` + "Tentar novamente" |

**Clima (forecast):**

| Estado | Quando | UI renderizada |
| --- | --- | --- |
| `idle` | Nenhuma cidade selecionada | `InitialState` (AC-07.1) |
| `loading` | Requisição de forecast em voo | `LoadingState` (`role="status"`, AC-07.2) |
| `success` | Resposta mapeada | `CurrentWeatherCard` + `ForecastList` |
| `error` (carrega `retry`) | falha/timeout/HTTP/inválida | `ErrorState` localizado (EC-14) |

> Nota: `empty` existe **só** na busca — "geocoding sem resultados" não é
> erro (EC-03). Forecast vazio não existe: toda `City` selecionada tem
> coordenadas válidas; falta de dados no forecast é `invalid-data` (EC-07)
> ou parcial (EC-13).

**Transições válidas (grafo mínimo):**

```text
SearchState:  idle ──submit──▶ searching ─┬─▶ results ──select──▶ (lista fecha; permanece exibível até nova busca)
                                          ├─▶ empty  ──submit──▶ searching
                                          └─▶ error  ──retry/submit──▶ searching
WeatherState: idle ──select──▶ loading ───┬─▶ success ──select──▶ loading
                                          └─▶ error  ──retry/select──▶ loading
```

### 7.3 Conversão °C/°F por derivação na renderização

- **A fonte de verdade é sempre Celsius.** `WeatherData` trafega em °C do
  service ao hook ao componente (R2); `unit` nunca entra em nenhuma
  requisição — a API nem é consultada (AC-06.2 garantido por construção).
- **Na renderização**, cada componente recebe o valor em °C + `unit` e exibe
  `formatTemp(value, unit)`, que aplica conversão + arredondamento **uma
  única vez**, na borda da UI:

```ts
// lib/temperature.ts — contrato
celsiusToFahrenheit(c: number): number        // c * 9/5 + 32
formatTemp(celsius: number, unit: Unit): string
// formatTemp(22.6, 'celsius') → "23°" · formatTemp(20, 'fahrenheit') → "68°F"
```

- **Implicação:** toggle = `setUnit()` no `App` → re-render puro → zero
  requisições (AC-06.2 testável com spy de `fetch`).
- Vento não é convertido (S-06): só `formatTemp` conhece `Unit`.

### 7.4 Concorrência

`useWeather` mantém um `AbortController` **por operação** (busca e forecast
independentes): nova operação aborta a anterior, tornando respostas fora de
ordem impossíveis por construção (EC-09, T4). Sem dedup/cache: S-09 manda
refrescar a cada seleção.

## 8. Error Handling Strategy

> Princípio: **services nunca propagam `throw` até a UI** — toda falha vira
> um `WeatherApiError` tipado, que o hook converte no estado `error`
> correspondente (NFR-05: o shell nunca quebra). A UI lê só o estado (R4).
> Validação de input (`trim().length < 2`) **não** é erro: não gera
> requisição nem estado de erro, só mensagem no campo (AC-01.3).

### 8.1 Rede (offline / DNS / CORS)

- **Detecção:** `fetch` rejeita com `TypeError` → kind `network`.
- **UI:** ErrorState "Não foi possível conectar — verifique sua conexão" +
  "Tentar novamente" (EC-05, AC-07.3).

### 8.2 API (HTTP 4xx/5xx, incluindo 429)

- **Detecção:** `response.ok === false` → kind `http`.
- **UI:** ErrorState genérico "Não foi possível carregar os dados…" + retry.
  **Códigos HTTP nunca são expostos** ao usuário (EC-16); o `status` só vive
  em log de desenvolvimento (`console.error`).

### 8.3 Timeout (10 s)

- **Detecção:** `AbortSignal.timeout(10_000)` combinado com o signal do
  chamador em `http.ts`; abort do timeout → kind `timeout`.
- **UI:** ErrorState "A resposta demorou demais…" + retry (EC-06, NFR-04).
- **Distinção obrigatória:** aborto por **timeout** → `timeout`; aborto por
  **nova operação do usuário** (EC-09) → silenciado, nunca vira estado de erro.

### 8.4 Resposta parcial ou inválida

| Caso | Detecção | Resultado |
| --- | --- | --- |
| Campo obrigatório ausente/tipo errado | validação na borda (§6) | kind `invalid-data` → ErrorState; **nunca** `undefined`/`NaN` na UI (EC-07) |
| `results` omitido no geocoding | chave ausente (não `[]`!) | estado **empty**, não erro (EC-03) |
| Forecast com menos de 5 dias | arrays diários curtos | **success** com os dias existentes; zip pelo menor array (EC-13) |
| Falha só no forecast (ou só na busca) | erro em uma operação | `error` localizado no bloco; o outro estado intacto (EC-14) |
| WMO desconhecido | lookup falha em `wmo.ts` | não é erro — "Condição desconhecida" + ícone neutro (EC-08) |

### 8.5 Retry

Cada estado `error` carrega um closure `retry` que repete **a mesma operação
com os mesmos parâmetros** (AC-07.3, EC-12) — sem estado global de "última
ação". Retry de busca refaz o geocoding do termo; retry de clima refaz o
forecast da cidade. O estado vazio **não** oferece retry (AC-07.4).

### 8.6 Acessibilidade dos estados

Erros em `role="alert"` (anunciados imediatamente); loading em
`role="status"` (AC-07.2); todas as mensagens em pt-BR iniciadas por
"Não foi possível…" (AC-07.3).

## 9. Testing Strategy

Pirâmide: muitos unitários (rápidos), poucos E2E (fluxos). API real **nunca**
é chamada (risco T9). A tabela de rastreabilidade da spec (§3) é o checklist
de cobertura.

| Nível | Alvo | Ferramentas | Cobre |
| --- | --- | --- | --- |
| Unit (puro) | `lib/temperature` (20→68, −3.5→26, arredondamento), `lib/wmo` (códigos + fallback), `lib/format` ("Hoje", HH:MM) | Vitest | AC-06.3, EC-08, EC-11, parte de AC-05.1 |
| Unit (service) | `geocoding`/`forecast`/`http` com `fetch` mockado: URL/params corretos, timeout, taxonomia de erros, validação de borda | Vitest + mock de `fetch` | AC-05.3, EC-05/06/07/15/16 |
| Unit (hook) | `useWeather`: transições de estado, abort de operação anterior, retry com mesmos parâmetros | Vitest + `renderHook` | EC-09, EC-12, EC-14, R4 |
| Component | cada componente nos estados relevantes, com queries acessíveis (`getByRole`, `getByLabelText`) | Testing Library | AC-01.x, 02.x, 04.x, 07.x, 08.1 |
| E2E | fluxo feliz (busca→lista→clima→toggle), fluxo de erro + retry, estado vazio, viewport mobile 320 px | Playwright + `page.route` | AC-03.x, 06.1/06.2, 07.5, 09.x, NFR-01 |

Convenções: fixtures compartilhadas em `tests/e2e/fixtures/` (respostas
reais da Open-Meteo capturadas uma vez); cada teste falha por uma única
razão; testar comportamento, não implementação.

## 10. Risks & Trade-offs

| Decisão | Alternativa rejeitada | Trade-off aceito |
| --- | --- | --- |
| React puro para estado | React Query / SWR | Perdemos cache/dedup automáticos — irrelevante: 1 recurso por vez, dados renovados por seleção (S-09). Menos 1 dependência, menos conceitos |
| Validação manual na borda | Zod/Valibot | Menos garantia exaustiva de schema; suficiente porque consumimos ~10 campos de 1 API estável. Revisitar se o modelo crescer |
| Props em 2 níveis | Context API | Props "verbosas" no App — mas explícitas e fáceis de testar; Context seria indireção sem ganho nesta árvore rasa |
| `fetch` + `AbortController` nativos | Axios | Sem interceptors/retry automático — não precisamos; retry é manual por requisito (AC-07.3) |
| Dois estados independentes (search/weather) | Máquina de estados única | Mais combinações possíveis — necessário para AC-07.5 e EC-14; a união discriminada mantém cada um simples |
| `timezone=auto` | Passar timezone do geocoding | Dependemos da resolução da API — é o comportamento documentado e cobre EC-10 com menos código |
| Ícones por texto/emoji mapeados em `wmo.ts` | Biblioteca de ícones | Visual mais simples — zero dependência; ícones são decorativos (a11y pela palavra, AC §4) |

**Riscos do plano** (além dos herdados da spec): `AbortSignal.timeout` exige
navegadores recentes — coberto pelo NFR-06 (evergreen); tema glassmorphism
pode ameaçar contraste ≥ 4.5:1 — validar tokens de cor com Lighthouse na
tarefa de UI base (NFR-02).
