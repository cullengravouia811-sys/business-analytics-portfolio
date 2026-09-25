# Predicting Early Charge-Off in SBA 7(a) Small-Business Loans: An Out-of-Time Validated Risk Model with Local Labor-Market Conditions

**Author:** Cullen Gravouia
**Course:** BUS 751 – Python for Business Analytics
**Date:** September 27, 2026
**Pathway:** Practical Business Application
**Target output:** Decision brief and interactive risk dashboard for small-business lenders and Small Business Development Center (SBDC) advisors, written in manuscript form so it can later be extended into a working paper

---

## 1. Introduction & Motivation

The U.S. Small Business Administration's 7(a) program is the federal government's main loan-guarantee program for small firms. A private lender makes the loan, and the SBA guarantees a share of it, which lets lenders serve borrowers who are thin on collateral or credit history. When a loan charges off, the lender loses the unguaranteed portion and the SBA (so, taxpayers and the fees other borrowers pay) absorbs the guaranteed portion. Early charge-offs, meaning losses in the first few years after approval, are the most costly kind: little principal has been repaid, and they point to a screening failure at origination rather than a shock that hit much later.

Lenders and SBA advisors make approval and structuring decisions (loan size, term, collateral, processing method) with information that is available *at the time of approval*. Most public analyses of SBA loan default use a single random train/test split on an older teaching dataset. That setup lets information from the future leak into training and ignores the fact that recent loans have not had time to default. So it does not tell a lender how well a model built on past cohorts would perform on next year's applicants. It also ignores the local economy the business operates in.

The stakes can be measured: every percentage point of early charge-off avoided on a lender's 7(a) book is recovered principal, lower guarantee losses, and capital that can be redeployed to creditworthy borrowers, including in regions like south Louisiana where small firms depend heavily on SBA-backed credit.

**Study question:** *Can approval-time borrower, loan, and local labor-market characteristics predict which SBA 7(a) loans will charge off within 36 months, and does that prediction hold up on future loan cohorts?*

## 2. Pathway & Target Contribution

- **Pathway:** Practical Business Application
- **Intended audience:** Credit officers at community banks and credit unions that originate 7(a) loans, SBDC business advisors who prepare applicants, and regional SBA district office staff. Louisiana results are reported as a separate subgroup for local stakeholders.
- **Contribution in one sentence:** After this study, a lender will have an honestly validated, interpretable estimate of a new applicant's 36-month charge-off risk, plus evidence on whether business age and local unemployment carry risk beyond loan structure and industry. Today that decision is made with rules of thumb or with models validated only in-sample.

## 3. Literature Review & Research Gap

**What prior work establishes.** Information asymmetry makes small-business lending hard: lenders cannot fully observe borrower quality, which leads to credit rationing (Stiglitz & Weiss, 1981). Small firms rely heavily on relationship lending and government-backed credit to close that gap (Berger & Udell, 1998). SBA-guaranteed loans measurably increase firm employment and growth (Brown & Earle, 2017), so screening quality in the program matters beyond the lender's balance sheet. Glennon and Nigro (2005) used survival analysis to show that SBA 7(a) default risk varies over the life of a loan and with economic conditions and loan characteristics. Li, Mickel, and Taylor (2018) published a large SBA loan dataset (FY1962–2014) for logistic-regression-based approval decisions, and it has become a widely used benchmark. In credit scoring more broadly, ensemble methods such as gradient boosting usually beat logistic regression, but by margins that depend on the evaluation design (Lessmann et al., 2015).

**The specific gap.** Three weaknesses run through applied SBA default modeling. (1) **Evaluation design:** Most models are tested on random splits, which mix old and new cohorts and overstate how well they would perform on future loans. (2) **Censoring and competing risks:** Recent loans look safe only because they have not had time to fail, and loans that prepay can never charge off. Treating paid-in-full and still-active loans the same way biases default rates (Fine & Gray, 1999). (3) **Omitted local context:** Public models seldom merge in the borrower's local economy, even though Glennon and Nigro (2005) found that economic conditions matter. The post-2010 FOIA data, which include COVID-era cohorts, have also received less attention than the pre-2014 benchmark file.

