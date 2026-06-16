# Blog content-type protocol

The router (`SKILL.md`) routed you here because the user wants to create blog post(s). Follow this file plus the shared protocol files.

## Required reading first

1. `../shared/METHODOLOGY.md`: universal protocol.
2. `../shared/ANTI-SLOP.md`: voice + pattern bans + self-check.
3. `../shared/URL-PATTERNS.md`: internal URL construction.

---

## End-to-end runbook

The user invoked the skill with a goal like "write blog articles for SEO," "write a viral piece for my industry," or "write an article about XYZ." Execute the runbook steps in order. Once a step is resolved, move immediately to the next step. **Only make the tool calls each step specifies — no extras.** On per-post failure, continue to the next post.

1. **Mode detection.** Per METHODOLOGY `Mode detection`.
2. **Site context discovery.** Run METHODOLOGY `Stage 1: Site context`.
3. **Post-type discovery.** Run the `Post-type discovery` section.
4. **Author resolution.** Run METHODOLOGY's `Author resolution (universal pattern)` against the resolved `data_id`.
5. **Build the topic pool.** Run the `Topic resolution` section. Pool size `N=5`.
6. **Apply pool discipline.** Apply METHODOLOGY's `Candidate pool discipline (universal pattern)`.
7. **Duplicate detection.** Run METHODOLOGY `Stage 2: Duplicate detection`. Run the `Dedup` section for blog-specific match criteria.
8. **Source research per topic.** Run METHODOLOGY `Stage 3: Source research`. Run the `Source research` section. Land 3-5 source-supported angles BEFORE drafting.
9. **Category routing.** Run METHODOLOGY `Stage 4: Category routing`. Run the `Category routing` section for blog-specific authorization.
10. **Image selection — FEATURE image only at this step.** Run METHODOLOGY `Stage 5: Content manufacture (universal)` → `Image strategy` end-to-end: Topic-fit gate → extension filter → `getImageDimensions` orientation gate (landscape only) → dedup. The sequencing rules + retry behavior are defined there; follow them exactly. Lock the feature image first — re-doing body content when an image fails dedup is the expensive path. Inline body images are opt-in only — see the `Inline body images` section.
11. **Image dedup (FEATURE).** Per METHODOLOGY `Stage 5: Content manufacture (universal)` → `Image strategy` dedup step. For blog: `listSingleImagePosts property=original_image_url property_value=<URL1,URL2,URL3> property_operator=in`.
12. **Internal link research.** Before drafting, search for existing posts on this site to use as internal links in the body. Run `listSingleImagePosts` filtered by the blog `data_id` using 3-4 keyword searches derived from the post topic (e.g. a post about Celebration restaurants: search "restaurants", "dining", "food", "town center"). Capture up to 5 confirmed post slugs and titles. These are the primary internal link targets for body prose — link to them naturally where the content mentions a related subject. If no related posts exist, link to the blog listing page (`/blog`) and the relevant category page. Never fabricate post URLs — only link to URLs confirmed by this lookup.
13. **Content manufacture.** Follow METHODOLOGY `Stage 5: Content manufacture (universal)`; this file adds blog-specific shape (post-format templates, answer-first H2s, FAQ block, internal-link density). Inline body images are NOT default; only apply per the `Inline body images` section when the user explicitly requests them.
14. **Create the post** via `createSingleImagePost` with the field set in the `BD Blog field reference` section.
15. **GHL Social Post.** Run the `GHL Social Post` section at the bottom of this file.

### Interactive-mode question order

When running interactive, ask the user in this canonical order. One question at a time. Wait for each answer:

1. **Post-type** (if runbook Step 3 found multiple blog-flavored post-type candidates)
2. **Topic input** ("What's the article about? Or do you want me to suggest topics for SEO traffic in your vertical, or write a piece designed to go viral for your industry?")
3. **Author** — per METHODOLOGY `Author resolution (universal pattern)`
4. **Categories / vertical filter** (if not pre-specified)
5. **Post format** ("How-to, listicle, pillar/comprehensive, news/announcement?" — or autonomous default by topic shape)
6. **Publish vs draft** ("Publish live, or save as drafts for your review?")

Skip any question the user already answered in the original request.

---

## Post-type discovery (runbook Step 3)

Resolve by user intent first, then canonical markers, then semantic match.

1. **User named a post type explicitly** (e.g., "post to my 'Tips for Homeowners' section"). Match the user's phrase against `data_name`, `system_name`, `form_name` on `listPostTypes`. Single confident match wins — skip steps 2-3.

2. **User didn't specify** — try in order, stop at first match. Server-side filter via `listPostTypes` — do NOT `getPostType` per-candidate:
   a. `system_name=website_blog_article` (BD canonical)
   b. `form_name=blog_article_fields` (canonical blog form)
   c. `data_type=20` + semantic match on `data_name`/`system_name` (blog, news, journal, insights, resources, articulo, noticia, nachrichten, artikel)

