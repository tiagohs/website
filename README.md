# Site dos aplicativos

Site estático hospedado no Cloudflare Pages. Cada produto é uma pasta; cada página é um `index.html` dentro de uma subpasta.

## Estrutura

```
/                                  → home (provisória)
/cinema-history/privacidade/       → política de privacidade do Cinema History
/cinema-history/termos/            → termos de uso do Cinema History
/rabisco-stickers/privacidade/     → política de privacidade do Rabisco Stickers
/whatss-stickers/privacidade/      → política de privacidade do AppFunStickers
/metasfera/privacidade/            → política de privacidade do Metasfera (pt-BR e inglês)
/planosync/privacidade/            → política de privacidade do PlanoSync (pt-BR e inglês; inclui o Health Connect)
/planosync/termos/                 → termos de uso do PlanoSync (pt-BR e inglês)
/script-reader/privacidade/        → política de privacidade do Script Reader (pt-BR, inglês e espanhol)
/script-reader/termos/             → termos de uso do Script Reader (pt-BR, inglês e espanhol)
/script-reader/blocklist.json      → roteiros que o app esconde ({"version": 1, "slugs": [...]}, slugs do Script Slug)
/app-ads.txt                       → autorização do AdMob (vale para todos os apps da conta)
/cinema-history/media/             → arquivos que o app Cinema History carrega pela internet
                                     (images/: imagens das timelines, antes no Firebase Storage; channels/: avatares dos canais)
/cinema-history/content/           → conteúdo que o app atualiza sem nova versão (prêmios), com manifest.json;
                                     gerado por content-src/remote.py no repo do app
/cinema-history/link/              → links compartilhados do app (App Links): abre o app quando ele está instalado;
                                     sem o app, mostra "Abra no app Cinema History" com o botão da Google Play
/.well-known/assetlinks.json       → Digital Asset Links: autoriza os apps Android a abrir os links do site
/_headers                          → cache longo para /*/media/*, curto para /*/content/* e para o assetlinks.json
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
- Metasfera: `<base>/metasfera/privacidade/` (o app abre `https://website-cb5.pages.dev/metasfera/privacidade`)
- Script Reader: `<base>/script-reader/privacidade/` e `<base>/script-reader/termos/` (o app abre os dois em
  `https://website-cb5.pages.dev/script-reader/…` e lê `/script-reader/blocklist.json` a cada verificação do catálogo;
  para esconder um roteiro, ponha o slug dele em `slugs`)

## Links que abrem os apps (App Links)

O Cinema History abre sozinho os links `https://website-cb5.pages.dev/cinema-history/link/?screen=…` (os parâmetros
são os mesmos da versão 2.x do app). Só esse caminho abre o app: o resto do site (privacidade, termos…) continua no
navegador. Domínio e caminho ficam em `SharedLinks.HOST`/`PATH` no repositório do app; se o site ganhar domínio
próprio, mude lá, no filtro do `AndroidManifest.xml` e publique estes mesmos arquivos no domínio novo.

O Android só abre o app direto se o site autorizar, em `/.well-known/assetlinks.json` (a Cloudflare Pages publica a
pasta `.well-known` normalmente):

- `sha256_cert_fingerprints` do Cinema History: `A5:2D:25:…:36:DD`, o SHA-256 do "Certificado da chave de assinatura
  do app" na Play Console (*Teste e lançamento → Configuração → Integridade do app → Assinatura de apps*). É o mesmo
  certificado que assina os builds de release gerados no repositório do app, então vale para os dois. Se a Play um dia
  trocar a chave de assinatura (upgrade de chave), acrescente o novo SHA-256 na lista sem tirar o antigo.
- **Outro app**: acrescente outro objeto no array, com o `package_name` e os SHA-256 dele, e o filtro com
  `android:autoVerify="true"` e um `pathPrefix` próprio (ex.: `/rabisco-stickers/link`) no manifest do app:

  ```json
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "com.exemplo.outro_app",
      "sha256_cert_fingerprints": ["AA:BB:…"]
    }
  }
  ```
- Conferir depois de publicar: `https://website-cb5.pages.dev/.well-known/assetlinks.json` deve responder com
  `Content-Type: application/json`, e o
  [Statement List Tester](https://developers.google.com/digital-asset-links/tools/generator) deve aprovar o pacote.
  No aparelho (Android 12+): `adb shell pm get-app-links com.tiagohs.cinema_history`.

**iOS (depois, quando houver conta paga da Apple):** os Universal Links precisam de um
`/.well-known/apple-app-site-association` (JSON **sem** extensão, servido com `Content-Type: application/json` pelo
`_headers`) com o Team ID e o bundle id do app, limitado ao mesmo caminho:

```json
{ "applinks": { "details": [ { "appIDs": ["<TEAM_ID>.com.tiagohs.cinemahistory"],
  "components": [ { "/": "/cinema-history/link*" } ] } ] } }
```

e, no app, `applinks:website-cb5.pages.dev` nas *Associated Domains*.
