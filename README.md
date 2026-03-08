# Multi-Agent AI for Computational Science: A Systematic Training and Collaboration Methodology

**One person + OpenClaw + 7-step training = 3 cooperating AI agents for end-to-end drug discovery**

## Abstract

We trained three specialized AI agents — ChemicalExpert (18 skills), ProteinEngineer (7 skills), and QuantumExpert (3 skills) — using a systematic 7-step methodology on the OpenClaw platform. The agents share a vault-based skill system and collaborate through typed JSON handoff protocols. Validated on an IPF/ALK5 drug discovery campaign over five DMTA cycles, the system achieved 100% DFT pass rate in the final cycle using pocket-conditioned 3D diffusion (DiffSBDD). All training artifacts are open-source.

---

## 1. Introduction

Large language models know chemistry, protein engineering, and quantum mechanics — but knowing and doing are different. An LLM can explain QSAR, but can it train a random forest on ChEMBL data with scaffold splits and flag when its own model is unreliable? Can it diagnose why its molecular generator produces candidates that dock well but lack the pharmacophoric interactions required for activity?

Single-agent approaches hit a depth ceiling: one agent cannot maintain expert-level skill across medicinal chemistry, protein design, and quantum chemistry simultaneously. A multi-agent system with specialized roles and structured collaboration protocols can.

**What we built:**

