# NextBite — Dish Tags

Every menu item in the dish catalogue gets the tags below. They feed four things in the [Recommendation PRD](../NextBite-Recommendation-PRD.md): the diner's taste profile (BR2), the pick mix and exploration (BR4), the guided flow tiles (J3, BR15) and the AI reason fact sheet (BR12).

**Rules**
- Every tag uses a **fixed list of values**. The AI (Menu Brain enrichment) has to choose from the list and can't invent new values, so the tags stay usable for filtering and clustering.
- Every AI-filled tag can be `unknown`. Unknown is better than a wrong guess (see Search PRD BR12: unknown is kept but ranked lower).
- **Source** column: **Menu** = taken from qlub menu data, **AI** = filled by the LLM from name + description + category, **Data** = computed from orders and prices.
- **Must** tags are needed for the S1 demo. **Should** tags come later if there's time.

---

## 1. Dish tags

| # | Tag | Values | Multi? | Source | Used for | Priority |
|---|-----|--------|--------|--------|----------|----------|
| T1 | `cuisine` | italian, american, japanese, chinese, thai, korean, vietnamese, pan_asian, indian, pakistani, lebanese, emirati, turkish, persian, arabic_other, mediterranean, greek, mexican, latin, french, spanish, british, international, fusion, unknown | Yes, max 2 | Menu (restaurant cuisine) → AI per dish | Profile cuisine mix, exploration picks (R13), onboarding coverage (BR6) | Must |
| T2 | `dish_type` | pizza, burger, sandwich_wrap, pasta, rice_bowl, noodles, sushi, curry, grill_kebab, shawarma, steak, fried_chicken, seafood, salad, soup, mezze_dip, bakery, breakfast_plate, dessert, drink_hot, drink_cold, shisha, other | No | AI | Guided flow step 1, "no burgers" exclusions, top level of the taste profile | Must |
| T3 | `dish_style` | Free text, but normalised per `dish_type` (e.g. pizza → margherita, pepperoni, truffle, bbq_chicken; burger → smash, wagyu, chicken, veggie) | No | AI | Guided flow step 2 ("which kind of pizza?"). Only shown when ≥ 3 nearby dishes share it (BR15) | Should |
| T4 | `main_ingredients` | chicken, beef, lamb, pork, fish, seafood, egg, cheese_dairy, tofu_plant, rice, pasta_noodles, bread_dough, potato, vegetables, mushroom, legumes, fruit, chocolate, nuts, unknown | Yes, max 3 | AI | Taste profile, exclusions ("no seafood"), reason facts ("also lamb") | Must |
| T5 | `spice_level` | none, mild, spicy, very_spicy, unknown | No | AI | Taste profile, "not spicy" (strict exclude) | Must |
| T6 | `flavour` | savoury, sweet, sour_tangy, smoky_grilled, creamy_cheesy, rich, fresh_light, umami, herby, truffle | Yes, max 3 | AI | Similar-dish matching, AI reasons ("creamy like the risotto you ordered") | Must |
| T7 | `cooking_method` | grilled, fried, baked, roasted, steamed, raw, slow_cooked, stir_fried, unknown | No | AI | "light vs heavy" profile, healthy picks | Should |
| T8 | `diet` | vegetarian, vegan, halal, gluten_free, dairy_free, contains_nuts, contains_pork, contains_alcohol | Yes | AI, shown as "based on menu description" | Hard exclusions only, never health guesses in reasons (BR12) | Should |
| T9 | `course` | starter, main, side, dessert, drink, shisha, combo | No | Menu category → AI | Only `main` (and some `dessert`) go into the swipe stack | Must |
| T10 | `meal_time` | breakfast, lunch, dinner, late_night, any | Yes | AI + Data (order hour) | Context ranking (R12): breakfast dishes in the morning, not at 10 pm | Should |
| T11 | `occasion` | quick_bite, everyday, treat, sharing | Yes | AI + Data (price, party size) | Weekday vs weekend picks (BR7): weekday → quick_bite/everyday, weekend → treat | Should |

## 2. Computed tags (from data, not AI)

| # | Tag | How it's computed | Used for | Priority |
|---|-----|-------------------|----------|----------|
| C1 | `price` | 5 levels: `very_cheap`, `cheap`, `reasonable`, `expensive`, `too_expensive`. Price is compared with other dishes of the same `dish_type` in the same city and split into fifths (lowest 20% = `very_cheap` … top 20% = `too_expensive`) | Price fit in scoring, "In your usual price range" reason, budget slider in onboarding | Must |
| C2 | `reorder_score` | Share of diners who ordered the dish again within 60 days | Quality signal (no star ratings needed), "People keep coming back for this" | Must |
| C3 | `popularity` | Orders in the last 4 weeks, normalised per restaurant | Cluster top picks, onboarding dish choice | Must |
| C4 | `trending` | Orders this week vs. the 4-week average, ≥ 1.5× | "Clubbers are loving these" badge | Should |
| C5 | `has_photo` / `photo_status` | approved, ai_pending, none | Only dishes with approved photos go to onboarding (BR6) and to the stack front | Must |
| C6 | `promo` | qlub+ discount %, bank offer | Promotion badge and small boost (BR7) | Should |
| C7 | `signature` | Top 3 most ordered mains in the restaurant | "Most loved here" badge, onboarding | Should |

## 3. Restaurant tags (inherited by every dish)

Not dish tags, but the stack needs them on each card: `area` (e.g. Dubai Marina, JLT), `lat`/`lng`, `restaurant_price_level` (1–4), `vibe` (casual, family, fine_dining, cafe, lounge_shisha, fast_casual), `opening_hours`.

---

## 4. What the S1 demo needs

Only the **Must** tags: `cuisine`, `dish_type`, `main_ingredients`, `spice_level`, `flavour`, `course`, plus `price`, `reorder_score`, `popularity`, `photo_status`. That is enough for the taste profile, the pick mix, clustering and the AI reasons. `dish_style` is the first one to add if the guided flow (R9) gets built on Day 2.

## 5. Enrichment prompt shape (Menu Brain)

One LLM call per dish (batched), input = restaurant cuisine + category + dish name + description + price. Output = JSON with the tags from section 1, every value from the lists above, `unknown` when not sure. Validate against the lists; any value not on the list → `unknown`.

```json
{
  "item_id": "12345",
  "cuisine": ["italian"],
  "dish_type": "pizza",
  "dish_style": "truffle",
  "main_ingredients": ["cheese_dairy", "bread_dough", "mushroom"],
  "spice_level": "none",
  "flavour": ["creamy_cheesy", "truffle", "rich"],
  "cooking_method": "baked",
  "diet": ["vegetarian"],
  "course": "main",
  "meal_time": ["lunch", "dinner"],
  "occasion": ["everyday", "sharing"]
}
```

## 6. Open questions

1. Does qlub's menu data already have a cuisine on the restaurant or on the item? If yes, T1 comes from Menu and AI only fills the gaps.
2. Should vendors be able to see and correct the tags in the vendor panel (same flow as AI photos)? (Search PRD Q10)
3. Do we show `halal` at all? In the UAE almost everything is halal, so it may be more useful to tag only the exceptions (`contains_pork`, `contains_alcohol`).
