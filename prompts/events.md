# Event Publishing Routine — The Celebration Life

Connect to my Brilliant Directories site MCP and run the `bd-skill-content` skill to autonomously create event posts.

## Core parameters

- COUNT: `2` event post
- PUBLISH STATUS: `post_status=1` (live)
- AUTHOR: `user_id=5`

---

## STEP 0 — Load the Events Tracker (run FIRST, before sourcing any event)

Read the Events Tracker Google Sheet using the Google Drive MCP:
Google Drive tool: read_file_content
fileId: 1I8hDSeD-B7ad2wolPrEv6SIGYzkJKvN6xzntigb1Aos

Cache (skip header/context rows 1-3; data starts at row 4): existing Titles (col D), Slugs (col E), Event Dates (col B) + Times (col C), Venues (col J), Cities (col L), Categories (col I).

If the read fails, log "Events Tracker unavailable — proceeding with BD dedup only" and continue. The tracker is a SUPPLEMENTAL dedup signal; the BD site is the authoritative ground truth (the hard BD gate below is decisive).

## STEP 1 — Geographic targeting (weighted random)

This site's primary audience is **Celebration, FL** and surrounding communities. Apply this distribution when sourcing events:

| Weight | Geographic focus |
|---|---|
| 75% (1-15) | **Celebration, FL** — events at or within Celebration, or serving Celebration residents |
| 10% (16-17) | **Kissimmee and Davenport / Champions Gate area** |
| 15% (18-20) | **Broader Central Florida** — Orlando, Winter Garden, Clermont, Windermere, Lake Nona, Osceola County |

**Mechanism:** Generate a random integer 1-20. Map to the tier above.

**Celebration event sources (check these first when Celebration is selected):**

- `celebration.org` — official Celebration community site and events calendar
- `celebrationfl.com` — community resource site
- Celebration Community Foundation event listings
- CROA (Celebration Residential Owners Association) community calendar
- Celebration Town Center social media / Facebook events for the area
- Eventbrite filtered to ZIP code 34747 (Celebration FL)
- Facebook Events filtered to Celebration, FL
- Downtown Celebration merchants events
- Celebration Golf Club events (if open to public)
- Water Tower Place and Artisan Park community events
- Osceola County Parks and Recreation events within or near Celebration

**Celebration recurring events always eligible for a new post:**

