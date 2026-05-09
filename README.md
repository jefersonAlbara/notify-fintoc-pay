# notify-fintoc-pay

Página estática que renderiza o widget [Fintoc](https://fintoc.com) usando o `widget_token` recebido do `gatewayCallback` da VTEX (Nike CL), permitindo que o cliente conclua o pagamento pelo método **Fintoc Mercado Pago** (`paymentSystem 898`) em qualquer browser.

Faz parte do projeto **Notify** ([notify-bot](https://github.com/jefersonAlbara/guadalupe-bot)). Hospedada em produção em:

```
https://notifybot.com.br/fintoc/payment/?wt=pi_xxx_sec_yyy&pk=APP_USR-...
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
├── fintoc/
│   └── payment/
│       └── index.html   # widget + UI dark Notify (#33201e/#e8364f)
├── _headers             # security headers (Cloudflare Pages / Netlify)
├── README.md
└── LICENSE
```

A pasta `fintoc/payment/` reflete o path final servido em produção.

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

Abrir `http://localhost:8080/fintoc/payment/?wt=pi_test&pk=APP_USR-test`. O widget vai falhar (token falso), mas o layout/contagem/erros são exercitados.

## Deploy — Cloudflare Pages (apex `notifybot.com.br`)

### 1. Conectar repo
1. https://dash.cloudflare.com → **Workers & Pages** → **Create** → **Pages** → **Connect to Git**
2. Selecionar o repo `notify-fintoc-pay`
3. **Build settings**:
   - Framework preset: **None**
   - Build command: *(vazio)*
   - Build output directory: `/`
4. **Save and Deploy**

### 2. Migrar DNS para Cloudflare (necessário para apex)
Para servir o conteúdo no apex `notifybot.com.br/fintoc/payment`, o DNS precisa estar no Cloudflare (Hostinger não suporta CNAME flattening no apex):

1. Cloudflare → **Websites** → **Add site** → digitar `notifybot.com.br` → plano Free
2. Cloudflare lista os DNS records atuais — confirma e dá 2 nameservers (ex: `xyz.ns.cloudflare.com`)
3. Hostinger → **Domínios** → `notifybot.com.br` → **Nameservers** → trocar para os do Cloudflare
4. Aguarda propagação (1-24h, normalmente <1h)
5. Cloudflare detecta automaticamente e ativa o site

### 3. Vincular domínio ao Pages
1. Volta no projeto Pages → **Custom domains** → **Set up a custom domain**
2. Digita `notifybot.com.br` (apex)
3. Cloudflare cria automaticamente os DNS records (CNAME flattening interno)
4. Aguarda status "Active" (poucos minutos)
5. URL final: `https://notifybot.com.br/fintoc/payment/`

## Atalho — usar subdomínio (sem migração de DNS)

Se preferir não migrar nameservers, use `pay.notifybot.com.br/fintoc/payment` em vez do apex:

1. Pages → Custom domains → adicionar `pay.notifybot.com.br` → copia o CNAME target
2. Hostinger → DNS → criar CNAME `pay` → `<projeto>.pages.dev`
3. URL final: `https://pay.notifybot.com.br/fintoc/payment/`

## Deploy — alternativas

- **Netlify**: import repo → publish dir `/` → custom domain
- **Vercel**: import repo → output `/` → custom domain
- **GitHub Pages**: configura raiz como source

## Licença

MIT
