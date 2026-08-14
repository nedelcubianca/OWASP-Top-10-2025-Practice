# Summary: How OWASP Calculates and Ranks Application Security Risks

## What is an "Application Security Risk"?
When attackers try to break into or damage an application, they don't have just one way in — there are **many different paths (vectors)** they could exploit. Each of these potential paths represents a distinct **risk** that needs to be studied and measured, so that organizations know which flaws are most dangerous and deserve the most attention.

## The Risk Calculation Formula — the Building Blocks
OWASP explains that to rate how dangerous a given type of weakness (CWE) is, several ingredients are combined:

- **Threat Agents** — Who might attack, and why (varies by organization, industry, and situation).
- **Attack Vectors** — How exposed the application is to being attacked.
- **Exploitability** — How easy is it for an attacker to actually exploit this flaw?
- **Likelihood of Missing Security Controls** — How often is this flaw left unguarded/unmitigated in real applications?
- **Technical Impact** — If exploited, how much damage does it cause to the system itself (data loss, downtime, etc.)?
- **Business Impact** — How much does that technical damage translate into real-world harm for the organization (financial loss, reputation damage, legal consequences)?

**Important nuance:** the same software can carry very different risk levels depending on *who* uses it. OWASP gives an example: if a public information website and a hospital both use the exact same content management system (CMS), a breach means something very different for each — the hospital's health records are far more sensitive than the public website's general content. So while OWASP's Top Ten gives a general, industry-wide risk ranking, **each organization still needs to evaluate its own specific exposure, threat agents, and business impact.**

## How the Risk Score Has Been Calculated Over Time
- **2017 edition:** Categories were chosen based on how often each was found (*incidence rate* = likelihood), then the OWASP team **manually ranked** them by discussing their own expert experience regarding exploitability, detectability, and technical impact. This was more subjective/expert-opinion based.
- **2021 edition:** OWASP switched to a more **data-driven approach**, pulling real exploit and impact scores from the **CVSS system** (explained below), sourced from the **National Vulnerability Database (NVD)**.
- **2025 edition:** OWASP **kept the same 2021 data-driven methodology**, just refreshed with new/larger data.

## Key Background Concepts You Need to Understand First

### What is a CVE?
A **CVE (Common Vulnerabilities and Exposures)** is a unique ID assigned to one **specific, real-world vulnerability** that was actually discovered in a specific piece of software (e.g., "CVE-2023-XXXX: a specific flaw found in Apache version X"). This is different from a CWE — remember, a **CWE is the general *type* of flaw** (like "SQL Injection" as a category), while a **CVE is one particular, actual instance of that flaw type being found in a real product.**

So the relationship is: **many CVEs can map to one CWE** — e.g., hundreds of individual real vulnerabilities discovered in different software products might all be examples of the CWE "SQL Injection."

### What is CVSS?
**CVSS (Common Vulnerability Scoring System)** is a standardized scoring system that rates how severe a given CVE (specific vulnerability) is. It produces numeric sub-scores, including:
- An **Exploit score** — how easy is this specific vulnerability to actually exploit?
- An **Impact score** — how much damage does exploiting it cause?

CVSS has gone through multiple versions over time — **CVSSv2, CVSSv3, and now CVSSv4** — and each version calculates these scores somewhat differently (details below).

### Why does the CVSS version matter?
- In **CVSSv2**: both Exploit and Impact scores could theoretically reach up to 10.0 each, but the final formula would scale them down — Exploit ends up weighted to 60% importance, Impact to 40%.
- In **CVSSv3**: the maximum theoretical scores were redesigned — capped at 6.0 for Exploit and 4.0 for Impact.
- **Net effect of the redesign:** when OWASP compared the two systems, Impact scores came out **about 1.5 points higher on average** under CVSSv3, while Exploit scores came out **about 0.5 points lower on average**, compared to CVSSv2. This means the *same* vulnerability could get a noticeably different score depending on which CVSS version was used to rate it — which is why OWASP had to carefully account for this when blending data from both versions.
- **CVSSv4** exists too, but OWASP explicitly **did not use it** this time, because v4 fundamentally changed its scoring algorithm in a way that no longer cleanly separates out "Exploit" and "Impact" sub-scores the way v2 and v3 did. OWASP says they may figure out how to incorporate v4 in a future edition, but couldn't do so in time for 2025.

## How OWASP Actually Gathered and Processed the Data

1. **Data source:** OWASP used a tool called **OWASP Dependency Check** to extract CVSS Exploit and Impact scores, and grouped them by the CWEs they relate to.