**How this study addresses it.** I use the current SBA FOIA 7(a) loan-level files (FY2010 onward) with (a) a fixed 36-month outcome window that every included loan has fully passed through, (b) strict out-of-time validation (train on earlier fiscal years, test on later ones), and (c) county unemployment from the Bureau of Labor Statistics, merged to each loan's project county. The contribution is practical: a risk model whose reported accuracy is the accuracy a lender would actually get on future applicants.

## 4. Research Question(s) & Hypotheses

**Primary research question:** Can we predict, using only information available at approval, whether an SBA 7(a) loan will charge off within 36 months of approval, and which factors drive that risk?

**Hypothesis 1 (H1 – predictive value).** A model using approval-time borrower, loan, and local-economic features will predict 36-month charge-off on a later, held-out set of cohorts better than a naive benchmark that assigns each loan the historical charge-off rate of its industry sector and loan-size band.
- **H0:** The approval-time model's out-of-time performance is no better than the benchmark's.
- **Falsification condition:** H1 is rejected if the 95% bootstrap confidence interval for the difference in out-of-time PR-AUC (model minus benchmark) includes or falls below zero.

**Hypothesis 2 (H2 – business age).** Loans to new businesses (startups or firms in operation less than two years at approval) have higher odds of 36-month charge-off than loans to established businesses, holding loan size, term, guarantee share, processing method, industry, state, and approval year constant.
- **H0:** The adjusted odds ratio for new vs. established businesses equals 1.
- **Falsification condition:** H2 is rejected if the estimated odds ratio is ≤ 1 or its 95% confidence interval includes 1.

**Hypothesis 3 (H3 – local labor market).** A higher county unemployment rate in the year before approval is associated with higher odds of 36-month charge-off, comparing loans within the same state and approval year.
- **H0:** The coefficient on lagged county unemployment is zero.
- **Falsification condition:** H3 is rejected if the coefficient is ≤ 0 or its 95% confidence interval (clustered by county) includes 0.

**Operationalization of key constructs**

| Construct | Measure |
|---|---|
| 36-month charge-off (DV) | 1 if loan status is *charged off* and the charge-off date falls within 36 months of the approval date; 0 otherwise (including loans paid in full before 36 months) |
| New business | 1 if the business-age field indicates a startup, a new business, or fewer than 2 years in operation; 0 if 2+ years. The field's category labels changed over time, so every label is mapped by hand to this definition before filtering |
| Local unemployment | BLS LAUS annual average unemployment rate for the loan's project county, in the calendar year before approval (lagged to rule out post-approval information) |
| Loan structure | Log gross approval amount; SBA-guaranteed share (guaranteed ÷ gross); term in months; initial interest rate; fixed vs. variable rate; revolving line indicator; processing method (`ProcessingMethod`, e.g., SBA Express vs. Preferred Lenders vs. 7(a) General) |
| Borrower / sector | 2-digit NAICS sector; business type (corporation, individual, partnership); franchise indicator; collateral indicator (where populated); jobs supported |
| Predictive performance | Out-of-time PR-AUC (primary, because charge-offs are the minority class), ROC-AUC, Brier score, and calibration slope |

## 5. Data

### Sources

