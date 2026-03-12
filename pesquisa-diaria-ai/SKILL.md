---
name: pesquisa-diaria-ai
description: Skill de pesquisa diária de conteúdos de IA para Fernando. Pesquisa YouTube, LinkedIn, newsletters e notícias da web sobre tendências de IA (Claude Code, LLMs, automação, mercado). Recomenda 3–5 conteúdos diversificados por dia, evitando repetições entre sessões. Atualiza automaticamente o CoworkOS (aba CRM de Estudos > IA). Use sempre que o usuário pedir pesquisa diária de IA, recomendação de conteúdos de IA para estudar, ou quando executada via agendamento automático diário.
---
# Pesquisa Diária de IA
Skill para descobrir e recomendar conteúdos de IA frescos todo dia, sem repetição.

---
## Arquivos do sistema
| Arquivo | Caminho | Descrição |
|---------|---------|-----------|
| CoworkOS | `/home/user/Coding/Cowork OS/CoworkOS.html` | Interface principal — recebe a sessão injetada |
| Histórico | `/home/user/estudo/historico-recomendacoes.json` | Lista de URLs/títulos já recomendados |

---
## Passo 1 — Ler o histórico
Leia `historico-recomendacoes.json`. Se não existir, crie com estrutura vazia:
```json
{
  "ultima_atualizacao": "YYYY-MM-DD",
  "recomendados": []
}
```
Cada item: `{ "titulo": "...", "url": "...", "data": "YYYY-MM-DD" }`

Guarde a lista de URLs e títulos já vistos — você vai usá-la no filtro do Passo 4.

---
## Passo 2 — Pesquisar trending topics
Faça buscas em paralelo via WebSearch cobrindo todos os eixos abaixo.

**Claude / Anthropic:**
- `claude code tutorial 2026`
- `anthropic claude news march 2026`
- `claude agents API 2026`

**OpenAI / outros LLMs:**
- `openai news march 2026`
- `GPT latest news 2026`
- `gemini AI update march 2026`

**Automação com IA:**
- `n8n AI automation tutorial 2026`
- `AI agents workflow automation 2026`
- `make.com AI automation 2026`

**Mercado de IA:**
- `AI startup funding march 2026`
- `AI market news march 2026`
- `artificial intelligence industry news 2026`

**Newsletters e artigos:**
- `"The Rundown AI" newsletter march 2026`
- `"TLDR AI" newsletter march 2026`
- `AI news site:techcrunch.com OR site:wired.com OR site:venturebeat.com march 2026`

**LinkedIn:**
- `site:linkedin.com/posts claude anthropic 2026`
- `site:linkedin.com/posts AI automation tools 2026`

Colete no mínimo 10–15 candidatos com título, URL, fonte e data.

---
## Passo 3 — Identificar trending topics do dia
Com base nos resultados, identifique 3–6 temas em alta. Cada um:
```json
{ "text": "Nome do tópico", "type": "hot | trending | new | rising" }
```
Exemplos: `"Claude Code avançando"`, `"AI Agents dominando"`, `"GPT-5 rumores"`, `"n8n + IA crescendo"`

---
## Passo 4 — Filtrar e selecionar recomendações

### Filtro anti-repetição
Para cada candidato, verifique se já foi recomendado:
- Compare URL (exact match, case-insensitive)
- Compare título (substring match, case-insensitive, 5+ palavras em comum = descarte)

Descarte qualquer candidato que passe em alguma dessas verificações.

### Seleção
Escolha 3–5 itens dos candidatos restantes priorizando:
1. **Frescor** — conteúdo dos últimos 7 dias primeiro; aceite até 30 dias
2. **Diversidade de tipo** — obrigatório: mínimo 1 vídeo E 1 artigo/newsletter/notícia
3. **Diversidade de subtema** — evite recomendar dois itens do mesmo assunto
4. **Qualidade da fonte** — prefira canais/publicações conhecidas e relevantes

