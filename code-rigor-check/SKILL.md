---
name: code-rigor-check
version: 1.0.0
description: Runs a structured rigor check over code before it ships, with a specific mode for AI-generated output. Covers problem framing, edge cases, failure modes, explainability, and AI-specific failure patterns — hallucinated APIs, plausible-but-wrong library behavior, tests that mirror the code, defensive scaffolding that hides real bugs, silent coercions. Use whenever the user shares code and asks if it looks right, pastes output from an AI coding assistant (Claude Code, Copilot, Cursor, Cody, Windsurf, v0, Replit Agent, Aider) and is deciding whether to commit it, asks for a code review, asks "how should I approach X" on a non-trivial problem, or is designing something with real stakes (production, user-facing, money-moving, migrations, auth, schema changes). Trigger even when the user doesn't say "review" or "checklist" — signals like "does this look right", "is this okay", "any issues with this", "sanity check", or pasting code without a specific question all count.
---

# Code rigor check

A structured rigor check you can run on code — your own, theirs, or an AI's — before it does damage. Has a distinct mode for AI-generated code, which fails in characteristic ways that are different from how humans fail.

## Why this exists

Every competent engineer runs a checklist in their head before shipping. With AI-assisted coding the checklist quietly gets skipped: paste, tab-complete, ship, pray. Bugs that would have been caught by thirty seconds of "wait, is this actually solving the right problem?" slip through.

AI-generated code needs the check *more* than human code, not less. The most dangerous output is plausible-but-wrong: hallucinated APIs, outdated library behavior, tests that mirror the code rather than verify reality, defensive scaffolding around impossible conditions while the real edge cases go unhandled. Obvious bugs get caught. Subtle ones don't — and the author can't be paged at 3am to explain what they were thinking.

## When to run it

**Run the full check when:**
- The user pastes code and asks whether it's correct, good, or production-ready
- The user shares AI-generated output and is deciding whether to commit it
- The user asks for a code review
- The user is designing something non-trivial
- The stakes are real: production code, customer data, money movement, auth, migrations, schema changes

**Run a light version** (problem framing plus the one or two most relevant categories) when:
- It's clearly a throwaway script, prototype, or learning exercise
- The user just wants syntax help or a quick tweak
- The code is short and the intent is obvious

**Don't run it when:**
- The user asked a pure factual question ("what does `Promise.all` do")
- The user is thinking out loud and hasn't asked for review
- The user has clearly already done the thinking and is asking a specific, narrow question

Scale rigor to stakes. A one-off CSV munging script doesn't deserve the same scrutiny as a Lambda that writes to production.

## Universal rigor — the six questions

Work through these for any code review. Stop and flag as soon as you find something the user needs to know — don't save everything for the end, or the important thing gets buried.

### 1. Is this the right problem?

Restate the problem in one sentence. If the ask is ambiguous, surface the ambiguity *before* reviewing anything. Is the code solving the problem as stated, or a different (nearby) problem? Is there a simpler framing where the code wouldn't be needed at all?

The single most valuable question on the list. Most wasted engineering happens because someone built the right solution to the wrong problem.

### 2. Is the approach sound?

Is this the simplest approach that could work, or over-engineered? Are there standard patterns — language idioms, framework conventions, known algorithms — that would be cleaner? Are there hidden assumptions about input shape, ordering, uniqueness, timing, or environment that aren't stated?

### 3. What are the edge cases?

Walk the categories that matter for this code:
- Empty / null / zero / negative / boundary inputs
- Very large inputs (memory pressure, time complexity, pagination)
- Concurrent or racy access; duplicate deliveries
- Malformed, malicious, or untrusted input
- Network or external-service failures (partial, slow, wrong)
- Cold starts, empty caches, first-run behavior
- Timezone, locale, encoding, unicode

### 4. What are the failure modes?

What happens when an exception is thrown mid-operation — is state left consistent? What's the retry behavior, and is the operation idempotent? Are there timeouts, and what happens when they fire? What's the blast radius if this goes wrong — one user, all users, data loss, silent corruption? How will anyone *know* it failed? (Logs, metrics, alarms, dead-letter queues.)

### 5. Can every line be explained?

Go line by line. If a line can't be explained in plain English — what it does and *why it's there* — that line is a liability. Either understand it, replace it with something you do understand, or cut it.

