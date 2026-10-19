# Feature PRD — NextBite: Personal Dish Recommendations

**Ticket:** TBD
**Related:** [Search PRD (parked)](NextBite-Search-PRD.md) · [MOM 19 Oct](MOM-2026-10-19-NextBite-Brainstorming.md)
**Status:** Draft v2: updated after the 19 Oct brainstorming (readiness 60/100, see section 13)
**Suggested first slice:** S1 — hackathon demo: weekly dish picks as swipe cards, a saved dishes page, and a feedback loop that visibly improves the next picks

> Written from a senior PM / staff engineer view. Every statement is labeled as one of:
> **[Fact]** (from qlub data or the existing product), **[Team]** (said in the 19 Oct meeting, not yet verified with data), **[Assumption]** (a default we ship unless corrected), or **[Rec]** (a recommendation).
> Missing business rules are listed as open questions. They are not invented here.
> **Search is out of scope for this PRD.** It has its own PRD and is a Day 2 stretch only.

---

## 1. Summary

**Feature:** NextBite gives every qlub diner **5–10 personal dish picks** when they open the app, with no typing and no questions first. The diner swipes right to **save** a dish or left to **skip** it. Every swipe, tap, save and order teaches NextBite what they like, so next week's picks are better.

**One-line pitch:** *Open qlub, see this week's dishes picked for you, swipe, go.*

**Problem:**
- **[Fact]** Diners only use qlub at the table: they scan, pay and leave. In the UAE, 34k payments led to only **4k app discovery views**. That leaves about 30k diners a month that qlub cannot reach after they pay.
- **[Team]** Everyone discovers restaurants on **Google Maps** (search, sort by distance or rating) or **Instagram** (save to a collection). Both are slow, and nobody uses qlub to discover.
- **[Team]** Qlub's in-restaurant (QSR) recommendations already exist. Diners add recommended items to their cart a lot, but the effect on sales looks small. Two reasons:
  1. They only work *inside* a restaurant.
  2. Their inputs are weak, because there is no real customer profile.
- **[Team]** People are lazy with typing and with AI prompts. A chat or search box alone won't build a habit. Instagram builds the habit with no chat, purely from how people interact with content.

**Our edge over Google [Team]:**
- **Accurate menu prices:** Google only has photos of menus.
- **Promotions:** qlub+ discounts, bank offers, seasonal dishes.
- **Real orders:** what qlub users actually order every week.
- **Per-dish reviews:** qlub knows exactly which dish was eaten.

**Users:**
- **Logged-in diners and qlub+ subscribers** (primary, best accuracy). **[Team]** About 60k subscribers and more than 100k logged-in customers.
- **Anonymous diners** (secondary). They have a user ID, but it changes per browser, so they get a rougher, cluster-based version and are invited to log in.
- **Restaurants** (downstream): their dishes reach diners who are likely to love them.
- **Marketing / Growth** (internal): taste clusters make campaigns more precise.

**Proposed outcome:**
- A diner opens NextBite and sees a stack of dish cards: photo, dish, price, restaurant, distance, and any promotion.
- Each card has a short personal reason, e.g. *"You've had the truffle pasta 3 times. This risotto is in the same flavour family and 20% cheaper."*
- Swipe right → the dish goes to **Saved dishes**. Swipe left → skipped, and similar dishes show up less.
- **"Not what you're looking for?"** opens a quick, tap-only guided flow and creates a fresh set of picks.
- When the diner later orders a saved dish through qlub, it's marked **tried** automatically, and they can rate it.

---

## 2. Goals

| ID | Goal |
|----|------|
| G1 | A diner sees 5–10 personal dish picks on opening NextBite, with no input needed. |
| G2 | The diner can swipe right (save) or left (skip) on each pick; every swipe improves future picks. |
| G3 | The diner has a **Saved dishes** list, and dishes are marked as tried automatically when ordered via qlub. |
| G4 | New and anonymous diners get good picks from day one, based on what similar qlub users order each week (cluster top picks). |
| G5 | A new diner can be profiled in under 30 seconds through a fun, picture-based swipe onboarding (~10 dishes), not a form. |
| G6 | **"Not what you're looking for?"** lets the diner steer picks with taps (e.g. pizza → which kind) and get a new set in real time. |
| G7 | Each pick shows one short, personal, **factually correct** reason (AI-written, fact-checked). |
| G8 | Dish quality is judged by the **reorder score** (how many people come back for it), then by per-dish ratings. |
| G9 | Picks fit the diner's context: price range, distance, weekday vs. weekend, and promotions they care about. |
| G10 | Raise UAE monthly discovery views from 4k to at least **15k**, driven by a weekly "open and swipe" habit. |

