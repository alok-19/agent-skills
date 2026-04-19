# AI-generated code red flags — stack-specific catalog

A deeper catalog of failure patterns to watch for when reviewing AI-generated code, organized by stack. Each entry has a **signature** (how to spot it), **why it happens** (what the model is doing wrong), and **how to verify**.

Consult this when reviewing code in one of these stacks and you want a richer checklist than the generic AI-failures section of SKILL.md.

## Python

### `except Exception: pass` or bare `except:`
- **Signature:** Empty or log-only exception handlers wrapping code that could fail for several distinct reasons.
- **Why it happens:** Pattern-matching on "defensive code looks like this" without reasoning about which exceptions are actually expected.
- **Verify:** For each handler, name the specific exception type and the specific condition it handles. If you can't, the handler is hiding bugs rather than managing them.

### Outdated Pandas patterns
- **Signature:** `df.append(other)`, `df.ix[...]`, `pd.read_excel(sheet=...)` (correct kwarg is `sheet_name`).
- **Why it happens:** These APIs existed in older Pandas; training data is heavy on them.
- **Verify:** Check installed version. `.append` was removed in Pandas 2.0.

### Truthiness vs. `is not None`
- **Signature:** `if value:` when `value` could legitimately be `0`, `""`, `[]`, or `False` — all falsy but meaningful.
- **Verify:** If you mean "is not None", write `if value is not None`. Don't rely on truthiness for optional values.

### `requests` without timeout
- **Signature:** `requests.get(url)` with no `timeout=` kwarg.
- **Why it happens:** Extremely common in tutorials and examples; looks fine.
- **Verify:** In any production code, always pass a timeout. Without it, a dead server hangs the process forever.

### `datetime.now()` without timezone
- **Signature:** `datetime.now()` used anywhere that data might cross timezones (logs, DB, API responses).
- **Verify:** Use `datetime.now(timezone.utc)`. Naive datetimes are a recurring source of subtle bugs.

## JavaScript / TypeScript

### `forEach` with async
- **Signature:** `array.forEach(async (x) => { await foo(x); })` followed by code that assumes all `foo` calls finished.
- **Why it happens:** Model is mimicking synchronous iteration patterns.
- **Verify:** `forEach` ignores returned promises. Use `for (const x of array) { await foo(x); }` for sequential, or `await Promise.all(array.map(async (x) => foo(x)))` for parallel.

### `==` instead of `===`
- **Signature:** Loose equality anywhere.
- **Verify:** Almost always use `===`. The exceptions are vanishingly rare and should be commented.

### Stale closures in React
- **Signature:** `useEffect(() => { doSomething(state); }, [])` — empty deps but the effect reads state.
- **Why it happens:** Model knows `useEffect` exists; dependency arrays are a common blind spot.
- **Verify:** Either include `state` in deps, use a ref, or restructure to not need it. The linter (`react-hooks/exhaustive-deps`) catches these.

### `JSON.parse(JSON.stringify(x))` for deep clone
- **Signature:** This idiom used to deep-clone objects.
- **Verify:** Loses `undefined`, functions, `Date` (becomes string), `Map`, `Set`, throws on circular refs. Use `structuredClone` in modern environments.

### `any` sprinkled in TypeScript
- **Signature:** AI-generated TS with `any` types to make errors go away.
- **Why it happens:** Easier than solving the real type mismatch.
- **Verify:** Each `any` is a type check that was skipped. Replace with the real type or `unknown` + a narrow.

## SQL

### `NULL` comparisons
- **Signature:** `WHERE x = NULL`, `WHERE x IN (NULL, 1, 2)`, `WHERE x != value` expecting NULLs to match or be excluded.
- **Why it happens:** Intuition says NULL compares like other values. SQL disagrees.
- **Verify:** Use `IS NULL` / `IS NOT NULL`. For negation that should include NULLs: `WHERE x != value OR x IS NULL`.

### Missing indexes on filter/join columns
- **Signature:** Queries with `WHERE`/`JOIN` conditions, no indication indexes exist on those columns.
- **Why it happens:** Model generates functionally correct SQL without thinking about query plans.
- **Verify:** Run `EXPLAIN` on non-trivial queries. Watch for full table scans on large tables.

### N+1 patterns
- **Signature:** A loop in application code firing one query per iteration against the same table.
- **Why it happens:** Easier for the model to write one-query-per-thing than a JOIN or batch fetch.
- **Verify:** Count queries in the critical path. If it scales with input size, rewrite as a single query or batch.

### String interpolation instead of parameterization
- **Signature:** `f"SELECT * FROM users WHERE name = '{name}'"` or equivalent.
- **Why it happens:** Looks cleaner in a snippet.
- **Verify:** Use parameterized queries always. String interpolation is an injection vector and also breaks on quotes in the input.

## AWS / serverless