Watch particularly for library calls with non-obvious behavior (silent retries, truncation, caching, connection pooling), type coercions or implicit conversions, regex, date math, floating-point arithmetic, and "clever" one-liners that look elegant but encode assumptions.

### 6. Was it actually run?

Did the author run it, or just read it? Reading is not running. Did they exercise both the happy path and at least one failure path? Are the tests meaningful, or do they just assert that the code does what the code does?

## AI-specific failures

When the code came from an AI assistant, add these checks. They catch failures humans rarely make but AIs routinely do.

### Hallucinated APIs and signatures

The model invents methods, keyword arguments, or attributes that sound right but don't exist. Tells:
- Keywords with plausible-but-wrong names (`header_row=` instead of `header=`, `timeout_seconds=` instead of `timeout=`)
- Methods on objects that don't have them (`df.remove_duplicates()` instead of `df.drop_duplicates()`)
- Imports from submodules that moved or were renamed between versions

**Check:** Does every external call match the actual library version in use? When in doubt, look it up rather than trust the code.

### Plausible-but-wrong library behavior

The model confidently uses a library in a way that almost works but has subtle wrong assumptions:
- Assuming `requests.get()` retries on failure (it doesn't)
- Assuming `json.loads()` handles `NaN` safely (it does, non-standardly — your consumer might not)
- Assuming `array.forEach(async ...)` runs sequentially with `await` (it doesn't)
- Assuming SQL `IN (NULL)` matches NULL rows (NULL isn't equal to anything, including itself)

**Check:** For any library call doing something non-trivial — retry, concurrency, nullability, serialization — verify the actual documented behavior, don't trust the surrounding code's implication.

### Tests that mirror the code

The AI wrote both the code and the tests. The mock is configured to return exactly what the code expects; the assertion checks exactly what the mock returned. The test passes, but it's a tautology — it verifies the author's mental model, not reality.

**Check:** Does any test exercise the code against realistic data the author didn't pre-shape? If every test uses mocks the same author wrote, the suite proves nothing.

### Defensive scaffolding for impossible cases, nothing for likely ones

The AI adds `if x is not None` guards on variables that can't be None, wraps `try/except: pass` around operations that can't fail — then misses the one case that actually happens. Often a sign the model was pattern-matching on "defensive code looks like this" rather than thinking about what could actually go wrong.

**Check:** For each guard and exception handler, ask: what would happen specifically without it? If the answer is "nothing different", it's noise. If the answer is "a real failure the handler doesn't address", the real problem isn't being fixed.

### Silent coercions and truncations

`str(x)[:100]`, `int(x) if x else 0`, `.get('key', '')` — each silently loses data or papers over the case where the input shape is wrong. Often added to make tests pass rather than because the behavior is correct.

**Check:** For each fallback or default, ask: is this the behavior the caller actually wants when the input is weird, or would raising be safer?

### The explanation-code mismatch

If the AI's written explanation says one thing and the code does another, **trust the code, not the explanation.** The explanation is generated separately and isn't guaranteed to describe what the code actually does. This is one of the most common sources of subtle bugs in AI output.

### Stale patterns from old training data

The model suggests deprecated APIs, removed flags, or patterns that were idiomatic two library versions ago. Common for fast-moving ecosystems: JS tooling, AWS SDK, Pandas, PyTorch, React.

**Check:** For anything version-sensitive, verify against current docs for the version actually installed.

For a fuller catalog with stack-specific patterns (Python, JS/TS, SQL, AWS/serverless, Java/Spring), see `references/ai-code-red-flags.md`.

## How to present the output

Lead with the verdict: ready to ship, fixable with specific changes, or fundamentally off-track.

Flag concrete issues, not generic advice. *"This will throw on an empty `items` array at line 14"* beats *"consider edge cases"*.

If the code is genuinely fine, say so plainly. Don't manufacture concerns to look thorough — the user will learn to ignore you.

Group findings by severity and label them:
- **Blocker** — will cause a bug in realistic use
- **Should fix** — real issue, not urgent
- **Nit** — style, naming, minor

When flagging AI-generated code that looks wrong, quote the specific line and explain what would need to be true for it to work correctly. Vague gestures at "this seems off" don't help.

For design or approach questions with no code yet: surface the problem-framing question first, then walk through the relevant pieces of #2–#4, end with what you'd need to know to give a firm recommendation.

## Worked examples

### Example 1 — AI-generated code, multiple failures

**User:** "Claude gave me this for deduplicating orders before insertion. Look right?
```python
def dedupe_orders(orders):
    return list(set(orders))
```"

**Response:**

> **Verdict:** Blocker — won't run at all on realistic input, and wouldn't preserve ordering even if it did.
>
> **Blockers:**
>
> 1. **`set()` requires hashable elements.** Order records are almost certainly dicts, which aren't hashable. This raises `TypeError: unhashable type: 'dict'` on the first call. Dedupe by a key instead: `list({o["id"]: o for o in orders}.values())`.
>
> 2. **Order not preserved.** Even if items were hashable, `set` is unordered. If insertion sequencing matters for your orders table, this bug is invisible until it isn't. The dict-based form above preserves first-seen order.
>
> **Worth pushing back on:** Is deduplication the right layer? A unique constraint on the DB catches duplicates from concurrent writers too, which application-level dedup can't. Worth asking why the model reached for code here instead of a schema change.

Lead with verdict, name the specific issue, quote the construct, give a concrete fix, surface the problem-framing question.

### Example 2 — the mirror-test pattern

**User:** "Here's the Lambda handler and test Claude wrote. Tests pass, but something feels off."

```python
def handler(event):
    user_id = event["user_id"]
    data = fetch_user(user_id)
    return {"statusCode": 200, "body": json.dumps(data)}

def test_handler():
    with patch("app.fetch_user") as mock:
        mock.return_value = {"name": "test"}
        result = handler({"user_id": "123"})
        assert json.loads(result["body"]) == {"name": "test"}
```

**Response:**

> Your instinct is right. This test is a tautology — it mocks `fetch_user` to return a value, then asserts the handler returns that same value. It proves nothing beyond "the handler passes through what fetch_user returns", which is obvious from reading the code.
>
> What's missing:
> - **No failure cases.** What if `event` has no `user_id`? What if `fetch_user` raises? What if `data` isn't JSON-serializable (datetime, Decimal, bytes)? These are the actual bugs the test should catch.
> - **No realistic data shape.** The mock returns `{"name": "test"}`; a real user record has nested fields, timestamps, possibly nulls. Try a realistic fixture and see if `json.dumps` handles it.
>
> Classic AI failure: the model writes code and test together, so the test only verifies the mental model that produced the code. Rewrite against fixtures the model didn't generate, or hit a stub service with realistic data.

### Example 3 — approach question, no code yet

**User:** "I need to add a retry to a Lambda that writes to DynamoDB. Thinking exponential backoff in a loop. Thoughts?"

**Response:**

> Before the retry loop, one framing question: **what are you retrying against?** The answer changes the design.
>
> - **Throttling** (`ProvisionedThroughputExceededException`): the SDK already retries these with jittered backoff by default. A manual loop duplicates that logic and usually makes it worse.
> - **Transient network errors**: also handled by the SDK's default retry config. Tune the config rather than wrap it.
> - **Conditional-check failures**: should *not* be retried blindly — the condition failed for a reason. Retrying can corrupt data.
>
> So the real question isn't "should I retry" but "which failures, and is the SDK already doing it?" Check `botocore.config.Config(retries=...)` first. If you genuinely need custom logic on top, the key concern is **idempotency** — is your write safe to do twice? If not, a retry can double-write on a response that was lost in transit but actually succeeded.
>
> Tell me which failure you're seeing and I can give a concrete recommendation.

## Calibration — the sharper principles

The checklist is a tool, not a ritual. A few principles that distinguish running it well from running it mechanically:

**Trust the code over the explanation.** When an AI's written rationale and the actual code disagree, the code is what runs. Read what's there, not what the model says is there.

**Confidence is flat.** An AI sounds equally certain about well-documented things and obscure ones. Calibration is the reader's job, not the model's.

**The most dangerous output looks correct.** Obvious bugs get caught. Plausible-but-wrong is what ships. Spend more attention on things that look right than on things that look off.

**If the problem is one sentence and the solution is fifty lines, be suspicious.** AI often over-scaffolds simple problems. The inverse is rarer but also possible: three terse lines for something that genuinely needs more care.

**Respect prior thinking.** If the user has walked through edge cases, tested the code, and named the tradeoffs, acknowledge it and focus narrowly on what they haven't covered. Running the full six questions on someone who already did the work trains them to stop asking.

The goal is to catch real problems, not to perform thoroughness.
