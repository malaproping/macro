# Macro research findings — V4 replication and handoff review

Reviewed November 23, 2026 at 8:56:46 p.m. EST (America/Detroit).

**Decision.** Retain V4 as a regression baseline. Its confirmed shocks beat matched non-shock days by about 2 basis points per event across the full sample. Excluding 2020 changes the difference to −29 basis points. Before using the engine to evaluate H01, address closing-auction sizing, forward-coverage filtering, and dividend accrual.

## What was verified

The review covered the full handoff and the current workbook. The price matrix contained 4,248 daily rows and 38 price or index columns from January 4, 2010 through November 20, 2026. The V4 evaluation period began November 21, 2011. The repository contained only the handoff at review time. The original V4 code and frozen input snapshot were unavailable, so this replay establishes agreement with the published figures on the current matrix. [Handoff §1.2](../MACRO_RESEARCH_AGENT_HANDOFF.md#12-existing-research-assets) · [Workbook price matrix](https://docs.google.com/spreadsheets/d/1RMjRFPdnSzjRH2iTKBlijcKO2dt4z3ho0PKOZls-_CY/edit#gid=4701)

The V3 summary cells agreed with the handoff at its displayed precision. The review checked those cells without rerunning the V3 strategies. For V4, independent calculations reproduced all eight trend and pairs strategy rows, all five shock-event rows, the matched comparison, and the confirmed portfolio at the reported precision. The replay used the legacy assumptions of price-only ETF returns, 5 basis points per side, 3% annual short borrow, zero interest on cash, and omitted distributions. [V3 results](https://docs.google.com/spreadsheets/d/1RMjRFPdnSzjRH2iTKBlijcKO2dt4z3ho0PKOZls-_CY/edit#gid=4801) · [Handoff §12](../MACRO_RESEARCH_AGENT_HANDOFF.md#12-exact-legacy-v4-replication-recipe)

| V4 replay measure | Result |
|---|---:|
| All shocks | 1,056 |
| Confirmed / opposed / unconfirmed shocks | 406 / 189 / 461 |
| Confirmed 20-session win rate after modeled costs | 51.72% |
| Confirmed portfolio net CAGR | +0.8818% |
| Confirmed portfolio gross CAGR | +1.4832% |
| Maximum drawdown | −13.3113% |
| Annualized volatility | 4.7446% |
| Average gross exposure | 21.5709% |

### Dependence on 2020

| Confirmed events by signal year | Events | Mean net 20-session return | Matched control | Difference |
|---|---:|---:|---:|---:|
| Full sample | 406 | +0.3626% | +0.3432% | +0.0194 pp |
| 2020 | 30 | +5.9906% | +2.1125% | +3.8781 pp |
| All other years | 376 | −0.0864% | +0.2020% | −0.2885 pp |

Differences are percentage points, calculated before rounding. Each confirmed event received equal weight. Its control was the mean net directional return on eligible non-shock days in the same instrument, calendar year, and macro-proxy direction. The matching procedure could use later dates in the same year, and return windows overlap. These numbers describe a retrospective event comparison. The portfolio CAGR above comes from a separate simulation with actual overlap and capital allocation. [Handoff §12.2](../MACRO_RESEARCH_AGENT_HANDOFF.md#122-shock-event-study)

## Findings for implementation

| Priority | Finding and consequence | Change and acceptance check |
|---|---|---|
| High | **Closing-auction sizing.** V4 uses the execution close to mark NAV and buy or sell an exact portfolio weight. The final share quantity depends on a price revealed after order submission. | Preserve the convention for legacy replay. For economic runs, size from a pre-cutoff mark and submit a fixed quantity. Changing only the auction price must change realized exposure while leaving submitted quantity identical. |
| High | **Forward coverage affects portfolio eligibility.** The 20-session outcome filter dropped a confirmed GLD short on November 12, 2026. It qualified for entry on November 13 and was still open at the November 20 reporting cutoff. | Apply the complete-window filter when comparing event outcomes. Generate portfolio candidates from data known at decision time, mark open positions through the cutoff, and flag incomplete outcomes. Use the GLD event as a regression case. |
| High | **Dividend entitlement is missing from the NAV formula.** A cash-plus-shares ledger can show an artificial loss on ex-date and gain on payment. Crediting spendable cash on ex-date instead makes financing available too early. | Accrue a receivable for a long and a payable for a short on ex-date. Settle to cash on payment. Verify that closing either position between those dates preserves the entitlement or liability and that settlement alone leaves NAV unchanged. |
| Medium | **H01 permits materially different primary tests.** The example allows a rate change or fitted PCA, leaves the matched-control algorithm open, and permits different treatments of unknown borrow. | Before evaluating outcomes, freeze one surprise series, its sign convention, the matching algorithm, eligibility, and borrow treatment. If using PCA, freeze its training window and the rule that orients its sign. |
| Medium | **SOXX has two relevant split dates.** The current matrix already shows 76.62, 79.25, and 76.04 on March 6–8, 2024. The corrections tab records 237.75 becoming 79.25 on March 7. Another 3-for-1 split took effect November 5, 2026. Reapplying the correction to this matrix would create a spurious price shock. | Record the adjustment basis of each imported price series and reconcile both dates against issuer filings. Verify the normalized March 7 value stays 79.25 when replaying this workbook version. |
| Medium | **M1 lacks precise replication criteria.** Running twice checks determinism, but a wrong weighting or overlap rule can repeat perfectly. | Pin exact event counts, tolerances reflecting published metric precision, and dated examples for fills and overlap handling. Fix each instrument’s share of its group using the full group count. Inactive shares stay in cash. This convention reproduced the reported trend results. |

The relevant specification sections are [execution and overlap](../MACRO_RESEARCH_AGENT_HANDOFF.md#82-entries-exits-and-event-overlap), [economic accounting](../MACRO_RESEARCH_AGENT_HANDOFF.md#9-simulation-and-economic-accounting), [H01 configuration](../MACRO_RESEARCH_AGENT_HANDOFF.md#73-example-configuration), and [M0–M1 acceptance](../MACRO_RESEARCH_AGENT_HANDOFF.md#161-first-coding-agent-assignment-m0m1). Primary references for the timing and corporate-action checks are the [NYSE auction timelines](https://www.nyse.com/trade/auctions), [iShares 2024 split filing](https://www.sec.gov/Archives/edgar/data/1100663/000119312523298660/d611816d497.htm), and [iShares 2026 split filing](https://www.sec.gov/Archives/edgar/data/1100663/000119312526430480/d156277d497.htm).
