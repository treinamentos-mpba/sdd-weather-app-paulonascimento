# Discovery — Weather App

> Análise de requisitos de negócio produzida na fase de Discovery (Module 02).
> Entrada para a especificação (`specs/weather-app-spec.md`).

## Contexto

A empresa solicitou uma **aplicação de previsão do tempo** voltada ao usuário
final. O briefing indica cinco capacidades centrais: busca de cidades,
visualização do clima atual, previsão de 5 dias, alternância de unidade de
temperatura (Celsius/Fahrenheit) e uso em dispositivos móveis.

- **Problema a resolver:** o usuário precisa consultar rapidamente as condições
  do tempo de qualquer cidade, agora e nos próximos dias, em qualquer
  dispositivo.
- **Persona principal:** usuário geral (não técnico), em contexto de uso rápido
  e frequentemente mobile.
- **Fonte de dados:** API pública **Open-Meteo** (geocoding + forecast), sem
  necessidade de API key — elimina custos e gestão de credenciais.
- **Natureza da entrega:** SPA estática (React + Vite), publicável via GitHub
  Pages — sem backend próprio.

## Personas

### 👩‍💼 Persona 1: Camila, a Planejadora do Dia a Dia

**Perfil:** 34 anos, analista de RH, mora em São Paulo. Consulta o tempo todo
dia de manhã antes de sair de casa e decidir roupa/transporte.

- **Objetivo principal:** em menos de 10 segundos, saber se precisa de
  guarda-chuva ou casaco **hoje** — temperatura atual, sensação térmica e se
  vai chover.
- **Contexto de uso:** 📱 quase 100% mobile, em pé, com uma mão só e conexão
  4G instável. Sessões de 10–30 segundos, alta frequência (diária).
- **Métrica de sucesso (visão dela):** *"Abri, vi, decidi."* — da abertura do
  app até a decisão em < 10 s; a cidade dela já aparece sem precisar buscar de
  novo (ver Open Question nº 4); zero cliques desperdiçados.
- **O que a irrita:** loading sem feedback, dado defasado sem indicação
  (risco P5), precisar digitar a cidade toda vez.

### ✈️ Persona 2: Ricardo, o Viajante Frequente

**Perfil:** 42 anos, executivo de vendas, viaja 2–3 vezes por mês entre
capitais brasileiras e ocasionalmente ao exterior.

- **Objetivo principal:** ver a **previsão de 5 dias de outra cidade** (o
  destino) para montar a mala e planejar compromissos — mínimas/máximas e
  condição por dia importam mais que o clima atual.
- **Contexto de uso:** 📱🖥️ misto — mobile no aeroporto/Uber, desktop no
  escritório. Usa a **busca intensamente**, inclusive para cidades homônimas e
  internacionais (Open Questions nº 6 e 12). É a persona que usaria o
  **toggle °F** em viagens aos EUA.
- **Métrica de sucesso (visão dele):** encontra a cidade certa na primeira
  busca (desambiguação com país/região — risco P1); vê os 5 dias completos de
  forma legível; alterna °C/°F em 1 clique e o app inteiro respeita a escolha.
- **O que o irrita:** selecionar a cidade homônima errada e só perceber no
  destino; previsão que não deixa claro quais dias cobre (risco P3).

### 👴 Persona 3: Seu Antônio, o Usuário Ocasional de Baixa Familiaridade

**Perfil:** 68 anos, aposentado, usa o celular principalmente para WhatsApp.
Consulta o tempo quando vai visitar os netos ou antes de eventos familiares.

- **Objetivo principal:** entender **sem esforço** se "vai fazer frio" na
  cidade dos netos no fim de semana — linguagem clara, letras legíveis, sem
  jargão meteorológico.
- **Contexto de uso:** 📱 mobile (Android de entrada), fonte do sistema
  aumentada, toque impreciso. Usa raramente (1–2×/mês), então **não lembra**
  como o app funciona — cada visita é como a primeira.
- **Métrica de sucesso (visão dele):** completa a busca sozinho; entende o
  resultado na primeira olhada (ícone + texto da condição); se algo der
  errado, a mensagem diz **o que fazer** em português claro.
