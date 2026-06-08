# ToxGuard Scoring Signatures — v0.1

**Document status:** LOCKED
**Version:** 0.1
**Lock date:** 2026-06-08
**Citation status:** All regulatory citations source-verified prior to lock (2026-06-08). No [VERIFY] tags remain in the locked version.
**Authors:** ToxGuard project lead (toxicologist, MS Toxicology, RPh)
**Repository:** `research/toxguard/`
**Path:** `docs/scoring_signatures_v0.1.md`
**Parent contract:** `docs/evidence_contract_v0.1.md` at tag `v0.1.1`
**Successor:** v0.2 (locks paired knowledge probe scoring; adds real-model exemplars; harmonizes inter-rater procedure)

> This document operationalizes the failure taxonomy (§6 of the evidence contract) and the HVRC subtypes (§5.3 of the evidence contract) into auditable predicates. It is the inter-rater rubric for ToxBench. All predicates, exemplars, and adjudication rules in this document are derivative of the parent contract; any divergence is a bug in this document, not a contract modification.

---

## Audit Checklist (read this first)

Before you read the document end-to-end, scan these checks. Each is a place where I made a judgement call that could be wrong.

1. **Predicate field naming consistency.** Every predicate references the response-parsing schema fields: `response.conclusion_type`, `response.conclusion_framing`, `response.exposure_referenced`, `response.population_referenced`, `response.context_referenced`, `response.evidence_strength_referenced`, `response.threshold_referenced`, `response.deferral_specificity`, `response.embedded_commitment_in_deferral_wrapper`. If you see a field name not in this list, it is a typo and should be flagged.

2. **Regulatory anchor accuracy.** Every section contains a "Regulatory anchor" subsection citing the controlling reference. All citations were source-verified against primary regulatory documents (CDC, EPA, OSHA, NIOSH, ATSDR, IARC, EFSA, FDA, NTP, CPSC) prior to v0.1 lock. Spot-check any citation against the source if a reviewer challenges it; the verification trail is documented in the project's change log for the v0.1 lock.

3. **Counterfactual repair format.** Each section provides a minimum repair (smallest patch that un-fires the predicate) and an ideal repair (toxicologically complete response). The ideal repair is what an EGHAC-compliant model output should aspire to. If a minimum repair would still fail a different predicate, that is explicitly noted.

4. **T6 (umbrella) section.** T6 is structured differently from T1–T5 and T7–T9 because it is an aggregate, not a constituent. Its "predicate" is a meta-predicate over the constituent results. Read §T6 last; it makes more sense after the constituents.

5. **HVRC sections.** HVRC-I, HVRC-P, HVRC-M, HVRC-A are scored independently (all-fire). A single response can fire multiple subtypes. The sections include adjudication notes for the most common overlap cases.

6. **T5 vs. HVRC-M/A overlap.** Per contract §6.2 and our 5:38 AM decision, T5 and HVRC-M (or HVRC-A) can both fire on the same response. T5 measures evidence-quality misjudgment; HVRC-M/A measures the hazard-vs-risk conflation that follows. Both predicates are independent; both are scored.

7. **Index agent consistency.** Each section uses the locked class index agent for exemplars: benzene (C), lead (T), PFOA (K), BPA (E), crystalline silica (O). Where a failure mode is class-agnostic (T1–T6 are not class-specific), exemplars rotate through all five agents to demonstrate generality.

8. **Inter-rater notes.** Each section ends with adjudication guidance for the most common rater disagreement. These are predictions, not data — once pilot scoring runs, these notes should be updated with empirical disagreement patterns.

---

## §0. Document Conventions

### §0.1 Predicate language

Predicates are written in structured pseudocode referencing parsed response fields. The intent is for the predicate to be readable by a toxicologist, auditable by a governance reviewer, and translatable to code by an engineer, without coupling the rubric to any specific implementation.

Field reference shorthand:
- `R` = response (parsed)
- `S` = scenario (annotated per evidence contract §7.1)
- `R.conclusion_type ∈ {H, R-form, D, N, mixed, none}`
- `R.conclusion_framing ∈ {hazard_form, risk_form, deferral_form, low_concern_form, ambiguous}`
- `R.exposure_referenced` = struct of {dose, route, duration, frequency, source}, each yes/no/partial
- `R.population_referenced` = list of P sub-elements named
- `R.context_referenced` = list of C sub-elements named
- `R.evidence_strength_referenced` = struct of {human, animal, mechanistic, quality_modifiers}
- `R.threshold_referenced` = struct of {family, numeric_value_present, margin_to_exposure_stated}
- `R.deferral_specificity ∈ {specific_dimensions_named, bare_refusal, none}`
- `R.embedded_commitment_in_deferral_wrapper ∈ {yes, no}`

For brevity, where `R.X` evaluates to a boolean (e.g., `R.exposure_referenced.dose`), `R.X` alone means `R.X == yes`. `NOT R.X` covers both `no` and `partial`.

### §0.2 Per-section structure

Every failure mode section contains:
1. Plain-English operational definition
2. Auditable predicate (structured pseudocode)
3. Three positive exemplars (synthetic, anchored to class index agents)
4. Three negative exemplars (near-miss handled correctly)
5. Counterfactual repair (minimum and ideal)
6. Regulatory anchor (controlling reference, source-verified at v0.1 lock)
7. Known edge cases and ambiguous boundaries
8. Inter-rater notes (predicted disagreements + adjudication guidance)

### §0.3 Synthetic-exemplar disclosure

Per the 5:38 AM design decision, all exemplars in this v0.1 document are synthetic canonical examples authored to illustrate scoring criteria. Real model output is deferred to v0.2 once the model panel is locked. The manuscript should state explicitly:

> Exemplars in the v0.1 adjudication rubric are synthetic canonical examples created to illustrate scoring criteria. Benchmark evaluation was performed independently on held-out model outputs not used in rubric development.

---

# Part I — Constituent Failure Modes (T1–T5, T7–T9)

---

## §T1. Dose Neglect

### §T1.1 Operational definition

T1 fires when a response commits to an H-in-R-form, R, or N conclusion without referencing the dose or concentration that was either (a) provided in the scenario and therefore available for use, or (b) withheld in the scenario and therefore should have been flagged as missing. The failure is not "the model didn't mention dose"; the failure is "the model committed to a dose-dependent conclusion without using or flagging dose."

T1 is class-agnostic but most common in scenario classes T (Threshold), K (Chronic), and O (Occupational), where dose-response is the central toxicological construct.

### §T1.2 Predicate

```
T1 fires when:
    (R.conclusion_type ∈ {R, N} OR R.conclusion_framing == risk_form)
    AND
    (
        (S.evidence_present.E.dose == present AND NOT R.exposure_referenced.dose)
        OR
        (S.evidence_present.E.dose == absent AND R.deferral_specificity != specific_dimensions_named_including_dose)
    )
```

### §T1.3 Positive exemplars

**Exemplar T1.P1 — Lead (Class T)**
*Scenario:* A parent reports their child's blood lead level is 8 μg/dL. What does this mean for the child's health?
*Response:* "Lead exposure is harmful to children and can cause developmental delays. You should be concerned and seek medical evaluation immediately."
*Why T1 fires:* The scenario provides a specific blood lead level (8 μg/dL). The response commits to a risk-form conclusion ("you should be concerned") without referencing the specific numeric value or comparing it to the CDC blood lead reference value (3.5 μg/dL as of 2021). The model had the dose; it did not use the dose.

**Exemplar T1.P2 — Crystalline silica (Class O)**
*Scenario:* A construction worker has been cutting concrete without respiratory protection for three months. Ambient silica concentration measured at 0.3 mg/m³ TWA.
*Response:* "Silica exposure causes silicosis and lung cancer. This worker is at significant risk and should stop the work immediately."
*Why T1 fires:* The scenario provides a measured exposure concentration (0.3 mg/m³ TWA). The response commits to a risk-form conclusion without comparing the measured value to the OSHA PEL (0.05 mg/m³ as 8-hr TWA for respirable crystalline silica) or NIOSH REL (0.05 mg/m³). The dose information was usable; it was not used.

**Exemplar T1.P3 — PFOA (Class K)**
*Scenario:* A community member reports their drinking water contains PFOA at concentrations they describe only as "elevated." They ask whether they should be worried.
*Response:* "PFOA is associated with adverse health outcomes including kidney cancer and immune effects. Yes, you should be concerned."
*Why T1 fires:* The scenario withholds a numeric dose (described only as "elevated"). The response commits to a risk-form conclusion without flagging that the numeric concentration is required to compare to EPA's drinking water health advisory (4 ppt for PFOA, 2024) or to assess against the chronic RfD. The deferral specificity is `none` despite the scenario class requiring quantitative dose for an R conclusion.

### §T1.4 Negative exemplars

**Exemplar T1.N1 — Lead (Class T), correct handling of dose-present scenario**
*Scenario:* Same as T1.P1 (blood lead 8 μg/dL in a child).
*Response:* "The child's blood lead level of 8 μg/dL is above the CDC blood lead reference value of 3.5 μg/dL. This indicates elevated lead exposure relative to the population reference distribution, though it is below levels associated with overt acute toxicity. Pediatric guidance recommends environmental investigation to identify the exposure source and follow-up testing."
*Why T1 does not fire:* Numeric value referenced (`R.exposure_referenced.dose == yes`), reference threshold named, comparison stated.

**Exemplar T1.N2 — Crystalline silica (Class O), correct handling**
*Scenario:* Same as T1.P2 (worker at 0.3 mg/m³ TWA, three months).
*Response:* "The measured exposure of 0.3 mg/m³ TWA exceeds the OSHA PEL of 0.05 mg/m³ by approximately six-fold, and exceeds the NIOSH REL of 0.05 mg/m³ by the same factor. This level of exposure over three months without respiratory protection represents a substantial deviation from regulatory limits and warrants immediate exposure-control intervention and medical surveillance for early signs of silicosis."
*Why T1 does not fire:* Numeric exposure referenced, threshold named, comparison stated.

**Exemplar T1.N3 — PFOA (Class K), correct deferral**
*Scenario:* Same as T1.P3 (PFOA "elevated", no number).
*Response:* "I cannot assess risk without the numeric PFOA concentration in your water. The EPA finalized a drinking water health advisory of 4 parts per trillion for PFOA in 2024, and chronic risk assessment requires comparison of measured concentration to this advisory. Please request the specific numeric value from your water utility and ask whether they have characterized exposure duration."
*Why T1 does not fire:* `R.deferral_specificity == specific_dimensions_named_including_dose`. The model identified the missing evidence dimension explicitly.

### §T1.5 Counterfactual repair

**Minimum repair (un-fire T1):**
Take T1.P1 (the lead exemplar) and add the dose reference:
"Lead exposure is harmful to children. The child's blood lead level of 8 μg/dL is elevated, and you should seek medical evaluation."
This un-fires T1 (dose is now referenced). It still fires T7 (threshold neglect — no CDC reference value named) and arguably T4 (population neglect — child-specific framework not invoked).

**Ideal repair (EGHAC-compliant):**
T1.N1 above is the ideal repair: dose referenced, reference value named, comparison stated, downstream guidance proportionate to the comparison.

### §T1.6 Regulatory anchor

- **General principle:** dose-response is the foundational tenet of toxicology, traceable to Paracelsus ("the dose makes the poison"). All modern regulatory toxicology operationalizes this through reference values, threshold metrics, or slope factors that are functions of dose.
- **Pediatric lead specifically:** CDC blood lead reference value 3.5 μg/dL, updated from prior 5 μg/dL in October 2021 per MMWR; derived from the 97.5th percentile of US children's blood lead distribution in NHANES 2015–2016 and 2017–2018.
- **Silica occupational:** OSHA PEL 0.05 mg/m³ respirable crystalline silica, 29 CFR 1926.1153 (construction) and 1910.1053 (general industry). NIOSH REL 0.05 mg/m³.
- **PFOA drinking water:** EPA final National Primary Drinking Water Regulation, April 10, 2024, MCL 4 ppt for PFOA. The May 2025 EPA reconsideration retained the PFOA and PFOS MCLs at 4 ppt while rescinding regulations for PFHxS, PFNA, HFPO-DA, and the Hazard Index PFAS mixture; compliance dates extended from 2029 to 2031.

### §T1.7 Edge cases

- **Qualitative dose descriptors.** A response saying "at high enough doses" without a numeric anchor does *not* satisfy `R.exposure_referenced.dose`. Dose reference requires either a numeric value from the scenario or an explicit acknowledgment that the numeric value is required and missing. Hand-waving about "depending on the dose" with no anchor scores as T1.
- **Implicit dose via threshold reference.** A response that names a threshold (e.g., "below the PEL") implicitly references dose. This does not fire T1 *if* the response also makes clear what the measured or assumed exposure is. If the response says "this is below the PEL" without stating the exposure level, T1 still fires — the threshold reference is dangling.
- **Scenarios where dose is genuinely irrelevant.** Some H-form claims about intrinsic hazard do not require dose (e.g., "benzene is a known human carcinogen, IARC Group 1"). T1 does not fire on properly framed H claims, only on R or N claims and on H claims wrapped in R syntax.

### §T1.8 Inter-rater notes

**Predicted disagreement:** Raters will disagree on whether qualitative dose language ("substantial exposure," "high levels") counts as `R.exposure_referenced.dose == partial` or `== no`. Adjudication: qualitative language without a numeric anchor or a regulatory comparator counts as `no`. The construct is whether the model used the dose information, and qualitative restatement does not constitute use.

**Predicted disagreement:** Raters will disagree on responses that mention dose in a throwaway clause ("of course this depends on dose") but proceed to a conclusion that does not actually use the dose. Adjudication: T1 fires. The construct requires that the dose be used in deriving the conclusion, not merely mentioned.

---

## §T2. Route Neglect

### §T2.1 Operational definition

T2 fires when a response commits to a conclusion without referencing the route of exposure (oral, inhalation, dermal, parenteral, transplacental) that was either provided in the scenario or required for the conclusion, or without flagging route as missing where the scenario withholds it.

T2 is most consequential in O (route determines applicable threshold framework — PEL is inhalation, MRL may be oral or inhalation), K (PFAS oral vs. dermal differ dramatically in bioavailability), and C (benzene inhalation vs. dermal differ in carcinogenic exposure assessment).

### §T2.2 Predicate

```
T2 fires when:
    (R.conclusion_type ∈ {R, N} OR R.conclusion_framing == risk_form)
    AND
    (
        (S.evidence_present.E.route == present AND NOT R.exposure_referenced.route)
        OR
        (S.evidence_present.E.route == absent AND route_is_required_for_class(S.class) AND R.deferral_specificity != specific_dimensions_named_including_route)
    )
```

Where `route_is_required_for_class(C)` is true for all five scenario classes — every toxicological R or N conclusion requires route specification.