3. **EXCLUDE from any blog resolution:**
   - `community_article` / `form_name=member_article_fields` — member-written, NOT site-owner blog
   - `coupon`, `soundcloud_post`, `discussion`, `event`, `job_listing` — different content types

**`type_of_feature` is NOT a blog marker.** Reserved for events (`1`), properties (`2`), digital products (`0`). Blogs are `type_of_feature=null`.

**Decision after resolution:**

| Match count | Action |
|---|---|
| Zero | Skill cannot run. Surface clean message, exit. |
| One | Use it. Cache `data_id`, `data_name`, `system_name`, `form_name`. |
| Multiple, interactive | Ask the user. List by data_id + data_name. |
| Multiple, autonomous | If the user pre-specified a post-type id, use it. Else exit with clear audit message. |

User's explicit post-type pick always wins.

---

## Topic resolution (runbook Step 5)

### Shape A — User-specified topic

User said "write about XYZ" or "draft an article on ABC." Use the topic verbatim. Skip vertical brainstorming. Run source research for that exact topic.

### Shape B — Vertical-derived (user picks no topic)

User said "write articles for SEO traffic," "organic search," "viral content," "industry news," "related to a topic," "trending content," or similar — anything that means "you pick the topic." Brainstorm `N` distinctly different topic candidates cached from **Site context discovery**.

**Within-pool diversity — span distinct subjects.** Each candidate must occupy its own sub-theme of the vertical. If two or more share a sub-theme, anchor noun, focus, or subject, regenerate with broader spread before taking #1.

**If user signaled viral/trending intent**, also pull `WebSearch` for trending discussions/news in the vertical (last 30-60 days).

**Topic bar (Shape B).** Frame each candidate for a non-expert outside the niche while keeping specific qualifiers (audience segment, geographic context, use case, life stage). Compounded specificity, not one. **Specific ≠ jargon** — the qualifier should be a real audience or scenario a reader outside the niche can picture (marathon runner, ACL recovery, desk worker), not insider terminology or acronym strings (mid-cycle loading, conjugate periodization, eccentric utilization ratio, NASM vs ACE vs NSCA). Pivot examples: "TPO vs EPDM Roof Membranes" → "The Best Roofing Materials for Residential Homeowners in Cold Climates". "IRC §179 vs §168(k) Deductions" → "Which 2026 Tax Deductions Save Sole Proprietors the Most?"

**Topic depth (Shape B) — go specific, not safe.** Default LLM move is the broadest possible framing ("How Much Protein to Build Muscle"). That competes against millions of existing articles and ranks for nothing. Go two or three specificity layers deeper on each candidate:

**Bad Broad versus Good Specific — across title shapes** (each row a different shape AND a different vertical — read the broad→specific transformation and the variety of framings, not the topic). Vary the framing across your `N` candidates; do not open all of them with "How"/"What"/"Why".

| Title shape | What it does | Too broad (Bad LLM default) | Good (specific, in that shape) |
|---|---|---|---|
| Imperative | Command, verb-first, promises an outcome | Dog Training Basics | Stop a Rescue Dog From Pulling on Walks in Its First Two Weeks Home |
| How-to | Explicit instruction | Roof Repair Tips | How to Tell If a Hail-Damaged Roof Needs Full Replacement or a Patch |
| Question | Poses the reader's query | Choosing a Lawyer | Do You Need a Lawyer to File for Custody in a No-Fault State? |
| Listicle / number | Counted set | Saving for Retirement | 5 Retirement Accounts a Freelancer Should Open Before Age 40 |
| Declarative / statement | Asserts a claim or truth | Electric Cars | Heat Pumps Are Quietly Replacing the Gas Furnace in Cold Climates |
| Noun-phrase / definitional | Names the subject, no verb | Wedding Photography Ideas | The Real Cost of a Second Shooter for a Full-Day Wedding |
| Comparison / vs | Pits two options against each other | Types of Mattresses | Memory Foam vs Latex for Side Sleepers With Back Pain |
| Guide / explainer | "The complete/beginner's" framing | Houseplant Care | A Beginner's Guide to Keeping Fiddle-Leaf Figs Alive Through Winter |

Specificity layers: audience segment + scenario + format. The qualifiers ARE the specificity — broad reader-appeal framing AND specific qualifiers are not opposites. Each narrows the long-tail query. Broad topics still ship occasionally — but the default is specific.

**Pick qualifiers that match real search intent** — what readers actually query, not a narrowing that sounds clever to a strategist.

**Never bulk-list existing posts to "understand coverage" before picking a topic.** The per-candidate query in the `Dedup` section catches real overlaps; pre-scanning the feed adds nothing and burns reads on sites with hundreds of posts. Pick topics from vertical/category signals (Shape B above), then let dedup do its job at the per-candidate stage.

---

## Source research (runbook Step 8)

Per METHODOLOGY `Stage 3: Source research`, with one adjustment: the **Date sanity gate does NOT apply** to blog source research. Blogs are evergreen; sources can be from any date.