### Formato de cada recomendação
```json
{
  "id": "r1",
  "num": 1,
  "title": "Título completo do conteúdo",
  "channel": "Nome do canal / publicação / autor",
  "type": "Vídeo | Artigo | Post LinkedIn | Newsletter | Notícia",
  "subtopic": "Ex: Claude Code | Automação | Mercado AI | LLMs | Ferramentas",
  "link": "URL completa e funcional",
  "views": "Indicador de alcance ex: '45k views', '1.2k likes', 'TechCrunch'",
  "status": "pendente"
}
```

---
## Passo 5 — Montar o objeto de sessão
```json
{
  "id": "session_YYYYMMDD",
  "date": "YYYY-MM-DD",
  "sources": "YouTube, LinkedIn, Web",
  "trends": [ /* trending topics do Passo 3 */ ],
  "recs": [ /* recomendações do Passo 4 */ ],
  "notes": ""
}
```

---
## Passo 6 — Injetar no CoworkOS.html
Leia o arquivo `/home/user/Coding/Cowork OS/CoworkOS.html`.

### Se o marcador `<!-- SESSAO_INJETADA_START -->` JÁ existir:
Substitua todo o bloco entre `<!-- SESSAO_INJETADA_START -->` e `<!-- SESSAO_INJETADA_END -->` pelo novo script abaixo.

### Se o marcador NÃO existir:
Insira o bloco imediatamente antes de `</body>`.

### Bloco a injetar:
```html
<!-- SESSAO_INJETADA_START -->
<script>
(function() {
  var KEY = 'crm_ai_sessions_v1';
  var SESSION = SESSION_JSON_PLACEHOLDER;
  var existing = [];
  try { existing = JSON.parse(localStorage.getItem(KEY) || '[]'); } catch(e) {}
  var alreadyExists = existing.some(function(s) { return s.id === SESSION.id; });
  if (!alreadyExists) {
    existing.unshift(SESSION);
    localStorage.setItem(KEY, JSON.stringify(existing));
  }
})();
</script>
<!-- SESSAO_INJETADA_END -->
```

Substitua `SESSION_JSON_PLACEHOLDER` pelo JSON da sessão do Passo 5 (sem aspas ao redor — é um objeto JS inline).

> **Por que isso funciona:** O script roda quando o usuário abre o HTML no browser. Ele faz um merge não-destrutivo: só adiciona a sessão se ainda não existir. Sessions criadas manualmente pelo usuário no browser são preservadas.

---
## Passo 7 — Atualizar o histórico
Atualize `historico-recomendacoes.json` adicionando as novas recomendações:
```json
{
  "ultima_atualizacao": "YYYY-MM-DD",
  "recomendados": [
    { "titulo": "...", "url": "...", "data": "YYYY-MM-DD" },
    /* ... itens anteriores mantidos ... */
  ]
}
```
Mantenha no máximo os últimos **90 dias** de histórico. Descarte entradas com `data` mais antiga que 90 dias.

---
## Passo 8 — Resumo para o usuário
```
📅 Pesquisa diária — {DATA}
🔥 Trending hoje: Tópico 1 · Tópico 2 · Tópico 3
📚 Recomendações:
1. [Vídeo] "Título" — Canal → URL
2. [Artigo] "Título" — Publicação → URL
3. [Post LinkedIn] "Título" — Autor → URL
...
✅ CoworkOS atualizado. Abra o arquivo para ver a aba CRM de Estudos > IA.
```

---
## Regras
- **Nunca repita**: URLs e títulos já vistos no histórico são descartados
- **Sempre diversifique**: mínimo 2 tipos de conteúdo diferentes por sessão
- **Prefira frescor**: conteúdo dos últimos 7 dias tem prioridade
- **Relevância obrigatória**: todos os itens devem ser sobre IA, LLMs, automação ou mercado de tech
- **Links reais**: use apenas URLs verificadas nos resultados de busca — nunca invente links
