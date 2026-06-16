# Blog Publishing Routine — The Celebration Life

Connect to my Brilliant Directories site MCP and run the `bd-skill-content` skill.

## Core parameters

- COUNT: `2` blog posts (use `2` only if explicitly stated in this session)
- PUBLISH STATUS: `post_status=1` (live)
- AUTHOR: `user_id=5`

---

## STEP 0 — Load the Blog Tracker (run FIRST, before any topic brainstorm)

Read the Blog Tracker Google Sheet using the Google Drive MCP:

```
Google Drive tool: read_file_content
fileId: 1sVn6SeSbjX0wrbFiHbwwwuB4RKAnb6xEvnaOufBOr9k
```

From the loaded sheet, extract and cache three datasets:

1. **Category history** — the Category column values for all posts, sorted newest-first. You need the last 6.
2. **Existing post titles** — all values in the Title column.
3. **Existing slugs** — all values in the Slug column.

If the Google Drive read fails, log "Tracker unavailable — proceeding without sheet data" and continue.

**IMPORTANT:** The tracker sheet is a SUPPLEMENTAL dedup signal only. The BD site itself is the authoritative ground truth. A topic that exists on the BD site must never be republished regardless of whether it appears in the tracker sheet. The BD dedup queries in STEP 3 are the hard gate — not the sheet.

---

## STEP 1 — Geographic targeting (weighted random)

This site's primary audience is **Celebration, FL** and the surrounding Central Florida communities it serves. Apply this distribution:

| Weight | Geographic focus |
|---|---|
| 75% (1-15) | **Celebration, FL** — town center, CROA/HOA, schools, neighborhoods, dining, real estate, community news, events, lifestyle |
| 10% (16-17) | **Kissimmee and Davenport / Champions Gate area** |
| 15% (18-20) | **Broader Central Florida** — Orlando, Winter Garden, Clermont, Windermere, Lake Nona, St. Cloud, Osceola County |

**Mechanism:** Internally generate a random integer 1-20. Map to the tier above. The selected geographic tier constrains the topic brainstorm in Step 3.

**Celebration content anchors** (use as brainstorm seed topics when Celebration is selected):

- Celebration CROA / HOA: fees, what it covers, rules, community governance, amenities
- Celebration schools: Celebration K-8, Celebration High School, private school options, school ratings
- Celebration neighborhoods deep-dives: Spring Valley, North Village, West Village, South Village, Artisan Park, Aquila Reserve, Mirasol
- Celebration town center: dining, shopping, events, walkability
- **Celebration Farmers Market at Lakeside Park** (every Sunday — has NOT been covered yet; excellent target)
- Seasonal Celebration events: Winter Wonderland, July 4th, spring festivals, Halloween, holiday lights
- Celebration real estate: market trends, buyer guide, HOA-included amenities, what makes Celebration different
- Celebration lifestyle: bike paths, lake activities, proximity to Disney and major highways
- Moving to Celebration guide (schools + HOA + commute + dining all-in-one)
- Celebration vs nearby communities (Lake Nona, Windermere, Dr. Phillips) — buyer comparison piece
- Ovation Orlando's impact on Celebration (development already covered — need a NEW angle)
- Celebration community organizations: Foundation for Celebration, HOA committees, volunteer opportunities

**DO NOT re-cover topics already in the tracker OR already on the BD site.**

---

## STEP 2 — Category rotation (round-robin across 27 categories)

The site has 27 blog categories. Rotate through them in this fixed sequence — every post advances to the next unused position. The cycle completes after 27 posts, then restarts from position 1.

```
Position  Category
1         Things to Do
2         Best Of
3         Real Estate
4         Neighborhood News
5         Family & Kids
6         Food & Dining
7         Entertainment & Nightlife
8         Outdoors & Recreation
9         Job & Career Advice
10        Health & Wellness
11        Education & Schools
12        Local History
13        Arts & Culture
14        Home & Garden
15        Volunteer & Charity
16        Shopping & Retail
17        New Business Openings
18        Lifestyle
19        Pet-Friendly Spots
20        Sports & Fitness
21        Local Interviews
22        Travel & Staycations
23        Seasonal Guides
24        Giveaways & Promotions
25        Local Events
26        Community Spotlights
27        Business Features
```

**Special geographic note for position 18 (Lifestyle):** When this category comes up in rotation, override Step 1's random selector and treat the geographic focus as broader Central Florida. Do NOT apply Celebration-first weighting to this category.

**Rotation rule:**

