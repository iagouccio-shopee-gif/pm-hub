# Passos para publicar — execução manual

A pasta `pm-hub-challenge/` já está inicializada como repo git local (1 commit: `chore: import PM Hub baseline`). Falta criar o repo no GitHub e empurrar.

---

## 1. Criar repo no GitHub (UI ou gh CLI)

### Opção A — pelo navegador (mais simples)
1. Abra https://github.com/new
2. Nome: `pm-hub` (ou outro de sua preferência)
3. Visibilidade: **Public**
4. **NÃO** marque "Add a README" / "Add .gitignore" / "license" — o repo precisa estar vazio
5. Create repository

### Opção B — via gh CLI (se tiver autenticado)
```bash
cd ~/Downloads/pm-hub-challenge   # ou onde você baixar essa pasta
gh repo create pm-hub --public --source=. --remote=origin
```

---

## 2. Empurrar o commit local

Substitua `SEU-USUARIO` pelo seu username do GitHub:

```bash
cd ~/Downloads/pm-hub-challenge   # ajuste o caminho conforme onde colocou a pasta
git remote add origin https://github.com/SEU-USUARIO/pm-hub.git
git branch -M main
git push -u origin main
```

---

## 3. Ativar GitHub Pages

1. No repo, vá em **Settings → Pages**
2. Source: **Deploy from a branch**
3. Branch: `main` / Folder: `/ (root)` → Save
4. Espere 1–2 minutos
5. URL pública: `https://SEU-USUARIO.github.io/pm-hub/`

Teste abrindo essa URL — você deve ver o PM Hub funcionando.

---

## 4. Gerar PAT fine-grained (escopo mínimo)

1. Acesse https://github.com/settings/personal-access-tokens/new
2. **Token name**: `pm-hub-sync`
3. **Expiration**: 30 dias (rotação obrigatória)
4. **Repository access**: Only select repositories → escolha `pm-hub`
5. **Permissions → Repository permissions**:
   - `Contents`: **Read and write**
   - Deixe todo o resto em `No access`
6. Generate token
7. **Copie o token** (ele só aparece uma vez) — começa com `github_pat_`

Guarde esse token. Vamos colá-lo no HTML quando implementar o sync (passo 4 do plano).

---

## 5. Confirmar que está tudo no ar

Checklist:

- [ ] Repo `pm-hub` criado e público
- [ ] `git push` funcionou (todos os 4 arquivos visíveis na UI do GitHub)
- [ ] Pages ativo e URL `https://SEU-USUARIO.github.io/pm-hub/` carrega o app
- [ ] PAT `pm-hub-sync` gerado e guardado em local seguro
- [ ] Data de expiração do PAT anotada na agenda

Quando esses 5 itens estiverem ✅, me avisa que partimos pro **passo 2: corrigir os 4 bugs de timezone**.