- **Métrica implícita de a11y:** alvos de toque generosos, contraste alto no
  tema dark, zoom de texto a 200% sem quebrar layout, estado vazio com
  orientação explícita (riscos P6 e P7; RNF-02).

### Como as personas tensionam o produto

| Persona | Puxa o produto para… | Valida requisitos |
|---|---|---|
| Camila | Velocidade, persistência da última cidade, clima atual em destaque | RF-02, RF-06, RNF-03, OQ-4, OQ-8 |
| Ricardo | Busca robusta + desambiguação, previsão 5 dias, toggle °C/°F | RF-01, RF-03, RF-04, OQ-1, OQ-6, OQ-7 |
| Seu Antônio | Acessibilidade, clareza, estados de erro/vazio orientativos | RF-06, RF-07, RNF-02, P6, P7 |

> Juntas, as três personas cobrem todos os requisitos funcionais do briefing e
> justificam os NFRs de a11y e performance — nenhuma feature fica "órfã" de
> persona, e nenhuma persona pede algo fora do escopo da v1.

## Requisitos Funcionais

| ID | Requisito | Origem no briefing |
| --- | --- | --- |
| RF-01 | Buscar cidades por nome, exibindo resultados do geocoding para seleção | "buscar cidades" |
| RF-02 | Exibir o **clima atual** da cidade selecionada (temperatura, condição do tempo e metadados relevantes como sensação térmica, umidade e vento) | "ver o clima atual" |
| RF-03 | Exibir a **previsão de 5 dias** (mín./máx. e condição por dia) | "ver a previsão de 5 dias" |
| RF-04 | Alternar a exibição de temperaturas entre **Celsius e Fahrenheit** | "alternar entre Celsius e Fahrenheit" |
| RF-05 | Funcionar plenamente em **dispositivos móveis** (layout responsivo, mobile-first) | "usar em dispositivos móveis" |

Comportamentos implícitos (derivados, a validar na spec):

- RF-06 — Tratar os estados de **carregamento**, **erro** (falha de rede/API) e
  **vazio** (nenhum resultado para a busca).
- RF-07 — Indicar claramente qual cidade está selecionada e a unidade ativa.

## Requisitos Não-Funcionais

| ID | Categoria | Requisito |
| --- | --- | --- |
| RNF-01 | Responsividade | Layout mobile-first, utilizável de 320px a desktop |
| RNF-02 | Acessibilidade | Navegação por teclado, roles/labels semânticos, contraste adequado (tema dark) |
| RNF-03 | Performance | Resposta perceptível rápida; feedback de loading em toda operação de rede |
| RNF-04 | Confiabilidade | Falhas da API tratadas com mensagem clara e opção de tentar novamente |
| RNF-05 | Simplicidade operacional | Sem API key, sem backend, deploy estático |
| RNF-06 | Qualidade | Código TypeScript strict, testes unitários (Vitest) e E2E (Playwright) |

## Riscos

### Riscos técnicos

