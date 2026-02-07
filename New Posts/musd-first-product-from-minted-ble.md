# mUSD: The Liquidity Bridge Between $6T in Institutional Assets and DeFi

---

Canton Network is home to **$6T+ in tokenized institutional assets**. DTCC settling Treasuries. Broadridge processing $8T/month in repo. Goldman and BlackRock tokenizing funds.

The most sophisticated financial infrastructure in crypto.

But it's a walled garden.

Low liquidity. Institutions hold, they don't trade. No composability — Canton assets can't plug into DeFi. Limited retail access. Permissioned by design.

Meanwhile, DeFi has $100B in TVL, infinite composability, and 24/7 markets — but zero connection to real institutional capital.

**Two worlds. Zero bridge.**

Until now.

---

## What mUSD Actually Is

mUSD is Minted's first core product — a stablecoin purpose-built to be the **liquidity layer between Canton's institutional infrastructure and Ethereum's DeFi ecosystem**.

Not just another stablecoin. A bridge.

Here's the architecture that makes it different from everything else in the market:

**One asset. Two chains. Full composability.**

mUSD mints on Canton *or* Ethereum. It exists as a DAML asset on Canton and an ERC-20 on Ethereum, bridging seamlessly between both through a 3-of-5 validator consensus mechanism. DeFi gets institutional-grade validation. Institutions get liquidity without custody risk.

The backing model is straightforward: **1:1 redeemable against Minted treasury reserves**. Not algorithmic. Not fractional. Direct mint/redeem against treasury — deposit USDC, get mUSD, no slippage, no DEX required.

But that's just the base layer. The real product suite is what sits on top.

---

## The Full Product Stack

Most people hear "stablecoin" and think it stops at the peg. mUSD is an entire financial primitive built across two chains with four interlocking products:

### sMUSD — The Yield Vault

Deposit mUSD, earn yield. Built on both Canton (DAML) and Ethereum (ERC-4626).

Auto-compounding yield sourced from **AI-aggregated treasury strategies** across 15+ of the largest DeFi protocols: Aave, Pendle, Morpho, Maple, Sky (formerly MakerDAO), Steakhouse Financial, Ethena, Compound, and others. The algorithms optimize across TVL, yield, pool maturity, and security profile.

But here's where Canton integration creates something DeFi alone can't: **sMUSD stakers who pair their deposit with Canton Coin staking unlock boosted yields** through Minted's validator rewards. Canton Coin is delegated to Minted's validator infrastructure, and earned rewards are distributed pro-rata back to boosted stakers. Base yield is guaranteed from DeFi treasury strategies. Boost yield is variable based on Canton Network activity and Minted's validator performance.

The CantonBoostPool DAML contract enforces this with precision — deposit caps are calculated against your sMUSD position (max Canton deposit = sMUSD value × 0.25, enforcing an 80/20 ratio), with cumulative per-user tracking, cooldown periods, and entry/exit fees routed to the protocol. Validator rewards split 60/40 between LPs and the protocol per epoch.

This isn't a generic yield farm. It's a dual-chain staking architecture that didn't exist before.

### Multichain Lending & Borrowing (CDP)

Collateralized borrow and lending positions for mUSD on Canton and Ethereum. Deposit ETH, wBTC, stETH, or approved Canton assets — borrow mUSD against them up to your collateral ratio (e.g., 70% LTV).

The leverage loop is atomic: **deposit → borrow → swap on DEX → add collateral** executes in one transaction. The Solidity contracts (BorrowModule, CollateralVault, LeverageVault, LiquidationEngine) handle this on Ethereum, while equivalent DAML contracts (CantonLending) handle it on Canton with sub-transaction privacy.

The interest rate model uses a utilization-based curve — rates increase as utilization rises, keeping the system solvent. The LiquidationEngine enforces health factors with Dutch auction mechanics.

### Cross-Chain Bridge (Canton ↔ Ethereum)

Bridge security is the #1 attack vector in cross-chain protocols. mUSD's bridge is purpose-built for institutional resilience.

**Canton → Ethereum:**
1. User calls `bridgeToCanton()` on Canton
2. Canton burns mUSD, creates attestation
3. Validator nodes detect attestation via Canton Asset API
4. Each validator verifies and signs via AWS KMS
5. Relay collects 3-of-5 signatures
6. Relay submits to BLEBridgeV9 on Ethereum
7. Ethereum mints mUSD ERC-20

**Ethereum → Canton** mirrors this with burn events detected by validator nodes.

The BLEBridgeV9 contract I read in the GitHub repo enforces: sorted-address deduplication on validator signatures, `address(this) + block.chainid` bound to every attestation for replay protection, sequential nonce ordering preventing out-of-order or duplicate attestations, 24-hour rolling net mint/burn rate limits, 110% collateralization enforced on-chain before any mint, and Chainlink-compatible NAV oracle with 24h staleness checks.

