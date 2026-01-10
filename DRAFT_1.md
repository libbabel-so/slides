---
marp: true
lang: en-US
title: Identification Schemes and Entity Authentication
description: Bala Dharnesh, Sai Pragnaan, Tegan Jain
theme: default
paginate: true
style: |
  section {
    background: #1a1a2e;
    color: #eee;
    font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
  }
  h1, h2, h3 {
    color: #00d4aa;
    font-family: inherit;
    text-shadow: 0 0 10px rgba(0, 212, 170, 0.5);
  }
  .terminal-accent {
    color: #ff6b6b;
  }
  .box {
    background: #2a2a40;
    color: #f8f8f2;
    padding: 1em 1.5em;
    border-radius: 10px;
    border-left: 4px solid #00d4aa;
    margin: 0.5em 0;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.3);
  }
  .mono-accent {
    color: #ffeb3b;
    font-weight: bold;
  }
  table {
    background: #2a2a40;
    color: #000000ff;
    border: 1px solid #00d4aa;
    border-radius: 5px;
  }
  th {
    background: #00d4aa;
    color: #1a1a2e;
  }
  .protocol {
    background: #16213e;
    padding: 1em;
    border: 2px solid #00d4aa;
    border-radius: 8px;
    font-family: monospace;
  }
  .attack {
    background: #3d1a1a;
    border: 2px solid #ff6b6b;
    padding: 1em;
    border-radius: 8px;
  }
  .secure {
    background: #1a3d1a;
    border: 2px solid #4caf50;
    padding: 1em;
    border-radius: 8px;
  }
  .proof-box {
    background: #1b1b2d;
    color: #aaaaff;
    padding: 1.2em;
    border-radius: 8px;
    border-left: 4px solid #4444ff;
    margin: 0.8em 0;
    font-size: 16px;
  }
  code {
    background: #2a2a40;
    color: #e26a07ff;
    padding: 2px 6px;
    border-radius: 3px;
  }
  .flow-diagram {
    background: #16213e;
    padding: 1em;
    border: 2px solid #00d4aa;
    border-radius: 8px;
    font-family: monospace;
    text-align: center;
  }
---

<!-- _class: lead terminal-accent -->
# WHO ARE YOU? 
## Identification Schemes & Entity Authentication

**Proving identity — in real life vs. on the wire**

---
## Presenters

- **Bala Dharnesh**
- **Sai Pragnaan**  
- **Tegan Jain**

*Ready to dive into the world of cryptographic identity?*

---

# The Three Authentication Factors

<div class="box">

**What you KNOW**
- Passwords, PINs, passphrases
- Example: Netflix login

**What you HAVE** 
- Cards, tokens, credentials
- Example: ATM card

**What you ARE**
- Biometrics: fingerprints, retina, face
- Example: Phone face-unlock

</div class="box">

**Multi-factor** = Combining factors for resilience

---

# Recognition ≠ Authentication

<div class="box">

| **Recognition** | **Authentication** |
|-----------------|-------------------|
| Visual/Physical ID | Real-time crypto proof |
| Offline verification | Interactive process |
| "I see you" | "Prove it's you" |

</div>

**Data Authentication**: Signatures verify data *after the fact*
**Entity Authentication**: Interactive proof in *real-time*

---

# Passwords: The Weak Link

<div class="box">

**Common Problems:**
- Weak human choices: `password`, `123456`, `abc123`
- Offline attacks after data breaches
- Rainbow table attacks

**Mitigations:**
- **Hash + Salt**: `h(password ∥ salt)`
- **Key Stretching**: Argon2, PBKDF2, bcrypt
- Iterate hashing 10,000+ times

</div>

*Tradeoff: Usability vs. Attack Resistance*

---

# Meet Oscar: The Adversary

<div class="attack">

**Deception Taxonomy:**
- **Passive eavesdropping**: Listen only
- **Replay attacks**: Reuse valid transcripts  
- **Parallel session attacks**: Replay between sessions
- **Man-in-the-middle**: Modify/forward messages
- **Social engineering**: Human deception

</div>

**Key insight**: Why randomness (freshness) matters!

---

# Protocol 10.1: Basic Challenge-Response

<div class="protocol">

**Protocol Steps:**
1. Bob → Alice: Random challenge `r`
2. Alice → Bob: Response `y = MAC_K(r)`
3. Bob verifies: `MAC_K(r) ?= y`

**Flow Diagram:**
```
Alice              Bob
      ← r ────────
      ─ y ────────→
                   Accept/Reject
```

</div>

<div class="attack">

