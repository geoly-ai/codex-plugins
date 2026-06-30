# Geoly MCP — public / industry intelligence tools (23)

_Use this when doing cross-brand work (leaderboards, whitespace, momentum, AI-search demand, perception, shopping) or anything involving the locale convention._

The **cross-brand** surface: AI brand visibility at the category/topic level, not just the
customer's own monitoring. This is the competitive-intelligence layer — leaderboards,
whitespace, momentum, AI-search demand, perception, shopping.

## Access & gating

- **Plan**: Grow tier and above — plan IDs `grow | advanced | plus | enterprise` (the gate
  `canAccessMcp`; "Grow" is $149/mo, the old "Growth+" label). **Context**: single-org only
  (multi-org tokens cannot use public tools in v1). If a public tool is missing, the token's
  org lacks the tier or an active entitlement.
- Public tools read the **`public_*` canonical model** — a *separate collection pipeline*
  from the brand's own monitoring. The same brand's numbers can differ between the self
  surface and the public surface; never reconcile one against the other.

## Locale convention (read this first)

Most public responses are wrapped as:

```jsonc
{ "localeUsed": {"country":"US","language":"en"},
  "defaulted": true,          // true = your requested locale fell back to US/en
  "data": { ... },
  "availableLocales": [ ... ] // present on resolve/list calls
}
```

- `country` / `language` params are usually **optional** and default to `US` / `en` — except
  `compare_public_brands`, where `country` **and** `language` are **required** (comparisons
  must share one locale to be valid).
- If `defaulted=true`, tell the user the data is the fallback locale, then offer to re-run with
  a locale from `availableLocales`.

## IDs: how to get them

Start from a name/domain, not an ID. `search_public_entities` resolves free text → brand /
category / topic IDs and slugs. Then use the typed tool. `list_public_topics` and
`list_public_locales` help enumerate.

---

## 1. Resolve & browse (3)

| Tool | Purpose | Key params |
|---|---|---|
| `search_public_entities` | Free-text resolver: brand / category / topic name or domain → public IDs + slugs | `query (2–120 chars), limit (1–20, default 6)` |
| `list_public_topics` | Browse/paginate public topics, optional status/search filter | `page, page_size (1–100), country, language, status: draft\|active\|rejected\|archived, search` |
| `list_public_locales` | List valid `{country, language}` pairs for an entity | `entity_kind: content\|brand\|topic_slug\|product_space, entity_id` |

---

## 2. Topic tools (9)

