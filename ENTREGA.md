# PM Hub — Documento de Entrega Completo

**Projeto:** Shopee PM Hub Challenge  
**Repositório:** https://github.com/iagouccio-shopee-gif/pm-hub  
**Live:** https://iagouccio-shopee-gif.github.io/pm-hub/

---

## O que é o projeto

Ferramenta interna de gestão de portfólio de produtos da Shopee Brasil. Single-file app (um único `index.html` com HTML, CSS e JS inline) que roda inteiramente no browser — sem servidor, sem framework, sem build step.

---

## Arquitetura

```
Browser
  └── index.html (toda a lógica: ~1.700 linhas)
        ├── localStorage  → cache local (sessão, dados offline)
        └── GitHub Contents API → data.json (fonte da verdade)

GitHub Pages → serve o index.html estaticamente
Google Identity Services → autenticação via conta corporativa
```

| Camada | Tecnologia | Detalhe |
|--------|-----------|---------|
| Frontend | HTML + CSS + JS | Single file, zero dependências npm |
| Banco de dados | `data.json` no GitHub | Lido e gravado via Contents API |
| Auth | Google Identity Services (GIS) | Client-side, sem backend |
| Hosting | GitHub Pages | Grátis, deploy automático |
| Sync | GitHub Contents API + PAT | Debounce 2s, retry em conflito |

---

## Missão A — Correção de Bugs

### Contexto técnico
Todos os 4 bugs têm a mesma raiz: tratamento incorreto de datas no JavaScript.  
`new Date("YYYY-MM-DD")` sem sufixo de hora é interpretado como **UTC midnight**.  
No fuso horário de Brasília (UTC-3), isso faz a data recuar para o dia anterior.

---

### Bug 1 — Timezone em `getQ()` e `getMK()`

**Impacto:** Datas no Roadmap e Heatmap apareciam no trimestre/mês errado.  
Ex: `2025-11-27` aparecia em outubro no heatmap (dia 26 → mês 10).

**Causa:**
```js
// Antes — interpretado como UTC, recua 3h no Brasil
const d = new Date(ds);
```

**Fix:**
```js
// Depois — força interpretação local
const d = new Date(ds + 'T00:00:00');
```

**Prompt usado:**
> "Em `getQ()` e `getMK()`, a linha `new Date(ds)` está causando bug de timezone: datas como `2025-11-27` são interpretadas como UTC e recuam um dia no Brasil. Corrija apenas essas duas funções adicionando o sufixo `T00:00:00` para forçar interpretação local. Não altere nenhuma outra função."

---

### Bug 2 — `today` sem zerar horas em `renderAdminDash()`

**Impacto:** Projetos com Live ETA = hoje apareciam como **overdue** no painel do L1 durante o dia inteiro.  
Ex: às 14h, um projeto com ETA `2026-05-19` era marcado como atrasado porque `14:00 > 00:00`.

**Causa:**
```js
const today = new Date(); // carrega hora atual — 14:00:00
// comparação: p.liveETA (00:00) < today (14:00) → overdue incorreto
```

**Fix:**
```js
const today = new Date();
today.setHours(0, 0, 0, 0); // zera para meia-noite
```

**Prompt usado:**
> "Em `renderAdminDash()`, a variável `today` é criada com `new Date()` mas nunca tem as horas zeradas. Isso faz projetos com ETA hoje aparecerem como overdue durante o dia. Adicione `today.setHours(0,0,0,0)` logo após a criação da variável. Não altere mais nada nessa função."

---

### Bug 3 — `today` sem zerar horas em `renderReports()`

**Impacto:** Mesmo problema do Bug 2, mas na aba Reports — tabela "Overdue Projects" marcava projetos incorretamente.

**Fix:** Idêntico ao Bug 2.

**Prompt usado:**
> "Mesma correção do bug anterior, agora em `renderReports()`. Adicione `today.setHours(0,0,0,0)` após `const today = new Date()`. Escopo: apenas essa linha nessa função."

---

### Bug 4 — Data recua um dia ao abrir modal de edição

**Impacto:** Ao clicar em "Edit" em qualquer projeto, a data do campo Live ETA aparecia com 1 dia a menos.  
Ex: projeto salvo com `2025-11-27` abria o modal mostrando `2025-11-26`.