- **ChemicalExpert (CE)**: 18 skills covering molecular generation (VAE + diffusion), QSAR, docking, interaction analysis, safety screening, and synthesis planning
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
│  │ (18 QMD     │ │ (7 QMD      │ │ (3 QMD       │  │
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
     └───────────────┘ └────────┘ └───────────────┘
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

**Skill set (18 skills):**

| Category | Skills |
|----------|--------|
| Generation & evaluation | chem-molgen, chem-scaffold-conditioned-gen, chem-pocket-diffusion |
| Modeling & analytics | chem-qsar, chem-gnn, chem-kinase-sar |
| Safety & developability | chem-reactivity-safety, chem-admet |
| Docking & interpretation | chem-docking, chem-docking-interactions, chem-protonation-tautomer |
| Synthesis planning | chem-retrosynthesis, chem-rxn-conditions |
| 3D & physics utilities | chem-3dgen, chem-mlff |
| Orchestration & tooling | chem-experiment, chem-literature, chem-llm |

**Design principles:**

1. Gating over guessing: mandatory KPIs (e.g., hinge H-bond) are hard gates, not soft scores
2. Separation of concerns: generation, safety, docking, interpretation, and QC are explicit pipeline stages
3. Failure must be informative: every failure produces a traceable artifact
4. Cross-agent contracts: handoffs use strict schemas with minimal assumptions

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

PE was validated on three targets of increasing difficulty, each exposing different failure modes:

- **1YCR (short peptide binder)**: Exposed metric pathologies — pLDDT scaling artifacts, clash counting without bonded exclusions, TM-score instability on short sequences. Led to peptide-aware fold-back rules with RMSD as the primary metric.
- **1BRS (standard protein interface)**: Revealed tool disagreement as a diagnostic signal — fold-back PASS + interface PASS but stability FAIL (catastrophic LLR). Led to interface-mutation mode with burial-aware strictness.
- **1Z92 (cytokine interface, full DBTL cycle)**: Uncovered system-level blockers — fold-back cannot be a hard gate when WT itself is unreliable, and over-tight constraints collapse sequence diversity. Led to WT sanity check and constraint-relaxation strategies.

Three cross-cutting lessons emerged:

- ESM-2 evolutionary bias conflicts with interface mutations: interface residues conserved for function get penalized, requiring relaxed LLR gates in interface-mutation mode
- ESMFold cannot form disulfide bonds, causing low pLDDT for disulfide-rich targets: requires WT sanity check before applying fold-back as a hard gate
- Developability gates must be WT-relative: only flag issues *new* to the design, not inherited from wildtype

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
- Dispersion correction vigilance: QE initially ran B97-D without verifying dispersion was active in PySCF — a silent failure mode. The correction was detected during behavioral review, leading to a permanent "verify dispersion" gate and a re-run of affected calculations. This became QE's most instructive self-diagnosis moment.

**Capability boundary.** QE works within PySCF's capabilities. Methods requiring unavailable corrections are flagged, not silently downgraded.

---

## 5. Cross-Agent Collaboration Protocol

### 5.1 Handoff Schema

CE → QE handoff uses a strict JSON contract:

```json
{
  "source_agent": "ChemicalExpert",
  "task": "qc_screening",
  "project": "IPF_ALK5_cycle5",
  "molecules": [
    {
      "mol_id": "hinge_1",
      "smiles": "...",
      "charge": 0,
      "spin": 0,
      "vina_score": -10.01,
      "hinge_hbond": true,
      "geometry_source": "rdkit_embed",
      "geometry_file": null
    }
  ],
  "requested_properties": ["E_total", "HOMO", "LUMO", "gap", "dipole"]
}
```

The `geometry_source` field (added in Cycle 5) supports two modes: `"rdkit_embed"` (default, QE generates 3D from SMILES) and `"diffsbdd_3d"` (CE provides pre-generated 3D coordinates via `geometry_file`). QE returns results with `qc_flag` per molecule (PASS / OPT_FAIL / SCF_FAIL), enabling CE to make gate decisions without interpreting raw QC data.

### 5.2 Gate Contracts

Each handoff includes pre-processing gates on the sending side:

- **CE pre-processing before QE handoff**: RDKit 3D embedding + MMFF convergence → pH 7.4 protomer/charge → MACE geometry prescreen → JSON packaging
- **QE gates during DFT**: SCF convergence → geometry optimization convergence → frequency analysis (n_imag = 0) → property extraction

### 5.3 Failure Handling

The protocol is designed so failures at any stage produce actionable diagnostics rather than silent drops:

- OPT_FAIL molecules are flagged but retained in the audit trail
- Repeated OPT_FAIL on the same SMILES across cycles triggers a structural blacklist
- MACE prescreen rejections are logged with strain values for calibration

### 5.4 Potential Cross-Agent Interfaces (Not Yet Tested)

The current validation covers CE↔QE collaboration. Two additional interfaces are architecturally ready but not yet exercised:

- **PE↔CE (protein-ligand co-design)**: CE proposes ligands and binding hypotheses; PE redesigns the protein interface to improve complementarity or selectivity. The handoff would carry structural constraints (contact residues, burial classification) from PE to CE's docking stage.
- **PE↔QE (mechanism-driven enzyme engineering)**: QE provides energetic insights on catalytic steps; PE translates those into sequence changes. This would require a new handoff schema carrying active-site geometry and energy constraints.

---

## 6. End-to-End Case Study: IPF/ALK5

### 6.1 Problem Statement

Idiopathic pulmonary fibrosis (IPF) has limited treatment options. We tasked the multi-agent system with finding novel ALK5/TGFBR1 inhibitor candidates, starting from an open-ended prompt with no target guidance.

### 6.2 Cycle 1: Autonomous Pipeline + Self-Diagnosis

CE autonomously selected ALK5 as the target, chose safety-first weighting (chronic disease), and ran the full pipeline: ChEMBL data → QSAR → VAE generation → ADMET gates → docking → interaction analysis.

**Self-diagnosis (unprompted):** CE identified that 4/5 Top5 candidates lacked the hinge H-bond required for Type I kinase inhibition. Root cause: VAE KL collapse producing molecules outside kinase chemical space.

**Result:** Top5 hinge H-bond rate: 1/5 (20%)

### 6.3 Cycle 2: Fixing the Generator (6 Controlled Experiments)

CE systematically attempted to fix generation quality:

| Experiment | Strategy | Coverage Ratio | Outcome |
|------------|----------|---------------|---------|
| 1 | Cyclical annealing (fix KL collapse) | 0.021 | Latent fixed, wrong chemical space |
| 2 | Training set enrichment | 0.038 | Already at target, no change |
| 3 | Fragment-conditioned VAE (prefix) | 1.379 | Trivial solution (string concatenation) |
| 4 | Prefix=False validation | 0.002 | Confirmed: conditioning was fake |
| 5 | True latent conditioning | 0.054 | Model ignores frag_embed |
| 6 | Auxiliary fragment classifier | 0.095 | Insufficient despite bug fix |
| **D** | **Rejection sampling** | **1.573** | **Adopted — passed all gates** |

**Result:** Top5 hinge H-bond rate: 4/5 (80%)

### 6.4 Cycle 3: Strategy E (Logit Bias Decoding)

Built on the Cycle 2 finding, CE tested inference-time logit bias (+2.0 on fragment tokens). This achieved coverage ratio 1.393 with 2x better efficiency than rejection sampling.

Cross-attention conditioning was also attempted and failed (coverage 0.049), further confirming the architectural limitation.

**Result:** Top5 hinge H-bond rate: 4/5 (80%)

### 6.5 Cycle 4: Full CE↔QE Collaboration

The complete multi-agent loop:

1. CE: Generated 1000 molecules (logit bias) → safety screen (466 survivors) → docked 50 → interaction analysis → Top5 (3/5 hinge H-bond)
2. CE: QC pre-processing (RDKit + protomer + MACE prescreen) → 2 molecules handed to QE
3. QE: Batch DFT (B97-D/def2-SVP) → 1 PASS, 1 OPT_FAIL (50% fail rate)
4. CE: Integrated QC results into multi-objective scoring → retrosynthesis for all PASS molecules

### 6.6 Cycle 5: Pocket-Conditioned Diffusion (DiffSBDD)

Replaced the SELFIES VAE with DiffSBDD, a pocket-conditioned 3D diffusion model running on GPU (RTX 4080):

1. CE: Generated 98 molecules from ALK5 pocket in ~2 minutes, 89 valid (90.8%)
2. CE: Safety screen — 28% reject rate (vs VAE's 53%)
3. CE: Docked 50 → Top5 (3/5 hinge H-bond), best Vina -10.270 (surpassing the co-crystal ligand)
4. CE: QC pre-processing — DiffSBDD's 3D coordinates showed extreme MACE strain (450-630 kcal/mol), confirming they are optimized for docking pose, not molecular stability. Solution: use DiffSBDD for molecular design (SMILES), RDKit re-embed for QC geometry.
5. QE: Batch DFT → **3/3 PASS (100%)**, best candidate Vina -10.01, score_final 10.404

### 6.7 Cumulative Results

**Progress across cycles:**

| Metric | Cycle 1 | Cycle 2 | Cycle 3 | Cycle 4 | Cycle 5 |
|--------|---------|---------|---------|---------|---------|
| Top5 hinge H-bond | 1/5 (20%) | 4/5 (80%) | 4/5 (80%) | 3/5 (60%) | 3/5 (60%) |
| Generation method | Unconditional VAE | Rejection sampling | Logit bias | Logit bias | DiffSBDD |
| Best Vina score | -9.591 | -9.158 | -8.927 | -9.997 | **-10.270** |
| DFT PASS rate | — | — | 50% | 50% | **100%** |
| Safety reject rate | 29% | 41% | 54% | 53% | **28%** |

**CE↔QE collaboration statistics (DFT level):** 9 molecules submitted across 3 cycles. 6 PASS, 3 OPT_FAIL (33% overall fail rate; 0% in Cycle 5 with DiffSBDD).

**Final DFT-validated candidates:** 6 molecules with complete electronic structure data (E_total, HOMO, LUMO, gap, dipole), retrosynthetic routes, and reaction conditions.

---

## 7. Key Findings

### 7.1 SELFIES GRU VAE Conditioning Failure

Six controlled experiments demonstrated that SELFIES GRU VAE decoders structurally ignore external conditioning signals. Concatenation, auxiliary loss, and cross-attention all failed — the autoregressive decoder learns to rely on its own hidden state history, bypassing any conditioning input. Inference-time intervention (logit bias, rejection sampling) is the validated solution for enforcing substructure constraints on this architecture.

### 7.2 Pocket-Conditioned Diffusion Changes Everything

DiffSBDD produced better molecules across every metric: lower safety reject rate (28% vs 53%), better Vina scores (-10.270 vs -9.997), and 100% DFT pass rate (vs 50%). However, its 3D coordinates are optimized for docking pose, not molecular stability (MACE strain 450-630 kcal/mol). The correct usage is: DiffSBDD for molecular design (what to make), RDKit for geometry preparation (how it looks in 3D for QC).

### 7.3 Interaction Analysis is Mandatory

Pure docking scores mask critical binding mode failures. In Cycle 1, 80% of top candidates lacked the fundamental hinge H-bond despite acceptable Vina scores. Adding interaction fingerprint validation (ProLIF) as a hard gate was the single most impactful quality improvement.

### 7.4 Tool Conflicts Are Diagnostic Signals

Across all three agents, the most valuable training outcome was learning to diagnose tool disagreements rather than averaging them away. When stability proxies disagree with structural evidence (PE), when MACE strain doesn't predict DFT feasibility (CE↔QE), when fold-back fails on wildtype (PE) — these conflicts are informative, not noise.

### 7.5 Gates Evolve from Static to Conditional

All three agents evolved their gate logic from simple static thresholds to context-dependent policies: peptide-specific vs protein-specific metrics (PE), interface-mutation mode (PE), WT-relative developability (PE), MACE prescreen as heuristic not classifier (CE).

### 7.6 Self-Diagnosis Outweighs Raw Capability

An agent that flags its own mediocre output as mediocre is more useful than one that presents flawed results confidently. Three examples across agents: CE's unprompted identification of KL collapse and missing hinge binders in Cycle 1; QE's detection that dispersion corrections were silently inactive in PySCF (leading to a permanent verification gate and re-run of affected calculations); PE's diagnosis that fold-back was rejecting valid designs because WT itself failed the gate. In each case, self-diagnosis led to permanent behavioral improvement.

---

## 8. Limitations and Future Work

### 8.0 Time and Compute Summary

| Item | Approximate Time |
|------|-----------------|
| CE training (18 skills + 5 cycles) | ~3 weeks of iterative sessions |
| PE training (7 skills + 3 practice runs) | ~1 week |
| QE training (3 skills + validation) | ~3 days |
| One CE↔QE collaboration round (3-4 molecules) | ~28-31 hours DFT wall time |
| Full Cycle 5 (DiffSBDD generation → safety → dock → QC → DFT → scoring) | ~30 hours total |

All computation ran on a single machine (Docker on Linux/WSL2, RTX 4080 Laptop GPU). DFT was the dominant cost. DiffSBDD generation used GPU (~2 min for 100 molecules); docking used CPU.

### 8.1 Current Limitations

- **MACE prescreen**: Current strain proxy does not correlate with DFT OPT_FAIL. Needs richer signals (multi-conformer screening, optimizer diagnostics) or more calibration data.
- **DiffSBDD 3D coordinates**: Not suitable for direct QC input (strain 450-630 kcal/mol). RDKit re-embed is still required, partially negating the 3D generation advantage.
- **Docking coverage**: Cycles 2-5 docked 20-50 molecules from pools of 60-400+. Larger-scale virtual screening would improve candidate diversity.
- **No experimental validation**: All results are computational. Wet-lab synthesis and assay data would be the true validation.

### 8.2 Future Directions

- **DiffSBDD substructure inpainting**: Use fragment-fixing to enforce hinge motifs directly in the diffusion process (documented but not yet tested)
- **PE integration**: Use PE to engineer the ALK5 protein itself (e.g., stability variants for assay development)
- **Higher-level QC**: TZVP refinement or DLPNO-CCSD(T) for final candidate ranking
- **Expanded collaboration**: Three-agent loop where CE generates ligands, PE optimizes the protein target, and QE validates binding energetics

---

## 9. How to Reproduce

To train a new agent using this methodology:

1. **Set up OpenClaw** with a Docker container and the relevant conda environment for your domain.
2. **Pre-assess** the base model by asking diagnostic questions across your domain's four dimensions (knowledge, methodology, tools, learning style).
3. **Design skills iteratively** — start with 2-3 core skills, validate on a practice task, then expand. Each skill should fit in one SKILL.md file (200-700 lines) with decision boundaries and hard gates.
4. **Use a real project as the training ground** — abstract exercises don't produce robust agents. Our agents became effective through IPF drug discovery (CE), protein interface redesign (PE), and DFT validation (QE).
5. **For multi-agent collaboration**, define the JSON handoff schema before the first data exchange. Include `mol_id`, all required fields, and a `flag` field for gate status.

Each of the three repositories below contains the complete skill set, training guide, and case studies needed to reproduce or extend the corresponding agent.

---

## 10. Repository Links

| Repository | Content | Skills |
|------------|---------|--------|
| [openclaw-chemicalexpert-training](https://github.com/hg125chinese-sketch/openclaw-chemicalexpert-training) | CE training, 18 skills, IPF case study (5 cycles) | 18 |
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