| Source | Variables / fields | Frequency | Coverage | Accessibility |
|---|---|---|---|---|
| SBA 7(a) FOIA loan-level files: *FY2010–FY2019* and *FY2020–Present* (data.sba.gov) | Approval date & fiscal year, gross and SBA-guaranteed amounts, term, initial rate, fixed/variable, NAICS code, business type, business age, franchise code, project county & state, processing method, revolver status, jobs supported, loan status, paid-in-full date, charge-off date and amount | Refreshed quarterly (current file as of 06/30/2026) | Every 7(a) loan approved, national | Public domain; CSV download (~243 MB and ~173 MB); no authentication |
| SBA 7(a)/504 FOIA Data Dictionary (XLSX) | Field definitions and status codes | Updated with the files | All fields | Public domain; XLSX download |
| BLS Local Area Unemployment Statistics (LAUS) | County labor force, employment, unemployment rate | Annual averages (monthly available) | All U.S. counties, 2009–present | Public; annual county files or BLS Public Data API (free key) |
| Census county FIPS reference file | County name ↔ 5-digit FIPS code | Static | All counties | Public; download from Census |

**Field names (verified against the SBA data dictionary and the downloaded files).** Both 7(a) files share the same 39 columns. The ones used here are `ApprovalDate`, `ApprovalFY`, `GrossApproval`, `SBAGuaranteedApproval`, `TermInMonths`, `InitialInterestRate`, `FixedorVariableInterestInd`, `ProcessingMethod`, `NaicsCode`, `FranchiseCode`, `ProjectCounty`, `ProjectState`, `BusinessType`, `BusinessAge`, `RevolverStatus`, `JobsSupported`, `CollateralInd`, `LoanStatus`, `PaidInFullDate`, `ChargeOffDate`, and `GrossChargeOffAmount`. `LoanStatus` codes are `PIF` (paid in full), `CHGOFF` (charged off), `CANCLD` (cancelled), `COMMIT` (undisbursed), and `EXEMPT` (disbursed and still active; status withheld under FOIA Exemption 4). The LAUS county file provides state and county FIPS codes, county name with state abbreviation, labor force, employed, unemployed, and unemployment rate.

### Sample construction

- **Population and unit:** Every 7(a) loan approved in FY2010–FY2022; one row per loan.
- **Time window:** FY2022 approvals end September 30, 2022, so every loan in the sample has at least 36 months of observable history by the June 30, 2026 data date. This means the binary outcome is fully observed for every loan, with no right-censoring.
- **Exclusions:** Cancelled (`CANCLD`) and undisbursed (`COMMIT`) loans; loans with missing approval date, amount, or project state; U.S. territories if county unemployment is unavailable.
- **Bias safeguards:** Cohorts are defined by *approval* year, not outcome, which avoids survivorship bias. Loans paid in full before 36 months stay in the sample as non-defaults (the competing-risk outcome), not dropped.

### Variables

- **Dependent:** 36-month charge-off (binary).
- **Independent (hypothesis) variables:** New-business indicator (H2); lagged county unemployment rate (H3); the full approval-time feature set (H1).
- **Controls:** Log loan amount, guarantee share, term, rate, fixed/variable, revolver, processing method, business type, franchise indicator, NAICS sector fixed effects, state × approval-year fixed effects.
- **Excluded for leakage:** Loan status, paid-in-full date, charge-off date/amount (outcome information), and current lender name. The FOIA lender field shows the institution currently holding the loan, not necessarily the one that originated it, so it can reflect post-approval secondary-market activity.

### Data quality & sufficiency

- **Size:** The program approves tens of thousands of loans each year, so the sample will be several hundred thousand loans, enough for held-out testing and for precise estimates of H2/H3 coefficients even with a low charge-off rate.
- **Known issues and planned cleaning:** (1) Business-age category labels change across files, so I will build an explicit mapping table and check that no loans are silently dropped. (2) The project county is recorded as a name, so it will be matched to FIPS codes with state plus a normalized county name, and I will report the match rate (target ≥ 98%). (3) The collateral indicator and first-disbursement date are not always populated, so missingness will be profiled by year and handled with explicit "missing" categories rather than row deletion. (4) The two FOIA files will be stacked after checking that their schemas match.
- **COVID relief:** CARES Act Section 1112 had the SBA temporarily pay principal and interest on existing 7(a) loans in 2020–2021. This mechanically suppresses early charge-offs for loans whose 36-month window overlaps that period, so it is handled explicitly in Sections 6 and 9.