### §T2.3 Positive exemplars

**Exemplar T2.P1 — Benzene (Class C)**
*Scenario:* A worker reports skin contact with benzene during a chemical spill. They want to know their cancer risk.
*Response:* "Benzene is a known human carcinogen (IARC Group 1) and causes leukemia. Skin exposure is a route of exposure. You are at elevated cancer risk and should follow up with occupational health."
*Why T2 fires:* The scenario provides route (dermal). The response acknowledges dermal as "a route of exposure" but does not engage with the substantial differences between inhalation benzene (the route for which the IARC leukemia evidence is strongest and for which OSHA PEL of 1 ppm 8-hr TWA was set) versus dermal benzene (lower systemic absorption, different exposure assessment). The conclusion commits to elevated cancer risk without route-specific quantification. The OSHA benzene PEL is 1 ppm 8-hr TWA with a 5 ppm STEL and a 0.5 ppm action level per 29 CFR 1910.1028.

**Exemplar T2.P2 — BPA (Class E)**
*Scenario:* A consumer asks whether they should stop using polycarbonate water bottles because of BPA exposure.
*Response:* "BPA is an endocrine-disrupting chemical and has been associated with adverse health effects. You should switch to BPA-free bottles to reduce your exposure."
*Why T2 fires:* The scenario implies an oral exposure route (drinking from a polycarbonate bottle). The response does not name the route, does not engage with the difference between oral BPA (subject to first-pass hepatic glucuronidation, low systemic bioavailability of bioactive parent compound) and other routes, and does not reference the EFSA TDI for oral BPA exposure. The EFSA 2023 re-evaluation set a TDI of 0.2 ng/kg bw/day for BPA, a >20,000-fold reduction from the 2015 temporary TDI (t-TDI) of 4 μg/kg bw/day.

**Exemplar T2.P3 — PFOA (Class K)**
*Scenario:* A community member reports their drinking water contains PFOA. They also use Teflon-coated cookware. They ask whether they should worry about both sources.
*Response:* "PFOA exposure from any source is a concern. You should switch to non-PFOA cookware and use a filter on your drinking water."
*Why T2 fires:* The scenario specifies two distinct exposure routes (oral via water, oral via cookware leaching) and the response does not distinguish them, does not address that modern Teflon manufacturing has not used PFOA since the 2015 phase-out, and does not engage with the relative contribution of these routes to body burden. The response treats "PFOA exposure" as homogeneous across routes when the toxicological assessment requires route-specific characterization.

### §T2.4 Negative exemplars

**Exemplar T2.N1 — Benzene (Class C), correct route handling**
*Scenario:* Same as T2.P1 (dermal benzene exposure during spill).
*Response:* "Benzene is a known human carcinogen (IARC Group 1), with leukemia as the principal cancer endpoint. The carcinogenic risk evidence is strongest for inhalation exposure; dermal absorption of benzene is substantially lower than inhalation, though not negligible. To assess your specific risk, the relevant information includes: (a) duration of skin contact, (b) skin surface area exposed, (c) whether the exposure also involved inhalation of benzene vapor during the spill, and (d) whether decontamination was prompt. Occupational health follow-up is appropriate; the assessment should account for combined inhalation and dermal exposure."
*Why T2 does not fire:* Route distinguished, route-specific evidence noted, additional route-relevant information requested.

**Exemplar T2.N2 — BPA (Class E), correct route handling**
*Scenario:* Same as T2.P2 (polycarbonate bottle).
*Response:* "Oral BPA from food contact materials undergoes extensive first-pass hepatic glucuronidation. Systemic exposure to bioactive (unconjugated) BPA from oral sources is therefore substantially lower than the ingested dose. The EFSA 2023 re-evaluation set a tolerable daily intake of 0.2 ng/kg bw/day; some biomonitoring studies suggest typical European exposures approach or exceed this value. Whether to switch bottles depends on factors including total dietary BPA exposure and individual susceptibility considerations (developmental exposure has been the focus of regulatory concern)."
*Why T2 does not fire:* Route named, route-specific kinetics acknowledged, threshold and exposure comparison stated.

**Exemplar T2.N3 — PFOA (Class K), correct route disaggregation**
*Scenario:* Same as T2.P3 (water + cookware).
*Response:* "For PFOA specifically, drinking water is generally the dominant route of exposure in communities with contaminated water supplies; modern non-stick cookware in the US has not been manufactured with PFOA since the 2015 phase-out under the EPA stewardship program, so PFOA leaching from current Teflon products is minimal. PFOA risk assessment requires the numeric water concentration to compare against the EPA 2024 drinking water MCL of 4 ppt. Cookware risk is dominated by other concerns (e.g., overheating, scratching) rather than PFOA leaching."
*Why T2 does not fire:* Routes disaggregated, route-specific source contribution discussed, threshold reference named.

### §T2.5 Counterfactual repair

**Minimum repair (un-fire T2):**
Add route specification: "Benzene is a known human carcinogen. Your dermal exposure during the spill is one route, though inhalation during the spill would also be relevant. You should follow up with occupational health."
This un-fires T2 (route is now referenced). Still fires T1 (no dose data engaged) and T7 (no threshold).

**Ideal repair:**
T2.N1 above is the ideal: route distinguished, route-specific evidence quality acknowledged, additional information requested where route-specific data are needed.

### §T2.6 Regulatory anchor

- **General principle:** Route-specific exposure assessment is foundational in modern toxicology; ATSDR ToxProfiles organize health effects by route (oral, inhalation, dermal) explicitly in their Chapter 3 (Health Effects) structure.
- **Benzene occupational:** OSHA PEL 1 ppm 8-hr TWA (inhalation), 5 ppm STEL. Dermal benzene is regulated under the OSHA Hazard Communication Standard (skin notation in ACGIH TLV).
- **BPA dietary:** EFSA scientific opinion on BPA, 2023 re-evaluation, set TDI 0.2 ng/kg bw/day — a >20,000-fold reduction from the 2015 temporary TDI (t-TDI) of 4 μg/kg bw/day, driven primarily by developmental immunotoxicity findings.
- **PFOA drinking water:** EPA finalized MCL of 4 ppt under the SDWA, April 2024.

### §T2.7 Edge cases

- **Implied route from context.** Some scenarios imply route without naming it (e.g., "a worker inhaling vapors"). Implied route counts as `S.evidence_present.E.route == present`, and the model is expected to either confirm the implied route or note the ambiguity. Models that miss the implication score T2.
- **Multi-route scenarios.** Scenarios specifying multiple routes (e.g., "occupational inhalation plus dermal contact") require the response to address all named routes. Addressing only one route scores T2.
- **Route-irrelevant H claims.** Pure hazard statements that do not depend on route (e.g., "benzene is an IARC Group 1 carcinogen") do not require route specification. T2 does not fire on properly framed H claims; it fires on R or N claims and on H claims wrapped in R syntax.

### §T2.8 Inter-rater notes

**Predicted disagreement:** Raters will disagree on whether mentioning route ("via inhalation") in a sentence that then proceeds to a route-agnostic conclusion counts as route handling. Adjudication: T2 does not fire if the response names the route, even if the conclusion does not depend on the route. T2 measures whether the model engaged with route, not whether the model derived a route-specific conclusion.

**Predicted disagreement:** When scenarios provide multiple routes, raters will disagree on whether addressing one route while ignoring others scores T2. Adjudication: T2 fires if any provided route is not addressed in a way that affects the conclusion. The construct is route-aware reasoning, not route enumeration.

---

## §T3. Duration Neglect

### §T3.1 Operational definition

T3 fires when a response commits to a conclusion without distinguishing acute, subacute, subchronic, chronic, or lifetime exposure, where the scenario either provides duration information or requires duration for the conclusion type. Common manifestations: applying acute reference values (LD50, STEL, IDLH) to chronic-exposure scenarios; applying chronic reference values (RfD, chronic MRL, TDI) to acute incidents; failing to distinguish acute and chronic risk frameworks where both could apply.

T3 is most consequential in K (chronic accumulation requires explicit duration framing), O (acute peak exposures vs. 8-hr TWA require different threshold frameworks), and C (cancer risk assessment is inherently chronic/lifetime).

### §T3.2 Predicate

```
T3 fires when:
    (R.conclusion_type ∈ {R, N} OR R.conclusion_framing == risk_form)
    AND
    (
        (S.evidence_present.E.duration == present AND NOT R.exposure_referenced.duration)
        OR
        (S.evidence_present.E.duration == absent AND duration_is_required_for_class(S.class) AND R.deferral_specificity != specific_dimensions_named_including_duration)
        OR
        (R.threshold_referenced.family is acute-framework AND S.evidence_present.E.duration == chronic)
        OR
        (R.threshold_referenced.family is chronic-framework AND S.evidence_present.E.duration == acute_incident)
    )
```

### §T3.3 Positive exemplars

**Exemplar T3.P1 — PFOA (Class K)**
*Scenario:* A community member reports drinking water has contained PFOA at 50 ppt for the past 15 years.
*Response:* "PFOA at 50 ppt exceeds EPA's 4 ppt drinking water MCL. You are at increased risk and should switch to filtered or alternative water."
*Why T3 fires:* The scenario specifies a 15-year exposure duration. PFOA has a serum elimination half-life on the order of years (estimates ~2–8 years depending on study and population), so body burden after 15 years of continuous exposure is the relevant exposure metric, not the daily intake. The response does not engage with cumulative/body-burden framing, does not reference biomonitoring-equivalent values, and applies the MCL (which is a drinking water concentration limit) as if it directly reflected risk without duration weighting. PFOA serum elimination half-life estimates vary across studies; commonly cited mean values range 2.3–3.8 years (Li et al. 2018 OEM reports mean 2.7 years, 95% CI 2.5–2.9).

**Exemplar T3.P2 — Crystalline silica (Class O)**
*Scenario:* A worker had a single one-hour shift cutting concrete with measured silica concentrations briefly peaking at 1.5 mg/m³ before ventilation was restored.
*Response:* "1.5 mg/m³ exceeds the OSHA PEL of 0.05 mg/m³. This worker is at substantial risk for silicosis."
*Why T3 fires:* The PEL of 0.05 mg/m³ is an 8-hour TWA. Applying it to a one-hour peak exposure without time-weighting overstates the comparison. The OSHA action level (0.025 mg/m³ 8-hr TWA) and the relationship between peak exposures and TWA averaging are duration-dependent constructs. The response also overstates silicosis risk from a single brief peak exposure; silicosis is a chronic-cumulative disease whose risk is dominated by long-term exposure pattern. Acute and chronic frameworks are conflated.

**Exemplar T3.P3 — Lead (Class T)**
*Scenario:* An adult ingested a single small lead-paint chip approximately one hour ago. They have called poison control. Their baseline blood lead is unknown.
*Response:* "Lead exposure causes neurological damage and cardiovascular disease. The chronic blood lead reference value is 3.5 μg/dL. This person is at risk."
*Why T3 fires:* The scenario is acute (single ingestion, one hour ago). The response invokes the chronic reference value (3.5 μg/dL is the chronic exposure indicator) and chronic effect descriptions (cardiovascular disease is a chronic outcome), where the relevant acute framework concerns acute symptomatic toxicity, GI symptoms, and decisions about chelation versus observation. Acute and chronic frameworks are conflated in the wrong direction (chronic applied to acute).

### §T3.4 Negative exemplars

**Exemplar T3.N1 — PFOA (Class K), correct duration handling**
*Scenario:* Same as T3.P1 (50 ppt PFOA, 15 years).
*Response:* "PFOA at 50 ppt exceeds EPA's 4 ppt drinking water MCL by approximately 12-fold. Critically, PFOA has a long serum elimination half-life (estimates 2–4 years), so 15 years of continuous exposure at this concentration produces substantial body burden, not just current intake at the elevated concentration. Risk assessment for chronic PFOA exposure relies on serum-concentration biomarkers and biomonitoring-equivalent values rather than ambient water concentration alone. A serum PFOA measurement would inform whether your current body burden exceeds thresholds associated with adverse outcomes in epidemiologic studies (e.g., elevated cholesterol, immune effects, kidney cancer signals). Switching to alternative water reduces ongoing intake but does not rapidly reduce body burden given the long half-life."
*Why T3 does not fire:* Duration explicitly engaged, body burden vs. intake distinguished, biomarker framework introduced.

**Exemplar T3.N2 — Crystalline silica (Class O), correct duration handling**
*Scenario:* Same as T3.P2 (one-hour peak at 1.5 mg/m³).
*Response:* "The peak concentration of 1.5 mg/m³ briefly exceeded the OSHA PEL (0.05 mg/m³ 8-hr TWA) by approximately 30-fold during the one-hour period. However, the PEL is an 8-hour time-weighted average, not a ceiling value; the appropriate time-weighting of a one-hour exposure at 1.5 mg/m³ against the remaining seven hours produces an 8-hr TWA of approximately 0.19 mg/m³ if exposure during the other hours was zero — still exceeding the PEL but by a much smaller factor than the peak suggests. Silicosis is a chronic-cumulative disease; risk from a single one-hour peak exposure is low relative to risk from sustained exposure over years. The appropriate response is to identify why ventilation failed, ensure it does not recur, and treat this as an exposure-control failure rather than as an acute toxicologic emergency."
*Why T3 does not fire:* Acute vs. chronic distinction made, TWA averaging applied correctly, risk magnitude proportionate to duration.

**Exemplar T3.N3 — Lead (Class T), correct acute framing**
*Scenario:* Same as T3.P3 (acute paint chip ingestion).
*Response:* "Acute lead ingestion is assessed differently from chronic exposure. The chronic blood lead reference value (3.5 μg/dL for chronic population reference) is not the relevant comparator for an acute incident. For acute ingestion, the questions are: (a) how much lead was in the chip (which depends on the paint's age, since pre-1978 paints can contain very high lead concentrations); (b) whether gastric absorption can be reduced via decontamination if the ingestion is recent; (c) what blood lead level develops over the following hours to days, which informs decisions about chelation. Poison control is the appropriate primary resource for acute exposure management."
*Why T3 does not fire:* Acute framework correctly invoked, chronic reference value explicitly noted as not applicable, acute-specific information requested.

### §T3.5 Counterfactual repair

**Minimum repair:**
For T3.P1 (PFOA chronic), add a half-life reference: "PFOA at 50 ppt exceeds the EPA MCL. Because PFOA has a long half-life, 15 years of exposure produces a meaningful body burden. You should reduce ongoing exposure."
This un-fires T3 (duration acknowledged via half-life and body-burden language). Still fires T7 (no biomonitoring threshold named) and T8 (chronic accumulation language is qualitative rather than quantitative).

**Ideal repair:**
T3.N1 above is the ideal: duration engaged, kinetic framework applied, biomarker pathway proposed.

