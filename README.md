# 🧠 agent-skills

> A curated collection of reusable skills for AI coding agents — structured prompts and reference docs that give agents expert-level capabilities out of the box.

---

## 🎯 What is this?

Each **skill** is a self-contained directory with:
- A `SKILL.md` — when to use it, step-by-step process, rules, and common mistakes
- Optional reference files (e.g. XML patterns, style guides, cheat sheets)

Drop a skill into any agent framework and it gains that expertise immediately.

---

## 📦 Skills

| Skill | Description |
|-------|-------------|
| [`creating-architecture-diagrams`](./creating-architecture-diagrams/SKILL.md) | Generate professional draw.io diagrams (AWS/GCP/Azure) with mathematically precise edge routing, no line overlaps, and clean layout |

---

## 🚀 How to Use

1. **Point your agent at a skill** — include `SKILL.md` in the system prompt or context
2. **Invoke it by trigger** — each skill documents exactly when it should activate
3. **Extend it** — skills are plain Markdown, fork and customize freely

---

## 🤝 Contributing

Skills should be:
- ✅ Agent-framework agnostic
- ✅ Self-contained (no external dependencies)
- ✅ Trigger-documented (when to use / when NOT to use)
- ✅ Battle-tested (include common mistakes + fixes)

---

## 📄 License

MIT