| # | Risco | Probabilidade | Impacto | Mitigação |
|---|-------|:---:|---------|-----------|
| T1 | **Open-Meteo indisponível ou lenta** (API pública, sem SLA) | Média | Usuário sem dados; app parece quebrado | Timeout explícito (~10 s) + estado de erro com retry; shell estático nunca depende da API para renderizar |
| T2 | **Mudança no contrato da API** (campos renomeados/removidos sem aviso) | Baixa | Quebra silenciosa; telas com `undefined` | Isolar rede em `src/services/`; tipar e validar as respostas na borda; testes de service com fixtures reais |
| T3 | **Rate limiting do geocoding** com autocomplete sem controle | Média | Busca para de funcionar durante o uso | Debounce (300–500 ms), mínimo de caracteres, cancelamento de requisições obsoletas (`AbortController`) |
| T4 | **Condições de corrida na busca** (resposta antiga chega depois da nova) | Média | Usuário vê clima de cidade diferente da buscada | Cancelar requisição anterior no hook `useWeather`; ignorar respostas fora de ordem |
| T5 | **Conversão °C/°F espalhada pelo código** | Média | Valores inconsistentes entre componentes | Armazenar sempre Celsius; conversão só na apresentação via função pura em `src/lib/` com testes unitários |
| T6 | **Mapeamento incompleto dos códigos WMO** (código desconhecido da API) | Média | Ícone/descrição vazios ou crash | Mapa centralizado em `lib/` com fallback explícito ("condição desconhecida"); teste cobrindo código inexistente |
| T7 | **Fuso horário nas datas da previsão** (dia "errado" para o usuário) | Média | Previsão desalinhada (off-by-one de datas) | Usar o parâmetro `timezone` da Open-Meteo; tratar datas como locais da cidade, não UTC; testar com fusos extremos |
| T8 | **Deploy no GitHub Pages com paths quebrados** (`base` do Vite) | Baixa | App em branco em produção, embora funcione local | Configurar `base` no Vite para o subpath do Pages; validar com `pnpm preview` e smoke test pós-deploy (Module 11) |
| T9 | **Testes E2E flaky por depender da API real** | Alta (se não prevenido) | CI vermelho intermitente; perda de confiança nos testes | Regra do projeto: nunca chamar API real — `page.route` no Playwright e mock de `fetch` no Vitest |

### Riscos de produto

| # | Risco | Probabilidade | Impacto | Mitigação |
|---|-------|:---:|---------|-----------|
| P1 | **Cidades homônimas → clima da cidade errada** sem o usuário perceber | Alta | Informação falsa com aparência de correta; perda de confiança | Lista de desambiguação com país/região; exibir sempre cidade + país selecionados no resultado |
| P2 | **Scope creep** (favoritos, mapas, alertas, geolocalização "já que estamos aqui") | Alta | Atraso; v1 nunca fecha | Seção *Out of Scope* explícita na spec; toda ideia nova volta para o backlog da v2 |
| P3 | **Interpretação divergente de "5 dias"** (inclui hoje ou não) | Média | Entrega rejeitada na validação com stakeholder | Resolver a Open Question 🔴 nº 1 antes da spec; critério de aceite explícito e teste E2E contando os dias |
| P4 | **"Mobile" subespecificado** (responsivo vs. PWA/offline) | Média | Expectativa frustrada ou esforço 10× maior que o planejado | Fechar na spec: responsivo mobile-first ≥ 320 px, sem PWA na v1 (registrado em Assumptions) |
| P5 | **Dados defasados sem indicação** (usuário confia em dado velho) | Média | Decisões erradas do usuário (ex.: sair sem guarda-chuva); desgaste da marca | Exibir "atualizado às HH:MM"; buscar dados frescos a cada seleção de cidade |
| P6 | **Acessibilidade tratada como "polish" final** | Média | Retrabalho 5× maior; exclusão de usuários; possível não-conformidade | A11y nos critérios de aceite desde a spec; queries acessíveis nos testes (`getByRole`) forçam o padrão |
| P7 | **Estado vazio/erro mal desenhados** (telas "mortas" sem orientação) | Média | Usuário não sabe o que fazer; abandono no primeiro uso | Tratar vazio/erro/loading como telas de primeira classe na spec, com texto de orientação e ação (retry/buscar) |
| P8 | **Sem métrica de sucesso definida** | Alta | Impossível declarar a entrega "pronta"; priorização vira opinião | Resolver Open Questions ⚪ 17–18: definir critério mínimo (ex.: fluxo busca→previsão funcional, Lighthouse ≥ 90) |

### Leitura executiva

- **Mais perigosos (probabilidade alta × impacto alto):** P1 (cidade errada),
  P2 (scope creep) e T9 (testes flaky) — os três têm mitigação barata se
  aplicada **desde o início**.
- **Mais caros se descobertos tarde:** T7 (fuso horário), T8 (base path do
  Pages) e P4 (PWA) — típicos de "funcionou no dev, quebrou na entrega".
