# Recoup — 5-minute demo script

Razorpay AI Buildathon 2026 · Track 03 (AI Revenue Recovery)

**Timed, not estimated.** The narration below is **776 spoken words**. At 155 wpm — a normal
brisk demo pace — that is **5:00 of speech**, leaving about ten seconds for screen
transitions and the one deliberate pause in §4. If you speak nearer 140, use the cut list at
the bottom. Re-measure after any edit:

```bash
python -c "import pathlib;t=pathlib.Path('docs/DEMO_SCRIPT.md').read_text(encoding='utf-8');w=sum(len(l[2:].split()) for l in t.splitlines() if l.startswith('> '));print(w,'words ->',f'{w//155}:{round(w/155%1*60):02d} at 155 wpm')"
```

**Every figure is measured.** Batch `728b11e9ce70`, seed 42, 45-day clock. If you re-run the
pipeline before recording, check the numbers against `reports/scoreboard.md` rather than
trusting this file.

---

## Before you record

```bash
python -m deduction_desk report --compare agent,b0,b1,b2,b3
python -m deduction_desk export-web
cd web && npm run dev
```

- Browser at `localhost:5173`, **1440×900**, zoom 100%.
- A terminal window in `C:\Buildathon\deduction-desk`, large font.
- Open the case explorer once beforehand so `cases.json` is warm — it is a 2 MB fetch.
- Notifications off.

---

## §1 — The problem · 0:00–0:35

**SCREEN.** The hero. Let the 3D run three seconds before you speak.

> A one-lakh-rupee invoice comes back as ninety-two thousand four hundred.
>
> Nobody rang. Nobody disputed anything. One line in an email says *"TDS deducted"*, and
> eight thousand rupees is gone.
>
> Across this batch — four hundred invoices, forty-five crore billed — that's **one crore
> twenty-two lakh**, in three hundred and eight deductions.

**SCREEN.** Point at the three-way split under the hero.

> And here's the hard part: **about half of it the buyer was entitled to keep.** Tax they
> had to withhold by law. A rebate we agreed to.
>
> So the job isn't chasing the shortfall. It's **telling the halves apart.**

---

## §2 — One case, end to end · 0:35–1:15

**SCREEN.** Nav → **cases** → the pinned chip **26as lag**.

> One deduction, end to end, on one screen. That's how you check an agent's work.

**SCREEN.** Point at the raw advice, then at *what the model said*.

> A 7B model on this laptop reads it and says: TDS under 194J. Then the claim gets
> **checked, not trusted.**

**SCREEN.** Point at *what verification found* — `provisional valid`.

> We look it up in Form 26AS. It isn't there.
>
> The obvious move is to chase. **That would be wrong** — 26AS is filed quarterly, so a
> legitimate deduction simply isn't visible yet.
>
> Verdict: provisional valid. Re-check in ninety days, **send nothing.**

**SCREEN.** Zero messages sent. Then flip the ground-truth switch.

> And that's the answer key, written before the agent ran. Correct verdict, zero letters —
> because they did nothing wrong.

---

## §3 — How it decides · 1:15–1:55

**SCREEN.** Nav → **how it works**. Let the cyclone spin.

> Four hundred invoices in at the top, eighteen recoveries out the bottom. Every particle
> stops where the **real stage counts** say it stops — measured, not animated.

**SCREEN.** Hover a funnel stage so a readout appears.

> Fifty-one of the three hundred and eight, the model refused to classify — an abstention,
> reported beside the accuracy, never folded into it.

**SCREEN.** Nav → **guardrails**.

> Contact windows, consent, stopping rules, forbidden phrases — read live from the same
> config the agent reads.
>
> Compliance is checked **twice, by two implementations** — a gate blocks the action, and a
> separate auditor rebuilds every violation without importing that gate. If one piece of
> code did both, "zero violations" would prove nothing.

---

## §4 — The scoreboard, including where we lose · 1:55–2:40

**SCREEN.** Nav → **results**. Stay on the scatter.

> Five policies. Same invoices, same clock, same cost model, same customers. Only the
> decision changes.

**SCREEN.** Hover **Write to everyone**, then **Recoup**.

> Blanket dunning collects five lakh fifteen thousand. We collect two lakh sixty-one.
> Roughly half.
>
> **We lose that row, and there's no reading of the data that makes it go away.**

