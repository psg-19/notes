# DeFi Telegram Bot — Project Interview Guide (for **Omnissa** SDE Intern)

> **One-line pitch:** *A Telegram bot that gives every user a crypto wallet and lets them buy/sell 7 custom ERC-20 tokens with test-net Ethereum. It fetches live prices, signs on-chain transactions through Solidity smart contracts I wrote and tested with Foundry, and stores each user's private key **encrypted (AES)** in MongoDB.*

> ⭐ **This is your strongest project — lead with it.** It hits the exact themes Omnissa cares about: **cryptography & key management, secure client–server communication, distributed/external systems, concurrency, and a contract (interface) between components.** You can go deep, so do.

This guide has 4 parts:
1. **Deep explanation** of the whole system, framed for an **Omnissa** interviewer.
2. **§11 — The conversational Q&A you'll ACTUALLY get** (explain it / problem / stack / architecture / challenges). ⭐ *Start here — this is what most project rounds are.*
3. **§12 — Deep code-level Q&A** (bonus, only if the interviewer goes technical).
4. **A 1-page revision cheat sheet** for the night before.

---

## 0. First — who is Omnissa, and why is *this* the project to lead with?

**Omnissa** is VMware's former **End-User Computing** business (Workspace ONE, Horizon VDI, App Volumes) — enterprise software for **securely delivering apps/desktops to any device.** So they hire for **security, identity, networking, distributed systems, OS/concurrency, and clean OOP design** (they're a **Java/Spring** shop).

| What Omnissa values | Where THIS project proves it |
| --- | --- |
| **Security & key management** (their core business) | You **generate, encrypt-at-rest (AES), and custody private keys** — and can discuss the trade-offs honestly. This is the single best part. |
| **Networking & protocols** | **Telegram long-polling**, **JSON-RPC** to an Ethereum node (Alchemy), **REST** to CoinGecko, **TLS** everywhere. You can compare **polling vs webhooks**. |
| **Concurrency / OS** | Node **event loop**, `async/await`, and **`Promise.all`** to fetch 8 balances in **parallel**. An in-memory **state machine** (`userBuyState`) handles multi-step conversations. |
| **Distributed systems / failure** | Money flows across **3 external systems** (Telegram, blockchain, price API). You can talk **atomicity, retries, idempotency, single points of failure.** |
| **Interfaces / OOP** | Solidity **`interface ITOKEN`**, a **master contract** owning child token contracts, the **Ownable** access-control pattern — clean abstraction boundaries. |
| **Testing** | **Foundry** unit tests + a **10,000-run fuzz** profile. Real tests — lean on them. |
| **DSA + complexity** | Price math, state map (O(1) lookups), parallel vs sequential I/O. |

**Golden rule:** This is **money software** running across **untrusted networks**. Whenever you explain something, add a line about **security, correctness, or what happens on failure.** That's precisely the mindset Omnissa hires for.

---

## 1. The pitch (memorize)

**30-second version:**
> "It's a Telegram bot for trading crypto on a testnet. When you `/start`, the bot **generates an Ethereum wallet** for you, encrypts the private key, and stores it in MongoDB. Through an inline-button menu you can **buy** tokens (you send test ETH, a smart contract mints you tokens at the live market rate), **sell** them (the contract burns your tokens and sends ETH back), **check balances**, get your **public key**, or **eject** (export + delete) your private key. I wrote 7 ERC-20 token contracts plus a **master contract** in Solidity and tested them with Foundry."

**2-minute version:** add —
> "The bot server is **TypeScript on Node** using `node-telegram-bot-api` in long-polling mode, talking to the Ethereum node via **ethers.js** over **JSON-RPC** (Alchemy), and to **CoinGecko's REST API** for live prices. The architecture has a clean trust boundary: each token contract's **ownership is transferred to the master contract**, so only the master can mint/burn, and only my server's owner key can call the master. The trickiest part is the **buy flow** — it's a two-step money operation (user pays ETH, then owner mints tokens), which raises real atomicity questions I can talk about."

---

## 2. Architecture (draw this — it impresses)

