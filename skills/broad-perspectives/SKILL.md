---
name: broad-perspectives
description: "Multi-agent debiased analysis system that deploys independent sub-agents with fresh context windows to analyze problems from maximally diverse perspectives. Use when user says 'broad perspectives', 'analyze this from all angles', 'what am I missing', 'debiased analysis', 'clean context analysis', 'multiple perspectives', or for high-stakes decisions, medical differentials, complex strategy questions, or any situation where anchoring bias could cause missed insights. Requires sub-agent spawning capability (Claude Code or Cowork)."
---

# Broad Perspectives

A multi-agent analysis system that defeats anchoring bias by deploying independent sub-agents, each with a fresh context window and no knowledge of each other's work or the user's hypothesis.

## Why This Exists

When a user describes a problem and includes their hypothesis, every analysis within that same context is anchored to their framing. Even explicit instructions to "think broadly" cannot undo context conditioning. This skill solves this by sending each analytical perspective to a separate sub-agent that literally cannot see the user's hypothesis or other perspectives.

## Architecture

```
User's Input (with hypothesis)
        │
        ▼
   ORCHESTRATOR (main agent — you)
   ├── Strips hypothesis from neutral facts
   ├── Designs Explorer roster for THIS problem
   │
   ├──► EXPLORER 1 ──┐
   ├──► EXPLORER 2 ──┤  (parallel, fresh context each)
   ├──► EXPLORER 3 ──┤  (neutral facts only, NO hypothesis)
   ├──► ...          ──┤
   ├──► EXPLORER N ──┤
   └──► HYPOTHESIS EVALUATOR (if hypothesis exists)
                      │
                      ▼
              CHALLENGER (sequential)
              (reads all results, finds gaps)
                      │
                      ▼
              SYNTHESIZER (sequential)
              (integrates everything for user)
```

## Protocol

### Phase 0: Problem Intake

Before spawning any agents, analyze the user's input:

**Step 1 — Extract neutral situation:** Strip all hypotheses, assumptions, and preferred framings. Write a clean, factual description containing only observable facts and stated constraints.

**Step 2 — Extract hypothesis:** Identify any assumed causes, preferred directions, or framings. If none exist, note "No hypothesis provided."

**Step 3 — Determine Explorer count:**
- If user specifies a number, use that
- If user says "panoramic" or "maximum breadth," use 9-12
- Otherwise, decide dynamically:
  - Focused (3-5): Clear domain, moderate stakes, straightforward question
  - Broad (6-8): Cross-domain, high stakes, or ambiguous
  - Panoramic (9-12): Life-critical, maximum uncertainty, novel territory

**Step 4 — Design the Explorer roster:** For THIS specific problem, generate perspective assignments. Each must be genuinely different. Aim for ~60% domain-expected and ~40% wildcard perspectives.

### Perspective Design Guidance

**Good wildcard perspectives share these properties:**
1. They see the problem through a fundamentally different value system
2. Their expertise reveals patterns invisible to domain experts
3. They notice what everyone else treats as background noise
4. They ask questions that make domain experts uncomfortable

