A/B Test: Express Checkout (Google Pay / Apple Pay)

A/B test case study analyzing whether adding express payment buttons (Google Pay & Apple Pay) to checkout improves conversion, using session/event data modeled in BigQuery and visualized in Tableau.

2. Hypothesis

The development team added the ability for users to fill in payment details via Google Pay or Apple Pay instead of manual card entry.
Hypothesis: Reduction in checkout friction will:
- increase add_payment_info / session by 2%
- increase begin_checkout / session by 2%

Note: begin_checkout can occur without add_payment_info, so a lift in the primary metric without a proportional lift in the secondary metric (or vice versa) was considered a plausible, valid outcome - not a contradiction.

3. Variants

Variant | Description
---|---
A - Control | Standard checkout flow with manual card entry
B - Experimental | Updated checkout flow with Google Pay and Apple Pay buttons added

- Audience: All users reaching checkout
- Traffic split: 50% Control / 50% Variant

4. Metrics

Metric | Type | Target
---|---|---
begin_checkout / session | Primary | +2%
add_payment_info / session | Secondary | +2%

5. Data & Methodology

Session, event, order, and account data were combined in BigQuery into a single long-format table (date, ga_session_id, dimensions, event_name, value), keyed by A/B test group. This shape lets any event (including begin_checkout and add_payment_info) be sliced by test_group, device, channel, country, and continent directly in Tableau.
Significance was evaluated per metric, site-wide, as well as within device, continent, and channel segments.

6. Results

Site-wide

Metric | Result | Significance
---|---|---
begin_checkout | +6.26% | Significant
add_payment_info | +12.12% | Significant

Both metrics beat their +2% targets by a wide margin, and traffic was evenly split across variants, so the site-wide lift can be trusted.

By Device

Device | begin_checkout | add_payment_info
---|---|---
Desktop | +14.04% (significant) | +11.15% (significant)
Mobile | −2.01% (not significant) | +16.30% (significant)
Tablet | −32.53% (not significant) | −35.42% (not significant)

- Desktop drove the strongest, cleanest result across both metrics.
- Mobile saw a strong, significant lift in add_payment_info (users happily adopted the wallet buttons), while begin_checkout was flat - a plausible outcome given the two events aren't strictly dependent.
- Tablet showed a large negative swing on both metrics, but on a much smaller sample, so it did not reach statistical significance. It's a signal to investigate, not a confirmed regression.

By Continent

Continent | begin_checkout | add_payment_info | Volume / Significance
---|---|---|---
Europe | +15.35% | +39.51% | High volume; both significant
Americas | +1.60% | +6.32% | Highest volume; add_payment_info significant
Asia | +9.51% | +6.44% | High volume; begin_checkout significant
Africa | −38.89% | +26.32% | Low volume; not significant
Oceania | +55.56% | +10.53% | Low volume; not significant

- Europe showed lifts in both metrics, with p < 0.05.
- Americas (largest volume) showed a lift in add_payment_info (+6.32%, p < 0.05), while begin_checkout remained flat (+1.60%, p > 0.05).
- Asia showed a lift in begin_checkout (+9.51%, p < 0.05) and add_payment_info (+6.44%, p > 0.05).
- Africa and Oceania had low sample sizes (~500 sessions per group), resulting in high variance without statistical significance.

By Channel

Channel | Traffic Share | Traffic Split (A/B) | Consistency
---|---|---|---
Organic Search | ~35% | 35% / 35% | Balanced
Paid Search | ~26% | 26% / 26% | Balanced
Direct | ~24% | 24% / 23% | Balanced
Social Search | ~9% | 9% / 9% | Balanced
Undefined | ~7–8% | 7% / 8% | Balanced

- Traffic distribution across acquisition channels remained identical between Control and Experimental groups.
- No Sample Ratio Mismatch (SRM) or channel bias was detected.

7. Conclusions & Recommendations

The test is a success. Express checkout via Google Pay/Apple Pay meaningfully improved the funnel: begin_checkout (+6.26%) and add_payment_info (+12.12%) both cleared their +2% targets with statistical significance, and simplifying payment entry did not cannibalize other checkout paths

The device-level breakdown indicates potential risk in the tablet segment, which experienced declines in both tracked metrics. Although this variance does not reach statistical significance due to the limited sample size, but a shift has to be further investigated

Segment Double-Check Signals:
- Tablet segment: Declines in both metrics require a viewport rendering and interaction audit.
- Africa region: `begin_checkout` dropped by 38.89%. Low sample size makes this non-significant, but technical gateway compatibility requires verification.

Recommended rollout:
- Ship Variant B to 100% of Desktop and Mobile traffic in Europe, Americas, and Asia.
- Hold Tablets on Variant A (control) until a UX/technical audit confirms the express payment buttons render and trigger correctly on tablet viewports and browsers.
- Audit technical setups in Africa prior to enabling Variant B in that region.
