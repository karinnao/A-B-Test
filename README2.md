## A/B Test: Express Checkout (Google Pay / Apple Pay)

A/B test case study analyzing whether adding express payment buttons (Google Pay & Apple Pay) to checkout improves conversion, using session/event data modeled in BigQuery and visualized in Tableau

[![Tableau Dashboard](./assets/dashboard_preview.png)](https://public.tableau.com/app/profile/karina.ohanisian/viz/ab_t_1/Dashboard1)

**Hypothesis**

The development team added the ability for users to fill in payment details via Google Pay or Apple Pay instead of manual card entry.
Hypothesis: Reduction in checkout friction will:
- increase add_payment_info / session by 2%
- increase begin_checkout / session by 2%

Note: begin_checkout can occur without add_payment_info, so a lift in the primary metric without a proportional lift in the secondary metric (or vice versa) was considered a plausible, valid outcome - not a contradiction

**Variants**

Variant | Description
---|---
A - Control | Standard checkout flow with manual card entry
B - Experimental | Updated checkout flow with Google Pay and Apple Pay buttons added

- Audience: All users reaching checkout
- Traffic split: 50% Control / 50% Variant

**Metrics**

Metric | Type | Target
---|---|---
begin_checkout / session | Primary | +2%
add_payment_info / session | Secondary | +2%

**Data & Methodology**

Session, event, order, and account data were combined in BigQuery into a single long-format table (date, ga_session_id, dimensions, event_name, value), keyed by A/B test group. This shape lets any event (including begin_checkout and add_payment_info) be sliced by test_group, device, channel, country, and continent directly in Tableau
Significance was evaluated per metric, site-wide, as well as within device, continent, and channel segments

**Results:**

*Site-wide*

Metric | Result | Significance
---|---|---
begin_checkout | +6.26% | Significant
add_payment_info | +12.12% | Significant

Both metrics beat their +2% targets by a wide margin, and traffic was evenly split across variants, so the site-wide lift can be trusted

*By Device*

Device | begin_checkout | add_payment_info
---|---|---
Desktop | +14.04% (significant) | +11.15% (significant)
Mobile | −2.01% (not significant) | +16.30% (significant)
Tablet | −32.53% (not significant) | −35.42% (not significant)

- Desktop drove the strongest, cleanest result across both metrics
- Mobile saw a strong, significant lift in add_payment_info (users happily adopted the wallet buttons), while begin_checkout was flat - a plausible outcome given the two events aren't strictly dependent
- Tablet showed a large negative swing on both metrics, but on a much smaller sample, so it did not reach statistical significance. It's a signal to investigate, not a confirmed regression

*By Continent*

Continent | begin_checkout | add_payment_info | Significance
---|---|---|---
Europe | +15.35% | +39.51% | Both significant
Americas | +1.60% | +6.32% | add_payment_info significant
Asia | +9.51% | +6.44% | begin_checkout significant
Africa | −38.89% | +26.32% | not significant
Oceania | +55.56% | +10.53% | not significant

- Europe showed lifts in both metrics
- Americas showed a lift in add_payment_info, while begin_checkout remained flat 
- Asia showed a lift in begin_checkout and add_payment_info 
- Africa and Oceania had low sample sizes, resulting in high variance without statistical significance

*By Channel*

Channel | begin_checkout | add_payment_info | Significance
---|---|---|---
Direct | +11.18% | +31.63% | Both significant 
Paid Search | −0.93% | +15.33% | add_payment_info significant
Social Search | +11.00% | +11.40% | Neither significant
Organic Search | −8.41% | −19.69% | Both significant regression

- Direct traffic had the strongest performance with statistically significant increases on both metrics
- Paid Search users showed higher adoption of express payment buttons 
- Organic Search experienced statistically significant drops in both metrics and requires technical investigation

 **Conclusion:**

The test is a success. Express checkout via Google Pay/Apple Pay meaningfully improved the funnel: begin_checkout and add_payment_info both cleared their targets with statistical significance, and simplifying payment entry did not cannibalize other checkout paths

**Recommendations:**
- Ship Variant B to 100% of Desktop and Mobile traffic in Direct, Paid Search, and Social Search channels across Europe, Americas, and Asia
- Hold Organic Search traffic on Variant A (control) pending technical investigation into the conversion drop
- Hold Tablets on Variant A (control) until a UX/technical audit confirms the express payment buttons render and trigger correctly on tablet viewports and browsers
- Audit technical setups in Africa prior to enabling Variant B in that region. Low sample size makes this non-significant, but technical gateway compatibility requires verification
