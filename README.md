# Replacing a Native Disulfide with a De Novo Coiled Coil to Enable Cytoplasmic Bacterial Expression of a Compact Cytokine Mimic

**Jack Archer**¹

¹ University of Wisconsin–Madison

*A computational protein-design case study — manuscript-style write-up.*

---

> ### ⚠️ Intellectual-property notice
> To protect proprietary information, the target protein is referred to throughout only as the **protein of interest (POI)**. Its identity, native sequence, experimental structure, and the specific residue numbering of its disulfides and receptor epitope are deliberately **withheld**. This document discloses **design reasoning, methods, and tooling only** — it contains **no designed sequences, no coordinate files, no expression or assay data, no screening yields, and no library composition**. All quantitative claims are stated at the level of *method behaviour*, not campaign results. **All figures are placeholders** (`figures/figureN.png`) pending publication-quality replacements.

---

## Abstract

Many therapeutically important cytokines are compact helical proteins whose fold is stabilised by native disulfide bonds. This stabilisation strategy is incompatible with cheap, scalable production in the **reducing cytoplasm of *E. coli***, where disulfides do not form reliably and expression drifts toward misfolding and inclusion bodies. Here I describe the computational design of a compact mimic of one such protein (the **POI**) in which the disulfide-dependent structural cap is replaced by a **de novo antiparallel coiled coil**, transferring the stabilising role from a covalent bond to non-covalent knobs-into-holes packing while holding the mutagenesis-validated receptor epitope fixed. The mimic was produced with an open-source design stack (RFdiffusion3 → ProteinMPNN), evaluated by multi-predictor co-folding (Boltz-2, AlphaFold3, RF3), and filtered on coiled-coil geometry (SOCKET2 knobs-into-holes analysis and DSSP junction continuity). I report the design rationale, a principled scheme for removing the native cysteines, two non-obvious geometric findings that govern whether the graft welds cleanly, and a critical evaluation of when the standard confidence metrics are and are not trustworthy for this class of interface. The result is a reproducible design-build-test-in-silico pipeline that triages large design pools into a small set of candidates for wet-lab validation.

---

## 1. Introduction

The POI is a small, disulfide-stabilised helical cytokine that signals through a receptor complex, engaging it across more than one interface — all of which a functional mimic must reproduce. It is a validated pharmacological scaffold but an awkward manufacturing target for two reasons: it is natively **glycosylated**, and its tertiary fold is stapled by **native disulfide bonds**.

Disulfide bonds form in oxidising subcellular compartments. Production in the *E. coli* cytoplasm — the cheapest, fastest, most iterable expression host, and the one best suited to high-throughput design-build-test loops — is a **reducing** environment in which those bonds do not form dependably. The practical consequence is poor yield of correctly folded protein. This is the gap between an attractive structure and a supply chain that supports rapid iteration.

I reasoned that the most robust way to close this gap is not to coax disulfide formation in a hostile compartment, but to **remove the dependency on disulfides entirely** by substituting an alternative, non-covalent stabilising element. Coiled coils are the natural choice: they are the most thoroughly characterised structural motif in protein design, and their stability is *specifiable in advance* — heptad-repeat sequences fold into helix bundles whose oligomeric state, orientation, register, and interface can be designed and then **verified directly from coordinates**. This makes a coiled coil a dependable structural part to trade in for a fragile covalent one.

**Figure 1** summarises the design strategy.

<p align="center">
  <img src="figures/figure1_design_strategy.png" alt="Figure 1 — placeholder" width="720">
</p>

> **Figure 1 | Design strategy: replacing a native disulfide with a de novo coiled coil.** *(Placeholder — to be added.)* Left: the native POI cap, stabilised by a disulfide bond (highlighted), requires an oxidising compartment to fold. Right: the redesigned mimic, in which the disulfide-stabilised cap is replaced by a de novo antiparallel coiled coil that stabilises the fold through knobs-into-holes packing; the receptor-contact epitope (★) is held fixed. The redesigned fold has no covalent-bond requirement and is therefore compatible with the reducing *E. coli* cytoplasm.

---

## 2. Results

### 2.1 Removal of the native cysteines with role-preserving substitutions

Eliminating a disulfide is not equivalent to mutating both cysteines to serine. Each cysteine contributes to the fold beyond the covalent bond itself, and any cysteine left unpaired becomes a reactive free thiol liable to aberrant oxidation and cross-linking. Substitutions were therefore selected from sequence homology and the protein-folding literature rather than by default (**Table 1**).

