---
name: triagem-de-emails
description: Skill de triagem e classificação de emails para Fernando. Recebe uma lista de emails (colada pelo usuário ou lida de arquivo), classifica por prioridade (alta/média/baixa), identifica ações necessárias, e injeta o resumo no CoworkOS para aparecer no widget de E-mails. Use quando o usuário pedir para triar emails, classificar caixa de entrada, ou fazer resumo de emails.
---
# Triagem de E-mails
Skill para processar, classificar e resumir emails, injetando o resultado no CoworkOS.

---
## Arquivos do sistema
| Arquivo | Caminho | Descrição |
|---------|---------|-----------|
| CoworkOS | `/home/user/Coding/Cowork OS/CoworkOS.html` | Interface principal — recebe emails injetados |

---
## Passo 1 — Receber os emails
O usuário vai fornecer os emails de uma dessas formas:
- **Colagem direta** no chat (texto com remetente, assunto, corpo)
- **Arquivo de texto** com emails exportados
- **Lista simples** no formato "De: X | Assunto: Y | Preview: Z"

Se o usuário não fornecer emails, pergunte: *"Cole aqui os emails que você quer triar, ou descreva os que estão na sua caixa de entrada."*

---
## Passo 2 — Classificar cada email
Para cada email, determine:

### Prioridade
- **high** (alta): requer ação imediata, cliente, financeiro, deadline, urgente
- **medium** (média): requer ação mas sem urgência imediata, follow-up, informação útil
- **low** (baixa): newsletter, notificação automática, FYI, pode esperar

### Campos de saída
```json
{
  "id": "e{número}",
  "sender": "Nome do remetente / empresa",
  "subject": "Assunto do email",
  "preview": "Resumo de 1 linha do conteúdo (máx 120 chars)",
  "priority": "high | medium | low",
  "read": false,
  "date": "YYYY-MM-DD",
  "action": "Opcional: ação recomendada ex: 'Responder até sexta', 'Arquivar', 'Encaminhar para time'"
}
```

---
## Passo 3 — Montar o array de emails
```json
[
  { "id": "e1", "sender": "...", "subject": "...", "preview": "...", "priority": "high", "read": false, "date": "YYYY-MM-DD" },
  { "id": "e2", ... },
  ...
]
```
Ordene por prioridade: high primeiro, depois medium, depois low.

---
## Passo 4 — Injetar no CoworkOS.html
Leia o arquivo `/home/user/Coding/Cowork OS/CoworkOS.html`.

### Se o marcador `<!-- EMAILS_INJETADOS_START -->` JÁ existir:
Substitua todo o bloco entre `<!-- EMAILS_INJETADOS_START -->` e `<!-- EMAILS_INJETADOS_END -->` pelo novo script abaixo.

### Se o marcador NÃO existir:
Insira o bloco imediatamente antes de `</body>`.

### Bloco a injetar:
```html
<!-- EMAILS_INJETADOS_START -->
<script>
(function() {
  var KEY = 'coworkos_emails_v1';
  var EMAILS = EMAILS_JSON_PLACEHOLDER;
  localStorage.setItem(KEY, JSON.stringify(EMAILS));
})();
</script>
<!-- EMAILS_INJETADOS_END -->
```

Substitua `EMAILS_JSON_PLACEHOLDER` pelo array JSON de emails do Passo 3 (sem aspas ao redor — é um array JS inline).

---
## Passo 5 — Resumo para o usuário
```
📧 Triagem concluída — {N} emails processados

🔴 Alta prioridade ({N}):
  • [Remetente] Assunto — Ação recomendada

🟡 Média prioridade ({N}):
  • [Remetente] Assunto

⚪ Baixa prioridade ({N}):
  • [Remetente] Assunto

✅ CoworkOS atualizado. Abra o arquivo para ver o widget de E-mails no Dashboard.
```

---
## Regras
- **Preserve o contexto**: nunca altere o conteúdo original dos emails, apenas resuma
- **Seja direto**: previews de no máximo 120 caracteres, focado na ação
- **Prioridade conservadora**: quando em dúvida, eleve a prioridade (melhor agir cedo do que tarde)
- **Ação clara**: para emails de alta prioridade, sempre sugerir uma ação específica e prazo
