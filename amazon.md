# 2.5-minute pitch script

Roughly 370 words. At a normal speaking pace that lands around 2 minutes 25
seconds, leaving a little room to breathe.

> **Before you use the AWS section:** it is written for after `npm run aws:check`
> passes against a real AWS account. Until then use the honest alternative at the
> bottom. Do not claim KMS or S3 in a room where someone may ask you to show it.

---

## 1 · The problem and what we built — *55 seconds*

If you've ever lived in a PG, you know this one.

You pay a forty-five thousand rupee deposit. At move-out the owner says the wall
is damaged, keeps fifteen thousand, and you can't argue — because the person
holding your money is also the person deciding whether to give it back.

And you can't sue. That's not worth two years in a civil court. So people just
absorb it.

We built **PG Escrow**. The deposit goes into a smart contract instead of the
owner's account. Both sides photograph the room at move-in, and those photos are
hashed onto the blockchain *before* anyone knows there'll be a dispute.

At move-out you photograph the same things again. If the owner claims damage and
the tenant disagrees, an AI panel reads the before-and-after photos and rules.

*(pause)*

The idea is simple: **separate who holds the money from who judges the outcome.**

---

## 2 · Tech stack and architecture — *55 seconds*

It's a TypeScript monorepo, end to end.

**Solidity** contracts hold the escrow. Every verdict is signed off-chain and
checked on-chain — the split has to equal the deposit exactly, and cite the
evidence locked in before the dispute started.

**Next.js** frontend, **Hono** API, **SQLite** through Drizzle.

The interesting part is the adjudicator. We run **three Gemini panelists** per
claim — one for the tenant, one for the owner, one deciding. How far the two
advocates disagree tells us how confident to be.

And here's the key design decision: **the AI never sees a rupee figure, and never
returns one.** It returns a fraction. A deterministic engine decides what that
fraction is a fraction of — capped by the item's depreciated value.

So a six-year-old mattress is worth zero, no matter what the model says.

*(pause)*

That's what makes it safe to run without a human in the loop.

---

## 3 · How we used AWS — *30 seconds*

**Build it —** we used the **AWS SDK for JavaScript v3**, and defined our
infrastructure and IAM permissions in a **CloudFormation** template.

**Ship it —** two services.

**AWS KMS** encrypts the wallet private keys our backend stores — each bound to
a user through an encryption context, decrypted only when we sign a transaction.
A stolen database is useless on its own.

**Amazon S3** holds the evidence photographs. Private bucket, SSE-KMS, and only
the tenant and owner can download them.

---

## Closing line — *5 seconds*

> Right now a tenant has no recourse. We're trying to make the fair outcome the
> default one.

---

## If AWS is not yet live

Swap section 3 for this. It is short, true, and much better than being caught
out:

> We've written the AWS integration — KMS for encrypting custodial wallet keys,
> S3 with SSE-KMS for evidence photographs, using the AWS SDK for JavaScript v3,
> with the resources defined in CloudFormation. It's tested against SDK mocks.
> We haven't provisioned the account yet, so I'd rather not claim it's running
> in production.

---

## Delivery notes

- **The line that lands is "you can't sue."** Slow down on it.
- **The mattress example** is your strongest technical point — it makes the
  bounding concrete in one sentence. Don't rush it.
- If you have 30 seconds spare, add: *"every verdict is published — all three
  panelists' reasoning, on a public audit page."*
- If you're cut to 90 seconds, drop section 2 to one line: *"TypeScript
  monorepo, Solidity contracts, Next.js, and a three-model Gemini panel."*