### Data source samples

A 5-row sample of each raw source (selected columns, borrower names removed) is saved in `docs/data_samples.md`, generated by `scripts/make_data_samples.py`, to document the exact field names and formats as downloaded.

## 6. Methods & Identification Strategy

**Estimators / models**

1. **Benchmark:** Historical charge-off rate by NAICS sector × loan-size band, computed on the training cohorts only.
2. **Logistic regression** (statsmodels): interpretable, gives odds ratios for H2 and H3, and serves as the transparent model a credit committee could audit.
3. **Gradient-boosted trees** (scikit-learn `HistGradientBoostingClassifier`; XGBoost if time allows): captures nonlinearities and interactions (e.g., term × sector). It is the strongest candidate for H1.
4. **Secondary (robustness):** A Fine–Gray competing-risks model for time to charge-off, with prepayment as the competing event. This checks that conclusions do not depend on the fixed 36-month window.

**Identification strategy.** This is mainly a *predictive* study (H1), and prediction needs honest out-of-sample evaluation rather than causal identification. For the association tests (H2, H3), I avoid overstating causality in two ways:
- **H3:** State × approval-year fixed effects absorb statewide and national shocks (including the COVID recession and federal relief) and state-level differences in lending practice. The remaining variation compares loans approved in the *same state and year* in counties with different unemployment. The key assumption is that, within a state-year, county unemployment is not correlated with unobserved borrower quality in ways the controls do not capture. Standard errors are clustered by county.
- **H2:** Business age is set before the loan is made, so reverse causality is not a concern. The remaining threat is selection (lenders may impose stricter terms on startups). Controlling for loan structure means the estimate is interpreted as risk *beyond what lenders already priced in*, which is the quantity a credit officer needs.

**Validation & robustness**

- **Out-of-time split:** Train on FY2010–FY2016, tune hyperparameters on FY2017–FY2018, test once on FY2019–FY2022. The test set is not touched until the final models are frozen.
- **Class imbalance:** Class weighting, evaluated with PR-AUC; no resampling of the test set.
- **Calibration:** Reliability curves and Brier score. Isotonic recalibration is fitted on the validation years if needed.
- **Uncertainty:** 1,000-replicate bootstrap confidence intervals on test-set metrics and on model-minus-benchmark differences.
- **Alternative specifications:** 24- and 48-month outcome windows; excluding FY2019–FY2020 approvals (the cohorts most affected by COVID payment relief); unemployment measured in the approval year instead of the year before.
- **Placebo test:** Unemployment in the county's year *t+5*, which cannot affect a loan approved in year *t*, should not predict charge-off once state-year fixed effects are included. A significant placebo would signal spurious correlation.
- **Interpretability:** SHAP values (Lundberg & Lee, 2017) for the boosted model, checked for agreement with the logistic odds ratios.

## 7. Analysis Plan & Expected Outcomes

**Pre-specified analyses**

1. Descriptives: 36-month charge-off rate by fiscal year, sector, business age, processing method, and state (with Louisiana highlighted).
2. Logistic regression of 36-month charge-off on the new-business indicator, lagged county unemployment, and all controls, with state × year and sector fixed effects (tests H2, H3).
3. Benchmark, logistic, and gradient-boosting models trained on FY2010–2016, tuned on FY2017–2018, and evaluated on FY2019–2022 (tests H1).
4. Robustness checks and the placebo test listed in Section 6.

**What supports vs. refutes each hypothesis**

| Hypothesis | Supported if… | Refuted if… |
|---|---|---|
| H1 | Best model's test PR-AUC exceeds the benchmark's, and the 95% bootstrap CI of the difference is entirely above 0 | CI includes 0 or the benchmark wins, meaning sector and size already capture the predictable risk |
| H2 | Adjusted OR > 1 with 95% CI excluding 1, stable across outcome windows | OR ≤ 1 or CI includes 1 |
| H3 | Unemployment coefficient > 0 with clustered 95% CI excluding 0, and the placebo is null | Coefficient ≤ 0, CI includes 0, or the placebo is equally "significant" |

