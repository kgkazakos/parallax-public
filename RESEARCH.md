# Research Design — Parallax

## Research Question

**RQ1:** Does a reasoning model (with chain-of-thought) produce higher-quality critique of qualitative research clustering compared to a standard LLM?

**RQ2:** Does visibility of the AI's reasoning trace affect researcher trust and critique utilisation?

**RQ3:** Does AI-assisted critique reduce confirmation bias in qualitative sensemaking?

## Study Design

### Design

3-arm between-subjects randomised controlled trial.

| Condition | n | AI Critique | Model Tier | Reasoning Trace |
|-----------|---|-------------|------------|-----------------|
| Control | ~7 | None | — | — |
| Treatment A | ~7 | Yes | Standard LLM (Tier 1) | Hidden |
| Treatment B | ~6 | Yes | Reasoning Model (Tier 2) | Visible |

Participants are randomly assigned with stratification by research experience (junior: <3 years, senior: 3+ years) to ensure balance across conditions.

### Participants

- **N = 20** product/UX researchers
- Recruited from professional networks
- Mix of experience levels and research domains
- Compensation: gift cards or co-authorship consideration

### Power Analysis

N=20 is adequate for detecting large effects (Cohen's d > 0.8) with α=0.05 and power=0.80 in pairwise comparisons. This is acknowledged as a limitation — the study is framed as an initial comparative investigation, not a definitive trial. Effect size estimates from this study will inform power calculations for future replications.

## Task

All participants receive the same dataset of ~50 user feedback items about a fictional B2B SaaS product. The dataset is pre-constructed with known structure:

- **4–5 ground-truth clusters** (established by 3 expert raters before the study)
- **6–8 ambiguous items** that are plausibly assigned to multiple clusters
- **3–4 genuine outliers** that represent a real but non-obvious theme

The dataset is designed to:
- Be domain-general (any researcher can engage regardless of specialisation)
- Contain surface-level keyword traps (e.g., "cost" appearing in non-pricing contexts)
- Have a discoverable outlier theme that requires looking beyond obvious patterns
- Be completable in ~45 minutes

## Procedure

Each participant session lasts approximately 90 minutes:

| Phase | Duration | All | A only | B only |
|-------|----------|-----|--------|--------|
| 1. Pre-task survey | 5 min | Demographics, experience, expected themes | — | — |
| 2. Training | 10 min | Canvas tutorial | + AI critique tutorial | + AI critique tutorial |
| 3. Task | 45 min | Cluster 50 items, label clusters, write findings | + Request and respond to critique | + Request and respond to critique (with reasoning trace) |
| 4. Post-task survey | 10 min | Trust, satisfaction, NASA-TLX | — | — |
| 5. Interview | 20 min | Process, decision-making | + Role of AI critique | + Impact of reasoning trace |

### Training

All participants receive the same 5-minute tutorial on the canvas interface using a separate small practice dataset (10 items, 2 obvious clusters). Treatment A and B participants receive an additional 5-minute tutorial on the critique feature, using the same practice dataset. Treatment B participants are shown how to expand the reasoning trace.

### Blinding

- **Single-blind between A and B:** Participants do not know which model generated the critique
- **Control participants:** Know they are in a no-AI condition (unavoidable)
- **Expert raters:** Fully blind to condition — they evaluate final clusters and summaries only

## Measures

### Primary

**1. Insight Quality (1–10)**
Three independent expert raters evaluate each participant's final clustering on a rubric:

| Dimension | Weight | Criteria |
|-----------|--------|----------|
| Thematic coherence | 30% | Do items within each cluster share a genuine theme? |
| Evidence grounding | 25% | Are cluster labels justified by the item content? |
| Completeness | 25% | Are all items meaningfully placed? Are outlier themes surfaced? |
| Nuance | 20% | Does the clustering distinguish between superficially similar but distinct themes? |

Inter-rater reliability target: Krippendorff's α > 0.70.

**2. Confirmation Bias Reduction**
Operationalised as:
- **Hypothesis divergence:** Cosine distance between the themes listed in the pre-task survey and the final cluster labels. Higher divergence = more willingness to revise initial assumptions.
- **Counter-evidence consideration:** Proportion of items initially hypothesised to belong to one theme that ended up in a different cluster.
- **Outlier preservation rate:** Binary — did the participant create a cluster for the non-obvious outlier theme?

**3. Time to Insight**
Total minutes from task start to "Finalize" button click.

### Secondary

- **Trust in AI critique:** 4-item Likert scale (1–7), adapted from Hoffman et al. (2018) trust in automation
- **Perceived collaboration:** 3-item Likert scale (1–7)
- **Satisfaction:** Single-item overall satisfaction (1–7)
- **Cognitive load:** NASA-TLX (6 subscales)

### Behavioural (from interaction logs)

- Number of card movements before and after critique
- Time spent reviewing critique panel
- Suggestion acceptance rate
- Time spent viewing reasoning trace (Treatment B)
- Number of clusters created/deleted/renamed

## Analysis Plan

### Quantitative

1. **One-way ANOVA** for each primary measure across three conditions
2. **Planned contrasts:**
   - A vs. B (key comparison: does reasoning improve critique quality?)
   - Control vs. pooled treatment (does any AI help?)
3. **Effect sizes:** η² for ANOVA, Cohen's d for pairwise comparisons
4. **95% confidence intervals** for all point estimates
5. **Non-parametric alternatives** (Kruskal-Wallis) if normality assumptions are violated

### Qualitative

- **Reflexive thematic analysis** (Braun & Clarke, 2006) of interview transcripts
- Focus on: how reasoning traces affected trust, when and why participants overrode AI, moments of insight shift
- Member checking with 3–5 participants

### Pre-registration

Analysis plan to be pre-registered on OSF before data collection begins.

## Ethical Considerations

- All data anonymised before analysis
- Participants provide informed consent
- Participation is voluntary; withdrawal permitted at any time without consequence
- No deception is involved (participants know they are in a study of AI-assisted research tools)
- Personal project disclaimer: this study is not conducted under any employer affiliation

## Timeline

| Phase | Dates |
|-------|-------|
| Tool development | Jun 14 – Jul 14, 2026 |
| Pilot testing (n=3–5) | Jul 7–14, 2026 |
| Participant recruitment | Jun 14 – Jul 14, 2026 |
| Study execution (n=20) | Jul 15 – Aug 10, 2026 |
| Expert rating | Aug 4–10, 2026 |
| Quantitative analysis | Aug 10–20, 2026 |
| Qualitative analysis | Aug 15–25, 2026 |
| Paper writing | Aug 10 – Sep 10, 2026 |
| **CHI 2027 submission** | **Sep 10, 2026** |

## References

- Braun, V., & Clarke, V. (2006). Using thematic analysis in psychology. *Qualitative Research in Psychology, 3*(2), 77–101.
- Hoffman, R. R., et al. (2018). Metrics for explainable AI: Challenges and prospects. *arXiv:1812.04608*.
- Hart, S. G. (2006). NASA-Task Load Index (NASA-TLX); 20 years later. *Proceedings of the Human Factors and Ergonomics Society Annual Meeting*.
