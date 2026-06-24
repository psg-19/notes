# DeFi Telegram Bot — "Why X and not Y?" Tech-Choice Guide (Omnissa SDE Intern)

> **Purpose of this file:** Interviewers love *"Why did you use X instead of Y?"* and *"What's the advantage of X over the alternatives?"* They're testing whether you **understand trade-offs**, not trivia. This bot has unusually rich tech choices (blockchain, crypto, bots), so it's a goldmine for these questions — and a great place to show **security and systems thinking**, which is exactly what Omnissa hires for.

---

## 0. The golden framework (use this for ANY "why X not Y" question)

Never say "X is just better." Answer in **4 beats** — it sounds instantly senior:

1. **Requirement** — "What I needed was ___."
2. **Why X fits** — "X gives me ___, which matched that."
3. **Respect Y** — "Y is genuinely good at ___" *(shows you know the landscape)*.
4. **Deciding factor + when I'd switch** — "The deciding factor was ___; if I needed ___, I'd pick Y."

> **One sentence to remember:** *"There's no universally best technology — only the best fit for the requirement and constraints."*

> ⚠️ **The Omnissa-specific one:** *"Why not Java?"* (they're a Java/Spring shop). Answer: *"The blockchain tooling ecosystem — ethers.js, Foundry, the Telegram library — is strongest in the JS/TS world, so TypeScript let me move fastest while still getting static typing, which matters for money software. The architecture maps onto Spring directly, so I'd be productive writing the Java version. I chose the tool for the ecosystem, not preference."*

---

## 1. Master comparison table (one-glance revision)

| You used | Instead of | One-line reason you chose it |
| --- | --- | --- |
| **TypeScript** | JavaScript | Static types catch bugs at compile time — critical for money. |
| **Node.js** | Python, Go, Java | Best blockchain-tooling ecosystem; non-blocking I/O fits the workload. |
| **Telegram bot** | Web/mobile app, Discord | Zero front-end to build; instant cross-platform reach. |
| **Polling** | Webhooks | No public HTTPS endpoint needed — simplest to run/develop. |
| **ethers.js** | web3.js | Lighter, cleaner API, better TypeScript support, well-maintained. |
| **Alchemy (managed node)** | Run own node, Infura | No DevOps to run a full node; reliable RPC + free tier. |
| **CoinGecko REST** | Chainlink oracle, DEX price | Free, simple off-chain prices; no on-chain oracle cost/complexity. |
| **Solidity** | Vyper, Rust (Solana) | Standard EVM language, biggest ecosystem & docs. |
| **Foundry** | Hardhat, Truffle | Fast, tests written in Solidity, built-in fuzzing. |
| **OpenZeppelin** | Hand-written ERC-20 | Audited, battle-tested security primitives — never roll your own. |
| **MongoDB** | SQL, Redis | Simple key→value access (`userId → wallet`); flexible & fast. |
| **AES (symmetric)** | Asymmetric, hashing, plaintext | Fast two-way encryption to store a recoverable secret at rest. |
| **Custodial wallets** | Non-custodial | Smooth UX (no seed phrases) — at the cost of holding keys. |
| **ERC-20 standard** | Custom token interface | Interoperability — wallets/exchanges understand it automatically. |

---

## 2. The detailed Q&A (say these out loud)

### 🟦 Language & runtime

**Q1. Why TypeScript, and not plain JavaScript?**
> "This is **money software** — a wrong type or a typo in a transaction can mean lost funds. TypeScript adds **static typing**, so a whole class of bugs (passing the wrong shape, undefined values, mismatched units) is caught at **compile time** instead of at runtime when real value is moving. It also gives me autocomplete and self-documenting code, which matters when working with complex libraries like ethers. Plain JS is faster to start with and fine for small scripts, but for anything financial the compile-time safety is worth the extra setup."

**Q2. Why Node.js, and not Python, Go, or Java?**
> "Two reasons. First, the **blockchain tooling ecosystem is strongest in JS/TS** — ethers.js, the Telegram library, Foundry's JS integration — so I stayed in one ecosystem. Second, the workload is **I/O-bound** (waiting on the blockchain, the price API, the database), which Node's non-blocking event loop handles efficiently. Go would give better raw concurrency and Python has good web3 libs too, but Node had the best mix of ecosystem + fit. Java/Spring would be the enterprise choice and the architecture maps onto it cleanly — I chose Node for the tooling."

### 🟩 The interface (Telegram)

**Q3. Why a Telegram bot, and not a web or mobile app?**
> "I wanted to focus my effort on the **hard parts** — wallets, smart contracts, the money flow — not on building a front end. Telegram gave me a **ready-made, cross-platform UI** with buttons, forced replies, and chat, for free, on web/iOS/Android at once. It also has a huge user base and zero install friction. A web/mobile app would give full control over UX and is what I'd build for a real product, but it would've multiplied the work. Telegram was the fastest path to a working transactional system."

**Q4. Why polling, and not webhooks?**
> "**Polling** — the bot repeatedly asks Telegram 'any new messages?' — needs **no public HTTPS endpoint**, so it runs anywhere, even on my laptop, with zero infra. That made development trivial. **Webhooks** are push-based (Telegram POSTs updates to my URL) — they're **lower-latency and scale better** because there's no constant polling, but they require a public TLS endpoint and more setup. For development and a demo, polling was right; for production I'd switch to webhooks for efficiency and scale."

### 🟧 Blockchain interaction

**Q5. Why ethers.js, and not web3.js?**
> "Both talk to Ethereum, but ethers.js is **lighter, has a cleaner and more modern API**, and **first-class TypeScript support**, which fit my stack perfectly. It also separates concepts nicely — providers (read), signers (write), contracts — which made the code clearer. web3.js is the older, larger library and totally capable, but ethers' ergonomics and TS support made it the better fit, and it's very actively maintained."

**Q6. Why Alchemy, and not run your own Ethereum node, or use Infura?**
> "Running a **full Ethereum node** means syncing hundreds of GBs and maintaining infrastructure 24/7 — huge overhead for a project. Alchemy is a **managed node provider**: I get a reliable **JSON-RPC endpoint** over HTTPS with a free tier, monitoring, and no DevOps. **Infura** is the main alternative and basically equivalent — I'd happily use either; Alchemy just had the developer tooling I liked. The real decision was 'don't run your own node,' which is the right call unless you specifically need that control or decentralization."

**Q7. Why get prices from CoinGecko (off-chain REST), and not an on-chain oracle like Chainlink or a DEX?**
> "For my needs — showing a user the current price to compute an exchange rate — a **free REST API** like CoinGecko is simple and good enough. **Chainlink** is the production-grade answer: it's an **on-chain oracle**, so the price is **tamper-resistant and usable inside the smart contract itself** — essential if real money depended on it, because an off-chain price can be manipulated or go stale. A DEX like Uniswap can also serve as a price source on-chain. The honest trade-off: my off-chain price is a **security weakness** for a real system, and I'd move price logic on-chain with Chainlink if this handled real value."

### 🟥 Smart contracts

**Q8. Why Solidity, and not Vyper or Rust (Solana)?**
> "I targeted **Ethereum/EVM**, and Solidity is its primary language — by far the **biggest ecosystem, documentation, tooling, and audited libraries**. Vyper is also for the EVM and is intentionally simpler/more secure (Python-like), but has a much smaller community and fewer resources. **Rust** is for non-EVM chains like Solana — a different ecosystem entirely. Since I wanted EVM + OpenZeppelin + Foundry, Solidity was the natural choice."

**Q9. Why Foundry, and not Hardhat or Truffle?**
> "Foundry is **fast** and lets me **write tests in Solidity itself** — so my tests are in the same language as my contracts, no JS context-switch. Crucially, it has **built-in fuzz testing** (I run 10,000 random-input iterations), which is huge for financial code where edge cases cost money. **Hardhat** is excellent and more JS-integrated (great if your tests/scripts are in JS/TS), and **Truffle** is the older, now-fading option. I chose Foundry for speed and native fuzzing."

**Q10. Why use OpenZeppelin's ERC-20, and not write your own token contract?**
> "Because **security-critical code should never be hand-rolled**. OpenZeppelin's contracts are **audited, battle-tested, and used across the industry** — writing my own ERC-20 from scratch would risk subtle bugs (overflow, allowance edge cases) in code that can't be patched once deployed. I extended their `ERC20` and `Ownable` base contracts and only wrote the custom logic on top. Reusing audited primitives is the professional move — it's the same reason you don't write your own crypto library."

**Q11. Why the ERC-20 standard at all — why not a custom token interface?**
> "**Interoperability.** ERC-20 is the agreed-upon interface (`transfer`, `balanceOf`, `approve`, etc.), so any wallet, exchange, or tool understands my tokens automatically without custom integration. A custom interface would mean nothing else could talk to my tokens. Standards are what make the ecosystem composable — same idea as HTTP or SQL being standards."

### 🟪 Database & security (Omnissa's favorite area)

**Q12. Why MongoDB, and not a SQL database or Redis?**
> "My data access is dead simple: look up a user by their Telegram ID and get their wallet — basically **key→value**. MongoDB handles that easily with a flexible schema and was quick to set up. **SQL** would be overkill since I have no complex relations or joins. **Redis** would actually be a great fit for this access pattern and even faster (in-memory), but it's primarily a cache and I wanted **durable** storage for keys — losing a key store would be catastrophic. So MongoDB gave me durability + simplicity; for the *conversation state* (not the keys), Redis would be the right tool."

**Q13. Why AES (symmetric encryption) for private keys — why not hashing, or asymmetric encryption, or plaintext?**
> "I need to **get the original private key back** to sign transactions, so it must be **reversible** — that immediately rules out **hashing** (one-way, you can never recover the input). **Plaintext** is obviously unacceptable for a secret that controls funds. That leaves encryption. I used **AES (symmetric)** because I only need to encrypt and decrypt on my own server — symmetric is fast and simple for that. **Asymmetric** encryption is for when two *different* parties need to exchange secrets (encrypt with a public key, decrypt with a private one), which isn't my case. So: reversible rules out hashing, single-party rules in symmetric AES."

**Q14. What are the weaknesses of your encryption choice, and what would you use instead?**
> *(Raising this yourself is a huge credibility win at a security company.)* "Two real weaknesses. First, I **reuse the same initialization vector (IV)** for every user — with AES that's a known anti-pattern, because identical plaintexts produce identical ciphertext and it leaks information; the fix is a **random IV per record**, stored next to the ciphertext. Second, it's symmetric with **one master key**, so whoever holds that key can decrypt everyone's — a single point of compromise. The production answer is a **Key Management Service (KMS) or HSM** so the key never sits in an env file, and ideally a **non-custodial design** where I never hold user keys at all."

**Q15. Why custodial wallets (you hold the keys), and not non-custodial (users hold their own)?**
> "**Custodial** gave a frictionless UX — the user just messages the bot and a wallet 'just works,' no seed phrases, no browser extensions. That was the whole point of the chat interface. But custodial means **I carry the security burden and liability** — a server breach risks everyone's funds. **Non-custodial** is more secure and is where serious crypto products go (the user signs with their own wallet, I never see the key), at the cost of UX. For a learning demo on a testnet, custodial was the right trade-off; for anything real, I'd go non-custodial. This is genuinely the central trade-off in the whole project."

---

## 3. Trade-off one-liners (memorize 6–7 — these are your "wow" lines)

- **"There's no best technology, only the best fit for the requirement and constraints."**
- **TypeScript vs JS:** "It's money software — I want bugs caught at compile time, not when funds move."
- **Hashing vs encryption:** "I need the key *back*, so it must be reversible — hashing is one-way, encryption isn't."
- **Symmetric vs asymmetric:** "Same party encrypts and decrypts → symmetric AES; different parties → asymmetric."
- **OpenZeppelin vs DIY:** "Never hand-roll security-critical code that can't be patched once deployed."
- **Custodial vs non-custodial:** "I traded security for UX — fine on a testnet, but I'd go non-custodial for real funds."
- **Alchemy vs own node:** "Don't run infrastructure you can rent — managed RPC, zero DevOps."
- **CoinGecko vs Chainlink:** "Off-chain price is simple but manipulable; a real system needs an on-chain oracle."

---

## 4. How to handle "but isn't Y better?" pushback

Interviewers will **challenge** your choice to see if you fold. Don't crumble, don't be stubborn — use this:
> "Y is genuinely better at **[its strength]** — no argument there. For **my** constraints — [testnet demo / solo dev / I/O-bound / smoothest UX] — X was the right fit. If the requirement became **[Y's sweet spot — real funds / massive scale]**, switching to Y would be the correct move."

For this project specifically, the **strongest pushbacks** they'll try — be ready:
- *"Custodial keys are dangerous."* → "Completely agree; that's why it's testnet-only, and non-custodial is exactly what I'd build for real value."
- *"An off-chain price can be manipulated."* → "Right — for real money I'd use an on-chain oracle like Chainlink."
- *"You hand-rolled crypto."* → "No — I used Node's standard `crypto` AES and OpenZeppelin's audited contracts; what I'd fix is the IV reuse and key storage."

**Defend the fit, concede the context.** That balance of conviction + honesty is the strongest signal you can send — especially to a security company like Omnissa.

---

*They're not testing what you know about X — they're testing whether you can reason about trade-offs and security. Always name what the alternative is good at, and always volunteer the security trade-off.* 🚀