**Reporting commitment.** Every pre-specified result will be reported whether it supports, refutes, or fails to resolve the hypotheses. A null result for H1 is itself useful to lenders: it would mean a simple sector-and-size rule is as good as a complex model on future loans.

## 8. Contribution & Significance

**Contribution to practice (primary).**
- **Decision informed:** Whether and how to approve and structure a 7(a) loan (amount, term, collateral, processing method), and where SBDC advisors should focus pre-application coaching.
- **Business value:** At a chosen review threshold, the model identifies a flagged group of applicants for closer underwriting. Value is estimated as *expected losses avoided = (charge-offs caught in the flagged group × average charge-off amount) − (cost of extra review + interest income forgone on good loans declined)*, using actual charge-off amounts from the FOIA data in the test years. The dashboard lets a user move the threshold and watch this trade-off change.
- **Regional relevance:** A Louisiana subgroup analysis gives local lenders and the Louisiana SBDC network a state-specific view.

**Contribution to knowledge (secondary).** An out-of-time, censoring-aware benchmark on post-2010 SBA data, including COVID-era cohorts, that later SBA default studies can compare against. The pipeline is public and reproducible.

**Success metrics (independent of the direction of results)**

- Complete, reproducible pipeline: raw download → SQLite → models → dashboard, rerunnable from a clean clone.
- County match rate ≥ 98%; all business-age labels mapped with zero silent drops.
- All three hypotheses tested exactly as pre-specified, with CIs reported.
- Test-set calibration slope between 0.8 and 1.2 for the deployed model, so its probabilities can be taken at face value.
- Deployed dashboard that loads in under 10 seconds and shows risk by segment, the loss-avoidance trade-off, and the Louisiana view.

## 9. Threats to Validity, Limitations & Fallback

**Internal validity**
- *Selection:* The data contain only *approved* loans, so the model predicts risk among approved applicants, not among everyone who applied. This matches how a lender would use it (screening applicants who resemble past approvals) but limits any causal claim.
- *Confounding (H3):* County unemployment may proxy for unobserved borrower quality. This is addressed with state-year fixed effects, the placebo test, and the H3 claim is stated as associational.
- *Policy shocks:* COVID-era payment relief suppressed charge-offs for FY2019–2020 cohorts. This is handled with year fixed effects and the exclusion robustness check, and test-set performance is reported with and without those years.

**External and construct validity**
- The results describe SBA 7(a) loans, not conventional small-business loans.
- The 36-month charge-off measures *early* loss, not lifetime loss. The Fine–Gray model provides the longer-horizon check.
- The FOIA file lacks borrower credit scores and financial statements, which lenders do see. So the model estimates the risk that remains *after* information a lender already has, and should be presented as a complement to underwriting, not a replacement.

**Limitations & fallback**
- If county matching fails for a large share of loans, H3 drops to state-level unemployment (with year fixed effects only), and the change is documented.
- If the full national dataset is too large for free dashboard hosting, the deployed app uses aggregated tables plus a stratified sample, with full-data results precomputed.
- If the boosted model does not beat logistic regression, the simpler model is deployed, a legitimate and more auditable outcome.
- A well-documented null result, or a finding that these public fields cannot predict early charge-off better than sector and size, is a valid pilot result.

## 10. Technical Implementation & Reproducibility

- **Database:** SQLite (`data/sba.db`), normalized:
  - `loans` (loan_id, approval_date, fiscal_year, amounts, term, rate, business_type, business_age_raw, new_business, naics_code, county_fips, processing_method, outcome fields)
  - `county_unemployment` (county_fips, year, unemployment_rate, labor_force)
  - `naics_sectors` (naics_code, sector_2digit, sector_name)
  - `business_age_map` (raw_label, new_business)
  - A `model_frame` view joins these for analysis. Raw CSVs are excluded from Git via `.gitignore` and re-downloaded by script.
