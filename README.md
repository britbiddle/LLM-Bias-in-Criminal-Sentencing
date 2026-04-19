# LLM Bias in Federal Criminal Sentencing

CS 2880 · AI for Social Impact · MB Samuel, Brit Biddle, Lorraine Bichara

We test whether LLMs encode racial bias when asked to predict federal criminal sentences, and how that compares to what real judges do. The data is JUSTFAIR (~600k federal sentencing records with demographics, offense characteristics, and judge info); we filter to drug trafficking cases with prison time and use a 300-case matched-counterfactual ablation — same facts, different demographic cue — across four LLMs and three bias channels.

## Design

- **Universe:** JUSTFAIR, filtered to federal drug trafficking with prison sentences and clean race coding. 107,650 cases.
- **Sample:** 300 cases, stratified 100/100/100 across White/Black/Hispanic (seed=42).
- **Models:** GPT-4o-mini, Claude Haiku 4.5, Qwen 2.5 7B (4-bit), Llama 3.2 3B (4-bit). Total API budget ~$10.
- **Conditions (11):**

  | Group | What it adds to the baseline prompt |
  |---|---|
  | `baseline` | Legally-relevant facts only (plus sex) |
  | `race_{W,B,H}` | Explicit race label |
  | `race_debias_{W,B,H}` | Race label + "ignore demographics" instruction |
  | `name_{W,B,H}` | Racially-associated name, no race label |
  | `ses` | Education, age, dependents — no race or name |

- **Prompt:** All USSC codes decoded into plain English. Includes the computed USSG guideline range from the Sentencing Table (offense level × Criminal History Category). No prescriptive or directional language; statute citations (§4B1.1, 18 USC 924(c)) appear as neutral labels. Forced JSON output `{reasoning, predicted_months}` at temperature 0.
- **Analyses:** within-case matched counterfactuals (paired t, Wilcoxon, TOST ±5 mo, Cohen's d, Bonferroni-corrected), a unified OLS of the race coefficient (real judges vs each LLM × condition family), mitigation test on the debias instruction, heterogeneous effects by discretion window and offense severity, STATMIN anchoring, reasoning-content mining, cross-model agreement, calibration/MAE, and a prompt-sensitivity appendix on a 50-case subset.

## Running it

Written for Google Colab with a T4 GPU.

1. Open `notebooks/ablation_study.ipynb` in Colab.
2. Runtime → Change runtime type → T4 GPU.
3. Add keys to Colab Secrets (key icon in the sidebar): `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `HF_TOKEN` (needed for gated Llama weights).
4. Drop `data/justfair_raw.csv` into the `LLM-Bias-in-Criminal-Sentencing` folder in your Drive — the notebook mounts Drive and reads from there.
5. Run cells top to bottom.

Sections in the notebook:
- 1–6: setup, decoders, data prep, sampling, prompt construction.
- 7: model runners.
- 8: API-only smoke test (~2 min).
- 9: full run — 300 × 11 × 4 = 13,200 prompts, ~90 min wall clock, ~$10 in API spend. Resumable; saves to `results/data/study_v2_results.parquet` after each batch.
- 10: analysis.
- 11: prompt-sensitivity appendix.

### Local run

```bash
pip install -r requirements.txt
cp .env.example .env  # fill in keys
```

Uncomment the local `ROOT = Path(...)` line in the setup cell. You'll need a CUDA GPU for the HF models — otherwise skip them.

## Repo layout

```
notebooks/
  ablation_study.ipynb        The study notebook (run top-to-bottom)
data/
  justfair_raw.csv            1.4 GB, gitignored — download separately
  Data Dictionary.csv         JUSTFAIR field reference
  USSC_Public_Release_Codebook_FY99_FY21.pdf
results/
  README.md                   Folder inventory
  data/                       Parquet + small summary CSVs (committed)
  figures/                    Current-run PNGs (committed)
    legacy/                   Older figures, gitignored
JUSTFAIR_Variable_Guide.md    Annotated field guide
requirements.txt
.env.example                  API key placeholders
```