- Vários riscos são mitigados pela **própria arquitetura planejada** (services
  isolados, lib pura, Celsius interno) — reforçando por que o Plan vem antes
  do Code.

## Decisões

> Decisões de produto fechadas com o stakeholder para destravar a
> especificação (Module 03). Cada decisão indica o que resolve das Open
> Questions abaixo.

| ID | Decisão | Justificativa | Resolve |
| --- | --- | --- | --- |
| D-01 | **Fonte de dados: Open-Meteo** (geocoding + forecast), sem API key | Gratuita, sem gestão de credenciais nem backend proxy — compatível com SPA estática no GitHub Pages; cobertura global de cidades | Confirma A-01; responde parcialmente a OQ-12 (busca é global, com suporte a `language` no geocoding) |
| D-02 | **"5 dias" = hoje + 4 dias seguintes** | O dia atual é o mais consultado (persona Camila); exibi-lo na previsão evita duplicar informação em componentes distintos e elimina o risco de off-by-one (P3) | Resolve OQ-1 🔴 |
| D-03 | **Unidade padrão: Celsius** | Público-alvo primário é brasileiro (UI pt-BR); °F fica a 1 clique para o caso de viagem (persona Ricardo). Internamente tudo em Celsius, conversão só na apresentação (A-02, risco T5) | Resolve a parte "unidade padrão" de OQ-8 (a persistência da escolha segue em aberto) |
| D-04 | **Sem autenticação e sem persistência de servidor** | App 100% estático: menor custo, menor superfície de segurança, deploy simples no Pages. Qualquer persistência futura (unidade, última cidade) será local (localStorage) | Confirma A-04; restringe as soluções possíveis de OQ-4 e OQ-8 a armazenamento local |
| D-05 | **Idioma da UI: pt-BR** | Persona e mercado-alvo brasileiros; i18n fica fora da v1 (evita custo de retrofit mapeado no risco de escopo) | Resolve OQ-15 (datas em formato pt-BR; demais idiomas ficam para versões futuras) |

**Ainda em aberto após estas decisões (prioridade para o Module 03):**
OQ-2 (granularidade da previsão), OQ-3 (composição do clima atual), OQ-4
(estado inicial — agora limitado a vazio ou última cidade via localStorage),
OQ-5 (autocomplete), OQ-6 (desambiguação), OQ-7 (escopo do toggle), OQ-9
(profundidade do "mobile"), OQ-10 (falhas/timeout) e OQ-11 (out of scope).

## Perguntas em Aberto (Open Questions)

### 🔴 Bloqueantes da spec (responder antes do Module 03)

1. ~~A previsão de 5 dias **inclui o dia atual** ou são os 5 dias seguintes?~~
   ✅ **Resolvida por D-02:** hoje + 4 dias seguintes.
2. Qual a **granularidade** da previsão: por dia (mín/máx) ou por hora dentro
   do dia? Inclui precipitação e probabilidade de chuva?
   - *Impacto:* previsão horária é um produto ~3× maior que a diária — a
     ambiguidade mais cara do briefing.
3. O que compõe exatamente o **"clima atual"**: só temperatura + condição, ou
   também sensação térmica, umidade, vento, pressão, UV, nascer/pôr do sol?
   - *Impacto:* cada campo a mais muda layout, chamada de API e testes; sem
     lista fechada, o escopo infla silenciosamente.
4. Deve haver uma **cidade padrão** ao abrir o app, um estado vazio com convite
   à busca, ou a última cidade consultada (implica persistência)?
   - *Impacto:* define a primeira impressão do produto e pode introduzir um
     requisito de persistência não especificado.

### 🟠 Decidir durante a spec

5. A busca deve sugerir resultados **enquanto o usuário digita** (autocomplete)
   ou apenas ao submeter?
   - *Impacto:* muda arquitetura (debounce, cancelamento de requisições), UX e
     volume de chamadas à API.
6. Quantos resultados de geocoding exibir em caso de **ambiguidade** e com
   quais dados de desambiguação (país, estado, população)?
   - *Impacto:* sem isso, o usuário pode ver o clima da cidade errada sem
     perceber — o app parece funcionar e entrega informação falsa.
