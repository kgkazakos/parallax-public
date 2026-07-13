# Parallax — Co-Reasoning Research Canvas

<p align="center">
  <em>A spatial research canvas that compares reasoning models against standard LLMs for critiquing qualitative clustering, making the AI's chain-of-thought visible to researchers so they can see why the AI disagrees — not just that it does.</em>
</p>

<p align="center">
  <a href="https://parallax-69yd.vercel.app/">Live Demo</a> ·
  <a href="#architecture">Architecture</a> ·
  <a href="#research-design">Research Design</a> ·
  <a href="#key-findings">Key Findings</a> ·
  <a href="#related-projects">Related Projects</a>
</p>

---

## The Problem

Qualitative researchers face a persistent challenge during sensemaking: **confirmation bias**. When clustering interview excerpts, survey responses, or user feedback into themes, researchers tend to see patterns that confirm their initial hypotheses and dismiss outliers as noise.

Standard LLMs can generate alternative groupings, but they pattern-match from training data — they don't *reason through* why a grouping might be wrong. When a researcher asks "challenge my clustering," the LLM produces plausible-sounding alternatives without transparent deliberation.

Commercial tools — Dovetail, Notably, Atlas.ti, NVivo, MAXQDA, Delve, Condens, Marvin, Looppanel, Thematic — all follow the same paradigm: AI generates codes and themes, researcher reviews and accepts. They optimize for premature closure. This undermines qualitative reflexivity and exacerbates automation bias: the "Automation Paradigm."

**Parallax asks:** does a *reasoning model* — one that shows its chain-of-thought — produce meaningfully better critique than a standard LLM? And does seeing the AI's reasoning process act as a cognitive forcing function, or does its authoritative verbosity create an illusion of rigor?

## What Parallax Does

Parallax is a spatial research canvas where:

1. **Researchers upload qualitative data** (feedback, quotes, observations) and cluster items into themes by dragging cards on a canvas
2. **An AI critique layer** reviews the clustering and identifies gaps, biases, misplacements, and alternative groupings
3. **The critique is structured** into cluster assessments, item-level move suggestions, and outlier/gap detection
4. **In the reasoning condition**, a collapsible trace shows the AI's deliberation — what it considered, what alternatives it explored, and why it landed on its assessment

Parallax is positioned as an **"Epistemic Partner"** — it intentionally introduces friction by slowing the researcher down and exposing the messy deliberative process of critique, rather than optimizing for speed and clean outputs.

