# Architecture — Parallax

## System Overview

Parallax separates concerns into three layers: the spatial canvas (user interaction), the critique engine (AI analysis), and the study infrastructure (data collection for the RCT).

### Layer 1: Spatial Canvas

The canvas is built on React Flow, a library for node-based interfaces. Each qualitative data item is rendered as a draggable card node. Researchers create clusters by selecting multiple cards and assigning a label. The canvas tracks all spatial positions, cluster assignments, and interaction events.

State management uses Zustand, with a single store holding items, clusters, node positions, and the full interaction log. Every user action generates a timestamped log entry — this interaction log is the primary data source for the RCT behavioral analysis.

### Layer 2: Critique Engine

The critique engine receives a snapshot of the current canvas state (items, clusters, positions) and returns structured critique. The engine is the experimental intervention — the only thing that changes between Treatment A and Treatment B is the model used.

The adapter interface is provider-agnostic:

```
Interface: generate(prompt, system, thinking_budget?) -> str
```

The critique prompt is identical across conditions. It instructs the model to:
1. Assess each cluster's internal coherence
2. Identify items that may be misplaced (with rationale)
3. Detect outliers, missing themes, and overlapping clusters
4. Provide confidence scores for each assessment

The response is structured JSON, parsed into the same data model regardless of provider. For the reasoning tier, extended thinking tokens are separately extracted and parsed into a step-by-step trace.

### Layer 3: Study Infrastructure

The study layer handles condition assignment, survey instruments, session recording, and data export. It is invisible to participants — they interact only with the canvas and critique panel.

Key components:
- **Condition assignment:** Random allocation with stratification by research experience
- **Session recording:** Full interaction log (card movements, timing, critique responses)
- **Survey engine:** Pre-task hypotheses capture, post-task trust/satisfaction scales
- **Export:** Anonymised dataset for statistical analysis

## Design Decisions

### Why Pull-Based Critique (Not Push)

The AI critique is triggered by the researcher clicking "Request critique," not generated automatically. This is a deliberate design choice for three reasons:

1. **Experimental control:** Pull-based gives a clean timestamp for the intervention and ensures all participants engage with the canvas before seeing AI output
2. **Researcher agency:** Unsolicited critique may feel adversarial; researcher-initiated critique feels collaborative
3. **Confound avoidance:** Push-based systems vary in interruption frequency, creating a confound between conditions

### Why One-Shot Critique (Not Iterative) for the RCT

During the study, participants can request critique once. This is not a product limitation — the production version supports iterative critique. One-shot is cleaner for analysis: it eliminates the confound that Treatment B's richer reasoning traces might generate more persuasive follow-up exchanges, making it impossible to disentangle model quality from conversational depth.

### Why Identical Prompts

The critique prompt does not change between Treatment A and Treatment B. This is the core experimental design principle: the only manipulation is the model's reasoning capability. If the prompts differed, any performance difference could be attributed to prompt engineering rather than model architecture.

### Why Visible Reasoning Traces

In Treatment B, the reasoning trace is visible to the researcher. This is a deliberate experimental choice — the trace is part of the intervention. We hypothesise that transparency of reasoning increases trust and engagement, which in turn improves critique utilisation. The study measures this directly via trust scales and interview data.

## Data Flow

```
Researcher uploads CSV
       │
       ▼
Items parsed → cards rendered on canvas
       │
       ▼
Researcher drags cards into clusters
       │
       ▼
Researcher clicks "Request critique"
       │
       ▼
Canvas state serialised → POST /critique
       │
       ▼
Critique engine selects adapter (Tier 1 or 2)
       │
       ▼
LLM generates structured JSON critique
       │
       ▼ (Treatment B only)
Extended thinking tokens → parsed reasoning trace
       │
       ▼
Critique panel renders:
  - Cluster assessments (coherent / mixed / split)
  - Item move suggestions (accept / reject)
  - Gap analysis (outliers / missing themes)
  - Reasoning trace (Treatment B, collapsible)
       │
       ▼
Researcher responds to suggestions
       │
       ▼
All interactions logged for RCT analysis
```

## Reasoning Trace Design

The reasoning trace is not raw token output — it is parsed into structured steps, each classified by type:

| Step Type | Icon | Colour | Example |
|-----------|------|--------|---------|
| Observation | 👁 | Grey | "Looking at this cluster, 3/5 items mention 'cost'" |
| Hypothesis | 💡 | Blue | "This could be a pricing-related cluster" |
| Counter | 🔀 | Amber | "But wait — 'costs too much time' isn't about money" |
| Revision | → | Violet | "Reconsidering: this is about friction, not pricing" |
| Conclusion | ✓ | Green | "I assess this cluster as mixed — conflating pricing and friction" |

This classification is heuristic, based on signal phrases in the chain-of-thought. It provides visual structure that helps researchers follow the AI's deliberation without reading raw text.