**Causa:**
```js
// toISO dentro de openModal()
const toISO = d => {
  const x = new Date(d); // UTC bug — recua 1 dia
  return isNaN(x) ? '' : x.toISOString().slice(0, 10);
};
```

**Fix:**
```js
const toISO = d => {
  const x = new Date(d + 'T00:00:00'); // força local
  return isNaN(x) ? '' : d.slice(0, 10); // retorna original, sem converter
};
```

**Prompt usado:**
> "Em `openModal()`, a função inline `toISO` converte datas para o campo `input[type=date]`. Ela usa `new Date(d)` sem sufixo de hora, causando o mesmo bug de timezone já corrigido em `getQ`. Corrija para usar `new Date(d + 'T00:00:00')` e retorne `d.slice(0, 10)` diretamente em vez de passar por `.toISOString()`. Altere apenas essa função inline."

---

## Missão B — Multi-usuário

### Arquitetura de níveis de acesso

| Badge | Nível | Perfil | Acesso |
|-------|-------|--------|--------|
| 🔴 L1 | Director/GPM | Matheus Mendes | Admin total: todos os projetos, aba Users, dashboard consolidado, pode editar qualquer projeto |
| 🟠 L2 | Team Leader | Danilo, Matheus T., Leandra, Lucas | Vê todos os 210 projetos; edita apenas os próprios (`p.pm === session.pm`); dashboard filtrado pelo time |
| 🔵 L3 | PM | Demais 9 membros | Mesmo comportamento do L2 |
| — | Visitante | Email fora do time | Read-only completo; acesso automático |

### Funções-chave implementadas

```js
// Visibilidade — todos veem tudo
function vp() { return projects; }

// Permissão de edição
function canEdit(p) {
  if (!session) return false;
  if (session.level === 'L1') return true;           // admin edita tudo
  if (session.level === 'visitante') return false;   // visitante não edita nada
  if (!session.pm) return false;                     // sem PM identifier = read-only
  return !p || p.pm === session.pm;                  // edita apenas próprios
}

// Read-only
function isReadOnly() {
  return !!session && (
    session.level === 'visitante' ||
    (!session.pm && session.level !== 'L1')
  );
}
```

### Fluxo de login

```
Usuário clica "Fazer Login com o Google"
    ↓
Google retorna JWT com email
    ↓
App decodifica JWT → extrai email
    ↓
Busca users[].email === email
    ├── Encontrou → cria sessão com level/pm do usuário
    └── Não encontrou → sessão Visitante (read-only)
```

### Funcionalidades por nível

**L1 (Matheus Mendes)**
- Dashboard consolidado com todos os 210 projetos
- Painel de produtividade (renderAdminDash)
- Aba Users — pode adicionar, editar, remover usuários e vincular emails Google
- Campo PM livre no modal (pode reatribuir projetos)
- Badge 🔴 L1 no header

**L2/L3 (Team Leaders e PMs)**
- Dashboard filtrado por `pm === session.pm` — vê só suas métricas
- Botão "👤 Meus Projetos" na barra de filtros
- Tasks filtradas por `owner === session.pm`
- Campo PM travado no modal de edição (não pode mudar dono do projeto)
- Badge 🟠 L2 ou 🔵 L3 no header

**Visitante**
- Banner amarelo "Somente leitura"
- Sem botões de edição, sem modal de criação
- Acesso automático para emails não cadastrados

### Resistência a localStorage stale

Problema real encontrado em produção: usuários abrindo o site com dados antigos no localStorage (seeds com níveis invertidos, apenas 1 usuário cadastrado, etc).

**Solução:** dois mecanismos de auto-correção no `init()`:

```js
// 1. Sempre adiciona membros do team[] que faltam em users[]
autoSeedUsers(); // guarda por nome, skipa existentes

// 2. Garante que o admin está como L1 e corrige seeds errados
(function ensureAdminLevel() {
  // Promove admin para L1 se necessário
  // Demove L2-sem-senha para L3 (seeds antigos)
  // Roda independente do estado atual
})();
```

### Time configurado

| Nome | Nível | Email Google |
|------|-------|-------------|
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

## Missão C — Deploy e Infraestrutura

### Hosting — GitHub Pages