```
   ┌────────────┐   long-polling (HTTPS)   ┌──────────────────────────────┐
   │  Telegram   │ ◄──────────────────────► │   BOT SERVER (Node + TS)     │
   │   user      │   getUpdates / sendMsg   │  • node-telegram-bot-api     │
   └────────────┘                           │  • userBuyState (state m/c)  │
                                            │  • ethers.js (signer)        │
                                            └───┬───────────┬───────────┬──┘
                                                │           │           │
                              JSON-RPC (Alchemy)│   REST    │  Mongoose │
                                                │ (prices)  │           │
                                   ┌────────────▼──┐  ┌─────▼─────┐ ┌──▼──────────┐
                                   │ Ethereum       │  │ CoinGecko │ │  MongoDB    │
                                   │ (Sepolia test) │  │   API     │ │ {userId,    │
                                   │                │  └───────────┘ │ encPrivKey, │
                                   │ ┌────────────┐ │                │ publicKey}  │
                                   │ │ Master     │ │                └─────────────┘
                                   │ │ Tokens_To_ │ │
                                   │ │ Server     │ │  owns ──► [USDC][USDT][BNB]
                                   │ │ (Ownable)  │ │           [WBTC][SHY][PEPE][SHIB]
                                   │ └────────────┘ │           (7 ERC-20 contracts)
                                   └────────────────┘
```

**Mental model:** an **event-driven gateway** (the bot) that orchestrates **three external systems** — a **blockchain** (state + money), a **price oracle** (CoinGecko), and a **database** (key custody). Being able to say "orchestration across external systems with a trust boundary" is exactly the systems vocabulary Omnissa wants.

---

## 3. Tech stack & *why* (justify every choice)

| Layer | Tech | Why (say this) |
| --- | --- | --- |
| Bot | `node-telegram-bot-api` (**polling**) | Telegram is the UI — no need to build a frontend; polling is simplest to run without a public HTTPS endpoint. |
| Language | **TypeScript** | **Static types** catch bugs at compile time — essential when bugs mean lost funds. Closest cousin to Java (interfaces, generics, OOP). |
| Blockchain SDK | **ethers.js v6** | Clean API for wallets, signing, JSON-RPC providers, and contract calls. |
| Node access | **Alchemy** (JSON-RPC) | Managed Ethereum node so I don't run my own; speaks standard JSON-RPC. |
| Prices | **CoinGecko REST API** | Free live price feed (token → ETH). Acts as my price oracle. |
| Contracts | **Solidity + OpenZeppelin** | Battle-tested `ERC20` + `Ownable` base classes — don't roll your own token/access-control. |
| Testing | **Foundry (forge)** | Fast Solidity tests in Solidity; supports **fuzzing** (10k runs) and cheatcodes (`deal`, `prank`). |
| DB | **MongoDB + Mongoose** | Simple key-value-ish store: `userId → {encryptedPrivateKey, publicKey}`. |
| Crypto | Node **`crypto`** (AES, `createCipheriv`) | Symmetric encryption of private keys at rest. |
| API/health | **Express** | Tiny HTTP server (health check / keep-alive on port 3000). |

---

## 4. End-to-end flows (this is where you win — know these cold)

### 4.1 `/start` → wallet creation — [BOT-Server/src/createKeypair.ts](BOT-Server/src/createKeypair.ts)
1. If the user already exists (by `chatId`), return their public key (idempotent — `/start` twice is safe).
2. Else `ethers.Wallet.createRandom()` generates a fresh keypair.
3. **Encrypt the private key** with AES (`encrypt()`), store `{userId, encryptedPrivateKey, publicKey}` in MongoDB.
4. Return the **public key** to the user.

> 🔑 **Key idea to say:** "The public key is the user's address; the private key is the secret that authorizes spending. I never store the private key in plaintext — it's **encrypted at rest**."

### 4.2 Encryption / decryption — [BOT-Server/Utils/encrypt.ts](BOT-Server/Utils/encrypt.ts), [decrypt.ts](BOT-Server/Utils/decrypt.ts)
- Symmetric **AES** via `crypto.createCipheriv(ALGORITHM, key, iv)` — key and IV are hex strings from env vars.
- Encrypt: utf8 → hex; Decrypt reverses it.

> ⚠️ **Two honest weaknesses (raise them — Omnissa is a security company):**
> 1. **The IV is fixed/reused** for every user (it comes from one env var). With AES-CBC, **reusing an IV leaks information** (identical plaintexts → identical ciphertext) and is a textbook crypto mistake. Fix: a **random IV per encryption**, stored alongside the ciphertext.
> 2. **It's symmetric**, so whoever holds the env key can decrypt *every* user's key — a **single point of compromise**. Better: a **KMS/HSM**, envelope encryption, or non-custodial design (never hold user keys at all).