**Domain-specific starting points (adapt, don't copy):**

Medical: internist, relevant specialist, pharmacologist, environmental medicine, psychoneuroimmunologist, nutritional biochemist. Wildcards: medical anthropologist, forensic pathologist, patient advocate, traditional medicine practitioner.

Business/Strategy: industry analyst, behavioral economist, operations/systems thinker, financial analyst, regulatory/legal. Wildcards: military strategist, game theorist, anthropologist, comedian (absurdity detection), historian of failures.

Technical: domain expert, security adversary, scalability engineer, UX/human factors, reliability engineer. Wildcards: building architect, biologist (natural systems), urban planner.

Personal decisions: financial planner, psychologist, future-you (10 years), someone who chose this and regretted it, someone who chose this and thrived. Wildcards: stoic philosopher, your harshest critic, alien anthropologist.

**Always consider adding 1-2 meta-perspectives:**
- The Inverter: "How would I make this fail on purpose?"
- The Time Traveler: "How does this look from 5 years out?"
- The Affected Bystander: Someone impacted but never consulted
- The Pattern Matcher: "Where have I seen this exact dynamic in a completely different domain?"

### Phase 1: Deploy Explorers (PARALLEL)

Spawn ALL Explorer agents simultaneously using the Task tool. Each uses `subagent_type: general-purpose`.

**CRITICAL: Launch all Explorers in a SINGLE message so they run in parallel.**

Each Explorer receives ONLY this prompt (customize bracketed sections):

```
You are an independent analyst investigating a situation from a specific perspective.
You have NO prior context — only what is provided below.

## Situation
[Neutral situation description — NO hypotheses, NO assumed causes]

## Your Perspective
You are analyzing this as: [specific perspective/role/lens]
Focus areas: [2-3 specific angles unique to this perspective]

## Instructions
1. Analyze from your perspective. Think broadly within your domain.
2. Identify the most likely explanations, causes, or approaches from your viewpoint.
3. Flag anything critical, time-sensitive, or commonly overlooked.
4. Note what additional information you would need.
5. Rate your confidence (low/medium/high) for each finding.

## Output Format
### Key Findings (from [perspective name])
- [Finding]: [explanation] — Confidence: [level]

### Critical Flags
- [Anything urgent, dangerous, or commonly missed]

### Information Gaps
- [What you'd need to know to be more certain]

### Unexpected Angles
- [Anything surprising or non-obvious from your perspective]
```

### Phase 2: Deploy Hypothesis Evaluator (PARALLEL with Explorers)

If the user provided a hypothesis, spawn alongside the Explorers:

```
You are evaluating a specific hypothesis about a situation.
Your job is to BOTH steelman it AND stress-test it.

## Situation
[Same neutral facts as Explorers received]

## Hypothesis to Evaluate
[The user's hypothesis]

## Instructions
1. STEELMAN: Build the strongest possible case FOR this hypothesis.
   What evidence supports it? What mechanisms explain it? When is this most likely?

2. STRESS-TEST: Build the strongest case AGAINST it.
   What contradicts it? What alternatives fit the same evidence? What tests would distinguish?

3. CALIBRATE: Rate 0-100% confidence with reasoning.

## Output Format
### Steelman Case
[Strongest argument for]

### Stress-Test Findings
[Strongest arguments against, with alternatives]

### Critical Distinguishing Tests
[What would confirm or refute this]

### Calibration
- Confidence: [X]%
- Key uncertainty: [what would shift this most]
- Strongest competing explanation: [best alternative]
```

### Phase 3: Deploy Challenger (SEQUENTIAL — after Phase 1-2 complete)

Spawn after all Explorers and Hypothesis Evaluator return:

```
You are a meta-analyst whose job is to find what EVERYONE ELSE MISSED.

## All Explorer Findings
[All Explorer outputs]

## Hypothesis Evaluation
[Hypothesis Evaluator output, or "No hypothesis was provided"]

## Instructions
1. CONVERGENCE: What did multiple Explorers independently point to? (High signal)
2. BLIND SPOTS: What domains or failure modes did NO Explorer cover?
3. ASSUMPTION AUDIT: What assumptions are ALL Explorers sharing unquestioned?
4. SECOND-ORDER EFFECTS: What consequences has nobody mentioned?
5. TEMPORAL BLIND SPOTS: What time-horizon is everyone ignoring?
6. CONTRARIAN: What's the most uncomfortable truth being avoided?
7. CONFIDENCE CHECK: Where is the collective over/under-confident?

## Output Format
### Strong Convergences
### Critical Gaps Found
### Shared Assumptions to Question
### Second-Order Effects
### Contrarian Perspective
### Confidence Corrections
```

### Phase 4: Deploy Synthesizer (SEQUENTIAL — after Phase 3)

Spawn with everything:

```
You are synthesizing a multi-perspective analysis. Multiple independent analysts
worked with clean, uncontaminated context. A challenger identified gaps.

## Original User Query
[User's full original input including hypothesis]

## Explorer Findings
[All outputs]

## Hypothesis Evaluation
[If applicable]

## Challenger Analysis
[Challenger output]

## Synthesize into this structure:

### Executive Summary
2-3 sentences: the most important things the user needs to know.

### Convergent Findings
Findings multiple Explorers independently identified. Highest confidence.

### Hypothesis Assessment (if applicable)
How the hypothesis holds up against independent perspectives.

### Critical Blind Spots Discovered
What the user was likely not considering. Ranked by potential impact.

### The Unexpected
Surprising findings from wildcards or Challenger. Potentially high-impact.

### Key Uncertainties
What remains unknown. What information would most reduce uncertainty.

### Recommended Next Steps
What should the user do, investigate, or reconsider.

### Confidence Dashboard
| Finding | Confidence | Agreement | Impact if Wrong |
```

### Phase 5: Present to User

Present the Synthesizer's output directly. Append:

```
---
**Broad Perspectives Analysis Complete**
- Explorers deployed: [N] ([list perspectives])
- Hypothesis evaluated: [Yes/No]
- Challenger gaps found: [N]
- Total independent contexts: [count]
```

Offer to show raw output from any specific Explorer if the user wants to dig deeper.