## 3. Non-Goals

| ID | Excluded |
|----|----------|
| NG1 | **Search** (keyword or plain language). See the [Search PRD](NextBite-Search-PRD.md). Day 2 stretch only. |
| NG2 | A chat assistant as the main way to get picks. The guided flow is tap-based. |
| NG3 | Asking the diner to choose one favourite cuisine. Preferences are learned, not asked (MOM D8). |
| NG4 | Recommending whole restaurants without a dish. We always recommend a dish. |
| NG5 | Ordering, booking or paying from NextBite. |
| NG6 | Showing AI-generated dish photos that the restaurant hasn't approved. |
| NG7 | Push, SMS or email campaigns. This PRD produces picks and clusters; sending is separate. |
| NG8 | Trained ML ranking models in S1–S2. Rules + clusters + an LLM for reasons first. |
| NG9 | Restaurants that aren't on qlub, or that have no digital menu. |
| NG10 | Paid placement (qlub+ restaurant boosts) until the rules are agreed (Q8). |

---

## 4. User Journey

### J1: First open: quick swipe onboarding (new diner)

**Actor:** A diner opening NextBite for the first time (logged in or anonymous)

**Flow:**
1. Screen: "Swipe 10 dishes and we'll find your next bite." (skippable)
2. The system shows 10 dish photos chosen to cover different cuisines, price levels and styles (BR6).
3. The diner swipes right (looks good) or left (not for me) on each.
4. Optional fun budget question: a slider, "Your usual dinner per person?", with food icons (AED 30 → AED 300).
5. The system builds a starter profile and shows the first picks (J2).

**Failure cases:** The diner skips onboarding → show cluster top picks (BR5) and learn from swipes on the main stack. No photos for a dish → don't use that dish in onboarding.

### J2: Weekly picks: the swipe stack (main screen)

**Actor:** Any diner

**Flow:**
1. The diner opens NextBite. A stack of 5–10 dish cards is shown ("Your picks this week").
2. Each card shows (BR10):
   - dish photo, dish name, price
   - restaurant name, area and distance
   - promotion badge, if any (e.g. "15% off with qlub+")
   - one personal reason
3. Swipe right → the dish is saved (J4). Swipe left → skipped (BR8). Tap → the dish detail and restaurant menu open (counts as interest, BR8).
4. At the end of the stack: "That's this week's picks. Want more?" → "Not what you're looking for?" (J3), or come back next week.
5. Picks refresh weekly, and also right after enough new feedback (BR4).

**Failure cases:**
- Not enough candidates near the diner → widen the distance once, then show cluster top picks.
- The reason AI is slow or down → template reason (BR13).
- The diner swipes left on everything → open J3 automatically: "Let's find what you're craving."

### J3: "Not what you're looking for?": guided, tap-only curation

**Flow:**
1. The diner taps **"Not what you're looking for?"** (under the stack, or automatically after many left swipes).
2. Step 1, mood: big picture tiles generated from real menu data, such as Pizza · Burgers · Asian · Healthy · Arabic · Dessert · "Surprise me".
3. Step 2, narrow it down: e.g. Pizza → Margherita · Pepperoni · Truffle · Spicy · Vegetarian (only options that exist nearby).
4. Step 3, optional quick chips: "Under AED 60", "Near me (≤ 15 min)", "Has a deal".
5. The system creates a new stack in real time (J2 rules apply), and the choices are stored as feedback.

**Failure cases:** A choice has no dishes nearby → that tile is not shown. The diner leaves mid-flow → keep the current stack.

### J4: Saved dishes and "tried" progress