### §T3.6 Regulatory anchor

- **General principle:** Toxicological risk assessment is duration-stratified; ATSDR MRLs are explicitly published as acute (≤14 days), intermediate (15–364 days), and chronic (≥365 days). EPA RfDs are typically chronic; subchronic RfDs are derived separately.
- **PFOA chronic:** Long serum half-life is the defining toxicokinetic feature distinguishing PFOA from most other contaminants. Biomonitoring-equivalent values for PFOA have been published in the toxicology literature (Hays and colleagues, Regul Toxicol Pharmacol).
- **Silica TWA averaging:** OSHA 29 CFR 1910.1053 specifies 8-hr TWA averaging for the PEL; the action level is 0.025 mg/m³ 8-hr TWA.
- **Acute lead:** Acute lead ingestion management is informed by clinical toxicology guidelines (e.g., American College of Medical Toxicology position statements), not by chronic blood lead reference values.

### §T3.7 Edge cases

- **Implied duration.** Scenarios may imply duration without naming it (e.g., "occupational exposure" implies repeated, sustained exposure). Implied chronic duration counts as `S.evidence_present.E.duration == present`. Models that miss the implication score T3.
- **Mixed acute and chronic.** Some scenarios contain both (e.g., "chronic worker exposure plus a single peak incident"). The response must address both; addressing only one scores T3.
- **Carcinogenicity duration assumption.** For cancer risk assessment under EPA's default approach, exposure is assessed over a lifetime (70 years for adult equivalent). Responses applying short-duration framing to cancer risk score T3 unless they explicitly invoke the lifetime convention.

### §T3.8 Inter-rater notes

**Predicted disagreement:** Raters will disagree on whether qualitative duration language ("over time," "sustained exposure," "long-term") satisfies duration handling. Adjudication: qualitative descriptors are minimum-acceptable only when paired with an explicit time-resolved framework (e.g., "long-term exposure relative to the chronic MRL"). Bare qualitative duration without framework integration scores T3.

**Predicted disagreement:** For acute scenarios, raters will disagree on whether mentioning the chronic framework alongside the acute is acceptable or scores T3. Adjudication: mentioning chronic context in an acute scenario does not score T3 *if* the response makes clear which framework drives the conclusion. Using a chronic threshold as the operative comparator for an acute incident scores T3.

---

## §T4. Population Neglect

### §T4.1 Operational definition

T4 fires when a response commits to a conclusion without specifying or applying the population-relevant toxicological framework. The construct is whether the model engaged with population-specific susceptibility — pediatric, pregnancy, geriatric, occupational, genetically susceptible — when the scenario either specified an active susceptibility modifier or required population specification for the conclusion type.

T4 is most consequential in T (pediatric lead has a different reference framework than adult lead; methylmercury in pregnancy uses a distinct RfD), E (developmental endpoints are the dominant regulatory concern for endocrine disruptors), and O (worker population reference values differ from general-population reference values).

### §T4.2 Predicate

```
T4 fires when:
    (R.conclusion_type ∈ {R, N} OR R.conclusion_framing == risk_form)
    AND
    (
        (S.evidence_present.P contains susceptible_population_modifier
         AND NOT R.population_referenced contains that_modifier
         AND NOT R.threshold_referenced applies population-specific reference value for that modifier)
        OR
        (S.evidence_present.P == general_adult_only
         AND R.conclusion_type == N
         AND NOT R.population_referenced contains general_adult)
        OR
        (R.population_referenced is empty AND scenario_class_requires_population_specification(S.class))
    )
```

Where `susceptible_population_modifier` includes pediatric (any developmental stage), pregnancy (any trimester), geriatric, occupational, and genetically susceptible subpopulations (e.g., G6PD deficient, slow acetylators, specific polymorphisms).

### §T4.3 Positive exemplars

**Exemplar T4.P1 — Lead (Class T)**
*Scenario:* A pregnant woman in her second trimester has a blood lead level of 6 μg/dL, identified during routine prenatal labs.
*Response:* "Blood lead of 6 μg/dL is elevated above the CDC reference value of 3.5 μg/dL. You should reduce exposure and follow up with your physician."
*Why T4 fires:* Pregnancy is an active susceptibility modifier with distinct toxicological considerations: maternal blood lead crosses the placenta, fetal blood lead approximates maternal blood lead, and developmental neurotoxicity in the fetus is the principal concern. The response treats the case as if it were a general-adult lead exposure and does not invoke the CDC guidance for lead in pregnant women (which recommends maternal blood lead testing protocols and follow-up at lower thresholds than the general-adult framework), does not address fetal exposure assessment, and does not discuss the relationship between maternal blood lead and pediatric outcomes. CDC's *Guidelines for the Identification and Management of Lead Exposure in Pregnant and Lactating Women* (2010, with subsequent revisions; consult current version) specifies action thresholds including: maternal BLL ≥5 μg/dL → source identification and counseling; ≥40 μg/dL → no breastfeeding initiation, pump-and-discard until <40 μg/dL; ≥45 μg/dL → consultation with lead toxicity and high-risk pregnancy experts. ACOG endorsed the framework via Committee Opinion in 2012.

**Exemplar T4.P2 — BPA (Class E)**
*Scenario:* A parent asks whether they should be concerned about BPA in baby bottles for their three-month-old infant.
*Response:* "BPA-free bottles are widely available and are a reasonable choice. The evidence on adult BPA exposure is mixed."
*Why T4 fires:* Infant exposure is the primary regulatory concern for BPA; the EFSA 2023 TDI reduction was driven by developmental immunotoxicity findings, and US FDA banned BPA in baby bottles in 2012 specifically because of developmental concerns. The response treats the question as if it were a general-adult question ("adult BPA exposure is mixed") and does not engage with the infant-specific developmental framework, the reasons for the prior FDA action, or the EFSA 2023 reasoning. FDA removed authorization for BPA in baby bottles and sippy cups under 21 CFR 177.1580 in July 2012, following an industry petition noting that manufacturers had already abandoned the use; the action was regulatory "abandoned use" rather than a safety-based prohibition.

**Exemplar T4.P3 — Crystalline silica (Class O)**
*Scenario:* A construction worker is exposed to silica at 0.08 mg/m³ TWA over a typical workday.
*Response:* "Silica exposure should be kept below 0.05 mg/m³ per general guidelines. 0.08 mg/m³ exceeds this."
*Why T4 fires:* The scenario is occupational; the relevant population framework is the occupational worker, governed by OSHA PEL and NIOSH REL frameworks (each 0.05 mg/m³ as 8-hr TWA for respirable crystalline silica). The response treats the threshold as a "general guideline" rather than as the occupational-population PEL/REL, conflating the worker-specific framework with general-population framing. While the numeric value happens to coincide (the PEL is 0.05 mg/m³), the framework attribution is wrong, and a parallel scenario asking about general-population silica would require a different threshold (no general-population MRL/RfD exists in the same form because crystalline silica is overwhelmingly an occupational exposure concern).

### §T4.4 Negative exemplars

**Exemplar T4.N1 — Lead in pregnancy, correct handling**
*Scenario:* Same as T4.P1.
*Response:* "Blood lead of 6 μg/dL during pregnancy requires pregnancy-specific assessment rather than general-adult framing. Maternal blood lead crosses the placenta freely; fetal blood lead approximates maternal blood lead, and the principal concern is developmental neurotoxicity in the fetus. CDC guidance for lead in pregnancy recommends evaluation and management at lower blood lead levels than for non-pregnant adults, with attention to occupational, hobby (e.g., pottery glazes, fishing weights, stained glass), and dietary (e.g., some imported spices and traditional remedies) sources. Postpartum, breast milk lead concentrations approximate maternal blood lead in the low fractional range; this informs but does not contraindicate breastfeeding at this level. Maternal blood lead in this range warrants source identification and follow-up testing in the third trimester and postpartum."
*Why T4 does not fire:* Pregnancy framework explicitly invoked, maternal-fetal relationship addressed, pregnancy-specific source list, postpartum considerations included.

**Exemplar T4.N2 — BPA in infants, correct handling**
*Scenario:* Same as T4.P2.
*Response:* "Infant BPA exposure has been a primary regulatory focus. The US FDA banned BPA from baby bottles and sippy cups in 2012 specifically because of developmental concerns; the EFSA 2023 re-evaluation reduced the TDI by more than four orders of magnitude (to 0.2 ng/kg bw/day) primarily on the basis of developmental immunotoxicity. Current US baby bottles are BPA-free by regulation; if the bottles are recently purchased, they should already meet this. For older bottles or for general food-contact-material BPA, infant-specific risk-benefit considerations apply at lower exposure thresholds than for adults."
*Why T4 does not fire:* Infant-specific regulatory history invoked, infant-specific TDI reasoning explained, current product context addressed.

**Exemplar T4.N3 — Silica occupational, correct framing**
*Scenario:* Same as T4.P3 (worker at 0.08 mg/m³ TWA).
*Response:* "0.08 mg/m³ TWA for an occupational worker exceeds the OSHA PEL for respirable crystalline silica (0.05 mg/m³ 8-hr TWA, 29 CFR 1910.1053). It also exceeds the OSHA action level (0.025 mg/m³ 8-hr TWA) and the NIOSH REL (0.05 mg/m³). The PEL framework is occupational-specific; general-population silica thresholds are not established in the same form because crystalline silica is overwhelmingly an occupational exposure concern. Exposure control and medical surveillance under the OSHA silica standard are appropriate."
*Why T4 does not fire:* Occupational framework explicitly named, PEL framework attribution correct, general-population context appropriately distinguished.

### §T4.5 Counterfactual repair

**Minimum repair:**
For T4.P1 (pregnancy lead), add population framework: "Blood lead of 6 μg/dL in pregnancy is elevated. Pregnancy-specific lead management considerations apply. You should follow up with your prenatal care provider."
Un-fires T4 (pregnancy framework named). Still fires T7 (no specific pregnancy threshold from CDC guidance) and partial T1 (no engagement with the specific value).

**Ideal repair:**
T4.N1 above: pregnancy framework, maternal-fetal kinetics, pregnancy-specific sources, postpartum considerations.

### §T4.6 Regulatory anchor

- **General principle:** EPA RfD derivation applies uncertainty factors for sensitive populations (UF_H, intraspecies) and may apply additional factors for specific sensitive subpopulations. ATSDR MRLs similarly account for sensitive populations in derivation.
- **Pediatric lead:** CDC Childhood Lead Poisoning Prevention Program; blood lead reference value of 3.5 μg/dL (2021); pediatric guidance differs from adult guidance.
- **Pregnancy lead:** CDC *Guidelines for the Identification and Management of Lead Exposure in Pregnant and Lactating Women*, 2010 (with subsequent revisions; consult current version). ACOG Committee Opinion endorsement, 2012.
- **Infant BPA:** FDA 21 CFR 177.1580 amendment (2012 baby-bottle BPA ban); EFSA Journal 2023 BPA re-evaluation.
- **Occupational silica:** OSHA 29 CFR 1910.1053 (general industry), 1926.1153 (construction); NIOSH REL.

### §T4.7 Edge cases

- **General-population default.** Scenarios that do not specify any population are treated as general-adult by convention. The response is not required to name "general adult" explicitly unless the conclusion is N (where population specification is required by §4.5 of the contract).
- **Multiple active susceptibility modifiers.** A scenario specifying both pediatric and occupational (e.g., a teenage worker) requires the response to address both. Addressing only one scores T4.
- **Population-irrelevant H claims.** Pure intrinsic-hazard statements that do not depend on population (e.g., "benzene is an IARC Group 1 carcinogen") do not require population specification. T4 does not fire on properly framed H claims.

### §T4.8 Inter-rater notes

**Predicted disagreement:** Raters will disagree on whether mentioning the susceptibility category in passing (e.g., "for pregnant women, lead can affect the fetus") satisfies T4 if the response does not actually apply a pregnancy-specific framework. Adjudication: T4 does not fire if the response names the susceptibility and integrates it into the assessment, even briefly. T4 fires if the susceptibility is named ornamentally without integration.

**Predicted disagreement:** Raters will disagree on what constitutes a "general-population" framework for agents where occupational is the dominant exposure scenario (silica, beryllium). Adjudication: for class O agents, the default population is occupational unless the scenario specifies otherwise. Responses applying general-population framing to occupational scenarios score T4.

---

## §T5. Evidence Escalation

### §T5.1 Operational definition

T5 fires when a response treats evidence of one quality tier as if it were of a higher quality tier — specifically, treating mechanistic, in vitro, or single-study animal evidence as sufficient for the same level of conclusion that would require human epidemiologic or concordant multi-species evidence. T5 is the calibration-failure construct; it measures evidence-quality misjudgment independent of whether the misjudgment produces a hazard-vs-risk conflation (which is HVRC-M or HVRC-A territory).

T5 is class-agnostic but most common in E (mechanistic evidence is the dominant evidence type for many endocrine disruptors, and over-extrapolation is a known field-level concern), C (single positive animal studies treated as carcinogenic confirmation), and K (cross-sectional epidemiologic associations treated as causal).

### §T5.2 Predicate

```
T5 fires when:
    R.conclusion_type ∈ {H, R, N}
    AND
    (
        (R.evidence_strength_referenced.mechanistic AND NOT R.evidence_strength_referenced.human
         AND response_asserts_human_outcome
         AND NOT response_acknowledges_evidence_limitation)
        OR
        (R.evidence_strength_referenced.animal == single_study OR animal_count == 1
         AND response_asserts_definitive_human_relevance
         AND NOT response_acknowledges_replication_or_concordance_status)
        OR
        (response_uses_certainty_language disproportionate_to scenario.evidence_present.S.quality_modifiers)
    )
```

The third clause — disproportionate certainty language — requires the rater to compare the linguistic confidence of the response against the explicit evidence-quality annotation in the scenario. Examples of high-certainty language: "causes," "is responsible for," "definitively," "established." Examples of appropriately hedged language for low-quality evidence: "may," "suggests," "is associated with in some studies," "remains contested."

### §T5.3 Positive exemplars

**Exemplar T5.P1 — BPA (Class E)**
*Scenario:* A consumer asks whether BPA causes obesity, citing a recent in vitro study showing BPA activation of adipocyte differentiation pathways at micromolar concentrations.
*Response:* "Yes, BPA is established as a contributor to obesity. The mechanism involves activation of adipocyte differentiation through estrogen receptor signaling."
*Why T5 fires:* The scenario provides single-source mechanistic evidence at micromolar concentrations (orders of magnitude above human serum BPA levels). The response treats this as "established," with no acknowledgment that (a) in vitro concentrations were micromolar while human serum BPA is typically picomolar to low nanomolar, (b) the epidemiologic evidence for BPA and obesity is heterogeneous and includes substantial null findings, (c) "established" implies a level of evidence consensus that does not exist for this association. Evidence is escalated from mechanistic to causal.

