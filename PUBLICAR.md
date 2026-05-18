# Missão C — Como publicar o PM Hub

O PM Hub é um **single-file app** (HTML + CSS + JS inline, sem dependências externas). Qualquer plataforma que sirva arquivos estáticos funciona — todas as opções abaixo são gratuitas para este caso de uso.

---

## Comparativo de plataformas

| Plataforma | Custo | Deploy | URL customizada | HTTPS | Melhor para |
|------------|-------|--------|-----------------|-------|-------------|
| **GitHub Pages** | Grátis | Push no `main` | Com domínio próprio | ✅ Automático | Quem já usa GitHub |
| **Netlify** | Grátis | Drag & drop ou Git | Subdomínio grátis + custom | ✅ Automático | Deploy mais rápido, sem CLI |
| **Vercel** | Grátis | Push no Git | Subdomínio grátis + custom | ✅ Automático | Quem prefere painel mais moderno |

**Recomendação:** GitHub Pages — já está configurado, custo zero, deploy automático a cada `git push`.

---

## Opção 1 — GitHub Pages (recomendado, já em uso)

**URL atual:** https://iagouccio-shopee-gif.github.io/pm-hub/

### Passo a passo (primeira vez)

1. Crie o repositório em https://github.com/new
   - Visibilidade: **Public**
   - Não marque "Add README" — deixe vazio

2. Suba os arquivos:
   ```bash
   git init
   git add index.html data.json README.md PUBLICAR.md
   git commit -m "chore: initial commit"
   git remote add origin https://github.com/SEU-USUARIO/pm-hub.git
   git branch -M main
   git push -u origin main
   ```

3. Ative o Pages:
   - Repo → **Settings → Pages**
   - Source: `Deploy from a branch`
   - Branch: `main` / Folder: `/ (root)` → **Save**
   - Aguarde ~2 minutos
   - URL: `https://SEU-USUARIO.github.io/pm-hub/`

4. Configure o sync (opcional mas recomendado):
   - Gere um PAT em https://github.com/settings/personal-access-tokens/new
   - Nome: `pm-hub-sync` · Expiração: 30 dias
   - Permissões: `Contents → Read and write` (apenas neste repo)
   - No app, clique em ⚪ no header e cole o token

### Atualizações futuras
Qualquer `git push` atualiza o site automaticamente em ~1 minuto.

---

## Opção 2 — Netlify (drag & drop, sem CLI)

**Custo:** Grátis para sites estáticos

1. Acesse https://app.netlify.com e crie uma conta (pode usar login com GitHub)
2. Na tela inicial, arraste a **pasta `pm-hub/`** para a área de drop
3. O Netlify faz o deploy em segundos
4. URL gerada automaticamente: `https://nome-aleatorio.netlify.app`
5. Para URL customizada: Site settings → Domain management → Add custom domain

**Vantagem:** Não precisa de Git nem CLI. Ideal para quem quer subir rápido sem configuração.

**Para atualizar:** Arraste a pasta novamente ou conecte ao repositório GitHub em Site settings → Build & deploy → Link repository.

---

## Opção 3 — Vercel

**Custo:** Grátis para projetos pessoais

1. Acesse https://vercel.com e faça login com GitHub
2. Clique em **"Add New Project"** → importe o repositório `pm-hub`
3. Configurações de build: deixe tudo em branco (é HTML estático)
4. Clique em **Deploy**
5. URL gerada: `https://pm-hub-SEU-USUARIO.vercel.app`

**Vantagem:** Interface moderna, preview automático em cada Pull Request.

---

## Segurança do PAT

O repositório é público — o `data.json` é visível para qualquer pessoa. O PAT **nunca deve ser commitado** no código. Ele fica armazenado apenas no `localStorage` de cada dispositivo.

Boas práticas adotadas neste projeto:

- PAT **fine-grained** com escopo mínimo: somente `Contents (read and write)` neste repositório
- Expiração de **30 dias** com rotação obrigatória
- Se o PAT for comprometido, o dano máximo é alteração do `data.json` deste repo — nenhum outro acesso à conta GitHub

---

## Checklist de publicação

- [ ] Repositório criado e público no GitHub
- [ ] `git push` realizado com sucesso
- [ ] GitHub Pages ativo — URL carregando o app
- [ ] PAT `pm-hub-sync` gerado com escopo mínimo
- [ ] PAT configurado no app (clique no ⚪ no header)
- [ ] Data de expiração do PAT anotada para rotação
- [ ] Primeiro login como L1 realizado e senha definida