- **Analysis:** Python 3.11+; pandas, numpy, statsmodels, scikit-learn, (optional) xgboost, shap, lifelines (Fine–Gray / survival), matplotlib/plotly. Numbered scripts in `scripts/` (`01_download.py` → `05_evaluate.py`), fixed random seeds, `requirements.txt`, and results written to `outputs/`.
- **Dashboard:** Streamlit app (`app/app.py`) with pages for portfolio overview (charge-off by year, sector, state), a risk explorer (filter by segment and see predicted vs. actual), the threshold / loss-avoidance trade-off, and the Louisiana view.
- **Deployment:** Streamlit Community Cloud deployed from the GitHub repo. The app reads a compact SQLite/Parquet extract (< 100 MB, under GitHub's file limit) produced by the pipeline.

### Timeline & milestones

| Week(s) | Milestone / activities | Course anchor |
|---|---|---|
| 1–2 | Download FOIA and LAUS data; build business-age map and county FIPS match; design and load SQLite schema; generate `docs/data_samples.md` | Assignment 4 (database) |
| 3–4 | Construct outcome and features; profile missingness; descriptive EDA by year, sector, state | Early analysis assignment |
| 5–8 | Fit benchmark, logistic, and boosted models; out-of-time validation; bootstrap CIs; robustness and placebo tests; SHAP | Analysis assignment(s) |
| 9–12 | Build Streamlit dashboard; deploy to Streamlit Cloud; draft results and discussion | Dashboard & deployment assignments |
| 13 | Final review, reproducibility check from a clean clone, polish write-up, submit | Final portfolio |

## 11. References

Berger, A. N., & Udell, G. F. (1998). The economics of small business finance: The roles of private equity and debt markets in the financial growth cycle. *Journal of Banking & Finance, 22*(6–8), 613–673.

Brown, J. D., & Earle, J. S. (2017). Finance and growth at the firm level: Evidence from SBA loans. *The Journal of Finance, 72*(3), 1039–1080.

Fine, J. P., & Gray, R. J. (1999). A proportional hazards model for the subdistribution of a competing risk. *Journal of the American Statistical Association, 94*(446), 496–509.

Glennon, D., & Nigro, P. (2005). Measuring the default risk of small business loans: A survival analysis approach. *Journal of Money, Credit and Banking, 37*(5), 923–947.

Lessmann, S., Baesens, B., Seow, H.-V., & Thomas, L. C. (2015). Benchmarking state-of-the-art classification algorithms for credit scoring: An update of research. *European Journal of Operational Research, 247*(1), 124–136.

Li, M., Mickel, A., & Taylor, S. (2018). "Should this loan be approved or denied?": A large dataset with class assignment guidelines. *Journal of Statistics Education, 26*(1), 55–66. https://doi.org/10.1080/10691898.2018.1434342

Lundberg, S. M., & Lee, S.-I. (2017). A unified approach to interpreting model predictions. In *Advances in Neural Information Processing Systems 30* (pp. 4765–4774).

Stiglitz, J. E., & Weiss, A. (1981). Credit rationing in markets with imperfect information. *The American Economic Review, 71*(3), 393–410.

U.S. Bureau of Labor Statistics. (2026). *Local Area Unemployment Statistics: County data* [Data set]. https://www.bls.gov/lau/

U.S. Small Business Administration. (2026). *7(a) & 504 FOIA* [Data set; files as of 06/30/2026]. https://data.sba.gov/dataset/7a-504-foia

---

### Submission checklist

- [x] Proposal committed under `docs/` (Markdown)
- [ ] Meaningful Git commit message, pushed before Sun Sep 27, 11:59 PM
- [ ] `docs/AI_USE.md` updated for this assignment
- [ ] `docs/data_samples.md` generated and committed
- [x] Hypotheses falsifiable; every construct operationalized
