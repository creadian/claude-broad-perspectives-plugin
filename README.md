# Broad Perspectives — Multi-Agent Debiased Analysis Plugin

A plugin for [Claude Code](https://code.claude.com/) and [Claude Cowork](https://support.claude.com/en/articles/13345190-getting-started-with-cowork) that deploys independent sub-agents with fresh context windows to analyze problems from maximally diverse perspectives.

## The Problem

When you share your hypothesis with an AI, it anchors all subsequent thinking. Even if you say "ignore my assumption and think broadly," the hypothesis is already in the context window — the model literally cannot unsee it. Every "alternative perspective" it generates is shaped by your framing.

This isn't a prompting problem. It's an architectural one.

## The Solution

Broad Perspectives uses Claude's sub-agent architecture to achieve genuine context isolation:

1. **You describe your problem** (including any hypothesis you have)
2. **The Orchestrator** strips your hypothesis from the neutral facts
3. **Multiple Explorer sub-agents** are spawned in parallel — each with a **fresh context** containing only neutral facts and a unique perspective assignment. They never see your hypothesis or each other's work.
4. **A Hypothesis Evaluator** receives your hypothesis separately and both steelmans and stress-tests it
5. **A Challenger** reads all independent results and looks for what everyone missed
6. **A Synthesizer** integrates everything into a clear, actionable report

The result is something a single-context conversation fundamentally cannot produce: genuinely unanchored perspectives where convergence is high-confidence signal and divergence reveals your real blind spots.

## Architecture

```
User's Input (with hypothesis)
        │
        ▼
   ORCHESTRATOR (main agent)
   ├── Strips hypothesis from neutral facts
   ├── Designs Explorer roster for THIS specific problem
   │
   ├──► EXPLORER 1 ──┐
   ├──► EXPLORER 2 ──┤  (parallel, fresh context each)
   ├──► EXPLORER 3 ──┤  (neutral facts only — NO hypothesis)
   ├──► ...          ──┤
   ├──► EXPLORER N ──┤
   └──► HYPOTHESIS EVALUATOR (steelman + stress-test)
                      │
                      ▼
              CHALLENGER (finds what everyone missed)
                      │
                      ▼
              SYNTHESIZER (integrates for user)
```

## Installation

### Claude Cowork (recommended for non-developers)

1. Download or clone this repo
2. Zip the contents: `zip -r broad-perspectives.zip .`
3. Open Claude Desktop → Cowork tab → Plugins → Upload plugin
4. Select the zip file

### Claude Code

**As a plugin:**
```bash
claude plugin install /path/to/this/repo
```

**As a slash command only:**
```bash
mkdir -p ~/.claude/commands
cp commands/broad-perspectives.md ~/.claude/commands/

mkdir -p ~/.claude/skills/broad-perspectives
cp skills/broad-perspectives/SKILL.md ~/.claude/skills/broad-perspectives/
```

## Usage

With the slash command:
```
/broad-perspectives I've been experiencing headaches every afternoon. I think it's from my new monitor.

/broad-perspectives Should we migrate to microservices? Our CTO says it's the only way to scale.

/broad-perspectives panoramic: I'm considering leaving my stable job to start a company.
```

Or just describe your problem naturally and mention "broad perspectives":
```
Can you do a broad perspectives analysis on our go-to-market strategy?
```

### Options

- **Explorer count**: Dynamic by default (3-12 based on complexity). Override with: `/broad-perspectives 10 explorers: [question]`
- **Maximum breadth**: Add "panoramic" for 9-12 Explorers
- **Drill down**: After results, ask to see any specific Explorer's raw output

## How It Works

### Explorer Design

The Orchestrator generates problem-specific perspectives — not a fixed checklist. For a medical question you might get an internist, pharmacologist, and environmental medicine specialist. For a business question: an industry analyst, behavioral economist, and regulatory expert.

~40% of Explorers are **wildcards** — deliberately unusual perspectives (a military strategist analyzing your business problem, a comedian spotting absurdities in your medical situation). These often surface the highest-impact insights.

### Information Firewall

| Agent | Sees neutral facts | Sees hypothesis | Sees other Explorers | Mandate |
|-------|:-:|:-:|:-:|---------|
| Explorers | ✅ | ❌ | ❌ | Broad analysis from one perspective |
| Hypothesis Evaluator | ✅ | ✅ | ❌ | Steelman + stress-test |
| Challenger | ✅ | ✅ | ✅ | Find what everyone missed |
| Synthesizer | ✅ | ✅ | ✅ | Integrate into actionable report |

## Requirements

- **Claude Code** or **Claude Cowork** (requires sub-agent spawning — does not work in regular Claude Chat or the Claude iOS app)
- A paid Claude plan (sub-agent spawning uses significant tokens)

## Background

This plugin was born from a conversation about how LLMs handle anchoring bias. Key insights:

- LLMs generate output conditioned on the full context — mentioning a hypothesis shifts probability mass toward related concepts
- Even instructions to "ignore my hypothesis" can't undo this — the tokens are already in the context window
- Prompting tricks (adversarial roles, forced categories) help but can't fully overcome context conditioning
- The only real solution is **architectural**: separate context windows that literally cannot see the contaminating information

For more on the underlying problem, see [this discussion of LLM anchoring mechanics](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview).

## Credits

Created by [Christian Dömges](https://github.com/creadian) as part of the [AI Junto](https://www.aijunto.com/) community.

## License

MIT — use it, modify it, share it.
