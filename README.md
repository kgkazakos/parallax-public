# Parallax — Co-Reasoning Research Canvas

<p align="center">
  <em>A spatial canvas where AI reasoning models critique qualitative research clustering</em>
</p>

<p align="center">
  <a href="#architecture">Architecture</a> ·
  <a href="#research-design">Research Design</a> ·
  <a href="#key-findings">Key Findings</a> ·
  <a href="#related-projects">Related Projects</a>
</p>

---

## The Problem

Qualitative researchers face a persistent challenge during sensemaking: **confirmation bias**. When clustering interview excerpts, survey responses, or user feedback into themes, researchers tend to see patterns that confirm their initial hypotheses and dismiss outliers as noise.

Standard LLMs can generate alternative groupings, but they pattern-match from training data — they don't *reason through* why a grouping might be wrong. When a researcher asks "challenge my clustering," the LLM produces plausible-sounding alternatives without transparent deliberation.

**Parallax** asks: does a *reasoning model* — one that shows its chain-of-thought — produce meaningfully better critique than a standard LLM? And does seeing the AI's reasoning process change how researchers engage with that critique?

## What Parallax Does

Parallax is a spatial research canvas where:

1. **Researchers upload qualitative data** (feedback, quotes, observations) and cluster items into themes by dragging cards on a canvas
2. **An AI critique layer** reviews the clustering and identifies gaps, biases, misplacements, and alternative groupings
3. **The critique is structured** into cluster assessments, item-level move suggestions, and outlier/gap detection
4. **In the reasoning condition**, a collapsible trace shows the AI's deliberation — what it considered, what alternatives it explored, and why it landed on its assessment

The key experimental manipulation: the same critique task is performed by a standard LLM (Tier 1) and a reasoning model (Tier 2), compared in a between-subjects RCT.

## Architecture

```
┌──────────────────────────────────────────────────┐
│  Spatial Canvas (Next.js + React Flow)           │
│                                                  │
│  ┌──────────┐  ┌───────────┐  ┌──────────────┐  │
│  │ Data     │  │ Clustering│  │ Critique     │  │
│  │ Upload   │→ │ Canvas    │→ │ Panel +      │  │
│  │ (CSV)    │  │ (drag &   │  │ Reasoning    │  │
│  │          │  │  drop)    │  │ Trace        │  │
│  └──────────┘  └───────────┘  └──────┬───────┘  │
│                                      │          │
└──────────────────────────────────────┼──────────┘
                                       │
                              POST /critique
                                       │
┌──────────────────────────────────────┼──────────┐
│  Critique Engine (Python FastAPI)    │          │
│                                      │          │
│  ┌───────────────────────────────────┴───┐      │
│  │        LLM-Agnostic Adapter           │      │
│  │   generate(prompt, system) -> str     │      │
│  └───┬──────────────┬──────────────┬─────┘      │
│      │              │              │            │
│  ┌───┴───┐    ┌─────┴────┐   ┌────┴───┐        │
│  │Tier 1 │    │ Tier 2   │   │ Alt    │        │
│  │Std LLM│    │ Reasoning│   │Provider│        │
│  └───────┘    └──────────┘   └────────┘        │
│                                                 │
│  Env-driven provider selection:                 │
│  LLM_PROVIDER · REASONING_PROVIDER              │
└─────────────────────────────────────────────────┘
```

### Design Principles

- **LLM-agnostic:** Both tiers use the same adapter interface (`generate(prompt, system) -> str`). Providers are swapped via environment variables, not code changes. Consistent with P3 and P4 architecture.
- **Identical prompts:** The critique prompt is the same for both tiers. The only experimental manipulation is the model and whether extended thinking is enabled.
- **Full interaction logging:** Every card movement, cluster creation, critique request, and suggestion accept/reject is timestamped and recorded for the RCT analysis.

### Technology Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 15, React Flow, Zustand, TypeScript |
| Backend | Python, FastAPI, Pydantic |
| Tier 1 (Standard) | Gemini 2.5 Flash (default), swappable |
| Tier 2 (Reasoning) | Claude Sonnet 4.5 extended thinking (default), swappable |
| Storage | Supabase |
| Deployment | Vercel (frontend), Google Cloud Run (backend) |

## Research Design