**Table 1 | Cysteine substitution scheme.**

| Native cysteine | Substitution | Rationale |
|---|---|---|
| Loop cysteine | **→ Pro** | A convergent natural substitution — proline occurs at this position across independent mammalian lineages — that pre-organises the adjacent loop for a rate-limiting folding step, assuming the disulfide's structural role. |
| Loop-cysteine partner | **→ Ser** | Rendered unpaired once the loop cysteine becomes proline; substituted to serine as a conservative, non-reactive fill. |
| Graft-orphaned cysteine | **→ Ser** | Left without a partner when the coiled-coil graft replaces the opposing helix; neutralised to serine as a first pass, with a co-designed substitution as follow-up. |

Across all designs the **receptor-contact hotspot residues** — the positions validated by alanine-scanning mutagenesis as critical for binding — were held **byte-identical**. The engineering is performed *around* the epitope, never through it.

### 2.2 De novo coiled-coil design and structural validation

The graft is only useful if it is a *bona fide* coiled coil in the intended orientation. The coil family was built around three deliberate design choices, each verified from structure rather than assumed from design intent (**Figure 2**).

**Antiparallel topology with alternating interfacial charge.** An antiparallel two-helix bundle directs both helices back toward the POI body, so the graft *extends* the terminal helix rather than projecting away from it. Charges were alternated along the interfacial `e`/`g` heptad positions so that each strand is close to **net-neutral** — avoiding a highly charged, aggregation-prone surface — while the helix *pairing* remains electrostatically complementary. Each design was confirmed to meet this specification: near-zero per-strand net charge with favourable interfacial electrostatics.

**Knobs-into-holes validation on the isolated graft.** Design intent is not evidence, so knobs-into-holes (KIH) packing was confirmed with **SOCKET2** for every candidate and, critically, on each candidate's *independent co-fold* — requiring the coiled-coil register to survive structure prediction, not merely diffusion. An important methodological subtlety governs this step: the POI's own helical core also registers as KIH, so analysing the intact complex would score the native fold as "a coil" and spuriously flatter every design. The analysis is therefore performed on the **two graft helices excised into an isolated two-chain model**, so the KIH determination is attributable to the graft alone. In practice essentially every designed coil passed this test — "is it a real coil" ceased to discriminate — which correctly relocated the decisive filter downstream, to whether the coil could be **joined to the POI cleanly**.

**A standardised heptad register makes the coil a modular part.** Fixing the heptad register to a single convention converts the coil library into interchangeable cassettes: one adaptor design transfers across coils of differing length and topology because they all leave the POI on the same axis and phase. This is what makes a screening library of many coils × adaptor variants tractable to build **modularly** rather than one design at a time.

<p align="center">
  <img src="figures/figure2_coiledcoil_design.png" alt="Figure 2 — placeholder" width="720">
</p>

> **Figure 2 | De novo coiled-coil design and validation.** *(Placeholder — to be added.)* (a) Helical-wheel representation of the antiparallel two-helix graft: core packing at the `a`/`d` positions (knobs-into-holes), interfacial charge carried at the `e`/`g` positions. (b) Alternating interfacial charge yields near-neutral individual strands with a complementary, electrostatically favourable interface. (c) Attribution control: SOCKET2 KIH analysis is run on the two graft helices excised into an isolated two-chain model, so the coiled-coil call is not confounded by the POI's own helical core.

### 2.3 Graft junction geometry governs whether the weld is clean

Grafting is fundamentally a geometry problem. The coil must meet the POI helix through a short adaptor that ideally reads as a **single continuous α-helix**, so that the graft is rigid and the receptor epitope does not wobble. Two findings from the coordinates drove the adaptor design.

**Junction continuity was measured, not eyeballed.** A well-formed adaptor resolves as clean α-helix or clean loop; a strained adaptor exhibits 3₁₀ / π / β signatures at the junction. Continuity was scored per design with **DSSP**. An earlier crossing-angle metric was **deprecated** after it disagreed with DSSP on the majority of designs, because it accepted welds that were collinear but internally distorted — a reminder that a geometric proxy must be reconciled against the actual secondary structure.

