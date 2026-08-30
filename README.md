# A/B Test: Express Checkout (Google Pay / Apple Pay)

A/B test case study analyzing whether adding express payment buttons (Google Pay & Apple Pay) to checkout improves conversion, using session/event data modeled in BigQuery and visualized in Tableau.

## Project Structure

```
├── README.md
├── ab_test_query.sql     -- BigQuery query used to build the analysis dataset
└── dashboard             -- Tableau dashboard file 
```

## 2. Hypothesis

The development team added the ability for users to fill in payment details via Google Pay or Apple Pay instead of manual card entry.

**Hypothesis:** Reduction in checkout friction will:
- increase `add_payment_info / session` by **2%**
- increase `begin_checkout / session` by **2%**

Note: `begin_checkout` can occur without `add_payment_info`, so a lift in the primary metric without a proportional lift in the secondary metric (or vice versa) was considered a plausible, valid outcome — not a contradiction

## 3. Variants

| Variant | Description |
|---|---|
| **A — Control** | Standard checkout flow with manual card entry  |
| **B — Experimental** | Updated checkout flow with Google Pay and Apple Pay buttons added |

| | |
|---|---|
| **Audience** | All users reaching checkout  |
| **Traffic split** | 50% Control / 50% Variant |

## 4. Metrics

| Metric | Type | Target |
|---|---|---|
| `begin_checkout / session` | Primary | +2% |
| `add_payment_info / session` | Secondary | +2% |

## 5. Data & Methodology

Session, event, order, and account data were combined in BigQuery into a single long-format table (`date, ga_session_id, dimensions, event_name, value`), keyed by A/B test group. This shape lets any event (including `begin_checkout` and `add_payment_info`) be sliced by `test_group`, `device`, `channel`, `country`, and `continent` directly in Tableau — see [`ab_test_query.sql`](.https://public.tableau.com/app/profile/karina.ohanisian/viz/ab_t_1/Dashboard1).

Significance was evaluated per metric, both at the site-wide level and within each device segment

## 6. Results

### Site-wide

| Metric | Result | Significance |
|---|---|---|
| `begin_checkout` | **+6.26%** | significant |
| `add_payment_info` | **+12.12%** | significant |

Both metrics beat their +2% targets by a wide margin, and traffic was evenly split across variants, so the site-wide lift can be trusted.

### By Device

| Device | `begin_checkout` | `add_payment_info` |
|---|---|---|
| **Desktop** | **+14.04%**  significant | **+11.15%**  significant |
| **Mobile** | −2.01% not significant | **+16.30%**  significant |
| **Tablet** | −32.53% not significant | −35.42% not significant |

- **Desktop** drove the strongest, cleanest result across both metrics.
- **Mobile** saw a strong, significant lift in `add_payment_info` (users happily adopted the wallet buttons), while `begin_checkout` was flat — a plausible outcome given the two events aren't strictly dependent.
- **Tablet** showed a large negative swing on both metrics, but on a much smaller sample, so it did **not** reach statistical significance. It's a signal to investigate, not a confirmed regression.

## 7. Conclusions & Recommendations

**The test is a success.** Express checkout via Google Pay/Apple Pay meaningfully improved the funnel: `begin_checkout` (+6.26%) and `add_payment_info` (+12.12%) both cleared their +2% targets with statistical significance, and simplifying payment entry did not cannibalize other checkout paths.

The device-level breakdown surfaces a real risk, though: Tablet trended sharply negative on both metrics. With a small sample size the result isn't statistically significant, but a swing of that size shouldn't be ignored.

**Recommended rollout:**
- ✅ **Ship Variant B to 100% of Desktop and Mobile traffic.**
- ⏸ **Hold Tablet on Variant A (control)** until a UX/technical audit confirms the express payment buttons render and trigger correctly on tablet viewports and browsers.

## 8. Dashboard

> **TODO:** Link your published Tableau dashboard and/or add screenshots.
> - `[View live dashboard on Tableau Public](your-link-here)`
> - `![Dashboard overview](dashboard/overview.png)`

**Suggested views:**
- `begin_checkout` and `add_payment_info` rate by test group (site-wide)
- Same metrics broken out by device (Desktop / Mobile / Tablet)
- Significance indicator per segment (e.g. confidence interval or highlight for significant vs. non-significant results)

## 9. How to Reproduce

1. Run [`ab_test_query.sql`](./ab_test_query.sql) in BigQuery against the `DA` dataset.
2. Connect the result set to Tableau (native BigQuery connector or export to extract/CSV).
3. Build/refresh the dashboard views listed above, filtering `event_name` to `begin_checkout` and `add_payment_info` for this test's `test` value.

## 10. Notes / Next Steps

- Run a formal UX audit on tablet checkout to understand the negative — but not yet significant — trend before making a rollout decision for that segment.
- Consider extending the test duration or widening tablet traffic allocation in a follow-up test to reach significance on that segment specifically.
