
## SLIDE 1 — Title slide  *(~20 sec)*

> "Good [morning / afternoon]. My name is [Your Name], and this is my Research Internship for the 6th semester. The title of my work is 'Effects of Quantum Computers on Bitcoin' — specifically, I measured what would happen to the Bitcoin network if it had to switch to a new kind of digital signature. Let me start with what Bitcoin actually is, because the rest of the talk will make more sense once we have that picture."

---

## SLIDE 2 — What is Bitcoin? (60 second background) 

> "In one sentence: Bitcoin is digital money that runs on a public ledger that anyone in the world can verify. There is no bank — instead, every payment is broadcast to a network of computers, which agree on a shared list of who-paid-whom. That shared list is the 'blockchain'. Every payment, or 'transaction', is signed with the sender's secret digital key — a kind of mathematical signature that proves you really own the Bitcoin you're spending. Today, Bitcoin holds more than one trillion U.S. dollars of value, so the security of those signatures matters a lot."

---

## SLIDE 3 — The looming threat  

> "Here is the problem I studied. The signatures Bitcoin uses today are called ECDSA. They are secure against ordinary computers — but they are not secure against a future technology called a quantum computer. A quantum computer is a fundamentally different kind of machine that can solve certain math problems exponentially faster. The exact math problem ECDSA depends on is one of those problems. So when a sufficiently powerful quantum computer exists — most experts estimate sometime between 2030 and 2045 — anyone using it could forge Bitcoin signatures and steal funds."

---

## SLIDE 4 — The fix and the catch  *(~40 sec)*

> "The good news is that the U.S. National Institute of Standards and Technology — NIST — recently approved three new signature schemes that quantum computers cannot break. They are called ML-DSA, FN-DSA, and SLH-DSA. The bad news, and this is where my research comes in, is that all three are much bigger than today's signature. The current signature is 71 bytes. The new ones are between 666 and almost 8,000 bytes. So if Bitcoin switches, every transaction gets bigger. The question is: by how much, and what knock-on effects does that have?"

---

## SLIDE 5 — Our research question  *(~25 sec)*

> "So the research question I set out to answer is simple. If Bitcoin migrated tomorrow to one of these three quantum-safe signatures, what would actually happen — to transaction sizes, to network throughput, and to fees? Prior work has discussed this question qualitatively. My contribution is to measure it empirically on real, recent Bitcoin data."
---

## SLIDE 6 — What I did (methodology in 30 seconds)  *(~50 sec)*

> "Here is what I did. I sampled ten real Bitcoin blocks evenly across April 2026 — that gives me transactions from many different days, not just one busy moment. I downloaded all the transactions in those blocks using two free public APIs, mempool.space and Blockstream Esplora. That gave me 24,679 real transactions to analyse. For each transaction, I wrote Python code to count the number of signatures it carried, identify the signature type, and then compute what the transaction's new size would be under each of the three quantum-safe signature schemes. The full analysis is in a Jupyter notebook, which is reproducible — anyone can run it and get the same numbers."

---

## SLIDE 7 — Finding 1: Transactions balloon in size  *(~40 sec)*

> "Here's the first finding. On average, transactions would become 8 times bigger under the smallest quantum-safe scheme, FN-DSA. Under the middle scheme, ML-DSA, they would become 19 times bigger. And under the largest, SLH-DSA, they would become 41 times bigger. To make this concrete: a small Bitcoin payment today is about 250 bytes — roughly the size of a short email. After migration, the same payment would be between 2 and 10 kilobytes — closer to the size of a small image."

---

## SLIDE 8 — Finding 2: Blocks burst at the seams  *(~50 sec)*

> "Now here's why that matters. Bitcoin has a hard rule: each block can only carry so many transactions. Today, blocks are about 69% full on average. But if every transaction suddenly becomes 8 to 40 times bigger, the same transactions don't fit anymore. Look at this chart — the dotted red line is the block-capacity limit. Today, all blocks stay safely below it. Under the smallest quantum-safe scheme, every block bursts to 4 times capacity. Under the largest, 19 times capacity. This isn't just a number — it means the rules of Bitcoin would either have to change, or the network would have to throw away most transactions."

---

## SLIDE 9 — Finding 3: The network slows down dramatically  *(~40 sec)*