*(Pause two seconds. This is the most persuasive moment in the video — don't rush it.)*

> Here's what it cost them: **four hundred and seventy-three letters to customers who owed
> nothing.** We sent twenty.
>
> Count the money handed to a human with the evidence already attached, and we address
> twenty-one lakh seventy-eight against their nine lakh — **2.3 times more of the money,
> with a twenty-fourth of the wrong letters.**

**SCREEN.** Switch to the **harm** tab.

> Most submissions don't report this table at all.

---

## §5 — What broke, and how we got out · 2:40–4:25

**SCREEN.** Terminal with `docs/BROKE.md`, or straight to camera.

> Seventeen entries, written the day each one broke. Three worth your time.

### Break 1 — the number that wouldn't hold still

> Match rate on an identical batch measured 77.6%, then 98.4% three times, then 77.9% —
> same content hash, no code changes.
>
> I chased three wrong theories: leftover state, a caching bug, a race.
>
> The matcher built its candidate list by **iterating a Python set.** Python randomises
> string hashing per process, so ties broke differently every time the interpreter started.
> Twenty points of "money located" rode on that.
>
> The fix was `sorted()`. One word. **I'd already published three different figures before
> I found it** — so the determinism test now runs two separate subprocesses.

### Break 2 — the helpful hint that made the model worse

> I added a feasibility layer: work out which reason codes are possible, put them in the
> prompt. It should have helped. Macro-F1 went **down** — 0.554 to 0.512.
>
> I assumed noise on a forty-case slice. It wasn't — the matrix had collapsed onto exactly
> the two codes my hints happened to **name**, in sentences written to rule them *out*.
>
> A 7B model doesn't carry the conditional. **It carries the label.** Mention a code and
> you've suggested it.
>
> Hints now state facts and never name a code. That rule is a test. F1 went to **0.778**.

### Break 3 — the invariant that caught us flattering ourselves

> Building a chart, "addressed" came out **above** the reachable ceiling. Arithmetically
> impossible.
>
> Four cases had been paid twice — contacted day one and day four, both credits applied.
> Every policy had been over-reporting.
>
> Our own headline fell from four lakh ten to **two lakh sixty-one thousand.** We corrected
> it *downward*, in public.
>
> Four lakh was completely plausible on its own. It only fell over because a chart put two
> numbers with a **required relationship** side by side.

---

## §6 — Reproduce it · 4:25–5:00

**SCREEN.** Terminal. Run this live.

```bash
python -m deduction_desk report --compare agent,b0,b1,b2,b3
```

> Every model call runs on a local 7B — no API key, no GPU, no spend. The cache is in the
> repo, so `--offline` rebuilds all of this with the model switched off.

**SCREEN.** Scroll to the scoreboard table.

> Two hundred and twenty-three tests. Zero compliance violations under an independent
> audit. Zero messages actually sent.
>
> Ground truth sits in a table no agent module may import — a test fails the build if it
> appears. **The agent cannot move its own score.**

**SCREEN.** Footer sign-off.

> Every invoice was paid. Not every rupee arrived. Recoup works out which rupees you can
> actually ask for, and leaves the rest of your customers alone.

---

## If you run long

Cut in this order:

1. §3's compliance paragraph (−16s) — the site shows it anyway.
2. Break 3 (−28s) — Breaks 1 and 2 are the stronger pair.
3. The §2 answer-key reveal (−9s) — mention it instead of clicking it.

**Never cut** the pause after *"we lose that row"*. That admission is what makes every other
number in the video credible.

---

## Numbers cheat-sheet

| | |
|---|---:|
| Invoices / billed | 400 · ₹45.14 Cr |
| Short-paid | **₹1,22,03,318** across 308 deductions |
| Buyer was entitled to keep | ₹60,70,365 (49.7%) |
| Genuinely recoverable | ₹47,32,790 |
| Would actually pay if chased | ₹22,49,826 |
| Recoup — collected / addressed | ₹2,61,722 · **₹21,78,552** |
| Write-to-everyone — collected / addressed | ₹5,15,167 · ₹9,32,559 |
| Wrong letters — Recoup vs blanket | **20 vs 473** |
| Contacts sent | 147 vs 669 |
| Match rate | 83.5% (182/218), 40 exceptions published |
| Classified / abstained | 257 / 51 (16.6%) |
| Macro-F1, answered | 0.778 |
| Verification vs ground truth | 100% exact-rupee, 308/308 |
| Compliance violations | **0**, independent audit |
| Messages actually sent | **0** |
| Tests | 223 passing |

## Case ids used

| Beat | Case | Why |
|---|---|---|
| §2 main demo | `CASE-0376-0` (SHOW-26AS-LAG) | Valid 194J absent from 26AS → provisional close, no contact |
| Backup | `CASE-0191-0` (SHOW-CONTRADICT) | Advice says 2%, arithmetic says 5% — ₹14.23 L, routed to a human |
| Backup | `CASE-0359-0` (SHOW-RELATIONSHIP-STOP) | ₹1,200 owed by a ₹3 Cr account → stopping rule fires by name |