**Flow:**
1. Swiped-right dishes appear in **Saved dishes**, grouped by restaurant, with price and distance.
2. The diner can remove a dish, or open the restaurant.
3. When the diner orders a saved dish through qlub (logged in), it's marked **Tried ✓** automatically (BR11).
4. Progress is shown: "3 of 10 saved dishes tried".
5. After a dish is tried → "Did you like it?" (J6).

**Failure cases:** An anonymous diner switches browser → their saved list is lost. Show a prompt: "Log in to keep your saved dishes".

### J5: Log in for better picks (anonymous diner)

**Flow:** After a few swipes or saves, show a soft prompt: "Log in so we remember your taste and your saved dishes." After login, the anonymous profile and saves merge into the account (BR3).

### J6: After the visit: "Did you like it?"

**Flow:**
1. After a qlub payment that includes a recommended or saved dish, the diner is asked about that dish: 👍 / 👎 (optionally stars).
2. 👍 → similar dishes are boosted, and the dish can appear in reasons ("You loved the karahi…").
3. 👎 → that dish and very similar dishes are removed for this diner.
4. The ratings also feed per-dish quality for everyone (BR9).

### J7: Weekend comeback (re-engagement)

**Flow:** On the next weekend, NextBite shows a "Again this weekend?" card for a dish the diner liked last time, alongside new picks (BR7).

---

## 5. Requirements

| ID | Requirement | Priority | Acceptance criteria |
|----|-------------|----------|---------------------|
| R1 | Show a stack of 5–10 dish picks on open, with no input needed. | Must | Given any diner, when NextBite opens, then at least 5 picks are shown, never an empty screen. |
| R2 | Dish card content (BR10). | Must | Each card shows photo, dish, price, restaurant, area/distance and one reason; a promotion badge when one applies. |
| R3 | Swipe right = save, swipe left = skip, tap = interest (BR8). | Must | Given a right swipe, then the dish is in Saved dishes. Given a left swipe, then it doesn't appear again for 30 days. |
| R4 | Feedback changes the next picks (BR8). | Must | Given 5 left swipes on dishes above AED 150, then the next stack has at most 1 dish above AED 150. |
| R5 | Cluster top picks for new and anonymous diners (BR5). | Must | Given a diner with no history, then the stack is filled from their cluster's top dishes this week. |
| R6 | Swipe onboarding with ~10 dishes and an optional budget slider (J1, BR6). | Should (S1 demo) | Given 10 swipes, then a starter profile exists, and the first stack reflects it (e.g. 7 right swipes on Asian → at least 3 Asian picks). |
| R7 | Saved dishes page with remove and open restaurant (J4). | Must | Given 3 saved dishes, then all 3 are listed with price and distance. |
| R8 | Auto-mark saved dishes as tried when ordered via qlub, with progress (BR11). | Should | Given a saved dish ordered via qlub, then it shows Tried ✓, and the progress updates. |
| R9 | "Not what you're looking for?" guided flow with tap-only steps from real menu data (J3). | Should | Given Pizza → Truffle, then the new stack has only truffle pizzas available nearby; tiles with 0 dishes are hidden. |
| R10 | Personal AI-written reason per pick, fact-checked, with template fallback (BR12–BR13). | Should | Given the truffle pasta persona, then the risotto card says "3 times" and "20% cheaper", and every fact matches the data. |
| R11 | Quality = reorder score, then per-dish ratings (BR9). | Must | Given two dishes with the same match, then the one more people reorder ranks first. |
| R12 | Context: weekday vs. weekend, distance, price band, promotions (BR7). | Should | Given a Saturday evening, then at least 2 picks are "treat" dishes (higher price or new cuisine) if the diner's profile allows. |
| R13 | Exploration: about 20% of picks are outside the diner's usual cuisines (BR4). | Should | Given a diner whose history is 90% Indian, then about 1–2 of 10 picks are other cuisines. |
| R14 | "Did you like it?" after ordering a recommended or saved dish (J6). | Should (S2) | Given 👎, then that dish and very similar dishes stop appearing for this diner. |
| R15 | Login prompt and anonymous-to-account merge (J5, BR3). | Should (S2) | Given an anonymous diner with 3 saves who logs in, then the 3 saves are in their account. |
| R16 | Only vendor-approved photos are shown (BR10). | Must | A dish with only an unapproved AI photo is shown with a neutral placeholder, or not picked in onboarding. |
| R17 | Log all feedback events (BR14). | Must | Swipes, taps, saves, removals, guided-flow choices, tried and ratings are stored with the diner ID. |
| R18 | Weekend comeback card (J7). | Could | Given a 👍 dish last weekend, then this weekend's stack includes it as "Again this weekend?". |