**Por que GitHub Pages:**
- Gratuito para repositórios públicos
- Deploy automático a cada `git push` (~1 minuto)
- HTTPS nativo
- Sem configuração de servidor

**Como foi configurado:**
1. Repositório criado em github.com/iagouccio-shopee-gif/pm-hub (público)
2. Settings → Pages → Source: `Deploy from branch` → `main` / `/ (root)`
3. URL gerada: https://iagouccio-shopee-gif.github.io/pm-hub/

---

### Banco de dados — `data.json` no GitHub

O arquivo `data.json` na raiz do repositório é o banco de dados da aplicação.

**Estrutura:**
```json
{
  "projects": [...],   // 210 projetos
  "team": [...],       // membros do time
  "tasks": [...],      // tasks vinculadas a projetos
  "users": [...]       // usuários com level e email
}
```

**Ciclo de leitura/escrita:**
```
Usuário edita → debounce 2s → GitHub Contents API (PUT)
    → data.json atualizado no repo
    → próximo usuário que abrir o app carrega a versão nova (GET)
```

**Conflito simultâneo (HTTP 409/422):**
```
Dois usuários salvam ao mesmo tempo
    → App detecta conflito
    → Busca SHA atualizado automaticamente
    → Tenta novamente com o SHA correto
```

---

### Autenticação Google (GIS)

**Por que GIS e não OAuth tradicional:**
- Funciona client-side — sem backend necessário
- Não expõe Client Secret (só o Client ID fica no HTML)
- Suporta login automático com conta corporativa

**Configuração no Google Cloud Console:**
1. Projeto criado: `PM Hub Shopee`
2. OAuth consent screen: tipo Internal (só contas Shopee)
3. Credencial: Web application
4. Authorized JavaScript origins: `https://iagouccio-shopee-gif.github.io`
5. Client ID: `81953791376-p77tmmq9ojboo9cephtoqnl3pgptj2b1.apps.googleusercontent.com`

**Implementação:**
```js
// Inicialização
google.accounts.id.initialize({
  client_id: GOOGLE_CLIENT_ID,
  callback: handleGoogleCredential
});

// Handler — decodifica JWT e faz login
function handleGoogleCredential(resp) {
  const payload = JSON.parse(atob(resp.credential.split('.')[1]));
  const email = payload.email;
  const u = users.find(u => u.email === email);
  
  if (!u) {
    // Email não cadastrado → Visitante automático
    session = { level: 'visitante', ... };
  } else {
    // Usuário encontrado → login com nível correto
    session = { level: u.level, pm: u.pm, ... };
  }
}
```

---

### GitHub Sync — PAT fine-grained

**Segurança:**
- PAT armazenado apenas no `localStorage` de cada dispositivo
- Nunca commitado no código
- Fine-grained: permissão `Contents: Read and write` apenas neste repositório
- Expiração: 30 dias com rotação obrigatória

**Indicador visual no header:**
- ⚪ PAT não configurado (dados ficam apenas no localStorage local)
- 🟡 Salvando...
- 🟢 Sincronizado
- 🔴 Erro (rede, PAT expirado, etc.)

---

## Resultado do Teste E2E

| Cenário | Resultado |
|---------|-----------|
| Tela de login — só botão Google | ✅ |
| Visitante — read-only, sem edição | ✅ |
| L3 (Iago) — edita próprios, não edita outros | ✅ |
| L2 (Danilo) — PM field travado no modal | ✅ |
| L1 (Matheus) — acesso total, aba Users | ✅ |
| Bug #1 — timezone getQ/getMK | ✅ |
| Bug #2 — today zerado no adminDash | ✅ |
| Bug #3 — today zerado no Reports | ✅ |
| Bug #4 — data sem desvio no modal | ✅ |
| Meus Projetos — filtra corretamente | ✅ |
| Sync indicator — ⚪ sem PAT | ✅ |
| Migration localStorage stale | ✅ |

---

## Como adicionar um novo usuário

1. Matheus (L1) acessa o app
2. Aba **Users** → **Add User**
3. Preenche: Nome, Email Google, PM Identifier, Nível
4. Salva — o usuário já consegue entrar pelo Google na próxima vez

## Como rodar localmente

Abra `index.html` diretamente no navegador. Tudo roda client-side.  
Para sync entre dispositivos, clique em ⚪ no header e configure o PAT.