**No single key can authorize a mint** (3-of-5 required). **No front-running possible** (Canton transactions are private until finalized). **No partial execution** (DAML's atomic commit means all-or-nothing). **No key exposure** (HSM-backed signing via AWS KMS).

The relay service runs as a TypeScript process with independent validator nodes, each maintaining their own KMS signer. The validator-node-v2 implementation handles both directions with event polling, signature aggregation, and automatic retry logic.

---

## Why Canton — Not Just Another Chain Choice

This isn't chain marketing. It's a hard architectural requirement.

Canton is the **only blockchain purpose-built for regulated institutional finance**. It's not competing with Ethereum for retail DeFi — it's the settlement layer that Wall Street is standardizing on.

What Canton provides that public chains structurally cannot:

| Requirement | Public Chains | Canton |
|---|---|---|
| Privacy | All transactions visible | Sub-transaction privacy |
| Regulatory Compliance | Bolted on | Native DAML contracts |
| Finality | Probabilistic | Deterministic |
| Interoperability | Bridges = attack vectors | Atomic cross-network |
| Formal Verification | Rare | DAML is formally verified |

**Sub-transaction privacy** means only parties involved in a transaction see its details — competitors never see your flows. **Selective disclosure** lets you share transaction proofs with regulators without exposing counterparty data. **No global state exposure** — unlike public blockchains, your TVL and positions aren't visible to front-runners.

KYC/AML checks happen off-chain; only pass/fail status crosses to Ethereum. GDPR-compatible — personal data stays in Canton, only hashes on-chain.

The DAML contracts I read in the repo are the real proof. Every mUSD operation on Canton — minting, staking, lending, bridging — is written in DAML with formal verification, dual-signatory safety, and privacy controls baked into the template structure. The InstitutionalAssetV4 module handles compliant transfers with whitelist enforcement, regulatory emergency transfers with audit trails, and UTXO-style position management with precision-aware splits.

This isn't theoretical. Goldman Sachs (GS DAP), BNY Mellon, Broadridge ($1T daily repo settlement), S&P Global, and Cboe/Visa are already on Canton.

**mUSD is the DeFi on-ramp for all of it.**

---

## The Revenue Architecture

mUSD isn't a charity project. The revenue model scales with TVL:

| TVL | Yield Spread | App Rewards | Fees | Total Revenue |
|---|---|---|---|---|
| $100M | $6M | $8.1M (variable) | $800K | **$14.9M** |
| $250M | $15M | $20.25M (variable) | $2M | **$37.25M** |
| $500M | $30M | $40.5M (variable) | $4M | **$74.5M** |
| $1B | $60M | $81M (variable) | $8M | **$149M** |

Revenue streams: yield spread (treasury yield minus holder payout, ~6% of TVL), Canton App Rewards (top app incentives from the Canton ecosystem), attestation fees (Canton Coin burns per attestation, 0.05% of turnover), mint/redeem fees (0.1% each way via DirectMint), and DEX LP fees from protocol-owned liquidity.

The Canton Boost distribution sends 60% of validator rewards back to sMUSD stakers who co-stake Canton Coin. The revenue table reflects gross protocol revenue before those distributions.

---

## How mUSD Connects to Minted's BLE

Here's the bigger picture that most people will miss.

Minted's core infrastructure is the **Beneficiary-Locked Environment (BLE)** — a dual-state token architecture that bridges utility speculation with legally-backed issuer equity tied to real valuations. The BLE lets Web3 projects offer their token holders actual equity exposure through SPV-held shares, NAV-priced, with full SPBD oversight and Reg D/Reg S compliance.

mUSD is the **first product Minted built using its own infrastructure**. It demonstrates the full stack in production:

- **DAML smart contracts** on Canton for institutional privacy and formal verification
- **Solidity contracts** on Ethereum for DeFi composability
- **Cross-chain bridge** with 3-of-5 validator consensus
- **Yield aggregation** across 15+ DeFi protocols
- **Compliance layer** with blacklist, freeze, and transfer validation on both chains
- **Validator infrastructure** earning Canton network rewards

Every component of the mUSD stack — the stablecoin, the yield vault, the lending module, the bridge, the boost pool — was built using the same engineering team and infrastructure philosophy that powers the BLE for Minted's clients.

Minted is currently onboarding clients including Bless.network (tokenomics repair post-Binance airdrop), Breakout.app (crypto consumer credit, backed by Naval Ravikant), and StandX (Solana-based stablecoin infrastructure, $1B valuation). The 2026 pro forma projects $12.78M in cash revenue and $10M in portfolio equity across 15 clients at 91.4% net margin.

**mUSD proves the infrastructure works. The BLE scales it across the ecosystem.**

---

## The Code Is Live

The full mUSD codebase is open and currently being audited by Softstack (DAML security leader):

**GitHub:** [github.com/luthatdude/Minted-mUSD-Canton](https://github.com/luthatdude/Minted-mUSD-Canton)

20+ Solidity contracts. 15+ DAML modules. Relay infrastructure. Frontend. 30+ test suites including deep audit tests and institutional audit tests. Deployed on Sepolia testnet.

**Architecture diagrams:** [app.eraser.io/workspace/OBmUok1Yvr1iNLutS3Tj](https://app.eraser.io/workspace/OBmUok1Yvr1iNLutS3Tj)

This isn't a whitepaper with "coming soon" at the bottom. The contracts are written, tested, and being audited. The bridge validators are built. The yield strategies are implemented. The frontend exists.

---

## What This Means

$6T in institutional assets sit on Canton with no DeFi access. $100B in DeFi TVL has no connection to institutional capital.

mUSD is the bridge between them.

Not conceptually. In production code. Written in DAML and Solidity. Bridged by validator nodes signing with AWS KMS. Yielding across Aave, Pendle, Morpho, and a dozen other protocols. Boosted by Canton validator rewards. Compliant with KYC/AML, GDPR, MiCA, and Basel III requirements.

One asset. Two chains. Full composability.

The tokenized economy doesn't need another stablecoin. It needs the stablecoin that connects the two halves of institutional finance that have been building in parallel and never talking to each other.

That's mUSD. And it's Minted's first move.