---

## 6. Business Rules

| ID | Rule | Example |
|----|------|---------|
| BR1 | **We recommend dishes, not restaurants** (MOM D2). Only dishes from active qlub restaurants with a live digital menu and an available item. | A churned restaurant's dishes disappear at the next refresh. |
| BR2 | **Profile = 3 inputs**, combined: (1) purchase history (cuisines, dishes, price per person, day and time), (2) onboarding swipes, (3) ongoing feedback (BR8). Recent signals weigh more. Purchase history is a weaker signal than a right swipe, because a diner may order something only because it was easy or discounted (MOM). | 3 orders of Margherita + 4 right swipes on Asian dishes → profile likes pizza and Asian. |
| BR3 | **Identity:** logged-in diners have one stable profile. Anonymous diners get a profile per user ID (it may reset when the browser changes). When an anonymous diner logs in, their swipes and saves merge into the account. | — |
| BR4 | **Pick mix per stack (10 picks)** **[Assumption A1]**: ~6 strong matches, ~2 cluster top picks, ~2 exploration (outside usual cuisines, still within the price band). Refresh weekly, and after every 10 new feedback events. | Diner who eats 80% Indian still sees 2 non-Indian dishes. |
| BR5 | **Clusters:** logged-in diners are grouped into ~6 taste clusters **[Assumption A2]** from their orders (cuisine mix, price per person, area, weekday/weekend). Each week, the top dishes per cluster = most ordered dishes with a good reorder score. New and anonymous diners are matched to the closest cluster (from onboarding swipes, area and first order). | Cluster "Mid-budget Asian lovers, Marina" → this week's top 10 dishes in that cluster. |
| BR6 | **Onboarding dishes** cover at least 5 cuisines and 3 price levels, have approved photos, and are popular (so the first impression is good). | — |
| BR7 | **Context:** weekdays favour quick, close, usual-budget dishes; weekends allow "treat" dishes (higher price, new cuisine, farther away) **[Assumption A3]**. Distance tolerance is learned per diner from where they actually go, not asked. Promotions (qlub+ discount, bank offers) are shown as a badge and give a small ranking boost to diners who used promotions before. | Weekday lunch → nearby AED 40 bowl. Saturday → AED 120 tasting dish 20 min away. |
| BR8 | **Feedback signals and weights** **[Assumption A4]**: right swipe / save = strong like; tap to open = weak like; left swipe = dislike (that dish hidden for 30 days, similar dishes lowered); ordered via qlub = strong like; 👍 = strongest like; 👎 = hide the dish and very similar dishes. Patterns are learned from repeated signals (e.g. many left swipes above a price → price cap; all Italian skipped → Italian lowered). Never from a single swipe. | 5 skips on dishes > AED 150 → price cap AED 150. |
| BR9 | **Quality = reorder score first, ratings second.** Reorder score = of the diners who ordered the dish in the last 180 days, the share who ordered it again on a different day. It's used with ≥ 20 distinct diners and shown in a reason with ≥ 50; below that, it's pulled toward the area average. Per-dish 👍/👎 ratings break ties and catch quality drops. Tourist areas and fine dining are compared with similar places only. | 62% of 140 diners reorder the karahi → high quality. |
| BR10 | **Card content:** approved photo (or a neutral placeholder; unapproved AI photos are never shown), dish name, menu price in AED, restaurant name, area and distance (or travel time), promotion badge if active, one reason. | — |
| BR11 | **Tried:** a saved dish is marked tried when the same diner (logged in) pays for that item via qlub. It then leaves the "to try" count, and J6 asks for feedback. | 10 saved, 3 ordered → "3 of 10 tried". |
| BR12 | **AI personal reason:** the system computes a fact sheet (anchor dish and how often it was ordered, shared flavour attributes, price difference rounded to 5%, distance, reorder score, promotion). The LLM writes ≤ 140 characters using only those facts. It never mentions spending totals, other people, diet or health guesses. Every number and name is fact-checked; any mismatch → template reason. | ✓ "This risotto is 20% cheaper." ✗ "You spent AED 900 last month." |
| BR13 | **Template reasons** (fallback): "Because you liked {dish}", "Clubbers in your area love this", "In your usual price range", "People keep coming back for this", "{discount} off with qlub+", "New for you: try {cuisine}". | — |
| BR14 | **Data use:** only consented diners are personalized (Q6). Only the anonymous fact sheet is sent to the LLM (no name, phone, card or payment IDs). | — |
| BR15 | **Guided flow options** (J3) are built only from dishes that exist and are available near the diner. No option leads to an empty stack. | "Truffle pizza" is shown only if at least 3 truffle pizzas are nearby. |

