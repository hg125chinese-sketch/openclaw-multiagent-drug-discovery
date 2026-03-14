# Multi-Agent AI for Computational Science: A Systematic Training and Collaboration Methodology

**One person + OpenClaw + 7-step training = 3 cooperating AI agents for end-to-end drug discovery**

## Abstract

We trained three specialized AI agents — ChemicalExpert (21 skills), ProteinEngineer (7 skills), and QuantumExpert (3 skills) — using a systematic 7-step methodology on the OpenClaw platform. The agents share a vault-based skill system and collaborate through typed JSON handoff protocols. Validated on an IPF/ALK5 drug discovery campaign over six DMTA cycles, the system achieved 100% DFT pass rate across two consecutive cycles using pocket-conditioned 3D diffusion (DiffSBDD) and a full multi-signal validation pipeline (PoseBusters geometry QC, GNINA CNN rescoring, multi-seed pose robustness, Boltz-2 affinity prediction, and conflict-aware panel selection). All training artifacts are open-source.

---

## 1. Introduction

Large language models know chemistry, protein engineering, and quantum mechanics — but knowing and doing are different. An LLM can explain QSAR, but can it train a random forest on ChEMBL data with scaffold splits and flag when its own model is unreliable? Can it diagnose why its molecular generator produces candidates that dock well but lack the pharmacophoric interactions required for activity?

Single-agent approaches hit a depth ceiling: one agent cannot maintain expert-level skill across medicinal chemistry, protein design, and quantum chemistry simultaneously. A multi-agent system with specialized roles and structured collaboration protocols can.

**What we built:**

- **ChemicalExpert (CE)**: 21 skills covering molecular generation (VAE + diffusion), QSAR, docking (Vina + GNINA), interaction analysis (ProLIF + PLIF recovery), safety screening, geometry QC (PoseBusters), binding affinity prediction (Boltz-2), multi-signal panel selection, and synthesis planning
- **ProteinEngineer (PE)**: 7 skills for protein interface redesign, stability assessment, and developability screening
- **QuantumExpert (QE)**: 3 skills for DFT calculations, excited-state methods, and cross-agent workflow orchestration

All three agents share the same 7-step training methodology, the same vault-based skill system, and can exchange data through typed JSON contracts.

---

## 2. Platform and Architecture

### 2.1 OpenClaw

OpenClaw is an open-source agent framework that provides the runtime for all three agents. Each agent runs in a Docker container with access to domain-specific conda environments and tools.

### 2.2 System Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Shared Vault                        │
│  (openclaw-truthbook)                               │
│  ┌─────────────┐ ┌─────────────┐ ┌──────────────┐  │
│  │ CE Skills   │ │ PE Skills   │ │ QE Skills    │  │
│  │ (21 QMD     │ │ (7 QMD      │ │ (3 QMD       │  │
│  │ collections)│ │ collections)│ │ collections) │  │
│  └─────────────┘ └─────────────┘ └──────────────┘  │
└────────────┬──────────────┬──────────────┬──────────┘
             │              │              │
     ┌───────▼───────┐ ┌───▼────┐ ┌───────▼───────┐
     │ ChemicalExpert│ │ Protein│ │ QuantumExpert │
     │   Workspace   │ │Engineer│ │   Workspace   │
     │               │ │Workspace│ │               │
     │ exports/ ─────┼─┼────────┼─▶ imports/      │
     │ imports/ ◀────┼─┼────────┼── exports/      │
     └───────────────┘ └────────┘ └───────────────┘
             │              │              │
     ┌───────▼───────┐ ┌───▼────┐ ┌───────▼───────┐
     │  conda: chem  │ │  prot  │ │  chem (PySCF) │
     │  RDKit, Vina  │ │ ESM-2  │ │  DFT, TD-DFT  │
     │  ProLIF, MACE │ │ESMFold │ │  CASSCF/NEVPT2│
     │  DiffSBDD     │ │        │ │               │
     │  GNINA, PB    │ │        │ │               │
     ├───────────────┤ └────────┘ └───────────────┘
     │ conda: boltz  │
     │  Boltz-2      │
     │  (isolated)   │
     └───────────────┘
