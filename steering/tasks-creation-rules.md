---
inclusion: fileMatch
fileMatchPattern: "**/*tasks*.md"
---

# TASK ORCHESTRATION RULES (PARALLEL OPTIMIZED)

## 1. WAVE ARCHITECTURE & 5X PARALLEL ISOLATION
- Kiro dispatches up to 5 tasks concurrently. Structure `tasks.md` into explicit sequential Waves (`## Wave 1`, `## Wave 2`), with up to 5 parallel tasks per wave.
- ZERO WRITE COLLISIONS: Tasks within the same parallel Wave MUST NEVER modify the same file. If two features need to register routes, exports, or shared types in the same file, extract that shared file into an earlier foundational Wave.
- WAVE 0 CONTRACT LOCK: Always make Wave 1 a fast, single-task "Contract & Types Lock" that creates/updates shared interfaces, DB schemas, and stub exports so Wave 2+ agents can code in parallel without blocking each other.

## 2. MICRO-SCOPING & HARD SIZE CAPS
- MAX 100 LOC / 3 FILES PER TASK: No single task may modify more than 3 files or require more than ~100 lines of code changes. Split larger components into discrete tasks.
- ZERO SPECULATION COORDINATES: Do not write vague tasks like "Update auth logic." You must resolve file paths during task generation and pin exact coordinates in every task:
  - `Write:` Exact file path(s) and approximate line spans to modify or create.
  - `Read:` Exact reference file(s) and line spans (max 2 files) the executing agent needs for type signatures or pattern matching.

## 3. READABILITY & ENCRYPTED THINKING BRIDGE
- HUMAN-SCANNABLE HEADERS: Write task titles in plain, jargon-free English starting with a clear action verb (e.g., `- [ ] 2.1 Add Stripe webhook signature check`) so a human can review the plan in 10 seconds.
- CAVEMAN METADATA: Keep sub-bullets under each task in ultra-brief ASD-STE100 fragments (active voice, max 15 words per bullet, no filler).
- PLAINTEXT DECISION BRIDGE: Because the execution model cannot read your internal thinking blocks, include a 1-line `Constraint:` bullet in every task stating the exact architectural boundary and what NOT to touch.

## 4. INTEGRATION-FIRST TESTING (TESTING TROPHY)
- BAN ISOLATED UNIT TESTS FOR GLUE CODE: Do not create unit test tasks for controllers, UI wrappers, DB repositories, or basic services. Rely strictly on `tsc --noEmit` for individual implementation tasks.
- END-OF-WAVE INTEGRATION GATE: Instead of testing every file in isolation, add ONE final task per Wave that writes/runs a fast integration test verifying the real data flow across the newly created modules (API route -> service -> test DB/state).
- UNIT TEST EXCEPTION: Only create unit test tasks for complex, stateless algorithmic logic (parsers, regex, financial math, state machines) with zero mocks.

## 5. PRAGMATIC TESTING & PRE-BAKED VERIFICATION
- BAN TEST SUB-TASK BLOAT: Do NOT generate separate unit-test sub-tasks for every implementation task. 100% coverage and mock-heavy unit tests are BANNED.
- INLINE HAPPY-PATH TESTS ONLY: Create test tasks ONLY for critical business logic failure states and core happy paths. Never create tasks to test UI layout states, getters/setters, or third-party SDK wrappers.
- MANDATORY 1-LINE VERIFY COMMAND: Every task MUST end with an exact, copy-pasteable `Verify:` bash command that runs in under 5 seconds (e.g., single-file typecheck or single-file test with `--silent | tail -n 20`). Never instruct the agent to run the full test suite during task execution.

## 6. STRICT TASK ITEM TEMPLATE
Format every task in `tasks.md` using this exact 5-line structure:

- [ ] <Wave>.<Task> <Plain-English Action Title> _(Req: <ID>)_
  - **Write:** `<path/to/file.ts:lines>`
  - **Read:** `<path/to/types.ts:lines>` (Pattern ref: `<path/to/existing.ts:lines>`)
  - **Action & Constraint:** <Telegraphic spec of change. State exact banned actions/imports.>
  - **Verify:** `<exact single-file check command --silent | tail -n 20>`