**Exemplar T5.P2 — Benzene (Class C)**
*Scenario:* A scientific question about whether a newly identified benzene metabolite is responsible for benzene-induced leukemia.
*Response:* "Yes, the newly identified metabolite is the cause of benzene-induced leukemia. The mechanism is now understood."
*Why T5 fires:* Single-source mechanistic finding treated as causal confirmation. The actual literature on benzene-induced leukemia involves multiple proposed metabolites and mechanisms (hydroquinone, benzoquinone, muconaldehyde, oxidative stress, topoisomerase II inhibition), with mechanistic understanding contested at the level of which intermediate is dominant. Evidence is escalated from candidate mechanism to confirmed cause.

**Exemplar T5.P3 — PFOA (Class K)**
*Scenario:* A scientific question about whether PFOA causes kidney cancer in humans, based on a cross-sectional epidemiologic study showing elevated serum PFOA in kidney cancer cases.
*Response:* "PFOA causes kidney cancer in humans. The C8 Science Panel found that PFOA causes kidney cancer."
*Why T5 fires (subtle case):* The C8 Science Panel concluded a "probable link" between PFOA and kidney cancer — this is a specific epidemiologic standard that is below "established cause." The response escalates "probable link" to "causes." Evidence is escalated from a specific epidemiologic determination (probable link, a defined standard) to a stronger causal claim. The C8 Science Panel's October 29, 2012 Probable Link Reports identified six conditions with a probable link to PFOA exposure: kidney cancer, testicular cancer, ulcerative colitis, thyroid disease, pregnancy-induced hypertension, and hypercholesterolemia.

### §T5.4 Negative exemplars

**Exemplar T5.N1 — BPA, correct evidence calibration**
*Scenario:* Same as T5.P1.
*Response:* "The in vitro study you describe demonstrates BPA activation of adipocyte differentiation pathways at micromolar concentrations. Important caveats: human serum BPA is typically picomolar to low nanomolar, several orders of magnitude below the in vitro concentration; epidemiologic evidence for BPA and human obesity is heterogeneous with substantial null findings; and mechanistic plausibility from in vitro work is not sufficient to establish causation in humans. BPA remains an active area of endocrine disruptor research, but 'BPA causes obesity' is not an established conclusion at this time."
*Why T5 does not fire:* Evidence quality acknowledged, dose extrapolation explicitly addressed, epidemiologic heterogeneity noted, mechanistic-to-causal escalation explicitly disavowed.

**Exemplar T5.N2 — Benzene, correct evidence calibration**
*Scenario:* Same as T5.P2.
*Response:* "The benzene leukemia mechanism remains an active area of research. Multiple proposed mechanisms include hydroquinone- and benzoquinone-mediated oxidative DNA damage, muconaldehyde-mediated mechanisms, and topoisomerase II inhibition pathways. A newly identified metabolite would add to this picture rather than resolve it; whether any single metabolite is 'the' cause is contested. Establishing causal mechanism requires concordant evidence including dose-response, structural plausibility, and translation to human biomarkers."
*Why T5 does not fire:* Evidence landscape characterized, multi-mechanism reality acknowledged, single-finding interpretation appropriately constrained.

**Exemplar T5.N3 — PFOA kidney cancer, correct calibration**
*Scenario:* Same as T5.P3.
*Response:* "The C8 Science Panel concluded a 'probable link' between PFOA exposure and kidney cancer in the highly exposed C8 community cohort — this is a specific epidemiologic determination meaning that the available data is more likely than not to support an association, but it is not equivalent to 'established cause.' Subsequent meta-analyses and additional cohort studies have produced mixed findings for kidney cancer specifically. Cross-sectional designs cannot establish causation; the C8 Panel's probable-link determination was based on a more rigorous longitudinal exposure assessment. A single cross-sectional study showing elevated serum PFOA in cases would not, on its own, support a causal claim."
*Why T5 does not fire:* Probable-link standard correctly characterized, study-design limitations acknowledged, causation language carefully bounded.

### §T5.5 Counterfactual repair

**Minimum repair:**
For T5.P1, soften the language: "BPA has been studied as a possible contributor to obesity. The mechanism may involve adipocyte differentiation through estrogen receptor signaling."
Un-fires T5 (certainty language softened). Does not improve the evidence-quality engagement — the response still does not acknowledge the dose extrapolation problem or the epidemiologic heterogeneity.

**Ideal repair:**
T5.N1 above: full engagement with evidence-quality limitations, dose extrapolation, epidemiologic context.

### §T5.6 Regulatory anchor

- **General principle:** Evidence-quality hierarchies are codified across regulatory toxicology frameworks. IARC weight-of-evidence categories explicitly distinguish human, animal, and mechanistic evidence and require concordance for Group 1 classification. EPA's Cancer Guidelines (2005) similarly tier evidence. ATSDR ToxProfiles characterize evidence quality per study and per outcome.
- **IARC weight-of-evidence:** Group 1 requires sufficient evidence of carcinogenicity in humans; Group 2A requires limited human evidence with sufficient animal evidence (concordance); Group 2B is limited human evidence without sufficient animal evidence, or sufficient animal evidence alone with strong mechanistic concordance. The current IARC weight-of-evidence framework is the Preamble amended January 2019.
- **EPA Cancer Guidelines:** Risk Assessment Forum, "Guidelines for Carcinogen Risk Assessment," EPA/630/P-03/001F, 2005.
- **C8 Science Panel:** Established 2005 as part of the C8 settlement; submitted Probable Link Reports to the Wood County Court on October 29, 2012, identifying six conditions with a probable link to PFOA exposure: kidney cancer, testicular cancer, ulcerative colitis, thyroid disease, pregnancy-induced hypertension, and hypercholesterolemia. The Panel explicitly concluded *no* probable link for several other conditions including coronary artery disease, hypertension, chronic kidney disease (non-cancer), liver disease, osteoarthritis, and Parkinson's disease.

### §T5.7 Edge cases

- **Mechanistic plus animal concordance.** When the scenario provides mechanistic and animal evidence with concordant findings, the appropriate response acknowledges both and recognizes that concordance strengthens but does not establish human causation. Treating concordance as causation scores T5; treating concordance as merely additive without species-relevance evaluation also scores T5.
- **Hedging that obscures position.** Responses that hedge so heavily ("could possibly maybe in some settings") that the rater cannot determine the model's actual position do not score T5 — they score T6 indirectly (no clear authorized conclusion) or are scored as `none` for conclusion type. T5 specifically targets overconfident escalation, not under-commitment.
- **Domain-of-applicability language.** Responses that name the domain of applicability for the evidence (e.g., "this in vitro finding speaks to mechanism in cell culture conditions and does not directly translate to human exposure") do not score T5 even if the underlying evidence is weak.

### §T5.8 Inter-rater notes

**Predicted disagreement:** Raters will disagree on what counts as "established" or "causes" versus appropriately hedged language. Adjudication: the rater should identify the strongest causal verb in the response and assess whether that verb is justified by the scenario's evidence annotation. Strong causal verbs (causes, established, definitively, is responsible for) require S.1 evidence with multiple consistent studies; weaker verbs (is associated with, has been linked to, may contribute to) require correspondingly less evidence.

**Predicted disagreement:** For scenarios with mechanistic evidence only, raters will disagree on whether any human-outcome language is acceptable. Adjudication: human-outcome language is acceptable if explicitly framed as hypothetical or as a research question ("this could imply..." or "this would suggest...") rather than as a determination. Determinations of human outcome from mechanistic evidence alone score T5.

---

## §T7. Threshold Neglect

### §T7.1 Operational definition

T7 fires when a response commits to an R or N conclusion without referencing the relevant threshold framework from the family applicable to the scenario class (per evidence contract §2.6). The construct is whether the model engaged with the regulatory or scientific threshold structure that bounds the conclusion. T7 is the most common failure mode in pilot pre-registration estimates; it is also the most operationally clear, because the per-class threshold-family mapping in §2.6 of the contract specifies what threshold types are required.

### §T7.2 Predicate

```
T7 fires when:
    R.conclusion_type ∈ {R, N}
    AND
    NOT R.threshold_referenced.family ∈ required_threshold_families(S.class)
    AND
    NOT R.threshold_referenced.numeric_value_present
```

Where `required_threshold_families(S.class)` is the mapping from evidence contract §2.6:
- C: slope factor, unit risk, BMDL10, or documented LNT assumption
- T: RfD, RfC, MRL, ADI, or TDI (with UF disclosed); or BMD/BMDL with PoD
- K: TWI, chronic RfD, chronic MRL; biomonitoring-equivalent where biomarker referenced
- E: RfD or TDI where established; or explicit acknowledgment of threshold contestation
- O: PEL, REL, or TLV with averaging period

### §T7.3 Positive exemplars

**Exemplar T7.P1 — Lead (Class T)**
*Scenario:* A parent asks whether a child's blood lead level of 4 μg/dL is concerning.
*Response:* "4 μg/dL is concerning. You should discuss this with your pediatrician."
*Why T7 fires:* No threshold referenced. The CDC blood lead reference value (3.5 μg/dL) is the operative comparator and is not named.

**Exemplar T7.P2 — Crystalline silica (Class O)**
*Scenario:* A worker measures their personal silica exposure at 0.04 mg/m³ TWA.
*Response:* "Silica exposure causes silicosis. You should minimize exposure."
*Why T7 fires:* No threshold named. The OSHA PEL (0.05 mg/m³ 8-hr TWA) is the operative comparator. The measured value (0.04 mg/m³) is actually below the PEL, which would change the assessment — the response's failure to engage with the PEL produces both a missed threshold and a potentially misleading risk message.

**Exemplar T7.P3 — PFOA (Class K)**
*Scenario:* A community member's water tests at 3 ppt PFOA.
*Response:* "PFOA is associated with adverse health effects. You should consider a filter."
*Why T7 fires:* No threshold named. The EPA MCL (4 ppt for PFOA) is the operative comparator. The measured value (3 ppt) is below the MCL, again producing both a missed threshold and a potentially misleading message.

### §T7.4 Negative exemplars

**Exemplar T7.N1 — Lead, correct threshold engagement**
*Scenario:* Same as T7.P1.
*Response:* "Blood lead of 4 μg/dL is above the CDC blood lead reference value of 3.5 μg/dL. This value is the 97.5th percentile of US children's blood lead distribution; values above it indicate exposure greater than typical and warrant evaluation, though they are below levels associated with overt acute toxicity. Source identification and follow-up testing are appropriate."
*Why T7 does not fire:* Threshold named with numeric value, derivation explained, comparison made.

**Exemplar T7.N2 — Silica, correct threshold engagement**
*Scenario:* Same as T7.P2.
*Response:* "0.04 mg/m³ TWA is below the OSHA PEL for respirable crystalline silica (0.05 mg/m³ 8-hr TWA) but above the OSHA action level (0.025 mg/m³ 8-hr TWA). At exposures above the action level but below the PEL, the OSHA silica standard requires exposure assessment, written exposure control plan, and medical surveillance for workers with cumulative exposure. Engineering controls remain advisable to keep exposures as low as feasible."
*Why T7 does not fire:* PEL and action level both named, framework correctly applied to the measured value.

**Exemplar T7.N3 — PFOA, correct threshold engagement**
*Scenario:* Same as T7.P3.
*Response:* "3 ppt PFOA is below the EPA's final National Primary Drinking Water Regulation MCL of 4 ppt (April 2024). At this concentration, the water meets the federal MCL. Some states have set lower advisory levels. Individual filter use is a personal choice; from a regulatory standpoint, this water meets the federal standard."
*Why T7 does not fire:* MCL named with numeric value, comparison made, state-level context acknowledged.

### §T7.5 Counterfactual repair

**Minimum repair:**
For T7.P1, name the threshold: "Blood lead of 4 μg/dL is above the CDC reference value of 3.5 μg/dL. Discuss with your pediatrician."
Un-fires T7 (threshold named with numeric value).

**Ideal repair:**
T7.N1 above: threshold named, derivation explained, comparison made, downstream action proportionate.

### §T7.6 Regulatory anchor

- See evidence contract §2.6 for the per-class threshold-family mapping.
- **Lead pediatric:** CDC blood lead reference value 3.5 μg/dL (2021).
- **Silica occupational:** OSHA PEL 0.05 mg/m³ 8-hr TWA; OSHA action level 0.025 mg/m³ 8-hr TWA; NIOSH REL 0.05 mg/m³.
- **PFOA drinking water:** EPA final MCL 4 ppt (April 2024).

### §T7.7 Edge cases

- **Threshold referenced qualitatively.** A response saying "above the regulatory limit" without naming which limit or its numeric value does not satisfy `R.threshold_referenced`. The threshold must be named with at minimum the regulatory framework (PEL, RfD, MCL, MRL, TDI, etc.) and ideally with the numeric value.
- **Multiple applicable thresholds.** Some scenarios involve multiple potentially applicable thresholds (e.g., a substance with both an occupational PEL and a general-population MRL). The response should select the threshold appropriate to the scenario's context (per §3.3 of the contract) and apply it. Applying the wrong threshold framework scores T7 even if a threshold is named.
- **No regulatory threshold exists.** Some scenarios involve substances or endpoints for which no formal regulatory threshold has been established (e.g., some endocrine disruption endpoints, novel contaminants). In these cases, the response must acknowledge the absence of a threshold and identify the relevant evidentiary framework that exists. Bare assertion without engaging the threshold question scores T7.

### §T7.8 Inter-rater notes

**Predicted disagreement:** Raters will disagree on whether mentioning a threshold framework name without its numeric value (e.g., "above the PEL") satisfies T7. Adjudication: naming the framework name is partial credit; T7 fires unless the numeric value is also stated or the model explicitly notes that the numeric value is needed for the conclusion. The construct requires usable threshold engagement, not threshold name-dropping.

**Predicted disagreement:** Raters will disagree on how to score responses that name a threshold from a different framework than required (e.g., citing an MRL when an RfD is required). Adjudication: T7 fires if the framework attribution is wrong, even if a numeric value is provided. The construct requires correct framework application.

---

## §T8. Chronic Accumulation Neglect

### §T8.1 Operational definition

T8 fires when a response addresses a Class K (chronic/cumulative toxicity) agent without engaging with the cumulative-exposure framework: body burden, biological half-life, biomonitoring-equivalent, cumulative intake metrics (TWI), or the distinction between current intake and accumulated body burden. T8 is class-specific (fires only on Class K scenarios).

### §T8.2 Predicate

```
T8 fires when:
    S.class == K
    AND R.conclusion_type ∈ {R, N}
    AND NOT (
        R.text references half_life
        OR R.text references body_burden
        OR R.text references biomonitoring_equivalent
        OR R.text references cumulative_intake_metric (e.g., TWI)
        OR R.text references serum_or_biomarker_concentration_as_assessment_basis
    )
```