**Problem**: Fixed responses allow replay attacks!
Oscar can reuse Alice's response in another session.

</div>

---

# Attack on Protocol 10.1: Parallel Session

<div class="attack">

**Oscar's Attack Strategy:**
1. Oscar impersonates Alice to Bob (Session 1)
2. Receives challenge `r` from Bob
3. **Initiates new session**: Oscar → Bob (Session 2)
4. **Reuses challenge**: Sends same `r` to Bob in Session 2
5. **Gets Bob's response**: `MAC_K(r)` 
6. **Replays to Session 1**: Sends Bob's response as Alice's

**Result**: Oscar successfully impersonates Alice! 🚨

</div>

---

# Attack Visualization: Protocol 10.1

<div class="flow-diagram">

```
Session 1:  Oscar ←─r─→ Bob
              ↓
            ┌─────┐
            │ r   │ (same challenge)
            └─────┘
              ↓
Session 2:  Oscar ←─r─→ Bob  
                  ←─y─┘
                    ↑
            ┌─────┐
            │ y   │ (Bob's MAC)
            └─────┘
              ↓
Session 1:  Oscar ─y─→ Bob ✓ ACCEPT
```

</div>

**Key Insight**: Oscar exploits Bob's own MAC computation!

---

# Protocol 10.2: Identity-Bound Response

<div class="secure">

**Enhanced Protocol Steps:**
1. Bob → Alice: Random challenge `r`
2. Alice → Bob: `y = MAC_K(ID(Alice) ∥ r)`
3. Bob verifies: `MAC_K(ID(Alice) ∥ r) ?= y`

**The Fix:**
- Include identity in MAC computation
- Prevents cross-identity replay
- Oscar gets `MAC_K(ID(Bob) ∥ r)`, needs `MAC_K(ID(Alice) ∥ r)`

</div>

**Why this works**: Different identities ⟹ Different MACs

---

# Security Analysis: Protocol 10.2

<div class="proof-box">

**Theorem 10.1**: If MAC is `(ε, Q)`-secure and challenges are `k` bits, then Protocol 10.2 is `(Q/2^k + ε, Q)`-secure.

**Proof Analysis - Oscar's Response Sources:**

1. **Bob created `y`**: ❌ Bob only creates `MAC_K(ID(Bob) ∥ r)`
2. **Alice reused `y`**: Probability ≤ `Q/2^k` (challenge collision)
3. **Oscar forged `y`**: Probability ≤ `ε` (MAC security)

**Total Deception Probability**: `Q/2^k + ε`

</div>

---

# Understanding the Security Bound

| **Component** | **Meaning** | **Mitigation** |
|---------------|-------------|----------------|
| `Q/2^k` | Challenge reuse probability | Increase `k` (challenge length) |
| `ε` | MAC forgery probability | Use secure MAC |
| `Q` | Max observed sessions | Limit protocol usage |

<div class="box">

**Example**: `k = 128` bits, `Q = 10^6` sessions, `ε = 2^{-80}`
- Challenge collision: `10^6 / 2^{128} ≈ 2^{-108}`  
- Total security: ≈ `2^{-80}` (dominated by MAC security)

</div>

---

# Protocol 10.3: Naïve Mutual Authentication

<div class="protocol">

**Protocol Steps:**
1. Bob → Alice: Challenge `r₁`
2. Alice → Bob: Challenge `r₂`, Response `y₁ = MAC_K(ID(Alice) ∥ r₁)`
3. Bob → Alice: Response `y₂ = MAC_K(ID(Bob) ∥ r₂)`
4. Both parties verify and accept/reject

**Goal**: Mutual authentication in 3 flows instead of 4

</div>

<div class="attack">

**Problem**: Still vulnerable to parallel session attacks!

</div>

---

# Attack on Protocol 10.3: The Setup

<div class="attack">

**Oscar's Strategy**: Fool Alice into accepting him as Bob

**Session 1** (Oscar → Alice as "Bob"):
1. Oscar → Alice: `r₁` 
2. Alice → Oscar: `r₂, MAC_K(ID(Alice) ∥ r₁)`
3. Oscar needs: `MAC_K(ID(Bob) ∥ r₂)` to fool Alice

