# PM Hub — Shopee Challenge

Ferramenta interna single-file (HTML + localStorage) com gestão de projetos, time, tasks e roadmap. Este fork adiciona multi-usuário (L1/L2), correções de bugs e sync com GitHub para uso colaborativo.

## Estado atual

- [x] Baseline original importado
- [ ] Bugs de timezone corrigidos
- [ ] Multi-login (L1/L2) implementado
- [ ] GitHub sync funcionando
- [ ] Métricas extras no dashboard
- [ ] README com prompts documentados

## Como rodar localmente

Abra `index.html` no navegador. Tudo roda client-side.

## Como publicar (GitHub Pages)

1. Fork ou clone deste repo
2. Em `index.html`, configure no topo do `<script>`:
   ```js
   const GITHUB_REPO = 'SEU-USUARIO/pm-hub';
   const GITHUB_PAT  = 'github_pat_...';
   ```
3. Settings → Pages → Branch: `main` → Folder: `/` → Save
4. URL pública: `https://SEU-USUARIO.github.io/pm-hub`

## Segurança do PAT

Como o repo é público, o PAT fica visível no HTML. Mitigação:

- Use um **fine-grained PAT** com escopo mínimo: somente `Contents (read and write)` neste repo
- **Rotacione a cada 30 dias**
- Não habilite outros escopos — comprometer o PAT só permite editar `data.json` deste repo

## Prompts documentados

Veja [PROMPTS.md](PROMPTS.md) (em construção).

## Histórico de bugs corrigidos

Veja [BUGS.md](BUGS.md) (em construção).
