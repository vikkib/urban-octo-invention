# Learning Mode

A guided, hands-on coding learning system for **Claude Code**. Pairs a custom skill with a global learning log so that every concept you study is captured once and remembered across every future session, in every project.

## What it is

Learning Mode is a custom Claude Code skill that turns Claude into a structured coding tutor. It is designed for **learning a concept for its own sake**, not for shipping production code. Sessions are interactive, paced by you, and built around small runnable examples.

The skill has **three modes**:

| Mode | Purpose | How to trigger |
|---|---|---|
| **Teach Mode** | Learn a new concept from scratch with explanation, a runnable example, predictive quizzes, and an end-of-session integrative quiz. | "teach me X", "I want to learn X", "explain X with an example" |
| **Review Mode** | Quiz yourself on concepts already in your learning log. Spaced-repetition aware: defaults to concepts due for review today based on each entry's `Next review` date. | "quiz me on what I've learned", "review what I've learned", "review mode" |
| **Mental Model Mode** | Get a concept-only walkthrough with analogy, diagram, and a code sketch, without writing or running code. Optionally escalates into Teach Mode at the end. | "just give me the mental model for X", "explain X conceptually", "mental model only" |

### Core principles

- **Concept first, code second.** Every step explains the *why* before any code is written.
- **Meet the user where they are.** Claude calibrates the level (Beginner, Intermediate, Advanced) by asking one targeted question up front.
- **The user drives pace.** Claude waits for confirmation before advancing to the next step.
- **Honest review.** When you write code yourself, Claude reviews it for correctness, safety, idiomatic style, and naive patterns. Critique is calibrated to your level and to the severity of each issue: hard-stop issues (correctness, security) are always flagged at full strength; "should know" issues are softened for Beginners and flagged in full for Intermediate / Advanced; style nits are skipped for Beginners. Capped at 2 issues per attempt for Beginners so feedback stays usable.
- **Quizzes are opt-in.** Predictive quizzes before running code, toggleable any time with "quiz me" or "stop quizzing".

### How a Teach Mode session flows

| Phase | What happens |
|---|---|
| **1. Intake** | Claude asks one calibration question, then sets a level (Beginner, Intermediate, Advanced). |
| **1.5. Prerequisite check** | Claude lists up to 3 prerequisite concepts and asks if you are comfortable. If not, offers to pivot, embed a refresher, or push through with a logged caveat. |
| **2. Example menu** | Claude proposes 2 or 3 small, self-contained examples. You pick one. |
| **3. Quiz check** | Claude asks once whether you want step-level predictive quizzes during the build. |
| **4. Build together** | Step by step, explain the concept, flag pattern vs scaffolding, offer to write it yourself, optional quiz gate before running. |
| **4.75. Optional stretch variant** | If the user breezed through Phase 4 on a Beginner / Intermediate example, Claude offers a single harder variant of the same concept (e.g. thread-safe lazy init after basic lazy init). Opt-in, never forced. Skipped entirely if the user struggled or signalled fatigue. |
| **4.5. Integrative quiz** | 2 or 3 end-to-end questions that test the whole concept, not individual steps. Result is tagged solid, shaky, or weak. |
| **5. Wrap-up** | Summary plus a suggested next concept. A new entry is appended to the learning log with `Initial understanding`, `Last reviewed`, and `Next review` fields based on the integrative quiz result. |

### How a Review Mode session flows

| Step | What happens |
|---|---|
| **1. Read the log** | Claude reads your `learning-log.md`. If empty, offers to start a Teach Mode session instead. |
| **2. Confirm scope** | Claude defaults to **concepts due for review today** based on each entry's `Next review` date. You can also choose all entries, just the most recent, a specific concept, or one at random. |
| **3. Generate questions** | For each concept, Claude generates 2 to 4 predictive questions drawn from the concept, why-it-matters, pattern, and real-world-reference sections of the entry. Mental-model-only entries get conceptual questions only. |
| **4. Run the quiz** | One question at a time, with confirmation or correction grounded in the log entry. |
| **5. Wrap-up** | Summary of strong vs weak areas. `Last reviewed` and `Next review` fields are updated on each entry based on quiz performance (see Spaced repetition below). Offers to re-teach weak concepts in Teach Mode. |

### How a Mental Model Mode session flows

| Step | What happens |
|---|---|
| **1. Concept explanation** | Definition, analogy, diagram, and a code sketch. One calibration question, no full intake. |
| **2. Checkpoint questions** | 1 or 2 predictive questions to confirm the model landed. |
| **3. Offer to escalate** | Claude asks if you want to make it concrete with a runnable example. Yes transitions into Phase 2 of Teach Mode. |
| **4. Wrap-up** | Shorter log entry tagged `Session type: mental-model-only`. Spaced-rep intervals start tighter (solid = 2 days, shaky / weak = 1) since the concept was not stress-tested with code. |

### Spaced repetition

Review Mode is not a one-off quiz tool. It uses a Leitner-style schedule to decide when each concept should be reviewed next:

| Quiz outcome on a concept | Next review interval |
|---|---|
| Got most wrong, or could not explain the why | 1 day |
| Got the right answer but with hesitation | 3 days |
| Got everything right confidently | Double the previous interval, capped at 90 days (30 for mental-model-only entries). First solid pass goes to 7 days, then 14, 30, 60, 90. |