**[Try the live demo →](https://parallax-69yd.vercel.app/)**

## Architecture

```
┌──────────────────────────────────────────────────┐
│  Spatial Canvas (Next.js 15 + React Flow)        │
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

- **LLM-agnostic:** Both tiers use the same adapter interface (`generate(prompt, system) -> str`). Providers are swapped via environment variables, not code changes. Consistent with the CPR portfolio architecture (P3, P4).
- **Identical prompts:** The critique prompt is the same for both tiers. The only experimental manipulation is the model and whether extended thinking is enabled.
- **Pull-based critique:** The researcher explicitly requests critique — it is never pushed. This preserves researcher agency and avoids the confound of AI interruption frequency.
- **Full interaction logging:** Every card movement, cluster creation, critique request, and suggestion accept/reject is timestamped and recorded for the RCT analysis.

### Technology Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js 15, React Flow, Zustand, TypeScript |
| Backend | Python, FastAPI, Pydantic |
| Tier 1 (Standard) | Gemini 3.1 Pro (default), swappable |
| Tier 2 (Reasoning) | Claude Opus 4.5 extended thinking (default), swappable |
| Deployment | Vercel (frontend), Google Cloud Run (backend) |

## Research Design

### Core Research Questions

**RQ1:** Does a reasoning model produce higher-quality critique of qualitative clustering compared to a standard LLM?

**RQ2:** Does visibility of the AI's reasoning trace affect researcher trust and critique utilisation?

**RQ3:** Does AI-assisted critique reduce confirmation bias in qualitative sensemaking?

### 4-Arm 2×2 Factorial RCT (N=28)

| Arm | AI Critique | Model | Reasoning Trace | Isolates |
|-----|-------------|-------|-----------------|----------|
| 1. Control | None | — | — | Baseline |
| 2. Standard LLM | Yes | Gemini 3.1 Pro | — | AI effect |
| 3. Reasoning (hidden) | Yes | Claude Opus 4.5 | Hidden | Model capability |
| 4. Reasoning (visible) | Yes | Claude Opus 4.5 | Visible | Transparency effect |

**Arms 3 vs 4** isolates the transparency effect: does *seeing* the reasoning trace change how researchers engage with critique?

**Arms 2 vs 3** isolates the model capability effect: does a reasoning model produce fundamentally better critique than a standard LLM, independent of UI?

This 2×2 design resolves a critical confound identified in the literature: without Arm 3, any improvement in Arm 4 could be attributed to either model capability or trace visibility — making the contribution unpublishable.

### Central Tension (Buçinca et al., CHI 2021)

Does the extended reasoning trace act as a **cognitive forcing function** (disrupting premature sensemaking closure and reducing bias) or does its authoritative verbosity create an **illusion of rigor** (making researchers overly deferential)?

### Task

50 pre-constructed user feedback items about a fictional product, including:
- 4–5 ground-truth clusters (established by expert consensus)
- 6–8 intentionally ambiguous items (plausible in multiple clusters)
- Keyword traps (e.g., "costs too much time" ≠ pricing)
- 3–4 genuine outliers representing a non-obvious hidden theme

### Measures

**Primary:**
- Insight quality: blind expert ratings (1–10), 3 independent raters, Krippendorff's α > 0.70
- Confirmation bias reduction: divergence between pre-task hypotheses and final clusters
- Outlier preservation: did the participant surface the hidden outlier theme?

**Secondary:**
- Trust in AI critique (Likert 1–7)
- Perceived collaboration (Likert 1–7)
- Cognitive load (NASA-TLX)
- Time to insight

### Analysis Plan
- 2×2 factorial ANOVA for each primary measure
- Planned contrasts: Arm 3 vs 4 (transparency), Arm 2 vs 3 (model capability)
- Effect sizes: η² (ANOVA), Cohen's d (pairwise)
- Qualitative: reflexive thematic analysis of semi-structured interviews

## Canvas Interaction Flow

```
Upload Data → Manual Clustering → [Request Critique] → Review Critique → Accept/Reject → Finalize
     │              │                     │                   │                │           │
  All arms       All arms           Arms 2-4 only        Arms 2-4         All arms    All arms
                                         │
                                    ┌────┴────┐
                                    │ Arm 4:  │
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

In Arm 4, each critique element includes the reasoning trace — the chain of deliberation the model went through before reaching its assessment.

## Study Infrastructure

The complete study flow is built into the application:

1. **Welcome** — Participant ID entry, condition assignment
2. **Informed consent** — Study description, data collection notice, voluntary participation
3. **Pre-task survey** — Demographics, research experience, AI familiarity, expected themes (confirmation bias baseline)
4. **Task** — 50-item dataset auto-loads on the canvas. Participants cluster, request critique (Arms 2-4), and finalize with a findings summary
5. **Post-task survey** — Condition-aware: trust scales (Arms 2-4 only), reasoning trace question (Arm 4 only), NASA-TLX (all arms)
6. **Session export** — Complete interaction log, surveys, timing, and clustering state exported as JSON

Session state auto-saves to prevent data loss from accidental page refresh.

## Key Findings

*Study in progress — results will be published here after analysis.*

## Related Projects

This is **Project 5** of the Computational Product Research (CPR) portfolio:

| # | Project | What It Does | Status |
|---|---------|-------------|--------|
| 1 | [CausalTrack](https://github.com/kgkazakos/causaltrack) | Say-Do Gap detection via knowledge graphs | ✅ Complete |
| 2 | [Synthetic User Council](https://github.com/kgkazakos/synthetic-user-council) | Multi-agent synthetic personas debating researcher assumptions | ✅ Complete |
| 3 | [Cognitive Load Monitor](https://github.com/kgkazakos/clm) | Telemetry-based cognitive load interpretation | ✅ Complete |
| 4 | [ResearchConductor](https://github.com/kgkazakos/research-conductor-public) | Autonomous research agent with safety gates | ✅ Complete |
| **5** | **[Parallax](https://github.com/kgkazakos/parallax-public)** | **Co-reasoning canvas for qualitative sensemaking** | **✅ Complete** |
| 6 | Constitutional Research AI | Research methodology encoded as constitutional principles | Planned |
| 7 | UXR-Arena | Living benchmark for research AI tools | Planned |

**CPR thesis:** *"Computational Product Research is the bridge from static insight to agentic intelligence. We are moving from studying users to building systems that co-evolve with them."*

## Author

**Kostas Kazakos, PhD**
HCI, University of Melbourne

## License

MIT — see [LICENSE](LICENSE).

## Disclaimer

This is a personal research project. See [DISCLAIMER.md](DISCLAIMER.md).