2. **Scale of the data:**
   - **~175,000 CVE records** mapped to CWEs in the NVD database (up from ~125,000 in the 2021 edition) — meaning significantly more real-world vulnerability data was available this time.
   - **643 unique CWEs** had at least one CVE mapped to them (up from 241 in 2021) — meaning far more distinct *types* of flaws now have real-world severity data attached to them.
   - Out of roughly 220,000 total CVE score entries analyzed: **160k had CVSSv2 scores, 156k had CVSSv3 scores, and 6k had CVSSv4 scores** (note: many individual CVEs have more than one type of score recorded, which is why these numbers add up to more than the total CVE count).

3. **Combining the scores:** For each CWE, OWASP took all the CVEs mapped to it, and calculated a **weighted average** Exploit score and Impact score — weighting by what percentage of that CWE's CVEs had CVSSv3 scores vs. how many only had older CVSSv2 scores. This produces one blended, representative Exploit score and one Impact score per CWE.

4. **Incidence Rate:** Separately from the CVE/CVSS severity data, OWASP also calculated, for each CWE, **what percentage of tested applications had that CWE present at least once** (again — presence/absence, not counting repeat occurrences within the same app, consistent with what was explained in the earlier summary about prevalence vs. frequency).

5. **Coverage:** This measures **how many organizations/applications actually tested for a given CWE at all**. This matters because: if only a handful of applications were ever tested for a particular CWE, the incidence rate calculated from that tiny sample might not be reliable or representative of the wider real world. Higher coverage = more confidence that the incidence rate number is accurate, because it's based on a bigger, more representative sample.

## The Final Risk Score Formula
Putting it all together, OWASP's 2025 formula for calculating a category's overall Risk Score is:

**Risk Score = (Max Incidence Rate % × 1000) + (Max Coverage % × 100) + (Avg Exploit × 10) + (Avg Impact × 20) + (Sum of Occurrences ÷ 10,000)**

In plain terms, this formula rewards a risk category for:
- Being found frequently across tested applications (**incidence rate**, weighted heavily × 1000)
- Having been tested widely enough to be statistically trustworthy (**coverage**, × 100)
- Being easy to exploit (**exploit score**, × 10)
- Causing serious damage when exploited (**impact score**, weighted more heavily than exploitability, × 20 — suggesting OWASP considers *damage potential* somewhat more important than *ease of attack*)
- Having simply occurred in a large raw number of applications overall (**total occurrences**, though this factor is heavily scaled down by dividing by 10,000, so it contributes only a small nudge to the final score compared to the other factors)

**Using this formula, the 2025 scores ranged from a high of 621.60 (Broken Access Control — the riskiest category) down to a low of 271.08 (Memory Management Errors — comparatively less risky by this measure).**

OWASP is upfront that this isn't a flawless, perfect system — but they consider it a **useful, consistent, and data-grounded way** to rank which categories of weakness deserve the most attention industry-wide.

## An Emerging Challenge: What Even *Is* an "Application" Anymore?
OWASP flags a growing difficulty: modern software isn't always built as one single traditional "application" anymore. With the rise of **microservices** (breaking one big application into many small, independent interconnected services) and similar modern architectures, it becomes unclear how to even define "one application" for counting/testing purposes. For example: if a company scans many small code repositories, does each repository count as its own separate "application," or are they all part of one bigger application? This ambiguity complicates the incidence rate and coverage calculations, and OWASP acknowledges that **future editions of the Top Ten may need to adapt their methodology** to keep up with how software architecture keeps evolving.

## Glossary: The Data Factors Listed for Each Top Ten Category
To help readers interpret the published data next to each category, OWASP defines these terms:

| Term | Meaning |
|---|---|
| **CWEs Mapped** | How many distinct CWEs the OWASP team decided belong under this category |
| **Incidence Rate** | The % of tested applications found to have this CWE, for that testing organization/year |
| **Weighted Exploit** | The blended Exploit sub-score (from CVSSv2 + CVSSv3 data), normalized onto a 0–10 point scale |
| **Weighted Impact** | The blended Impact sub-score (from CVSSv2 + CVSSv3 data), normalized onto a 0–10 point scale |
| **(Testing) Coverage** | The % of all applications (across all organizations) that were actually tested for this specific CWE |
| **Total Occurrences** | The total number of applications found to have any of the CWEs belonging to this category |
| **Total CVEs** | The total number of real-world CVE records in the NVD database that map to the CWEs in this category |
