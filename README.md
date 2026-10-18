# AI Exposure vs. Projected Employment Change (U.S., 2025–2035)

A Power BI analysis of all 831 detailed U.S. occupations, combining the Bureau of Labor Statistics' first-ever **AI exposure classification** (published August 27, 2026) with its **2025–35 employment projections**.

**The question:** Where does high AI exposure line up with projected job decline, and where does it line up with projected growth?

---

## Key findings

**1. Exposure is widespread. Projected decline is concentrated.**

56% of U.S. employment (roughly 95.4 million workers) sits in occupations BLS rates **High** or **Very high** for relative AI exposure. But only **34.4 million** of those workers are in occupations actually projected to shrink by 2035 — about a third of the exposed workforce. The other two-thirds are in occupations projected to hold steady or grow.

This is the opposite of the common framing. Exposure is not a decline forecast, and the data shows the two diverge sharply.

**2. Highly exposed occupations are larger than average.**

The four exposure categories contain almost identical numbers of occupations (213 / 206 / 206 / 206), but very different employment:

| Exposure | Occupations | Employment 2025 | Share | Avg. size |
|---|---|---|---|---|
| Low | 213 | 30,264,700 | 17.8% | 142,100 |
| Moderate | 206 | 44,651,300 | 26.2% | 216,800 |
| High | 206 | 42,142,000 | 24.8% | 204,600 |
| Very high | 206 | 53,209,200 | 31.2% | 258,300 |

A Very high exposure occupation employs, on average, 1.8× as many people as a Low exposure one.

**3. The declining-and-exposed cluster is office and administrative work.**

Among the largest occupations, the pattern is stark:

| Occupation | Exposure | Employment | Projected change |
|---|---|---|---|
| Retail salespersons | Very high | 3,998,100 | −0.3% |
| Cashiers | High | 3,106,300 | −6.5% |
| Customer service representatives | Very high | 2,666,000 | −5.3% |
| Office clerks, general | Very high | 2,600,000 | −6.0% |
| Secretaries and administrative assistants | Very high | 1,889,800 | −6.0% |
| Bookkeeping, accounting, and auditing clerks | Very high | 1,532,400 | −5.6% |

Meanwhile the fastest-growing large occupations are physical and in-person, and rate Low or Moderate:

| Occupation | Exposure | Employment | Projected change |
|---|---|---|---|
| Home health and personal care aides | Moderate | 4,677,100 | +18.1% |
| Fast food and counter workers | Moderate | 3,855,900 | +5.8% |
| Construction laborers | Low | 1,489,300 | +7.3% |
| Nursing assistants | Low | 1,505,900 | +2.6% |

**4. High exposure does not mean decline.**

The clearest counterexamples sit at the top of the exposure scale and the top of the growth scale simultaneously:

- **Software developers** — Very high exposure, 1,717,800 workers, **+10.2%**
- **Data scientists** — Very high exposure, 275,600 workers, **+34.6%**
- **Nurse practitioners** — High exposure, 336,300 workers, **+41.0%**

**5. Replacement demand dwarfs net growth.**

Net projected growth across the decade is 5.9 million jobs, but average **annual** openings are 17.5 million. Most hiring replaces workers who leave an occupation rather than filling newly created positions. A declining occupation can still generate substantial annual openings.

---

## Data sources

All sources are works of the U.S. federal government and in the public domain. Raw downloads are included in `/data`.

