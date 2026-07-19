# AI Gift Recommendation System

**[Download PRD (PDF)](Dushyanth_Gift_Recommendation_PRD.pdf)**

**An AI-powered gifting engine for a fast-fashion retail app targeting teenage users.**

A fast-fashion retailer hit 10x growth in three years by capturing the U.S. teenage self-purchase market — then ran into a ceiling. The core demographic was saturated. The next growth lever was already sitting in the data: customers were discussing style preferences with 2-3 close friends and family inside the app. This PRD defines an MVP that monetizes that existing social behavior by turning self-purchasers into gifters.

---

## The Problem

The business had plateaued in its primary segment. Existing teenage users had high engagement (NPS of 75, above Amazon's) and established purchase habits, but the self-purchase market was saturated. Growing transaction volume required unlocking a new use case — gifting — without disrupting the behavior that already worked.

The opportunity: users already had social graphs inside the app, existing payment methods on file, and saved addresses. 35% of target customers had made 2+ purchases in the past 12 months. The friction to gift was low. The behavior just wasn't being surfaced or supported.

---

## Goal

Build an AI recommendation engine that suggests relevant products for existing teenage customers to purchase as gifts for friends and family — surfacing recommendations at key moments in the app and via email — and deliver it to 100% of users by November 1st to capture the full holiday season.

---

## Target Segment

Existing teenage users (ages 13–19) who:
- Have at least one prior purchase in the past 12 months (brand familiarity, payment method on file)
- Are connected to 2+ friends or family members in the app
- Have saved addresses reducing checkout abandonment
- 35%+ have made 2+ purchases in the past year, indicating both intent and affordability

---

## Success Metrics (First 60 Days)

| Metric | Target | Rationale |
|---|---|---|
| New gifters acquired | 150,000 | 30% of active user base attempting gifting for the first time |
| Gift transactions as % of total | 8-12% | Gifting should grow from near-zero to a meaningful share of transaction mix |
| Repeat gifting rate (2+ gifts in 60 days) | 35% | If repeat rate approaches self-purchase rate, gifting becomes habitual |
| Average order value: gift vs. self-purchase | $5-8 higher | Gift purchases consistently outperform self-purchases across retail |
| Gift transaction conversion rate | 2.5-3.5% | Above baseline self-purchase conversion of 2% |
| Total gifting GMV | $5M+ | Conservative: 150K gifters × $35 avg order value |
| Overall app transaction volume increase | +15-20% | Gifting adds incremental volume without cannibalizing self-purchase |
| Customer LTV uplift for users exposed to gifting | +$25-50 | Gifting expands use cases and increases wallet share |

---

## AI Architecture

### Hybrid Recommendation Engine (Collaborative + Content-Based Filtering)

A single-model approach is insufficient for the gifting use case. Two distinct problems need solving:

1. **Who to buy for** — solved by social graph analysis and past gifting history
2. **What they'll like** — solved by matching recipient purchase behavior to product attributes

The hybrid approach combines:
- **User-Based Collaborative Filtering** — identifies friends with similar taste profiles and recommends products they've engaged with
- **Content-Based Filtering** — matches past purchase behavior to product attributes for recipients

Recommendation ranking: **70% recipient past purchase similarity + 20% user past gifting patterns + 10% trending in recipient's location**

### Data Classification

Explicitly classifying each data source as exogenous or autoregressive matters because it determines how the model should be trained and how recommendations degrade over time.

| Data Source | Type | Why It Matters |
|---|---|---|
| Purchase history (past 12 months) | Exogenous | Ground truth — fixed facts that don't depend on prior recommendations |
| Past gift purchases for specific recipients | Autoregressive | Learned patterns — improve with each gifting transaction, making the model more accurate over time |
| Social connections and interests | Exogenous | Static facts — who to buy for; prevents recommendations outside the user's social graph |
| Browsing and engagement signals | Autoregressive | Dynamic feedback loop — add-to-cart and wishlist activity reveal intent before purchase |
| Demographic and contextual data | Exogenous | Static attributes — enables occasion-triggered and location-aware recommendations |
| Trending products by occasion and geography | Exogenous | Market signal — prevents over-narrowing; surfaces popular gifts new to the recipient |

Exogenous data is safe to use immediately. Autoregressive data improves as gifting volume accumulates — the model gets materially better after the first holiday season.

### Demand Forecasting: Time-Series Autoregressive Model

A separate forecasting model predicts gifting demand by occasion, product category, and time horizon — enabling inventory planning and marketing spend allocation before the holiday peak.

Data inputs: historical gifting transactions (autoregressive), calendar events and holidays (exogenous), user cohort age distribution (exogenous), competitor gifting signals (exogenous), economic indicators (exogenous).

---

## User Stories

### P0 — Core gifting experience

**Gift Recommendation Feed (Gifting Tab)**
When a user opens the Gifts tab, they see 8-12 personalized recommendations paired with suggested recipients from their social graph. Each card shows product image, price, recipient name, reason for the pick, and a direct "Gift to [Friend]" button that pre-fills checkout with the recipient's address. Feed refreshes every 24 hours.

**Occasion-Triggered Notifications**
14 days before a friend's birthday or major holiday, the app sends an in-app notification with 2-3 specific recommendations for that person. Tapping opens a curated feed of 6-8 occasion-specific recommendations. Capped at 1-2 notifications per month per user to avoid fatigue.

**Purchase Intent Capture**
At checkout, a modal asks: "Is this for you or a gift?" If gift, follow-up captures recipient and occasion. Data stored with `purchase_type`, `recipient_id`, `occasion` metadata tags. Editable for 24 hours post-purchase. Feeds both the recommendation model and the forecasting model.

**Social Gifting Attribution**
When a gift is delivered, the recipient gets a notification and can view it in a "Gifts Received" section. Reactions and favorites are stored as `recipient_reaction`, creating a feedback loop — if a recipient consistently receives hoodies, the system infers preference and weights similar items higher in future recommendations.

### P1 — Business intelligence

**Executive Dashboard**
Real-time tiles: gift transactions MTD, new gifters acquired, gift GMV, gift attach rate, average gift order value. 90-day forward forecast with confidence intervals. Category performance heatmap. Recipient segment breakdown (friend, parent, sibling). Conversion funnel by occasion.

**Weekly Stakeholder Email (Monday morning)**
Prior week: total gift transactions, GMV, new gifters, repeat gifters vs. target. Forward 4 weeks: predicted gifting volume and GMV by week with peak week flagged. Top 5 gift categories. Confidence note explaining forecast reliability based on data availability.

---

## Trade-offs

**Notification volume vs. conversion.** More notifications drive short-term clicks but erode long-term permission rates. The 1-2 per month cap is conservative relative to what would maximize 60-day GMV — but protecting notification permission protects lifetime value.

**Personalization vs. discovery.** Pure collaborative filtering risks over-narrowing to items similar to what the recipient has already purchased. The 10% trending weight in the ranking formula ensures new products surface, balancing relevance with discovery.

**Cold-start for new gifters.** Users with no prior gifting history get recommendations based on recipient purchase patterns only (no autoregressive component yet). This produces weaker first-session recommendations. The purchase intent capture at checkout (User Story E) is the primary mechanism for bootstrapping gifting data quickly.

**Model accuracy improves after, not before, the first holiday season.** The autoregressive components (past gift purchases, engagement signals) need gifting transaction volume to learn from. The MVP launches with a cold model; Q1 analysis and model retraining is a required post-launch step.

---

## Open Questions

- **Consent for younger users (ages 13–15):** Sharing social graph data and recipient profiles may require parental consent under COPPA. Legal review needed before launch.
- **Recipient opt-out:** Can a user's friend opt out of being a suggested gift recipient? Absence of an opt-out creates a privacy exposure.
- **Notification channel:** In-app notifications are lower-friction to implement but have lower reach for inactive users. Email vs. push notification strategy needs a channel test before holiday launch.
- **Group gifting scope:** The PRD scopes this to future roadmap, but if group gifting (multiple friends pooling) is close to MVP quality, it may be worth pulling forward — average order value on group gifts is materially higher.

---

## Roadmap (Post-MVP)

- Gifting recommendations for younger siblings
- Physical gift cards and e-gift options
- Group gifting (multiple friends pooling for a larger gift)
- AI-powered occasion discovery (surfacing gifting moments the user didn't know about)

---

## About This Project

Product requirements document for an AI recommendation system built on top of an existing social graph. The technical architecture section explicitly classifies each data source as exogenous or autoregressive — a distinction that matters for model training, feedback loop management, and understanding how recommendation quality changes over time.