- Celebration Farmers Market (Sundays at Lakeside Park — check for upcoming dates; each season's opening or special market days are eligible)
- Celebration Movie in the Park series
- Holiday events: Celebration of Lights, Easter in the Park, Independence Day fireworks at Celebration
- Downtown Celebration seasonal events
- Celebration golf tournaments open to public

---

## Event category taxonomy (reference — for category routing in the events content skill)

When routing events to a BD category, match to the closest category from this list. These are the site's event categories — do NOT auto-create new ones:

Anniversary Celebrations, Art Exhibitions, Author Talks, Auctions, Baby Showers, Birthday Parties, Boat Expos, Book Signings, Bridal Showers, Business Conferences, Car Shows, Career Expos, Charity Galas, Christmas Markets, College Fairs, Comedy Nights, Community Festivals, Concerts, Craft Fairs, Dance Performances, Engagement Parties, Esports Events, Easter Celebrations, Farmers Markets, Fashion Shows, Film Screenings, Fitness Classes, Food Festivals, Fun Runs, Gaming Tournaments, Halloween Parties, Hiking Events, Holiday Celebrations, Home Expos, Job Fairs, Kids Activities, Local Markets, Marathons, Museum Events, Music Festivals, Networking Events, New Years Eve Events, Outdoor Adventures, Parades, Pet Expos, Picnics, Religious Services, Retirement Parties, Reunions, School Events, Science Fairs, Seminars, Sports Games, Technology Expos, Theater Shows, Trade Fairs, Training Courses, Triathlons, Wedding Fairs, Workshops, Yoga Sessions

Per METHODOLOGY Stage 4 category routing rules: fuzzy-match the sourced event's type against this list. 70%+ confidence = use that category. Below 70% = skip the record rather than guess or auto-create.

Events do NOT rotate through categories — pick whichever category is the honest match for the event found.

---

Per METHODOLOGY `Stage 3: Source research`. Focus on events that:

- Have a verified date within the next 90 days (Date sanity gate applies strictly to events)
- Have an official source with a confirmed URL (event page, organizer site, Eventbrite, etc.)
- Match the geographic focus selected in Step 0
- Offer strong local SEO signal (real search demand for this event name or type)

Prioritize events with registration/ticketing pages for CTA button linking.

---

## STEP 1.5 — HARD DUPLICATE GATE (MANDATORY — do not skip)

Build a pool of ~5 candidate events first (verified date within 90 days, official source URL, matching the STEP 0/geographic focus). Number them 1-5. For EACH candidate, before selecting it, run BOTH checks. A flag from either = drop the candidate and advance.

**Check A — Events Tracker (from STEP 0):** duplicate if a tracked row's Title semantically matches AND Event Date is within ±24 hours AND same Venue (or same City when venue unknown). A reworded title for the same real-world instance is still a duplicate.

**Check B — Live BD (authoritative, never skip):** run on the events post type and filter to data_id=8:
1. `listSingleImagePosts property=post_title property_operator=like property_value=<first 3 distinctive words>% limit=3`
2. `listSingleImagePosts property=post_title property_operator=like property_value=<core event noun>% limit=3`
3. `listSingleImagePosts property=post_title property_operator=like property_value=%<core event noun> limit=3`
(Single-anchor wildcards only — never `%X%`.) For any title-similar hit, confirm post_start_date within ±24h AND same venue/city (getSingleImagePost if needed) → duplicate.

On a duplicate flag, drop and advance to the next candidate. Do NOT repaint with a tweaked title. If all 5 fail, generate a fresh pool of 5 distinctly different events and repeat; if that also fails, stop and report no non-duplicate events found. Take the first candidate that passes BOTH checks; log each dropped candidate and the reason.

---

## STEP 2 — Run the events content skill

Run `content-types/events.md` end-to-end with these additions:

### A. Event CTA button (registration or information link)

If a registration, ticketing, or official information URL is confirmed live (URL liveness gate passed), insert this block IMMEDIATELY AFTER the intro paragraph:

```html
<div style="text-align:center; margin: 2em 0;">
  <a href="[official event URL]" class="btn btn-secondary btn-lg vmargin" target="_blank" rel="nofollow noopener">
    [Get Tickets / Register Now / Learn More / RSVP Free] — [Event Name]
  </a>
</div>
```

Choose CTA text based on event type:
- Ticketed event → "Get Tickets"
- Free registration required → "Register Free"
- Free drop-in event → "Learn More and Plan Your Visit"
- Application or submission-based → "Submit / Apply Now"

### B. Real estate mention (natural, 1 per post)

In the body — not a button — weave in one contextually natural mention of Celebration real estate. Example placements:

- At the close of a section describing Celebration's community feel: "It's the kind of event that reminds people why so many families choose to [put down roots in Celebration](https://celebrationlivingrealestate.com/listings?boardId=32&propertyType=SFR,CND&status=active&sort=importDate&dateRange=0&cityId=3753)."
- In a closing paragraph about the neighborhood: "If events like this make you curious about living here, [current homes for sale in Celebration FL](https://celebrationlivingrealestate.com/listings?boardId=32&propertyType=SFR,CND&status=active&sort=importDate&dateRange=0&cityId=3753) are worth a browse."
- Skip the real estate mention entirely if it would feel forced or off-topic for the event type (e.g., a county fair listing where Celebration real estate has no natural connection).

### C. Business advertising mention (in closing section, prose only)

At the close of the post, weave in one sentence naturally:

> Local businesses interested in reaching Celebration residents and event-goers can [learn about advertising options here](https://www.livethecelebrationlife.com/join).

Rewrite the sentence to fit the post's tone. Keep the URL and destination unchanged.

---

## Final output

```
Event title:
Date / time:
Location:
Category:
Geographic focus: [Celebration FL / Kissimmee-Davenport / Broader Central Florida]
Internal-linking strategy used:
Published URL:
Admin edit URL:
```

---

## STEP 3 — GHL Social Post

**Run this step once per event post created in this run.** If COUNT is 2 and both posts published successfully, this step fires twice — immediately after each individual post is confirmed live, using that post's own data. Never batch both posts into one GHL call.

For each published event post, invoke the `ghl-social-poster` skill with:

- POST_TITLE: the event title for this post
- POST_HOOK: opening 1-2 sentences of the event body for this post, stripped of HTML, ≤200 chars
- IMAGE_URL: the post_image URL for this post from the Final Output block
- POST_URL: the Published URL for this post from the Final Output block
- POST_TYPE: "event"
- POST_DATE: the event date for this post (YYYY-MM-DD)
- POST_TAGS: [category, "Celebration FL", "Central Florida events"]
- SCHEDULE_TIME: per scheduling logic in ghl-social-poster SKILL.md

Append to each post's Final Output:
GHL Social Post: [SCHEDULED for {date/time} on Facebook + Instagram] | [SKIPPED — {reason}]
