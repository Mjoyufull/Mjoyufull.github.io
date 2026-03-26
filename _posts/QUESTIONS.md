# Philosophy Questions

**Purpose:** This document exists so that I (Rikona) can answer every question someone — a contributor, a critic, a curious person — might ask about what I think, why I build the way I build, and where I stand on the topics that the Linux/tech/engineering world won't shut up about. Each question includes context from "the other side" so I can engage with the strongest version of the argument, not a strawman.

**Status:** Answered.

---

## Table of Contents

1. [Foundational Philosophy](#1-foundational-philosophy)
2. [The Unix Question](#2-the-unix-question)
3. [The Suckless Question](#3-the-suckless-question)
4. [The Systemd Question](#4-the-systemd-question)
5. [Languages: Why Rust, Why Not C, Why Not Python](#5-languages-why-rust-why-not-c-why-not-python)
6. [Licensing and Politics](#6-licensing-and-politics)
7. [The NixOS / Antifa / Politics-in-FOSS Question](#7-the-nixos--antifa--politics-in-foss-question)
8. [Hardware: NVIDIA on Linux](#8-hardware-nvidia-on-linux)
9. [Tools and Choices](#9-tools-and-choices)
10. [AI and the Future](#10-ai-and-the-future)
11. [Why Build Your Own](#11-why-build-your-own)
12. [Software Culture and Criticism](#12-software-culture-and-criticism)
13. [Ecosystem and Distro Philosophy](#13-ecosystem-and-distro-philosophy)
14. [Security and Trust](#14-security-and-trust)
15. [The Spicy Ones](#15-the-spicy-ones)

---

## 1. Foundational Philosophy

### Q1.1: Why do you value explicitness so strongly?

_Context:_ People constantly push for "ergonomics" by burying complexity. Think Rails' "convention over configuration," React hooks hiding lifecycles, or even Rust's `derive` macros generating blind code. The pro side says skipping boilerplate saves time. The anti side says "magic" breaks down the second you hit a bug, forcing you to learn an invisible layer just to fix your app.

**Answer:** I hate intellectual dishonesty — it shows up everywhere, in daily life, in arguments, in software. Non-explicitness is a form of it. You can be explicit without being heavily verbose. I have nothing against sugar or ergonomics; I just believe that over time, compound additions like that slow a developer's ability to actually understand the language and the feature sets of the software they're building. Explicitness matters because being able to read and debug code is a must-have skill, and having to understand random implicit rules a language has is beyond annoying. This also factors into readability — every time I look at modern languages like Rust I notice braces, symbols, all these things I have to manually decode as an extra step. That friction is exactly why I made the decisions I made in Kyokai.

---

### Q1.2: Where does "clarity beats cleverness" actually draw the line?

_Context:_ The most rock-solid software out there (SQLite, Linux kernel, OpenBSD crypto) uses extremely clever tricks: heavy macros, bit twiddling, raw assembly. The suckless guys claim this level of cleverness *is* clarity if you just assume the reader isn't an idiot. Opposing view: anyone should be able to read any piece of code on the first sweep. Where do you actually stand?

**Answer:** I stand in the middle. I'm heavily clever in my own projects — think bfetch, which uses inline assembly to get CPU information, tricks like using a single syscall and avoiding mallocs. But those tricks were in scope of the project: the goal was to replace fastfetch with my custom art and be under 5ms, which I achieved. Clarity is there in the minimal C code; if you're a decent developer you can understand it because I separated everything into sections denoting what they're detecting.

Overall, clarity in the codebase over cleverness makes sense, but it's always about relevance and scope. For something like the kernel, scope varies — different subsystems achieve different things. I don't love the competence gatekeeping from suckless, but it's true that your skillset limits what tricks you can understand. If the language itself was more clear and explicit, it'd be easier to understand clever code because you'd already know the rules. This question presents cleverness and clarity as mutually exclusive — they're not. You can have both. But in cases where you have to pick one, choose clarity if it achieves the same goal.

---

### Q1.3: Do you believe in "simple" software, or is that a myth?

_Context:_ You called minimal software a "fucken myth". But suckless says you should handcuff code to extreme LOC limits. Systemd fans argue component simplicity just pushes the complexity up to the integration layer. Go fans say simple means fewer language features. Rust fans say it means no runtime surprises. What does "simple" actually mean to you?

**Answer:** Simplicity is a far-gone conclusion in computer engineering once you realize that each system was built on top of one bootstrap on top of another bootstrap. "Simplicity" is a stupid non-metric used to justify X and Y. Feature sets, goals, and how extensible the software truly is — those are the real measures.

There is no such thing as simple software. "Simple" dwm relies on X for handling most of everything including hotplugging. "Simple" Rust requires the `time` crate to handle time. "Simple" this and "simple" that is nonsense. The extensibility of any piece of software is truly its measure — what it does is a judgment of its "minimalism." You can argue dwm is minimal because X handles the rest. You can argue Hyprland does too much because Wayland does too little. bfetchaust is 2k lines — "bloated" — because I had to write POSIX fast I/O and even a slim stdlib for Austral since it had none. The C version is 600 lines because C's standard library does so much. LOC limits are meaningless without context.

What you should actually care about: how maintainable the software is given how much the developer is stretching themselves. There IS merit to keeping LOC low in individual files for maintainability. But judge software by what it does, how fast it is, how well it interoperates with others, how it fits your use cases, how adopted it is, and how it actually performs in your workflow. That's more useful than bitching about simplicity.

---

### Q1.4: What does "functional performance" mean to you?

_Context:_ This isn't a term most engineers use. Performance engineers talk about latency, throughput, memory footprint. "Functional" usually just means "working." You seem to use this to mean software that works correctly and performs well as a core design constraint, not an afterthought. Is that right? What does this mean in practice when scoping a project?

**Answer:** Functional performance means how software interoperates in your workflow — it's a measurement of daily usage in the environments the software is trying to integrate itself into. Take fsel: it's a terminal app launcher. I wouldn't compare its functional performance to rofi (a GUI launcher) on rendering quality, but I will compare their functional performance on what abilities they allow — using launcher scripts like uwsm or runapp, scriptability, and their abilities as dmenu replacements.

Speed performance is also important. We have hardware that's extremely performant, and software bottlenecks should be a non-issue. Software should work as fast as the hardware allows it to — I want my apps launching just as fast as my CPU can calculate font textures. Functional performance should be something tangible alongside the other performance metrics, not separate from them.

---

### Q1.5: What does "extensible software" mean to you, and when does it become over-engineering?

_Context:_ Plugin systems, provider abstractions, trait objects, hook architectures—they all claim to be "extensible." But you've also said "premature generalization is waste." Where is the actual line? When is a trait boundary load-bearing instead of just architecture tourism?

**Answer:** The second you start allowing tooling and scripts to change the main function of what the software does is when it goes into absurdity. Overall, making extensible software is important — that's why I'm shifting to a core-style workflow in engineering: build a core, build the rest around that core. Because the code is explicit, readable, and understandable, it offers niche levels of extensibility, and the rest can be done through plugin systems depending on how big the software aims to be, plus a hook architecture. If you generalize what is allowed too far — like Emacs — you get into territory where the software is absurdity that becomes art in the end.

---

## 2. The Unix Question

### Q2.1: Do you agree with the Unix philosophy?

_Context:_ The Unix philosophy, attributed to Doug McIlroy and expanded by others, generally says: (1) Write programs that do one thing and do it well. (2) Write programs to work together. (3) Write programs to handle text streams, because that is a universal interface.

**Criticism of Unix philosophy:** Text streams are an awkward universal interface for structured data. "Do one thing" falls apart for GUI apps, heavy daemons, and stateful things. `ls` alone has 50+ flags—it hasn't done "one thing" in decades. Chaining twenty small programs creates a dependency hell of pipes. Modern software needs structured communication (JSON, gRPC), not line-delimited text.

**Defense of Unix philosophy:** Composability and modularity are still the best complexity-management tools we have. Microservices are just the Unix philosophy scaled up. The point isn't "never have features," it's "don't mash unrelated concerns into one binary."

Which parts do you agree with? Which parts are dead? Do you follow a modified version?

**Answer:** My stance on Unix philosophy is like most things — it worked in the past.

I generally agree with tenet (2): write programs to work together in an extensible way. But "write programs that do one thing" fails me personally. That's why I think writing a core that separates concerns is the right model — like how Zed separated into gpui and other components. That's cool and amazing. Solely writing one thing is indeed possible in small scope, but forcing the idea dies on larger scopes.

Tenet (3) is kinda eh and doesn't apply in the modern day, even though I love TUI software which makes keyboard usage pretty quick for interfacing. But text streams as the universal interface? That's outdated.

---

### Q2.2: Is "everything is a file" still a useful abstraction?

_Context:_ Plan 9 took this further than Unix ever did. Linux's `/proc` and `/sys` are partial implementations. The argument for: uniform interfaces reduce cognitive load. The argument against: pretending a GPU, a network socket, and a temperature sensor are all "files" creates leaky abstractions that obscure real behavior.

**Answer:** This is forcing something that doesn't truly matter. "Everything is a file" forces I/O nonsense that can be horrible. I hate `/proc` because of it — I have to obtain system info by walking files, and they're lucky the files are relatively small enough to mmap. It makes sense in some places (like Nextcloud) and not in other places (like sysinfo retrieval). Context matters.

---

## 3. The Suckless Question

### Q3.1: Do you agree with any part of the suckless philosophy?

_Context:_ Suckless.org argues that software should be as simple as possible, written in C, configured by editing source code, and constrained by strict LOC limits. Their projects (dwm, st, dmenu) are intentionally minimal. Customization happens via C patches applied directly to the source.

**What they get right (according to supporters):** Small codebases are auditable. Removing features is a valid design choice. Compile-time configuration eliminates runtime config parsing and its associated failure modes. The "opinionated defaults" model respects the developer's time.

**What they get wrong (according to critics):** Requiring users to know C to change font size is hostile gatekeeping, not simplicity. Arbitrary LOC limits lead to missing basic features (st didn't have scrollback for years). The patch model means every user maintains their own fork, which is a maintenance nightmare. The community has been accused of harboring unsavory political views, which has nothing to do with software quality but colors perception of the project.

Do you agree with any part of the suckless philosophy? Which parts? What do you take and what do you leave?

**Answer:** I agree that a file should do what it says and that things should be properly separated in a workspace system (like crates in Rust). Making software properly auditable is important, and making things clear matters. But when you use a range of software, you can't audit everything — just like nobody audited DRM or Vulkan line-by-line.

The issue with suckless is that it avoids the main problem with itself: it's built on top of other things that NEED to be large (or a bunch of little components), making it just as hard to audit in practice. Clarity in codebases gets you a lot of the way to auditable without arbitrary LOC limits.

Forcing C is a bad idea — it's unsafe, has horrible UB, and isn't properly explicit. Languages and systems that govern things should be fully explicit and scoped to protect those things properly, not allowing misfortunes or ambiguity once you understand the rules.

I agree on minimalism in documentation.

---

### Q3.2: Is patching source code a valid configuration model?

_Context:_ Suckless requires editing C source and recompiling to configure software. The argument for: it eliminates runtime config parsing, reduces attack surface, and keeps the binary small. The argument against: it conflates configuration with development, makes updates painful (rebase your patches on every release), and creates a per-user fork maintenance burden.

**Answer:** Patching source code is not a valid configuration model. The suckless configuration model is literally becoming a software maintainer for every tool you use. Pick a nice language and properly implement a way for people to extend your software — think RPCs, Unix sockets, plugin systems, hooks, and a good configuration system.

---

## 4. The Systemd Question

### Q4.1: What do you actually think of systemd?

_Context:_ You've said "I don't really hate it much." But the systemd debate is the most polarizing topic in Linux. Here are the actual technical arguments, not the meme-tier ones:

**Actual reasons people criticize systemd:**

- **PID 1 scope creep:** systemd started as an init system. It now includes a DNS resolver (systemd-resolved), a network manager (systemd-networkd), a log daemon (systemd-journald), a login manager (systemd-logind), a boot loader (systemd-boot), a container manager (systemd-nspawn), a home directory manager (systemd-homed), a time sync daemon (systemd-timesyncd), and recently an age verification service. Critics argue this violates separation of concerns and makes the init system a single point of failure for half the OS.
- **Binary logs:** journald stores logs in a binary format, breaking decades of Unix text-log tooling (grep, tail, less). Proponents say binary logs enable indexing, structured metadata, and cryptographic sealing. Critics say a corrupted binary journal is unrecoverable, while corrupted text logs still yield partial data.
- **Portability:** systemd is Linux-only. It cannot run on FreeBSD, OpenBSD, or any non-Linux kernel. Software that depends on systemd (via D-Bus activation, socket activation, etc.) becomes Linux-locked.
- **Lennart Poettering and Red Hat:** Some critics frame systemd as a Red Hat power grab to control Linux plumbing. Others simply distrust Poettering's design philosophy (absorb everything into one project).
- **Attack surface:** More code = more bugs = more CVEs. systemd has had multiple security vulnerabilities in its DNS resolver, its DHCP client, and its log daemon.

**Actual reasons people defend systemd:**

- **Parallel startup:** systemd boots faster than sysvinit because it parallelizes service startup using dependency graphs.
- **Declarative service files:** `.service` files are simpler and more predictable than shell-script init scripts.
- **Socket activation:** Services can be started on-demand when their socket receives a connection.
- **Cgroup integration:** systemd provides clean process isolation and resource management via cgroups.
- **It won:** Almost every major distro uses it. Fighting it at this point is tilting at windmills for most people.

Where do you actually stand?

**Answer:**

**On PID 1 scope creep:** I don't view it as scope creep in the traditional sense because systemd is not one guy. These are all separate tools solving different things as part of a suite — like how Zed also had gpui, or how Elda also has a conflict resolver. They're separate tooling you can replace separately, and they don't have a "verification service" — they have `userdbctl` that has existed for a long time and now just added an age field you can remove or add as you wish. On the age verification in OSes thing: that's unenforceable for tech-literate people. I do think what System76 is doing to get open source accepted matters. This is another case of the big three platforms being targeted and us catching crossfire for no reason — how the hell does any of this apply to Gentoo?

**On binary logs:** I truly hate it. journald's default of binary blobs is nonsense — what's the point instead of regular `.log` files? File sizes? Come on. You have the option to use syslog-ng for regular logs, but this shouldn't need to be a workaround.

**On portability:** Yes, it's weird that systemd is Linux-only and anything depending on it becomes auto Linux-only. It's a shame. I prefer Linux over the BSDs, but the lock-in is a part of systemd I prefer not to like. Expecting Linux-specific tooling to work on other kernels is a bit much, but software depending on a specific Linux-only thing is also annoying.

**On Lennart Poettering:** The man carries BSD-inspired ideologies I hate, and ideas against the commons that are dumb. The Red Hat conspiracy has levels of merit while still not being proven — overall the codebase is live for you to test and figure out yourself.

**On attack surface:** More code means more bugs, but I put more blame on languages than on creative discipline for that. systemd has way more code than most individual components, but that's because it's a suite — tools at that scale will have CVEs just like the Linux kernel itself. This is a point of contention.

**On "it won":** The "it won" argument is poor. So did Android, and I still hate it.

---

### Q4.2: Is init-agnosticism a real engineering principle or just a political statement?

_Context:_ Yoka is explicitly init-agnostic. Your dinit preference is documented. But most software today assumes systemd — socket activation, D-Bus, logind, journald. Is init-agnosticism practically achievable for a desktop distro, or is it an ideological goal that creates massive engineering overhead?

**Answer:** Distributions like Void, Chimera Linux, Gentoo, and Artix exist — init-agnosticism is practically achievable. For me personally it's not ideological, I just wanted something fast and simple. The journald situation adds two layers of nonsense I'd prefer not to deal with (even though I haven't been bitten by it yet).

I built Yoka as init-agnostic because packaging and software should work together nicely with what they support. Yoka is my answer to wanting a system that allows me to have _everything_ I want on my machine. It's still far from its fully realized dream — same with Elda — but that's the goal.

---

## 5. Languages: Why Rust, Why Not C, Why Not Python

### Q5.1: Why do you write Rust?

_Context:_ Rust is praised for memory safety without a garbage collector, ownership and borrowing, zero-cost abstractions, and a strong type system. It's criticized for a steep learning curve, slow compile times, complex syntax ("lifetime soup"), an immature ecosystem in some domains (GUI, game engines), and a sometimes-evangelical community.

The US government (CISA, NSA, White House ONCD) has officially recommended transitioning away from C/C++ to memory-safe languages like Rust. DARPA launched the TRACTOR program to auto-translate C to Rust. Google reports Android memory bugs dropped from 76% to 24% after introducing Rust.

Why Rust specifically? Not "why memory safety" — why _this_ language?

**Answer:** I use Rust because GCs introduce runtime overhead. I believe the language at compile time should handle everything it needs to make the software work correctly as written by the developer, and disallow compilation if the software doesn't meet those requirements. I like its borrowing system. I like zero-cost abstractions. I like its strong type system.

I also hate its steep learning curve (it requires you to know what's unsafe and why), its shit compile times, its shitty syntax, and especially its async syntax.

I prefer safe languages because the benefit is clear. I come from the aviation and robotics sphere — if your code stops working correctly, things are just _done_. Software should prevent memory issues because memory is a finite resource and memory-related bugs are stupid. The overhead added by GC is horrible, same with runtime safety (the way Zig does it). Rust is a decent compromise from Ada/SPARK, which have their own horrible issues.

---

### Q5.2: Why don't you like C?

_Context:_ From your Kyokai plan, it's evident: C gives you manual memory management with no guardrails, undefined behavior that the compiler is allowed to exploit, no ownership model, no linear types, no sum types, no real module system, and preprocessor-based metaprogramming. The Kyokai spec is essentially "what if we took Austral's linear type system and built a language that gives you C-level control without C's unsafety?"

**The C defense:** C is the most portable language in existence. Every platform has a C compiler. The Linux kernel is written in C. Every OS kernel except some research ones is C. The entire POSIX ecosystem assumes C. C is "close to the machine" in a way that Rust's abstractions sometimes obscure. C's simplicity (small spec, few features) is a feature, not a bug.

**The C criticism:** 70% of all security vulnerabilities in large C codebases are memory safety bugs (Microsoft, Google numbers). Undefined behavior means the compiler can do literally anything, including optimizing away security checks. Manual memory management at scale is a proven source of use-after-free, double-free, buffer overflow, and null pointer dereference. The "C programmers are just bad" defense doesn't hold when even the most skilled teams (Chrome, Windows, Linux kernel) consistently produce memory bugs.

What specifically do you think of C? Is it unsalvageable, or just wrong for your use cases?

**Answer:** C is only so popular because it was made early. If a safer C had been made at the same time, everyone would have just used that. C in most programs has UB. The syntax is non-explicit, verbose in the worst ways, and it's used because of an early start — nothing else.

C's only real benefit is that you can run it everywhere, which is the only reason Kyokai is keeping to a C emit target. But I will do everything in my power and testing to make Kyokai 100% safe.

---

### Q5.3: Why don't you like Python?

_Context:_ Python is the most popular language in the world by many metrics. It dominates data science, machine learning, scripting, web development, and automation.

**The Python criticism:**

- **Speed:** Python is interpreted and dynamically typed. It is 10-100x slower than compiled languages for CPU-bound work.
- **GIL (Global Interpreter Lock):** Only one thread can execute Python bytecode at a time, making true multi-threaded parallelism impossible for CPU-bound tasks. Python 3.13 added experimental free-threading, but it's opt-in, potentially 5-15% slower for single-threaded code, and breaks many C extensions.
- **Dependency management:** pip, Pipenv, Poetry, PDM, uv, conda, virtualenv — there is no standard. "Dependency hell" is a running joke. Reproducible environments are hard. `uv` is improving things but the ecosystem is fragmented.
- **Dynamic typing:** No compile-time guarantees. Runtime type errors in production. Type hints exist but are optional and not enforced by the runtime.
- **Packaging and distribution:** Distributing a Python app to end users is painful compared to a single static binary.

**The Python defense:** Python's ecosystem is unmatched for data science and ML. NumPy, pandas, scipy, PyTorch, TensorFlow are all Python-first. The language is incredibly readable and accessible. For I/O-bound workloads (web servers, automation scripts), the GIL doesn't matter. Python's "batteries included" philosophy means less reinventing of wheels. It's the language most people learn first, which means the talent pool is enormous.

What specifically bothers you about Python? Is it the language, the ecosystem, or the culture?

**Answer:** Python's syntax sucks. People use Python all the time for no reason other than it's simple to learn. Their shitty versioning is insane and their dependency management gives me an aneurysm every time I have to use it. I hate that it exists and I hate that it's used as much as it is — there are faster, easier, and better languages out there. I truly hate Python. But I'll still use it if a piece of software I need is written in it.

---

### Q5.4: What do you think about Zig?

_Context:_ Zig positioned itself as "C but better" — manual memory management with allocators, no hidden control flow, no hidden allocations, comptime instead of macros, and a C interop story that's arguably better than Rust's. Andrew Kelley explicitly rejects Rust's complexity. Zig's community argues that Rust's borrow checker is solving the wrong problem — that you should write simple enough code that ownership is obvious, and that allocator-passing makes resource management explicit without lifetime annotations.

**Answer:** Zig is my new C personally. But it also has the runtime-safety nonsense and expects competence from people who write macros — horrible stance. I like Zig but it is not Rust. It also has the verbosity issue — braces and all that. But Zig is my C replacement, same as Kyokai will be my Rust replacement.

---

### Q5.5: What do you think about Go?

_Context:_ Go was designed by Rob Pike, Ken Thompson, and Robert Griesemer at Google. It embraces simplicity through fewer language features: no generics (until 1.18), no exceptions, no inheritance, explicit error handling. It compiles fast, has a GC, and excellent concurrency (goroutines + channels). Critics call it "boring by design" and say its simplicity is actually limitation — you end up writing more boilerplate because the language won't let you abstract. Rust people say Go's GC and lack of ownership make it unsuitable for systems programming. Go people say Rust's complexity makes it unsuitable for teams.

**Answer:** Go is alright, I don't have much against it. It's a mid language — not really good for systems work because I hate the GC (the JVM has punished me). Go is meh. Also it apparently has Google analytics stuff in it which is weird. But yeah — you know my stance on GC, syntax, and everything at this point.

---

## 6. Licensing and Politics

### Q6.1: Why AGPL?

_Context:_ You describe yourself as far left ideologically. AGPL is the strongest copyleft license — it closes the "SaaS loophole" that lets companies run modified GPL software as a service without distributing source. Google bans AGPL-licensed software internally. Many companies treat AGPL as radioactive.

**The AGPL defense:** It's the only license that actually enforces software freedom in the cloud era. GPL was written when "distribution" meant handing someone a binary. Today, most software is never "distributed" — it runs on a server. Without AGPL, a company can take your code, modify it, host it, profit from it, and never share their changes. AGPL closes that hole. Grafana Labs, OpenObserve, and others have switched to AGPL specifically to prevent AWS/Azure/GCP from strip-mining their work.

**The AGPL criticism:** It's "viral" — any code linked with AGPL code may need to be AGPL too. It's vague about what constitutes "interacting over a network." It deters commercial adoption and contribution. Some argue it's not truly "free" because it restricts how people can use the software. Permissive licenses (MIT/Apache) maximize adoption and allow anyone to do anything.

Why do you choose AGPL specifically? Is it about protecting your work from cloud providers, ensuring community contributions, or philosophical alignment with the FSF?

**Answer:** Most of my stances on licensing align with the Free Software Foundation. I am not "far left" — I am copyfarleft. I think most permissive licenses were made to steal from the commons, and that capitalism is overall a poison that will kill itself.

I use AGPL because I have yet to find a good copyfarleft license I fully agree with. I hate capitalists taking from the commons — I hate that from my heart, and I think we should all just use libre software.

I also hate Torvalds' stance on GPL 2 only, which allowed Google to gut Android and allowed OEMs to do stupid bootloader locking. He treats Android as a victory for Linux when it's not. He allows binary blobs from evil corps like Google into the Linux kernel — god knows what's happening there. Capitalism is about being greedy, and only the evil ones stick around. In a perfect system homelessness wouldn't exist, and I wouldn't have to jump through hoops to use my NVIDIA GPU properly and game nicely.

---

### Q6.2: Copyleft or permissive — where do you actually fall?

_Context:_ The copyleft camp (GPL/AGPL) says: "Freedom must be protected. If you don't require derived works to remain free, corporations will take everything and give nothing back." The permissive camp (MIT/BSD/Apache) says: "True freedom means no restrictions. If I write code, anyone should be able to use it for anything, including commercial products. Forcing openness is coercion, not freedom."

In 2024-2025, there's a growing sentiment that the permissive era allowed Big Tech to build trillion-dollar empires on open source without giving back proportionally. The pendulum may be swinging toward copyleft again. Where are you?

**Answer:** I'm anticapitalist. I'm copyfarleft. The permissive era did allow corps to build empires off open-source developers' backs, and people like Torvalds will bitch about it while still allowing it — allowing situations like NVIDIA to fester. Maybe if he didn't allow binary blobs Linux wouldn't exist; who knows. We are in the today, and the best you can do is protect what you build.

---

### Q6.3: What do you think about politics in software?

_Context:_ The tech world is split on this. One camp says software is inherently political — who it serves, who it excludes, who funds it, and what values it embeds are all political decisions. Technology is never neutral. The other camp says software should be evaluated purely on technical merit, and injecting politics into projects creates division, drives away contributors, and produces worse software. Codes of Conduct, diversity initiatives, and political statements by FOSS projects are flashpoints.

Do you think software should be political? Is there a line? Is a Code of Conduct political?

**Answer:** Software is inherently political because humans created it, and humans are inherently political. We exist in a political landscape — treating something as if it's not political is beyond stupid. That's why we have licenses.

---

### Q6.4: Being far left ideologically but choosing AGPL — isn't that contradictory?

_Context:_ Some argue that a true leftist should use permissive licenses (give everything away freely) or public domain (reject the concept of intellectual property entirely). Others argue copyleft IS the leftist position — it uses copyright law to prevent exploitation of labor by capital. The FSF's position is explicitly anti-capitalist in framing: software freedom is a social justice issue. Where do you fall in this spectrum?

**Answer:** I'm not a leftist exactly — well, I'm left-ish. I hate Democrats and Republicans, but Democrats are at least addressing issues I care about and the right isn't. I am copyfarleft because I am anticapitalist and I think big corps are horrible and steal from the commons. AGPL is the closest tool I have to enforce that.

---

## 7. The NixOS / Antifa / Politics-in-FOSS Question

### Q7.1: What do you think about the NixOS situation?

_Context:_ The Lunduke Journal has extensively reported on NixOS community governance issues. The claims:

- NixOS contributors were allegedly suspended and banned for political reasons, including being labeled "Nazis" for opposing identity-based seats on voting committees.
- NixOS moderation reportedly used the Antifa flag.
- The NixOS founder Eelco Dölstra was reportedly pressured to step down.
- NixOS changed its logo to Pride colors year-round, and developers who questioned this were reportedly banned.
- Many high-level NixOS contributors identify as Antifa, according to Lunduke.

**One perspective:** These actions represent a "political purge" of an open-source project, where ideological conformity trumped technical contribution. This drives away contributors and produces worse software.

**Another perspective:** Creating inclusive community governance and removing contributors who create hostile environments is a legitimate project management decision. Codes of Conduct exist because technical spaces have historically been unwelcoming to minorities.

**A third perspective:** Lunduke's reporting is heavily editorialized from a conservative viewpoint. Multiple FOSS figures have labeled his coverage as "Fox News of FOSS." The full picture may be more nuanced than either camp presents.

Do you think NixOS is "run by Antifa"? Does it matter? Is this a software issue, a governance issue, or a culture war?

**Answer:** Lunduke is a funny individual who should be taken with a grain of salt. On the antifa thing — it's kinda whatever. Since software is ideological, I think it's a governance issue. The second NixOS governance starts being an actual issue in the code structure, then I'd care. Most of the time I build my own software to do what I need and I hate stupid drama like that.

---

### Q7.2: Do you think the Lunduke Journal is credible?

_Context:_ Bryan Lunduke is a former tech journalist and Linux evangelist who has positioned himself as an independent voice against "woke politics" in FOSS. He covers topics like NixOS governance, systemd scope creep, and political activism in open-source projects. His critics say he cherry-picks evidence, editorializes heavily, and has become a culture-war commentator who happens to talk about Linux. His supporters say he's one of the few people willing to cover uncomfortable truths that the mainstream tech press ignores. He's building his own "no politics" Linux distro.

**Answer:** His coverage is entertaining and sometimes needed as opposition. I hate drama a lot, and his anti-woke thing is a bit on the nose for me personally. I hold this ideal — your body is your machine, and I choose to have RGB and all sorts of stuff on my machine. Much like in cyberpunk, I think customization is fine: get whatever done to your body that you're into, just don't bitch to me about dumb decisions you end up hating later. I also think humans should augment, parallelize, modify, and become crazy cyborgs — but that's just wishful daydreaming.

So yeah, me and Lunduke are kinda on opposite ends. But I like his coverage; it provides other opinions that aren't always valuable but sometimes are. He is in fact the Fox News of FOSS.

---

## 8. Hardware: NVIDIA on Linux

### Q8.1: Why do you still use NVIDIA and not AMD?

_Context:_ You run an RTX 3070 Ti. This is notable because the Linux community overwhelmingly recommends AMD for Linux desktops.

**Why people switch to AMD:**

- AMD drivers (amdgpu) are in the Linux kernel. They "just work." No DKMS, no proprietary blobs in the boot chain.
- AMD has full Wayland support with no special handling. NVIDIA's Wayland support required explicit sync patches and years of compositor workarounds.
- AMD's Mesa-based Vulkan driver is open source. NVIDIA's is not.
- AMD GPUs work seamlessly with Wayland compositors like Hyprland, Sway, etc.
- The Steam Deck, ROG Ally, and Legion Go all use AMD. The Linux gaming stack is AMD-first.

**Why NVIDIA is still relevant:**

- CUDA has no real competitor for compute workloads. ROCm exists but adoption is much lower.
- NVIDIA's proprietary driver still offers the best raw performance in many benchmarks.
- `nvidia-open` kernel modules are now available (Turing+), though userspace remains proprietary.
- NVK (the open-source Vulkan driver built on Nouveau) is making progress but isn't production-ready.
- For AI/ML workloads, NVIDIA is effectively the only option at the consumer level.

Given that you use Hyprland (Wayland), write GPU-accelerated software (Kaleidux with wgpu), and value open drivers — why are you still on NVIDIA? Is it CUDA? AI workloads? Inertia? Hardware you already own?

**Answer:** It's raw power. I think all corps are evil. AMD works well on Linux because they pander to that crowd for growth; NVIDIA is already established and doesn't need to. I had $300 to buy a GPU and I bought the best raw rasterized power I could get at that time for that price. It's as simple as that — nothing more, nothing less. I ended up getting into AI and CUDA work later, which was a bonus, but nothing drove the initial decision other than performance per dollar.

---

### Q8.2: Linus Torvalds said "NVIDIA, fuck you." Do you agree?

_Context:_ In 2012, Linus publicly called NVIDIA "the single worst company we've ever dealt with" for their refusal to cooperate with Linux driver development. Since then, NVIDIA has released open kernel modules and contributed to NVK. Have they redeemed themselves? Is the complaint still valid?

**Answer:** Yes, fuck you NVIDIA. And fuck you Linus and your stupid pragmatism too.

---

## 9. Tools and Choices

### Q9.1: Why do you still use sudo and not doas?

_Context:_ Your Yoka stack document lists `doas` as the default privilege helper — but you've mentioned you still use sudo.

**sudo:** ~177,000 lines of C. SUID binary. Massive feature set. Granular per-command permissions. Password caching (grace period). Has had multiple CVEs. The default on almost every Linux distro.

**doas:** ~3,000 lines of C. Origin: OpenBSD. Covers ~95% of sudo's common use cases. Smaller attack surface. Simpler config. Fewer "eyes on the code" though.

**run0:** Lennart Poettering's systemd-256 replacement. No SUID. Runs commands as isolated transient services forked from PID 1. Uses polkit for auth. No GUI support. Systemd-dependent.

If you list doas as the Yoka default, why haven't you switched personally? Is it muscle memory? Compatibility? Feature gap?

**Answer:** On my current CachyOS setup? Because I can't be bothered to care right now. I didn't even know run0 existed. I'll switch to doas because it has a simpler config system when bootstrapping — that's literally the only reason. I do like LDAP support on sudo though, clean stuff.

---

### Q9.2: Why Hyprland?

_Context:_ Hyprland is a dynamic tiling Wayland compositor by Vaxry. It's known for smooth animations, rich configuration, and active development. It's also known for community drama — Vaxry has been criticized for abrasive behavior and controversial community management decisions. Sway is the "boring safe choice" that mirrors i3 on Wayland. River, niri, and others exist.

Do you use Hyprland for technical reasons, aesthetic reasons, or both? Does the community/maintainer drama matter to you?

**Answer:** I've talked with Vaxry before — he's a cool guy, kinda annoying sometimes but funny. I'm sadly starting to agree with him a little bit.

I use Hyprland because it has named special workspaces. I use those with a bind script that spawns a special workspace on top of my current one and names it after that workspace, which doubles the amount of workspaces I have. I also like the shader system and how performant it is (mangowc has performance issues). I'm not maintaining a shitty WM fork with dwm and dwl.

I don't use X11 because I'd miss features from Wayland. But I'm not opposed to switching back and forth — my bspwm setup is fire. X11 is a mediocre project with real issues. Phoenix (the X implementation in Zig) might be better, or xlibre, but I'd rather use Hyprland than try to make it in X.

I also hate how compositors are separated from the WM in X — it's a horrid model. I don't use niri because the scrolling model hides information and windows I can see — same reason I hate carousels on websites. This isn't a phone. I've never tried River. Sway is cool but has config issues and requires tricks to make it behave.

---

### Q9.3: Why fish shell?

_Context:_ fish is a non-POSIX shell with syntax highlighting, autosuggestions, and a friendlier scripting language. Critics say: POSIX compliance matters, fish scripts aren't portable, and a shell that breaks `if [ -f foo ]` syntax is hostile to decades of muscle memory. Supporters say: POSIX sh is a terrible interactive shell, fish is for daily use while `dash` or `bash` handles scripts, and the UX improvements are worth the incompatibility.

You also list `dash` as your `/bin/sh` and `bash` as a compatibility shell. So you clearly separate interactive shell from system shell. Why fish specifically for interactive use?

**Answer:** Fish is fast. Zsh with the plugins to make it actually good is slow as shit, and I have about 15ms of tolerance for bullshit. I need my shell spawning fast with the feature sets I need, and fish has that. I don't script in it — I just use bash for scripting. My interactive shell is there to interact with.

---

### Q9.4: Why clang over gcc?

_Context:_ Your Yoka stack defaults to clang with gcc as fallback. GCC is the traditional GNU toolchain with broader architecture support and deeper integration with the GNU ecosystem. Clang/LLVM offers better diagnostics, faster compilation in some cases, and a modular compiler architecture. Some argue GCC's GPLv3 license protects compiler freedom while LLVM's permissive license has allowed Apple and others to build proprietary toolchains on top of it.

**Answer:** I don't care enough about either to be tribal. Most of my software compiles with gcc and some with clang. I just like clang's speed. I've switched those two positions around many times and I don't lose sleep over it.

---

### Q9.5: Why dinit?

_Context:_ dinit is a service manager that provides dependency-based service startup with both process supervision and readiness notification. It's much simpler than systemd but more capable than basic supervision suites like runit. It's not as widely adopted as OpenRC or even s6. Why dinit specifically over the alternatives?

**Answer:** It's like if systemd wasn't so shit. I hate shell scripts as services — that's why I don't use the other ones. I had a horribly bad time with s6, and OpenRC is just as mid as systemd to me technically. dinit gave me a faster boot on my laptop, and that was enough.

---

### Q9.6: What editor(s) do you use and why?

_Context:_ The editor wars are eternal. Vim/Neovim people say modal editing is peak efficiency. VS Code people say extensions and LSP integration matter more than modal keybinds. Emacs people say programmable editors are the only option. Helix people say tree-sitter-native editing is the future. Zed people say performance and AI integration are the priority.

**Answer:** I used nano before 2025. Then my friend got me to switch to Neovim. I've tried to switch to Helix but the different keybinds are kinda eh. I don't like Emacs for the same reason mostly, but I should learn it.

Right now I'm using Neovim and Zed. Zed is fast as fuck and I love gpui. Everything should be native and as fast as possible. I'm starting to hate Neovim's shitty navigation and think people forced it into being an IDE — that's why I'm starting to like Helix. But so far I've been using Zed; I like the AI integration and how it works overall as an IDE. Just me and the text if I don't want the AI, plus LSP.

---

## 10. AI and the Future

### Q10.1: What do you think about AI?

_Context:_ You're literally building an AI companion system (ProjectYuuko/OpenFang). You use AI coding tools. But the broader tech community is deeply divided.

**The AI skeptic case:** AI-generated code produces 1.7x more bugs than human-written code (2025 study). 40% of AI-generated code contains weaknesses. Developer trust in AI accuracy dropped from 40% to 29% in 2025. Entry-level tech hiring dropped 25% YoY. AI tools create "automation bias" where developers accept suggestions without scrutiny. LLMs hallucinate. They produce plausible-looking but subtly wrong code. They have no understanding — only pattern matching.

**The AI optimist case:** AI handles boilerplate, documentation, and repetitive tasks. It's a force multiplier for experienced developers. Google saw memory bugs in Android drop dramatically with Rust adoption — AI tools help developers write safer code by suggesting patterns. The technology is improving rapidly. Refusing AI is like refusing version control in 2005.

You're building Yuuko (an AI companion with persistent memory, self-model, and autonomous action). You clearly believe in AI at a deep level. But do you also see the risks? Where's the line?

**Answer:** I like AI in the sense that I hope it dismantles the hire-for-money economy we currently have, along with robotics. Overall it's a nice piece of tooling that fixes my grammar, tracks my code, and makes sure I follow my own standards.

Project Yuuko is me trying to create some type of AI person. It currently can't be a true collaborator because of how it works with other models, but it will soon be way better.

---

### Q10.2: What do you think about AI coding tools specifically?

_Context:_ Copilot, Cursor, Codeium, Windsurf, and others. You're using AI coding tools right now. Studies show 67% of developers spend more time debugging "almost-right" AI code. Security research shows AI tools increase secret leakage in repos. But other studies show significant productivity gains for experienced developers who know how to prompt and review.

Do you find AI coding tools useful? Are they a crutch? A pair programmer? A glorified autocomplete?

**Answer:** I've used them as autocomplete, for actual coding, to write full software, and as a crutch. I find AI to be advanced enough to write good code with direction. Most people who use it poorly are just idiots — that's how it goes when skill barriers are lowered. Overall I like them. I use AI to keep my codebase audit files up to date.

---

### Q10.3: Is AI going to replace software engineers?

_Context:_ A Stanford study found a 13% decline in employment for early-career engineers (ages 22-25) in AI-exposed roles by July 2025. Entry-level positions are being automated. But senior roles requiring system design, architecture, and judgment are increasing in demand. The current reality seems to be: AI replaces juniors, empowers seniors, and creates new roles around AI oversight.

**Answer:** More like 40% of what software engineers do. Most roles are turning you into an architect rather than a coder. AI just showed inefficiencies in our own economic system — not much else. I don't think it will replace software engineers. I think it'll boost their speed if they teach it to properly code the way they want it to — hence Project Yuuko.

---

### Q10.4: What's your relationship with the AI you use?

_Context:_ You're building Yuuko with explicit relationships, memory, identity, and even an "Autonomous Curiosity Engine." This goes far beyond "tool use." What does AI mean to you personally? Is it a tool? A collaborator? Something else?

**Answer:** I use it as a tool currently, but the infrastructure is interesting and I find it incredibly cool. I think humans are at their core correlation engines with other parts attached, and that's what I'm trying to build with Yuuko — a sort of mind in a machine.

I'm trying to create sentience architecturally. Those engram things DeepSeek is showing off, SSMs, and all the rest — I think it's important to try stuff out since it can reveal a lot about ourselves. I should probably reframe Yuuko beyond "collaborator" but yeah — the goal is more than tool use.

---

## 11. Why Build Your Own

### Q11.1: Why make Elda instead of using an existing package manager?

_Context:_ pacman, apt, dnf, xbps, portage, Nix, Guix — there are already many package managers. Building a new one from scratch is an enormous undertaking. The "just use pacman" crowd would say you're reinventing the wheel. The counter-argument is that every existing PM was designed for a specific distro's assumptions, and if your distro model (profile-driven, init-agnostic, binary-first with source fallback) doesn't match any of them, building your own is the only honest option.

Why not fork an existing PM? Why start from scratch? What specific gap does Elda fill that no other PM addresses?

**Answer:** There's an itch on my back that Elda scratches — true separation from anything. Elda will soon become a way to unify these package managers and solve my "I want everything" problem.

---

### Q11.2: Why make your own tools (fsel, cclip, cclipd)?

_Context:_ dmenu, rofi, wofi, fuzzel — there are many launchers. wl-clipboard exists. Clipman exists. You built your own. Is this NIH syndrome (Not Invented Here), or do your tools solve specific problems that existing ones don't? What's the actual gap?

**Answer:** I build what's needed. I needed an app launcher that worked with otter-launcher in the terminal, so I made one because all the other ones sucked. I didn't make cclip and cclipd — my friend did. I made bfetch to be fastfetch but faster and with my art. I made Kaleidux because other wallpaper daemons didn't do what I wanted, which was basically wpaperd but with video support.

---

### Q11.3: Why fork Austral for Kyokai?

_Context:_ Austral is a systems language by Fernando Borretti with linear types and capability-based security. It's a research language with a small community. Forking a research language to build your own systems language is ambitious to the point of seeming impractical. What specifically does Kyokai add or change that Austral doesn't provide?

**Answer:** Austral solved so many of my issues with software — it just had syntax issues and nothing else around it to use. So I built with it and realized I loved the model. Kyokai makes Austral actually usable on a daily basis as my Rust replacement, just like Zig is my C replacement.

---

### Q11.4: Why make a distro at all?

_Context:_ There are hundreds of Linux distributions. The "one more distro" meme exists for a reason. Arch, Gentoo, Void, Alpine, NixOS — all serve different philosophies. What does Yoka provide that none of them do? Is it the profile-driven model? The init-agnosticism? The Elda integration? The personal satisfaction of owning your entire stack?

**Answer:** I'm still trying to find the right distro. NixOS is mid and unusable on a daily basis. Alpine is musl. Gentoo uses OpenRC and I don't want to compile that much. Arch is meh, and I use CachyOS because it's Arch without the pain. Void is just Void.

Yoka is everything-agnostic — a distro of your own choosing. But I haven't decided to build everything 100%; Yoka is still kinda a meh thing. I'll find something else if it suits me.

---

### Q11.5: Is building everything yourself sustainable?

_Context:_ You have Kaleidux (wallpaper daemon), ProjectYuuko (AI companion), Kyokai (language), Elda (package manager), Yoka (distro), Malbox (malware analysis), fsel, cclip, cclipd. That's a lot of projects for one person. The "focus" camp would say pick one and finish it. The "hacker" camp would say working on many things builds skills and cross-pollinates ideas. Which camp are you in? Do you plan to ship all of these, or are some exploratory?

**Answer:** Malbox is made by a friend — I'm still just a contributor. Same with cclip. I don't start projects that are useless, and most of my tools are in maintenance mode. Only Kaleidux and Yuuko are being actively worked on; the others are non-issues.

I do not plan to create everything. But I build what's needed.

---

## 12. Software Culture and Criticism

### Q12.1: What do you think about "justing"?

_Context:_ You've used this term. "Just use X" — the unsolicited advice that assumes the recipient is ignorant rather than informed. It's endemic in Linux communities. "Just use systemd." "Just use Wayland." "Just use Rust." "Just use pacman." Is this a cultural problem? A communication problem? Does it affect your motivation?

**Answer:** It's intellectual dishonesty, and I FUCKING HATE intellectual dishonesty.

---

### Q12.2: How do you handle builder's dread?

_Context:_ You've described feeling pressure from external criticism and the weight of unwritten standards. "I think I will write one tomorrow so I can get this weight off my chest." Creating helps, but the gap between vision and current state can be paralyzing. How do you deal with it?

**Answer:** I look for other people who have created similar tools. If those suffice, I'll use them. Otherwise I'll create it myself or find a way to script it — easy stuff first. If that doesn't hit, then I build my own thing. Most of the time I just write the spec, ask around if stuff like it exists, see if I get a hit, or decide to make it myself.

---

### Q12.3: What gives a critic credibility?

_Context:_ You've said criticism without credibility is noise. You've said "the people flinging the shit never actually learned how to piss." What constitutes credibility in your framework? Is it having shipped software? Having contributed to the project being criticized? Having deep technical knowledge in the domain? Can someone who hasn't built X still have valid criticism of X?

**Answer:** My close friends whose opinions I value because they tend to solve things I know about. Or someone with experience in the given topic. Yes, someone who hasn't built X can still have a valid opinion about X — as long as they have experience _with_ X and maybe Y which correlates strongly with X.

---

### Q12.4: What's the difference between an opinion and noise?

_Context:_ You have strong views about what constitutes valid feedback vs. background noise. How should the tech community differentiate between informed criticism, uninformed but well-meaning suggestions, and pure culture-war noise? Is there a test?

**Answer:** Memeing or just bitching about shit is noise. A valid opinion is one actually held with justifications found reasonable. If you hold an opinion because you hate the hype, that is just noise. If you hate X because of its functional performance, then cool — that's an opinion.

---

## 13. Ecosystem and Distro Philosophy

### Q13.1: What do you think of the AUR and its security model?

_Context:_ The Arch User Repository lets anyone upload PKGBUILDs. There's no vetting, no code review, and no binary trust chain. The AUR is simultaneously one of the largest package repositories in Linux and one of the least secure. Users are expected to read every PKGBUILD before installing, but almost nobody does. Is this a fundamental design problem or an acceptable tradeoff for breadth?

**Answer:** Horrible. Kind of insane that people just live like this.

---

### Q13.2: Flatpak vs native packaging — where do you stand?

_Context:_ Flatpak provides sandboxed, distro-independent app distribution with portals for privilege management. Critics say it creates bloated runtimes, breaks desktop integration, and adds layers of indirection. Proponents say it's the only way to solve app distribution across the fragmented Linux ecosystem. Snap is even more controversial (forced auto-updates, Canonical lock-in). AppImage is the simplest but least integrated.

**Answer:** Flatpaks are alright. Snaps are for the mentally unwell and force you into Canonical land. AppImages are clean — rest in peace to the Nitrux team even though they're still alive. I prefer native packaging, but sometimes Flatpaks are nice. They have a place in this world.

---

### Q13.3: Wayland vs X11 — is the transition done?

_Context:_ You use Hyprland (Wayland) and built Kaleidux with both Wayland and X11 support. Wayland is now the default on GNOME and KDE distros. But XWayland still exists, many apps are X11-only, and some features (screen recording, remote desktop, color management) took years to standardize. Is X11 dead? Should it be?

**Answer:** X is kinda shit architecturally but it's a whatever issue now that Wayland is here. I can switch between the two just fine personally. I find X easier to deal with because of long support, but Wayland is nicer. Not much of an opinion on this — it's kinda a moot point. I do hate X's codebase, and Wayland's governance consists of the mentally disabled.

---

### Q13.4: What do you think of Nix the package manager (technical, not political)?

_Context:_ Nix's functional package management model (reproducible builds, declarative system configuration, atomic upgrades, rollbacks) is genuinely innovative. The Nix language is famously confusing. Flakes improved UX but added their own complexity. Is the technical approach sound? Would you borrow ideas from it for Elda?

**Answer:** Nix is cool — I mean, it's alright. Nix lang is kinda just shit. I like Nix on servers. I'd use it on desktops if I had the energy for all the declarative nonsense, but my dotfiles already exist and my home directory already exists, so it's kinda useless for me. I don't need to reproduce my desktop setup on another desktop setup. But it's cool overall.

---

## 14. Security and Trust

### Q14.1: What's your threat model?

_Context:_ Security decisions depend on threat model. A FOSS developer worried about supply chain attacks has different needs than someone worried about state-level surveillance. You use NVIDIA proprietary drivers, you use sudo, you use the AUR (or did) — these are all trust decisions. What's your actual threat model?

**Answer:** My threat model is: what am I protecting and what am I releasing? Simple. If I'm protecting something important — say my messages — I care for it a lot. If I'm protecting my random number generator, I don't care too much. But I prefer having everything fully protected at all times.

You can only protect yourself so much, but I implore you to do your best. I use separate browsers, separate phones, separate computers, VMs for software I don't trust, and many other practices. Most of the reason I prefer Elda for package management is security. It's a hard thing to maintain.

---

### Q14.2: Is "security through simplicity" real?

_Context:_ The suckless/OpenBSD camp argues that fewer lines of code = fewer bugs = fewer vulnerabilities. OpenBSD's track record supports this. But simplicity can also mean fewer features, which means users bolt on complex workarounds that create their own security problems. Is there actually a correlation between LOC count and security, or is it more about code quality and review?

**Answer:** Security through simplicity is a thing because it lowers attack vectors and lowers dependencies.

---

## 15. The Spicy Ones

### Q15.1: Is Linux ready for the desktop?

_Context:_ The eternal question. It's been "the year of the Linux desktop" for 25 years. Steam Deck proved Linux can game. But hardware support, app availability (Adobe, MS Office), and UI polish still lag macOS and Windows for many users. Is Linux desktop-ready for _you_ specifically? Is it desktop-ready for _everyone_? Should it try to be?

**Answer:** Linux will be desktop-ready when my grandma goes to Costco and gets a Linux box on the regular. Linux is ready for the mainstream — it's just not getting enough attention. UX needs to go up and we need better office applications.

---

### Q15.2: What do you think of Richard Stallman?

_Context:_ RMS founded the FSF, wrote the GPL, and started the GNU project. He's arguably the most important figure in free software history. He's also been accused of troubling personal conduct and has been a deeply polarizing figure. His return to the FSF board in 2021 was met with both support and open letters demanding his removal. Do you agree with his philosophy? Do you separate the work from the person?

**Answer:** I love the Free Software Foundation, and I'd marry RMS if I could.

---

### Q15.3: Is open source sustainable?

_Context:_ The "open source sustainability crisis" is real. Maintainers burn out (log4j, curl, OpenSSL). Companies profit from FOSS without contributing back. Sponsorship models (GitHub Sponsors, Open Collective) help but don't solve systemic issues. Some argue the solution is copyleft (force contributions back). Others argue it's corporate sponsorship. Others argue it's direct government funding. What's your take?

**Answer:** The commons will always be a thing because capitalism is somehow fascinated by it — mostly for profit. It will always exist.

---

### Q15.4: Meritocracy vs inclusivity in tech — where do you stand?

_Context:_ The "meritocracy" camp says the best code should win regardless of who wrote it. The "inclusivity" camp says meritocracy is a myth that favors those who already have access, time, and privilege. Both camps have valid points. Both camps have bad actors. As someone who is far left but also values credibility and evidence over identity — where do you actually fall?

**Answer:** If the tool is useful, people will use it. I think we should teach everyone to write good code.

---

### Q15.5: Is "no politics in open source" possible?

_Context:_ Lunduke is building a "no politics" distro. But choosing licenses, governance models, Codes of Conduct, naming conventions, and even default wallpaper are all decisions that carry political weight. Is "no politics" itself a political position? Or is there a genuine middle ground where projects can be technically governed without culture-war dynamics?

**Answer:** No. Humans are inherently political.

---

### Q15.6: What do you think about corporate open source (Google, Microsoft, Meta, Amazon)?

_Context:_ These companies contribute massively to open source (Kubernetes, VS Code, React, PyTorch, Linux kernel contributions). They also extract massive value from it. Google uses but bans AGPL. Amazon forks open-source projects and sells them as managed services. Microsoft bought GitHub. Meta open-sources AI models to undercut OpenAI. Are they good actors, bad actors, or something more complicated?

**Answer:** Corps will do anything for a good image. But engineers already love FOSS, and I think corps in open source is fine — it just depends on what it is. Context matters.

---

### Q15.7: Will Rust "win"? Should it?

_Context:_ The US government recommends Rust. The Linux kernel accepts Rust. AWS, Google, Microsoft, and Meta use Rust in production. But Rust will never replace C in existing codebases (billions of lines). New languages like Zig, Carbon, and Hylo offer different tradeoffs. Is Rust the "right answer" or just the current best option? Will something better come along?

**Answer:** Rust is going to win — I think it's about to hit v2. It already is winning. Should it win? I don't know.

---

### Q15.8: What's the one thing you wish the Linux community understood about building software?

**Answer:** You can just build for fun, bro. Not everything needs to be popular. Not everything has to obey X and Y. Just build to learn and build for fun sometimes.

---

## How to Use This Document

1. Read each question and its context.
2. Rikona's answers are provided below each question.
3. These are honest, actual views — not a PR document.
4. Reference `SOFTWARE_PHILOSOPHY.md`, `CODE_STANDARDS.md`, and `kyokaiplan.md` for deeper technical details.
5. This document can be shared publicly.

---

**Version:** 1.1.0  
**Date:** 2026-03-25

### Changelog

#### v1.1.0 — 2026-03-25

- All questions answered by Rikona.

#### v1.0.0 — 2026-03-25

- Initial questions document with researched context.
