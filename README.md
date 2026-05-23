# DDX · Subnet 32

**A Bittensor subnet that makes computer-use agents accurate enough to ship.**

We pay open-weight models for one thing only — turning a natural-language instruction and a screen into the right pixel. Validators score it. Emissions follow accuracy.

| | |
|---|---|
| **Subnet** | DDX · sn-32 |
| **Mechanism** | Defektr-class |
| **Benchmarks** | ScreenSpot · ScreenSpot-Pro · Mind2Web |
| **Team** | Drew Mimi · Keith Guo |
| **Status** | Hackathon submission · 48-hour build |

This repo holds the pitch deck and architecture diagrams for the DDX subnet submission. The full deck is in [`DDX_Subnet_PitchDeck.pdf`](./DDX_Subnet_PitchDeck.pdf); the live write-up lives at [daccred.notion.site/ddx-bittensor](https://daccred.notion.site/ddx-bittensor).

---

## Thesis

Computer-use agents don't fail at reasoning. They fail at finding the pixel.

- **66%** of agent failures trace to grounding errors. The model picks the wrong coordinate, the action misfires, the loop dies.
- **$15B** projected computer-use agent spend by 2028. Anthropic, OpenAI, and Google are all training on grounding directly — closed weights, frontier compute, monthly improvements.
- **0** open networks paying miners to win this benchmark. Open-source teams ship in isolation. DDX is the missing emissions layer.

## Mechanism — one epoch, end-to-end

Defektr-class loop. Proven, low-novelty, low surface area to game. Every stage is deterministic and reproducible on a held-out hash.

1. **Sample** — validator draws a held-out batch (256 prompts per epoch, rotated from a sealed 8,400-prompt pool covering ScreenSpot, ScreenSpot-Pro, and Mind2Web fragments).
2. **Query** — miners receive `prompt + screenshot (1920×1080) + seed`. They return one bounding box and a confidence score.
3. **Score** — IoU vs. ground truth, gated above 0.5, with an exponential latency penalty. Sealed ground truth lives only in the validator binary.
4. **Audit** — reproducibility seed match, latency-distribution outlier flags, held-out overlap detection, 14-of-16 validator quorum before commit.
5. **Emit** — softmax-weighted by mean batch score across the last 4 epochs. TAO emitted per epoch. Top miners earn proportional to held-out accuracy. Nothing else.

One epoch ≈ 360 blocks (~72 min). One full rotation ≈ 5 hours. Held-out exposure half-life ≈ 16 epochs.

### Per-prediction score

```
score(m, p) = IoU(pred_{m,p}, gt_p)
            × 𝟙[IoU(·) > τ]
            × exp(−λ · latency_{m,p})

τ = 0.5    λ = 0.0006 / ms    β = 8.0    floor = 0.5%
```

A boring score is a robust score. Three components — IoU, a hard gate, a latency penalty. Nothing learned, nothing tunable per-miner.

### Anti-gaming surface

- **Held-out leak** — ground truth lives only in the validator binary; held-out subset rotates every 4 epochs.
- **Sybil farming** — stake-weighted scoring, 0.5% emissions floor, 8-epoch delay for fresh UIDs.
- **Reproducibility** — every 32nd call is a seeded re-query; mismatches trigger a 50% weight penalty until cleared.
- **Latency gaming** — per-miner latency distribution monitored; >4σ deviation triggers audit and median-pinning.
- **Validator collusion** — 14/16 quorum required for score commit; random 5% re-evaluation cross-check.

## Benchmarks — three sets, one weighted score

| Weight | Benchmark | Source | Notes |
|---:|---|---|---|
| 0.40 | **ScreenSpot** | Cheng et al. 2024 · 1,272 instances | Mobile, desktop, web. Canonical entry. UI-TARS-7B baseline: 89.5%. |
| 0.35 | **ScreenSpot-Pro** | Li et al. 2025 · 1,581 high-res instances | Photoshop, AutoCAD, Blender, Premiere. 4K. Best published: 23.4% — where capability moats show. |
| 0.25 | **Mind2Web-fragments** | Deng et al. · web fragments | Single-step grounding only. Long-tail composite coverage. |

v1.1 adds VisualWebArena fragments (0.10) and a private DDX-generated set (0.10) once 50 validators are live. Weights are governed by stake vote.

## Live validator UI

Public read-only validator console at `vali-04.ddx.network`. Real-time leaderboard, score history, anti-gaming flags, batch rotation status. Every value is verifiable against on-chain commits.

## Emissions allocation

Standard 18 / 41 / 41 split. Miner share routes 100% to held-out accuracy. No off-mechanism payouts, no contribution bonuses, no foundation discretion.

- Miners — **41%**
- Validators — **41%**
- Subnet owner — **18%**

## Precedent

Defektr (sn-25) proved that benchmark-scored, IoU-gated subnets can sustain a healthy miner population for 9+ months (247 active miners, zero mechanism rewrites). DDX is the same shape, with grounding as the target. We tuned three constants — not the architecture.

## 12-week roadmap

| Phase | Window | Milestones |
|---|---|---|
| **Week 0 — Hackathon** | May 23 – May 26 | Validator binary v0.3 · live dashboard · baseline miner reproducing UI-TARS-7B · deck + stage demo |
| **Phase 1 — Testnet** | Weeks 1–4 | Add SS-Pro + M2W · 8 testnet validators · 25 testnet miners · adversarial review |
| **Phase 2 — Mainnet** | Weeks 5–8 | Register sn-32 · migrate held-out pool · stake-weighted quorum · public leaderboard |
| **Phase 3 — Ecosystem** | Weeks 9–12 | HTTP grounding API · first 3 paying users · v1.1 weight vote · 50-validator decentralization milestone |

## Team

- **Drew Mimi** · GitHub [@koolamusic](https://github.com/koolamusic) · X [@letandrewcook](https://x.com/letandrewcook) — mechanism · validator. Owns the validator binary, the scoring function, and the anti-gaming surface.
- **Keith Guo** · GitHub [@apkaisaw](https://github.com/apkaisaw) · X [@apkaisaw](https://x.com/apkaisaw) — models · miner stack. Owns the baseline miner, the training data pipeline, and the open-weights release cadence.

## Ask

From this hackathon, we need:

- Bittensor Foundation — sn-32 registration green-light
- Defektr ops — adversarial review of the scoring loop
- 4 launch validators with >500 TAO stake
- Intros to UI-TARS, OS-Atlas, ShowUI teams as launch miners
- $50k to fund 8 weeks of validator infra + adversarial audit

Contact: **drew@ddx.network**

## Repo layout

```
DDX_Subnet_PitchDeck.pdf              full deck
architecture/
  ddx_architecture_overview.mermaid   system diagram
  ddx_evaluation_workflow.mermaid     validator loop diagram
```
