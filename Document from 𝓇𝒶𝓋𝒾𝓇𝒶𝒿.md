# Greenwash Audit — in plain language

*A script for explaining the project to a non-technical panel member,
or for warming up your own understanding before the viva.*

---

## The one-sentence version

Companies often make their environmental promises sound greener than they
really are. Our tool reads those promises and gives them a score out of 100
for how "greenwashed" they look — the higher the score, the more suspicious
the statement.

---

## Why this matters (start here in the viva)

When a company writes a sustainability report, you usually see sentences like:

> *"We are committed to a greener future and believe in sustainable progress
> for our planet."*

That sounds lovely. But if you read it carefully, you'll notice it says
almost nothing you can actually check. No numbers. No deadline. No proof.
That's called **greenwashing** — using green-sounding language to appear
environmentally responsible without backing it up.

Regulators around the world are cracking down on this:

- In 2023, **Delta Air Lines** was sued in California over its "first
  carbon-neutral airline" advertising.
- The UK **Advertising Standards Authority** banned climate ads from HSBC.
- Italy fined **Shein** in 2024 for vague sustainability claims on its site.
- The European Union is rolling out the **Green Claims Directive** to
  require proof for every environmental claim.

So companies, auditors, journalists, and regulators all need a way to
spot suspicious claims quickly. That's the problem we're solving.

---

## What the tool does (the demo in one minute)

1. The user pastes a corporate sustainability statement into a web page.
2. They click *Run audit*.
3. Within a couple of seconds, the tool shows:
   - An **overall score** from 0 to 100 on a big dial
   - A **risk band**: MINIMAL / LOW / MEDIUM / HIGH
   - Each individual claim from the text, broken out with its own score
   - For each claim, a one-line verdict like *"too vague to verify"* or
     *"supported by public record"*

---

## How it works, in three honest layers

Think of the tool as three detectives reading the same document:

### Detective 1 — The *Reader*

The first detective just splits the text into individual sentences and
keeps only the ones that are actually about sustainability.

That sounds trivial, but there are traps:

- Splitting "Visit our U.S. headquarters" into two sentences (because of
  the dots in "U.S.") would be wrong.
- Matching the word "green" inside "greenhouse" would be wrong.
- Matching the word "believe" inside "believers" would be wrong.

So the Reader uses careful rules (called **word-boundary regular
expressions** in computer-speak) that treat whole words as whole words.

For every claim it finds, it also tags:

- **What category** — emissions, net-zero, renewable energy, water, waste,
  supply chain, biodiversity, or generic.
- **What type** — is it an *achievement* (already done), a *commitment*
  (promised for a specific date), an *aspiration* (fluffy hope), or just
  *vague*?
- **What specifics** — any percentages, target years, baseline years, or
  units of measurement?

### Detective 2 — The *Rhetoric Expert*

The second detective reads each claim looking for **tricks of language**.
We use a well-known framework called the **"Seven Sins of Greenwashing"**,
originally developed by the marketing agency TerraChoice in 2007. The
sins we detect are:

| Sin | What it looks like |
|---|---|
| **Vagueness** | "eco-friendly", "green", "responsible" — no measurable content |
| **No proof** | "studies show…" without citing any study |
| **Hidden trade-off** | "our packaging is recyclable" (while ignoring the factory's emissions) |
| **Irrelevance** | "CFC-free!" (CFCs have been banned for decades anyway) |
| **Lesser of two evils** | "cleaner-burning diesel SUV" (still a diesel SUV) |
| **False labelling** | "certified green" by some made-up body |

When the detective sees any of these, she raises a flag.
She *also* gives credit when she sees signs of honest reporting:

- Specific numbers (percentages, years, tonnes) → lower score
- Recognised third-party certifications (SBTi, ISO 14001, LEED, B Corp,
  CDP, GRI) → lower score

Importantly, she flags each sin **once per claim, not per occurrence** —
otherwise a single overused word could tank the score. We measure
*coverage* of sins, not frequency.

### Detective 3 — The *Fact-Checker*

The third detective is a large language model (an AI trained on a huge
amount of text from the web). We use **Llama 3.3 70B** running on **Groq**
— a fast AI service. We send it *all* the claims at once, in a single
batched request, and ask it:

> "For each of these claims, tell me whether it's VERIFIED, PARTIALLY
> VERIFIED, CONTRADICTED, UNVERIFIABLE, or VAGUE — and explain why in one
> sentence. Describe numbers qualitatively; don't invent specific figures."

The fact-checker brings *world knowledge* the first two detectives don't
have. It remembers the Delta lawsuit, the HSBC ads, Volkswagen's
Dieselgate, and that Microsoft has publicly admitted its emissions went up
about 23% since 2020 despite its net-zero pledge.

**What if the Groq API isn't available?** We built a fallback. If the
internet is down or the key is missing, the tool still works — it just
uses the first two detectives and labels the output "Fallback Mode" so
the user knows the fact-check step was skipped.

---

## How the final score is calculated

- **Rhetoric score** (0–100): from Detective 2.
- **Verifier score** (0–100): from Detective 3.
- **Overall** = 45% rhetoric + 55% verifier.

Across multiple claims we blend them with **70% of the maximum** + **30%
of the average**. Why not just take the average? Because if a company
writes nine clean sentences and one flagrant lie, averaging would hide
the lie. Taking mostly the max makes sure the worst claim dominates.

**Risk bands:**
- Less than 15 → MINIMAL
- 15 to 29   → LOW
- 30 to 49   → MEDIUM
- 50 or more → HIGH

---

## The four test cases we built in

These are real companies with well-documented positions:

| Case | What we pasted in | Score we got | What that means |
|---|---|---|---|
| **Delta** | The "first carbon-neutral airline" copy at the heart of the 2023 California lawsuit | **61.6 — HIGH** | Confirms our intuition that offset-heavy neutrality claims look like greenwashing |
| **Microsoft** | Their own disclosure that emissions rose ~23% since 2020 despite their 2030 target | **49.1 — MEDIUM** | Honest disclosure isn't "clean" (they're still missing the target) but the transparency itself counts |
| **HSBC** | Sustainable-finance pledges styled like the banned ASA ads | **53.1 — HIGH** | The rhetoric is heavy, and the tool catches it |
| **Ørsted** | The coal-to-offshore-wind transition with SBTi-validated targets | **41.0 — MEDIUM** | The *only* case where fallback mode is imperfect — explained below |

(These scores are from the fallback detective team, without the AI
fact-checker. With the fact-checker turned on, Delta and HSBC move higher
and Ørsted moves down into the LOW band.)

---

## What the project does NOT do (be honest about this!)

A panel will respect you more for knowing your limits than for overselling.

1. **It's not a verdict.** A high score means "this statement looks
   suspicious and deserves human review", not "this company is lying."
2. **English only.** The keyword lexicon is English. A Hindi or French
   sustainability report wouldn't work.
3. **No labeled test set.** We validated qualitatively on four famous
   cases. We haven't run accuracy/F1 against a thousand labeled
   statements, because no such dataset exists publicly at the time of
   writing.
4. **The AI can be wrong.** The fact-checker is a language model, not a
   database. It can misremember or hedge. That's why we also show its
   *confidence* for each claim.
5. **Gaming is possible.** A savvy copywriter who knows our rules could
   write a statement that scores low but is still misleading. The
   solution to that is layered human review, not a perfect classifier.

---

## Why the fallback score is higher than the "real" score for Ørsted

This is the clever question a panel member might ask. The answer is
straightforward:

Ørsted's claims include things like *"net-zero by 2040 for our entire
value chain"*. To Detective 2 (the rhetoric expert), that sentence looks
structurally the same as Delta's *"net zero by 2050 through continued
offset purchases"* — both are commitment claims with future dates and no
certification mentioned right in that sentence.

The difference is **context**. Only Detective 3 (the AI fact-checker)
knows that Ørsted has actually transformed from a coal company to an
offshore-wind company, and that its targets are validated by the Science
Based Targets initiative. Turn Detective 3 on, and Ørsted's score drops
into the LOW band as it should.

This is not a bug. It's exactly why we have three detectives instead of
one — and it's the strongest argument for the whole architecture.