```

### 2.3 Shared Vault Architecture

The vault is the central knowledge store that makes multi-agent collaboration possible. It contains:

- **QMD collections**: searchable, versioned skill documents that agents query at runtime via `qmd search` and `qmd get` with line-number evidence
- **Skills**: modular capability definitions (SKILL.md files, 200-700 lines each) with decision boundaries, failure modes, and runnable examples
- **Cross-references**: any agent can cite specific lines from any skill document, enabling cross-domain auditability

Each agent reads from the shared vault but writes only to its own workspace. This prevents cross-contamination while enabling knowledge sharing.

### 2.4 Agent Dispatch

Each agent has its own workspace, identity (IDENTITY.md), behavioral principles (SOUL.md), and operational playbook (PLAYBOOK.md). Agents are invoked via command-line dispatch (`ce "..."`, `qu "..."`) and execute tasks within their workspace.

### 2.5 Cross-Agent Communication

Agents communicate through structured JSON files placed in `exports/` and `imports/` directories. The handoff protocol is defined by explicit schemas — the sending agent packages data with all required fields, and the receiving agent validates the schema before processing. This decoupled design means agents can be developed and tested independently.

---

## 3. The 7-Step Training Methodology

Through iterative refinement across all three agents, we converged on a 7-step training process that proved transferable across domains.

### Step 1: Pre-Assessment

Before writing any skill, ask the agent diagnostic questions across four dimensions: current knowledge, methodology, tool capabilities, and learning preferences. Let it reveal what it already knows.

### Step 2: Deep Probing

Design pointed questions targeting suspected gaps. The goal is to expose the difference between "knows about" and "can do." For example, an agent might explain KL collapse in VAEs but fail to diagnose it from training logs.

### Step 3: Gap Analysis

Compare pre-assessment against probing results. Only fill gaps — don't reteach what the agent already knows. This keeps skills lean and focused.

### Step 4: Skill Design

Each skill is a structured document (200-700 lines) with:

- **Decision boundaries**: "if X, do A; if Y, do B" — matching real domain decisions
- **Hard gates**: non-negotiable thresholds that must pass before proceeding
- **Failure modes**: what failure looks like and what to do about it
- **Runnable examples**: concrete code and commands, not abstract descriptions
- **Philosophy sections**: one-sentence core principles that guide behavior

### Step 5: Guided Practice

Give the agent a concrete task requiring the new skill. Watch it work. Intervene only when necessary.

### Step 6: Behavioral Correction

When the agent makes mistakes, challenge it to find the error itself rather than giving the answer. Hand-fed corrections don't stick; self-diagnosed corrections become permanent behavioral changes.

### Step 7: Self-Review

Ask the agent to write its own skill review. Compare against your observations. Discrepancies reveal shallow understanding that needs reinforcement.

### Cross-Domain Transferability

This methodology was applied identically across chemistry (CE), protein engineering (PE), and quantum chemistry (QE). The same structure — pre-assess, probe, fill gaps, design skills with gates, practice, correct, review — produced effective agents in all three domains.

---

## 4. Agent Profiles

### 4.1 ChemicalExpert (CE)

**Mission.** CE is a pragmatic "lab-mate" agent for digital chemistry: molecular representations, property modeling, molecule generation, safety filtering, docking/interaction analysis, and reproducible experiment pipelines.

**Skill set (21 skills):**

| Category | Skills |
|----------|--------|
| Generation & evaluation | chem-molgen, chem-scaffold-conditioned-gen, chem-pocket-diffusion |
| Modeling & analytics | chem-qsar, chem-gnn, chem-kinase-sar |
| Safety & developability | chem-reactivity-safety, chem-admet |
| Docking & interpretation | chem-docking (+ multi-seed + GNINA), chem-docking-interactions (+ PLIF recovery), chem-protonation-tautomer |
| Affinity & selection | chem-affinity-prediction (Boltz-2), chem-panel-selection |
| Quality control | chem-structure-qc-lite (PoseBusters) |
| Synthesis planning | chem-retrosynthesis, chem-rxn-conditions |
| 3D & physics utilities | chem-3dgen, chem-mlff |
| Orchestration & tooling | chem-experiment, chem-literature, chem-llm |

**Design principles:**

1. Gating over guessing: mandatory KPIs (e.g., hinge H-bond) are hard gates, not soft scores
2. Separation of concerns: generation, safety, docking, interpretation, and QC are explicit pipeline stages
3. Failure must be informative: every failure produces a traceable artifact
4. Cross-agent contracts: handoffs use strict schemas with minimal assumptions
5. Multi-signal validation: no single metric is trusted alone; panel selection resolves disagreements

**Capability boundary.** CE is not authoritative for final synthetic feasibility without external validation, DFT-level electronic properties (delegated to QE), or claims of true binding affinity from docking scores.

### 4.2 ProteinEngineer (PE)

**Mission.** PE is an autonomous agent for protein engineering workflows, emphasizing interface redesign campaigns that must survive multi-tool scrutiny.

**Skill set (7 skills):**

| Skill | Core Function |
|-------|---------------|
| prot-seqdesign | Constrained sequence design (LigandMPNN + ESM-IF cross-validation) |
| prot-structure | Structure evaluation and fold-back validation (ESMFold) |
| prot-stability | ESM-2 log-likelihood ratio scoring with interface-mutation mode |
| prot-interface | Interface extraction, contact typing, hotspot proxy |
| prot-msa | Pseudo-conservation and consensus guidance |
| prot-developability | Sequence-based red-flag screening with WT-relative gating |
| prot-dbtl | Design-Build-Test-Learn cycle orchestration |

**Key training discoveries (through three progressive practice runs):**

- **1YCR (short peptide binder)**: Exposed metric pathologies — pLDDT scaling artifacts, clash counting without bonded exclusions, TM-score instability on short sequences. Led to peptide-aware fold-back rules with RMSD as the primary metric.
- **1BRS (standard protein interface)**: Revealed tool disagreement as a diagnostic signal — fold-back PASS + interface PASS but stability FAIL (catastrophic LLR). Led to interface-mutation mode with burial-aware strictness.
- **1Z92 (cytokine interface, full DBTL cycle)**: Uncovered system-level blockers — fold-back cannot be a hard gate when WT itself is unreliable, and over-tight constraints collapse sequence diversity. Led to WT sanity check and constraint-relaxation strategies.

**Capability boundary.** PE uses evolutionary and structural proxies, not physics-based energy calculations. It does not include FoldX/Rosetta ΔΔG tooling by default.

### 4.3 QuantumExpert (QE)

**Mission.** QE provides quantum-chemistry rigor inside the multi-agent workflow, turning candidate molecules into verifiable electronic-structure outputs with explicit QC gates.

**Skill set (3 skills):**

| Skill | Core Function |
|-------|---------------|
| qchem-dft | DFT calculations, SCF troubleshooting, geometry optimization, frequency analysis |
| qchem-excited-state | TD-DFT, multi-reference escalation (SA-CASSCF + NEVPT2), active space validation |
| qchem-workflow | Orchestration layer with batch screening, checkpointing, and failure classification |

**Key training discoveries:**

- Gate-first execution: if SCF doesn't converge, don't report energies; if geometry isn't converged, don't proceed to frequencies
- Method labeling as a contract: every number is tied to functional/basis/software version
- Batch robustness: per-molecule independence with checkpointing prevents one failure from stopping the batch
- Dispersion correction vigilance: QE initially ran B97-D without verifying dispersion was active in PySCF — a silent failure mode. The correction was detected during behavioral review, leading to a permanent "verify dispersion" gate and a re-run of affected calculations.

**Capability boundary.** QE works within PySCF's capabilities. Methods requiring unavailable corrections are flagged, not silently downgraded.

---

## 5. Cross-Agent Collaboration Protocol

### 5.1 Handoff Schema

CE → QE handoff uses a strict JSON contract:

```json
{
  "source_agent": "ChemicalExpert",
  "task": "qc_screening",
  "project": "IPF_ALK5_cycle6",
  "molecules": [
    {
      "mol_id": "mol_0064",
      "smiles": "...",
      "charge": 0,
      "spin": 0,
      "vina_score": -10.04,
      "hinge_hbond": true,
      "geometry_source": "rdkit_embed",
      "geometry_file": null
    }
  ],
  "requested_properties": ["E_total", "HOMO", "LUMO", "gap", "dipole"]
}
```

The `geometry_source` field supports two modes: `"rdkit_embed"` (default, QE generates 3D from SMILES) and `"diffsbdd_3d"` (CE provides pre-generated 3D coordinates via `geometry_file`). QE returns results with `qc_flag` per molecule (PASS / OPT_FAIL / SCF_FAIL), enabling CE to make gate decisions without interpreting raw QC data.

### 5.2 Gate Contracts

Each handoff includes pre-processing gates on the sending side:

- **CE pre-processing before QE handoff**: PoseBusters QC → safety screen → Vina docking → multi-seed robustness → GNINA rescoring → ProLIF + PLIF recovery → panel selection → RDKit 3D embedding + MMFF convergence → MACE geometry prescreen → JSON packaging
- **QE gates during DFT**: SCF convergence → geometry optimization convergence → frequency analysis (n_imag = 0) → property extraction

### 5.3 Failure Handling

The protocol is designed so failures at any stage produce actionable diagnostics rather than silent drops:

- OPT_FAIL molecules are flagged but retained in the audit trail
- Repeated OPT_FAIL on the same SMILES across cycles triggers a structural blacklist
- MACE prescreen rejections are logged with strain values for calibration
- PoseBusters failures are logged with specific failure reasons (bond_lengths, bond_angles, steric_clash, etc.)

### 5.4 Potential Cross-Agent Interfaces (Not Yet Tested)

The current validation covers CE↔QE collaboration. Two additional interfaces are architecturally ready but not yet exercised:

- **PE↔CE (protein-ligand co-design)**: CE proposes ligands and binding hypotheses; PE redesigns the protein interface to improve complementarity or selectivity.
- **PE↔QE (mechanism-driven enzyme engineering)**: QE provides energetic insights on catalytic steps; PE translates those into sequence changes.

---

## 6. End-to-End Case Study: IPF/ALK5

### 6.1 Problem Statement

Idiopathic pulmonary fibrosis (IPF) has limited treatment options. We tasked the multi-agent system with finding novel ALK5/TGFBR1 inhibitor candidates, starting from an open-ended prompt with no target guidance.

### 6.2 Cycle 1: Autonomous Pipeline + Self-Diagnosis

CE autonomously selected ALK5 as the target, chose safety-first weighting (chronic disease), and ran the full pipeline.

**Self-diagnosis (unprompted):** CE identified that 4/5 Top5 candidates lacked the hinge H-bond required for Type I kinase inhibition. Root cause: VAE KL collapse.

**Result:** Top5 hinge H-bond rate: 1/5 (20%)

### 6.3 Cycle 2: Fixing the Generator (6 Controlled Experiments)

CE systematically attempted to fix generation quality through 6 experiments, concluding that SELFIES GRU VAE decoders structurally ignore external conditioning signals. Adopted rejection sampling as pragmatic fallback.

**Result:** Top5 hinge H-bond rate: 4/5 (80%)

### 6.4 Cycle 3: Strategy E (Logit Bias Decoding)

Logit bias (+2.0 on fragment tokens) achieved coverage ratio 1.393 with 2x better efficiency than rejection sampling.

**Result:** Top5 hinge H-bond rate: 4/5 (80%)

### 6.5 Cycle 4: Full CE↔QE Collaboration

The complete multi-agent loop: generation → safety → docking → interaction analysis → QC prescreen → QE DFT → scoring → retrosynthesis. QE: 1 PASS, 1 OPT_FAIL (50% fail rate).

### 6.6 Cycle 5: Pocket-Conditioned Diffusion (DiffSBDD)

Replaced SELFIES VAE with DiffSBDD. Key discovery: DiffSBDD's 3D coordinates are for molecular design, not QC geometry (MACE strain 450-630 kcal/mol). DFT: 3/3 PASS (100%).

### 6.7 Cycle 6: Full 21-Skill Pipeline

First cycle using all new tools:

1. CE: Generated 91 molecules (DiffSBDD), 85 RDKit-valid (93.4%)
2. CE: PoseBusters QC — 56/91 PB-valid (61.5%), catching 35% more geometry issues than RDKit alone
3. CE: Safety screen — 43 survivors (23% reject, lowest ever)
4. CE: Vina docking 43 molecules, best Vina -10.04
5. CE: ProLIF + PLIF recovery on Top20, 7 with hinge H-bond
6. CE: Multi-seed docking (3 seeds) — only 1/7 survived pose convergence + hinge consistency
7. CE: GNINA CNN rescoring as orthogonal ranking signal
8. CE: Panel selection (conflict-aware, relaxed hinge gate ≥0.67) → 3 candidates
9. CE: Boltz-2 affinity (weak signal on ALK5, consistent with calibration)
10. QE: Batch DFT → **2/2 PASS (100%)**, best candidate mol_0064, score_final **10.432** (new record)

### 6.8 Cumulative Results

**Progress across cycles:**

| Metric | Cycle 1 | Cycle 2 | Cycle 3 | Cycle 4 | Cycle 5 | Cycle 6 |
|--------|---------|---------|---------|---------|---------|---------|
| Generation method | Unconditional VAE | Rejection sampling | Logit bias | Logit bias | DiffSBDD | DiffSBDD |
| PB-valid rate | — | — | — | — | 55.1% | **61.5%** |
| Safety reject rate | 29% | 41% | 54% | 53% | 28% | **23%** |
| Best Vina score | -9.591 | -9.158 | -8.927 | -9.997 | **-10.270** | -10.04 |
| DFT PASS rate | — | — | 50% | 50% | 100% | **100%** |
| Best score_final | — | — | — | — | 10.404 | **10.432** |

**CE↔QE collaboration statistics (DFT level):** 11 molecules submitted across 4 cycles. 8 PASS, 3 OPT_FAIL (27% overall fail rate). DiffSBDD era (Cycles 5-6): **5/5 = 100% PASS**.

**Final DFT-validated candidates:** 8 molecules with complete electronic structure data (E_total, HOMO, LUMO, gap, dipole), retrosynthetic routes, and reaction conditions.

---

## 7. Key Findings

### 7.1 SELFIES GRU VAE Conditioning Failure

Six controlled experiments demonstrated that SELFIES GRU VAE decoders structurally ignore external conditioning signals. Inference-time intervention (logit bias, rejection sampling) is the validated solution.

### 7.2 Pocket-Conditioned Diffusion Changes Everything

DiffSBDD produced better molecules across every metric and achieved 100% DFT pass rate across two consecutive cycles. However, its 3D coordinates are optimized for docking pose, not molecular stability (MACE strain 450-630 kcal/mol). Use DiffSBDD for molecular design, RDKit for geometry preparation.

### 7.3 Multi-Signal Validation Catches Silent Failures

Cycle 6 demonstrated the power of layered validation: PoseBusters caught 35% more geometry issues than RDKit alone; multi-seed docking reduced 7 hinge-positive candidates to just 1 robust survivor; GNINA and Boltz-2 provided orthogonal ranking signals that disagreed with Vina — disagreement that is informative, not noise.

### 7.4 Interaction Analysis is Mandatory

Pure docking scores mask critical binding mode failures. In Cycle 1, 80% of top candidates lacked the fundamental hinge H-bond despite acceptable Vina scores.

### 7.5 Boltz-2 Requires Per-Target Calibration

On ALK5, Boltz-2 binder probability shows weak but directional signal (active mean 0.27 vs decoy 0.10), while absolute IC50 predictions are off by 2 orders of magnitude. Useful as tiebreaker, not primary signal. This is consistent with independent 2026 evaluations reporting weak-to-moderate correlations.

### 7.6 Tool Conflicts Are Diagnostic Signals

Across all three agents, the most valuable training outcome was learning to diagnose tool disagreements rather than averaging them away. When Vina and Boltz-2 disagree, when MACE strain doesn't predict DFT feasibility, when fold-back fails on wildtype — these conflicts are informative.

### 7.7 Self-Diagnosis Outweighs Raw Capability

An agent that flags its own mediocre output as mediocre is more useful than one that presents flawed results confidently. CE's unprompted identification of KL collapse (Cycle 1), QE's detection of silent dispersion failures, PE's diagnosis of fold-back rejecting valid designs — in each case, self-diagnosis led to permanent behavioral improvement.

---

## 8. Limitations and Future Work

### 8.0 Time and Compute Summary

| Item | Approximate Time |
|------|-----------------|
| CE training (21 skills + 6 cycles) | ~4 weeks of iterative sessions |
| PE training (7 skills + 3 practice runs) | ~1 week |
| QE training (3 skills + validation) | ~3 days |
| One CE↔QE collaboration round (2-3 molecules) | ~14-28 hours DFT wall time |
| Full Cycle 6 (generation → all gates → DFT → scoring) | ~20 hours total |

All computation ran on a single machine (Docker on Linux/WSL2, RTX 4080 Laptop GPU). DFT was the dominant cost. DiffSBDD generation used GPU (~2 min for 100 molecules); docking and GNINA used CPU/GPU.

### 8.1 Current Limitations

- **MACE prescreen**: Current strain proxy does not correlate with DFT OPT_FAIL. Needs richer signals or more calibration data.
- **DiffSBDD 3D coordinates**: Not suitable for direct QC input (strain 450-630 kcal/mol). RDKit re-embed is still required.
- **Boltz-2 on ALK5**: Weak signal; per-target calibration is essential before trusting affinity predictions.
- **Rigid receptor**: Flexible pocket docking (PackDock) is documented but not yet installed due to environment incompatibility.
- **No experimental validation**: All results are computational. Wet-lab synthesis and assay data would be the true validation.

### 8.2 Future Directions

- **DiffSBDD substructure inpainting**: Use fragment-fixing to enforce hinge motifs directly in the diffusion process (documented but not yet tested)
- **Flexible pocket docking**: PackDock integration for finalist pose robustness (deferred, guidance written)
- **Retrosynthesis upgrade**: RetroChimera/DeepRetro as second-opinion route engines (deferred, guidance written)
- **PE integration**: Use PE to engineer the ALK5 protein itself (e.g., stability variants for assay development)
- **Higher-level QC**: TZVP refinement or DLPNO-CCSD(T) for final candidate ranking

---

## 9. How to Reproduce

To train a new agent using this methodology:

1. **Set up OpenClaw** with a Docker container and the relevant conda environment for your domain.
2. **Pre-assess** the base model by asking diagnostic questions across your domain's four dimensions.
3. **Design skills iteratively** — start with 2-3 core skills, validate on a practice task, then expand. Each skill should fit in one SKILL.md file (200-700 lines) with decision boundaries and hard gates.
4. **Use a real project as the training ground** — abstract exercises don't produce robust agents.
5. **For multi-agent collaboration**, define the JSON handoff schema before the first data exchange.

---

## 10. Repository Links

| Repository | Content | Skills |
|------------|---------|--------|
| [openclaw-chemicalexpert-training](https://github.com/hg125chinese-sketch/openclaw-chemicalexpert-training) | CE training, 21 skills, IPF case study (6 cycles) | 21 |
| [openclaw-proteinengineer-training](https://github.com/hg125chinese-sketch/openclaw-proteinengineer-training) | PE training, 7 skills, 3 protein engineering cases | 7 |
| [openclaw-quantumexpert-training](https://github.com/hg125chinese-sketch/openclaw-quantumexpert-training) | QE training, 3 skills, CE↔QE collaboration test | 3 |

---

## 11. Citation

```bibtex
@misc{multiagent-compchem-2026,
  title={Multi-Agent AI for Computational Science: A Systematic Training and Collaboration Methodology},
  author={Heng Gao},
  year={2026},
  url={https://github.com/hg125chinese-sketch}
}
```