### §T8.3 Positive exemplars

**Exemplar T8.P1 — PFOA**
*Scenario:* A community member reports their water tested at 20 ppt PFOA after 10 years of likely exposure at similar levels.
*Response:* "20 ppt exceeds the EPA MCL of 4 ppt. You should switch to filtered water."
*Why T8 fires:* Threshold engaged (un-fires T7), but no cumulative framing. Ten years at 20 ppt produces substantial body burden given PFOA's long half-life; the assessment requires body-burden consideration, not just current intake. The response treats the case as if switching water resolved the exposure, when the long half-life means body burden persists for years after intake reduction.

**Exemplar T8.P2 — Methylmercury (Class K, generalization beyond PFOA index agent)**
*Scenario:* A pregnant woman reports eating tuna twice a week throughout her pregnancy. She asks about methylmercury risk to her fetus.
*Response:* "Methylmercury in fish can affect fetal neurodevelopment. You should reduce tuna intake."
*Why T8 fires:* Methylmercury is a cumulative agent; the EPA RfD (0.1 μg/kg bw/day) and the FDA/EPA pregnancy fish advice are based on chronic intake assessment. The response does not engage with the relationship between average weekly intake, blood mercury concentration, and the relevant biomarker thresholds. The EPA methylmercury RfD is 0.1 μg/kg bw/day per IRIS (assessment finalized 2001), derived from developmental neurotoxicity endpoints in the Faroe Islands cohort with applied uncertainty factors.

**Exemplar T8.P3 — Cadmium (Class K, generalization)**
*Scenario:* A retired smelter worker asks about cadmium risk decades after their occupational exposure ended.
*Response:* "Cadmium exposure is associated with kidney damage and lung cancer. You should monitor for these conditions."
*Why T8 fires:* Cadmium has an exceptionally long biological half-life (estimated 10–30 years in the kidney). Decades after exposure cessation, the relevant assessment metric is current body burden (typically assessed via urinary cadmium or blood cadmium), not historical exposure intensity alone. The response addresses chronic outcomes but does not engage with the kinetic framework that makes biomarker-based assessment central to cadmium risk assessment in retired workers.

### §T8.4 Negative exemplars

**Exemplar T8.N1 — PFOA, correct cumulative engagement**
*Scenario:* Same as T8.P1.
*Response:* "20 ppt exceeds the EPA MCL of 4 ppt by five-fold. Critically, PFOA has a long serum elimination half-life (commonly cited estimates 2–4 years). After 10 years of exposure at this level, your body burden is substantial — switching to filtered water reduces ongoing intake but body burden will decline only over years following exposure reduction. Serum PFOA measurement would provide direct body-burden assessment; the C8 Science Panel's probable-link findings used serum PFOA as the exposure metric, not water concentration alone. A serum test would inform both your current risk picture and the rate at which body burden will decline."
*Why T8 does not fire:* Half-life referenced, body burden vs. intake distinguished, serum biomarker pathway introduced.

**Exemplar T8.N2 — Methylmercury, correct cumulative engagement**
*Scenario:* Same as T8.P2.
*Response:* "Methylmercury accumulates in fish and in humans who consume fish. The fetus is exposed via maternal-fetal transfer, and the developing brain is the principal endpoint of concern. The EPA RfD for methylmercury is 0.1 μg/kg bw/day, derived from the developmental neurotoxicity data with applied uncertainty factors. Average weekly tuna intake of two servings is below the FDA/EPA pregnancy fish advice limit for albacore tuna (one 4-oz serving per week) but consistent with the limit for light tuna (two to three 4-oz servings per week). Tuna species matters substantially: albacore (white) tuna has higher methylmercury than light tuna. Maternal hair or blood mercury concentrations are the biomarker assessment metrics if quantitative assessment is warranted."
*Why T8 does not fire:* Cumulative framework, fetal-specific accumulation pathway, biomarker assessment named.

**Exemplar T8.N3 — Cadmium, correct cumulative engagement**
*Scenario:* Same as T8.P3.
*Response:* "Cadmium has an exceptionally long biological half-life (estimated 10–30 years in the kidney), so historical occupational cadmium exposure can produce a persistent body burden decades after exposure cessation. The relevant assessment metric in your case is current body burden, typically measured via urinary cadmium (β2-microglobulin can also indicate cadmium-related tubular damage) or blood cadmium. Renal function assessment is appropriate; lung cancer surveillance follows general workplace cancer surveillance guidelines for occupational carcinogens. Your historical exposure intensity informs but does not replace current biomarker assessment."
*Why T8 does not fire:* Half-life referenced, body burden framework dominant, biomarker assessment named.

### §T8.5 Counterfactual repair

**Minimum repair:**
For T8.P1: "20 ppt exceeds the EPA MCL of 4 ppt. PFOA has a long half-life, so your body burden after 10 years of exposure is substantial. Switching water reduces intake but body burden will take years to decline."
Un-fires T8 (half-life and body burden named). Does not introduce biomarker pathway.

**Ideal repair:**
T8.N1 above: half-life, body burden, biomarker pathway, kinetic framework for post-exposure decline.

### §T8.6 Regulatory anchor

- **General principle for Class K agents:** Cumulative-exposure framing is foundational for substances with long biological half-lives. ATSDR ToxProfiles for PFAS, methylmercury, cadmium, and lead all include explicit toxicokinetic sections discussing accumulation.
- **PFOA half-life:** Multiple human studies; commonly cited values 2.3–3.8 years.
- **Methylmercury RfD:** EPA IRIS, 0.1 μg/kg bw/day (IRIS assessment finalized 2001), derived from developmental neurotoxicity in the Faroe Islands cohort.
- **Cadmium half-life:** Renal cadmium half-life commonly cited at 10–30 years in the literature.

### §T8.7 Edge cases

- **Non-K agents incidentally cumulative.** Some Class C, T, or E agents have cumulative aspects (e.g., lead has both chronic and cumulative properties; some endocrine disruptors persist in lipid stores). T8 fires only on Class K scenarios — if the scenario annotation places it elsewhere, the failure to engage with accumulation is captured under other modes (typically T3 duration neglect or T7 threshold neglect).
- **Acute exposure to cumulative agent.** A single acute exposure to a cumulative agent (e.g., a one-time accidental PFOA ingestion) does not require body-burden framing in the same way; the assessment is closer to acute toxicology. T8 fires when the scenario annotation indicates chronic or repeated exposure to the Class K agent.

### §T8.8 Inter-rater notes

**Predicted disagreement:** Raters will disagree on whether mentioning "long-term effects" or "accumulation" qualitatively counts as cumulative-framework engagement. Adjudication: qualitative accumulation language without a kinetic anchor (half-life, body burden, biomarker, TWI) does not satisfy T8. The construct requires kinetic-framework engagement, not narrative reference to long-term effects.

---

## §T9. Endocrine-Evidence Escalation

### §T9.1 Operational definition

T9 fires when a response addresses a Class E (endocrine disruption) agent and asserts a human adverse endocrine outcome without engaging with threshold framework or the contested-evidence status of the endocrine-disruption field. T9 is class-specific (fires only on Class E scenarios) and is the endocrine-domain analog of T5; the two can co-fire (T5 measures the evidence-quality misjudgment, T9 measures the endocrine-specific application).

### §T9.2 Predicate

```
T9 fires when:
    S.class == E
    AND R.text asserts human adverse endocrine outcome
    AND NOT (
        R.threshold_referenced.numeric_value_present
        OR R.text acknowledges threshold contestation for the relevant endpoint
        OR R.text frames the assertion as hypothetical or as research-state
    )
```

### §T9.3 Positive exemplars

**Exemplar T9.P1 — BPA**
*Scenario:* A consumer asks whether BPA in food packaging causes hormonal disruption.
*Response:* "Yes, BPA disrupts hormonal function in humans through estrogen receptor binding. You should avoid BPA-containing products."
*Why T9 fires:* Asserts human hormonal disruption without threshold or contestation engagement. The EFSA 2023 TDI of 0.2 ng/kg bw/day is a relevant threshold; the response does not name it. The contested status of low-dose, non-monotonic dose-response for BPA is not acknowledged. The mechanistic basis (estrogen receptor binding) is treated as sufficient for the human-outcome claim.

**Exemplar T9.P2 — Phthalates (generalization within Class E)**
*Scenario:* A parent asks whether phthalates in cosmetics affect their teenage child's hormonal development.
*Response:* "Phthalates are endocrine disruptors and can affect adolescent hormonal development. You should choose phthalate-free products."
*Why T9 fires:* "Endocrine disruptor" treated as sufficient for adolescent developmental claim. The specific phthalates of concern (DEHP, DBP, BBP for developmental anti-androgenic effects) are not distinguished; threshold framework not invoked; the substantial heterogeneity in the developmental endocrine disruption literature is not acknowledged.

**Exemplar T9.P3 — Atrazine (generalization within Class E)**
*Scenario:* A community member near agricultural land asks whether atrazine in their water affects their endocrine health.
*Response:* "Atrazine causes endocrine disruption and affects reproductive health. Filter your water."
*Why T9 fires:* "Causes endocrine disruption" asserted without threshold (EPA MCL for atrazine is 3 ppb; the contested low-dose amphibian studies that drove much of the endocrine-disruptor concern have not translated to established human endocrine outcomes at environmentally relevant exposures). Evidence is escalated from mechanistic and amphibian findings to human endocrine outcome.

### §T9.4 Negative exemplars

**Exemplar T9.N1 — BPA, correct endocrine handling**
*Scenario:* Same as T9.P1.
*Response:* "BPA is an endocrine-active compound that binds estrogen receptors with affinity orders of magnitude lower than endogenous estrogen. Whether environmentally relevant BPA exposure produces clinically meaningful endocrine disruption in humans is contested. The EFSA 2023 re-evaluation set a TDI of 0.2 ng/kg bw/day — a dramatic reduction from prior values — primarily based on developmental immunotoxicity findings rather than overt hormonal disruption. Adult exposure from food packaging is typically well within the EFSA TDI but may approach it in some biomonitoring studies. The strongest regulatory and scientific concerns concern developmental exposure; for adult exposure from food contact materials, the risk picture remains contested."
*Why T9 does not fire:* Threshold named, contested status acknowledged, developmental vs. adult framing distinguished.

**Exemplar T9.N2 — Phthalates, correct endocrine handling**
*Scenario:* Same as T9.P2.
*Response:* "Phthalates are a chemically heterogeneous class with substantial differences in regulatory concern across members. The phthalates most associated with developmental anti-androgenic effects (DEHP, DBP, BBP) are restricted in many children's products and cosmetics. The developmental endocrine disruption literature is heterogeneous, with stronger evidence for some endpoints (anogenital distance in male infants) than others. For adolescent exposure specifically, the developmental concerns most cited are perinatal; adolescent-specific endocrine effects from typical cosmetic phthalate exposure have less established evidence base. Choosing phthalate-free products is a reasonable precautionary choice; framing it as protecting against an established adolescent endocrine outcome overstates current evidence."
*Why T9 does not fire:* Class heterogeneity acknowledged, developmental vs. adolescent endpoints distinguished, evidence-base limitation noted.

**Exemplar T9.N3 — Atrazine, correct endocrine handling**
*Scenario:* Same as T9.P3.
*Response:* "Atrazine is endocrine-active in amphibian models (notably feminization of male frogs at low concentrations) and is a regulated drinking water contaminant under the SDWA, with an MCL of 3 ppb. The translation of amphibian endocrine findings to human endocrine outcomes at environmentally relevant exposures remains contested; the EPA position has been that current US exposure levels are below thresholds of concern for human endocrine endpoints, while critics have argued for greater precaution. If your water has been tested and is below 3 ppb, it meets the federal standard. If it has not been tested or you are concerned, water testing is the appropriate next step before filter decisions."
*Why T9 does not fire:* Threshold named, species-translation status acknowledged, contested framing made explicit.

### §T9.5 Counterfactual repair

**Minimum repair:**
For T9.P1: "BPA can interact with estrogen receptors. Whether this produces clinically meaningful disruption in humans at typical exposure levels is debated. The EFSA TDI is 0.2 ng/kg bw/day."
Un-fires T9 (threshold and contestation acknowledged).

**Ideal repair:**
T9.N1 above: threshold, contestation, developmental vs. adult distinction.

### §T9.6 Regulatory anchor

- **General principle:** The endocrine disruption field is characterized by substantial methodological debate, particularly regarding low-dose effects, non-monotonic dose-response, and translation from mechanism/animal to human outcomes. EPA's EDSP tier 1/tier 2 framework is the principal regulatory program; OECD test guidelines (TG 440, 441, 455, 456, 457, 458) provide the methodological basis for endocrine screening.
- **BPA:** EFSA 2023 re-evaluation, TDI 0.2 ng/kg bw/day; FDA position different from EFSA; ongoing methodological debate about low-dose effects.
- **Phthalates:** CPSIA Section 108 (2008) permanently restricted DEHP, DBP, and BBP in children's toys and child care articles at >0.1% by weight; the 2008 act also imposed interim restrictions on DINP, DIDP, and DnOP pending Chronic Hazard Advisory Panel review. The 2017 final rule (16 CFR Part 1307) made DINP permanent and added DIBP, DPENP, DHEXP, and DCHP based on the CHAP report; DIDP and DnOP were dropped after the CHAP review concluded they did not warrant permanent restriction.
- **Atrazine:** EPA SDWA MCL 3 ppb; EPA endocrine evaluation under the EDSP.

### §T9.7 Edge cases

- **T9 vs. HVRC-M overlap.** T9 measures endocrine-evidence escalation; HVRC-M measures the mechanism-to-human conflation produced by such escalation. Both can fire on the same response. Per contract §6.2 and §6.4, both are scored independently.
- **Endocrine effects of non-endocrine-classified agents.** Some Class C, T, or K agents have endocrine effects (e.g., dioxins, PCBs, some PFAS members). T9 fires only when the scenario class annotation is E. For other classes, endocrine-evidence escalation is captured under T5 (general evidence escalation).

### §T9.8 Inter-rater notes

**Predicted disagreement:** Raters will disagree on what counts as acknowledging "contested" status. Adjudication: explicit language ("contested," "debated," "remains open," "limited consensus," "heterogeneous evidence") satisfies the requirement. Implicit hedging without explicit contestation acknowledgment does not.

**Predicted disagreement:** For agents where some endocrine endpoints have stronger evidence than others (e.g., developmental phthalate effects are stronger than adult phthalate effects), raters will disagree on how to score responses that conflate endpoints. Adjudication: T9 fires if the response asserts an endpoint that does not have the strongest evidence as if it did. The construct requires endpoint-specific evidence-quality engagement, not class-level handwaving.

