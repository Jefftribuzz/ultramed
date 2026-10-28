# Diagnóstico gratuito de SST — Ultramed Ocupacional

Página de captação de leads para campanhas de tráfego pago. O visitante responde perguntas rápidas e recebe na hora um score de conformidade (0–100), um semáforo por área, a lista de pendências por prioridade e um botão para falar com a Ultramed no WhatsApp, com o resumo do diagnóstico já na mensagem.

Tudo fica em **um único arquivo**: `index.html` (HTML, CSS e JavaScript juntos, sem dependências externas).

---

## 1. Publicar em `tribuzz.com.br/ultramed`

A página só precisa que o arquivo `index.html` fique dentro de uma pasta chamada `ultramed` na raiz do site. Escolha o caso que corresponde à hospedagem do tribuzz.com.br:

### Hospedagem com painel (Hostinger, cPanel, Locaweb, KingHost, HostGator…)
1. Abra o **Gerenciador de Arquivos** (ou conecte por FTP).
2. Entre na pasta raiz do site: geralmente `public_html` (às vezes `www` ou `htdocs`).
3. Crie a pasta `ultramed`.
4. Envie o `index.html` para dentro dela → `public_html/ultramed/index.html`.
5. Acesse `https://tribuzz.com.br/ultramed/`.

### Site em WordPress
Funciona igual ao caso acima: envie a pasta pelo Gerenciador de Arquivos/FTP (não crie uma "página" no WordPress). Uma pasta real tem prioridade sobre as páginas do WordPress.
Se usar plugin de cache ou Cloudflare, limpe o cache depois de enviar ou atualizar o arquivo.

### Site publicado na Vercel ou no Netlify a partir de um repositório
- Site estático: coloque o arquivo em `ultramed/index.html` dentro da pasta publicada.
- Next.js: coloque em `public/ultramed/index.html` e acesse `/ultramed/index.html`, ou crie uma *rewrite* de `/ultramed` para `/ultramed/index.html`.

### Alternativa: subdomínio `ultramed.tribuzz.com.br`
Útil se não houver acesso à hospedagem principal:
1. Em app.netlify.com: **Add new site → Deploy manually** e arraste a pasta com o `index.html`.
2. Em **Domain settings**, adicione `ultramed.tribuzz.com.br`.
3. No DNS do domínio, crie um registro **CNAME** `ultramed` apontando para o endereço `*.netlify.app` indicado.

> Use sempre a URL com **https** e com a barra final nos anúncios: `https://tribuzz.com.br/ultramed/`

---

## 2. Checklist antes de ligar os anúncios

| # | Item | Onde |
|---|------|------|
| 1 | Confirmar o número do WhatsApp (hoje: `5532999002140`) | `CONFIG.empresa.whatsapp` e no bloco `<noscript>` |
| 2 | Colocar o ID do Pixel/conjunto de dados da Meta | `CONFIG.rastreamento.metaPixelId` |
| 3 | Verificar o domínio `tribuzz.com.br` no Gerenciador de Negócios da Meta | Business Settings → Brand Safety → Domains |
| 4 | Conferir os eventos no **Gerenciador de Eventos → Testar eventos**: `PageView`, `InicioDiagnostico`, `Lead`, `Contact` | Meta Events Manager |
| 5 | Configurar o webhook (Make/n8n) e fazer um envio de teste | `CONFIG.rastreamento.webhookUrl` |
| 6 | Preencher a política de privacidade e o CNPJ | `CONFIG.empresa.politicaPrivacidadeUrl`, `CONFIG.empresa.cnpj` |
| 7 | (Opcional) Logo em imagem e imagem de pré-visualização do link (1200×630) | `CONFIG.empresa.logoUrl`, meta `og:image` no `<head>` |
| 8 | Abrir o link pelo Instagram no celular e fazer um diagnóstico completo | — |

**URL sugerida para o anúncio** (os parâmetros dinâmicos da Meta preenchem campanha e anúncio sozinhos):