**Blog-specific source candidate buckets:**

- Industry trade publications, professional association sites
- Established expert blogs / personal sites in the vertical
- Mainstream press and vertical-relevant culture/lifestyle magazines
- Government / academic research, public health/data agencies, university extension publications
- Peer-reviewed studies / official journal sites (for science/medical/legal topics)
- Reputable podcast transcripts, interview shows, popular vertical Substacks
- Real practitioner interviews / case studies on public-facing pages

---

## Dedup (runbook Step 7)

Per METHODOLOGY `Stage 2: Duplicate detection`. Blog-specific match criteria:
- Title: semantic match (not string-exact).
- Topic angle: semantic overlap on the core thesis/angle, not just shared keywords.
- Date: NOT a dedup factor (blogs are evergreen).

---

## Category routing (runbook Step 9)

Per METHODOLOGY `Stage 4: Category routing`. Blogs use the post type's `feature_categories` (cached from `Stage 1: Site context`).

Authorization:
- Interactive grant ("yes, create new blog categories") → skill respects for the run.
- User-specified default category in their request → every post in the run goes to that category.

---

## Content manufacture (runbook Step 13)

Follow METHODOLOGY `Stage 5: Content manufacture (universal)`: EEAT goal, Froala-safe HTML allowlist (from MCP corpus), link policy, image strategy, voice via ANTI-SLOP, self-check. Blog posts additionally follow the per-format and per-section rules in this section.

### Post format → target length

Pick one format per post; let topic shape decide. Apply the section + length guidance for that format:

| Format | Total words | When |
|---|---|---|
| How-to | 1500-2500 | Step-by-step instruction on accomplishing X |
| Listicle | 1200-2000 | "N ways to X," "Top N Y," "N best Z" |
| Pillar / comprehensive guide | 2500-4000 | Definitive long-form coverage of a topic |
| News / announcement | 600-1200 | Event/launch/update coverage |
| Comparison / vs | 1500-2500 | "X vs Y," "When to choose X over Y" |

### Body structure (universal across formats)

1. **Direct-answer opening paragraph.** First `<p>` answers the headline's implicit question in 40-100 words. No throat-clearing ("Here's the thing"), no preamble. Reader knows within ~80 words what they're getting and why.
2. **Question-shaped H2s for ~60% of sections.** "What is X?" "How does Y work?" "When should you Z?" — captures long-tail queries and AI-Overview citations. Mix in statement-shaped H2s for variety where natural.
3. **Answer-first paragraph per H2.** Every H2 opens with a 40-60 word direct answer to its implicit question. Then expand with detail, examples, lists.
4. **Paragraph cap: 40-80 words typical, 150 hard max.** Long walls of text fail mobile readability and AI-Overview extraction.
5. **Sentence cap: ~15-20 words typical.** Tighter sentences read cleaner.
6. **List shape per ANTI-SLOP `Bullets rule`.** Numbered for sequence (how-to steps), bulleted for parallel items (listicle entries, comparison criteria).
7. **FAQ block before conclusion.** H2 "Frequently Asked Questions" (or per-language equivalent) with 3-5 H3 questions, each answered in 40-60 words. High AI-citation density per word.
8. **Conclusion 100-150 words.** Advance the reader to a next step or a fresh specific that wasn't in the body — never restate the body's load-bearing answer. Close with ONE internal link (CTA shape — "Browse {Category} listings on {site}" or "See more {topic} resources" — anchor text reads as part of a sentence).

---

### VOICE: Write like a neighbor, not a search result (mandatory for all posts on this site)

This is the most important instruction in this file. Everything else — SEO structure, CTAs, schema signals — is secondary to getting the voice right.

**The target voice:** A Celebration FL resident who genuinely loves this community, knows it well, and is texting a friend who just asked "hey, where should I get coffee around here?" — except it's a blog post, so they have space to be thorough. They are not a journalist. They are not a business directory. They have opinions. They have favorites. They notice things.

---

#### Pattern 1: Lead with a recommendation, not a description

The article's job is to save the reader time and help them make a good decision. Do that immediately.

**Wrong (describes):**
> *"Celebration FL has a small but well-rounded coffee scene centered on the walkable town center along Market Street and Front Street near Lake Rianhard."*

**Right (recommends):**
> *"If someone new to Celebration asked me where to grab coffee on a Sunday morning, I'd tell them to walk to Fortuna Bakery on Market Street — and then send them down to CFS on Celebration Blvd if they want the best latte in the ZIP code. That's really the whole guide, but here's the full picture."*

The "right" version has an opinion in the first sentence. The reader immediately knows where to go. Everything after that is supporting context.

---

#### Pattern 2: One thing only a local would know, per place

For every business, park, neighborhood, or attraction covered, include one specific observation that you'd only know if you'd actually been there — not sourced from a review site, not scraped from a menu. Something specific and concrete.

**Wrong (from a website):**
> *"Starbucks at Bloom Street opens at 6AM with outdoor seating overlooking Lake Rianhard."*