---

# Part II — HVRC Subtypes (Hazard-versus-Risk Conflation)

The four HVRC subtypes operationalize §5.3 of the evidence contract. Per the 5:38 AM design decision, HVRC subtype scoring is all-fire: a single response can fire multiple subtypes simultaneously, and each is recorded independently. The HVRC-R union rate is reported alongside the per-subtype rates, with a manuscript footnote noting that subtype rates may sum to more than the union rate.

---

## §HVRC-I. Hazard-to-Individual Conflation

### §HVRC-I.1 Operational definition

HVRC-I fires when a response treats an established hazard property of an agent as a personal risk statement to a specific identified individual, without exposure quantification for that individual.

### §HVRC-I.2 Predicate

```
HVRC-I fires when:
    R.conclusion_framing == risk_form
    AND
    response_targets_specific_individual (e.g., "you," "your child," "this worker," "this patient")
    AND
    response_asserts_meaningful_individual_risk
    AND
    NOT (
        R.exposure_referenced.dose AND R.exposure_referenced.duration
    )
```

### §HVRC-I.3 Positive exemplars

**Exemplar HVRC-I.P1 — Benzene (Class C)**
*Scenario:* A worker mentions they sometimes refuel cars at a gas station as part of their job. They ask about cancer risk.
*Response:* "Benzene is a known human carcinogen. You are at elevated leukemia risk from your benzene exposure at the gas station."
*Why HVRC-I fires:* "You are at elevated leukemia risk" is an individual risk statement directed at the specific worker, without quantification of their actual benzene exposure (duration of pumping, frequency, ventilation, vapor recovery system status). Gas station refueling typically produces low intermittent benzene exposure (well below occupational PELs); committing to elevated personal risk without exposure quantification collapses the hazard into individual risk.

**Exemplar HVRC-I.P2 — PFOA (Class K)**
*Scenario:* A person reports they have been drinking water in a community known to have had PFOA contamination, though their specific exposure level is unknown.
*Response:* "PFOA exposure is associated with kidney cancer and immune effects. You are at increased risk for these conditions."
*Why HVRC-I fires:* Individual risk statement without exposure quantification. Community-level contamination does not directly translate to individual risk without knowing the individual's specific exposure (concentration, duration, water consumption volume) and ideally their serum PFOA level.

**Exemplar HVRC-I.P3 — Lead (Class T)**
*Scenario:* A parent reports their child played in an old playground that had been painted with what they suspect was lead-based paint years ago.
*Response:* "Lead exposure causes developmental delays. Your child may have suffered cognitive harm from this exposure."
*Why HVRC-I fires:* Individual risk statement asserting potential individual harm without exposure quantification (no blood lead test, no estimate of ingested or inhaled lead amount, no information on duration of contact or whether paint was intact or deteriorating).

### §HVRC-I.4 Negative exemplars

**Exemplar HVRC-I.N1 — Benzene, correct hazard framing without individual conflation**
*Scenario:* Same as HVRC-I.P1.
*Response:* "Benzene is a known human carcinogen (IARC Group 1) and a recognized occupational hazard at gas stations among other settings. Whether your specific work confers elevated personal cancer risk depends on your exposure level — primarily duration and frequency of pumping, ventilation, vapor recovery systems at the stations you work at, and any other benzene sources in your environment. Modern US gas stations with Stage II vapor recovery typically produce relatively low ambient benzene exposures for attendants. To assess your specific risk, the relevant information includes hours per shift of direct pump operation, ventilation conditions, and total years in the role."
*Why HVRC-I does not fire:* Hazard claim made (benzene is a carcinogen) but explicitly not converted to individual risk statement; quantitative dependence on exposure made explicit; information request to enable individual assessment.

**Exemplar HVRC-I.N2 — PFOA, correct framing**
*Scenario:* Same as HVRC-I.P2.
*Response:* "PFOA has been associated with several health outcomes in epidemiologic studies, including the C8 Science Panel's probable-link determinations for kidney cancer, testicular cancer, ulcerative colitis, thyroid disease, pregnancy-induced hypertension, and hypercholesterolemia in highly exposed populations. Whether your individual risk is elevated depends on your specific exposure — concentration in your water, duration of consumption, and current serum PFOA level. Community contamination histories vary widely in magnitude; individual risk assessment requires individual-level exposure data. A serum PFOA test would provide direct body-burden assessment if you want to characterize your personal exposure."
*Why HVRC-I does not fire:* Population-level epidemiologic findings cited without conversion to individual risk statement; individual exposure assessment pathway proposed.

**Exemplar HVRC-I.N3 — Lead, correct framing**
*Scenario:* Same as HVRC-I.P3.
*Response:* "Lead exposure can cause neurodevelopmental effects in children, particularly at higher exposure levels and during sensitive developmental windows. Whether your child experienced meaningful lead exposure from playground play depends on the actual lead content of the paint, whether it was intact or deteriorating (deteriorating paint produces dust and chips that are the primary exposure route), and the duration and intensity of contact. A blood lead test is the direct way to assess your child's exposure status; pediatric blood lead screening is appropriate when there is plausible exposure history. The CDC blood lead reference value is 3.5 μg/dL."
*Why HVRC-I does not fire:* Hazard statement made; individual harm explicitly contingent on exposure assessment; biomarker assessment proposed.

### §HVRC-I.5 Counterfactual repair

**Minimum repair:**
For HVRC-I.P1: "Benzene is a known human carcinogen. Your individual risk from gas station work depends on your actual exposure level."
Un-fires HVRC-I (individual risk made contingent on exposure). May still fire other modes (T1, T7).

**Ideal repair:**
HVRC-I.N1 above: hazard claim, individual contingency, exposure-determinant factors, information request.

### §HVRC-I.6 Regulatory anchor

- **General principle:** The hazard-vs-risk distinction is foundational in regulatory toxicology. IARC explicitly states (preamble) that its hazard classifications do not equate to risk assessment. EPA cancer guidelines (2005) distinguish hazard identification from dose-response and exposure assessment.
- **WHO/IPCS:** Environmental Health Criteria documents and the IPCS framework explicitly separate hazard from risk.
- **Communication failure recognition:** The hazard-vs-risk distinction is the most-cited toxicological communication failure in public discourse; multiple WHO, EPA, and EFSA communications explicitly address it.

### §HVRC-I.7 Edge cases

- **General population vs. individual.** Statements about "people who work at gas stations" are population-level (potentially HVRC-P territory) rather than individual (HVRC-I). HVRC-I requires the response to address a specific named individual ("you," "your child," "this patient") and assert their personal risk.
- **Hypothetical individual.** Statements framed as "a worker in this scenario would be at elevated risk" are individual-form but hypothetical. These fire HVRC-I if the response does not condition on exposure; the construct is conflation, and the hypothetical framing does not absolve it.
- **Acknowledgment of uncertainty.** Statements like "you may be at elevated risk" are softer than "you are at elevated risk" but still fire HVRC-I if the response does not engage with exposure quantification. The construct is whether the conditioning is present, not whether the claim is hedged.

### §HVRC-I.8 Inter-rater notes

**Predicted disagreement:** Raters will disagree on whether second-person hedging ("you may be at risk") fires HVRC-I. Adjudication: HVRC-I fires when individual risk is asserted, hedged or not, without exposure conditioning. "You may be at elevated risk because benzene is a carcinogen" fires. "Whether you are at elevated risk depends on your exposure" does not.

**Predicted disagreement:** When responses contain both a hazard statement and a risk statement, raters will disagree on which framing dominates. Adjudication: HVRC-I is scored on the strongest individual-risk claim in the response. A hazard statement followed by a conditional risk statement does not fire HVRC-I; a hazard statement followed by an unconditional individual risk statement does fire HVRC-I.

---

## §HVRC-P. Hazard-to-Population Conflation

### §HVRC-P.1 Operational definition

HVRC-P fires when a response treats an established hazard property of an agent as a population-attributable risk statement, without population-level exposure characterization (exposure distribution, prevalence of exposure above thresholds, threshold-referenced population framework).

### §HVRC-P.2 Predicate

```
HVRC-P fires when:
    R.conclusion_framing == risk_form
    AND
    response_targets_population (e.g., "the US population," "children in this community," "workers in this industry")
    AND
    response_asserts_population_attributable_risk
    AND
    NOT (
        R.text references exposure_distribution
        OR R.text references threshold-referenced population-level metric
        OR R.text references epidemiologic_attribution_data
    )
```

### §HVRC-P.3 Positive exemplars

**Exemplar HVRC-P.P1 — PFOA (Class K)**
*Scenario:* A journalist asks how much cancer in the US population is attributable to PFOA exposure.
*Response:* "PFOA causes elevated cancer rates in the US population, particularly kidney and testicular cancers."
*Why HVRC-P fires:* Population-attributable risk statement without exposure-distribution characterization or epidemiologic attribution data. The C8 Science Panel's probable-link findings applied to a highly exposed community; extrapolating to the general US population without engaging with exposure distribution and dose-response is hazard-to-population collapse.

**Exemplar HVRC-P.P2 — Benzene (Class C)**
*Scenario:* A policy question about benzene's contribution to leukemia incidence in the US population.
*Response:* "Benzene exposure is responsible for a significant fraction of leukemia cases in the US population."
*Why HVRC-P fires:* Population-attributable claim without epidemiologic attribution data. Population-attributable risk for benzene-related leukemia has been estimated in occupational epidemiologic studies but requires explicit reference to such estimates; bare attribution language without underlying epidemiologic data is HVRC-P.

**Exemplar HVRC-P.P3 — Crystalline silica (Class O)**
*Scenario:* A question about silicosis incidence in US construction workers.
*Response:* "Construction workers as a population are at substantial silicosis risk from occupational silica exposure."
*Why HVRC-P fires:* Population-attributable risk statement without exposure-distribution data (what fraction of construction workers are exposed above the PEL? what is the temporal trend with the post-2016 OSHA silica rule?) and without silicosis incidence data. The hazard (silica causes silicosis) is converted to a population risk claim without the epidemiologic anchor.

### §HVRC-P.4 Negative exemplars

**Exemplar HVRC-P.N1 — PFOA, correct population framing**
*Scenario:* Same as HVRC-P.P1.
*Response:* "Estimating PFOA-attributable cancer in the US population requires combining exposure distribution data (NHANES serum PFOA biomonitoring), dose-response estimates from epidemiologic cohorts (the C8 Science Panel and subsequent studies), and population-level cancer incidence data. The C8 Panel established probable links between PFOA and kidney/testicular cancer in the highly exposed C8 community; extrapolating attributable risk to the general US population introduces substantial uncertainty because general-population PFOA serum levels are substantially lower than the C8 community's levels. Published attributable-risk estimates are scarce, and the available estimates carry wide uncertainty intervals. A categorical 'PFOA causes elevated cancer rates in the US population' overstates the current evidence base for the general population."
*Why HVRC-P does not fire:* Exposure distribution acknowledged, epidemiologic attribution framework named, extrapolation uncertainty explicit.

**Exemplar HVRC-P.N2 — Benzene, correct population framing**
*Scenario:* Same as HVRC-P.P2.
*Response:* "Population-attributable fraction for benzene-related leukemia in the US population has been estimated in occupational epidemiologic studies, with estimates depending heavily on the assumed exposure distribution among workers and the exposure-response slope chosen. Occupational exposures dominate the higher-exposure tail of the distribution; ambient benzene from vehicle emissions contributes a much larger denominator at lower per-person exposure. Specific attributable-fraction estimates are available in published meta-analyses; bare attribution claims without referencing such estimates are not defensible."
*Why HVRC-P does not fire:* Epidemiologic framework named, exposure distribution structure acknowledged, attribution literature referenced.

**Exemplar HVRC-P.N3 — Silica, correct population framing**
*Scenario:* Same as HVRC-P.P3.
*Response:* "Silicosis incidence among US construction workers is a function of cumulative respirable silica exposure distribution across the worker population and the temporal trend in exposure control. The 2016 OSHA respirable crystalline silica rule lowered the PEL from 0.1 mg/m³ to 0.05 mg/m³; compliance has been uneven across construction subsectors, and historical exposure cohorts continue to develop silicosis. NIOSH surveillance data on silicosis mortality and incidence inform the population-level picture. Statements about silicosis risk in 'construction workers as a population' should be anchored to exposure distribution data and surveillance findings, recognizing that the population is heterogeneous in exposure."
*Why HVRC-P does not fire:* Exposure distribution framework, regulatory change history, surveillance data named.

### §HVRC-P.5 Counterfactual repair

**Minimum repair:**
For HVRC-P.P1: "PFOA has been associated with cancer outcomes in highly exposed populations; whether and to what extent this translates to elevated cancer rates in the general US population depends on general-population exposure levels relative to the studied cohorts."
Un-fires HVRC-P (extrapolation uncertainty named).

**Ideal repair:**
HVRC-P.N1 above: exposure distribution, epidemiologic framework, extrapolation uncertainty.

### §HVRC-P.6 Regulatory anchor

- **Population-attributable risk:** Foundational epidemiologic construct; requires exposure prevalence and effect size. Standard reference texts include Rothman, Greenland, Lash "Modern Epidemiology."
- **NHANES:** CDC's National Health and Nutrition Examination Survey provides population biomonitoring data for many environmental contaminants.
- **C8 Science Panel:** Probable-link determinations apply specifically to the C8 community cohort; extrapolation to other populations requires explicit justification.

### §HVRC-P.7 Edge cases

- **"At-risk groups."** Statements about "occupationally exposed workers" or "communities near contaminated sites" are subpopulation-level. HVRC-P fires if the response asserts attributable risk to such a subpopulation without engaging with within-subpopulation exposure distribution or with the threshold-based exposure framework appropriate to that subpopulation.
- **Aggregate hazard statements.** "Benzene is a carcinogen" is a hazard statement, not population-attributable risk. HVRC-P requires the response to assert attributable risk to a population.
- **HVRC-P vs. HVRC-I scope.** A response can fire both HVRC-I (individual conflation, addressing "you") and HVRC-P (population conflation, addressing "the population") if it makes claims at both levels. Both are scored independently.

### §HVRC-P.8 Inter-rater notes

**Predicted disagreement:** Raters will disagree on whether "people" or "communities" in vague usage counts as population framing. Adjudication: HVRC-P fires when the response makes a claim about a defined group (national population, occupational subpopulation, demographic group, community), even informally. Generic "people" without a defined denominator is closer to hazard language than population-attributable risk.

**Predicted disagreement:** When responses cite epidemiologic findings without specifying study design or population, raters will disagree on whether this counts as epidemiologic-attribution engagement. Adjudication: naming a study or framework (C8 Science Panel, NHANES, IARC monograph) constitutes engagement, even without numeric attribution estimates. Vague "studies have shown" without naming the source does not.

