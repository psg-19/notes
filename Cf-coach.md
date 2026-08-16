# NOTES — understanding and defending this project

Everything here is about *this* project, not ML in general. If you can explain
this file, you can hold a conversation about the work.

---

## 1. The 60-second version

> Codeforces rates every problem 800–3500, but only after people have attempted
> it, so recent problems sit unrated. I pulled all 11,356 problems from their
> API and trained a model to predict that rating from what's known at
> publication time — tags, position in the contest, division. LightGBM gets
> within 213 rating points on average, 69% better than guessing the mean, R²
> 0.85. The most interesting finding was that topic tags barely matter —
> position in the contest alone is 53% of the model's power, because setters
> already order problems by difficulty. Then I used the model to build a
> practice recommender: it rates the 305 unrated problems so they can be
> recommended, and finds which topics you're weakest at.

Practise saying that out loud. It's the whole interview in one paragraph.

---

## 2. The concepts, in order

### Supervised learning
You have rows where you already know the answer (11,051 problems with official
ratings). You show them to an algorithm, which finds patterns linking the
input columns to the answer. Then it can answer for rows you *don't* know
(the 305 unrated ones).

You're not writing the rules. You're providing examples and the algorithm
derives the rules.

### Features and target
- **Target** (`y`) — what we predict. Here: the `rating` column.
- **Features** (`X`) — the inputs. Here: 49 columns — position in contest,
  points, number of tags, year, one column per division, one per tag.

### One-hot encoding
Models do arithmetic; they can't read `"div. 2"` or `"dp|greedy"`. So a
`division` column becomes 8 columns (`div_div. 1`, `div_div. 2`, …) each
holding 0 or 1. Same for tags: 37 columns, 1 if the problem has that tag.

Why not just number them 1–8? Because that would imply Div. 4 is "twice"
Div. 2, which is meaningless. One-hot avoids inventing an order.

### Train / test split
We train on 8,840 older problems and test on 2,211 newer ones the model has
never seen. Testing on data it trained on would be like grading an exam where
you handed out the answers — it measures memory, not understanding.

**Why chronological rather than random?** Two reasons, and this is a very
likely question:
1. It matches the real job — rating problems that don't exist yet.
2. A random split would put problems from the *same contest* in both train and
   test. Problems in one contest share a setter, a theme and a difficulty
   curve, so the model could half-memorise the contest. That inflates the
   score without making the model better.

### The baseline
Always guess the average rating (1866). MAE 689.5.

This matters more than it looks. "MAE 213" means nothing on its own — 213 out
of what? The baseline is what makes the number interpretable: we cut error by
69%. **Always build the dumb baseline first.** A model that can't beat it has
learned nothing.

### The metrics
- **MAE** (Mean Absolute Error) = 212.5. On average we're off by 213 rating
  points, in either direction. The most intuitive one — quote this.
- **RMSE** (Root Mean Squared Error) = 309.9. Same idea, but squares errors
  before averaging, so big misses hurt disproportionately. RMSE > MAE always;
  the gap tells you there are some large misses.
- **R²** = 0.854. The share of variation in difficulty the model explains.
  1.0 is perfect, 0.0 is no better than the mean. 0.85 is strong.

### Linear regression
The simplest real model: one weight per feature, and it adds them up.
`rating = w1·position + w2·points + … + intercept`. Training means finding the
weights that minimise total error.

It got MAE 277 — much better than baseline. That tells you a lot of the signal
here is genuinely simple and additive.

Its weakness: it can only add. It cannot express "position matters much more in
Div. 1 than in Div. 3" — an *interaction* between features.

### Decision trees and LightGBM
A **decision tree** is a flowchart of yes/no questions:
`is index_num > 3?` → `is division Div. 1?` → predict 2400. Exactly the nested
`if` statements you'd write by hand — except the algorithm chooses the
questions and thresholds by trying them and keeping whichever splits the data
best.

**Gradient boosting** builds hundreds of trees *in sequence*. Tree 1 makes a
rough guess. Tree 2 is trained to predict tree 1's *errors*. Tree 3 predicts
what's still wrong. Each tree patches its predecessors' mistakes; the final
prediction sums them all.

**LightGBM** is a fast implementation. Ours used 319 trees and got MAE 212.5.

It beats linear regression because trees naturally capture interactions — a
branch can say "if Div. 1 *and* position > 4" without being told to look for
that combination.

### Overfitting and early stopping
Add enough trees and the model starts memorising quirks of the training data
that don't generalise — training error keeps falling while real-world error
rises. That's **overfitting**.

Guard: hold back a slice of training data as a **validation set**, and after
each tree check the error on it. When it stops improving for 100 trees, stop.
That's why we stopped at 319 rather than the 2000 we allowed.

Note there are three splits: **train** (fit the trees), **validation** (decide
when to stop), **test** (final honest score, touched once).