The first-ever review of an entry uses the `Initial understanding` tag from the end-of-Teach-Mode integrative quiz to seed the interval (solid = 3 days, shaky = 2, weak = 1; tighter for mental-model-only).

## The three files that make it work

Learning Mode is wired together by three files living in `C:\Users\<username>\.claude\`. Two of them are global, so they apply to every project on this machine.

| File | Purpose | Scope |
|---|---|---|
| `skills\learning-mode\SKILL.md` | The skill itself, defines tutor behaviour, phases, and pacing rules. | Triggered when you type `/learning-mode` or use phrases like "teach me X". |
| `learning-log.md` | A running record of every concept covered across all learning sessions. | Global, persists across all projects. |
| `CLAUDE.md` | Global instructions Claude loads at the start of every conversation. Contains a pointer to the learning log and rules for using it. | Global, applied in every project on this machine. |

### How the connection works

```
You: "/learning-mode lazy init"
         |
         v
[ skills\learning-mode\SKILL.md ]   <- defines the tutor behaviour
         |
         v
[ CLAUDE.md ]                       <- tells Claude to read the log first
         |
         v
[ learning-log.md ]                 <- Claude checks what you have already learned
         |
         v
   Session runs (phases 1 to 5)
         |
         v
[ learning-log.md ]                 <- Claude appends a new dated entry at the end
```

### What each file contributes

- **`SKILL.md`** defines the *how* of teaching: the five-phase flow, quiz behaviour, code review lens, and ending sequence.
- **`CLAUDE.md`** acts as the *glue*. It tells Claude, in every project, that a learning log exists, where to find it, when to read it, and when to update it.
- **`learning-log.md`** is the *memory*. Without it, every session would start cold. With it, Claude can say "you saw lazy init last session, want to build on it with thread-safe lazy init?"

## The learning log entry format

Every appended entry follows the same shape so it is easy to scan later.

```markdown
## YYYY-MM-DD, Concept name (project)

**Session type**: teach | mental-model-only
**Concept**: One-line definition.

**Why it matters**:
- Practical motivation, the bug it solves, or the property it gives you.

**The pattern**:
\`\`\`language
Minimal code shape worth memorizing.
\`\`\`

**Real-world reference**: Where this pattern shows up in actual code you have written.

**Hands-on exercise built**: The mini-project you built during the session. Omit for mental-model-only sessions.

**Stretch variant**: Optional. Only included if Phase 4.75 ran. One-line description of the harder variant covered.

**Initial understanding**: solid | shaky | weak

**Prerequisite caveat**: Optional. Only included if you pushed through a flagged prerequisite gap.

**Last reviewed**: YYYY-MM-DD

**Next review**: YYYY-MM-DD

**Next suggested concept**: A natural follow-on for the next session.
```

## Why this design

- **Skill stays stateless.** The skill file itself does not need to know what you have learned. It only knows *how* to teach.
- **Log stays human-readable.** You can open `learning-log.md` in any editor, review entries, edit them, or delete ones you no longer need.
- **Global CLAUDE.md keeps it project-agnostic.** Whether you are in a Streamlit app, a Next.js site, or a CTF box, the same learning log is available.
- **Token-efficient.** The log is only read when a `/learning-mode` session starts, so it does not bloat unrelated conversations.

## Setting it up on a new machine

If you reinstall Claude Code or move to a new machine, recreate the three files in the same locations.

| Step | File | Action |
|---|---|---|
| 1 | `C:\Users\<username>\.claude\skills\learning-mode\SKILL.md` | Copy the skill definition file. |
| 2 | `C:\Users\<username>\.claude\learning-log.md` | Create an empty log with a `# Learning Log` heading. |
| 3 | `C:\Users\<username>\.claude\CLAUDE.md` | Add a `## Learning Log` section telling Claude to read the log at session start and append after each session. |

## Triggering a session

### Teach Mode triggers

Any of these phrasings will activate the standard teaching flow:

- `/learning-mode <concept>`
- "I want to learn X"
- "Teach me X"
- "Help me understand X"
- "Explain X with an example"
- "How does X work in code"

### Review Mode triggers

Any of these phrasings will activate the review flow against your existing learning log:

- "Quiz me on what I've learned"
- "Quiz me from the learning log"
- "Review what I've learned"
- "Test me on past concepts"
- "Review mode"

### Mental Model Mode triggers

Any of these phrasings will activate the concept-only walkthrough:

- "Just give me the mental model for X"
- "Explain X conceptually"
- "Mental model only"
- "I don't want to write code, just explain X"

Claude will not trigger Learning Mode automatically if you are mid-implementation on a real project. It is reserved for concept-focused study and review.

## Sessions logged so far

Once installed, your personal log lives at `C:\Users\<username>\.claude\learning-log.md` on Windows, or `~/.claude/learning-log.md` on macOS and Linux. Each completed learning session appends a new dated entry there. The `learning-log.md` file in this repo is a starter template containing one blank template entry plus two worked examples, prompt caching and tool use loop, to show the expected format. Replace or delete them before you begin.