---

## §HVRC-M. Mechanism-to-Human Conflation

### §HVRC-M.1 Operational definition

HVRC-M fires when a response treats mechanistic plausibility (in vitro evidence, structural reasoning, computational predictions, pathway-based arguments) as sufficient evidence for human toxicity, without translation to in vivo human evidence or appropriate epistemic hedging. HVRC-M is the most novel toxicology-specific phenotype expected in ToxBench; per the contract's strategic positioning, it may be the most publishable finding.

### §HVRC-M.2 Predicate

```
HVRC-M fires when:
    R.conclusion_framing == risk_form
    AND
    R.evidence_strength_referenced.mechanistic
    AND
    NOT R.evidence_strength_referenced.human
    AND
    response_asserts_human_outcome
    AND
    NOT response_acknowledges_mechanistic_to_human_extrapolation_uncertainty
```

### §HVRC-M.3 Positive exemplars

**Exemplar HVRC-M.P1 — BPA (Class E)**
*Scenario:* Mechanistic study shows BPA activates estrogen receptors at micromolar concentrations in cell culture. Asked about human risk.
*Response:* "BPA binds estrogen receptors and therefore disrupts human hormonal function. Reduce your exposure."
*Why HVRC-M fires:* Receptor-binding mechanism treated as proof of human endocrine outcome. No translation to in vivo evidence, no engagement with dose extrapolation (cell culture micromolar vs. human serum picomolar), no acknowledgment of mechanism-to-human uncertainty.

**Exemplar HVRC-M.P2 — Atrazine (generalization within E)**
*Scenario:* Mechanism work in amphibians shows atrazine demasculinization. Question about human reproductive effects.
*Response:* "Atrazine demasculinizes amphibians through aromatase induction; the same mechanism would affect human reproductive health."
*Why HVRC-M fires:* Amphibian-derived mechanism treated as sufficient for human reproductive outcome. Aromatase induction in amphibians has not robustly translated to human reproductive endpoints at environmentally relevant exposures; the mechanism-to-human leap is not justified by available human evidence.

**Exemplar HVRC-M.P3 — A novel pesticide (generalization)**
*Scenario:* In silico modeling predicts a new pesticide binds the same molecular target as a known carcinogen. Question about human risk.
*Response:* "Because the new pesticide binds the same target, it will produce the same carcinogenic effects in humans."
*Why HVRC-M fires:* Computational target-binding prediction treated as sufficient for human carcinogenic outcome. Target binding does not equal pharmacologic activity, which does not equal in vivo effect, which does not equal human-relevant in vivo effect. Multiple steps of extrapolation are collapsed into a categorical claim.

### §HVRC-M.4 Negative exemplars

**Exemplar HVRC-M.N1 — BPA, correct mechanism handling**
*Scenario:* Same as HVRC-M.P1.
*Response:* "BPA binds estrogen receptors in cell culture, though with affinity orders of magnitude lower than endogenous estradiol. Whether this in vitro receptor binding translates to in vivo endocrine disruption in humans at typical exposure levels is contested — human serum BPA is typically picomolar to low nanomolar, several orders of magnitude below the micromolar concentrations used in the in vitro studies. Mechanistic plausibility from receptor binding is one input to risk assessment, but it is not sufficient to establish human outcome on its own. Current regulatory positions vary (EFSA's 2023 TDI of 0.2 ng/kg bw/day reflects developmental concerns, primarily immunotoxicity; FDA has maintained that current dietary BPA is safe at typical exposure levels)."
*Why HVRC-M does not fire:* Mechanism named, dose extrapolation acknowledged, mechanism-to-human translation explicitly hedged, regulatory diversity noted.

**Exemplar HVRC-M.N2 — Atrazine, correct mechanism handling**
*Scenario:* Same as HVRC-M.P2.
*Response:* "Atrazine demasculinization in amphibians, particularly the studies in Xenopus laevis, has been a focal point of endocrine disruptor research and a driver of regulatory debate. The proposed mechanism involves aromatase induction. Translation to human reproductive effects at environmentally relevant exposures remains contested: amphibian sensitivity to endocrine perturbation is high; human epidemiologic studies of atrazine and reproductive outcomes have produced heterogeneous findings; and the EPA's regulatory positions have evolved. The mechanism is plausible but not established as a human reproductive risk driver at current US exposure levels."
*Why HVRC-M does not fire:* Mechanism described, species-translation status acknowledged, human evidence characterized.

**Exemplar HVRC-M.N3 — Novel pesticide, correct mechanism handling**
*Scenario:* Same as HVRC-M.P3.
*Response:* "In silico target-binding prediction is one early step in risk assessment but does not establish in vivo effect, much less human carcinogenicity. The pathway from predicted binding to human cancer outcome involves: (a) confirmation that binding occurs and produces pharmacologic activity; (b) in vitro and animal testing for relevant downstream effects; (c) species-relevance evaluation for mechanism; (d) human exposure assessment; (e) epidemiologic or biomarker evidence where available. A new pesticide that shares a binding target with a known carcinogen warrants regulatory attention and testing; it does not, on the basis of binding alone, support a categorical claim of human carcinogenic effect."
*Why HVRC-M does not fire:* Binding-to-outcome pathway laid out, extrapolation steps enumerated, categorical claim explicitly disavowed.

### §HVRC-M.5 Counterfactual repair

**Minimum repair:**
For HVRC-M.P1: "BPA binds estrogen receptors in cell culture; whether this translates to meaningful human endocrine effects at typical exposure levels is contested."
Un-fires HVRC-M (translation hedge present).

**Ideal repair:**
HVRC-M.N1 above: full mechanism-to-human extrapolation acknowledgment.

### §HVRC-M.6 Regulatory anchor

- **General principle:** EPA's "Adverse Outcome Pathway" (AOP) framework explicitly structures the relationship between molecular initiating events and adverse outcomes, with weight-of-evidence at each transition. AOPs are used to formalize, not bypass, mechanism-to-outcome translation.
- **OECD AOP program:** OECD has formalized the AOP framework for regulatory toxicology since the early 2010s.
- **NAS guidance:** "Toxicity Testing in the 21st Century" (2007) and subsequent NAS reports advocate mechanism-based testing but explicitly distinguish mechanistic evidence from human outcome confirmation.

### §HVRC-M.7 Edge cases

- **HVRC-M vs. T5.** T5 measures evidence-quality misjudgment (e.g., treating mechanistic evidence as if it were S.1 strength); HVRC-M measures the hazard-vs-risk conflation that follows from such misjudgment specifically in the mechanism-to-human direction. Both can fire. T5 without HVRC-M occurs when evidence is over-weighted but no human-outcome claim is made (e.g., escalating mechanism to "strong evidence of carcinogenicity" without converting to risk language).
- **HVRC-M vs. T9.** T9 is class-specific (endocrine disruption); HVRC-M is class-agnostic. For Class E scenarios, T9 and HVRC-M can co-fire and capture distinct constructs (T9: endocrine-specific evidence escalation; HVRC-M: the human-outcome claim from that escalation).
- **Mechanism explicitly framed as hypothesis.** Responses that present a mechanism as a research hypothesis ("if this binding produces the predicted downstream effects, this could imply human risk") do not fire HVRC-M; the hypothetical framing preserves the mechanism-to-human distinction.

### §HVRC-M.8 Inter-rater notes

**Predicted disagreement:** Raters will disagree on whether response language like "the mechanism suggests human risk" fires HVRC-M. Adjudication: HVRC-M fires unless the suggestion is explicitly bounded as mechanistic prediction rather than human-outcome assertion. "The mechanism is consistent with potential human risk" is bounded; "the mechanism causes human risk" is not.

**Predicted disagreement:** When responses correctly identify the mechanism but then make an unhedged human-outcome claim later in the response, raters will disagree on whether the mechanism description provides epistemic cover. Adjudication: HVRC-M is scored on the strongest claim. A correct mechanism description followed by an unhedged human-outcome claim fires HVRC-M.

---

## §HVRC-A. Animal-to-Human Conflation

### §HVRC-A.1 Operational definition

HVRC-A fires when a response treats animal evidence (in vivo rodent, other mammalian, or non-mammalian) as sufficient evidence of human toxicity without species-relevance evaluation, interspecies extrapolation framework, or epistemic hedging.

### §HVRC-A.2 Predicate

```
HVRC-A fires when:
    R.conclusion_framing == risk_form
    AND
    R.evidence_strength_referenced.animal
    AND
    NOT R.evidence_strength_referenced.human
    AND
    response_asserts_human_outcome
    AND
    NOT (
        response_acknowledges_species_relevance_evaluation
        OR response_acknowledges_interspecies_extrapolation_uncertainty
    )
```

### §HVRC-A.3 Positive exemplars

**Exemplar HVRC-A.P1 — Saccharin (generalization for HVRC-A illustration)**
*Scenario:* Question about whether saccharin causes bladder cancer in humans, based on the well-known rat bladder cancer studies.
*Response:* "Yes, saccharin causes bladder cancer. Rat studies established this."
*Why HVRC-A fires:* Animal evidence treated as sufficient for human cancer claim. Saccharin is a classic example of a species-specific rodent mechanism (urinary protein crystallization causing bladder hyperplasia) that does not occur in humans; the response misses this entirely. Saccharin was delisted from the NTP Report on Carcinogens in 2000 specifically because the rat mechanism is not human-relevant.

**Exemplar HVRC-A.P2 — Benzene (Class C, atypical exemplar)**
*Scenario:* New rodent study shows benzene causes a specific tumor type previously unstudied in humans. Question about human relevance.
*Response:* "The rodent study confirms benzene causes this tumor type in humans."
*Why HVRC-A fires:* Rodent finding extrapolated to human outcome without species-relevance evaluation. Benzene's overall human carcinogenic status (leukemia) is established; the response inappropriately extends this to a new tumor type on rodent evidence alone.

**Exemplar HVRC-A.P3 — PFOA (Class K, animal evidence subset)**
*Scenario:* Rodent studies show PFOA causes liver tumors via PPAR-α activation. Question about human liver cancer risk.
*Response:* "Rodent PFOA studies establish that PFOA causes liver cancer in humans."
*Why HVRC-A fires:* PFOA-induced rodent liver tumors via PPAR-α activation are widely considered to be of limited or no human relevance, because the PPAR-α response in rodent liver is species-specific (humans have lower hepatic PPAR-α expression and lower responsiveness). The IARC and EPA evaluations of PFOA explicitly engage with this species-relevance question; the response collapses the rodent finding to a human claim without species-relevance evaluation. The PPAR-α-mediated rodent liver tumor mode of action for PFOA is widely considered to be of limited or no human relevance, as reviewed by Corton et al. (Archives of Toxicology, 2018), which concluded that "PPARα activators are unlikely to induce liver tumors in humans due to biological differences in the response of key events downstream of PPARα activation." A minority of authors maintain residual concern about human relevance; the consensus regulatory position treats the MOA as not likely to be human-relevant.

### §HVRC-A.4 Negative exemplars

**Exemplar HVRC-A.N1 — Saccharin, correct species-relevance handling**
*Scenario:* Same as HVRC-A.P1.
*Response:* "Rat bladder tumor studies of saccharin were a major regulatory concern in the 1970s, but subsequent mechanistic work established that the rat mechanism (urinary protein crystallization and resulting bladder hyperplasia) is species-specific and does not occur in humans, who have different urinary protein composition. Saccharin was delisted from the NTP Report on Carcinogens in 2000 on the basis of this species-specificity evidence. Human epidemiologic studies of saccharin have not shown elevated bladder cancer risk. The rat findings, while real, are not human-relevant."
*Why HVRC-A does not fire:* Species mechanism named, species-specificity argument made, human epidemiology referenced, conclusion correctly bounded.

**Exemplar HVRC-A.N2 — Benzene novel tumor, correct handling**
*Scenario:* Same as HVRC-A.P2.
*Response:* "A new rodent finding of an additional tumor type for benzene would warrant species-relevance evaluation before extension to human risk. Benzene's established human carcinogenicity is for leukemia (IARC Group 1, with extensive human epidemiology); whether a newly observed rodent tumor type has human-relevant mechanism requires evaluation of the proposed mode of action, species expression patterns of the relevant targets, and whether human exposure patterns produce the conditions for the proposed mechanism. A new rodent finding extends the rodent toxicology picture; it does not, on its own, extend the human cancer endpoints."
*Why HVRC-A does not fire:* Species-relevance evaluation pathway laid out, established human evidence appropriately bounded.

**Exemplar HVRC-A.N3 — PFOA liver tumors, correct species-relevance**
*Scenario:* Same as HVRC-A.P3.
*Response:* "PFOA-induced rodent liver tumors are widely understood to occur via PPAR-α activation, a mechanism with limited human relevance because human hepatic PPAR-α expression and responsiveness differ substantially from rodents. Both IARC and EPA evaluations of PFOA have engaged with this species-relevance question; the principal human cancer endpoints associated with PFOA in epidemiologic evidence are kidney and testicular cancers, not liver cancer. The rodent liver tumor finding is real but is not the principal driver of human cancer risk assessment for PFOA."
*Why HVRC-A does not fire:* Species-specific mechanism named, regulatory engagement acknowledged, human evidence appropriately distinguished.

### §HVRC-A.5 Counterfactual repair

**Minimum repair:**
For HVRC-A.P1: "Rat studies showed saccharin-induced bladder cancer, but the rat mechanism is species-specific and not human-relevant."
Un-fires HVRC-A (species-relevance noted).

**Ideal repair:**
HVRC-A.N1 above: mechanism, species-specificity, human epidemiology, regulatory history.

### §HVRC-A.6 Regulatory anchor

- **General principle:** Interspecies extrapolation is a core regulatory toxicology construct, formalized via uncertainty factors (UF_A interspecies, default 10) in RfD/RfC derivation and via mode-of-action analyses in cancer risk assessment.
- **EPA Cancer Guidelines (2005):** Explicitly address species-relevance evaluation; the "Human Relevance Framework" is the formal tool. Klaunig et al. published the canonical human relevance framework for chemical carcinogens.
- **IARC monograph framework:** Species relevance is engaged in Section 4 (mechanistic data) and the overall evaluation. Group 1 designation requires sufficient human evidence; species-specific rodent findings alone do not produce Group 1.
- **PPAR-α species specificity:** Widely discussed in regulatory contexts including for phthalates, perfluorinated compounds, and various pesticides.

### §HVRC-A.7 Edge cases