| Tool | Purpose | Key params |
|---|---|---|
| `get_public_topic_overview` | Overview of one public topic | `topic_id \| slug, country, language` |
| `get_public_topic_brand_leaderboard` | Brand leaderboard ranked by Share of Mention (absolute 1-based rank) | `topic_id, platform_id, date_from, date_to, page, page_size (1–100)` |
| `get_public_topic_som_trend` | Daily SoM trend (tracks #1 leader by default, or a specific `brand_known_id`) | `topic_id, brand_known_id, date_from, date_to, max_points (2–90)` |
| `get_public_topic_prompt_matrix` | Prompt × brand heatmap (per-prompt SoM %) | `topic_id, top_brands (1–20), max_prompts (1–200)` |
| `get_public_topic_prompt_detail` | One prompt: metadata, per-brand breakdown, recent records w/ snippets, top citation domains | `prompt_id, record_limit (1–50), citation_limit (1–50)` |
| `get_public_topic_record_detail` | One public record (AI answer): metadata, snippeted answer (~8k), capped citations/sources/shopping, brands mentioned | `record_id` |
| `get_public_topic_citation_domains` | Citation-domain leaderboard for the topic (most-cited domains + rates) | `topic_id, limit (1–200)` |
| `get_public_topic_commerce` | Commerce aggregate: activation rate, price stats, retail channels, price bands, products | `topic_id` |
| `get_topic_competition_difficulty` | AI-visibility **difficulty 0–100** (like SEO keyword difficulty) — percentile of CR3 within category | `topic_id \| prompt_id \| product_space_id, country (default US), language (default en)` |

---

## 3. Brand tools (4)

| Tool | Purpose | Key params |
|---|---|---|
| `get_public_brand` | One public brand across topics, **faceted** | `brand_id, view, country, language, limit (1–200), include_trend (bool), days (2–90)` |
| `get_public_brand_perception` | AI perception profile: canonical aspects + polarity + evidence + `hasEnoughSignal` | `brand_id, country, language, limit (1–100), min_mentions_per_aspect` |
| `get_public_brand_perception_aspect_mentions` | Drill-down: source mentions behind one perception aspect | `brand_id, normalized_label, country, language, limit (1–200)` |
| `compare_public_brands` | Side-by-side of **2–4** brands on one facet, **shared locale required** | `brand_ids[] (2–4), view, country (required), language (required), limit (1–200), include_trend (bool), days (2–90)` |

`get_public_brand` / `compare_public_brands` `view` facets: `overview` (default) ·
`visibility` · `footprint` · `competitors` · `citations` · `category_ranking` · `revenue` ·
`shopping`. (`compare_public_brands` defaults to `visibility`.)

---

## 4. Category / product-space (3)

| Tool | Purpose | Key params |
|---|---|---|
| `get_public_category` | One product-space category, **faceted** | `view (default overview), slug (required for overview), product_space_id (required for other views), country (default US), language (default en), limit (1–200), topic_ids[] (≤500), public_brand_id, date_from, date_to, max_points (2–90)` |
| `get_category_whitespace` | Opportunity map for a subject brand: classifies topics into strengths (covered/leading/close/defend) vs opportunities (prioritize/gap/watch) | `product_space_id, public_brand_id, country (default US), language (default en), topic_ids[] (≤500), limit (1–50)` |
| `get_category_brand_momentum` | Period-over-period SoM change ranking: risers vs fallers, with window bounds | `product_space_id, country (default US), language (default en), days (2–45, default 7), date_to, public_brand_id, topic_ids[] (≤500), limit (1–50)` |

`get_public_category` `view` facets: `overview` · `brand_leaderboard` · `som_trend` ·
`topics` · `citation_domains` · `recent_mentions`.

---

## 5. AI-search query intelligence (2)

| Tool | Purpose | Key params |
|---|---|---|
| `get_public_search_queries` | AI-search demand for a product-space, **faceted** | `mode (default queries), product_space_id (required except `product_spaces` mode), country (default US), language (default en), page, page_size (10–100, default 20), search, coverage: all\|2plus\|3plus\|5plus, sort` |
| `get_public_search_query_detail` | Drill-down on one query (brands + prompts + top sources) or one theme | `detail_mode: query\|theme, query (query mode), product_space_id (query mode), topic_id (theme mode), country (default US), language (default en)` |

`get_public_search_queries` `mode` facets: `queries` (default) · `themes` ·
`brand_landscape` · `prompt_map` · `product_spaces`.

---

## 6. Shopping (2)

| Tool | Purpose | Key params |
|---|---|---|
| `list_public_shopping_products` | Shopping overview for a product-space slice: counts, paginated grid, top channels, price bands | `product_space_id, country (default US), language (default en), days (1–180, default 30), search, page, page_size (1–100)` |
| `get_public_shopping_card_detail` | Full detail for one product card: header, evidence, up to 20 topics/prompts/retail offers | `card_key, product_space_id, country, language, days (1–180, default 30)` |

---

## Competitive-intelligence playbook (question → tool chain)

- **"How do we rank in <category> and who leads?"**
  `search_public_entities` (resolve category + my brand) → `get_public_category` view=`brand_leaderboard`
  → `get_public_brand` view=`category_ranking`.

- **"Where should we invest — what topics can we win?"**
  `get_category_whitespace` (subject = my `public_brand_id`). Act on the `prioritize` / `gap`
  buckets; `defend` = leads at risk.

- **"Who's gaining/losing share lately?"**
  `get_category_brand_momentum` (risers vs fallers over `days`). Drill a riser with
  `get_public_topic_som_trend`.

- **"What is AI actually being asked in our space, and who wins those answers?"**
  `get_public_search_queries` mode=`queries` (or `brand_landscape`) →
  `get_public_search_query_detail` detail_mode=`query` for the winners + sources of one query.

- **"How does AI describe us vs a rival?"**
  `compare_public_brands` view=`visibility` (or `competitors`), **country+language required** →
  `get_public_brand_perception` per brand → `get_public_brand_perception_aspect_mentions` for evidence.

- **"Is this topic worth targeting?"**
  `get_topic_competition_difficulty` (0–100; higher = more concentrated/harder) +
  `get_public_topic_brand_leaderboard` to see who holds it.

## Caveats specific to public tools

- **Separate pipeline**: do not reconcile public numbers against the self-surface
  (`get_brand_overview`) — different collection, possibly different cadence.
- **Locale defaulting**: always check `defaulted`; surface it.
- **Truncation**: public responses cap at ~120k chars (`[truncated: …narrow your query]`).
  Narrow by topic/locale/date or paginate.
- **Ranks are absolute** (1-based leaderboard position), not relative to the returned page.