**Motif-scaffolding silently re-docks the coil.** Coils positioned ~20 Å *away* from the POI in the input nonetheless returned as continuous chains, because the motif-scaffolding step applies a rigid-body move that pulls the coil inward to close a short linker. Adaptor **length therefore steers graft geometry**: to preserve a long, straight weld, the coil must be seeded farther out. This behaviour is non-obvious and visible only by comparing input and output coordinates.

A deliberately unresolved question underlies these choices: **must the weld be a rigid continuous helix, or can it be flexible?** The literature indicates this is target-dependent, so rather than assume, the design was structured as an explicit test — rigid-helix, helical-linker (`EAAAK`-type), and flexible (`GGGGS`-type) adaptors on identical cores — to be resolved by assay rather than preference.

### 2.4 A reproducible design–fold–filter pipeline for pool-scale triage

The purpose of the tooling is triage: converting a large diffusion pool into a short, defensible list of candidates for synthesis. The pipeline is implemented as reproducible **HTCondor DAGs** with per-stage checkpointing and fan-out (scaffold → sequences → folds), so that campaigns of tens of thousands of designs are a matter of bookkeeping rather than manual effort (**Figure 3**).

<p align="center">
  <img src="figures/figure3_pipeline.png" alt="Figure 3 — placeholder" width="860">
</p>

> **Figure 3 | Design–fold–filter pipeline.** *(Placeholder — to be added.)* (1) **Scaffold** the graft with RFdiffusion3, holding the epitope and core fixed and setting adaptor length via the contig. (2) **Design** sequence with ProteinMPNN, pinning the ★ contacts and omitting cysteine so no new thiols are introduced. (3) **Co-fold** the complex with Boltz-2 / AlphaFold3 / RF3, reading fold fidelity (Cα RMSD to the design) and interface confidence. (4) **Check geometry** with SOCKET2 (KIH on the excised coil) and DSSP (junction continuity). (5) **Rank and triage**, joining fold and geometry scores and re-verifying the epitope, to yield a short orderable set. Stages run as fan-out HTCondor DAGs.

---

## 3. Interpreting predictor scores

The most consequential component of this work is not a model but the discipline of not trusting one uncritically. Several confidence metrics are misleading in specific, learnable ways for this class of interface; each was caught by an explicit control (**Figure 4**).

- **The interface confidence score (ipTM) is uninformative for this system — even on the native complex.** Folding the wild-type complex as a control, a validated predictor scored the *native* assembly at the same low ipTM as the designs. The low value reflects the predictor's calibration on this interface type, not design failure. This control is what justified ranking designs by an alternative predictor rather than discarding the pool.
- **Predictors disagree on binding, and the disagreement is itself informative.** Where one predictor favoured the monomer fold but predictors split on receptor engagement, the designs were classified as **"geometry-correct, not validated binders"** — a deliberately weaker claim — rather than promoted. The gate is cross-predictor agreement, not any single ipTM.
- **A plausible metric can be quietly wrong.** The initial junction-quality metric accepted collinear-but-distorted welds and disagreed with DSSP on most designs, and was retired. A score that cannot be reconciled against structure should not be shipped.
- **In silico is not in vitro.** Continuity, KIH, ipTM, and RMSD narrow the pool; they do not establish expression, stability, or binding. The design is constructed to hand off to a wet-lab readout — surface display with a labelled receptor — that reports *folded and functional* in a single gate.

<p align="center">
  <img src="figures/figure4_score_interpretation.png" alt="Figure 4 — placeholder" width="720">
</p>

> **Figure 4 | Confidence metrics require target-specific controls.** *(Placeholder — to be added.)* (a) Interface confidence (ipTM) of the wild-type complex versus designs under a validated predictor: the native complex scores as low as the designs, showing the metric does not discriminate for this interface. (b) Cross-predictor disagreement on receptor engagement, used to separate "geometry-correct" designs from validated binders. (c) Deprecated crossing-angle metric versus DSSP junction calls, motivating the switch to a structure-reconciled continuity measure.

---

## 4. Discussion

Replacing a native disulfide with a de novo coiled coil reframes a manufacturing constraint as a design problem. Rather than engineering an oxidising environment or accepting low cytoplasmic yield, the fold is re-derived so that its stability no longer depends on chemistry the host cannot perform. The approach generalises beyond this POI: many compact, disulfide-stabilised helical proteins present the same expression barrier, and the same substitution — a well-characterised, register-standardised coiled coil in place of a covalent staple, with role-preserving cysteine cleanup — should transfer, provided the receptor epitope is held fixed and the graft junction is designed and *verified* rather than assumed.