7. O toggle °C/°F converte **apenas temperaturas** ou também outras unidades
   (vento km/h↔mph)?
   - *Impacto:* app meio-convertido parece quebrado justamente para a persona
     que usaria °F.
8. A preferência de unidade (°C/°F) deve **persistir** entre sessões
   (localStorage)? ✅ **Parcialmente resolvida por D-03/D-04:** unidade padrão
   é Celsius; se houver persistência, será via localStorage. **Em aberto:**
   persistir ou não.
   - *Impacto:* sem persistência, o usuário reconfigura a cada visita;
     adicioná-la depois significa mexer em estado já testado.
9. "Usar em dispositivos móveis" significa **site responsivo** ou algo mais
   (PWA instalável, offline)? Qual o menor viewport suportado (320px)?
   - *Impacto:* a interpretação varia ordens de grandeza em esforço; PWA
     descoberto tarde = refazer build e estratégia de cache.
10. Qual o comportamento esperado sob **falha da API**: mensagem + retry?
    Timeout de quantos segundos? E com dados parciais?
    - *Impacto:* APIs públicas vão falhar; sem spec, o comportamento sob falha
      fica indefinido (tela branca em produção).
11. Confirmamos o **out of scope** da v1: favoritos, histórico, alertas de
    tempo severo, mapas e geolocalização ficam de fora?
    - *Impacto:* sem out-of-scope escrito, cada demo gera um "mas eu achei que
      teria…" — é a principal defesa contra scope creep.

### 🟡 Podem virar suposição registrada (default razoável + registro)

12. Busca **global ou só Brasil**? "Munique" deve encontrar "München"?
    Tolerância a erro de digitação?
    - *Impacto:* define parâmetros do geocoding e expectativa do usuário;
      "não acha minha cidade" é motivo nº 1 de abandono.
13. O que acontece com **entrada inválida/vazia** na busca (só espaços, 2
    caracteres, emojis)?
    - *Impacto:* sem critério, cada dev decide diferente e o QA não tem o que
      testar.
14. Os dados precisam de **atualização automática** (polling) ou só ao buscar?
    Mostramos "atualizado às HH:MM"?
    - *Impacto:* dado defasado sem indicação destrói confiança; polling não
      planejado adiciona complexidade imprevista.
15. ~~Há necessidade de **idiomas** além de pt-BR na interface? Datas em qual
    formato?~~
    ✅ **Resolvida por D-05:** UI somente em pt-BR; datas em formato pt-BR.
16. Detecção automática de localização (**geolocalização** do navegador) está
    no escopo desta versão?
    - *Impacto:* envolve permissões, fallback e privacidade — melhor decidir
      explicitamente do que descobrir na implementação.

### ⚪ Processo (não bloqueiam a spec, mas definem "pronto")

17. Quais são as **métricas de sucesso** do app (uso, performance,
    satisfação)?
    - *Impacto:* sem critério de sucesso, trade-offs viram opinião e não há
      como declarar a entrega concluída.
18. Existe **deadline**? Pode haver um MVP (busca + clima atual) com a
    previsão em uma fase 2?
    - *Impacto:* sem fases, tudo é P0 e nada é negociável; qualquer atraso
      vira crise.

## Suposições (Assumptions)

- A-01 — A fonte de dados será a **Open-Meteo** (geocoding + forecast), sem
  API key, conforme stack definida pelo projeto.
- A-02 — Temperaturas serão armazenadas internamente em **Celsius**; a
  alternância °C/°F é apenas de apresentação e **não** dispara nova requisição.
- A-03 — "Previsão de 5 dias" = 5 dias com mínima, máxima e condição por dia.
- A-04 — O app é uma **SPA sem backend**; não há autenticação nem persistência
  de dados no servidor.
- A-05 — O estado inicial do app é **vazio** (sem cidade selecionada), com
  orientação para buscar.
- A-06 — Interface em **pt-BR**; identificadores de código em en-US.
- A-07 — Suporte aos navegadores modernos (evergreen); sem requisito de IE/legados.