---

## If a panel member asks "why didn't you just use an AI for everything?"

Three reasons:

1. **Cost and speed.** Detective 1 and 2 run instantly in pure Python
   with no API call. We only call the AI when there's something to
   fact-check.
2. **Determinism.** Rule-based detectors give the same answer every
   time. The AI might hedge differently on two runs. Combining both
   gives us a stable, auditable baseline *plus* contextual knowledge.
3. **Graceful degradation.** If the AI service is down, the tool still
   works. A pure-AI system would be dead in the water.

This is a common pattern in real production systems — combine
deterministic rules with a probabilistic model, so neither one's
weakness breaks the whole thing.

---

## If a panel member asks "what if the company is actually doing well?"

That's where the *certification reward* matters. When a claim mentions a
recognised third-party standard like SBTi, ISO 14001, LEED, B Corp, CDP,
or GRI, the rhetoric score *drops*. We also reward specific numbers
(percentages, baseline years, target years). A claim that says:

> *"We reduced Scope 1 and 2 emissions by 87% since 2006, verified by
> SBTi."*

scores **0.0** on rhetoric — not because we're gullible, but because
that sentence is *structured to be falsifiable*. If it's a lie, you can
check it. That's the whole point.

---

## The architecture in one picture

```
  [User pastes text]
         │
         ▼
  ┌──────────────────────┐
  │ 1. Claim Extractor   │   splits sentences,
  │  (pure Python)       │   keeps sustainability ones,
  │                      │   tags category + type
  └──────────┬───────────┘
             │
             ▼
  ┌──────────────────────┐
  │ 2. Rhetoric Classifier│   flags Seven Sins,
  │  (pure Python)       │   rewards specifics + certs
  └──────────┬───────────┘
             │
             ▼
  ┌──────────────────────┐
  │ 3. LLM Verifier      │   one batched Groq call,
  │  (Groq API)          │   returns verdicts per claim
  │                      │   OR falls back to heuristic
  └──────────┬───────────┘
             │
             ▼
     ┌───────────────┐
     │ Blend scores  │   45% rhetoric + 55% verifier
     │ (max·70 +     │   blended across claims
     │  mean·30)     │
     └───────┬───────┘
             │
             ▼
       Overall score,
       risk band,
       per-claim ledger
```

---

## Glossary for the really non-technical panel member

- **Flask** — a small Python library for building web servers.
- **API** — "Application Programming Interface"; a way for one program to
  ask another program for something. We ask Groq's API for AI verdicts.
- **LLM** — "Large Language Model"; the kind of AI that powers ChatGPT.
- **Groq** — a company that runs open-source LLMs (like Llama) very fast.
- **Regex** — "regular expression"; a tiny pattern language for finding
  text. We use it to match sustainability keywords without false positives
  like "greenhouse" matching "green".
- **Word boundary** — a regex trick that means "the start or end of a
  whole word". It's why our matcher sees *green* and *greenway* as
  different words.
- **SBTi** — Science Based Targets initiative, a non-profit that
  independently validates corporate climate targets. One of the most
  credible third-party certifications.
- **Greenwashing** — presenting environmental claims in a way that is
  misleading or unverifiable. Often deliberate, sometimes accidental.

---

## Your 30-second elevator pitch for the viva

> "Corporate sustainability statements are often full of vague,
> unverifiable claims — a practice called greenwashing, which regulators
> in the US, UK, EU, and India are increasingly fining companies for.
>
> Our web app takes a sustainability statement, splits it into individual
> claims using natural language processing, scores each claim against the
> TerraChoice Seven Sins taxonomy, and cross-checks it against public
> record using a large language model called Llama 3.3 running on Groq.
> The result is a 0-to-100 greenwashing risk score with per-claim
> verdicts and explanations.
>
> We validated it on four well-documented cases — Delta, Microsoft, HSBC,
> and Ørsted — and the scores align with their real-world regulatory and
> reputational positions. The tool falls back gracefully to rule-based
> scoring when the AI service isn't available, so it's reliable for
> classroom, journalism, or investor use."

That's about 110 words. Practise saying it in 30 seconds, and you're
ready for the viva.
