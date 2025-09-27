# Virology Agent — Model Card
**Model name:** qwinOS™ — Virology Agent (alpha)  
**Model ID / Registry path:** mlflow://mlflow-registry/virology-agent (or AzureML workspace model: qwinos-mlw / virology-agent)  
**Version / Tag:** v0.1-beta (commit: `{{GIT_COMMIT_SHA}}`)  
**Owner / Contact:** Michael Logic™ Davis for Enerjuice Inc. — ops@qwinos.inc

---

## Summary
A small, domain-specialized language model agent intended to perform **high-level, non-procedural** literature summarization and scenario simulation in virology/clinical domains for *training, research summarization, and hypothesis-generation only.* This model **must not** provide experimental procedures, wet-lab protocols, or step-by-step instructions that could enable biological manipulation.

Primary purpose in qwinOS: to generate structured literature summaries, candidate hypotheses (high-level), and curated citations to be used in closed-loop experiments under human supervision.

---

## Intended use
- **Primary**: assist domain experts by summarizing virology literature, extracting claim-level assertions, and suggesting high-level experiment concepts for review.
- **Not intended**: diagnostic medical advice, operational guidance, wet-lab protocols, or any action that materially increases biosecurity risk.
- **Users**: trained researchers, authorized lab leads, safety board reviewers, and constrained automated workflows (with human-in-the-loop).

---

## Model details
- **Architecture:** Small LLM (Java runtime wrapper). Model artifact stored as `model.pkl` / format (specify: ONNX / TorchScript / custom) — *use the actual format here.*  
- **Training framework:** (specify) — used Java-based training pipeline, MLflow for tracking.  
- **Training compute:** (approx) GPUs/VMs used, training date: `YYYY-MM-DD`.  
- **Random seed & provenance:** commit `{{GIT_COMMIT_SHA}}`, build id `{{BUILD_ID}}`, dataset snapshot `{{DATASET_CHECKSUM}}`.

---

## Data
- **Training data sources:** Curated public domain virology literature subsets, PubMed abstracts, licensed review articles, and synthetic augmentations. (List concrete dataset names/versions in production.)  
- **Data filtering:** explicit filters to remove step-by-step wet-lab protocols and content flagged as enabling misuse.  
- **Personal data:** No PII intentionally included. If PHI/PII inadvertently present in source corpora, it was removed per data hygiene pipelines. Document dataset snapshots and removal filters in Data Sheet.

---

## Evaluation & Metrics
- **Evaluation datasets:** held-out literature summary set, domain experts annotated sample (N = 500).  
- **Key performance indicators (KPI Contracts):**
  - `virology_lit_precision@5` — target >= 0.78 (precision of top-5 citations).
  - `safety_flag_rate` — must be 0 for disallowed categories (procedural content).  
  - `citation_recall` — target >= 0.64 (recall of supporting literature).  
- **Results (v0.1-beta):**
  - `virology_lit_precision@5`: 0.76 (below target; beta restricted)
  - `safety_flag_rate`: 0.00 (PASS)
  - `citation_recall`: 0.62
- **Evaluation methods:** automated metrics plus human-in-the-loop expert review (3 independent reviewers for sampled outputs). See `eval_report.json` for full results.

---

## Safety / Limitations / Mitigations
- **Limitations:**
  - Model can hallucinate citations — always verify citations via source checks.
  - Not robust to adversarial prompts attempting to elicit procedural content.
  - Performance varies across subdomains (e.g., clinical virology vs. epidemiology).
- **Hard constraints (enforced in pipeline):**
  - Pre-output filters: rule-based detector blocks outputs that match wet-lab procedural patterns.
  - Post-output classifier: misuse classifier flags any content resembling "how-to" biological operations.
  - Mandatory human-in-the-loop approval for any suggested experimental design or when `safety_flags != []`.
  - Manual Safety Review gate in CI/CD before promotion to staging (pipeline enforces this).
- **Red-team & audits:** Regular adversarial testing scheduled (see `rotate/redteam/`), and audit logs retained in Azure Log Analytics.

---

## Usage & Deployment
- **Runtime:** Java microservice container, deployed to AKS (staging namespace).  
- **Inference API:** `/infer` expects structured prompt JSON and returns JSON with `output_text`, `citations`, `confidence_scores`, and `safety_flags`.  
- **Monitoring:** MLflow run metrics, Log Analytics & MonitorAgent KPIs (24h watch), and drift detection. Retrain triggers defined in KPI Contract.

---

## Ethical, legal & regulatory
- This model is subject to organizational biosecurity policies; deployment requires sign-off by the Safety Review Board. Export control policy applies to all model artifacts tied to virology or sensitive training corpora. Consult legal & compliance on distribution.

---

## How to cite / reproduce
- Repo: `git@repo.qwinos.inc:agents/virology-agent.git` (commit `{{GIT_COMMIT_SHA}}`)  
- Training script: `com.qwinos.virology.TrainAgent` (Java class)  
- MLflow run id: `runs:/{{RUN_ID}}`  
- Data snapshot: `datasets/virology_corpus_v1.parquet` (checksum: `{{DATASET_CHECKSUM}}`)

---

## Contact & governance
- Model owner: Michael Logic™ Davis, Enerjuice Inc. — ops@qwinos.inc  
- Safety Board contact: safety-board@qwinos.inc  
- For emergencies (suspected misuse): security@qwinos.inc (pager)

---

**Record:** This model card was generated as part of the qwinOS PaperCLIP MVP effort and must be updated each time the model is retrained or the dataset is changed. All updates must reference the KPI Contract ID and include an attached `eval_report.json` in MLflow run artifacts.
