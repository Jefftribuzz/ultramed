# Diagnóstico gratuito de SST — Ultramed Ocupacional

Página de captação para campanhas de tráfego pago. O visitante responde perguntas rápidas e recebe na hora:
- um score de conformidade (0–100);
- um semáforo por área;
- a lista de pendências por prioridade;
- um botão para falar com a Ultramed no WhatsApp, já com o resumo do diagnóstico na mensagem.

**Endereço:** https://ultramed.tribuzz.com.br/

| Arquivo | O que é |
|---|---|
| `index.html` | O diagnóstico (HTML, CSS e JavaScript em um único arquivo, sem dependências) |
| `politica-de-privacidade.html` | Política de privacidade (LGPD), com link no formulário e no rodapé |
| `logo-ultramed.png` | Logo da Ultramed em versão horizontal (símbolo + nome), usada no cabeçalho das duas páginas |

Os três arquivos precisam ser publicados **juntos, na mesma pasta**. Se a logo faltar, o cabeçalho mostra a logo em texto.

---

## 1. Publicar em `ultramed.tribuzz.com.br`

O site fica hospedado no **Netlify** (grátis), e o Registro.br só aponta o subdomínio para lá. As entradas atuais do DNS (`tribuzz.com.br`, `lp` e `www`) não mudam.

### Passo 1 — Netlify
1. Crie uma conta em https://app.netlify.com (pode entrar com o Google).
2. Abra https://app.netlify.com/drop e **arraste a pasta** que contém `index.html`, `politica-de-privacidade.html` e `logo-ultramed.png`.
3. O Netlify cria o site com um endereço do tipo `nome-aleatorio-123.netlify.app`. Abra esse endereço e confira se a página funciona.
4. (Opcional) Em **Site configuration → Change site name**, troque para algo como `ultramed-diagnostico`. O endereço passa a ser `ultramed-diagnostico.netlify.app`.
5. Em **Domain management → Add a domain**, digite `ultramed.tribuzz.com.br` e confirme. Se o Netlify perguntar sobre DNS, escolha configurar em um provedor externo.

### Passo 2 — Registro.br
1. Em https://registro.br, abra o domínio **tribuzz.com.br** e vá em **DNS → Configurar zona DNS**.
2. Clique em **NOVA ENTRADA** e preencha:

   | Tipo | Nome | Dados |
   |---|---|---|
   | CNAME | `ultramed` | `ultramed-diagnostico.netlify.app` (o endereço do seu site no Netlify) |

3. Clique em **SALVAR ALTERAÇÕES**.

### Passo 3 — HTTPS
A propagação costuma levar de alguns minutos a algumas horas. Depois disso, o Netlify ativa o **https** sozinho, em **Domain management → HTTPS**. Se aparecer "Awaiting External DNS", é só aguardar.

### Para atualizar a página depois
No Netlify, abra o site, vá na aba **Deploys** e arraste a pasta de novo. O endereço não muda.

---

## 2. Checklist antes de ligar os anúncios

| # | Item | Situação |
|---|------|----------|
| 1 | WhatsApp `(32) 99900-2140` | ✅ Configurado |
| 2 | Meta Pixel `1818935802782375` | ✅ Configurado (no `CONFIG` e no `<noscript>`) |
| 3 | Política de privacidade, CNPJ e contato | ✅ `politica-de-privacidade.html` |
| 4 | Publicar no Netlify e criar o CNAME `ultramed` no Registro.br | ⬜ Seção 1 |
| 5 | Abrir `https://ultramed.tribuzz.com.br/` e conferir o cadeado (https) | ⬜ |
| 6 | Verificar o domínio `tribuzz.com.br` na Meta (veja abaixo) | ⬜ |
| 7 | Em **Gerenciador de Eventos → Testar eventos**, conferir `PageView`, `InicioDiagnostico`, `Lead` e `Contact` | ⬜ |
| 8 | Abrir o link pelo Instagram no celular e fazer um diagnóstico completo, até o WhatsApp | ⬜ |
| 9 | (Opcional) Logo em imagem e imagem de pré-visualização do link (1200×630) | `CONFIG.empresa.logoUrl` e meta `og:image` |