### RCT: 3-Arm Between-Subjects (N=20)

| Condition | AI Critique | Reasoning Trace |
|-----------|-------------|-----------------|
| Control | None | None |
| Treatment A | Standard LLM | Hidden |
| Treatment B | Reasoning Model | Visible |

### Task

All participants receive the same dataset of ~50 user feedback items about a fictional product. Items are pre-constructed to include:
- 4–5 ground-truth clusters (established by expert consensus)
- 6–8 intentionally ambiguous items (plausible in multiple clusters)
- 3–4 genuine outliers representing a non-obvious theme

### Measures

**Primary:**
- **Insight quality:** Blind expert ratings (1–10), 3 independent raters, Krippendorff's α > 0.70
- **Confirmation bias reduction:** Divergence between pre-task hypotheses and final clusters
- **Outlier preservation:** Did the participant surface the non-obvious outlier theme?

**Secondary:**
- Time to insight
- Trust in AI critique (Likert 1–7)
- Perceived collaboration
- Cognitive load (NASA-TLX)

### Procedure (per participant, ~90 min)

1. Pre-task survey: demographics, research experience, expected themes
2. Training: canvas tutorial (all) + AI critique tutorial (A/B)
3. Task: 45 min clustering + critique + finalization
4. Post-task survey: trust, satisfaction, NASA-TLX
5. Semi-structured interview: 20 min

### Analysis Plan

- One-way ANOVA across three conditions (each primary measure)
- Planned contrasts: A vs. B (key comparison), Control vs. pooled treatment
- Effect sizes: η² (ANOVA), Cohen's d (pairwise)
- Qualitative: reflexive thematic analysis of interviews

**Target venue:** CHI 2027

## Canvas Interaction Flow

```
Upload Data → Manual Clustering → [Request Critique] → Review Critique → Accept/Reject → Finalize
     │              │                     │                   │                │
     │              │                     │                   │                │
  All conditions  All conditions    A/B only            A/B only       All conditions
                                         │
                                    ┌────┴────┐
                                    │Treatment│
                                    │   B:    │
                                    │Reasoning│
                                    │ trace   │
                                    │ visible │
                                    └─────────┘
```

### Critique Output Structure

The AI returns structured critique at three levels:

- **Cluster assessment:** Each cluster rated as coherent / mixed / split-recommended, with confidence score and explanation
- **Item suggestions:** Specific items that may fit better in a different cluster, with rationale
- **Gap analysis:** Outliers that don't fit any cluster, missing themes, and cluster overlaps

In Treatment B, each critique element includes the reasoning trace — the chain of deliberation the model went through before reaching its assessment.

## Key Findings

*Study in progress — results expected August 2026.*

## Related Projects

This is **Project 5** of the Computational Product Research (CPR) portfolio:

| # | Project | What It Does | Status |
|---|---------|-------------|--------|
| 1 | [CausalTrack](https://github.com/kgkazakos/causaltrack) | Say-Do Gap detection via knowledge graphs | ✅ Complete |
| 2 | [Synthetic User Council](https://github.com/kgkazakos/synthetic-user-council) | Multi-agent synthetic personas debating researcher assumptions | ✅ Complete |
| 3 | [Cognitive Load Monitor](https://github.com/kgkazakos/clm) | Telemetry-based cognitive load interpretation | ✅ Complete |
| 4 | [ResearchConductor](https://github.com/kgkazakos/research-conductor-public) | Autonomous research agent with safety gates | ✅ Complete |
| **5** | **Parallax** | **Co-reasoning canvas for qualitative sensemaking** | **🔨 Building** |
| 6 | Constitutional Research AI | Research methodology encoded as constitutional principles | Planned |
| 7 | UXR-Arena | Living benchmark for research AI tools | Planned |

**CPR thesis:** *"Computational Product Research is the bridge from static insight to agentic intelligence. We are moving from studying users to building systems that co-evolve with them."*

## Author

**Kostas Kazakos, PhD**
HCI, University of Melbourne
[GitHub](https://github.com/kgkazakos) · [LinkedIn](https://linkedin.com/in/kgkazakos)

## License

MIT — see [LICENSE](LICENSE).

## Disclaimer

This is a personal research project. See [DISCLAIMER.md](DISCLAIMER.md).