### 4.3 Buy flow (the crown jewel) — [BOT-Server/src/getToken.ts](BOT-Server/src/getToken.ts)
1. User picks **Buy** → bot checks the user's **ETH balance** via `provider.getBalance`. If 0, it sends the **Sepolia faucet** link and the user's address. *(Can't transact without gas — good "you need gas for any state change" point.)*
2. Bot shows **live prices** for all 7 tokens (CoinGecko), then sets `userBuyState[chatId] = 'USDC_buy'` and uses **`force_reply`** to ask "how much ETH?".
3. On the reply, input is validated with a **regex** (`/^[0-9]*\.?[0-9]+$/`).
4. `getTokens()` computes: `tokensToMint = (1 / priceTokenInEth) * ethAmount`.
5. **Decrypt** the user's private key → build the user's wallet → check they have enough ETH.
6. **Step A:** the user's wallet **sends ETH** to the owner/contract address.
7. **Step B:** the **owner's wallet** calls `contract.GetToken(userAddr, tokensToMint, token)` which **mints** the tokens to the user (payable).
8. Reply with the **transaction hash**.

> ⚠️ **Atomicity = your best systems talking point:** Steps A and B are **two separate transactions**. If A succeeds (user pays ETH) but B fails (mint reverts), the **user is charged but gets no tokens.** This is a **distributed-transaction / two-phase-commit** problem. Fixes: do the payment **inside** the contract call (`GetToken` is already `payable` — send the ETH *with* the mint call so it's one atomic on-chain transaction), or add **idempotent retries / refund-on-failure**.

### 4.4 Sell flow — [BOT-Server/src/burnToken.ts](BOT-Server/src/burnToken.ts)
1. Read the user's token balance from the contract (`GetBalances`).
2. If they have enough, compute `ethToReturn = price * amount`.
3. Owner calls `contract.BurnToken(user, tokenAmount, ethAmount, token)` → contract **burns** the tokens and **transfers ETH** back to the user.
4. Reply with the tx hash.

### 4.5 Balances — [BOT-Server/src/getUserBalance.ts](BOT-Server/src/getUserBalance.ts)
- Uses **`Promise.all`** to fire **8 reads in parallel** (7 token balances + native ETH) instead of awaiting them one-by-one.

> 💡 **Concurrency talking point:** "Sequential `await`s would take the **sum** of 8 round-trips; `Promise.all` overlaps the I/O so it takes roughly the **max** of one. On a single-threaded event loop, that's the right way to parallelize I/O-bound work." This is a *fantastic* OS/concurrency answer.

### 4.6 Eject (export + delete key) — in [BOT-Server/index.ts](BOT-Server/index.ts)
- Shows a Yes/No confirmation → on Yes: **decrypt** the private key, **delete** the user from MongoDB, and **send the private key in chat**.

> ⚠️ Sending a plaintext private key over a chat message is risky (it lives in Telegram's history/servers). Fine for a testnet demo; mention you'd warn the user and never do this with real funds.

### 4.7 The conversational state machine — `userBuyState` in [BOT-Server/index.ts](BOT-Server/index.ts)
- A bot is **stateless per message**, but buying needs two steps (pick token → enter amount). `userBuyState[chatId] = 'USDC_buy'` remembers "this user is mid-buy"; the next text message is interpreted as the amount, then the state is cleared.

> ⚠️ **Scale weakness:** it's an **in-memory object**, so (a) state is **lost on restart** and (b) it **breaks if you run multiple bot instances** (a user's two messages could hit different servers). Fix: store conversation state in **Redis/MongoDB** keyed by `chatId` → makes the server **stateless and horizontally scalable**. This is a great distributed-systems answer.

---

## 5. The smart contracts (know the design + the *why*)

### 5.1 The token contracts — [Smart-Contracts/Tokens/USDC.sol](Smart-Contracts/Tokens/USDC.sol) (×7)
- Each extends OpenZeppelin **`ERC20`** + **`Ownable`**, with `mint`/`burn` guarded by **`onlyOwner`**.

### 5.2 The master contract — [Smart-Contracts/src/Tokens_To_Server.sol](Smart-Contracts/src/Tokens_To_Server.sol) ⭐
- Holds `mapping(string => address) Tokens` (name → token contract) and tracks `ETH_BALANCES` + `TotalEth`.
- `AddToken`, `GetBalances`, `GetToken` (mint + accept ETH), `BurnToken` (burn + return ETH) — all `onlyOwner`.
- Calls tokens through an **`interface ITOKEN`** (mint/burn/balanceOf).

> 🔑 **The key design move:** after deploying each token, you **transfer its ownership to the master contract** (you can see it in the test: `usdc.transferOwnership(address(c))`). So:
> - Only the **master** can mint/burn any token (single choke point for policy).
> - Only the **server's owner key** can call the master.
>
> This is a clean, layered **access-control / authorization** model — exactly the kind of "trust boundary" reasoning Omnissa likes. Compare it to RBAC: the master is the privileged service account; tokens trust only it.

### 5.3 Testing with Foundry — [Smart-Contracts/test/Contract.t.sol](Smart-Contracts/test/Contract.t.sol)
- `test_GetToken`: add token → transfer ownership to master → mint via `GetToken{value:1 ether}` → assert balance.
- `test_BurnToken`: uses Foundry **cheatcodes** `deal(...)` to fund addresses, mints, then burns and asserts balance returns to 0.
- **Fuzzing:** `foundry.toml` sets `[profile.ci.fuzz] runs = 10_000` — property-based testing with thousands of random inputs.

> 💡 "I tested the contracts because they handle value and are **immutable once deployed** — you can't hotfix a bug. Fuzzing throws random inputs to find edge cases I wouldn't think of." That sentence alone signals real engineering maturity.

---

## 6. Security deep-dive (Omnissa WILL go here — be the one who raises the issues)

| Area | What you did | What you'd improve |
| --- | --- | --- |
| Key generation | `ethers.Wallet.createRandom()` (CSPRNG) | Good. |
| Key at rest | AES-encrypted in MongoDB | **Random IV per record**; move the master key to a **KMS/HSM**; or go **non-custodial** (never hold keys). |
| Authorization (chain) | `onlyOwner` on tokens + master; tokens owned by master | Solid layered model; could add per-action roles. |
| Single point of failure | One **owner private key** controls all mint/burn | Multisig / threshold signing; least-privilege keys. |
| Atomicity | Buy = pay-then-mint (2 txns) | Make it **one atomic on-chain call** (send ETH with the mint), or refund-on-failure. |
| Input validation | Regex on amounts | Add bounds, decimals, and **nonce/replay** handling. |
| Key export | Eject sends plaintext key in chat | Warn user; never for mainnet. |
| Network | TLS to Telegram/Alchemy/CoinGecko | Add **secrets management** instead of `.env`. |
| Scope | **Sepolia testnet only** (disclaimer in bot) | Explicitly *not* real money — a deliberate safety choice. |

> **Power line:** "Because I'm custodying private keys with a single symmetric key and a reused IV, the most important hardening is **key management**: random IVs, a KMS, and ideally moving to a **non-custodial** model so a server breach can't drain anyone. For a security company like Omnissa, that's the first conversation I'd want to have about this project."

---

## 7. CS fundamentals this project demonstrates (name them)

- **Cryptography:** symmetric (AES) vs asymmetric (the ECDSA keypair); **encryption at rest**; **digital signatures** (every on-chain txn is signed by the private key); hashing; IV/nonce concepts.
- **Networking:** **polling vs webhooks**; **JSON-RPC** vs **REST**; request/response over **TLS**; latency and why you **parallelize I/O**.
- **OS / Concurrency:** single-threaded **event loop**, **non-blocking I/O**, `async/await` (microtask queue), **`Promise.all`** parallelism; an in-memory **state machine**.
- **Distributed systems:** orchestration across 3 external services; **atomicity / two-phase-commit**; **idempotency**; **single point of failure**; **stateless vs stateful** services (the `userBuyState` problem).
- **OOP / design:** Solidity **interfaces** (`ITOKEN`), **inheritance** (`ERC20`, `Ownable`), the **owner/access-control** pattern, a **facade/master** contract; TypeScript types/interfaces.
- **DSA:** hash-map lookups (`Tokens`, `userBuyState`), price arithmetic, parallel vs sequential complexity.

---

## 8. "What I'd do differently" (your ownership story — pick 2–3)

1. **Make buy atomic** — send ETH *with* the mint call so payment + mint succeed or fail together.
2. **Random IV per encryption** + move the master key to a **KMS**; ideally go **non-custodial**.
3. **Externalize conversation state** to Redis → stateless, horizontally scalable bot.
4. **Switch polling → webhooks** for lower latency and better scale once there's a public HTTPS endpoint.
5. **Cache prices** (CoinGecko has rate limits) with a short TTL instead of calling on every action.
6. **Fix the real bug** in [getTokenPrice.ts](BOT-Server/src/getTokenPrice.ts): it always reads `response.data['usd-coin']` regardless of the requested token — so non-USDC prices are wrong. (Spotting your own bug is a *huge* credibility signal.)
7. **Nonce/queue management** so concurrent transactions from the owner key don't collide.

---

## 9. Scalability / system design ("how would this handle 100k users?")

- **Stateless bot** → move `userBuyState` to **Redis**, run **N instances** behind a load balancer; switch to **webhooks** so Telegram pushes updates and any instance can handle them.
- **Price oracle:** cache CoinGecko responses (Redis, ~10–30s TTL) to avoid rate limits and cut latency.
- **Blockchain throughput:** the single owner key serializes transactions — add a **signing service** with **nonce management** and a **transaction queue**, or shard across multiple owner keys.
- **DB:** index on `userId`; it's a simple key-value access pattern, so it scales well and could even be Redis/DynamoDB-style.
- **Reliability:** make money operations **idempotent + retryable**; record intent before acting so you can recover after a crash (the atomicity story).

---

## 10. TS → Java/OOP bridge (Omnissa is a Java shop)

| This project | Java equivalent (say this) |
| --- | --- |
| TypeScript interfaces / types | Java **interfaces / generics** |
| Solidity `interface ITOKEN` | Java **interface** (contract between classes) |
| `Ownable` / `onlyOwner` | Spring Security **role check** / `@PreAuthorize` |
| `async/await`, `Promise.all` | `CompletableFuture` / `allOf(...)` |
| Mongoose model | Spring Data **Repository** + `@Document`/`@Entity` |
| `userBuyState` map → Redis | Server-side session / distributed cache |
| Bot event handlers | Event listeners / message-driven beans |

> "It's TypeScript + Solidity, but it's all OOP: interfaces, inheritance, access control, and async orchestration. The Java/Spring version would be the same architecture — repositories, a signing service, an auth filter, and an interface-based contract layer."

---

## 11. The questions you'll ACTUALLY get asked (high-level — interviewer hasn't read your code) ⭐

> These are the *real* project questions: explain it, the problem, the stack & why, the architecture, the workflow, and the challenges. The interviewer is testing **how you think and communicate** — they will not read your codebase. Practice these until they flow like a story. **This section matters more than §12.**

**Q1. Tell me about this project / walk me through it.**
> "It's a Telegram bot that lets anyone trade cryptocurrency right inside a chat. When you message `/start`, the bot **creates an Ethereum wallet for you** behind the scenes and securely stores it. Then through a button menu you can **buy** custom tokens (you pay with test Ethereum and a smart contract issues you the tokens at the live market price), **sell** them back for Ethereum, **check your balances**, or **export your private key** and leave. I built 7 of my own tokens as smart contracts in Solidity, plus a 'master' contract that controls them, and I tested those contracts thoroughly. It runs on a **test network**, so no real money is involved — it's a safe, working demo of how a custodial crypto exchange works end to end." *(Pause for follow-ups.)*

**Q2. What problem does it solve, and why did you build it?**
> "Using crypto normally has a steep learning curve — you install a wallet, secure a seed phrase, fund it, connect to a decentralized exchange. I wanted to see if I could make buying and selling tokens as easy as **sending a chat message**, with the wallet handled for the user. Honestly, the bigger driver was learning: I wanted to understand how blockchains, wallets, smart contracts, and price feeds actually fit together by building a real transactional system rather than reading about it. The chat interface was the simplest possible front end so I could focus on the hard parts — keys, contracts, and money flow."

**Q3. What's your tech stack, and why did you choose each part?**
> "The bot server is **TypeScript on Node.js**. I picked **TypeScript** specifically because this is money software — static types catch a whole class of bugs at compile time before they can cause a wrong transaction. I used the **Telegram Bot API** so I didn't have to build a front end at all. **ethers.js** is the library that talks to the blockchain — creating wallets, signing transactions, calling contracts. I used **Alchemy** as my managed Ethereum node so I didn't have to run my own. **CoinGecko's API** gives me live token prices. The smart contracts are **Solidity** built on **OpenZeppelin's** audited token templates, and I tested them with **Foundry**. User data sits in **MongoDB**."

**Q4. Why did you use smart contracts at all? And why your own tokens?**
> "The smart contracts *are* the product — they're the trustless rules that actually mint tokens when someone pays and burn them when someone sells. I created my own ERC-20 tokens so I had full control to mint and burn for the demo without needing real liquidity. The interesting design decision was a **master contract that owns all 7 token contracts**, so only the master can change token supply, and only my server can call the master — that's a clean, layered permission model, like having one privileged admin service instead of scattering control everywhere."

**Q5. Walk me through the overall architecture.**
> "Think of the bot server as an **orchestrator** sitting between four systems. The user talks to it through **Telegram**. To do anything financial, it talks to the **Ethereum blockchain** (via Alchemy) to read balances and run transactions through my smart contracts. It calls **CoinGecko** to get live prices so the exchange rate is fair. And it uses **MongoDB** to store each user's wallet — specifically their public address and their **encrypted** private key. So the bot itself holds almost no logic about *value*; the rules live in the smart contracts, and the bot's job is to coordinate the conversation and the calls."

**Q6. Describe the end-to-end workflow — what happens when someone buys a token?**
> "The user opens the menu and picks Buy. The bot first checks they have some test-Ethereum for transaction fees — if not, it sends them a faucet link to get free test ETH. It shows current prices, then asks how much ETH they want to spend. It calculates how many tokens that buys at the live rate, then runs the transaction: the user's wallet pays the ETH, and the master smart contract mints the tokens to the user's address. The bot replies with the transaction hash, which is the on-chain receipt. Selling is the mirror image — the contract burns the tokens and sends ETH back."

**Q7. What was the most challenging part, and how did you solve it?**
> "**Safely handling users' private keys.** A private key is the one secret that controls someone's funds — if it leaks, the money's gone. I couldn't store it in plain text, so I learned about encryption and **encrypted every private key before saving it** to the database, decrypting it only in memory when a transaction needs signing. The deeper lesson came afterward: I realized my approach still has weaknesses — I reuse the same encryption initialization vector, and because it's one symmetric key, whoever holds it could decrypt everyone's keys. So I can now explain *why* the gold standard is a dedicated key-management service, or better, a **non-custodial design where you never hold the user's key at all**. Thinking through key custody was the most valuable part of the whole project."

**Q8. Tell me about another challenge — making the bot 'remember' a conversation.**
> "A bot is **stateless** — each message arrives on its own with no memory of the last one. But buying is a two-step conversation: first you pick a token, then you type an amount. I solved it with a small **state machine** — when you pick 'buy USDC', I record that you're mid-purchase, so your next message is interpreted as the amount, and then I clear that state. It worked, but it taught me a real systems lesson: I kept that state **in memory**, which means it's lost if the server restarts and breaks if I run multiple servers. The proper fix is to keep conversation state in a shared store like Redis so the service can be **stateless and scaled horizontally**."

**Q9. A third challenge — getting the money flow correct.**
> "The buy is actually two separate steps: the user pays ETH, then my contract mints the tokens. I hit the classic problem — what if the payment goes through but the minting fails? The user would be charged and get nothing. That's a **distributed-transaction / atomicity** problem, and working through it is how I learned about two-phase commit and idempotency. The right fix is to make payment and minting a **single atomic on-chain transaction** so they either both succeed or both fail — my mint function already accepts the payment, so I'd send the ETH together with the mint call."

**Q10. What was hard about the smart contracts specifically?**
> "Two things. First, **you can't patch them** — once a contract is deployed it's immutable, so a bug is permanent and could cost money. That's why I tested them carefully with Foundry, including **fuzz testing**, which throws thousands of random inputs to find edge cases I'd never think of by hand. Second, **decimal handling** — token amounts on-chain are huge integers (18 decimals), so converting between what the user types and what the contract expects was fiddly and a common source of bugs. Getting unit conversion right taught me to respect how money is represented in code."

**Q11. What would you do differently if you built it again?**
> "Top of the list is **key management** — random IVs, a proper key-management service, or going fully non-custodial. Then making the buy **atomic**, moving conversation state to **Redis** so the bot scales, switching from polling to **webhooks** for lower latency, and **caching prices** so I don't hammer the price API. And I'd put the smart-contract tests in CI so they run on every change."

**Q12. What did you learn from this project?**
> "A huge amount about **security and systems thinking** — cryptography, signing, key custody, and what 'atomic' really means when money moves across multiple systems. I also learned to **integrate several external services** that can each fail independently, and to **test code you can't afford to get wrong**. Most of all, I learned to look at my own design and honestly name where it's not secure or won't scale — which is the mindset I think real engineering needs."

**Q13. Did you build it alone, and how long did it take?**
> *(Answer truthfully — adjust.)* "Yes, solo — which is why I understand every layer, from the Solidity contracts up to the Telegram interface. Building it alone forced me to learn blockchain, backend, and security together instead of just one slice."

**Q14. What are you most proud of?**
> "That it's a **complete, working transactional system** — real wallets, real on-chain mint/burn, live prices — and that I tested the value-handling parts properly with fuzzing. And that I can give an honest security review of my own project: here's what's solid, here's exactly what I'd harden before it ever touched real money. For a security-focused company, that self-awareness is what I'd want to show."

**Q15. How would you extend it / what's the future scope?**
> "Make it **non-custodial** so users keep their own keys; support real networks with proper safeguards; add a transaction history and price charts; multi-signature control of the admin key so there's no single point of failure; and a signing service with proper queueing so it scales to many concurrent users. Each of those is a step from 'student demo' toward 'production financial system,' and I can explain the trade-off at every step."

---

## 12. Deep code-level Q&A (bonus — only if the interviewer goes technical)

> Use these if the interviewer digs into implementation. For most rounds, §11 is enough.

**Q1. Walk me through what happens when a user buys a token.**
> The bot checks the user has ETH for gas; if not it sends the faucet link. It shows live prices, then asks for an ETH amount via a forced reply and validates it with a regex. It computes `tokens = (1/price) * eth`, decrypts the user's private key, and does two on-chain steps: the **user's wallet sends ETH**, then the **owner's wallet calls the master contract's `GetToken`**, which mints the tokens to the user. The bot replies with the transaction hash.

**Q2. Public key vs private key — explain like I'm new.**
> The keypair is asymmetric. The **public key/address** is like an account number — safe to share, it's where funds live. The **private key** is the secret that **signs** transactions to authorize spending — anyone with it controls the funds. So the whole security problem is protecting that private key, which is why I encrypt it at rest.

**Q3. How and where do you store private keys, and what are the risks?**
> Encrypted with **AES** (Node `crypto`) and stored in MongoDB keyed by Telegram chatId. Two risks I'm aware of: I **reuse a single IV**, which is a crypto anti-pattern (identical plaintexts produce identical ciphertext), and it's **symmetric with one master key**, so a breach of that key exposes everyone. I'd use a **random IV per record**, a **KMS/HSM** for the key, and ideally a **non-custodial** design where I never hold keys at all.

**Q4. Symmetric vs asymmetric encryption — where is each used here?**
> **Asymmetric** (ECDSA) is the wallet itself — sign with the private key, verify with the public key. **Symmetric** (AES) is how I encrypt that private key at rest, because symmetric is fast and I only need to decrypt it server-side. Different jobs: asymmetric for identity/signatures, symmetric for bulk encryption.

**Q5. Your buy is two transactions. What's the failure mode and how do you fix it?**
> If the user's ETH payment succeeds but the mint reverts, the **user paid and got nothing** — a broken distributed transaction. The clean fix is to make it **one atomic on-chain call**: `GetToken` is `payable`, so send the ETH *with* the mint call — then it's all-or-nothing on chain. Otherwise I'd need **idempotent retries** and **refund-on-failure** logic.

**Q6. Why a master contract that owns all the token contracts?**
> It's a layered **access-control** design. Each token's `mint`/`burn` is `onlyOwner`, and I **transfer each token's ownership to the master**. So only the master can change supply, and only my server's owner key can call the master. One choke point to enforce policy, and a clear trust boundary — like a privileged service account in RBAC.

**Q7. What is `Promise.all` doing in your balance check, and why does it matter?**
> I fetch 8 balances (7 tokens + ETH) **in parallel** with `Promise.all` instead of awaiting each in sequence. Sequentially the latency is the **sum** of 8 network round-trips; in parallel it's about the **max** of one. On Node's single thread, overlapping I/O like this is the correct way to speed up I/O-bound work.

**Q8. How does a single-threaded Node server handle many users?**
> Through the **event loop** and **non-blocking I/O**. While one request waits on the blockchain or DB, Node serves others; when the I/O completes, the callback runs as a microtask. It's ideal for **I/O-bound** work like this. CPU-heavy work would block the loop, so I'd offload that to workers or another service.

**Q9. Polling vs webhooks — which do you use and what's the trade-off?**
> I use **long-polling** (`node-telegram-bot-api`): the bot repeatedly asks Telegram for updates. It's simple and needs no public HTTPS endpoint, great for development. **Webhooks** are push-based — Telegram POSTs updates to my URL — which is lower-latency and scales better, but needs a public TLS endpoint. For production I'd switch to webhooks.

**Q10. JSON-RPC vs REST — you use both. What's the difference?**
> **REST** (CoinGecko) is resource-oriented over HTTP verbs/URLs. **JSON-RPC** (Alchemy/Ethereum) is a remote-procedure-call protocol — you POST a method name like `eth_getBalance` with params and get a result. ethers.js wraps the JSON-RPC calls so I work with nice methods instead of raw payloads.

**Q11. How do you keep multi-step conversation state, and what's wrong with it?**
> An in-memory map `userBuyState[chatId]` acts as a **state machine**: it remembers a user is mid-buy so the next message is read as the amount. The problem: it's **in-memory**, so it's **lost on restart** and **breaks across multiple instances**. I'd move it to **Redis** keyed by chatId, which makes the bot **stateless and horizontally scalable**.

**Q12. Why did you test the smart contracts, and how?**
> Contracts handle value and are **immutable once deployed** — you can't patch a bug, so testing is non-negotiable. I used **Foundry**: unit tests in Solidity (mint, burn, balance assertions) with cheatcodes like `deal` to fund accounts, plus a **10,000-run fuzz** profile that throws random inputs to surface edge cases.

**Q13. What is fuzz testing and why is it valuable here?**
> Fuzzing runs a test with **many random inputs** and checks that **properties** always hold (e.g. "balance never goes negative"). It finds edge cases a human wouldn't enumerate — overflow, weird amounts, ordering. For financial contract code, that breadth of coverage is exactly what you want.

**Q14. Why do users need ETH even to use tokens?**
> Every **state-changing** transaction on Ethereum costs **gas**, paid in ETH, to compensate validators and prevent spam. So even to buy/sell tokens the user needs some ETH for gas — which is why the bot checks the ETH balance first and links the **Sepolia faucet** if it's zero.

**Q15. What's the single biggest security risk in this system?**
> **Key custody.** I hold every user's private key, encrypted with one symmetric key and a reused IV. A breach of that env key would expose everyone. The right answers are random IVs, a KMS/HSM, least-privilege keys, and ideally a **non-custodial** model so I never hold the secret at all.

**Q16. Is there a bug you know about in the code?**
> Yes — `getTokenPrice.ts` always reads `response.data['usd-coin']` regardless of which token was requested, so any non-USDC price is wrong. The main flows use a correctly-parameterized version, but that helper is buggy and I'd fix it to index by the actual token id. *(Volunteering this is gold.)*

**Q17. How would you scale this to 100k users?**
> Make the bot **stateless** (state in Redis), run multiple instances behind a load balancer, and switch to **webhooks**. **Cache prices** to dodge CoinGecko rate limits. On chain, add a **signing service with nonce management and a transaction queue** so the owner key's transactions don't collide. The DB is a simple keyed lookup, so it scales easily with an index on `userId`.

**Q18. ERC-20 — what is it and why use OpenZeppelin?**
> ERC-20 is the **standard interface** for fungible tokens — `transfer`, `balanceOf`, `approve`, etc. — so wallets/exchanges can treat any token uniformly. I extended **OpenZeppelin's** audited `ERC20` and `Ownable` base contracts instead of writing my own, because security-critical primitives should be battle-tested, not hand-rolled.

**Q19. What's the role of the owner private key, and why is it risky?**
> It's the server's key that calls the master contract to mint/burn — effectively the system's admin. The risk is it's a **single point of failure**: lose it or leak it and an attacker controls all token supply and the contract's ETH. Mitigations: a **multisig**, threshold signatures, and least-privilege separation.

**Q20. If you rebuilt this for an enterprise like Omnissa, what changes?**
> **Key management** first: KMS/HSM, random IVs, or non-custodial. **Atomicity**: single on-chain payment+mint, idempotent retries. **Scale**: stateless via Redis, webhooks, price caching, a signing service with nonce control. **Observability**: structured logs, metrics, tracing on every money operation. **CI** running the Foundry tests. And I'd write the server in **Java/Spring** with the same architecture.

---

## 13. One-page cheat sheet (read this last)

- **What:** Telegram bot to buy/sell 7 custom ERC-20 tokens with **Sepolia testnet** ETH. Wallet per user, live prices, on-chain mint/burn.
- **Stack:** TypeScript + `node-telegram-bot-api` (polling) · **ethers.js** + **Alchemy** (JSON-RPC) · **CoinGecko** REST (prices) · **MongoDB** (encrypted keys) · **Solidity + OpenZeppelin** · **Foundry** tests (10k fuzz).
- **Wallet:** `/start` → `Wallet.createRandom()` → **AES-encrypt** private key → store `{userId, encPrivKey, publicKey}`.
- **Buy:** check ETH/gas → price → `tokens=(1/price)*eth` → user sends ETH → owner calls master `GetToken` (mint). **2 txns = atomicity risk** (fix: send ETH *with* the payable mint).
- **Sell:** check balance → owner calls `BurnToken` (burn + return ETH).
- **Balances:** **`Promise.all`** → 8 reads in parallel (= max latency, not sum).
- **Contracts:** 7 ERC-20 (`onlyOwner` mint/burn) all **owned by a master `Tokens_To_Server` (Ownable)** → layered access control / trust boundary.
- **State:** `userBuyState` in-memory state machine → **should be Redis** (stateless, scalable).
- **Know-your-weaknesses (say first):** reused **IV** + symmetric **custody** (single point of compromise) · **non-atomic buy** · in-memory state · `getTokenPrice.ts` bug (always reads usd-coin) · polling not webhooks.
- **Fundamentals to name:** AES vs ECDSA (crypto) · polling vs webhooks, JSON-RPC vs REST (networking) · event loop + `Promise.all` (OS/concurrency) · atomicity/idempotency/SPOF (distributed systems) · interface + Ownable (OOP) · fuzzing (testing).
- **Java bridge:** TS/Solidity interfaces→Java interfaces · Ownable→@PreAuthorize · Promise.all→CompletableFuture.allOf · Mongoose→Spring Data Repo.
- **Closing line:** "It's money software across untrusted networks, so I focused on signing, encryption, and testing the contracts; if I productionized it I'd fix key custody, make the buy atomic, and make the bot stateless."

---

*Lead with this project. Raise the security trade-offs before they ask — for Omnissa, that's the strongest signal you can send.* 🚀
