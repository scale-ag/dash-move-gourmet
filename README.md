# Dashboard de Captura de Leads · Move Gourmet (Fernanda)

Dashboard **100% na nuvem** do funil **Move Gourmet** (cliente Fernanda) de
controle de **tráfego pago (Meta Ads)**. Lê a planilha de Meta Ads, trata as
**conversas iniciadas** no WhatsApp pelo anúncio como **Leads** e publica tudo no
**GitHub Pages**. Reconstrói sozinha a cada ~30 min, disparada pelo
**cron-job.org** — sem depender de nenhum PC ligado.

**URL pública:** `https://scale-ag.github.io/dash-move-gourmet/`

---

## O que ela mostra

- **KPIs**: Gasto Total, Impressões, Cliques, CTR, CPC, CPM, **Leads** (conversas iniciadas) e **CPL**.
- **Evolução diária**: gasto/dia, leads/dia, CPL/dia (heatmap por coluna).
- **Distribuição de leads**: por origem, por **cidade** (da Campaign Name), por plataforma e por **conjunto** (Ad Set).
- **Hierarquia Campanha → Conjunto → Anúncio** com filtro cruzado bidirecional.
- **Aba Relatório**: painel de metas editável + Top Anúncios (ranqueados por Leads/CPL) + Insights de Tráfego.
- **Toggle de imposto da mídia paga** e **modo claro/escuro**.

> ℹ️ **Estágios sem fonte nesta conta:** não há critério de **MQL**, nem abas de
> **Compradores/Vendas**. Por isso **MQL · Vendas · Faturamento · ROAS** aparecem
> como **“-”** (lacuna) até existir uma fonte para eles. O funil efetivo é
> **Impressões → Cliques → Leads (conversas iniciadas)**.

## Lead = conversa iniciada

Esta conta não tem aba de Conversas/qualificação. Cada **conversa iniciada**
(coluna `Messaging Conversations Started` da aba de Meta Ads) vira um **lead**,
herdando data/campanha/conjunto/anúncio da linha. Não há critério de MQL
(`has_mql = False` em `build.py`).

## Fontes de dados (somente leitura)

Planilha central `Extração dashboard atualizado`
(`1MnBVUg6ZdmR3FsUPy6ppjCAGno5moQJkUWDTrd-5POQ`):

| Aba | gid | Uso |
|-----|-----|-----|
| Página1 (Meta Ads) | `0` | única aba: `Day` · `Campaign Name` · `Ad Set Name` · `Ad Name` · `Impressions` · `Link Clicks` · `Messaging Conversations Started` · `Amount Spent`. Fonte de gasto/impressões/cliques **e** dos leads (conversas iniciadas). |

O build lê essa aba via **export CSV público** (`.../export?format=csv&gid=0`).
**Nada é escrito de volta** na planilha.

---

## Arquitetura

```
cron-job.org  ──(POST workflow_dispatch a cada 30 min)──▶  GitHub Actions
                                                              │
                          build/build.py  lê o CSV   ◀────────┘
                                 │  gera leads das conversas iniciadas
                                 ▼
                          dist/index.html  ──▶  deploy  ──▶  GitHub Pages (URL pública)
```

- `build/build.py` — baixa o CSV do Meta Ads, gera `dist/index.html`.
- `build/template.html` — layout/gráficos/tema (Chart.js via CDN).
- `.github/workflows/deploy.yml` — roda o build e publica no Pages.

**Cache-bust:** a página usa `Cache-Control: no-cache`, mostra o horário do último
build, tem botão **Atualizar** e se recarrega sozinha (`?t=timestamp`) ~30 min após
aberta — sempre pegando a versão mais nova.

## Rodar localmente (opcional)

```bash
python build/build.py --out dist/index.html            # busca o CSV ao vivo
# ou, com arquivo local para teste:
python build/build.py --meta-file meta.csv --out dist/index.html
```

---

## Ativação (uma vez) e cron-job.org

O disparo por `workflow_dispatch` só funciona quando o workflow está na branch
**`main`**. Veja **`SETUP-CRON.md`** para o passo a passo e os valores exatos
(URL, headers e body, com marcadores a preencher) a colar no cron-job.org.

> ⚠️ **Segurança:** nunca comite tokens no repositório. Gere um token
> *fine-grained*, só com **Actions: read/write** neste repositório, e use-o
> apenas no cron-job.org (ou em GitHub Secrets, se aplicável).

## Como usar este template para um novo cliente

Veja o **CHECKLIST DE NOVO CLIENTE** no topo de `CLAUDE.md` (ou `AGENTS.md`) e
o passo a passo completo em `GUIA-REPLICACAO.md`.
