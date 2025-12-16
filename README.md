# SPLIT: Safety Prioritization for Long-COVID Drug Repurposing via a Causal Integrated Targeting Framework

[![DOI](https://img.shields.io/badge/DOI-10.xxxx%2Fxxxxx-blue)](https://doi.org/10.xxxx/xxxxx)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)

> **Integrating Causal Driver Genes with Clinical Knowledge Graphs for Safety-First Drug Repurposing in Long COVID**

A computational framework that combines causal gene discovery with clinical knowledge graph learning to identify drugs that should be **avoided** (deprioritized) for Long COVID clinical trials based on unfavorable predicted safety, toxicity burden, and efficacy profiles.

---

## 🎯 Overview

![SPLIT Framework Workflow](Figures/Workflow.png)

**SPLIT** addresses a critical gap in drug repurposing: rather than simply ranking candidates by predicted efficacy, we prioritize **safety-first deprioritization**—systematically identifying drugs with unacceptable risk-benefit profiles before they enter costly clinical trials.

### Key Features

- **Multi-method causal gene discovery**: Integrates Mendelian Randomization (MR), Control Theory (CT), and Differential Causal Effects (DCE) to identify 1,725 Long COVID driver genes
- **Knowledge graph-based prediction**: Uses PlaNet to generate cohort-anchored predictions of safety (S), adverse events (AE), and efficacy (E)
- **Counterfactual trial design**: Evaluates drugs within fixed cohort contexts by substituting interventions while preserving protocol structure
- **Multi-tier deprioritization**: Generates complementary avoid lists (composite, threshold-based, counterfactual-delta) with configurable stringency
- **Pharmacology enrichment**: Contextualizes predictions with wet-lab evidence from PubChem, ChEMBL, Open Targets, and DailyMed

---

## 📊 Key Results

### Causal Gene Landscape

| Evidence Source | Genes |
|-----------------|-------|
| Control Theory (CT) | 1,641 |
| Mendelian Randomization (MR) | 50 |
| Differential Causal Effects (DCE) | 45 |
| **Multi-method convergent** | **11** |
| **Total unique genes** | **1,725** |

### Safety-Coverage Trade-off

A critical finding: **maximizing causal gene coverage does not improve efficacy but substantially compromises safety**.

| Drug | Causal Genes | Safety (S) | Efficacy (E) |
|------|--------------|------------|--------------|
| Niagen (Original) | — | 0.549 | 0.519 |
| Leronlimab | 1 | 0.441 | 0.521 |
| Vortioxetine | 11 | 0.332 | 0.521 |
| Ritonavir | 70 | 0.215 | 0.527 |

### Deprioritization Outputs

From 1,625 screened drugs, we generated multi-tier avoid lists:

| List Type | Threshold | Drugs (n) | Description |
|-----------|-----------|-----------|-------------|
| Primary q05 | Worst 5% | 82 | Core deprioritized set |
| Primary q10 | Worst 10% | 163 | Default threshold |
| Primary q20 | Worst 20% | 325 | Extended set |
| Threshold-based | Any extreme | 855 | ≥1 extreme flag |
| Multi-extreme | ≥2 flags | 121 | Highest concern |
| Delta (vs Niagen) | Any worse | 457 | Worse than original |

**113 drugs** appear in all three avoid lists, representing the highest-confidence deprioritization targets.

---

## 🔬 Data Files

### Core Data

| File | Description | Rows |
|------|-------------|------|
| `1_Causal_Genes.csv` | Long COVID driver genes with evidence source labels | 1,725 |
| `2_Drug_Targets.csv` | Drug-to-causal-gene mappings from 5 databases | 19,172 |
| `3_Ground_Truth.csv` | PlaNet predictions for template + counterfactuals | 4 scenarios |

### Avoid Lists (NCT04809974)

| File | Description | Drugs |
|------|-------------|-------|
| `avoid_primary_composite_q10.csv` | Worst 10% by composite score | 163 |
| `avoid_secondary_guard_q20.csv` | Any extreme metric flag | 855 |
| `avoid_delta_q10.csv` | Worse than Niagen on any dimension | 457 |
| `avoid_sensitivity_primary_q05.csv` | Worst 5% (most stringent) | 82 |
| `avoid_sensitivity_primary_q20.csv` | Worst 20% (extended) | 325 |

### Enrichment Tables

| File | Description | Candidates |
|------|-------------|------------|
| `NCT04809974_enriched_pharmacology.xlsx` | Cognitive cohort enrichment | 42 |
| `NCT04880161_enriched_pharmacology.xlsx` | Respiratory cohort enrichment | 12 |

---

## 🚀 Quick Start

### Installation

```bash
git clone https://github.com/yourusername/SPLIT.git
cd SPLIT
pip install -r requirements.txt
```

### Basic Usage

```python
from src.deprioritization import AvoidListGenerator
from src.prediction import PlaNetInterface

# Load causal genes and drug mappings
causal_genes = pd.read_csv('data/causal_genes/1_Causal_Genes.csv')
drug_targets = pd.read_csv('data/drug_targets/2_Drug_Targets.csv')

# Initialize PlaNet interface
planet = PlaNetInterface(model_path='models/planet_weights.pt')

# Generate cohort-anchored predictions
predictions = planet.predict_cohort(
    trial_id='NCT04809974',
    drug_panel=drug_targets[drug_targets['n_targets'] >= 11]
)

# Generate avoid lists
generator = AvoidListGenerator(
    predictions=predictions,
    reference_drug='Niagen'
)

avoid_lists = generator.generate_all(
    quantiles=[0.05, 0.10, 0.20],
    include_delta=True,
    include_threshold=True
)

# Export results
avoid_lists.to_csv('results/avoid_lists.csv')
```

---

## 🏥 Clinical Cohorts

### Template Cohort: NCT04809974

| Characteristic | Value |
|----------------|-------|
| Drug | Niagen (Nicotinamide Riboside) |
| Target Domain | Cognitive (brain fog) |
| Sample Size | 72 |
| Age (mean ± SD) | 45.9 ± 13.4 |
| Female | 72.4% |
| Race (White) | 91% |

### Sensitivity Cohort: NCT04880161

| Characteristic | Value |
|----------------|-------|
| Drug | Ampion (Inhaled Biologic) |
| Target Domain | Respiratory |
| Sample Size | 32 |
| Age (mean ± SD) | 52.1 ± 13.0 |
| Female | 56.3% |
| Race (White/Black) | 75% / 25% |

---

## 🎯 Top Deprioritized Drugs

### NCT04809974 (Cognitive Cohort) - Top 10 to Avoid

| Rank | Drug | Composite | Safety | Efficacy | AE Score | Rationale |
|------|------|-----------|--------|----------|----------|-----------|
| 1 | Upadacitinib | 0.159 | 0.241 | 0.515 | 2.27 | JAK inhibitor; immunosuppressive |
| 2 | Secnidazole | 0.164 | 0.124 | 0.516 | 1.14 | Lowest safety score |
| 3 | Fluticasone propionate | 0.185 | 0.127 | 0.516 | 0.82 | Very low safety |
| 4 | Pivampicillin | 0.188 | 0.320 | 0.514 | 2.18 | Low efficacy; high AE |
| 5 | Empagliflozin | 0.191 | 0.183 | 0.517 | 1.66 | SGLT2 inhibitor; metabolic risks |
| 6 | Fidaxomicin | 0.191 | 0.200 | 0.517 | 2.03 | High AE burden |
| 7 | Ethionamide | 0.201 | 0.196 | 0.517 | 1.87 | Hepatotoxicity risk |
| 8 | Bithionol | 0.204 | 0.299 | 0.515 | 2.55 | Highest AE burden |
| 9 | Thiethylperazine | 0.205 | 0.219 | 0.517 | 1.97 | Extrapyramidal risk |
| 10 | Tavaborole | 0.210 | 0.168 | 0.519 | 2.25 | Low safety; high AE |

### Drug Classes Enriched in Avoid Lists

- **Antimicrobials** (15 drugs; 9.2%): nitroimidazoles, penicillins, antitubercular agents
- **CNS/Psychiatric** (11 drugs; 6.7%): phenothiazines, tetrabenazine derivatives
- **JAK inhibitors** (7 drugs; 4.3%): upadacitinib, filgotinib, tofacitinib
- **Antidiabetics** (6 drugs; 3.7%): SGLT2 inhibitors, DPP-4 inhibitors
- **Corticosteroids** (3 drugs): fluticasone formulations

---

## 📖 Methods Summary

### Stage A: Causal Gene Discovery

1. **Mendelian Randomization (MR)**: Identifies genes with genetically-anchored causal effects on Long COVID risk using GWAS + GTEx eQTL data
2. **Control Theory (CT)**: Identifies network-indispensable nodes whose removal disrupts system controllability
3. **Differential Causal Effects (DCE)**: Quantifies pathway-level gene-gene interaction differences between Long COVID cases and controls

### Stage B: Drug-Gene Mapping

- Maps 1,725 causal genes to pharmacological modulators using DrugBank, ChEMBL, TTD, DrugCentral, and GtoPdb
- Screening panel: 1,776 drugs with ≥11 causal gene targets

### Stage C: PlaNet Prediction

- Generates cohort-anchored predictions of Safety (S), Adverse Events (AE), and Efficacy (E)
- Uses counterfactual trial design to evaluate drug substitutions within fixed cohort contexts

### Stage D: Safety-First Deprioritization

- **Primary composite**: Ranks by weighted combination of S, E, and inverse AE
- **Threshold-based**: Flags drugs with extreme values on individual metrics
- **Counterfactual-delta**: Identifies drugs worse than the original trial intervention

---

## 📚 Citation

If you use SPLIT in your research, please cite:

```bibtex
@article{split2025,
  title={SPLIT: Integrating Causal Driver Genes with Clinical Knowledge Graphs 
         for Safety-First Drug Repurposing in Long COVID},
  author={[Authors]},
  journal={[Journal]},
  year={2025},
  doi={10.xxxx/xxxxx}
}
```

---

## 🤝 Contributing

We welcome contributions! Please see [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- [PlaNet](https://github.com/mims-harvard/PlaNet) for clinical knowledge graph learning
- [GTEx Consortium](https://gtexportal.org/) for eQTL data
- [ClinicalTrials.gov](https://clinicaltrials.gov/) for trial data
- [DrugBank](https://go.drugbank.com/), [ChEMBL](https://www.ebi.ac.uk/chembl/), [Open Targets](https://www.opentargets.org/) for drug-target data

---

## 📧 Contact

For questions or collaborations, please open an issue or contact:

- **Lead Author**: [Name] - [email@institution.edu]
- **Project Link**: [https://github.com/yourusername/SPLIT](https://github.com/yourusername/SPLIT)

---

<p align="center">
  <b>SPLIT: Because knowing what to avoid is as important as knowing what to pursue.</b>
</p>