1. From the Blog Tracker sheet (Step 0), extract the Category column for ALL posts, sorted newest-first.
2. Map each published category to its position number in the 27-item list above.
3. Find the lowest-numbered position in the 27-item list that has NOT yet been published.
4. That is the category for this post.
5. If all 27 positions have been published at least once, restart: use the category whose most recent post is OLDEST.

**Mapping published posts to positions (as of tracker data loaded in Step 0):**
- "Things to Do" = position 1 (Post #6 — Winter Garden Farmers Market)
- "Real Estate" = position 3 (Post #3 — Celebration HOA)
- "Family & Kids" = position 5 (Post #1 — Winter Garden Family Spots)
- "Food & Dining" = position 6 (Post #4 — Celebration Restaurants)
- "Community Spotlights" = position 26 (Post #5 — Ovation Orlando)
- "Local Events" = position 25 (Post #2 — Windermere Events)

Always re-derive from the live tracker data in Step 0.

---

## STEP 3 — Topic brainstorm, BD dedup, and selection

### 3A — Brainstorm 5 candidates

Build a pool of 5 topic candidates that match the geographic focus (Step 1) and category (Step 2) and target real local search intent.

### 3B — Hard BD dedup gate (MANDATORY — do not skip)

For EACH of the 5 candidates, before selecting it, run ALL of the following BD search queries:

1. Search BD posts for the **exact topic phrase** (e.g. "coffee shops in Celebration FL")
2. Search BD posts for **the core subject noun** (e.g. "coffee shops")
3. Search BD posts for the **primary location keyword** (e.g. "Celebration coffee")
4. Search BD posts for **2-3 synonym or close-variant phrasings** (e.g. "cafes in Celebration," "where to get coffee in Celebration")

A topic FAILS dedup if ANY of these queries returns a published post covering substantially the same ground — even if the title is worded differently. "The Best Coffee Shops in Celebration FL" and "Where to Get Coffee in Celebration" are the same topic. Both would fail.

**Do not trust the tracker sheet as proof a topic is safe.** The sheet may be out of date. The BD site is the ground truth.

### 3C — Selection

Take the first candidate that passes ALL four BD dedup queries. If all 5 fail, generate 5 new candidates and repeat. Log each dedup failure and why.

---

## STEP 4 — Run the blog content skill

Run `content-types/blog.md` end-to-end with additions from `shared/CELEBRATION-GEO.md` and `shared/SEO-AEO-SCHEMA.md` applied.

### Required additions for every blog post on this site:

#### A. Key Takeaways box (required)

Insert immediately AFTER the opening paragraph (before the first H2):

```html
<div style="background-color:#f5f4f0; border-left:4px solid #b0a898; padding:16px 20px; margin:1.5em 0; border-radius:3px;">
  <p><strong>Key Takeaways</strong></p>
  <ul>
    <li>[Takeaway 1 — specific, useful, 10-20 words]</li>
    <li>[Takeaway 2 — specific, useful, 10-20 words]</li>
    <li>[Takeaway 3 — specific, useful, 10-20 words]</li>
    <li>[Optional Takeaway 4 if the post warrants it]</li>
  </ul>
</div>
```

Each takeaway must be a standalone factual claim a reader could cite — not a teaser or headline restatement.

#### B. Mid-article real estate soft CTA (required — 1 per post)

Place ONCE in the body at the 40-60% point (between H2 sections, never mid-paragraph). Match the CTA type to the post's topic:

| Post signal | CTA text and URL |
|---|---|
| Buyer focus / moving / relocating / neighborhood guide | "Ready to search homes for sale in Celebration FL? [Browse the latest listings here.](https://celebrationlivingrealestate.com/listings?boardId=32&propertyType=SFR,CND&status=active&sort=importDate&dateRange=0&cityId=3753)" |
| Seller / homeowner / renovation / market conditions | "Curious what your Celebration home is worth right now? [Get your free instant home valuation.](https://celebrationlivingrealestate.com/home-valuation)" |
| Market data / pricing trends / investment | "[Check out the latest Celebration FL real estate market report](https://celebrationlivingrealestate.com/market-report) for current pricing and inventory data." |
| Deals / price drops mentioned | "If you're watching the market, [these price-reduced Celebration homes](https://celebrationlivingrealestate.com/listings?boardId=32&bedrooms=0&bathCount=0&propertyType=SFR,CND&status=active&sort=importDate&dateRange=0&cityId=3753&misc6Yn=true) are worth a look." |
| New development / new construction | "Interested in brand-new construction in Celebration? [See new build listings here.](https://celebrationlivingrealestate.com/listings?boardId=32&propertyType=SFR,CND&status=active&sort=importDate&dateRange=0&cityId=3753&newConstructionYn=true)" |
| Lifestyle / dining / events / things-to-do / family (default) | "Thinking about living here? [Search homes for sale in Celebration FL](https://celebrationlivingrealestate.com/listings?boardId=32&propertyType=SFR,CND&status=active&sort=importDate&dateRange=0&cityId=3753) and see what's available." |
| Luxury lifestyle / high-end amenities | "For those eyeing Celebration's finest homes, [luxury listings $1M+ are here.](https://celebrationlivingrealestate.com/listings?boardId=32&minListPrice=1000000&propertyType=SFR,CND&status=active&sort=importDate&dateRange=0&cityId=3753)" |

Format as a full-width Bootstrap button block between two H2 sections:

```html
<div style="text-align:center; margin: 2em 0;">
  <a href="[URL]" class="btn btn-primary btn-lg vmargin" target="_blank" rel="noopener">[CTA Text]</a>
</div>
```

#### C. Business advertising CTA (required — in conclusion section)

Weave naturally into the conclusion paragraph as prose — not a button. Keep the link to `/join` unchanged. Do not use "vibrant," "bustling," "amazing," "local gem," or any ANTI-SLOP banned vocabulary.

#### D. Newsletter CTA (required — final element of post)

```html
<div style="text-align:center; margin: 2em 0;">
  <p><strong>[Rewrite this line to match the post topic — see rules below]</strong><br>
  [One supporting sentence specific to what was just covered — not generic]</p>
  <a href="https://www.livethecelebrationlife.com/newsletter" class="btn btn-success btn-lg vmargin" target="_blank" rel="noopener">
    Subscribe to The Celebration Life Newsletter
  </a>
</div>
```

**Rules:**
- Always the last element of the post body. Nothing follows it inside the post content.
- Rewrite the bold line and supporting sentence to match the post's topic specifically.
- Button text is fixed: "Subscribe to The Celebration Life Newsletter"
- URL is fixed: `https://www.livethecelebrationlife.com/newsletter`
- No banned vocabulary.

#### E. Entity mention limits — addresses and numbers

- **Addresses:** State the full address ONCE at first introduction. All later references use a natural short form ("the shop," "the spot on Market Street"). Never restate an address in a later section, FAQ, or conclusion.
- **Phone numbers:** Omit entirely for dining, retail, coffee, and lifestyle posts. Include once only for medical, legal, or service businesses.
- **Hours:** State ONCE per business. May restate in the FAQ only if the FAQ question is genuinely about hours.
- **Prices:** State a specific figure ONCE. Later references use relative language.
- **Distances and drive times:** State ONCE total.

For multi-business roundups (coffee shops, restaurants, parks): consolidate addresses, hours, and price ranges into ONE summary table near the end of the post — do not repeat this data in each H2 section.

#### F. SEO/AEO/GEO content structure

Apply all rules from `shared/SEO-AEO-SCHEMA.md`:
- Definition blocks where applicable
- FAQ block (3-5 Q&A pairs, city name in at least 2 questions)
- Tables for comparison or data-heavy content
- Answer-first paragraph structure throughout
- Entity specificity throughout

---

## Final output summary

```
Title: [post title]
Category: [category] (position N of 27)
Geographic focus: [area]
Target query: [main search phrase]
Published URL: [full URL]
Admin edit: [BD admin URL]
Next category due: [name] (position N of 27)
```

---

## STEP 5 — GHL Social Post

**Run this step once per post created in this run.** If COUNT is 2 and both posts published successfully, this step fires twice — immediately after each individual post is confirmed live, using that post's own data. Never batch both posts into one GHL call.

For each published post, invoke the `ghl-social-poster` skill with:

- POST_TITLE: the value passed as `post_title` for this post
- POST_HOOK: first 1-2 sentences of `post_content` for this post, stripped of all HTML tags, ≤200 chars
- POST_URL: the Published URL for this post from the Final output summary
- IMAGE_URL: the post_image URL returned by createSingleImagePost for this post
- POST_TYPE: "blog"
- POST_DATE: today's date (YYYY-MM-DD)
- POST_TAGS: 3-5 topic-relevant tags derived from this post's subject
- SCHEDULE_TIME: next morning at 9:00 AM ET (America/New_York)

Only run this step if `post_status=1` (live publish). If a post was saved as a draft, skip that post's GHL call and log:
GHL Social Post: SKIPPED — post saved as draft, not published

Append to each post's Final output summary:
GHL Social Post: [SCHEDULED for {date} at 9:00 AM ET on Facebook + Instagram] | [SKIPPED — {reason}]
