# Novarys Creator Summit · Landing Page

Página de vendas do Novarys Creator Summit (1 e 2 de agosto de 2026, Auditório Bassoft, Alphaville-SP).

Versão de referência no ar: https://novarys-summit.vercel.app

## Estrutura

Site 100% estático, sem build e sem dependências:

```
index.html        página completa (HTML + CSS + JS no mesmo arquivo)
og.html           gerador da imagem de compartilhamento (og.jpg)
vercel.json       headers de cache (opcional, específico do Vercel)
assets/fonts/     Anton, Archivo e Space Mono (self-hosted, WOFF2)
assets/img/       fotos dos palestrantes, logo e og.jpg
```

## Como publicar no domínio da Novarys

Qualquer hospedagem de arquivos estáticos serve (Vercel, Netlify, Cloudflare Pages, Apache, Nginx). Basta servir a pasta inteira na raiz do domínio.

Após apontar o domínio, **atualizar 2 linhas no `<head>` do index.html**:

```html
<meta property="og:image" content="https://SEU-DOMINIO/assets/img/og.jpg">
<meta property="og:url"   content="https://SEU-DOMINIO/">
```

## Fluxo de compra

Todos os botões "Garantir minha vaga" abrem um pop-up que coleta nome, e-mail e WhatsApp e redireciona para o checkout da Greenn já pré-preenchido:

```
https://payfast.greenn.com.br/pre-checkout/gnvan7d?name=...&email=...&phone=...
```

Antes do redirect, o lead pode ser enviado para um webhook da Novarys (`LEAD_WEBHOOK_URL` no bloco `CONFIG`, no início do `<script>` do index.html). A falha dessa gravação nunca bloqueia o redirect. Se a URL ficar vazia, o pop-up apenas redireciona ao checkout.

## Para onde vão os leads (planilha da Novarys)

Receita para gravar cada lead em uma **planilha Google da Novarys**, sem servidor e sem custo:

1. Crie uma planilha no Google Sheets (na conta da Novarys).
2. Menu **Extensões > Apps Script**, apague o conteúdo e cole:

```js
function doPost(e) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var s = ss.getSheetByName('Leads') || ss.insertSheet('Leads');
  if (s.getLastRow() === 0) {
    s.appendRow(['Data', 'Nome', 'E-mail', 'WhatsApp', 'Lote', 'Origem']);
  }
  s.appendRow([
    new Date(),
    e.parameter.name, e.parameter.email, e.parameter.phone,
    e.parameter.ticket, e.parameter.source
  ]);
  return ContentService.createTextOutput('ok');
}
```

3. **Implantar > Nova implantação > Tipo: App da Web**, com "Executar como: **eu**" e "Quem pode acessar: **qualquer pessoa**". Autorize e copie a URL gerada (`https://script.google.com/macros/s/.../exec`).
4. Cole essa URL em `LEAD_WEBHOOK_URL` no `CONFIG` do index.html.

Pronto: cada envio do pop-up vira uma linha na planilha, com data, nome, e-mail, WhatsApp, lote e origem. O mesmo campo aceita qualquer outro webhook (CRM, n8n, Zapier) que receba POST form-encoded.

## Pontos editáveis mais comuns

- **Vagas restantes**: buscar por `47` (carimbo do hero e seção final) e `100` (auditório).
- **Lote/preço**: buscar por `97` e `Primeiro lote`.
- **Palestrantes**: seção `LINE-UP` (uma `row-spk` por pessoa) e a faixa `marquee`.
- **Pixel/GTM**: colar o snippet no comentário indicado no início do `<script>`. Os eventos `open_lead_popup` e `lead_submit` já são disparados no `dataLayer`.
- **Countdown**: a data alvo está em `EVENT_START` no `<script>`.

---

Desenvolvido por [Opus Midias](https://opusmidias.com.br).
