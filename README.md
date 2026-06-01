# PM Hub — Shopee Challenge

Ferramenta interna single-file (HTML + CSS + JS inline) com gestão de projetos, time, tasks e roadmap. Fork com correções de bugs, multi-usuário (L1/L2/L3), login via Google e sync com GitHub.

**Live:** https://iagouccio-shopee-gif.github.io/pm-hub/

---

## Stack

| Camada | Tecnologia |
|--------|-----------|
| Frontend | HTML + CSS + JS inline (single file, zero dependências) |
| Persistência | `data.json` no GitHub via Contents API |
| Auth | Google Identity Services (GIS) |
| Hosting | GitHub Pages |
| Sync | GitHub Contents API + PAT fine-grained |

---

## Missão A — Bugs corrigidos

### Bug 1 — Timezone em `getQ()` e `getMK()`
**Problema:** `new Date("2025-11-27")` sem sufixo de hora é interpretado como UTC midnight. No fuso -3, isso vira `2025-11-26T21:00:00` — a data recua um dia nos agrupamentos de Roadmap e Heatmap.

**Prompt usado:**
> "Em `getQ()` e `getMK()`, a linha `new Date(ds)` está causando bug de timezone: datas como `2025-11-27` são interpretadas como UTC e recuam um dia no Brasil. Corrija apenas essas duas funções adicionando o sufixo `T00:00:00` para forçar interpretação local. Não altere nenhuma outra função."

**Fix:** `new Date(ds)` → `new Date(ds + 'T00:00:00')` em ambas as funções.

---

### Bug 2 — `today` sem zerar horas em `renderAdminDash()`
**Problema:** `const today = new Date()` carrega a hora atual. Às 15h, um projeto com `liveETA = hoje` aparece como overdue no painel de produtividade do L1.

**Prompt usado:**
> "Em `renderAdminDash()`, a variável `today` é criada com `new Date()` mas nunca tem as horas zeradas. Isso faz projetos com ETA hoje aparecerem como overdue durante o dia. Adicione `today.setHours(0,0,0,0)` logo após a criação da variável. Não altere mais nada nessa função."

**Fix:** `const today = new Date(); today.setHours(0,0,0,0);`

---

### Bug 3 — `today` sem zerar horas em `renderReports()`
**Problema:** Mesmo problema do Bug 2 na aba Reports.

**Prompt usado:**
> "Mesma correção do bug anterior, agora em `renderReports()`. Adicione `today.setHours(0,0,0,0)` após `const today = new Date()`. Escopo: apenas essa linha nessa função."

**Fix:** `const today = new Date(); today.setHours(0,0,0,0);`

---

### Bug 4 — Data recua um dia ao abrir modal de edição (`toISO`)
**Problema:** Dentro de `openModal()`, a função `toISO` usa `new Date(d)` sem sufixo, convertendo a data para UTC. No fuso -3, `2025-11-27` vira `2025-11-26` no campo de edição.

**Prompt usado:**
> "Em `openModal()`, a função inline `toISO` converte datas para o campo `input[type=date]`. Ela usa `new Date(d)` sem sufixo de hora, causando o mesmo bug de timezone já corrigido em `getQ`. Corrija para usar `new Date(d + 'T00:00:00')` e retorne `d.slice(0, 10)` diretamente. Altere apenas essa função inline."

**Fix:** `const toISO = d => { const x = new Date(d + 'T00:00:00'); return isNaN(x) ? '' : d.slice(0, 10); }`

---

## Missão B — Multi-usuário

### Arquitetura de níveis

| Badge | Nível | Quem | Acesso |
|-------|-------|------|--------|
| 🔴 L1 | Director/GPM | Matheus Mendes | Admin total — todos os projetos, aba Users, dashboard completo |
| 🟠 L2 | Team Leader | Danilo L., Matheus T., Leandra, Lucas | Vê tudo, edita apenas próprios projetos |
| 🔵 L3 | PM | Demais 9 membros | Vê tudo, edita apenas próprios projetos |
| — | Visitante | Qualquer email fora do time | Somente leitura automático |

### O que foi implementado

**Login via Google (GIS)**
- Botão "Fazer Login com o Google" — único método de entrada
- Email não cadastrado entra automaticamente como Visitante (read-only)
- Sem senha, sem dropdown de usuário

**Controle de acesso**
- `vp()` — todos os níveis veem todos os projetos
- `canEdit(p)` — L1 edita tudo; L2/L3 apenas `p.pm === session.pm`; visitante nunca edita
- `isReadOnly()` — visitante ou usuário sem PM identifier
- PM selector travado no modal para não-L1

**Dashboard e Tasks filtrados**
- L2/L3 veem métricas apenas do próprio time no Dashboard
- Tasks filtradas por `owner === session.pm` para L2/L3
- Botão "👤 Meus Projetos" na barra de filtros (só para L2/L3)

**Resistência a localStorage stale**
- `autoSeedUsers()` — sempre roda no init, adiciona membros faltantes do team[]
- `ensureAdminLevel()` — promove admin para L1 e corrige seeds incorretos para L3

### Time configurado

| Nome | Nível | Email |
|------|-------|-------|
| Matheus Mendes | L1 | matheus.mendes@shopee.com |
| Danilo L. | L2 | danilo.leite@shopee.com |
| Leandra | L2 | leandra.mieko@shopee.com |
| Lucas | L2 | lucas.menegaldo@shopee.com |
| Matheus T. | L2 | matheus.torresi@shopee.com |
| Iago | L3 | iago.uccio@shopee.com |
| Erika | L3 | erika.izawa@shopee.com |
| Mauricio L. | L3 | mauricio.lambert@shopee.com |
| Ricardo R | L3 | ricardo.rocha@shopee.com |
| Tatiane Porto | L3 | tatiane.porto@shopee.com |
| Vithoria | L3 | vithoria.vasconcelos@shopee.com |
| William | L3 | william.azevedo@shopee.com |
| Mateus R. | L3 | mateus.alvarenga@shopee.com |
| Laecio Rodrigues | L3 | laecio.rodrigues@shopee.com |

---

## Missão C — Deploy e infraestrutura

### Hosting
- **Plataforma:** GitHub Pages (gratuito, deploy automático a cada `git push`)
- **URL:** https://iagouccio-shopee-gif.github.io/pm-hub/

### Banco de dados
- `data.json` no repositório é o banco de dados (projetos, tasks, team, users)
- Leitura e escrita via GitHub Contents API
- Indicador no header: ⚪ sem PAT · 🟡 salvando · 🟢 sincronizado · 🔴 erro

### GitHub Sync
- PAT armazenado no `localStorage` — nunca commitado no código
- Debounce de 2s — evita múltiplas requisições em edições rápidas
- Retry automático em conflito (HTTP 409/422)
- PAT fine-grained: permissão `Contents: Read and write` neste repo, 30 dias

### Autenticação Google
- Google Identity Services (GIS) — client-side, sem backend
- Client ID configurado para `iagouccio-shopee-gif.github.io`
- Email fora do time → sessão Visitante automática

---

## Como rodar localmente

Abra `index.html` no navegador. Tudo roda client-side, sem dependências externas.
Para o sync funcionar, configure o PAT clicando no indicador ⚪ no header.