| File | Source | Vintage |
|---|---|---|
| `ai-exposure-categories.xlsx` | [BLS AI exposure categories](https://www.bls.gov/emp/publications/ai-exposure-categories.htm) | Published Aug 27, 2026 |

The single BLS file carries the exposure category, 2025 and 2035 employment, numeric and percent change, annual openings, median wage, percent self-employed, typical entry education, work experience, and on-the-job training — all keyed to 6-digit 2018 SOC codes.

### How BLS built the exposure measure

BLS combined **five external sources** — three theoretical exposure measures and two based on observed AI usage — into a four-category classification covering all 831 occupations for which projections are produced:

- Felten, Raj & Seamans (AI capability / occupational ability overlap, 2020)
- Eloundou, Manning, Mishkin & Rock (LLM task-time reduction, 2023)
- Eisfeldt, Schubert, Taska & Zhang (LLM task exposure, 2023)
- Massenkoff & McCrory / Anthropic (observed Claude usage mapped to O*NET tasks, 2025)
- Tomlinson, Jaffe, Wang, Counts & Suri / Microsoft (observed Copilot usage mapped to work activities, 2024–25)

Each source was converted to a percentile rank, then reduced to two dimensions (median theoretical rank, median observed rank) and clustered into Low / Moderate / High / Very high.

---

## Data preparation

Done entirely in Power Query so every transformation is auditable. Steps, in order:

1. **Disabled Power BI's automatic type detection** (File → Options → Data Load). This matters: auto-typing inserts a `Changed Type` step immediately after header promotion, which converts the wage column to a number *before* suppression markers are removed, turning them into unrecoverable errors.
2. **Removed the title row**, then promoted headers.
3. **Filtered out footnote rows** by removing nulls in the SOC code column rather than using Remove Bottom Rows, so the query doesn't silently delete real data if BLS adds a note in a future release. Row count: 836 → **831**.
4. **Verified no aggregate rows.** BLS files often mix major-group totals (`43-0000`) with detail rows (`43-4051`), which double-counts every total. Landing at exactly 831 confirmed this file contains detail rows only.
5. **Replaced em-dash suppression markers.** BLS denotes unavailable values with `—`. Replaced with null using *Match entire cell contents*, before any type conversion.
6. **Converted employment from thousands to units.** BLS publishes employment in thousands (291.6 = 291,600). Multiplied `Emp2025`, `Emp2035`, `EmpChange`, and `AnnualOpenings` by 1,000. Percent and wage columns excluded.
7. **Added `ExposureSortOrder`** (Low=1 … Very high=4) via conditional column, then set `ExposureCategory` to Sort by Column. Without this, every visual sorts the categories alphabetically as High, Low, Moderate, Very high.
8. **Extracted `MajorGroupCode`** from the first two characters of the SOC code, kept as text so it joins correctly.
9. **Set all types explicitly**, as the final step.

### Validation checks

| Check | Expected | Result |
|---|---|---|
| Row count | 831 detailed occupations | 831 |
| SOC codes distinct / unique | 831 / 831 (no duplicates) | pass |
| Exposure categories | 4 | 4 |
| Major groups | 22 (55 Military Specific excluded by BLS) | 22 |
| Total employment 2025 | ~170M | 170,267,200 |
| Net projected change | BLS headline: 5.9M | 5,900,000 |
| Errors in any column | 0% | 0% |

`MedianWage` contains **[FILL IN]** nulls, the occupations for which BLS does not publish a wage. These rows are **retained** — filtering them out would understate every employment total. The weighted-wage measure excludes them inside the measure instead, keeping numerator and denominator consistent.

### External validation

Model output was checked against BLS's own published aggregates in *The Economics Daily*, ["Total employment projected to grow 3.5 percent between 2025 and 2035"](https://www.bls.gov/opub/ted/2026/total-employment-projected-to-grow-3-5-percent-between-2025-and-2035.htm) (September 21, 2026).

All 22 major occupational groups reproduce the published figures within rounding:

| Major group | BLS published | This model |
|---|---|---|
| Healthcare support | 1,122,700 / +13.3% | 1,123,000 / +13.3% |
| Healthcare practitioners and technical | 819,500 / +8.0% | 819,500 / +8.0% |
| Management | 844,200 / +6.2% | 843,800 / +6.2% |
| Business and financial operations | 628,600 / +5.5% | 628,500 / +5.5% |
| Computer and mathematical | 403,100 / +7.3% | 402,700 / +7.3% |
| Construction and extraction | 385,700 / +5.1% | 385,500 / +5.1% |
| Life, physical, and social science | 122,900 / +7.5% | 123,300 / +7.5% |
| **Office and administrative support** | **−752,200 / −4.0%** | **−752,200 / −4.0%** |
| **Economy-wide total** | **5.9M / +3.5%** | **5,917,300 / +3.5%** |

Residual differences of a few hundred jobs come from BLS publishing employment in thousands to one decimal place; summing detail rows rounds slightly differently than the published group aggregate.

Office and administrative support is both the largest projected decline of any major group and the group most heavily weighted toward Very high AI exposure — the clearest point of contact between the exposure data and the projections.

---

## Data model

A small star schema:

```
MajorGroups (22 rows)          Projections (831 rows)
  MajorGroupCode  ──1────*──►    MajorGroupCode
  MajorGroupName                 SOC, Occupation
                                 Emp2025, Emp2035, EmpChange
                                 PctChange, AnnualOpenings
                                 MedianWage, ExposureCategory
                                 EducationLevel, WorkExperience, OnJobTraining
```

Single-direction one-to-many. `MajorGroups` is a hand-entered lookup mapping SOC major group codes to names, so visuals show "Office and Administrative Support" rather than `43`.

---

## Measures

```dax
Employment 2025 = SUM(Projections[Emp2025])

Employment 2035 = SUM(Projections[Emp2035])

Projected Change = [Employment 2035] - [Employment 2025]

Projected Growth % = DIVIDE([Projected Change], [Employment 2025])

Occupation Count = COUNTROWS(Projections)

Total Annual Openings = SUM(Projections[AnnualOpenings])

Employment in High Exposure =
CALCULATE(
    [Employment 2025],
    Projections[ExposureCategory] IN { "High", "Very high" }
)

Share in High Exposure =
DIVIDE([Employment in High Exposure], [Employment 2025])

Share of Total Employment =
DIVIDE(
    [Employment 2025],
    CALCULATE([Employment 2025], ALL(Projections))
)
```

One measure worth explaining:

```dax
Employment-Weighted Median Wage =
VAR WagedRows =
    FILTER(Projections, NOT ISBLANK(Projections[MedianWage]))
RETURN
    DIVIDE(
        SUMX(WagedRows, Projections[Emp2025] * Projections[MedianWage]),
        SUMX(WagedRows, Projections[Emp2025])
    )
```

A plain `AVERAGE(MedianWage)` would weight a 3,000-person occupation the same as a 3-million-person one. Filtering inside the measure rather than in Power Query means the wage-suppressed occupations are excluded from *this* calculation without disappearing from employment totals.

---

## Report pages

### 1. Overview

![Overview page](overview.png)

KPI cards (total employment, projected change, share in high exposure, annual openings), employment by exposure category, and employment by occupational group with exposure as a stacked legend.

### 2. Exposure vs. Growth

![Exposure vs growth page](exposure-vs-growth.png)

Scatter of projected percent change against median wage, bubble-sized by employment and colored by exposure category, with a constant line at 0% growth. Beside it, the filtered list of High and Very high exposure occupations projected to decline — the 34.4 million figure.

### 3. Occupation Explorer

![Occupation explorer page](occupation-explorer.png)

All 831 occupations with exposure, growth, openings, wage, and entry education. Slicers on exposure, occupational group, and education; search on occupation name. `PctChange` carries a diverging color scale with white pinned at 0%, so decline reads red and growth reads green.

Exposure uses a single-hue sequential ramp (light → dark for Low → Very high) on every page, because the categories are ordered.

---

## Limitations

These come from the BLS documentation and apply directly to anything concluded from this dashboard.

**Exposure is not a forecast.** BLS states explicitly that an exposure category is not a forecast of employment growth or decline, not a wage forecast, not an estimate of the probability of AI adoption, not a worker replacement estimate, and not a productivity forecast.

**Exposure is relative, not absolute.** The categories rank occupations against each other within this method. "Low" does not mean unaffected, and "High" does not imply any absolute threshold.

**Automation and augmentation are not distinguished.** The classification does not separate AI that replaces work from AI that assists a worker doing it.

**The underlying AI capabilities are dated.** The three theoretical sources conceptualize AI capabilities as those available no later than mid-2023. The two observed sources may skew toward early adopters of specific models. Most sources focus on language models and exclude image and video generation.

**Occupations are treated as bundles of independent tasks.** All five sources score tasks, abilities, or work activities independently and do not account for bottlenecks, complementarities, or other non-separable features of a job.

**Some values are imputed.** Of 4,155 possible occupation-source combinations (831 × 5), 3,944 were observed and 211 imputed, affecting 75 occupations.

**Projections reflect many factors, not just AI.** The employment projections incorporate demographics, demand, industry structure, and technology together. The percent-change figures here cannot be attributed to AI alone, and BLS cautions that its projections framework cannot isolate the effect of a single technology.

**Analyst's note on the wage axis.** Page 3 shows an unweighted mean wage across occupations. The employment-weighted figure is the more meaningful one and is available as a separate measure.

---

## Reproducing this

1. Download the [AI exposure categories file](https://www.bls.gov/emp/ind-occ-matrix/ai-exposure-categories.xlsx) (or use `ai-exposure-categories.xlsx` in this repo)
2. In Power BI Desktop: File → Options → Data Load → **untick** "Automatically detect column types and headers for unstructured sources" under both GLOBAL and CURRENT FILE
3. Open `ai-exposure-dashboard.pbix` and point the source at your local file path
4. Refresh

Step 2 is not optional. Skipping it reintroduces the type-conversion errors described above.

---

## Files

```
├── README.md
├── ai-exposure-dashboard.pbix      Power BI report
├── ai-exposure-categories.xlsx     Raw BLS source file (public domain)
├── overview.png
├── exposure-vs-growth.png
└── occupation-explorer.png
```

---

*Source: U.S. Bureau of Labor Statistics, Employment Projections program — AI exposure categories and 2025–35 employment projections, published August 27, 2026. Public domain.*