### Verificar o domínio na Meta
Faça isso no Gerenciador de Negócios que é dono do Pixel `1818935802782375`:
1. **Configurações do negócio → Segurança da marca → Domínios → Adicionar** e digite `tribuzz.com.br`.
2. Escolha a verificação por **registro TXT de DNS** e copie o código `facebook-domain-verification=...`.
3. No Registro.br, adicione uma **NOVA ENTRADA**: tipo **TXT**, nome `tribuzz.com.br`, dados com o código copiado. Salve.
4. Volte à Meta e clique em **Verificar**.

### URL para os anúncios
```
https://ultramed.tribuzz.com.br/?utm_source=meta&utm_medium=paid_social&utm_campaign={{campaign.name}}&utm_content={{ad.name}}&utm_term={{adset.name}}
```
Os parâmetros entre chaves duplas são preenchidos automaticamente pela Meta.

Use o evento **Lead** (envio do formulário) como conversão da campanha. O evento **Contact** (clique no WhatsApp) mede a intenção mais forte.

---

## 3. O que é rastreado

| Momento | Meta Pixel | `dataLayer` (GTM) |
|---|---|---|
| Abrir a página | `PageView` | — |
| Responder o filtro "É uma empresa?" | `filtro-empresa` (personalizado), com `resposta` = `empresa` ou `pessoa_fisica` | `filtro-empresa` |
| Responder "Sim, é uma empresa" (o diagnóstico começa) | `InicioDiagnostico` (personalizado) | `diagnostico_inicio` |
| Enviar o formulário | `Lead` | `diagnostico_lead` |
| Clicar no WhatsApp | `Contact`, com `app` = `whatsapp`, `whatsapp_business` ou `padrao` | `diagnostico_whatsapp` |

O Pixel **não recebe dados pessoais**: só score, classificação e quantidade de pendências.

Os eventos são os mesmos no computador e no celular:
- **PageView** é contado uma vez por visita. O Pixel está configurado para não contar cada pergunta como uma página nova (`fbq.disablePushState`).
- **Contact no celular:** o toque no WhatsApp registra o evento e só abre a conversa 0,35 s depois. Sem essa espera, o app do WhatsApp poderia abrir antes de o evento sair, principalmente no navegador interno do Instagram. No computador, o WhatsApp abre em outra aba e o resultado continua na tela.
- Bloqueadores de anúncio e navegadores com proteção contra rastreamento podem impedir o Pixel em qualquer aparelho. Para cobrir esses casos, o caminho é a API de Conversões.

### Fluxo do questionário
1. **Início**: botão "Começar diagnóstico".
2. **Filtro**: "O diagnóstico é para uma empresa?".
   - **Sim, é uma empresa**: o diagnóstico começa.
   - **Não, sou pessoa física**: aparece um aviso e, cerca de 1 s depois, a pessoa é levada ao Instagram da Ultramed (`CONFIG.empresa.instagramUrl`). A espera garante que o evento `filtro-empresa` chegue à Meta.
3. **5 etapas**, cada uma com as perguntas do mesmo assunto na mesma página: Sua empresa · Documentos-base · Exames · Laudos · eSocial e treinamentos. Perguntas que dependem de outra aparecem na hora, na mesma página (ex.: LTCAT só se houver exposição). Empresas sem CLT respondem só a etapa 1.
4. **Formulário** (nome e empresa) e **resultado**.

Para criar públicos personalizados ou conversões, use o evento `filtro-empresa` com o parâmetro `resposta`. Por exemplo, um público de quem respondeu `pessoa_fisica`, para excluir das campanhas.