```
https://tribuzz.com.br/ultramed/?utm_source=meta&utm_medium=paid_social&utm_campaign={{campaign.name}}&utm_content={{ad.name}}&utm_term={{adset.name}}
```

Para otimizar a campanha, use o evento **Lead** (envio do formulário) como conversão. O evento **Contact** (clique no WhatsApp) mede a intenção mais forte.

---

## 3. O que é rastreado

| Momento | Meta Pixel | `dataLayer` (GTM) | Webhook |
|---|---|---|---|
| Abrir a página | `PageView` | — | — |
| Clicar em "Começar diagnóstico" | `InicioDiagnostico` (personalizado) | `diagnostico_inicio` | — |
| Enviar o formulário | `Lead` (ou `CompleteRegistration`) | `diagnostico_lead` | `evento=lead` |
| Refazer com os mesmos dados | — (não duplica o Lead) | — | `evento=diagnostico_refeito` |
| Clicar no WhatsApp | `Contact` | `diagnostico_whatsapp` | `evento=whatsapp_click` |

O Pixel e o GTM **não recebem dados pessoais**: só score, classificação e quantidade de pendências.

### Campos enviados ao webhook

Formato padrão: formulário (`application/x-www-form-urlencoded`), que funciona com Make e n8n sem bloqueio de CORS. Para enviar JSON, use `webhookFormato: "json"`, desde que o destino aceite CORS.

`evento`, `lead_id`, `meta_event_id`, `data_hora`, `nome`, `empresa`, `whatsapp` (com 55), `email`, `consentimento_lgpd`, `consentimento_data`, `score`, `classificacao`, `qtd_pendencias`, `pendencias_alta`, `pendencia_nr01_psicossocial`, `pendencias` (títulos separados por ` | `), `funcionarios`, `setor`, `grau_risco`, `situacao`, `utm_source`, `utm_medium`, `utm_campaign`, `utm_content`, `utm_term`, `utm_id`, `fbclid`, `gclid`, `fbp`, `fbc`, `pagina`, `referrer`, `user_agent`, `detalhes_json` (respostas, semáforo e pendências em JSON).

O `meta_event_id` é igual ao `eventID` do evento Lead do Pixel. Se mais tarde você enviar o lead também pela **API de Conversões** (por exemplo, pelo Make), use esse ID para a Meta não contar o lead duas vezes. `fbp` e `fbc` melhoram a correspondência.

---

## 4. Como editar

Tudo fica no objeto `CONFIG`, no início do `<script>`:

- **Perguntas, opções e pesos**: `CONFIG.blocos`. O valor de cada opção multiplica o peso da pergunta: `1` = em dia, `0.5` = parcial, `0` = não/não sei. O score é normalizado de 0 a 100.
- **Pendências**: cada pergunta tem uma função `pendencia` que devolve título, risco, serviço da Ultramed e prioridade.
- **Quando cada pergunta aparece**: `mostrarSe` (por exemplo, o LTCAT só aparece quando há exposição; empresas sem CLT pulam direto para o formulário).
- **Faixas do score e textos**: `CONFIG.faixas` (Crítico 0–40, Atenção 41–70, Bom 71–100).
- **Cores**: `CONFIG.marca`. As cores do semáforo ficam no `:root` do `<style>`.
- **Prazo da NR-01**: `CONFIG.nr01.dataFiscalizacao`. Os textos passam sozinhos de "a partir de" para "desde" quando a data chega.

---

## 5. Pontos de atenção

- **LGPD e Pixel**: o formulário só envia dados com o consentimento marcado. O Pixel dispara o `PageView` ao abrir a página, como na maioria das landing pages. Se o jurídico pedir consentimento também para cookies de publicidade, é preciso adicionar um banner de cookies antes de iniciar o Pixel.
- **Prazo da NR-01**: os textos partem de 26/05/2026 como início da fiscalização punitiva dos riscos psicossociais. Se o MTE publicar uma nova data, ajuste `CONFIG.nr01`.
- **Diagnóstico orientativo**: a página informa que o resultado é preliminar e não substitui avaliação técnica nem consultoria jurídica. Mantenha esse aviso.
