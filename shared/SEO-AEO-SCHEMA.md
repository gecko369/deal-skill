# SEO-AEO-SCHEMA: Structured content standards for livethecelebrationlife.com

Read this file when creating any blog post for this site. It layers on top of the universal METHODOLOGY and the per-type content-types/blog.md.

**Scope of this file:**
- Schema markup strategy (what to do, what can't be done in BD, how to compensate)
- Key Takeaways box specification
- Definition blocks for AEO (Answer Engine Optimization)
- Tables for data-rich content
- Mid-article CTAs
- AEO content patterns for Google AI Overviews, ChatGPT, and Perplexity citation
- GEO (Generative Engine Optimization) principles for 2025-2026

---

## Part 1: Schema markup strategy

### What BD's Froala editor allows

BD's Froala editor strips `<script>` tags from `post_content`. This means JSON-LD cannot be injected per-post through the content API. The content skill does NOT attempt to inject schema tags into `post_content`.

### What the content skill DOES for schema

The skill produces structured HTML that communicates schema semantics without the JSON-LD wrapper. This is sufficient for Google to extract schema-adjacent signals and for AI engines to cite content:

| Structural element | Schema signal it communicates |
|---|---|
| Question-shaped H2 + direct-answer paragraph | `FAQPage.mainEntity.Question` + `Answer` |
| Definition block with explicit "What is X?" H3 | `DefinedTerm` + `Description` |
| Key Takeaways box (bulleted) | `Article.abstract` / structured summary |
| Table with headers | `Table` / dataset signal |
| Byline + publish date in meta fields | `Article.author` + `datePublished` |
| Answer-first opening paragraph | `Article.abstract` / featured snippet target |

### Theme-level schema (implement once in BD admin)

To get full JSON-LD benefits, the site owner should implement this at the BD theme level. This is a one-time developer task, not a per-post content task. Recommended schema types for a community blog:

**BlogPosting (for each blog post page):**
```json
{
  "@context": "https://schema.org",
  "@type": "BlogPosting",
  "headline": "{{post_title}}",
  "description": "{{post_meta_description}}",
  "image": "{{post_image_full_url}}",
  "datePublished": "{{post_date_iso}}",
  "dateModified": "{{post_modified_iso}}",
  "author": {
    "@type": "Person",
    "name": "Brad Salyers",
    "url": "https://celebrationlivingrealestate.com"
  },
  "publisher": {
    "@type": "Organization",
    "name": "The Celebration Life",
    "url": "https://www.livethecelebrationlife.com",
    "logo": {
      "@type": "ImageObject",
      "url": "https://www.livethecelebrationlife.com/[logo-url]"
    }
  },
  "mainEntityOfPage": {
    "@type": "WebPage",
    "@id": "{{post_full_url}}"
  }
}
```

**FAQPage (add to blog posts that contain a FAQ block — can be conditional in template):**
```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "{{faq_question_1}}",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "{{faq_answer_1}}"
      }
    }
  ]
}
```

**Note to site owner:** Raise this with your BD developer or check Admin → Website Settings → Custom Scripts for a head-tag injection point. BD's Developer Hub may have template variable documentation.

---

## Part 2: Key Takeaways box

### Purpose

The Key Takeaways box is the single most impactful structural addition for:
1. **Featured Snippet capture** — Google frequently pulls bullet lists near the top of an article for the "Key points" module
2. **AI Overview citation** — ChatGPT, Perplexity, and Google AI Overviews cite structured bullet summaries more often than prose paragraphs
3. **Reader retention** — skimmers see value immediately and read on

### Specification

**Placement:** Immediately AFTER the opening paragraph. BEFORE the first H2.

**Format (Bootstrap-safe, Froala-safe):**

```html
<div class="alert alert-info" role="note">
  <p><strong>Key Takeaways</strong></p>
  <ul>
    <li>[Standalone factual claim — specific, citable, 10-25 words]</li>
    <li>[Standalone factual claim — specific, citable, 10-25 words]</li>
    <li>[Standalone factual claim — specific, citable, 10-25 words]</li>
  </ul>
</div>
```

**Content rules:**
- 3 takeaways default; 4 maximum
- Each takeaway must be a standalone, self-contained factual claim or actionable insight that would make sense even out of context
- Takeaways are NOT teasers ("We'll explain more below"), NOT restatements of the headline, NOT vague benefits ("A great place to eat")
- Every takeaway must be supported by content later in the post — no orphan facts
- Include specific numbers, names, or qualifiers: "8AM-1PM every Sunday" not "most Sunday mornings"
- No banned vocabulary (ANTI-SLOP), no em-dashes, no smart quotes

**Example — "Things to Do in Celebration FL on a Rainy Day":**

```html
<div class="alert alert-info" role="note">
  <p><strong>Key Takeaways</strong></p>
  <ul>
    <li>Celebration Health (AdventHealth) offers an indoor aquatic center accessible to the public via day pass during non-peak hours.</li>
    <li>The AMC 24 at The Loop in Kissimmee (15 minutes from Celebration) is the closest major movie theater with full-service options.</li>
    <li>Several Front Street restaurants in the Celebration town center offer extended hours on rainy days due to heavier indoor foot traffic.</li>
  </ul>
</div>
```

---

## Part 3: Definition blocks (AEO pattern)

### Purpose

Definition blocks are the #1 pattern for capturing AI Overview and Perplexity citations on "What is X?" queries. Celebration FL has many terms that local prospects don't know: CROA, CDD, HOA vs. COA, fee simple, title insurance, etc. A well-structured definition block captures that query.

### When to use

Use a definition block when the post introduces a term, acronym, fee structure, or concept that:
- A prospective buyer or new resident would likely Google
- Has a precise definition that can be stated in 2-4 sentences
- Appears in the post's H2 section where it's first relevant

### Format

```html
<blockquote>
  <p><strong>What is [Term]?</strong></p>
  <p>[Definition — 2-4 sentences. Answer-first. Specific. Source-grounded. Includes the term's official name, scope, and one practical implication for the reader.]</p>
</blockquote>
```

### Rules

- One definition block per unique term. Never chain multiple definitions in sequence.
- The `<strong>` label must follow the exact pattern "What is [Term]?" — this matches the verbatim query format AI engines extract.
- The definition paragraph opens with the answer (what it is), then scope (where it applies), then practical implication (why the reader should care). No preamble.
- No em-dashes. No smart quotes. No banned vocabulary. No passive voice.

### Example

```html
<blockquote>
  <p><strong>What is CROA?</strong></p>
  <p>CROA stands for Celebration Residential Owners Association, the master homeowners association that governs the entire community of Celebration, FL. Every property in Celebration — single-family homes, condos, and townhomes alike — is subject to CROA's covenants, conditions, and monthly fees. CROA fees fund private parks, nature trail maintenance, common area landscaping, and Celebration's community programming. They are separate from any sub-association fees tied to specific neighborhoods like Artisan Park or Aquila Reserve.</p>
</blockquote>
```

---

## Part 4: Tables

### Purpose

Tables score high in Google's Featured Snippets for comparison and list queries. They are also structured data that AI engines extract reliably. A table with clear headers that answers a "comparison" query can rank independently of surrounding prose.

### When to use tables

| Use a table | Use prose or a bulleted list instead |
|---|---|
| Comparing 3+ options across 2+ attributes | Comparing 2 options (write a "vs" paragraph) |
| Displaying specifications (fees by neighborhood, hours by day) | Listing items with no comparison dimension |
| Ranked or scored data (restaurants by cuisine, schools by rating) | 3 or fewer items |
| Market data organized by time period or category | Sequential steps (use numbered list) |

### Format (Bootstrap table — Froala-safe)

```html
<table class="table table-bordered table-hover">
  <thead>
    <tr>
      <th>[Column 1 Header]</th>
      <th>[Column 2 Header]</th>
      <th>[Column 3 Header]</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>[Value]</td>
      <td>[Value]</td>
      <td>[Value]</td>
    </tr>
    <tr>
      <td>[Value]</td>
      <td>[Value]</td>
      <td>[Value]</td>
    </tr>
  </tbody>
</table>
```

### Rules

- Column headers must be specific and scannable — "Monthly HOA Fee" not "Fee"
- Table cells should be concise — 1-10 words per cell; use Notes column for elaboration
- One table per post (default); two allowed for comparison or market-data posts
- Never place a table in the opening paragraph or conclusion
- Table must be preceded by a sentence introducing what it shows ("Here's how Celebration's main neighborhoods compare on HOA fees:")
- Every value in the table must be source-supported — no fabricated data

---

## Part 5: AEO content patterns (Answer Engine Optimization)

AEO covers how content is structured to appear in AI-generated answers (Google AI Overviews, ChatGPT web search, Perplexity, Bing Copilot). A local community blog wins AEO for hyper-local queries by being the most structured, entity-specific, and directly answering source.

### Pattern 1: Direct-answer opening paragraph (already required — reinforce)

The first paragraph MUST answer the post's central question within 80 words. Structure:

```
[Direct answer to the headline's implicit question in 1-2 sentences] [Scope of the post — what specific angle is covered] [1-2 sentence supporting context]
```

Never start with: "In this article we will...", "If you're wondering...", "Many people ask...", "Welcome to..." — these delay the answer and reduce AEO capture rate.

### Pattern 2: Question-shaped H2s

60% of H2s should be question-shaped to capture long-tail query variants. Structure:

- "How Much Are HOA Fees in Celebration FL?" — captures "$" query variants
- "What Schools Are in the Celebration FL Area?" — captures school-related queries
- "Is Celebration FL a Good Place To Live?" — captures the most common local community query

Mix with statement-shaped H2s for variety ("The Best Restaurants for Date Night in Celebration"). All in title case.

### Pattern 3: FAQ block optimization

The FAQ block (required per blog.md) should follow these AEO rules:

- Questions must be real search queries — check Google autocomplete and "People also ask" for the post's main topic + "Celebration FL"
- Every FAQ answer is 40-70 words max — longer and AI engines lose the extraction boundary
- Include the city name ("Celebration FL" or "Celebration, Florida") in at least 2 of the 5 FAQ questions
- Lead with the answer in every FAQ response — never hedge first ("It depends on...")
- FAQ block title: `<h2>Frequently Asked Questions About [Post Topic] in Celebration FL</h2>` — include the city for local signal
- Each Q is H3, each A is a `<p>` directly under it

### Pattern 4: Entity specificity

Every named entity in the post must include at least one specific attribute that makes it unambiguously identifiable. Generic references reduce AEO citation probability.

| Weak (generic) | Strong (specific) |
|---|---|
| "the HOA" | "CROA (Celebration Residential Owners Association)" |
| "the farmers market" | "the Celebration Farmers Market at Lakeside Park (Sundays, 8AM-1PM)" |
| "a local school" | "Celebration K-8 School at 400 Campus St, rated 9/10 on GreatSchools" |
| "the golf course" | "Celebration Golf Club, an 18-hole semi-private course on Golf Park Drive" |
| "nearby hospitals" | "AdventHealth Celebration (formerly Florida Hospital Celebration Health) at 400 Celebration Place" |
| "Central Florida location" | "6 miles from Walt Disney World, 20 miles from Orlando International Airport via FL-417" |

---

## Part 6: GEO principles (Generative Engine Optimization, 2025-2026)

GEO covers how AI systems (LLMs used in search) decide which web sources to summarize and cite in generated answers. For a local community blog, the key GEO signals are:

### Signal 1: Topical authority depth

A post that covers ONE topic with 5-10 specific, structured dimensions outperforms 10 posts that each cover 1 dimension superficially. For livethecelebrationlife.com, this means:

- A single "Complete Guide to the Celebration FL HOA" post beats 4 thin "HOA overview" posts
- Include everything: fees, what's covered, what's not, who governs CROA, how to appeal a decision, move-in one-time contribution, contact info, how to get involved

### Signal 2: Citation-readiness

AI engines prefer sources whose content can be extracted as discrete cited facts. Structure for citation-readiness:

- Lead paragraphs of each section contain the citable claim in the first sentence
- Numbers, dates, and proper nouns appear early in sentences (not buried in dependent clauses)
- Every data point is attributed or dated ("as of 2025" or "per the CROA 2025 disclosure")

### Signal 3: Freshness signals

For community blogs, freshness matters for event posts (obvious) but also for real estate and lifestyle content. Signal freshness by:

- Including the current year in the meta title when relevant: "Celebration FL HOA Fees Explained (2025-2026)"
- Noting when data was last confirmed: "Fee ranges current as of Q1 2026"
- Mentioning recent developments: "Following the 2026 groundbreaking of Ovation Orlando..."

### Signal 4: Internal coherence (avoid topic scatter)

GEO systems reward sites where multiple pages reinforce the same topical cluster. For this site, posts should actively cross-link within the Celebration FL topic cluster:

- Real estate posts link to lifestyle posts, and vice versa
- Event posts link to the "Things to Do in Celebration" category
- HOA posts link to the neighborhood guide, and vice versa

This is in addition to the internal link strategy already required in blog.md.

### Signal 5: Conversational but precise

AI systems trained on conversational queries reward content that sounds like a knowledgeable local answering a friend's question. The ANTI-SLOP.md voice target ("friend telling someone about a cool local thing") is already correct. Add:

- Use "you" and "your" naturally when addressing the reader directly in how-to and guide sections
- Include one practical "what to do first" or "where to start" recommendation per post — this matches the action-intent pattern AI systems are trained to satisfy
- Avoid institutional tone ("Celebration FL is home to a vibrant community...") — talk like you live there

---

## Self-check for SEO/AEO/GEO compliance (run before publish)

- [ ] Does the post open with a direct-answer paragraph (80 words max) that answers the headline's implicit question?
- [ ] Is there a Key Takeaways box immediately after the opening paragraph?
- [ ] Do ~60% of H2 headings use a question structure?
- [ ] Does every H2 section open with a 40-60 word direct-answer paragraph?
- [ ] Is there a FAQ block with 3-5 Q&A pairs, city name in at least 2 questions, answers under 70 words each?
- [ ] Are all named entities (schools, parks, HOA, streets) accompanied by at least one specific attribute?
- [ ] Is there one definition block for any term that readers might Google as "What is X?"?
- [ ] Is there one table for comparison or data content (if applicable to the post format)?
- [ ] Is there one mid-article real estate CTA button (placed between H2 sections, 40-60% through)?
- [ ] Is there one advertising CTA in the conclusion paragraph (prose, not a button)?
- [ ] Does the FAQ block heading include "Celebration FL" or the relevant city name?
- [ ] Are all data points attributed or dated?
- [ ] No em-dashes, no smart quotes, no banned vocabulary (ANTI-SLOP self-check passed)?