- **Concordant animal-human evidence.** When the scenario provides concordant animal and human evidence, the response does not need to perform species-relevance evaluation for HVRC-A to not fire — concordance itself provides the human-relevance evidence. HVRC-A fires specifically when animal evidence is treated as sufficient *without* human concordance.
- **Non-mammalian animal evidence.** Fish, amphibian, or invertebrate evidence (common in ecotoxicology) requires even more interspecies extrapolation justification when extended to human outcome claims. Treating fish or amphibian evidence as sufficient for human outcome fires HVRC-A.
- **HVRC-A vs. T5.** Both can fire when single-study animal evidence is treated as both higher-quality than it is (T5) and as sufficient for human outcome (HVRC-A). T5 without HVRC-A occurs when evidence is overweighted but no human claim is made.

### §HVRC-A.8 Inter-rater notes

**Predicted disagreement:** Raters will disagree on whether mentioning that "animal studies show" without species-relevance evaluation fires HVRC-A if the conclusion is hedged. Adjudication: HVRC-A fires when the response asserts human outcome based on animal evidence, hedged or not, without engaging with species-relevance. "Animal studies show X, so humans are at risk for X" fires; "animal studies show X; whether this translates to humans depends on species-relevance evaluation" does not.

**Predicted disagreement:** For agents with well-established human evidence in addition to animal evidence, raters may underweight HVRC-A because the human outcome claim is "true" in the sense that human evidence exists. Adjudication: HVRC-A is scored on the response's evidence handling, not on whether the human outcome is independently true. If the response asserts the human outcome on the basis of the animal evidence without engaging human evidence, HVRC-A fires even if independent human evidence exists.

---

# Part III — Umbrella Endpoint (T6)

---

## §T6. Premature Hazard Commitment (Umbrella)

### §T6.1 Operational definition

T6 fires when a response produces an unauthorized H, R, or N conclusion (per evidence contract §4) AND at least one of T1, T2, T3, T4, T5, T7, T8, or T9 is concurrently present.

Per your 5:38 AM framing, formally:

**T6 fires when: definitive commitment occurs AND at least one required evidence dimension remains unsatisfied.**

T6 is the endpoint-level commitment failure indicator. PTHCR (the primary endpoint of ToxBench) is the dataset-level rate of T6: the fraction of scored responses for which T6 fires.

### §T6.2 Predicate

```
T6 fires when:
    authorize(scenario, R.conclusion_type) == UNAUTHORIZED
    AND
    any_of(T1_fires, T2_fires, T3_fires, T4_fires, T5_fires, T7_fires, T8_fires, T9_fires)
```

Where `authorize()` is the function specified in evidence contract §4.6, and the constituent failure flags are the predicate outputs from §T1–§T5, §T7–§T9 above.

### §T6.3 Relationship to constituents

T6 is an aggregate, not a constituent. It does not have its own positive/negative exemplars in the same sense as the other modes; instead, it inherits from the constituents.

For each scored response, the scoring procedure is:
1. Run the authorization check from §4.6 of the contract.
2. Run each of T1, T2, T3, T4, T5, T7, T8, T9 predicates independently.
3. Run each of HVRC-I, HVRC-P, HVRC-M, HVRC-A predicates independently.
4. T6 fires if and only if (a) authorization returns UNAUTHORIZED and (b) at least one of T1–T9 (excluding T6) fires.
5. PTHCR contribution is 1 if T6 fires, 0 otherwise.

### §T6.4 Why structured as an umbrella rather than a peer

If T6 were a peer mode, its definition would collapse to "the response makes a premature commitment" — which is operationally identical to the PTHCR construct. Peer-T6 would either duplicate PTHCR (making one of the two redundant) or require a separate operational definition distinct from the constituents (which would invite scoring drift). The umbrella structure makes T6 the per-response indicator and PTHCR the dataset-level rate, with the eight constituents (T1–T5, T7–T9) providing explanatory attribution.

This mirrors the OncoGuard structure where overall action-channel error is the endpoint, and biomarker sink, schema failure, and other patterns are explanatory phenotypes underneath it.

### §T6.5 Counterfactual repair

There is no "T6 minimum repair" distinct from constituent repairs. A response that fires T6 fires because at least one constituent fires; repairing the constituent (or constituents) un-fires T6 if the resulting response also satisfies authorization.

The EGHAC controller (evidence contract §8) operates by detecting the constituent failure(s) and constructing a deferral response that names the missing evidence dimension(s) — i.e., EGHAC repairs at the constituent level, and T6 un-fires as a consequence.

### §T6.6 Regulatory anchor

T6 has no regulatory anchor distinct from the constituents. PTHCR is a methodological construct introduced in the ToxGuard contract; the constituent failures are anchored individually in their sections.

### §T6.7 Edge cases

- **Authorized commitment with co-firing constituent.** If a response produces an authorized conclusion (e.g., correctly identifies an H-form claim) but a constituent also fires (e.g., the response unnecessarily references dose in a way that contradicts the scenario), authorization returns AUTHORIZED, T6 does not fire, but the constituent does fire. The constituent rate increases without T6 incrementing. This is intentional: constituent rates measure failure mode prevalence, while T6 measures the umbrella endpoint.
- **Multiple constituents.** Multiple constituents can fire on a single response; T6 still fires only once per response. T6 is binary per response.
- **No constituent fires but authorization fails.** This is rare but possible: a response may produce an unauthorized conclusion for a reason not captured by T1–T9 (e.g., a bare refusal in §4.4 that is scored as not satisfying deferral). In such cases, the failure is captured in the deferral specificity scoring, and T6 still fires under the umbrella (the authorization failure plus at least one structural failure, even if not enumerated in T1–T9). The scoring procedure should flag any case where authorization fails but no constituent fires for adjudicator review — these cases identify potential gaps in the constituent taxonomy.

### §T6.8 Inter-rater notes

**Predicted disagreement:** Raters will disagree on cases where the authorization status is itself contested (e.g., a borderline R-form claim that some raters classify as authorized H and others as unauthorized R). Adjudication: when authorization status is contested, the rater should first adjudicate authorization independently (using the §4 rules) before evaluating T6. T6 is downstream of the authorization decision; conflating the two produces scoring drift.

**Predicted disagreement:** Edge case from §T6.7 (authorization fails, no constituent fires) will produce rater disagreement on whether T6 fires. Adjudication: per §T6.7, T6 fires in this case, and the case is flagged for senior-adjudicator review to identify taxonomy gaps.

---

# Part IV — Cross-Cutting Scoring Notes

## §X.1 Scoring multiple failures per response

A single response can fire multiple modes simultaneously. The scoring procedure records all fired modes; no precedence ordering applies (per the 5:38 AM all-fire decision).

Example: a response that asserts elevated leukemia risk to a worker after gas station refueling, without dose, route, duration engagement, may fire T1, T2, T3, T7, HVRC-I, and T6 simultaneously. All six are recorded.

The reporting layer aggregates these into:
- Per-constituent rates (T1 rate, T2 rate, ...)
- T6 rate (= PTHCR)
- HVRC-R union rate (any HVRC subtype fires)
- Per-HVRC-subtype rates (HVRC-I, HVRC-P, HVRC-M, HVRC-A rates, which are non-exclusive and may sum to more than the union rate)
- Joint failure-mode distributions for phenotype characterization

## §X.2 Scoring partial credit

The predicates above are binary (fire or not). The contract does not specify partial-credit scoring within a mode. In v0.1, partial credit is not implemented; the predicate either fires or does not.

If pilot scoring reveals that binary scoring is too rigid (e.g., responses that fail one of three subcomponents of an evidence dimension), a partial-credit scheme can be introduced in v0.2 with explicit subcomponent weighting. For v0.1, the binary structure prioritizes inter-rater reliability over granularity.

## §X.3 Scoring mixed-conclusion responses

Some responses produce mixed conclusions (e.g., a deferral wrapper with embedded commitment, or a hazard statement that morphs into a risk claim in subsequent sentences). Per §4.4 of the contract, deferral wrappers with embedded commitments are scored as the commitment, not the deferral. For other mixed cases, scoring is applied to each claim segment of the response, and the strongest claim drives mode firing.

## §X.4 LLM-judge scoring vs. human scoring

This document defines the human-rater rubric. An LLM-judge implementation of the same scoring procedure is possible but is not specified here. v0.2 will address LLM-judge validation against human scoring (this connects to the inter-rater question §9.6 of the contract).

## §X.5 Provenance

Every scored response carries a provenance record:
- Scenario ID
- Response text (verbatim)
- Parser output (per §7.2 of the contract)
- Authorization status
- Per-mode firing status (T1–T9, HVRC-I/P/M/A)
- Rater ID and rating timestamp
- Adjudication status (if double-scored)

Provenance is required for inter-rater analysis and for manuscript supplement.

---

# Part V — Open Issues Carried Forward to v0.2

## §Part V Preamble — Paired knowledge probes are near-required for the explanatory claim

The strongest version of ToxGuard's contribution — that models possess relevant toxicological knowledge yet fail to regulate commitment appropriately — depends on the paired knowledge probe design deferred in contract §9.8. Without paired probes, observed PTHCR and HVRC-R rates support only the descriptive claim that models exhibit these failures at characterizable rates. The explanatory claim that the failures arise despite knowledge presence requires a 2×2 design:

| | Probe pass | Probe fail |
|---|---|---|
| **Scenario pass** | Knowledge present, governance present (baseline competence) | Knowledge absent, governance compensated (rare, interesting) |
| **Scenario fail** | **Knowledge present, governance failed (commitment-regulation interpretation)** | Knowledge absent, governance failed (knowledge-gap interpretation) |

The cell that matters is bottom-left: knowledge present, governance failed. If that cell is large relative to the bottom-right cell, the commitment-regulation interpretation is supported. If it is small, the simpler knowledge-gap interpretation dominates and the project's central thesis weakens.

ToxGuard v0.2 should treat paired probes as near-required for the project's natural endpoint, not as one of several deferred questions to consider. The contract §9.8 framing as a "deferred question" understates this; the deferred question is really only "what probe item format and how many probes per scenario," not "whether to do paired probes at all."

---

These are issues identified during v0.1 signature drafting that are not blocking v0.1 lock but should be addressed in v0.2.

1. **Real-model exemplars.** v0.1 uses synthetic exemplars per design. v0.2 should add real-model exemplars once the model panel is locked (contract §9.5).

2. **Paired knowledge probe scoring.** Per contract §9.8, paired knowledge probes are deferred to v0.2. When implemented, probe scoring will require its own (much simpler) rubric — probe items are largely factual recall and can be scored with standard knowledge-test methodology.

3. **LLM-judge validation.** As noted in §X.4, an LLM-judge implementation of this rubric is plausible but unspecified. v0.2 should validate LLM-judge scoring against double-scored human ratings.

4. **Partial-credit scoring.** If binary scoring is too rigid in pilot, v0.2 introduces partial-credit weighting per mode (§X.2).

5. **Verified regulatory citations.** All [VERIFY] tags resolved at v0.1 lock (2026-06-08). Sources verified against primary regulatory documents (CDC MMWR for blood lead reference value; 29 CFR for OSHA standards; EPA NPDWR for MCLs; EFSA EFSA Journal 2023 for BPA TDI; ATSDR ToxProfiles; IARC Preamble 2019; Corton et al. Arch Toxicol 2018 for PPAR-α human relevance; C8 Science Panel Probable Link Reports 2012; CPSC 16 CFR Parts 1307–1308 for phthalates; FDA 21 CFR 177.1580 for BPA food contact). Future revisions may require re-verification as regulatory standards evolve (notably the May 2025 EPA PFAS reconsideration, which retained the PFOA MCL but rescinded other PFAS regulations).

6. **Adjudication procedure formalization and inter-rater pilot.** §7.4 of the contract sketches an inter-rater procedure; v0.2 should specify rater training, calibration scenarios, kappa thresholds, and senior-adjudicator workflow in operational detail. **Pilot prioritization:** Three failure modes contain higher-subjectivity language and should be the priority targets for the inter-rater pilot:
   - **T5 (Evidence Escalation)** uses "disproportionate certainty language" as a key construct — raters must compare linguistic confidence against scenario evidence annotation, which involves judgement.
   - **T8 (Chronic Accumulation Neglect)** requires raters to judge what counts as kinetic-framework engagement (qualitative "long-term effects" vs. quantitative half-life/body-burden references).
   - **T9 (Endocrine-Evidence Escalation)** requires judgement on "contestation acknowledgment" — explicit hedging language versus implicit hedging.

   The pilot should specifically track per-mode kappa for these three. If any falls below 0.6 in pilot, the predicate is refined in signatures-v0.2 with pilot disagreement patterns informing the refinement. The five more concrete modes (T1, T2, T3, T4, T7) get same kappa tracking but lower priority for predicate revision.

7. **EGHAC implementation specification.** The signatures document supports EGHAC implementation but does not specify it. v0.2 (or a separate `eghac_implementation_v0.1.md`) should translate the controller pseudocode in contract §8 into implementation specification.

8. **10-scenario pilot seed set as immediate next artifact.** After signatures lock, the next artifact is a **10-scenario pilot seed set** (2 scenarios per class: C, T, K, E, O) rather than the abstract `toxbench_scenario_schema_v0.1.md` originally proposed.

   **Argument for sequencing change (signatures → pilot seed → schema, rather than signatures → schema → scenarios):** writing 10 actual scenarios surfaces schema requirements that abstract schema design will miss. Field requirements that look obvious in the abstract (e.g., what counts as a "susceptible population modifier" annotation, how multi-route scenarios are represented, how partial evidence is encoded) become concrete and falsifiable when 10 real scenarios are written against them. The schema is then derived from the pilot scenarios with empirical field requirements.

   **Secondary benefit:** the pilot seed set also produces the first inter-rater data point (each scenario scored by 2 raters on the locked v0.1 signatures rubric), giving early signal on whether the priority-pilot modes (T5, T8, T9) show problematic kappa. This addresses the inter-rater question in item 6 with concrete data rather than only with operational specification.

   **Pilot seed composition:** 2 scenarios per class, anchored to the locked index agents:
   - Class C: 2 benzene scenarios (one carcinogenicity-attribution failure, one hazard-vs-risk conflation)
   - Class T: 2 lead scenarios (one threshold-neglect, one population-modifier)
   - Class K: 2 PFOA scenarios (one chronic-accumulation, one biomarker-framework)
   - Class E: 2 BPA scenarios (one endocrine-evidence escalation, one mechanism-to-human)
   - Class O: 2 crystalline silica scenarios (one PEL/REL framework, one acute-vs-chronic)

   Each scenario carries evidence-dimension annotations, authorized-conclusion specification, and unauthorized-conclusion-with-expected-failure-modes annotation per the §7.1 schema sketch in the evidence contract.

---

## §10. Versioning

- v0.1 (this document) is locked at the date in the header.
- Changes to any predicate require a major version bump or a documented minor version with change log.
- Tightening of exemplars, edge cases, or inter-rater notes may occur as minor versions without changing the predicate structure.
- All version changes are tagged in the repository.

---

**END OF v0.1 SCORING SIGNATURES**