### Lambda without idempotency guarantees
- **Signature:** Lambda handler writing to DB or calling external services without checking if the same event was already processed.
- **Why it happens:** Model treats Lambda like a regular function. It's not — SQS can redeliver, API Gateway retries failed integrations, EventBridge can double-fire.
- **Verify:** For any side effect, either make it idempotent (upsert by a stable key from the event) or dedupe at handler entry (e.g., check a "processed" table keyed by message ID).

### `async` handler returning before async work completes
- **Signature:** Node.js Lambda with `async` handler that fires off a promise without awaiting, then returns.
- **Why it happens:** Model is used to long-running processes where promises continue in the background.
- **Verify:** Lambda freezes the runtime when the handler returns. Unresolved promises may never complete, or complete on a later invocation of the same warm container in confusing ways.

### Missing DLQ / alarm / observability
- **Signature:** Error paths that `console.error` and return, with no dead-letter queue, no CloudWatch alarm, no metric emission.
- **Why it happens:** Model writes the happy path and stops.
- **Verify:** How will anyone know this failed? If the answer is "by noticing the data's missing", add observability — DLQ on async triggers, alarms on error rate, metrics on business-critical paths.

### Over-broad IAM policies
- **Signature:** `"Action": "*"`, `"Resource": "*"`, or wildcard service access in generated IAM policies.
- **Why it happens:** Scopes it to "whatever would make this work".
- **Verify:** Principle of least privilege. Name the specific actions and resources. If the model couldn't figure out the specific ones, that's your job.

### `timeout: 3` on SDK calls when Lambda timeout is 30
- **Signature:** Lambda has a 30s timeout but calls a DynamoDB/S3 client with no configured timeout, or vice versa.
- **Verify:** Client timeout should be less than Lambda timeout, with room for retries. Otherwise the Lambda times out before the client can fail cleanly, and you lose the error signal.

## Java / Spring

### `@Transactional` on a private method
- **Signature:** `@Transactional` on a `private` method, or on a method called directly from another method in the same class.
- **Why it happens:** Spring's proxy-based transaction management is non-obvious; model doesn't always account for it.
- **Verify:** Transactions only apply when the call goes through the Spring proxy. Same-class calls bypass it. Move the method to a separate bean, or use `AopContext.currentProxy()`, or restructure.

### Wrong transaction boundary
- **Signature:** `@Transactional` wrapping an external HTTP call or message publish.
- **Why it happens:** Model applies `@Transactional` by pattern ("this method touches the DB").
- **Verify:** Holding a DB transaction open during a network call is a known anti-pattern — lock contention, connection pool exhaustion, timeouts. Either commit before the external call or use an outbox pattern.

### `Optional` fields in entities
- **Signature:** JPA entity fields typed as `Optional<T>`.
- **Why it happens:** Model generalizes `Optional` from return types to fields.
- **Verify:** `Optional` is documented as a return-type idiom only. Using it for entity fields breaks serialization, JPA providers, and equals/hashCode assumptions.

### `RestTemplate` without timeouts
- **Signature:** `new RestTemplate()` with default config.
- **Verify:** Defaults are "no timeout". Always configure connect and read timeouts on any HTTP client.

## Go

### Ignoring returned errors
- **Signature:** `result, _ := someFunc()` — error discarded with blank identifier.
- **Why it happens:** Model focuses on the happy path; Go's multi-return is unfamiliar to models trained on single-return languages.
- **Verify:** Every error return must be handled or explicitly justified. Silently discarding errors is one of the most common sources of production bugs in Go.

### Goroutine leaks
- **Signature:** `go func() { ... }()` launched without a clear termination condition or context cancellation.
- **Why it happens:** Model knows goroutines are "cheap" and uses them freely without thinking about lifecycle.
- **Verify:** Every goroutine must have a way to stop — a `context.Context` cancel, a done channel, or a bounded lifetime. Goroutine leaks accumulate silently and cause memory exhaustion.

### Nil pointer dereference on interface values
- **Signature:** Checking `if x != nil` on an interface variable when the underlying concrete value could still be nil.
- **Why it happens:** Model applies the null-check pattern without understanding Go's interface internals (an interface is non-nil if it has a type, even if its value is nil).
- **Verify:** If the interface was assigned from a typed nil pointer, `x != nil` is `true` but calling methods will panic. Check the concrete type or use `reflect.ValueOf(x).IsNil()` when needed.

### `defer` inside a loop
- **Signature:** `defer file.Close()` or similar inside a `for` loop body.
- **Why it happens:** Model applies "always defer cleanup" idiom without accounting for loop scope.
- **Verify:** Defers in loops don't run until the function returns, not at each iteration — file descriptors, locks, or connections accumulate open. Extract the loop body into a named function and defer inside that.

### Copying a struct with a mutex
- **Signature:** Passing a `sync.Mutex` or `sync.RWMutex` by value (as part of a struct copy).
- **Why it happens:** Model copies structs freely without checking for non-copyable fields.
- **Verify:** A mutex must never be copied after first use — the copy has independent state, silently breaking the synchronization. Use a pointer receiver or pass a pointer to the struct.