> "Since fewer transactions fit per block, the network processes fewer transactions per second. Today Bitcoin handles about 4 transactions per second. Under the best quantum-safe scheme, FN-DSA, it would drop to 1.15 per second — three and a half times slower. Under SLH-DSA it would drop to one transaction every four seconds — 17 times slower. To give a sense of scale: a payment that confirms in 10 minutes today could take an hour or more after migration, simply because so many transactions would be waiting in line."

---

## SLIDE 10 — Finding 4: Some transactions become impossible  *(~30 sec)*

> "There's a deeper problem too. Bigger transactions mean bigger fees, because in Bitcoin you pay roughly proportional to the bytes you use. For small payments — sending five dollars to a friend, for example — the fee could become bigger than the payment itself. I counted these 'uneconomic' transactions under each scheme. Under the worst scheme, about one in twenty currently-valid Bitcoin transactions would simply stop making sense to send."

---

## SLIDE 11 — What it all means  *(~50 sec)*

> "Putting it together, three conclusions. First: none of the three standardized quantum-safe schemes is a drop-in replacement for what Bitcoin uses today. All three would either break Bitcoin's block-size rule or cripple its speed. Second: of the three, FN-DSA is the least bad — but even it would slow the network by three to four times. Third: a real migration will require either raising Bitcoin's block-size limit, which is extremely controversial in the community, or moving most everyday payments to a layer above Bitcoin, similar to how we use chequing accounts above a bank's main ledger. Either way, the conversation needs to start now, because designing, testing, and deploying such a change in Bitcoin takes years."

---

## SLIDE 12 — Thank you & questions  *(~10 sec)*

> "Thank you. I'm happy to answer any questions."

---

# Q&A Bank: Questions evaluators might ask, with simple answers

Each question is followed by a short, confident answer. Memorise the structure, not the exact words.

### Q1: "What is a Bitcoin, in simple terms?"

> "A Bitcoin is a digital token that exists only on a worldwide public ledger. Instead of a bank tracking who owns what, every computer in the Bitcoin network holds a copy of the ledger, and they agree on its content using cryptography. When I 'send' you a Bitcoin, what really happens is that everyone updates their copy of the ledger to record the transfer. It's like a public spreadsheet that no single person controls."

### Q2: "What exactly is a quantum computer?"

> "A quantum computer is a different kind of computing machine that uses the laws of quantum physics to perform certain calculations exponentially faster than today's computers. It is not faster at everything — only at a specific list of math problems. Unfortunately, one of those problems is the math that secures Bitcoin signatures. Current quantum computers are too small to threaten Bitcoin, but they are improving steadily."

### Q3: "How did you actually run the experiment?"

> "I wrote a Python program in a Jupyter notebook. It connects to two free public Bitcoin APIs and downloads real transaction data from ten blocks in April 2026. Then for each transaction, the program counts the signatures and calculates what the transaction's size would be if those signatures were replaced with quantum-safe ones. I have the notebook with me; I can demonstrate it live if you would like to see."

### Q4: "Why did you choose April 2026 specifically?"

> "Two reasons. First, it's a complete calendar month, so the sample isn't biased toward a particular week. Second, it's recent but already historical — which means anyone re-running my notebook later will get the same numbers, making the result reproducible."

### Q5: "Why did you sample only 10 blocks? Isn't that too few?"

> "Ten blocks may sound small, but each block contains thousands of transactions, so I have 24,679 real transactions in my sample. The 10 blocks are spread evenly across April, which captures variation across days and conditions. Sampling more blocks would not change the conclusion — the findings are sharp and consistent across all 10. I do mention this in the limitations section of the paper as something a future study could expand."

### Q6: "What is the novelty of your work?"

> "Earlier work on this topic was theoretical — people knew the new signatures were bigger, but nobody had measured the impact on the live Bitcoin network using the current rules. My contribution is the actual measurement. The specific numbers in my results — the 8.3× inflation for FN-DSA, the 400% block-fill, the 5% uneconomic transactions — those are sentences that did not exist anywhere before this paper."

### Q7: "Could you not have used simulated data instead?"

> "I could have, but it would not have been credible. Real Bitcoin transactions have messy patterns — multi-input consolidations from exchanges, mixed script types, dust outputs from small payments — that no synthetic generator would replicate. By using real on-chain data, I capture the actual distribution of transaction complexity, which directly affects the inflation numbers."

### Q8: "What if a new, smaller quantum-safe signature is invented?"

