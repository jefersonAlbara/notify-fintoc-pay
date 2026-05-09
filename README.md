# notify-fintoc-pay

Página estática que renderiza o widget [Fintoc](https://fintoc.com) usando o `widget_token` recebido do `gatewayCallback` da VTEX (Nike CL), permitindo que o cliente conclua o pagamento pelo método **Fintoc Mercado Pago** (`paymentSystem 898`) em qualquer browser.

Faz parte do projeto **Notify** ([notify-bot](https://github.com/jefersonAlbara/guadalupe-bot)). Hospedada em produção em:

```
https://notify-fintoc-pay.<workers-subdomain>.workers.dev/?wt=pi_xxx_sec_yyy&pk=APP_USR-...
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

## Estrutura

```
.
├── index.html   # widget + UI dark Notify (#33201e/#e8364f)
├── _headers     # security headers (Cloudflare Pages / Netlify)
├── README.md
└── LICENSE
```

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

## Deploy — Cloudflare Workers Static Assets

1. https://dash.cloudflare.com → **Workers & Pages** → **Create** → conectar o repo `notify-fintoc-pay`
2. Cloudflare auto-detecta o `index.html` na raiz e faz deploy
3. URL automática: `https://notify-fintoc-pay.<workers-subdomain>.workers.dev/`
4. (Opcional) Renomear o **workers subdomain** da conta para um nome mais curto: **Workers & Pages** → **Settings** (nível da conta) → **Subdomain**

Push em `main` aciona deploy automático.

## Deploy — alternativas

- **Netlify**: import repo → publish dir `/` → custom domain
- **Vercel**: import repo → output `/` → custom domain
- **GitHub Pages**: configura raiz como source

## Licença

MIT