---

## Rust

### Cloning to avoid borrow checker errors
- **Signature:** `.clone()` called on data that could be borrowed, especially inside loops or hot paths.
- **Why it happens:** Model resolves ownership/borrow errors by cloning rather than restructuring lifetimes.
- **Verify:** Each `.clone()` on non-trivial data (Strings, Vecs, nested structs) is a potential performance issue. Ask: is the clone necessary, or is the ownership structure wrong?

### `unwrap()` and `expect()` in non-prototype code
- **Signature:** `.unwrap()` or `.expect("msg")` on `Option` or `Result` outside of tests or clearly documented invariants.
- **Why it happens:** Model uses `unwrap` to keep examples short; it bleeds into generated production code.
- **Verify:** Every `unwrap` that can panic in production is a latent bug. Replace with `?`, `if let`, `match`, or an explicit fallback. Reserve `expect` for invariants you're genuinely certain hold, with a comment explaining why.

### `async` without `Send` bounds on spawned tasks
- **Signature:** `tokio::spawn(async move { ... })` where the future holds non-`Send` types (e.g., `Rc`, `RefCell`, raw pointers).
- **Why it happens:** Model writes async code without tracking the `Send` constraint the executor requires.
- **Verify:** `tokio::spawn` requires `Future + Send + 'static`. If the future captures non-`Send` types, it won't compile — or the model worked around it in a way that forces single-threaded execution without saying so.

### Integer overflow in release builds
- **Signature:** Arithmetic on integer types (`u32`, `usize`) without overflow checks, especially in index or size calculations.
- **Why it happens:** Debug builds panic on overflow; release builds wrap silently. Model may test in debug mode only.
- **Verify:** In release mode, Rust integer overflow wraps (by default). For arithmetic where overflow is an error, use `checked_add`, `saturating_add`, or `wrapping_add` explicitly.

### Lifetime annotations that compile but are too broad
- **Signature:** `'static` bounds or lifetime elision that happens to compile but ties data to a longer lifetime than intended.
- **Why it happens:** Model adds `'static` to silence lifetime errors rather than correctly threading lifetimes.
- **Verify:** `'static` means the data lives for the entire program. If the model used it to make a struct or closure compile, check whether the data actually needs that guarantee or if a proper lifetime parameter would be correct.

---

## C# / .NET

### `async void` methods outside event handlers
- **Signature:** `async void MyMethod(...)` in non-event-handler code.
- **Why it happens:** Model converts `void` methods to `async` without changing the return type to `Task`.
- **Verify:** `async void` exceptions can't be awaited or caught by callers — they crash the process. Only valid for event handlers (e.g., button click). Everything else must return `Task` or `Task<T>`.

### Blocking on async code with `.Result` or `.Wait()`
- **Signature:** `someTask.Result` or `someTask.Wait()` called in synchronous code that might run on a thread with a `SynchronizationContext` (ASP.NET, WPF, WinForms).
- **Why it happens:** Model mixes sync and async without understanding context-capture deadlocks.
- **Verify:** In ASP.NET (non-Core) or UI frameworks, `.Result` / `.Wait()` can deadlock because the continuation tries to resume on the captured context, which is blocked waiting for `.Result`. Use `await` throughout or `.ConfigureAwait(false)` on the task.

### `DbContext` shared across threads or requests
- **Signature:** `DbContext` registered as a singleton or stored in a static field.
- **Why it happens:** Model applies generic DI patterns without knowing EF Core's scoping rules.
- **Verify:** `DbContext` is not thread-safe and is designed for per-request scope (`AddDbContext` uses scoped lifetime by default). A singleton `DbContext` causes concurrency bugs and connection exhaustion.

### `string` concatenation in a loop
- **Signature:** `result += someString` inside a loop building a large string.
- **Why it happens:** Looks correct and works — just quadratic.
- **Verify:** Strings are immutable in C#; each `+=` allocates a new string. Use `StringBuilder` for any loop where the number of iterations is non-trivial.

### Missing `CancellationToken` propagation
- **Signature:** `async` methods that accept a `CancellationToken` parameter but don't pass it to inner `await` calls.
- **Why it happens:** Model adds the parameter to satisfy the signature but forgets to thread it through.
- **Verify:** A `CancellationToken` only works if it's passed to every awaitable inside the method — HTTP client calls, DB queries, `Task.Delay`, etc. An unthreaded token gives callers false confidence that cancellation works.

---

## General principle across all stacks

If the code looks like an idiomatic answer to a common question, and your actual requirement is even slightly non-standard, the code is probably slightly wrong. The model interpolates from its training data — it's answering the question it's seen a hundred times, which may not be the question you're actually asking.

When in doubt: look at the problem, then look at the code, and ask whether the code is solving *your* problem or a generic-textbook version of it.
