Purpose:
Analyze associations between trust in doctors and perceived value of
multi-cancer early detection (MCED) blood tests using HINTS 7 public-use data.
Evaluate mediation by awareness of MCED, effect modification by age, and
robustness via model diagnostics. Produce descriptive tables, figures, and
survey-weighted logistic regression results (unadjusted, awareness-adjusted,
and fully adjusted models).

Author:
Ewnet Azerefegne

Input files / required resources (expected in working directory):

* hints7_public.rda        : HINTS 7 public-use dataset object (loaded as `public`)
* MCED_dag.png             : conceptual DAG image included in the report

Primary packages / dependencies (script expects these to be available):
pacman (for loading), officer, flextable, dplyr, readr, forcats,
gridExtra, knitr, kableExtra, mosaic, table1, broom, survey, car,
ResourceSelection, DiagrammeR, ggplot2, tibble

Output:

* HTML report (bookdown::html_document2) including:

  * Table 1: sample characteristics overall and by trust in doctors
  * Analytic sample flow diagram
  * Descriptive bar charts for perceived MCED value and awareness by trust
  * DAG image
  * Regression tables for:

    * Model 1: survey-weighted unadjusted (trust only)
    * Model 2: survey-weighted (trust + awareness)
    * Model 3: survey-weighted fully adjusted (trust + awareness + covariates)
  * Interaction test results (trust × age)
  * Diagnostics from an unweighted glm: VIFs, Hosmer–Lemeshow, Cook's distance
  * Summary regression table stratified by age groups

Data preparation and derived variables (high-level descriptions):

* Selected variables: trust in doctor, perceived MCED value, MCED awareness,
  sociodemographics, health insurance, health literacy (confidence with forms),
  general trust in health care system, and sampling design variables (person-level weight,
  stratum, cluster).

* Key derived binary outcome and exposures:

  * val_high (outcome): 1 if respondent answered "Very valuable" to perceived
    MCED value; 0 if responded "Somewhat valuable" / "A little" / "Not at all".
  * trust_high (primary exposure): 1 if respondent reported "A lot" of trust in doctor;
    0 for "Some", "A little", or "Not at all".
  * aware (mediator): 1 if respondent answered "Yes" to hearing about MCED tests;
    0 for "No", "Don't know", or "Refused".

* Covariate recoding (summary):

  * Sex: factor (Female, Male) from BirthSex.
  * Age: numeric conversion where possible, then categorized into 18-49, 50-64, 65+.
  * Race/ethnicity: 5-level collapsed to White, Black, Hispanic, Asian, Other.
  * Education: three-level (≤HS, Some college, College+).
  * Household income: three-level (<$35k, $35-74.9k, ≥$75k).
  * Insurance: Insured vs Uninsured.
  * Health literacy proxy: confidence with medical forms categorized as High vs Not high.
  * Trust in health system: "A lot" vs other responses as High vs Not high.

Missing data and analytic sample:

* Observations with NA on any of the key variables (outcome, exposure, mediator,
  all covariates, and sampling variables PERSON_FINWT0, VAR_STRATUM, VAR_CLUSTER)
  are excluded from the analytic dataset.
* Reported sample size change in the script: original N = 7,278 → analytic N = 5,880
  after exclusions (document actual counts produced during execution).

Survey design and weighting:

* A complex survey design object is created using:

  * ids      = cluster variable (VAR_CLUSTER)
  * strata   = stratification variable (VAR_STRATUM)
  * weights  = person-level final weight (PERSON_FINWT0)
  * nest = TRUE
* Survey-weighted generalized linear models use family = quasibinomial() to estimate ORs.

Modeling approach:

* make_or_table: helper to extract exponentiated coefficients, CIs, and p-values.
* Model 1: val_high ~ trust_high (survey-weighted) — unadjusted OR.
* Model 2: val_high ~ trust_high + aware (survey-weighted) — evaluates mediator impact.
* Model 3: val_high ~ trust_high + aware + covariates (survey-weighted) — fully adjusted.
* Interaction analysis: fit survey-weighted model with trust_high × age_cat interaction;
  use regTermTest (Wald test) to assess interaction significance.

Diagnostics and sensitivity:

* For diagnostics, an unweighted glm (binomial) mirroring the covariates in Model 3 is fit.
* Multicollinearity: assessed with car::vif; consider VIF threshold (e.g., < 5 acceptable).
* Calibration: Hosmer–Lemeshow test via ResourceSelection::hoslem.test (on unweighted model).
* Influence: Cook's distance plots (identify and consider observations with high Cook's D).

Table and figure generation:

* Descriptive tables: table1 for weighted characteristics and subgroup tables.
* Regression results: presented as OR (95% CI) with p-values; formatted via knitr/kable/kableExtra.
* Figures: ggplot2 bar charts (proportions) for perceived value and awareness by trust category.
* Flow diagram: DiagrammeR to illustrate sample inclusion/exclusion.

Notes / assumptions / caveats:

* Binary coding choices (e.g., treating "Somewhat valuable" as not "Very valuable")
  affect interpretation; ensure these choices match research question and sensitivity analyses.
* Awareness coded with "Don't know" and "Refused" grouped with "No" — consider alternate codings.
* Survey-weighted inference depends on correct use of weights, clusters, and strata; verify
  that PERSON_FINWT0 is the appropriate final person-level weight for the target estimates.
* Quasibinomial family is used with svyglm to address potential overdispersion in binary model;
  check whether results differ with binomial family or alternative variance estimators.
* The Hosmer–Lemeshow test and Cook's distance are applied to the unweighted model for diagnostics;
  weighted diagnostic alternatives are more complex—interpret diagnostics accordingly.
* Interaction test p-values and subgroup estimates should be interpreted with attention to
  multiple comparisons and sample size within strata.

Suggested extensions and sensitivity checks:

* Re-run models using alternative outcome codings (e.g., ordinal multinomial models if treating
  perceived value as ordered categories) or alternative exposure definitions.
* Consider mediation analysis methods appropriate for complex survey data if formally quantifying
  mediation by awareness.
* Report weighted sample sizes and design-based standard errors for all estimates.
* Check robustness to exclusion/inclusion of respondents with "Don't know" / "Refused" responses.

Execution notes:

* Set working directory to the project/report folder containing the Rmd and required files.
* Ensure pacman or listed packages are installed prior to knitting.
* Output is intended for an HTML bookdown document (table of contents enabled).

End of documentation.


