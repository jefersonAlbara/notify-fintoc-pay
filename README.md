# notify-fintoc-pay

Página estática que renderiza o widget [Fintoc](https://fintoc.com) usando o `widget_token` recebido do `gatewayCallback` da VTEX (Nike CL), permitindo que o cliente conclua o pagamento pelo método **Fintoc Mercado Pago** (`paymentSystem 898`) em qualquer browser.

Faz parte do projeto **Notify** ([notify-bot](https://github.com/jefersonAlbara/guadalupe-bot)). Hospedada como site estático para gerar links one-shot tipo:

```
https://pay.seudominio.com/?wt=pi_xxx_sec_yyy&pk=APP_USR-...
```

## Como funciona

1. O bot detecta `paymentMethod == "Fintoc Mercado Pago"` no `gatewayCallback`
2. Extrai do `appPayload`:
   - `externalReferenceId` → `widget_token` (formato `pi_xxx_sec_yyy`)
   - `publicKey` → public key do Mercado Pago
3. Bot monta o link e envia ao cliente
4. Cliente abre, vê monto + comércio + contagem regressiva, clica **Pagar**
5. Widget Fintoc abre como modal → escolhe banco → loga → confirma transferência
6. `onSuccess` mostra confirmação; VTEX recebe callback automaticamente via Mercado Pago

## Query params

| Param | Obrigatório | Default | Descrição |
|-------|-------------|---------|-----------|
| `wt`  | sim         | —       | `widget_token` do payment intent Fintoc |
| `pk`  | sim         | —       | `publicKey` do Mercado Pago |
| `ht`  | não         | `individual` | `holderType` (`individual` / `business`) |
| `pr`  | não         | `payments`   | Produto Fintoc |

## Segurança

- O `widget_token` é one-shot e expira em **10 minutos** (`expires_at` retornado por `widget_config`)
- Quem receber o link consegue tentar pagar, mas o pagamento sempre vai para o comerciante; não há vetor de exfil
- Sem credenciais hardcoded; tudo vem da query string
- Cookies do Fintoc ficam em `wizard.fintoc.com`, não no domínio próprio
- Headers de segurança em `_headers` (lido por Cloudflare Pages e Netlify)

## Desenvolvimento local

```bash
python -m http.server 8080
```

Abrir `http://localhost:8080/?wt=pi_test&pk=APP_USR-test`. O widget vai falhar (token falso), mas o layout/contagem/erros são exercitados.

## Deploy — Cloudflare Pages

1. https://dash.cloudflare.com → **Workers & Pages** → **Create** → **Pages** → **Connect to Git**
2. Selecionar o repo `notify-fintoc-pay`
3. **Build settings**:
   - Framework preset: **None**
   - Build command: *(vazio)*
   - Build output directory: `/` *(raiz)*
4. **Save and Deploy**
5. Em **Custom domains**, adiciona `pay.seudominio.com` → Cloudflare exibe um CNAME target
6. No registrar (Hostinger, Registro.br, etc.), cria um **CNAME** apontando o subdomínio para `<projeto>.pages.dev`
7. Aguarda 1-15 min — `https://pay.seudominio.com/` ativo com HTTPS

## Deploy — alternativas

- **Netlify**: import repo → publish dir `/` → custom domain
- **Vercel**: import repo → output `/` → custom domain
- **GitHub Pages**: configura raiz como source

## Estrutura

```
.
├── index.html   # widget + UI dark Notify (#33201e/#e8364f)
├── _headers     # security headers (Cloudflare Pages / Netlify)
└── README.md
```

## Licença

MIT