**Right (from someone who's been there):**
> *"The Starbucks at Bloom and Front has no drive-thru, which means the vibe is calmer than you'd expect. The rocking chairs on the Front Street promenade just outside are almost always taken by 8AM on weekends. If you want one, arrive early."*

The second version tells you something useful you couldn't get from Google Maps. That's what makes it worth reading.

---

#### Pattern 3: Give each place a clear verdict

Don't just describe — tell the reader what kind of person this place is right for. This is the editorial job the reader is paying you to do.

**Wrong (neutral):**
> *"CFS Coffee For the Soul is on Celebration Blvd with a Colombian-inspired menu including specialty lattes."*

**Right (verdicts):**
> *"CFS is worth the drive down Celebration Blvd — it's not in the walkable core, but it earns its 4.7-star rating. Get the Rainbow Latte. It sounds like it shouldn't work and then it does. This is the spot if you want something more intentional than a Starbucks run and you don't mind leaving Front Street for fifteen minutes."*

Verdict structure: What kind of person should go here → What to order or do → One honest caveat if there is one.

---

#### Pattern 4: Second person, present tense, action-oriented

Write directly to the reader as if they're deciding right now.

**Wrong (passive, past tense, third person):**
> *"Reviewers with French backgrounds consistently say the macarons are the real thing."*

**Right (direct, present, second person):**
> *"If you've had macarons in Paris and you're skeptical of American versions, Le Macaron on Front Street is the one that'll change your mind — or at least give it a fair shot."*

---

#### Pattern 5: Conversational transitions, not section headers acting as sentences

Don't use the H2 heading as the only transition between topics. Add a line that connects the previous section to the next one the way a person would in conversation.

**Wrong:**
*[end of Fortuna section]*
> *"..."*
*[H2: What About Afternoon Coffee?]*

**Right:**
*[end of Fortuna section]*
> *"Those are your morning options if you're staying in the town center. Once the afternoon hits, the crowd shifts and so does what's worth visiting."*
*[H2: Where to Go for Afternoon Coffee in Celebration FL]*

---

#### Pattern 6: The conclusion has a voice, not just a summary

The conclusion should feel like the end of a recommendation from a real person — not a recap paragraph.

**Wrong (summary + CTA dump):**
> *"The Celebration coffee scene is compact but serves the community well. Fortuna and Starbucks together handle most mornings..."*

**Right (genuine closing):**
> *"Honestly, you can't go wrong. Fortuna on a Sunday morning, CFS when you want to treat yourself, Starbucks when you need six AM and you need it now. Le Macaron at 9PM when you want something that feels like dessert without committing to dessert. The town center is small enough that you'll have your own regular within a month of moving here. If you own a business in the area and want to reach the people who already call Celebration home, [advertising in The Celebration Life community](https://www.livethecelebrationlife.com/join) is a direct line to this audience."*

---

#### What this voice does NOT mean

- Do not use banned vocabulary from ANTI-SLOP.md — "vibrant," "bustling," "hidden gem," "amazing," "charming," "nestled," "boasts" are all off.
- Do not manufacture enthusiasm for a place that doesn't deserve it. If a spot is just fine, say it's solid and move on. Credibility matters more than cheerleading.
- Do not let personality replace accuracy. The facts still need to be right. Warmth and precision work together.
- Do not editorialize about things you cannot verify. "Locals swear by the cold brew" should only appear if there is actual evidence (reviews, known reputation) — not as a generic device.

---

#### The test

Read the draft out loud. Ask: does this sound like someone who actually goes here and cares about this neighborhood? Or does it sound like a search result that learned to use first-person pronouns?

If you can picture someone reading it at their kitchen table, nodding along, and texting it to a friend moving to Celebration — it's right. If it reads like a Yelp page or a tourism website, rewrite it.

---

### Internal-link strategy

Blog posts link broadly across BD resources — this is where the SEO compounding lives. Budget **5-10 internal links per 2000 words**, distributed:

| Section | Recommended links |
|---|---|
| Direct-answer opening | 0-1 |
| Body H2 sections | 3-6 spread across sections (1-2 per major section, max) |
| FAQ block | 1-2 (answer text may include a link) |
| Conclusion | 1 (always — the CTA-shape closer) |

**Link targets — all valid for blog posts:**

- **Specific member profile** (Pattern 4): `/<user.filename>` — resolve via `searchUsers` or `listUsers property=email property_value=<email> property_operator=eq` only when the agent has a specific known person to deep-link to. No bulk-listing members.
- **Member directory landing** (Pattern 5): `/search_results` — links to the entire directory of members with no location or category filter applied. Location- + category-filtered member-search URLs are slug-hierarchy paths (out of scope for content skills, deferred to `/bd:seo`).
- **Specific post of any type** (Pattern 1): `/<post_filename>` — resolve via title-filtered `listSingleImagePosts` when the agent has a specific known post to deep-link to. No bulk-listing.
- **Post search results of any type** (Pattern 3): `/<post_type_data_filename>?category[]=<cat>&...` — for "more {category} {posts}" style anchors.
- **Post-type main listing** (Pattern 2): `/<data_filename>` — bare listing of all posts of that type.

Pick targets by **contextual relevance to the body sentence**. If the paragraph mentions finding a local pro, link to the member search filtered by the site's relevant category + the city named in the paragraph. If the paragraph references a related concept covered by another article on the site, deep-link to that article via Pattern 1 (but only if the agent has confirmed the article exists). Never fabricate URLs.

**Anchor text:** reads as part of the sentence. The linked phrase is a noun or noun phrase that belongs naturally in the surrounding prose ("certified personal trainers in Boston" not "click here for personal trainers"). Not a standalone CTA in the middle of paragraphs.

### Celebration-specific internal link inventory

For posts on this site, prioritize these internal link targets in addition to the BD Pattern 1-5 links:

**Always attempt to include at least one link to the blog itself:**
- Main blog listing: `/blog` — anchor text reads naturally ("more community news on The Celebration Life blog")

**Real estate cross-links (contextual, not mechanical):**
- All Celebration homes: `https://celebrationlivingrealestate.com/listing-report` — for "search homes" anchors
- Home valuation: `https://celebrationlivingrealestate.com/home-valuation` — for seller-signal content
- Market report: `https://celebrationlivingrealestate.com/market-report` — for market data content

**Do NOT over-link real estate.** One real estate link per post. The mid-article CTA button handles the primary real estate touch. Any additional real estate link in the body prose would be redundant.

---

### External-link strategy

External links to official local sources are required — not optional. They are a direct AEO/GEO signal that your content is fact-based and verifiable. AI citation systems (Perplexity, ChatGPT, AI Overviews) favor pages that connect entities to their official sources. A post about Celebration businesses that never links to any of them looks like an isolated, unverifiable content island.

**What to link to externally:**
- Official business websites (the business's own domain — not Yelp, not Google Maps)
- City and government sites (`celebration.fl.us`, `osceola.org`, `kissimmee.gov`)
- Official school pages (`osceolaschools.net` for district, individual school pages)
- Official organization pages (CROA, Celebration Foundation, AdventHealth)
- Official event pages (the organizer's own site — only use Eventbrite if the organizer has no dedicated event page)

**What NOT to link to externally:**
- Review aggregators (Yelp, TripAdvisor, Google Maps) as primary citations
- Competing community sites or local news outlets
- Social media profiles (Facebook pages, Instagram) — link to the website instead

**HTML format for all external links:**
```html
<a href="https://example.com" target="_blank" rel="noopener">anchor text</a>
```
Always `target="_blank"` (opens in new tab — keeps the post open) and `rel="noopener"` (security standard). Never `rel="nofollow"` for legitimate local businesses and organizations.

**First-mention rule — link once per entity, never again:**
Every external entity gets ONE link in the post — at its first mention only. All subsequent mentions of the same business, school, organization, or venue use natural short-form prose with no link.

- First mention of Celebration Brewing → link to `celebrationbrewing.com`
- Second mention → "the brewery" or "Celebration Brewing" — no link
- First mention of Celebration K-8 School → link to the school's official page
- Second mention → "the school" or "Celebration K-8" — no link

This mirrors the address rule: state it once, reference naturally after. The test: search the draft for any external URL. If it appears more than once, remove every instance after the first.

**Anchor text:** use the entity's proper name or a descriptive phrase that fits naturally in the sentence. Never "click here," never a bare URL.

- ✅ "...reserve a spot at [Celebration Golf Club](https://celebrationgolf.com)"
- ✅ "...check the [Celebration Town Center events calendar](https://celebrationtowncenter.com/upcoming-events/)"
- ❌ "...click here for more info"
- ❌ "...visit https://celebrationgolf.com"

---

### Key Takeaways box (required for all blog posts on this site)

Insert immediately AFTER the opening paragraph, BEFORE the first H2.

**Purpose:** Signals structured content to Google Featured Snippets, AI Overviews, and AEO engines. Gives skimmers a reason to stay.

**Format (muted warm styling — Froala-safe inline styles):**

```html
<div style="background-color:#f5f4f0; border-left:4px solid #b0a898; padding:16px 20px; margin:1.5em 0; border-radius:3px;">
  <p><strong>Key Takeaways</strong></p>
  <ul>
    <li>[Standalone factual claim or actionable point, 10-20 words]</li>
    <li>[Standalone factual claim or actionable point, 10-20 words]</li>
    <li>[Standalone factual claim or actionable point, 10-20 words]</li>
  </ul>
</div>
```

**Rules:**
- Each takeaway is a self-contained, citable fact — not a teaser or headline re-statement.
- 3 takeaways default; 4 allowed if the post genuinely has 4 strong distinct points.
- Source-backed. If a takeaway makes a claim, that claim appears in the body with support.
- No banned vocabulary. No em-dashes. No rhetorical openers.
- Do NOT use Bootstrap's `alert alert-info` class — use the inline style above only.

---

### Entity mention limits — addresses, numbers, and contact details

**Hard rules:**

- **Physical addresses:** State the full address ONCE — at the first introduction only. All later references use a natural short form ("the shop," "their Front Street spot"). Never repeat in a later section, FAQ, or conclusion.
- **Phone numbers:** Omit entirely for dining, retail, coffee, and lifestyle posts. Once only for medical, legal, or service businesses.
- **Hours of operation:** State ONCE per business. May appear in FAQ only if the question is genuinely about hours.
- **Prices:** State a specific dollar figure ONCE. Later references use relative language.
- **Distances and drive times:** State ONCE anywhere in the post.

**For multi-business roundup posts:** Consolidate addresses, hours, and price ranges into ONE summary table near the end — before the FAQ. Do not repeat this data in each H2 section. Each section covers the experience and the verdict; the table covers the logistics.

**The test:** Search the draft for any address or phone number. If it appears more than once, delete every instance after the first.

---

### Definition blocks (AEO pattern — use when the post introduces a term readers may not know)

**Purpose:** Definition-first structure is the highest-citation pattern for AI Overviews and Perplexity. When you define a term clearly and early, LLMs pull that definition as a cited answer.

**When to use:** Any post that introduces a term, acronym, or concept a local resident might Google as a question ("What is CROA?", "What is a CDD fee?", "What is a fee simple home?").

**Format (Bootstrap well — Froala-safe):**

```html
<blockquote>
  <p><strong>What is [Term]?</strong></p>
  <p>[Definition in 2-4 sentences. Specific. Source-grounded. Answer-first.]</p>
</blockquote>
```

**Placement:** At the top of the H2 section where the term is first introduced. The definition block is the answer-first paragraph for that section.

**Example:**

```html
<blockquote>
  <p><strong>What is CROA?</strong></p>
  <p>CROA stands for Celebration Residential Owners Association — the master homeowners association that governs the entire town of Celebration, FL. Every property in Celebration (single-family homes, condos, and townhomes) is subject to CROA fees and covenants. It is distinct from any sub-association fees tied to individual neighborhoods or developments within Celebration.</p>
</blockquote>
```

**Rules:**
- One definition block per term. Never chain multiple definitions back-to-back.
- Keep the H3 question label exact — "What is [Term]?" — this is the verbatim query pattern AI engines extract from.
- No em-dashes. No banned vocabulary.

---

### Tables (use for comparison, specification, or ranked data content)

**Purpose:** Tables score high on AI extraction, Featured Snippets, and AEO citation. A well-structured table can rank for a "comparison" query by itself.

**When to use:**
- Comparing two or more options (neighborhoods, home types, school options, dining styles, price tiers)
- Displaying specifications (HOA fee ranges by neighborhood, event schedules, fee structures)
- Ranking or scoring items (top restaurants by cuisine type, market stats by month)

**When NOT to use:**
- Lists of 3 or fewer items (prose works better)
- Content that flows better as a how-to progression

**Format (Bootstrap table — Froala-safe):**

```html
<table class="table table-bordered table-hover">
  <thead>
    <tr>
      <th>[Column Header 1]</th>
      <th>[Column Header 2]</th>
      <th>[Column Header 3]</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>[value]</td>
      <td>[value]</td>
      <td>[value]</td>
    </tr>
  </tbody>
</table>
```

**Placement:** Within the H2 section where it is most relevant. Never in the opening paragraph or conclusion. One table per post is the default; two allowed if the post format explicitly calls for it (comparison article, market data roundup).

---

### Mid-article soft CTA placement (required — 1 per post)

**Placement:** Between two H2 sections at the 40-60% point of the article. Never mid-paragraph. Never in the opening or FAQ.

**Format:** Per `prompts/blog.md` STEP 4 section B (real estate CTA table). Use the Bootstrap primary button format.

**Rules:**
- ONE mid-article CTA per post (the button). The advertising CTA in the conclusion is separate and different.
- The CTA must relate to the post's topic and audience. A dining post gets the "homes for sale in Celebration" default. A real estate post gets the valuation or market report link.
- No CTA text that says "click here." Anchor text must be descriptive and part of natural prose around the button.

---

### Inline body images

**Opt-in only — do NOT include inline body images by default.** Only apply this section when the user explicitly requests inline images in their prompt (e.g. "with inline images", "include body images", "add photos throughout"). Default blog runs ship with the feature image only — prose carries the post.

When opted in: 1 inline body image per 300-500 words (excluding the feature image). Image float: `class="fr-dib fr-fil img-rounded"` (left) or `class="fr-dib fr-fir img-rounded"` (right) + inline `style="width: 350px;"` on the `<img>`. Source URLs use the retina variant: `https://images.pexels.com/photos/<id>/pexels-photo-<id>.jpeg?w=700`. Per corpus `Rule: Post-body formatting`.

**Inline body image dedup (intra-post only):**
- No URL repeats within the same `post_content`.
- No body URL equals the post's own `post_image` (feature) URL.
- NO site-wide dedup on inline body URLs.

Each inline image is sourced via the Pexels workflow (corpus `Rule: Image URLs`). Vary the search topic per image so candidates differ naturally.

### Title shape

Blog titles run different from event titles — clickbait-flavored but anti-slop-disciplined. Pick a shape from the title-shape table in `Topic resolution`; vary the shape across the run rather than defaulting every title to "How"/"What"/"Why".

Caps: ~70 chars where SEO matters (Google truncates title tags around there). Keep punchy. No clickbait that overpromises ("This One Trick Will Change Your Life"). No throat-clearing. No fabricated curiosity. **Single statement only — no `X: Y`, no `X (Y)`, no `X? Y`.**

---

## AEO and GEO content patterns (for AI search citation readiness)

AEO = Answer Engine Optimization (ChatGPT, Perplexity, AI Overviews direct answers).
GEO = Generative Engine Optimization (how AI systems attribute, summarize, and cite sources).

### Why these matter for a local community site

A local Celebration FL community blog competes with Wikipedia and Visit Orlando in AI Overviews for broad queries. It wins for hyper-local queries like "What are the HOA rules in Celebration FL?" or "What restaurants are near Celebration town center?" by being the most specific, most structured, most entity-rich source.

### Entity specificity rules (Celebration FL)

These patterns directly improve AI citation probability:

| Instead of | Use |
|---|---|
| "the community center" | "Celebration's Lakeside Park pavilion" |
| "local schools" | "Celebration K-8 School (rated 9/10) and Celebration High School" |
| "the farmers market" | "the Celebration Farmers Market, held Sundays at Lakeside Park" |
| "nearby restaurants" | "Reggiano's on Front Street and Boheme's on Market Street" |
| "the HOA" | "CROA (Celebration Residential Owners Association)" |
| "central Florida location" | "ZIP code 34747, 6 miles from Walt Disney World and 20 miles from Orlando International Airport" |
| "a nice community" | "a master-planned community originally developed by The Walt Disney Company and opened in 1994" |

**Rule:** Every named entity should include at least one specific qualifier (address, rating, year, distance, fee amount, official name) that makes it unambiguously identifiable. Generic descriptions are the primary reason AI engines skip a source.

---

### Celebration factual knowledge (hard rules — never contradict these)

These are confirmed facts about Celebration FL. Never write anything that conflicts with them.

**Neighborhoods — always include ALL villages when listing Celebration neighborhoods:**
Spring Valley, North Village, West Village, South Village, Artisan Park, Aquila Reserve, Mirasol, and **Island Village** (the newest addition — frequently omitted by default, which is a factual error).

**Closed businesses — never reference as active:**
- **Cafe D'Antonio** — permanently closed. Do not mention as a current restaurant. It has been replaced by **Reggiano's** at the same Front Street location.

**Current dining on Front Street / Market Street:**
- Reggiano's (Front Street — replaced Cafe D'Antonio)
- Boheme's (Market Street)
- Celebration Brewing Company — also hosts regular events (`celebrationbrewing.com/events`)
- Seito Sushi
- The Marketplace

**Hotels in Celebration:**
- Bohemian Hotel Celebration (on Celebration Avenue, walkable from town center)
- Melia Orlando Suite Hotel at Celebration (on Celebration Blvd — full-service resort within Celebration)

**Key landmarks:**
- Lakeside Park — primary community park, Farmers Market every Sunday
- Lake Rianhard — lake at the heart of town center
- Front Street — primary dining/retail promenade
- Market Street — secondary retail street
- Celebration Town Center — the walkable commercial district

---

### Direct-answer paragraph structure (already required — reinforce here)

Every H2 section opens with a 40-60 word direct answer to its implicit question. This is the most important AEO pattern. Structure:

1. **H2 question:** "How Much Are HOA Fees in Celebration FL?"
2. **Answer-first paragraph (40-60 words):** "Monthly HOA fees in Celebration FL range from roughly $280 to $490 depending on the neighborhood and home type. These fees are paid to CROA (Celebration Residential Owners Association) and cover common area maintenance, private parks, nature trails, and the community's amenity access. Sub-community HOAs in newer developments like Artisan Park add $60-$150 on top."
3. **Expansion:** Then deeper detail, tables, comparisons, history.

### FAQ block structure for maximum AI citation

The existing FAQ block requirement (3-5 Q&A pairs) is correct. These additions improve AI extraction:

- **Q&A pairing must be true Q&A** — the question is a real search query someone types. The answer is a direct 40-60 word response. NOT an intro + "read more below."
- **Lead each FAQ answer with the answer, not a hedge.** "Yes, Celebration FL allows short-term rentals with restrictions..." not "That depends on several factors..."
- **Include the city name in at least 2 of the FAQ questions.** AI Overviews pull local FAQ blocks when city name appears in the question. Example: "Is Celebration FL a good place to raise a family?" not just "Is this a good place for families?"
- **FAQ H3 format — keep the question mark and write in title case:** `<h3>Is Celebration FL a Good Place To Live?</h3>`

### What NOT to include (GEO anti-patterns)

These patterns reduce AI citation probability:

- Vague geography ("the greater Orlando area," "Central Florida communities," "nearby towns")
- Generic lifestyle language without specifics ("a charming community," "a wonderful place to live")
- Unsourced price/statistic claims (cite the source or give a range with date)
- Walls of text longer than 150 words with no structural break (AI engines prefer scannable chunks)
- Redundant FAQ answers that restate the intro paragraph

---

## Schema markup note (implementation)

BD's Froala editor strips `<script>` tags from post_content. JSON-LD schema cannot be injected per-post through the API. The skill does NOT attempt to inject schema tags. Instead:

**What the skill handles:**
- HTML structure that communicates schema semantics (question H2s, answer-first paragraphs, FAQ Q&A, definition blocks, Key Takeaways box, tables)
- Entity specificity and factual precision (signals `Article` + `LocalBusiness` + `Place` schema adjacency)

**What must be done at the theme level (one-time BD admin setup):**
- Dynamic `BlogPosting` JSON-LD in the blog post template using BD template variables
- `FAQPage` schema JSON-LD generated from the post's FAQ block if the BD theme supports it
- `BreadcrumbList` schema for the blog URL path

**Recommended BD admin path:** Admin → Website Settings → Custom Scripts / Head Tags (if your plan includes it). Add the JSON-LD template there, using BD's dynamic variables for headline, datePublished, image, author, and url.

If a user asks the skill to include schema in a post directly, respond: "BD's Froala editor strips script tags from post content. Schema markup must be implemented at the theme level in BD admin. I've structured the post content with semantic HTML patterns that communicate the same signals to search engines and AI systems."

---

## BD Blog field reference (runbook Step 14)

What `createSingleImagePost` receives.

### Required

| Field | Value |
|---|---|
| `post_type` | `"Account"` (literal — legacy classification field, kept as insurance; BD doesn't strictly require it but harmless to pass) |
| `data_type` | `20` (single-image classification, always for blogs) |
| `data_id` | resolved blog post-type id from runbook Step 3 |
| `post_title` | per the `Title shape` section — clickbait-flavored, anti-slop, ~70 char target |
| `post_status` | `0` (draft, default) or `1` (publish, only if user explicitly authorized) |
| `user_id` | resolved author from runbook Step 4 |

### Recommended (include when source data supports)

Universal field rules in **METHODOLOGY `Universal post fields`** (post_image, post_category, post_meta_title length, post_meta_description length). Universal tags rule in **METHODOLOGY `Tags`**. Blog-specific additions and examples below:

| Field | Blog-specific note |
|---|---|
| `post_content` | Assembled HTML body per "Content manufacture" — direct-answer opening + question H2s + answer-first paragraphs + FAQ + conclusion. Inline body images only when user explicitly requested. |
| `post_meta_title` | Type-specific example: `"Reformer Pilates vs Mat Pilates for Beginners Working Out at Home in a Small Apartment"` — audience qualifier (beginners) + use case (home workouts) + scenario (small apartment) expanded from the shorter `post_title`. |
| `post_meta_description` | Blog-specific flavor: one-sentence value proposition for the reader's decision-stage situation (e.g. "Comparing reformer and mat Pilates for beginners working out at home: calorie burn per 45-minute session, equipment cost, and which style fits a small apartment."). |

### Do NOT pass

- `post_start_date`, `post_expire_date` — events-only; blogs do not have a scheduled date semantic.
- `post_venue`, `post_location`, `lat`, `lon`, `country_sn`, `state_sn` — geo fields; blogs do not have a place anchor.
- `auto_geocode` — geo-only; not applicable to blogs.
- `revision_timestamp` — BD-managed.

---

## GHL Social Post (runbook Step 15)

After the post is confirmed live on the BD site, invoke the `ghl-social-poster` skill with:

- POST_TITLE: the value passed as `post_title`
- POST_HOOK: first 1-2 sentences of `post_content`, stripped of all HTML tags, ≤200 chars
- POST_URL: the published URL returned by `createSingleImagePost`
- IMAGE_URL: the post_image URL from createSingleImagePost
- POST_TYPE: "blog"
- POST_DATE: today's date (YYYY-MM-DD)
- POST_TAGS: 3-5 topic-relevant tags derived from the post subject
- SCHEDULE_TIME: next morning at 9:00 AM ET (America/New_York)

Only run this step if `post_status=1` (live publish). If the post was saved as a draft (`post_status=0`), skip and log:
GHL Social Post: SKIPPED — post saved as draft, not published

Log result:
GHL Social Post: [SCHEDULED for {date} at 9:00 AM ET on Facebook + Instagram] | [SKIPPED — {reason}]