The central open question — rigid versus flexible junction — is left to experiment by design. The same is true of the ultimate readout: every computational filter here is a hypothesis-narrowing step, and the pipeline's value is realised only when its shortlist meets a wet-lab assay. The engineering is therefore built for that hand-off: reproducible, auditable, and explicit about the difference between a design that is geometrically correct and one that is experimentally validated.

---

## 5. Methods

**Backbone generation.** Graft backbones were produced by motif-scaffolding with **RFdiffusion3**, holding the POI core and receptor epitope fixed and setting adaptor length through the diffusion contig.

**Sequence design.** Sequences were designed with **ProteinMPNN**, pinning the receptor-contact (★) positions and omitting cysteine to prevent introduction of new free thiols. **HyperMPNN** (thermostable-retrained weights) was evaluated as a parallel design arm; because *hyperstable* and *hypersoluble* objectives pull in opposite directions on surface composition, arm selection was left to downstream ranking rather than fixed a priori.

**Structure prediction / co-folding.** Complexes were co-folded with **Boltz-2**, **AlphaFold3**, and **RF3**, scoring fold fidelity as Cα RMSD to the design model and interface quality by predictor confidence (ipTM/PAE), with wild-type controls to calibrate metric behaviour.

**Coiled-coil geometry.** Knobs-into-holes packing was assessed with **SOCKET2** (with **DSSP**-derived secondary structure), run on the two graft helices excised into an isolated two-chain model to attribute the KIH call to the graft. Junction continuity was scored with DSSP.

**Compute.** All stages were orchestrated as fan-out, per-stage-checkpointed **HTCondor** DAGs, smoke-tested on a single pose before scale-out.

**Table 2 | Software.**

| Purpose | Tool |
|---|---|
| Backbone / motif scaffolding | RFdiffusion3 |
| Sequence design | ProteinMPNN; HyperMPNN (parallel arm) |
| Co-folding / structure prediction | Boltz-2, AlphaFold3, RF3 |
| Coiled-coil (KIH) analysis | SOCKET2 |
| Secondary-structure assignment | DSSP (`mkdssp`) |
| Orchestration | HTCondor DAGMan |

**Custom tooling developed for this project.** (i) A fast mmCIF reader (hand-rolled `_atom_site` parser with label/auth-numbering fallback) reducing per-structure parse time from ~10 s to seconds across a campaign; (ii) an **arm64 build of SOCKET2** with a batch wrapper incorporating the excise-to-two-chains step; (iii) DAG generators producing checkpointed, uniquely-named per-chunk outputs to avoid transfer-back collisions on the scheduler.

---

## 6. Data and code availability

Design reasoning, method descriptions, and tooling are documented in this repository. Consistent with the intellectual-property notice above, designed sequences, coordinate files, expression and assay data, screening yields, and library composition are **not** disclosed. The methods described here are built on publicly available software and on public structural and mutagenesis data for the POI.

---

## References

1. Dauparas J. *et al.* Robust deep-learning–based protein sequence design (ProteinMPNN). *Science* (2022).
2. Watson J.L. *et al.* De novo design of protein structure and function with RFdiffusion. *Nature* (2023).
3. Abramson J. *et al.* Accurate structure prediction of biomolecular interactions with AlphaFold3. *Nature* (2024).
4. Wohlwend J. *et al.* Boltz: democratizing biomolecular interaction modeling (Boltz-2). (2025).
5. Walshaw J. & Woolfson D.N. SOCKET: a program for identifying knobs-into-holes packing in coiled coils. *J. Mol. Biol.* (2001); SOCKET2 update, *Protein Sci.* (2021).
6. Kabsch W. & Sander C. Dictionary of protein secondary structure (DSSP). *Biopolymers* (1983).
7. Ertelt M. *et al.* HyperMPNN: generating thermostable proteins with ProteinMPNN. *bioRxiv* (2024).

---

*By **Jack Archer** · University of Wisconsin–Madison · 2026.*
*Built on public methods and public structural/mutagenesis data for the POI. No proprietary sequences, structures, yields, or library designs are included. Figures are placeholders pending publication-quality replacements.*