**Session 2** (Oscar → Bob as "Alice"):  
1. Oscar → Bob: `r₂` (reusing Alice's challenge!)
2. Bob → Oscar: `r₃, MAC_K(ID(Bob) ∥ r₂)` ✅

**Result**: Oscar gets exactly what he needs!

</div>

---

# Attack Visualization: Protocol 10.3

<div class="flow-diagram">

```
Session 1:          Session 2:
Alice ←─r₁─ Oscar ─r₂─→ Bob
       ─r₂,y₁─→    ←─r₃,y₂─
       ←─y₂─┘              
       ACCEPT ✓             
```

**Flow Reuse**: 
- `y₂ = MAC_K(ID(Bob) ∥ r₂)` from Session 2
- Replayed to Alice in Session 1
- Alice accepts Oscar as Bob!

</div>

---

# Protocol 10.4: Secure Mutual Authentication

<div class="secure">

**Enhanced Protocol Steps:**
1. Bob → Alice: Challenge `r₁`
2. Alice → Bob: Challenge `r₂`, **`y₁ = MAC_K(ID(Alice) ∥ r₁ ∥ r₂)`**
3. Bob → Alice: Response `y₂ = MAC_K(ID(Bob) ∥ r₂)`
4. Both parties verify and accept/reject

**Key Change**: `y₁` now depends on BOTH challenges
- Prevents flow reuse between sessions
- Each response is contextually unique

</div>

---

# Why Protocol 10.4 Works

<div class="secure">

**Flow Binding Analysis:**

| **Tag** | **Depends On** | **Unique Properties** |
|---------|----------------|----------------------|
| `y₁` | `ID(Alice) ∥ r₁ ∥ r₂` | Both challenges, Alice's identity |
| `y₂` | `ID(Bob) ∥ r₂` | Single challenge, Bob's identity |

**Attack Prevention:**
- `y₁ ≠ y₂` (different computation methods)
- Can't reuse `y₁` as `y₂` or vice versa
- Each session has unique challenge combinations

</div>

---

# Security Proof: Protocol 10.4

<div class="proof-box">

**Theorem 10.2**: If MAC is `(ε, Q)`-secure and challenges are `k` bits, then Protocol 10.4 is `(Q/2^k + 2ε, Q/2)`-secure.

**Analysis:**
- **Sessions observed**: `Q/2` (2 MACs per session)
- **Challenge collision**: `Q/2^k` (same as before)
- **Forgery attempts**: `2ε` (Oscar can try forging `y₁` OR `y₂`)
- **Flow isolation**: `y₁` and `y₂` computed differently

**Security Bound**: `Q/2^k + 2ε`

</div>

---

# Real-World Systems & Attacks

| **System** | **Factors** | **Vulnerability** |
|------------|-------------|-------------------|
| OTP Tokens | Have + Know | SIM swapping |
| WebAuthn/FIDO2 | Have + Are | Phishing resistant |
| SMS 2FA | Have | SMS interception |
| Smart Cards | Have + Know | Physical theft |

**Remember**: Strong crypto + Poor UX = Broken system

---

# The Human Factor

<div class="attack">

**Social Engineering Attacks:**
- Phishing: Fake login pages
- Vishing: Voice-based deception  
- Pretexting: False scenarios
- UI deception: Lookalike domains

</div>

**Reality Check**: Crypto won't save you from people!
*User training and UX design matter as much as algorithms*

# Protocol Comparison Summary

| **Protocol** | **Security** | **Flows** | **Key Vulnerability** | **Fix** |
|--------------|-------------|-----------|---------------------|---------|
| 10.1 | ❌ Insecure | 2 | Parallel session | Add identity |
| 10.2 | ✅ Secure | 2 | None | N/A |
| 10.3 | ❌ Insecure | 3 | Flow reuse | Bind challenges |
| 10.4 | ✅ Secure | 3 | None | N/A |

**Key Principles:**
- Include identity in computations
- Bind flows together contextually  
- Make each response unique to its session

---

# Key Takeaways

<div class="box">

✅ **Interactive freshness + crypto binding** = Secure ID
✅ **Be explicit** about what each flow depends on  
✅ **Think beyond crypto**: Implementation, UX, humans matter
✅ **Mathematical proofs** provide security guarantees
✅ **Security through binding**: Identity + challenges prevent replay

</div>

**Remember**: Security is a chain - you're only as strong as your weakest link!

---

# Questions & Further Reading

<div class="box">

**References:**
- Stinson & Paterson, Chapter 10
- Protocols 10.1-10.4 detailed analysis
- MAC security definitions and proofs

</div>

**Questions?** 🤔

---

# Thank You!

<div class="terminal-accent">

```
> whoami
cryptography_students

> ls protocols/
10.1_insecure.txt  10.3_mutual_insecure.txt
10.2_secure.txt    10.4_mutual_secure.txt

> echo "Stay curious, stay secure!"
Stay curious, stay secure!

> exit
```

</div>

**Keep exploring the fascinating world of cryptographic protocols!**
