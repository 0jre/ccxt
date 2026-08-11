# Plano de Melhorias — 0jre/ccxt

> Avaliação e roadmap gerados em 05–11 ago 2026 a partir de clone raso do fork.
> Versão do fork: **4.5.70** · upstream: **4.5.73** (release de 11 ago 2026).

## Diagnóstico

| Métrica | Atual | Observação |
|---|---|---|
| Atraso vs upstream | 3 patches | 4.5.70 → 4.5.73 |
| Exchanges | 105 | 17 certificadas |
| Cobertura WebSocket (Pro) | 79 / 105 (75%) | 26 exchanges só-REST |
| Fixtures de resposta estáticos | 88 | 9 exchanges standalone sem fixture |
| TODO/FIXME em `ts/src/**` | 36 | inclui 2 de `aster.ts` (liq. price COIN-M) |
| Suporte Rust | early-stage | `rust.yml` sem transpile/testes |

As 9 exchanges **standalone** sem fixtures de resposta (aliases como `myokx`, `okxus`,
`gateeu`, `kucoineu`, `bybiteu`, `binancecoinm`, `binanceusdm` herdam do pai — sem ação):

`bitbank`, `bitflyer`, `btcbox`, `btcmarkets`, `btcturk`, `coinspot`, `fmfwio`, `mercado`, `paymium`, `zaif`

As 26 exchanges sem suporte WebSocket:

`bigone`, `bit2c`, `bitbank`, `bitbns`, `bitflyer`, `bitso`, `bitteam`, `btcbox`,
`btcmarkets`, `btcturk`, `coinmate`, `coinsph`, `coinspot`, `cryptomus`, `delta`,
`digifinex`, `fmfwio`, `foxbit`, `hibachi`, `indodax`, `latoken`, `mercado`, `paymium`,
`tokocrypto`, `zaif`, `zebpay`

---

## Roadmap priorizado

### Fase 0 — Sincronização & Fundação (imediato · 1–2 dias)

- [ ] **Sincronizar fork com upstream `v4.5.73`** — rebase/merge de `ccxt/ccxt@master`.
      Sem mudanças locais no código-fonte, o merge é limpo.
- [ ] **Provisionar toolchain de desenvolvimento** — `npm install` + dotnet SDK +
      Go 1.25 + PHP 8.1+ + Python (tox). Hoje o sandbox/CI local não roda lint/build/testes.
- [ ] **Automatizar sync periódico** — workflow `cron` semanal que abre PR de sync
      automático, evitando acúmulo de dívida de merge.

### Fase 1 — Cobertura de Testes (curto prazo · 2–4 semanas)

- [ ] **Fechar gap de fixtures estáticos (9 exchanges)** — gerar fixtures
      `request`/`response` via CLI (`node cli.js <ex> <method> --report` / `--response`)
      e validar nos 5 langs (`npm run request-tests && npm run response-tests`).
- [ ] **Zerar TODO/FIXME de maior risco** — priorizar `aster.ts` (preço de liquidação
      COIN-M ×2) e `bitflyer.ts` (produtos de 4 chars). Triage dos 36 restantes em lote.
- [ ] **Expandir fixtures de ordem nas certificadas** — `createOrder`/`cancelOrder`/
      `fetchOrder` para reduzir dependência de testes live com credenciais.

### Fase 2 — Cobertura WebSocket (médio prazo · 1–3 meses)

- [ ] **Adicionar Pro/WS às 26 exchanges só-REST** — priorizar por relevância:
      `digifinex`, `delta`, `latoken`, `bitso`, `indodax`, `tokocrypto`. Cada uma exige
      `ts/src/pro/<ex>.ts` + testes `watch*`.
- [ ] **Padronizar fixtures de WS** — capturar mensagens WS em fixtures para testes
      offline de `watchTicker`/`watchOrderBook` (hoje WS depende mais de live).

### Fase 3 — Expansão & Diferenciação (contínuo)

- [ ] **Amadurecer suporte a Rust** — `rust.yml` existe mas não transpila nem testa.
      Definir transpiler AST→Rust ou bindings para o core.
- [ ] **Consolidar a camada de IA do fork** — versionar/documentar as 15 skills Binance
      (`.agents/skills/`) e o `ccxt-pr-reviewer`. Criar skill "nova exchange" ponta-a-ponta.
- [ ] **Expandir prediction markets** — já há 5 (hyperliquid, kalshi, limitless, myriad,
      polymarket). Padronizar `PredictionExchange` e seguir o roadmap do upstream.

---

## Matriz impacto × esforço

| Fazer agora (alto impacto · baixo esforço) | Planejar (alto impacto · alto esforço) |
|---|---|
| Sync upstream 4.5.73 | Fixtures das 9 exchanges |
| Provisionar toolchain | Pro/WS nas 26 exchanges |
| Cron de sync automático | Transpiler Rust |

| Oportunista (baixo impacto · baixo esforço) | Avaliar antes (baixo impacto · alto esforço) |
|---|---|
| Triage dos 36 TODO/FIXME | Fixtures WS offline |
| Docs das skills de IA | Novas prediction markets |

---

## Como executar

**Regras de ouro (do `CLAUDE.md`):**
- Nunca editar arquivos gerados — só `ts/src/**`. Banner "PLEASE DO NOT EDIT" = achar o TS.
- TDD-first: teste/fixture antes do código.
- Verificar nos 5 langs: TS que compila pode quebrar no transpiler regex.
- Um PR por exchange — não empacotar mudanças.

**Loop de verificação:**
```bash
npm run tsBuild && npm run lint
npm run transpile              # Python + PHP
npm run transpileCS && npm run buildCS
npm run transpileGO && npm run buildGO
npm run request-tests && npm run response-tests
node run-tests <ex> --js       # live smoke
```

**Riscos a gerenciar:**
- Conflito de merge — o fork não toca código, então o sync é de baixo risco; skills locais podem divergir.
- Credenciais live — fixtures novas via `--report`/`--response` sem chaves; nunca commitar `keys.local.json`.
- Go 1.25 — exige toolchain recente em todos os ambientes de CI.

---

## KPIs de sucesso

| Métrica | Hoje | Meta | Prazo |
|---|---|---|---|
| Atraso vs upstream | 3 patches | ≤ 1 patch | Fase 0 (semanal, automatizado) |
| Exchanges standalone sem fixture | 9 | 0 | Fase 1 |
| TODO/FIXME em `ts/src` | 36 | < 20 | Fase 1 |
| Cobertura WebSocket | 75% | ≥ 85% | Fase 2 |
| Linguagens com CI completo | 6 (+Rust stub) | 7 | Fase 3 |