---

## 7. UX States

**Swipe stack:**
- **onboarding** (first open)
- **loading** (card skeleton)
- **default** (cards)
- **end of stack** ("That's this week's picks")
- **all skipped** (opens the guided flow)
- **no nearby dishes** (widen the distance → cluster picks)
- **error** (retry; never a blank screen)

**Card:**
- AI reason, or template reason
- with or without a promotion badge
- with an approved photo, or a placeholder

**Saved dishes:**
- empty ("Swipe right on dishes you want to try")
- list
- tried ✓
- progress
- lost after a browser change (anonymous; show a login prompt)

**Guided flow:** steps 1–3, plus "nothing nearby" (that tile is hidden).

### Design

**Canvas:** [qlub+ Home — Dish Recommendations](https://claude.ai/artifact/5vimkh14rSS4VYK1U3qkAN) (interactive prototype, 390 px phone frames). Source files: [`design/nextbite-ux/`](design/nextbite-ux/).

| Screen | What it shows | PRD link |
|--------|---------------|----------|
| qlub+ Discover, with Dishes You'll Love | The swipe stack inside the qlub+ Discover home, below "Visit again" and above "Best offers near you" | J2, R1–R2 |
| Swipe — live prototype | Full-screen swipe stack: drag to save or skip, tap a card for "Why this pick", Save / Skip buttons | J2, R3 |
| 1 · First-time coach | "Swipe to teach your taste" overlay on first open | J1 |
| 2 · Dragging right (Save) | Save stamp while dragging right | R3 |
| 3 · Dragging left (Skip) | Skip stamp while dragging left | R3 |
| 4 · Tap — Why this pick | Card flips to the match % and 3 personal signals, with View restaurant / Save dish | R10, BR12 |
| 5 · After save (Undo) | "Saved to Your Cravings" toast with Undo | R3 |
| 6 · After skip (Why skip?) | Skip toast with quick reasons ("Why skip?"), then "We'll show fewer dishes like this" | R4, BR8 |
| 7 · All caught up | End of stack: saved count, Start over, Plan a visit | J2 (end of stack) |
| Saved — Dishes & Restaurants | Dishes / Restaurants tabs, "2 of 4 tried" tasting list, All / To try / Tried filters, auto-tick on qlub payment | J4, R7–R8, BR11 |
| Foodie Score | Points, 4 levels, cuisine passport, areas explored, next challenges, shareable card | Not in this PRD yet (see below) |

**Where the design and the PRD differ (to settle):**
- **Refresh:** the design says picks are "Refreshed daily"; the PRD says weekly plus after every 10 feedback events (BR4).
- **"Why skip?" reasons:** the design asks the diner for a skip reason; the PRD learns from patterns only (BR8). If kept, skip reasons should be added as a feedback signal.
- **Foodie Score** (points for tried dishes, new cuisines and new areas) is in the design but not in the requirements. It fits S3 habit features; it needs its own rules before it's built.
- **"Not what you're looking for?"** (J3) has no screen yet.

---

## 8. Technical Impact

> All technical choices below are **[Rec]**. For the hackathon this is a **standalone app in a team GitHub repository outside qlub's organisation**, running on a data extract or demo data. Qlub source code is not used.

- **UI (hackathon):**
  1. Swipe onboarding
  2. Picks swipe stack
  3. Saved dishes page
  4. "Not what you're looking for?" guided flow (if time allows)
  5. Mock "Did you like it?" after payment (optional)
- **API:**
  - `GET /v1/nextbite/picks?lat=&lng=` → picks with fact sheets and reasons
  - `POST /v1/nextbite/feedback` → swipe, tap, save, remove, guided choice, rating
  - `GET /v1/nextbite/saved`
  - `POST /v1/nextbite/onboarding`
- **Recommendation engine:**
  1. **Dish catalogue:** menu items with cuisine, dish type, flavour attributes, price and photo status. AI can fill in missing ingredients and attributes from the dish name and description (Menu Brain enrichment). The full tag list is in [data/dish-tags.md](data/dish-tags.md).
  2. **Clusters:** a nightly job groups logged-in diners into ~6 taste clusters and computes the top dishes per cluster per week.
  3. **Diner profile:** purchase history + onboarding + feedback, with recent signals weighed more (BR2, BR8).
  4. **Scoring:** taste match + price fit + quality (reorder score) + distance + context + promotion boost, then the pick mix (BR4).
  5. **Reasons:** a fact-sheet builder, one batched LLM call (e.g. Claude Haiku) for the top 10, a fact checker, and template fallback.
- **Data sources (hackathon, MOM A2):**
  - about 100 restaurants with menus, prices and photo status
  - sample logged-in diners with orders (or realistic generated data with the same shape)
  - promotions (qlub+ discounts)
  - existing product-click events, if available
- **Database:** `dish` (+ attributes, embedding, photo_status), `diner_profile`, `cluster`, `cluster_top_dishes_weekly`, `dish_reorder_score`, `feedback_event`, `saved_dish` (diner_id, item_id, saved_at, tried_at).
- **Reuse in the real product (later):** the existing saved-restaurants feature (extend it to dishes), the existing product-click JS events, and the vendor-approved AI photo pipeline (expected ~1 month, MOM).
- **Privacy:** consent flag, opt-out, and only the anonymous fact sheet goes to the LLM.

---

## 9. Edge Cases

- **Diner orders for convenience, not taste** (always the same dish because it's quick or discounted) → purchase history weighs less than swipes and ratings (BR2).
- **Diner mixes cuisines** (no single favourite) → the profile keeps a cuisine *mix*; exploration picks stay on (BR4).
- **Group bills** → only the diner's own split items count. If the bill isn't split, it counts at low weight and is never used as the anchor in a reason.
- **Anonymous diner changes browser** → the profile resets; show cluster picks and a login prompt.
- **Dish with no approved photo** → placeholder, lower rank, never in onboarding.
- **Restaurant with no qlub digital menu** → its dishes can't be recommended (BR1).
- **Few dishes near the diner** → widen the distance once, then show cluster picks; never an empty stack.
- **Price changes** after a dish is saved → show the new price in Saved; recompute reasons.
- **Saved dish removed from the menu** → show "No longer available", with similar dishes from the same restaurant.
- **The diner swipes left on everything** → open the guided flow; don't infer a strong dislike from one session alone.
- **Promotion expired** → remove the badge and the boost immediately.
- **Late-night or alcohol items** → excluded from reasons and from onboarding.

---

## 10. Implementation Slices

| Order | Slice | User outcome | Requirements | Dependencies |
|-------|-------|--------------|--------------|--------------|
| 1 | **S1 — Hackathon demo** | Onboarding swipes → personal stack with reasons → swipe right to save → Saved page → the next stack visibly changes after feedback | R1–R7, R10, R11, R16, R17 | Data extract (~100 restaurants, sample diners) |
| 2 | S1+ — Guided flow (Day 2 if time) | "Not what you're looking for?" taps create a new stack | R9 | S1 |
| 3 | S2 — Product pilot | Real clusters, tried auto-mark, "Did you like it?", login merge, weekday/weekend context, exploration | R8, R12–R15 | S1, Q1–Q3 answered |
| 4 | S3 — Habit features | Weekend comeback card, progress challenges, re-engagement | R18 | S2 |

### S1 acceptance criteria (the demo)
- A new diner swipes 10 onboarding dishes, and the first stack reflects the swipes (e.g. more Asian after Asian right swipes).
- Each card shows photo (or placeholder), dish, price, restaurant, distance and one reason.
- The truffle pasta persona sees "You've had the truffle pasta 3 times. This risotto is in the same flavour family and 20% cheaper.", and every fact is correct.
- 5 left swipes on expensive dishes → the next stack has at most 1 expensive dish.
- Right-swiped dishes appear on the Saved page.
- A dish ordered by more returning diners ranks above a similar dish ordered once by many.
- Turning off the LLM → template reasons; the stack still loads.

### S2 acceptance criteria
- An anonymous diner who logs in keeps their saves.
- A saved dish ordered via qlub shows Tried ✓, and the progress updates.
- 👎 on a dish removes it and very similar dishes for that diner.
- About 2 of 10 picks are exploration picks.

---

## 11. Rollout

- **Hackathon:** standalone demo app on a data extract or demo data, deployed from the team repository.
- **Pilot (after the hackathon):**
  1. Feature flag in the qlub app, UAE only.
  2. Start with logged-in diners and qlub+ subscribers.
  3. Then anonymous diners, with the login prompt.
- **A/B test:** personalized stack vs. cluster top picks only vs. no NextBite.
- **Rollback:** turn off the flag. AI reasons have their own flag, and templates are the fallback.

## 12. Success Metrics

| Metric | Target | Guardrail |
|--------|--------|-----------|
| UAE monthly discovery views | 4k → **15k+** | — |
| Weekly returning NextBite users (habit) | Baseline in month 1, then grow | — |
| Right-swipe (save) rate on picks | Rises week over week per diner (learning works) | — |
| Saved dishes later ordered via qlub ("tried") | Baseline, then improve | — |
| Pick → qlub payment at that restaurant within 7 days | Higher than the cluster-only control | — |
| Onboarding completion | ≥ 60% complete 10 swipes (A5) | Drop-off > 50% → shorten it |
| AI reasons passing the fact check | ≥ 95% | Wrong facts shown = 0 |
| Login conversion from the "log in for better picks" prompt | Informational | — |
| Left-swipe-on-everything sessions | Falls over time | If high, review clusters and the scoring |

---

## 13. Open Questions and Assumptions

**Blocking questions (before the pilot):**

| ID | Question |
|----|----------|
| Q1 | Can we use real logged-in order data for clustering? Are the numbers mentioned (~60k qlub+ subscribers, 100k+ logged-in customers) correct? |
| Q2 | What share of menu items have vendor-approved photos today? (Yağmur has the data; thought to be > 50%.) |
| Q3 | Is item-level order data available for logged-in diners, so we can compute favourites and reorder scores? |
| Q6 | What consent and privacy rules apply to personalization and to sending anonymous fact sheets to an LLM provider? |

**Non-blocking questions:**

| ID | Question |
|----|----------|
| Q4 | How did the in-restaurant (QSR) recommendations perform? Can we get the report? |
| Q5 | Where do promotion data (qlub+ discounts, bank offers) come from, and can the demo use them? |
| Q7 | What is the name of the assistant behind "Not what you're looking for?" |
| Q8 | Can qlub+ restaurants be boosted in picks, and if so, how is that labeled? |
| Q9 | Can we use the existing product-click events and the saved-restaurants feature in the pilot? |

**Assumptions (ship defaults):**

| ID | Assumption | Impact if wrong |
|----|------------|-----------------|
| A1 | Pick mix 6 matches / 2 cluster / 2 exploration per 10 | Tune after the pilot |
| A2 | ~6 taste clusters | Too few → generic picks; too many → thin data |
| A3 | Weekdays = quick, close, usual budget; weekends = treats | Learn per diner instead |
| A4 | Feedback weights as in BR8 | Tune from the save and order rates |
| A5 | Onboarding of 10 swipes, ≥ 60% completion | Shorten to 6 |
| A6 | Reorder window 180 days; minimum 20 diners to rank, 50 to mention | Tune |
| A7 | Exploration share ~20% | Tune per diner |

**Slice order:** S1 → S1+ → S2 → S3. **First slice:** S1 (hackathon demo).