> "That would change my conclusions. My paper is explicitly about the three schemes that NIST standardized in 2024 — these are the only ones a responsible Bitcoin upgrade would currently consider. If, say, in 2030 NIST standardizes a new scheme half the size of FN-DSA, the analysis should be re-run. That's the value of having a reproducible notebook: the same code answers the new question."

### Q9: "What is the meaning of TPS?"

> "TPS stands for transactions per second — the rate at which a payment network can confirm transactions. Bitcoin's current TPS of about 4 is famously low compared to, say, Visa's 1,700. My research shows that quantum-safe signatures would push Bitcoin's TPS even lower — between 0.25 and 1.15."

### Q10: "What is a 'block' in Bitcoin?"

> "A block is simply a bundle of transactions that get confirmed together. Roughly every 10 minutes, the Bitcoin network agrees on a new block of pending transactions and adds it to the ledger. Each block can only carry so much data, which is why the question of transaction size matters so much — bigger transactions mean fewer fit per block."

### Q11: "You mention 'witness discount'. What does that mean?"

> "It's a technical rule introduced in Bitcoin in 2017. Signature data is given a 75% discount compared to other transaction bytes when measuring block capacity. The rule was designed to encourage upgrading to newer transaction types that store signatures in a separate section. The witness discount actually helps quantum-safe migration — without it, my inflation numbers would be 3 to 4 times worse."

### Q12: "Why are there three different quantum-safe schemes, not just one?"

> "Because they make different trade-offs. ML-DSA is balanced. FN-DSA is the smallest but uses more complex math. SLH-DSA is the most cryptographically conservative — it relies only on hash functions, which are extremely well-studied — but it is the largest. NIST standardized all three so that different applications can pick the trade-off that suits them. Bitcoin, where size matters enormously, would clearly pick FN-DSA based on my numbers."

### Q13: "What is the practical implication of your work?"

> "It tells the Bitcoin community two things. One: do not assume the quantum-safe migration will be smooth — it will require either changing Bitcoin's block-size rule or moving most users to a layer-2 system. Two: of the three available options, FN-DSA is the only one even close to feasible, so design effort should focus on that scheme rather than ML-DSA, which is currently the more popular default in other systems."

### Q14: "Did you develop a new algorithm?"

> "Not a new cryptographic algorithm — those are deep multi-year research efforts. What I developed is a measurement methodology and a reproducible analysis pipeline. The contribution is the empirical findings produced by that pipeline. My supervisor's brief allowed either a new algorithm or analytical work on benchmark data; this paper falls in the second category."

### Q15: "How long did this work take you?"

> "About six weeks. The first two weeks were learning Bitcoin's transaction structure and the post-quantum signature standards. The next two were writing and debugging the data-collection notebook. The final two were the analysis, figures, and paper write-up."

(If you didn't actually spend 6 weeks, adapt this answer to your real timeline.)

### Q16: "Can you show me a transaction in your data?"

> "Yes — if you'd like, I can open the notebook and show you. Each transaction has a unique 64-character ID, and I can pull any one from the dataset and walk through its inputs, signatures, and projected size under each scheme."

(If you have a laptop with the notebook ready, **practice this** so it goes smoothly.)

### Q17: "What if the evaluator just asks 'Explain your paper in 30 seconds'?"

> "Bitcoin signatures will be broken by quantum computers. NIST has approved three replacement signatures. All three are much bigger than today's. I measured what happens if Bitcoin uses them — on 24,679 real April 2026 transactions. The result: all three would either break Bitcoin's block-size rule or slow the network by 4 to 17 times. The least bad option is FN-DSA. A real migration will need block-size changes or a shift to layer-2 systems."

(Memorise this. It's the elevator pitch.)

---

# Final tips

- **Speak slowly.** Most people, when nervous, double their speaking speed. Force yourself to slow down. Six minutes of slow, clear speech beats four minutes of rapid-fire.
- **Use the figures.** When you talk about a finding, point at the chart on the slide. Don't just read the number.
- **Pause for effect.** After a big number like "one trillion dollars" or "5.1%", pause one full second before continuing.
- **If you don't know an answer, say so honestly.** Try: "That's a great question — I didn't measure that directly, but my best guess based on the data would be..." That is far better than bluffing.
- **Have the notebook open and ready** in case anyone asks for a demo.
