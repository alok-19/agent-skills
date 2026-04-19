# 🧠 agent-skills

> A curated collection of reusable skills for AI coding agents — structured prompts and reference docs that give agents expert-level capabilities out of the box.

---

## 🎯 What is this?

Each **skill** is a self-contained directory containing:
- `SKILL.md` — when to activate, step-by-step process, rules, and common mistakes
- Optional reference files (e.g. XML patterns, style guides, cheat sheets)

Skills are agent-framework agnostic. Use them with Claude Code, Antigravity, custom agents, or any LLM-powered workflow.

---

## 📦 Available Skills

| Skill | Description | Languages |
|-------|-------------|-----------|
| [`creating-architecture-diagrams`](./creating-architecture-diagrams/SKILL.md) | Generate professional draw.io diagrams (AWS/GCP/Azure) with mathematically precise edge routing, no line overlaps, and clean layout | Any |
| [`code-rigor-check`](./code-rigor-check/SKILL.md) | Structured pre-ship code review covering problem framing, edge cases, failure modes, and AI-specific failure patterns (hallucinated APIs, mirror tests, silent coercions). Includes a stack-specific red flags reference. | Any · Python · JS/TS · SQL · AWS · Java · Go · Rust · C# |

---

## 🚀 Setup Guide

### Option 1 — Project-Level via `.agents/skills/` (Recommended)

This works with any agent framework that resolves skills from a local directory — including Antigravity, Claude Code, and custom setups.

**Step 1: Create the skills directory in your project**

```bash
mkdir -p /your/project/.agents/skills
```

**Step 2: Clone and copy the skills you need**

```bash
git clone https://github.com/alok-19/agent-skills.git /tmp/agent-skills

# Copy a specific skill
cp -r /tmp/agent-skills/creating-architecture-diagrams \
      /your/project/.agents/skills/
```

**Step 3: Point your agent at the directory**

Configure your agent to load skills from `.agents/skills/`. The exact config key depends on your framework:

| Framework | Config |
|-----------|--------|
| Antigravity | Set skills path to `.agents/skills/` in your agent config |
| Claude Code | Skills in `.claude/skills/` are auto-loaded per project |
| Custom agent | Read each `SKILL.md` and inject into system prompt at startup |

Your project structure should look like:

```
your-project/
├── .agents/
│   └── skills/
│       ├── creating-architecture-diagrams/
│       │   ├── SKILL.md
│       │   └── drawio-xml-reference.md
│       └── code-rigor-check/
│           ├── SKILL.md
│           └── references/
│               └── ai-code-red-flags.md
└── ... rest of project
```

---

### Option 2 — Paste into System Prompt (Any Agent)

For any LLM-powered agent or custom workflow with no directory support:

1. Open the skill's `SKILL.md`
2. Copy its full content
3. Paste into your agent's **system prompt** or **context window**

The skill's `## When to Use` section tells the model exactly when to activate it.

---

### Option 3 — Claude Code Global Plugin

To make skills available across **all** your projects in Claude Code:

```bash
mkdir -p ~/.claude/plugins/custom/agent-skills/.claude-plugin
mkdir -p ~/.claude/plugins/custom/agent-skills/skills

cat > ~/.claude/plugins/custom/agent-skills/.claude-plugin/plugin.json << 'EOF'
{
  "name": "agent-skills",
  "description": "Reusable skills for AI coding agents",
  "author": { "name": "alok-19" }
}
EOF

cp -r /tmp/agent-skills/creating-architecture-diagrams \
      ~/.claude/plugins/custom/agent-skills/skills/
```

---

## 🤝 Contributing

Skills should be:
- ✅ Agent-framework agnostic
- ✅ Self-contained (no external dependencies)
- ✅ Trigger-documented (when to use / when NOT to use)
- ✅ Battle-tested (include common mistakes + fixes)

PRs welcome. Open an issue to propose a new skill topic.

---

## 📄 License

MIT
