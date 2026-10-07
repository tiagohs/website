# Site dos aplicativos

Site estático hospedado no Cloudflare Pages. Cada produto é uma pasta; cada página é um `index.html` dentro de uma subpasta.

## Estrutura

```
/                                  → home (provisória)
/cinema-history/privacidade/       → política de privacidade do Cinema History
/cinema-history/termos/            → termos de uso do Cinema History
/rabisco-stickers/privacidade/     → política de privacidade do Rabisco Stickers
/whatss-stickers/privacidade/      → política de privacidade do AppFunStickers
/app-ads.txt                       → autorização do AdMob (vale para todos os apps da conta)
```

Para um produto novo: criar `/<produto>/index.html` e `/<produto>/privacidade/index.html`.

## Publicar no Cloudflare Pages

1. Crie um repositório no GitHub (ex.: `site`) e envie esta pasta para ele.
2. No painel da Cloudflare: **Workers & Pages → Create → Pages → Connect to Git**, e escolha o repositório.
3. Configuração do build:
   - Framework preset: **None**
   - Build command: *(vazio)*
   - Build output directory: `/`
4. Clique em **Save and Deploy**. O site fica em `https://<nome-do-projeto>.pages.dev`.
5. (Opcional) Domínio próprio: **Custom domains → Set up a custom domain**.

A cada `git push` no branch principal, a Cloudflare publica a nova versão automaticamente.

## Links para colocar nos apps e na Play Store

Troque `<base>` pelo domínio final (ex.: `https://meusite.pages.dev` ou `https://meusite.com.br`):

- Cinema History: `<base>/cinema-history/privacidade/`
- Rabisco Stickers: `<base>/rabisco-stickers/privacidade/`
- AppFunStickers: `<base>/whatss-stickers/privacidade/`
