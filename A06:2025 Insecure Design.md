# A04:2025 Insecure Design — Explained Simply

## What Is "Insecure Design"?
This category is about a **flaw in the planning/blueprint of the application** — not a coding mistake. It means the necessary security protections were **never even planned for** in the first place.

## Design Flaw vs. Implementation Bug — The Key Distinction
This is the most important idea in this section:

- **Insecure design** = the security control was **never planned or built at all**, because nobody thought it was needed
- **Insecure implementation** = the security control **was planned**, but a developer **made a mistake** while coding it

**Why the difference matters:** if a control was never designed in the first place, **no amount of "perfect coding" can fix it** — because there's nothing there to code correctly. You must go back and **redesign** the system to include that missing protection.

**Simple analogy:** if a building's blueprint never included a lock on the back door (insecure design), it doesn't matter how well the door itself is built — there's no lock to begin with. Compare this to a blueprint that *does* include a lock, but the construction worker installed it upside-down (implementation bug) — the fix there is much simpler: just fix the installation.

## Why Insecure Design Happens
Often it's because the team never asked: **"What's actually at risk here, and how much protection does this specific feature need?"** — this is called **business risk profiling**. Without figuring out what needs protecting and how much, teams don't know what security controls to design in the first place.

## Three Pillars of Secure Design (Simplified)

### 1. Requirements & Resource Management
Before building anything, figure out — together with the business side, not just developers — **what needs to be protected**: how confidential must the data be, does it need to stay accurate (integrity), does it need to stay available, is exposure to the public an issue, do different customers'/tenants' data need to be kept separate from each other. Also plan a **budget** that includes security work, not just development.

### 2. Secure Design (Culture & Process)
Security shouldn't be something you "add on" later — it has to be **built into the thinking process from the start**. This mainly happens through **threat modeling**: sitting down (e.g., during planning meetings) and asking **"how could someone attack this feature?"** for every new piece of functionality — especially anything involving data flow or access permissions. Teams should clearly agree on: what's the correct/expected behavior, and what should happen when things fail or go wrong.

### 3. Secure Development Lifecycle
Security shouldn't only be a one-time meeting — it needs to be a **continuous process** throughout the whole project: reusable secure design patterns, proper tools, threat modeling, and learning from past incidents/mistakes to improve. Security specialists should be involved **from the very beginning** of a project, not brought in only at the end. OWASP has a framework called **SAMM** to help structure this.

## How to Prevent Insecure Design (Simplified)
- Involve **security experts early**, throughout the whole project, not just at the end
- Build and reuse a library of **proven, secure design patterns** ("paved road" — a pre-approved, safe way of doing common things, so developers don't have to invent security from scratch each time)
- Use **threat modeling** on critical parts: login, access control, business logic
- Add **sanity/plausibility checks** at every layer of the application (frontend AND backend — remember, frontend checks alone aren't real security, as covered earlier)
- Write **tests specifically designed to verify security assumptions hold up** (not just that features "work")
- **Separate/isolate** different parts of the system (and different customers' data) from each other, so a problem in one part can't easily spread to another

## Example Attack Scenarios (Explained)

### Scenario #1: "Security Questions" for Password Recovery
Some apps let you recover your account by answering questions like *"What's your mother's maiden name?"* or *"What was your first pet's name?"* **The problem:** these answers **aren't actually secret** — other people can often know them, find them (e.g., on social media), or guess them. This isn't a coding bug — it's a **fundamentally flawed design choice**. No amount of "implementing it correctly" fixes this; the entire approach needs to be replaced with something more secure (like proper multi-factor recovery methods).

### Scenario #2: Cinema Group Booking Abuse
A cinema lets people book up to 15 seats without needing to pay a deposit first. But the **business logic** never considered: *what if someone books 15 seats, in many rapid requests, across every cinema simultaneously?* An attacker could exploit this gap to book **hundreds of seats** without ever paying a deposit, causing massive financial damage — not because of a coding bug, but because nobody designed a check for this abuse pattern in the first place.

### Scenario #3: Bots Buying Up Limited-Stock Products
An online store selling in-demand video cards has no protection against **automated bots** that buy up all the stock instantly (to resell at a markup — "scalping"). Real customers can never get a fair chance to buy at the normal price. The fix isn't a bug fix — it requires **designing in anti-bot protections from the start** (e.g., flagging purchases completed suspiciously fast after a product goes live as likely bots, and rejecting them).

