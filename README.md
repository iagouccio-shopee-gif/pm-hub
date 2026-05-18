# PM Hub — Shopee Challenge

Ferramenta interna single-file (HTML + localStorage) com gestão de projetos, time, tasks e roadmap. Este fork adiciona multi-usuário (L1/L2), correções de bugs e sync com GitHub para uso colaborativo.

**Live:** https://iagouccio-shopee-gif.github.io/pm-hub/

---

## O que foi feito

- [x] Baseline original importado
- [x] Bugs de timezone corrigidos (4 funções)
- [x] Multi-login L1/L2 implementado
- [x] GitHub sync funcionando (Contents API + PAT no localStorage)
- [x] README com prompts documentados

---

## Missão A — Bugs corrigidos

### Bug 1 — Timezone em `getQ()` e `getMK()`
**Problema:** `new Date("2025-11-27")` sem sufixo de hora é interpretado como UTC midnight. No fuso -3, isso vira `2025-11-26T21:00:00` — a data recua um dia nos agrupamentos de Roadmap e Heatmap.

**Prompt usado:**
> "Em `getQ()` e `getMK()`, a linha `new Date(ds)` está causando bug de timezone: datas como `2025-11-27` são interpretadas como UTC e recuam um dia no Brasil. Corrija apenas essas duas funções adicionando o sufixo `T00:00:00` para forçar interpretação local. Não altere nenhuma outra função."

**Fix:** `new Date(ds)` → `new Date(ds + 'T00:00:00')` em ambas as funções.

---

### Bug 2 — `today` sem zerar horas em `renderAdminDash()`
**Problema:** `const today = new Date()` carrega a hora atual. Às 15h, um projeto com `liveETA = hoje` aparece como overdue no painel de produtividade do L1, pois `T00:00:00 < 15:00:00`.

**Prompt usado:**
> "Em `renderAdminDash()`, a variável `today` é criada com `new Date()` mas nunca tem as horas zeradas. Isso faz projetos com ETA hoje aparecerem como overdue durante o dia. Adicione `today.setHours(0,0,0,0)` logo após a criação da variável. Não altere mais nada nessa função."

**Fix:** `const today = new Date(); today.setHours(0,0,0,0);`

---

### Bug 3 — `today` sem zerar horas em `renderReports()`
**Problema:** Mesmo problema do Bug 2, mas na aba Reports — a tabela "Overdue Projects" marca projetos com ETA hoje como atrasados durante o dia.

**Prompt usado:**
> "Mesma correção do bug anterior, agora em `renderReports()`. Adicione `today.setHours(0,0,0,0)` após `const today = new Date()`. Escopo: apenas essa linha nessa função."

**Fix:** `const today = new Date(); today.setHours(0,0,0,0);`

---

### Bug 4 — Data recua um dia ao abrir modal de edição (`toISO`)
**Problema:** Dentro de `openModal()`, a função `toISO` usa `new Date(d)` sem sufixo, convertendo a data para UTC antes de retornar o ISO string. No fuso -3, `2025-11-27` vira `2025-11-26` no campo de edição.

**Prompt usado:**
> "Em `openModal()`, a função inline `toISO` converte datas para o campo `input[type=date]`. Ela usa `new Date(d)` sem sufixo de hora, causando o mesmo bug de timezone já corrigido em `getQ`. Corrija para usar `new Date(d + 'T00:00:00')` e retorne `d.slice(0, 10)` diretamente em vez de passar por `.toISOString()`. Altere apenas essa função inline."

**Fix:** `const toISO = d => { const x = new Date(d + 'T00:00:00'); return isNaN(x) ? '' : d.slice(0, 10); }`

---

## Missão B — Multi-usuário

### Arquitetura

Dois níveis de acesso controlados pelo campo `level` no objeto de usuário:

| Nível | Quem é | Login | Acesso |
|-------|--------|-------|--------|
| **L1** | GPM / Diretor | Senha obrigatória | Tudo — projetos, tasks, painel admin, aba Users |
| **L2** | Team Leader | Sem senha | Vê todos os projetos e tasks; edita apenas os próprios |
| **L2 sem pm** | Visitante / Observer | Sem senha | Read-only completo — vê tudo, edita nada |

O campo **PM Identifier** (configurável na aba Users pelo L1) é o vínculo entre o usuário e os projetos: um L2 com `pm = "Erika"` edita apenas projetos onde `p.pm === "Erika"`.

### Dashboard filtrado para L2

L2 vê no Dashboard as métricas do **próprio time** (Total, Live, Ongoing, ETAs). L1 vê o consolidado de todos os times.

### Botão "Meus Projetos"

L2 tem um botão de atalho na barra de filtros que aplica `customFilter = p => p.pm === session.pm` em um clique — sem precisar navegar pelo filtro de PM manualmente.

### Prompts usados

**Estrutura de usuários e login:**
> "Adicione um sistema de login ao PM Hub com dois níveis: L1 (GPM/Diretor, senha obrigatória, acesso total) e L2 (Team Leader, sem senha, acesso restrito). Crie a tela de login com dropdown de usuários. L2 entra direto ao selecionar o nome. L1 exige senha. Na primeira vez que L1 seleciona o nome, ele define a própria senha. Use SHA-256 via `crypto.subtle` para hash. Não altere `saveP()`, `saveT()`, `saveTasks()`, `uid()` nem nenhuma função de render existente."

**Filtro de escopo L2:**
> "Implemente a função `vp()` que retorna os projetos visíveis para o usuário logado: L1 retorna todos; L2 retorna apenas os projetos onde `p.pm === session.pm`. Crie também `canEdit(p)` e `isReadOnly()`. Use essas funções em `renderProjects()`, `renderDashboard()` e `renderTasks()`. Não modifique a lógica de salvamento."

**Dashboard filtrado:**
> "Em `renderDashboard()`, adicione uma constante `scope` no início da função: se o usuário for L2 com pm definido, filtra `projects` pelo pm dele; caso contrário usa `vp()`. Substitua os usos de `vp()` dentro dessa função por `scope`. Não altere `renderHeatmap()` nem `renderAdminDash()`."

**Botão Meus Projetos:**
> "Em `buildFilters()`, adicione após o botão Clear um botão '👤 Meus Projetos' que só aparece para usuários L2 com pm definido. Ao clicar, aplica `customFilter = p => p.pm === session.pm` e chama `renderProjects()`. Crie a função `filterMyProjects()` para isso."

**Auto-seed de usuários:**
> "Crie `autoSeedUsers()` que, ao detectar `users[]` vazio, itera sobre `team[]` interno e cria um usuário L2 para cada membro. O primeiro membro com `level === 1` no time recebe nível L1 (sem senha, será definida no primeiro login). Chame essa função no `init()` após carregar dados remotos."

---

## Missão C — Como publicar

Veja [PUBLICAR.md](PUBLICAR.md) para o passo a passo completo com comparação de plataformas.

---

## GitHub Sync

O sync usa a [GitHub Contents API](https://docs.github.com/en/rest/repos/contents) para ler e gravar `data.json` no repositório:

- **PAT armazenado no `localStorage`** — nunca commitado no código
- **Debounce de 2s** — evita múltiplas requisições em edições rápidas
- **Retry automático em conflito** — se dois usuários salvarem ao mesmo tempo (HTTP 409/422), o app busca o SHA atualizado e tenta novamente
- **Indicador no header** — ⚪ não configurado · 🟡 salvando · 🟢 sincronizado · 🔴 erro

Para configurar: clique no indicador ⚪ no header e cole o PAT fine-grained com permissão `Contents: Read and write` neste repositório.

---

## Como rodar localmente

Abra `index.html` no navegador. Tudo roda client-side, sem dependências externas.