### WhatsApp ou WhatsApp Business
O link comum do WhatsApp (`wa.me`) abre o app que o celular escolher, em geral o WhatsApp comum.
- **Android:** ao tocar no botão, a pessoa escolhe **WhatsApp** ou **WhatsApp Business**, e o app certo abre direto. Se o app escolhido não estiver instalado, abre o link `wa.me`.
- **iPhone:** a Apple não permite escolher o app por link. Abre o `wa.me`, e o iPhone decide qual app usar.
- **Computador:** o WhatsApp Web ou o app de desktop abre em outra aba.

### Sem webhook: como o lead chega
Hoje o formulário pede só **nome e empresa**, que servem para personalizar o resultado e a mensagem do WhatsApp. Nada é gravado em servidor. **O lead chega à Ultramed quando a pessoa toca no botão do WhatsApp e envia a mensagem.** Quem preenche e não clica fica registrado só como evento `Lead` na Meta, sem os dados de contato.

Para guardar todos os leads (por exemplo, numa planilha do Google ou no Kommo):
1. Configure `CONFIG.rastreamento.webhookUrl`.
2. Mude `CONFIG.formulario.pedirContato` para `true`. O formulário volta a pedir WhatsApp e e-mail, e o texto de consentimento muda junto.
3. Atualize as seções 2 a 5 da política de privacidade.

---

## 4. Como editar

Tudo fica no objeto `CONFIG`, no início do `<script>` do `index.html`:

- **Perguntas, opções e pesos**: `CONFIG.blocos`. O valor de cada opção multiplica o peso da pergunta: `1` = em dia, `0.5` = parcial, `0` = não/não sei. O score é normalizado de 0 a 100.
- **Pendências**: cada pergunta tem uma função `pendencia` com título, risco, serviço da Ultramed e prioridade.
- **Quando cada pergunta aparece**: `mostrarSe`. Por exemplo, o LTCAT só aparece quando há exposição, e empresas sem CLT vão direto ao formulário.
- **Etapas (páginas)**: o campo `etapa` de cada bloco. Blocos seguidos com a mesma `etapa` ficam na mesma página, e cada bloco continua sendo uma área separada no semáforo.
- **Filtro**: textos em `CONFIG.filtro`. O destino de quem não é empresa fica em `CONFIG.empresa.instagramUrl`.
- **Faixas do score**: `CONFIG.faixas` (Crítico 0–40, Atenção 41–70, Bom 71–100).
- **Cores**: `CONFIG.marca` (hoje com os verdes da logo: `#00853b` e `#205830`). As cores do semáforo ficam no `:root` do `<style>`.
- **Logo**: `CONFIG.empresa.logoUrl` (arquivo `logo-ultramed.png`, na mesma pasta).
- **Prazo da NR-01**: `CONFIG.nr01.dataFiscalizacao`. Os textos passam sozinhos de "a partir de" para "desde" quando a data chega.

---

## 5. Pontos de atenção

- **Revisão jurídica da política**: o texto segue a LGPD e descreve o funcionamento real da página, mas vale passar por um advogado antes de anunciar. Três pontos merecem confirmação: o papel da Tribuzz como operadora, a hospedagem no Netlify e a base legal escolhida para o Pixel (legítimo interesse).
- **Pixel antes do consentimento**: o `PageView` dispara ao abrir a página, como na maioria das landing pages. Se o jurídico exigir consentimento prévio para cookies de publicidade, será preciso um banner de cookies, e isso reduz o volume de dados para otimizar a campanha.
- **Prazo da NR-01**: os textos partem de 26/05/2026 como início da fiscalização punitiva dos riscos psicossociais. Se o MTE publicar outra data, ajuste `CONFIG.nr01`.
- **Diagnóstico orientativo**: a página avisa que o resultado é preliminar e não substitui avaliação técnica nem consultoria jurídica. Mantenha esse aviso.
