# Software Philosophy

**Author:** Rikona (Mjoyufull)  
**Version:** 2.0.0  
**Date:** 2026-03-25  
**Companion:** See [QUESTIONS.md](file:///home/chris/projects/dsocs/PROJECT_STANDARDS/QUESTIONS.md) for detailed positions on controversial and cultural topics.

This document writes down the rules I actually use to build software, scope projects, and make architecture calls. It's not a wishlist—it's just what I already do. All my other standards and audit docs flow from here.

---

## 1. Why This Document Exists

Software engineering culture is loud and opinionated. People will tell you systemd is a honeypot, that Python is always wrong, that your crate layout is not Unix enough, that the AUR is a security nightmare, that every abstraction is over-engineering, and that every dependency is a liability. Most of these criticisms contain truth. Almost none of them are offered by people who have shipped and maintained the thing they are criticizing.

I'm tired of carrying unwritten rules in my head and re-deriving them from scratch every time I sit down to scope a project or push back on a design choice. This is the explicit version. If a decision I make contradicts something written here, either I had a good reason and should document it, or I drifted and should fix it.

---

## 2. Core Axioms

These aren't suggestions. They apply everywhere, in any language.

### 2.1 Clarity Beats Cleverness

Readable code wins over impressive code every time. If I can't look at a function signature and a quick scan of the body to know what it does, it's too clever. Compression tricks, macro abuse, and those annoying "elegant" one-liners are just tech debt painted over.

### 2.2 Make Illegal States Unrepresentable

If a state shouldn't exist, the compiler shouldn't let it compile. Stop relying on runtime discipline for things the type system can catch at build time. Use enums, newtypes, and module boundaries to enforce reality.

### 2.3 Explicit Over Implicit

Hidden control flow, ambient global state, and "magic" abstractions kill understanding. If code does a thing, I want to see it doing that thing. Stuff like `#[derive(Debug, Clone)]` is fine because it's obvious. Macros that generate invisible control flow are not fine.

### 2.4 "Simplicity" Is a Myth — Measure Extensibility and Maintainability Instead

Simplicity is a far-gone conclusion in computer engineering. Everything is built on top of a bootstrap on top of another bootstrap. "Simplicity" is just a buzzword used to justify positions without evidence. "Simple" dwm relies on X for handling everything; "simple" Rust requires the `time` crate to tell time; bfetchaust is 2k lines because Austral has no stdlib while the C version is 600 lines because C's stdlib does the heavy lifting. LOC limits without context mean nothing.

What actually matters: how maintainable the software is, what it does, how fast it is, how well it interoperates, and how it fits your workflow. Keeping LOC low in individual files helps maintainability. But extensibility and functional performance are the real metrics—not arbitrary simplicity claims.

### 2.5 Honesty in Software

Software needs to mirror what the machine is actually doing. If the code says a resource is freed, it better be freed. If a function is marked safe, it must be safe. Dead parameters and fake facade types are just lying to whoever reads the code next.

### 2.6 Ownership Is Not Optional

If you take a resource, you release it. Lifetimes need to be explicit or statically provable. If you grab a file handle, socket, or GPU allocation, the release path needs to be visible in the code—and ideally enforced by the compiler. "The GC will handle it" isn't a memory model, it's hope.

### 2.7 Build What's Needed

I don't start projects that are useless. If existing tools work, I use them. If the tool I need doesn't exist or existing options fail at my requirements, I build it. This isn't NIH syndrome. fsel exists because other terminal launchers didn't work with otter-launcher how I needed them to. Kaleidux exists because no wallpaper daemon handled video the way I needed. bfetch exists because fastfetch was too slow and couldn't show my custom art. Look for existing solutions first, then script or extend, then build.

### 2.8 Intellectual Honesty Is Non-Negotiable

Intellectual dishonesty is the root of most failures. "Justing" is intellectual dishonesty. Claiming software is simple when it offloads complexity elsewhere is intellectual dishonesty. Hiding behind "competence" to justify unreadable code is intellectual dishonesty. Evaluating software on hype rather than functional performance is noise. A valid opinion requires justified, reasonable evidence. Everything else is background noise.

---

## 3. Engineering Principles

How the axioms translate into daily decisions.

### 3.1 Boundaries Are Load-Bearing

Module boundaries, API surfaces, and process borders aren't just there to keep folders neat. They limit blast radius and enforce rules. Crossing a boundary should take an explicit call or import, not some weird side channel.

### 3.2 Error Handling Is Architecture

Errors are normal program behavior, so treat them like it. Error types need as much thought as success types. Panicking on recoverable stuff in production is a bug. Random `unwrap()` calls in hot paths are trash. Keep errors typed and explicit—don't swallow them or use lazy string errors everywhere.

### 3.3 Performance Is a Design Constraint, Not an Afterthought

Performance matters from day one. I'm not talking about micro-optimizing every loop, I'm talking about not making structural choices that make speed impossible later. O(n) scans in hot paths, allocating per-frame, or synchronous I/O in async contexts aren't "we'll fix it later" things—they are fundamental architecture mistakes.

### 3.4 Functional Performance

Functional performance is how software interoperates in your workflow—a measurement of daily usage in the environments the software actually lives in. It isn't raw benchmarks in isolation. Comparing fsel (a terminal launcher) to rofi (a GUI launcher) on rendering quality is pointless; comparing their abilities as dmenu replacements, their scriptability, and their interop with tools like uwsm or runapp is functional performance.

Hardware is extremely performant; software bottlenecks should be a non-issue. Apps should launch as fast as the CPU can calculate font textures. Functional performance has to be tangible alongside raw speed, not separate from it.

### 3.5 Core-Style Architecture

Build software as a core with the rest built around it. Not a monolith, not a thousand tiny scripts, but a core that separates concerns cleanly. Look at how Zed separated into gpui and other components. The core is explicit and readable. It offers niche extensibility through clear boundaries—plugin systems, hook architectures, or RPC depending on scope.

The second tooling and scripts can change the main function of what the software does, it has gone into absurdity. Emacs is the extreme example here—where extensibility became the product and the software became art rather than a tool.

### 3.6 Composition Over Configuration

Build systems out of composable chunks with good interfaces. Profile-driven composition—where you just pick what providers you want—beats a giant monolith with a thousand boolean config flags. This holds true whether you're building a distro, a plugin system, or just setting up crates.

### 3.7 Evidence Over Opinion

Technical choices need to be backed by reality: benchmarks, specs, or actual observed behavior. "I heard X is bad" means nothing. "X causes this specific, measurable problem" is valid. Credibility comes from actually building and maintaining real code, not parroting Twitter threads.

### 3.8 Determinism Is a First-Class Requirement

If you put the same inputs in, you should get the same outputs. Non-determinism should be quarantined. Builds, dependencies, and test suites need to be reproducible. If something genuinely can't be reproducible, document exactly why and box it in.

### 3.9 Incremental Over Revolutionary

Build things incrementally. A walking skeleton that actually does something is worth way more than a massive type model with no runtime. Scope aggressively. Ship the smallest honest piece, then build on top of it. Stop building Phase 3 abstractions when Phase 1 doesn't even work yet.

---

## 4. What I Refuse

I will not adopt these patterns, no matter how popular they get.

### 4.1 "Just" Culture

If you tell me to "just use X" without showing you actually understand why I'm not using it, I'm ignoring you. "Justing" is the lowest form of tech advice. It assumes I'm just ignorant instead of having actually considered and rejected it. If your advice requires me to break rules I've already documented, it's just noise.

### 4.2 Premature Generalization

Building a framework before you have a second use case is a waste of time. Building a plugin system before you have two real plugins is stupid. Abstractions are fine only if they are trivially cheap, or if the second use case is concrete and happening right now.

### 4.3 Architecture Tourism

Grabbing a shiny pattern just because some big company uses it isn't engineering, it's cargo-culting. Microservices, event sourcing, CQRS, hexagonal architecture—they are just tools, not religions. Use them if they actually solve the problem you have. Otherwise, leave them alone.

### 4.4 Death by Abstraction

Abstractions aren't free. They cost indirection, mental overhead, and make the codebase harder to jump into. Three layers of trait objects wrapping a basic `println!` isn't "extensible", it's just unreadable garbage. Only introduce abstractions to kill actual pain or duplication, not because you think you might need it someday.

### 4.5 Criticism Without Credibility

If you haven't built, maintained, or actually used the kind of system you're criticizing, your opinion holds zero weight. Code review isn't a spectator sport. "This is bad" with no evidence, no alternative, and no skin in the game isn't feedback—it's just complaining.

---

## 5. Project-Scoping Principles

Rules for deciding what to build, what to put off, and what to kill.

### 5.1 Scope Is Everything

Almost every dead project—mine or anyone else's—died because of scope. Either they tried to boil the ocean, or the scope was left vague and just crept until it killed the motivation. Figure out the scope before writing line one. Revisit it when reality hits. Cut it when delivery gets risky.

### 5.2 Hardness-Driven Build Order

Work should be ordered by difficulty and maturity, not some arbitrary calendar. Grab the easy wins first. Treat the hard problems as research. Anything gated by missing hardware or heavy research gets shoved to the back of the line without apology. This is the HDB (Hardness-Driven Build) model I use everywhere.

### 5.3 Plans Are Living Documents

A plan that you write once and ignore is a tombstone. Keep it updated. Update it after every big merge. Don't just claim something is "Implemented"—drop a file path or a test name next to it so it's backed by reality.

### 5.4 Audits Are Not Punishment

Audits exist to hold a mirror up to the codebase. They aren't performance reviews to bash people with. They purely exist to track what's real, what's faked, what's stubbed, and what's actively broken. A good audit should make it way easier for someone to jump in and contribute.

---

## 6. Language and Tool Preferences

These are defaults, not dogma. Each can be overridden with documented justification.

### 6.1 The Language Ladder

| Slot                  | Current                                                   | Future             | Reasoning                                                                                                                                                                                                                        |
| --------------------- | --------------------------------------------------------- | ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Safe systems language | Rust                                                      | Kyokai (long-term) | Rust has the right model (ownership, zero-cost abstractions, strong types) but horrible syntax, slow compilation, and bad async. Kyokai will take Austral's linear type system and make it daily-drivable.                       |
| Unsafe / low-level    | C (when needed)                                           | Zig                | C is only popular because it was made early. Zig is my C replacement — better allocators, no hidden control flow, comptime over macros. But Zig still expects competence for safety and has runtime-check overhead I don't love. |
| Scripting             | Lua (embedded), Fish (interactive), Bash (system scripts) | Same               | Fish is fast with the feature sets I need; Zsh with plugins is slow as shit. I don't script in Fish — Bash handles that.                                                                                                         |
| Refuse                | Python                                                    | Python             | Syntax sucks, versioning is insane, dependency management gives me an aneurysm. There are faster, better languages. I'll still use it if software I need is written in it.                                                       |
| Neutral               | Go                                                        | Go                 | Mid language. GC disqualifies it for systems work (the JVM punished me). Fine for what it does.                                                                                                                                  |

**Why safe languages:** I come from the aviation and robotics sphere. If your code stops working correctly, things are _done_. Software should prevent memory issues because memory is a finite resource and memory-related bugs are stupid. The language at compile time should handle everything it needs to make the software work correctly as written, and disallow compilation if requirements aren't met.

**Why not C:** C is only popular because of an early start. If a safer C had existed at the same time, everyone would have used that. C in most programs has UB, the syntax is non-explicit, and it's verbose in the worst ways. Kyokai keeps a C emit target solely for portability.

### 6.2 Tool Defaults

| Category             | Default                                                                               | Justification                                                                                                                         |
| -------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Build system         | Cargo (Rust), Meson (C/C++)                                                           | First-party, deterministic, well-documented                                                                                           |
| Init system          | Init-agnostic (dinit preference)                                                      | No init assumptions baked into software. Distros like Void, Chimera, Gentoo, Artix prove init-agnosticism is practical.               |
| Package format       | Elda-native definitions                                                               | Profile-driven composition, explicit metadata, security-first                                                                         |
| GPU API              | wgpu (Vulkan/Metal abstracted)                                                        | Cross-backend, Rust-native, explicit resource management                                                                              |
| Database             | Purpose-matched (SQLite for embedded, Postgres for service, HelixDB for graph+vector) | No "one database for everything"                                                                                                      |
| Toolchain            | clang/gcc interchangeable                                                             | Both work. I've swapped them many times and don't lose sleep over it.                                                                 |
| Privilege escalation | doas (Yoka default)                                                                   | Simpler config for bootstrapping. sudo's LDAP support is nice but 177k LOC of C is not.                                               |
| WM/Compositor        | Hyprland                                                                              | Named special workspaces double my workspace count. Shader system, performance. Not opposed to switching.                             |
| Editor               | Zed + Neovim                                                                          | Everything should be native and as fast as possible. Zed for gpui and AI integration. Neovim for muscle memory. Helix is interesting. |

---

## 7. On External Criticism

Most criticism of software is correct in the abstract and useless in practice. Yes, systemd violates Unix philosophy. Yes, the AUR has security concerns. Yes, Electron is wasteful. Yes, Python is slow. None of that helps unless the critic can articulate what to do instead, given the constraints the builder actually faces.

I will engage with criticism that meets these criteria:

1. **Specific**: Points to a concrete behavior, not a vibes-based objection.
2. **Evidenced**: Demonstrates the problem with data, reproduction steps, or code references.
3. **Constructive**: Offers an alternative that is compatible with my constraints.
4. **Credible**: Comes from someone who has built or deeply understood systems at this level, or has real experience using the software being criticized.

Someone who hasn't built X can still have a valid opinion about X — as long as they have experience _with_ X and relevant correlating experience. Criticism that does not meet these criteria is noise, and I will ignore it without guilt.

---

## 8. Political and Licensing Stance

Software is inherently political because humans created it and humans are inherently political. Pretending otherwise is intellectually dishonest — that's why licenses exist.

### 8.1 Copyfarleft

I am not "far left" in the conventional sense. I am copyfarleft: anticapitalist, aligned with the Free Software Foundation's principles, and opposed to corporations extracting value from the commons without reciprocation. Most permissive licenses were designed — intentionally or not — to enable that extraction. AGPL is the closest tool I have to enforce reciprocity, even though I haven't found a copyfarleft license I fully agree with yet.

Torvalds' GPL-2-only stance allowed Google to gut Android and OEMs to lock bootloaders. He treats Android as a Linux victory when it isn't. He allows binary blobs from corps into the kernel. The "it won" argument is not a technical argument — Android "won" too, and I still hate it.

### 8.2 On Building for Fun

You can just build for fun. Not everything needs to be popular. Not everything has to obey X and Y philosophy. Build to learn and build for fun sometimes. The Linux community needs to understand this.

---

## 9. Versioning

This document follows the same versioning scheme as `CODE_STANDARDS.md`:

- **MAJOR**: Fundamental principle added or removed.
- **MINOR**: Existing principle refined, new section added.
- **PATCH**: Clarification, typo, formatting.

---

## Changelog

### v2.0.0 — 2026-03-25

- Added axioms: Build What's Needed (2.7), Intellectual Honesty Is Non-Negotiable (2.8).
- Reframed Simplicity axiom (2.4) from reliability feature to "simplicity is a myth" — measure extensibility and maintainability instead.
- Added engineering principles: Functional Performance (3.4), Core-Style Architecture (3.5).
- Expanded Language Preferences into Language Ladder with future migration paths (Kyokai for Rust, Zig for C).
- Added Political and Licensing Stance section (8) with copyfarleft position.
- Updated tool defaults with privilege escalation, WM, and editor preferences.
- Cross-referenced QUESTIONS.md for detailed positions on controversial topics.
- Synthesized from answered QUESTIONS.md responses.

### v1.0.0 — 2026-03-25

- Initial codification of existing engineering philosophy.
- Synthesized from `CODE_STANDARDS.md`, `kyokaiplan.md`, `elda-critique.md`, project audits, and lived practice.
