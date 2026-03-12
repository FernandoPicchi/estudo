# CoworkOS — Contexto do Projeto

## O que é
CoworkOS é um sistema pessoal de produtividade e gestão de conhecimento para Fernando. Funciona como um single-page app HTML que roda localmente no browser, com toda persistência via `localStorage`.

## Arquivos principais
| Arquivo | Localização | Função |
|---------|-------------|--------|
| `CoworkOS.html` | `/home/user/Coding/Cowork OS/` | App principal — abrir no browser |
| `CRM de Conteúdos.html` | `/home/user/estudo/` | App legado (descontinuado) |
| `historico-recomendacoes.json` | `/home/user/estudo/` | Histórico anti-repetição da pesquisa de IA |
| `pesquisa-diaria-ai/SKILL.md` | `/home/user/estudo/` | Skill de pesquisa diária de IA |
| `Comunicações/triagem-de-emails/SKILL.md` | `/home/user/estudo/` | Skill de triagem de emails |

## localStorage Keys
| Key | Tipo | Conteúdo |
|-----|------|----------|
| `crm_ai_sessions_v1` | Array | Sessões de estudo de IA `[{id, date, sources, trends, recs, notes}]` |
| `coworkos_todos_v1` | Array | Tarefas `[{id, text, done, createdAt}]` |
| `coworkos_intention_v1` | String | Intenção do dia |
| `coworkos_briefing_v1` | String | Resumo diário |
| `coworkos_notas_v1` | String | Notas rápidas |
| `coworkos_emails_v1` | Array | Emails triados `[{id, sender, subject, preview, priority, read, date}]` |
| `coworkos_skills_v1` | Array | Progresso de skills de IA `[{id, name, icon, color, pct, level, next}]` |
| `coworkos_claude_api_key` | String | API key Anthropic (para botão "Como fazer?") |

## Skills disponíveis
- **pesquisa-diaria-ai**: Pesquisa tendências de IA e injeta sessão em `crm_ai_sessions_v1` no CoworkOS.html
- **triagem-de-emails**: Classifica emails e injeta em `coworkos_emails_v1` no CoworkOS.html

## Padrão de injeção (como as skills atualizam o CoworkOS)
As skills injetam dados diretamente no HTML via blocos de script marcados:
```html
<!-- SESSAO_INJETADA_START -->
<script>(function(){ /* seta localStorage */ })();</script>
<!-- SESSAO_INJETADA_END -->

<!-- EMAILS_INJETADOS_START -->
<script>(function(){ /* seta localStorage */ })();</script>
<!-- EMAILS_INJETADOS_END -->
```

## Estrutura do CoworkOS.html
- **Sidebar**: Dashboard | Tarefas | CRM de Estudos | E-mails
- **Dashboard**: Briefing do Dia, Notas Rápidas, Lista de Tarefas, Triagem de E-mails
- **CRM de Estudos**: Abas (IA / Programação / Design / +), Claude Insights, Feed de Estudos, Progresso de Skills
- **Tarefas**: Lista completa com filtros
- **E-mails**: Lista completa de emails triados

## Como usar o botão "Como fazer?"
1. Clique em "✦ Como fazer?" em qualquer tarefa
2. Se API key não configurada: clique em "Configurar Claude" no rodapé da sidebar
3. Insira sua Anthropic API key (começa com `sk-ant-api03-`)
4. A key fica salva localmente — nunca é enviada a nenhum servidor além da API Anthropic

## Desenvolvendo novas features
- Sempre edite o arquivo único `CoworkOS.html`
- Novos dados: adicionar nova `localStorage key` e documentar aqui
- Novas skills: criar `SKILL.md` em `/home/user/estudo/{nome-da-skill}/`
- Após mudanças: commitar e fazer push para a branch `claude/evolve-coworkos-system-rLjQz`