### Target leakage — the important one
`solved_count` improves MAE from 212.5 to ~123. Using it would be cheating:

1. Codeforces computes the official rating **from** how people performed, so
   `solved_count` is derived from the answer. Circular.
2. A brand-new problem has no solve count — so the model couldn't do its job.

**Leakage means a feature that contains information you wouldn't have at
prediction time.** It's the most common way beginner projects produce
impressive nonsense. The warning sign is a score that suddenly gets *too* good.

We kept the experiment in `train.py` deliberately, so the repo demonstrates
catching it rather than just claiming to know about it.

### Why coverage, not "difficulty reached"
For the recommender, the obvious weakness metric — "what rating have I reached
in each topic?" — is **confounded**. `implementation` problems average 1516 and
`fft` problems average 2861, so your reached-rating per topic mostly reflects
the topic's intrinsic difficulty. Rank by it and everyone's "weakest" topics
come out as `implementation`, `greedy`, `sortings`.

**Coverage** fixes it: inside your own difficulty band, what fraction of each
topic have you solved? That's comparable across topics. Tested on `tourist`,
this correctly surfaced `fft`, `flows` and `graph matchings`.

Being able to say "my first metric was wrong and here's how I noticed" is worth
more in an interview than any accuracy number.

---

## 3. Likely questions

**Why this project?**
> I've solved 1200+ problems on Codeforces, so I know the domain. I wanted a
> project where I collected the data myself rather than downloading a prepared
> CSV, and where I could sanity-check the model's output against my own
> judgment.

**Why LightGBM and not a neural network?**
> This is tabular data with 11K rows and 49 features. Gradient-boosted trees
> are the strong default there — they handle mixed feature types and missing
> values natively and train in seconds. Neural nets tend to win on text, images
> and audio, or on much larger tabular data. With 11K rows a net would most
> likely overfit while being harder to tune.

**Your model is 213 off. Is that good?**
> Relative to the 690 baseline, yes — 69% better. In context, 213 points is
> about one step on the Codeforces scale, roughly a Div. 2 B versus a Div. 2 C.
> It won't distinguish 1900 from 2000, but it reliably places a problem in the
> right band. For the actual use case — flagging roughly how hard an unrated
> problem is — that's useful.

**What's the biggest weakness?**
> It leans heavily on position in the contest, which is 53% of total gain. That
> means it's weakest exactly where it'd be most valuable — a standalone problem
> with no contest context. I'd want statement text features to fix that.

**How would you improve it?**
> Scrape the problem statements. Constraint sizes especially — `n ≤ 10^5`
> versus `n ≤ 10^18` tells you a lot about intended complexity, and none of
> that is in the API. I'd also try predicting a difficulty *band* rather than
> an exact number, since the ratings are quantised to hundreds anyway.

**Why MAE rather than accuracy?**
> Accuracy is for classification — right or wrong. This is regression, a
> continuous number, so "wrong" has a magnitude. Predicting 2100 when the truth
> is 2000 is nearly right; predicting 800 is not. MAE captures that.

**Did you tune the hyperparameters?**
Be honest:
> Only lightly — learning rate, number of leaves, and early stopping on a
> validation split. I didn't run a full search. Given the gap between baseline
> and model is so large, tuning wasn't the bottleneck; feature quality was.

**What would you do with more data?**
> More rows wouldn't help much — I have every rated problem that exists. The
> constraint is *columns*, not rows. That's why statement text is the next step.

---

## 4. Be honest about

- You built this in a few days while learning Python. Say so if asked. It's a
  learning project on public data, not a production system.
- The recommender is a **heuristic**, not a learned model, and is not evaluated
  against ground truth. Only the difficulty prediction is measured properly.
- You have not deployed it as a service.

Volunteering a limitation before someone finds it reads as competence.
Overclaiming and getting caught does the opposite.

---

## 5. Three-day plan

**Day 1 — Python + the data.**
Read `fetch_data.py` and `explore.py` line by line. Run them. Change things and
watch what breaks: chart a different column, change the `count >= 100` filter.
Python syntax will feel familiar coming from TypeScript; the new part is
pandas. Learn four things: `read_csv`, selecting columns, `groupby`, and
filtering with a boolean mask.

**Day 2 — the modelling.**
Read `train.py`. Make sure you can explain every section of §2 above. Then
experiment: delete `index_num` from `NUMERIC_FEATURES` and see how much MAE
degrades — that's a fast way to feel what a feature is worth.

**Day 3 — the recommender + rehearsal.**
Read `recommend.py`. Run it on your own handle and check whether the weak
topics match your intuition. Then say the 60-second summary out loud until it's
comfortable, and work through §3 without reading the answers.

If day 3 goes well and you want more, the natural addition is a small FastAPI
endpoint that loads `models/model.pkl` and returns a predicted rating for a
posted problem — about 10 lines, and it's ordinary backend work.
