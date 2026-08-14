# Cryptographic Failures — Explained Simply

## The Basic Idea
This category is about **protecting sensitive data using encryption** — both while it's **traveling** between systems (in transit) and while it's just **sitting stored somewhere** (at rest). When encryption is missing, weak, or done incorrectly, attackers can read or steal data they shouldn't have access to.

## Two Key Concepts to Understand First

**Encryption "in transit"** = protecting data **while it moves** from one place to another (e.g., from your browser to a website's server). This is normally done using **TLS** (the technology behind "HTTPS" — the padlock icon in your browser).

**Encryption "at rest"** = protecting data **while it's stored** somewhere (a database, a file, a backup) — even if someone breaks into the storage system, the data should still be unreadable without the right key.

**What kind of data needs this extra protection?** Passwords, credit card numbers, health records, personal information, business secrets — especially anything covered by privacy laws (like GDPR in Europe, or PCI DSS for payment card data).

## Common Ways Encryption Goes Wrong (Simplified)

- Using **old, weak, or outdated encryption methods** instead of modern, trusted ones
- Using **default or weak encryption keys**, reusing the same key everywhere, or never rotating (changing) keys over time
- Accidentally **storing encryption keys directly in the code** (e.g., committed to GitHub) where anyone with access can find them
- Not actually **enforcing** encryption — e.g., a site technically supports HTTPS but still allows plain, unencrypted HTTP too
- Not properly checking that a server's **security certificate** (the thing that proves "this website is really who it claims to be") is valid and trustworthy
- Using **outdated/broken hashing functions** like MD5 or SHA1 (a "hash" is a one-way scrambling of data, often used for passwords) when modern, safer options exist
- Using **passwords directly as encryption keys** instead of properly converting them first
- Using **random numbers that aren't truly random/unpredictable enough** — weak randomness makes encryption easier to break
- Allowing the **encryption method to be downgraded or bypassed** entirely by an attacker

## How to Prevent It (Simplified)

- **Figure out which data is sensitive** first (passwords, personal info, financial data, etc.) — you can't protect what you haven't identified
- **Don't store sensitive data if you don't need to.** Data that was never kept can't be stolen — delete it as soon as it's no longer needed
- **Encrypt all sensitive data at rest** (in storage/databases)
- **Encrypt all data in transit** using modern, strong protocols (TLS version 1.2 or newer) — and force browsers to always use encrypted connections (a setting called **HSTS**)
- **Don't cache (temporarily save) sensitive data** anywhere it could be later exposed (servers, CDNs, etc.)
- **Never store passwords in plain, readable form or with weak hashing.** Use strong, modern password-hashing methods (like **Argon2** or **bcrypt**) — these are specifically designed to make password-cracking slow and difficult, even if the database is stolen
- **Use trusted, well-tested encryption libraries** — never build your own custom encryption; even security experts avoid "rolling their own crypto" because it's extremely easy to get subtly wrong
- Keep systems and encryption settings **regularly reviewed and updated**
- Start preparing for **"post-quantum cryptography"** — future quantum computers may eventually be powerful enough to break today's encryption methods, so high-risk systems need to prepare for stronger, quantum-resistant encryption well in advance (OWASP recommends being ready by the end of 2030)

## Real Attack Examples (Simplified)

**Scenario #1 — Stealing a login session over insecure WiFi:**
A website doesn't properly enforce HTTPS everywhere. An attacker on the same network (like public WiFi) intercepts the unencrypted traffic, **forces the connection to downgrade** from secure HTTPS to insecure HTTP, and steals the victim's **session cookie** (the small piece of data that proves you're logged in). The attacker can then **reuse that stolen cookie** to impersonate the victim — accessing their account or even changing data, like redirecting a money transfer to a different recipient.

**Scenario #2 — Cracking stolen passwords:**
A company stores passwords using a **weak, unsalted hash** (meaning identical passwords always produce the identical scrambled result, with no extra randomness added). An unrelated bug (a file upload flaw) lets an attacker download the entire password database. Because the hashes are weak/unsalted, the attacker can instantly look them up using a **precomputed table of common password hashes** (called a "rainbow table"), or simply crack them quickly using powerful graphics cards (GPUs) — revealing everyone's real passwords.

# Two Concepts Explained More Clearly

## 1. "Using Passwords Directly as Encryption Keys Instead of Properly Converting Them First"

### The problem, step by step

**What an encryption key actually needs to be:** Modern encryption algorithms (like AES) require a key that is a specific length (e.g., exactly 256 bits) and is made of **completely random-looking bytes** — no patterns, no predictability.

**What a human password looks like:** Passwords are things people choose and remember, like `Summer2024!` or `MyDog123`. Compared to true randomness, passwords are:
- **Too short** — a password key needs to be a precise length like 256 bits (32 characters of raw random data); a human password rarely matches this exactly
- **Not random enough** — humans pick predictable patterns: real words, common substitutions (`a` → `@`), birthdays, keyboard patterns like `qwerty`. This is called having **low entropy** (low unpredictability)

**What happens if you use the password directly as the key?** Some lazy/incorrect implementations just take the password text and try to plug it straight in as if it were the encryption key. Because passwords are short and predictable compared to real random keys, **the encryption becomes much weaker than it appears.** An attacker doesn't need to break the actual encryption algorithm (AES is very strong) — they just need to **guess the password**, the same way they'd guess it for a login form (trying common passwords, dictionary words, leaked password lists, etc.). Once they guess the password correctly, they instantly have the "key" too, and can decrypt everything.

### The correct solution
Instead of using the password directly, you run it through something called a **Password-Based Key Derivation Function (PBKDF)** — examples: PBKDF2, Argon2, scrypt. Think of this as a **conversion machine**:

```
Weak, short, human-guessable password  →  [PBKDF conversion process]  →  Strong, proper-length, random-looking encryption key
```

This conversion process is also intentionally **slow** (it takes a small but noticeable amount of computer time), which is a deliberate design choice: it makes it **expensive for an attacker to test each password guess**, because they have to run this same slow process for every single guess. This is the same idea mentioned earlier with **Argon2/bcrypt for password storage** — same principle, applied here to turn passwords into usable encryption keys.

---

## 2. "Allowing the Encryption Method to Be Downgraded or Bypassed Entirely by an Attacker"

### What "downgrade" means, concretely
Many systems are built to **support multiple versions or strength-levels of encryption at once**, often for compatibility with older devices/software. For example, a web server might technically support:
- TLS 1.3 (modern, strong, current standard)
- TLS 1.2 (still okay)
- TLS 1.0 or SSL 3.0 (old, known to be weak/broken)

**Why keep the old ones at all?** So that older browsers/devices that don't understand the newest protocol can still connect. This seems reasonable — but it creates an opening.

### How the attack works, step by step
1. Normally, when your browser connects to a server, they **negotiate** which encryption version to use — both sides try to agree on the strongest one they both support.
2. An attacker positioned **in the middle of the connection** (e.g., on the same public WiFi network) can **interfere with this negotiation process**. They intercept the initial connection request and **trick the server (or your browser) into thinking the other side only supports the old, weak protocol** — even though both sides actually support the strong modern one.
3. Both sides, believing this is the best they can agree on, **fall back to the old, weak encryption** — this is the "downgrade."
4. Now that the weak/broken protocol is in use, the attacker can **exploit its known weaknesses** to read or tamper with the supposedly "encrypted" traffic — even though strong encryption was technically available the whole time.

### What "bypassed entirely" means
In some cases it's even simpler — the system might have a bug or misconfiguration that lets an attacker **skip the encryption step completely**, sending or receiving data in plain, unencrypted form, even on a connection that's supposed to always be encrypted.

### The prevention (referenced earlier in the text)
This is exactly why the prevention advice says to use **TLS 1.2 or higher ONLY** — meaning the server should be configured to **completely refuse** connections that try to use older, weaker protocols, removing the attacker's ability to force a downgrade in the first place. There's simply no weak option left to fall back to.

---

## 3. How Do Attackers "Crack" Hashes Using GPUs?

### What a hash actually is
A **hash function** takes any input (like a password) and turns it into a fixed-length, scrambled-looking string of characters. Critically, hashing is designed to be a **one-way process** — you're not supposed to be able to reverse it back into the original password.

```
"password123"  →  [hash function]  →  "ef92b778bafe771e89245b89ecbc08a"
```

### So if it's "one-way," how can attackers reverse it?
**They don't actually reverse the math.** Instead, they use a much simpler brute-force trick:

1. The attacker takes a **huge list of guesses** — every common password, every word in the dictionary, every leaked password from previous data breaches (these lists have billions of entries), plus systematic combinations (`password1`, `password2`, `Password!`, etc.)
2. For **each guess**, they run it through the **exact same hash function** the website used
3. They **compare the resulting hash** to the stolen hash from the database
4. If the hashes match, **they've found the original password** — because if two inputs produce the same hash output, the guess must have been the actual original password (in practice, for good hash functions, matching hashes essentially guarantee matching inputs)

### Why GPUs specifically make this fast
A **GPU (Graphics Processing Unit)** — normally used for rendering video game graphics — happens to be extremely good at doing **many small, simple mathematical calculations simultaneously, in parallel** (thousands at once), which is exactly the kind of workload hashing calculations involve.

A regular CPU might test a few million password guesses per second. A **GPU (or a cluster of GPUs) can test billions of guesses per second** for fast/weak hash functions like plain MD5 or SHA1 — because it can run thousands of these hash calculations at the exact same time instead of one after another.

**This is precisely why the earlier prevention advice matters so much:** modern password-hashing functions like **Argon2 or bcrypt** are specifically designed to be **slow and resistant to this kind of parallelization** — they're built to be difficult to speed up even with powerful GPU hardware, unlike old, fast, simple hash functions like MD5, which GPUs can chew through at enormous speed.
