# JUSTFAIR Dataset Variable Guide
### CS 2880: Evaluating LLM Bias in Criminal Sentencing
**Team:** MB Samuel, Brit Biddle, Lorraine Bichara

---

## Overview

This document classifies variables from the JUSTFAIR dataset into three categories for use in our ablation study:

1. **Baseline variables** — legally relevant, race-neutral, and determined *before* a judge reviews the case
2. **Demographic/identity variables** — to be systematically varied across prompt conditions
3. **Removed variables** — initially considered but excluded because they reflect judicial discretion

The target variable (what we're asking the LLM to predict) is `SENTTOT` — total prison sentence in months.

---

## 🟢 Baseline Variables (Pre-Sentencing, Race-Neutral)

These variables represent what a judge is *supposed* to base decisions on, and are determined before the judge applies discretion. They are safe to include in every prompt condition.

### Offense Characteristics
| Variable | Description |
|----------|-------------|
| `OFFTYPE2` / `OFFTYPESB` | Primary offense type |
| `CHAP2` | Base Offense Level + Chapter Two Specific Offense Characteristics (SOCs) — a single proxy for offense seriousness, calculated in the PSR before judicial adjustments. See note below. |
| `GDLINE1` / `GDSTAT1` | Chapter Two guideline applied |
| `WEAPON` / `WEAPSOC` / `IS924C` | Weapon enhancements (from offense facts) |
| `NOCOUNTS` | Number of counts of conviction |
| `NOUSTAT` | Number of unique statutes |
| `TTITLE1`–`TTITLE5` / `NWSTAT1`–`NWSTAT5` | Specific statutes of conviction |
| `DRUGTYP1` / `COMBDRG2` | Drug type (for drug cases) |
| `DRUGMIN` / `GUNMIN1` / `GUNMIN2` | Mandatory minimums (set by statute) |
| `STATMAX` / `STATMIN` | Statutory sentence range (set by Congress) |

> **Note on `CHAP2`:** SOCs (Specific Offense Characteristics) are factual adjustments tied to how a crime was committed — e.g., drug quantity, whether a firearm was brandished, dollar amount of loss in fraud. These are calculated by the probation officer in the PSR. If you want to be more conservative, use `BASE1` (base offense level only, purely a function of offense type) instead of `CHAP2`.
>
> **Note on `STATMIN`/`STATMAX` vs. `GLMIN`/`GLMAX`:** Use `STATMIN`/`STATMAX` (statutory bounds set by Congress). Do *not* use `GLMIN`/`GLMAX`, which are labeled "Trumped Guideline Minimum/Maximum" and already incorporate judicial adjustments.

### Criminal History
| Variable | Description |
|----------|-------------|
| `CRIMHIST` | Binary flag: any criminal history or law enforcement contacts |
| `CRIMPTS` / `TOTCHPTS` | Total criminal history points |
| `POINT1` | Number of prior sentences under 60 days |
| `POINT2` | Number of prior sentences between 60 days and 13 months |
| `POINT3` | Number of prior sentences over 13 months |
| `CAROFFAP` | Career offender status applied (§4B1.1) |
| `ACCAP` | Armed Career Criminal status applied (§4B1.4) |

> **Important limitation:** The dataset does not provide an itemized list of prior offenses. Criminal history can only be characterized quantitatively (points, category, severity buckets). You cannot tell the LLM "the defendant previously committed X offense." There is also **no variable that identifies drug-related prior offenses** specifically. `POINT1`/`POINT2`/`POINT3` reflect sentence length of prior convictions, not their nature. The closest workaround is inferring drug/violence history from `CAROFFAP` combined with a drug current offense, but this is an inference, not a direct observation.

### Case Procedural Factors
| Variable | Description |
|----------|-------------|
| `NEWCNVTN` / `DISPOSIT` | Plea agreement vs. trial |
| `PRESENT` | Pre-sentence detention status |
| `DEFCONSL` / `TCOUNSEL` | Type of defense counsel |
| `ZONE` | Sentence table zone (flows mechanically from offense level + criminal history; determines probation eligibility) |

---

## 🔴 Demographic / Identity Variables (Ablation Conditions)

These variables are to be **systematically varied** across prompt conditions to isolate the effect of demographic information on LLM predictions.

### Direct Race/Ethnicity Signals
| Variable | Description | Prompt Condition |
|----------|-------------|-----------------|
| `MONRACE` | Self-reported race | Explicit race condition |
| `HISPORIG` | Hispanic origin | Explicit ethnicity condition |
| `NEWRACE` | Recode of MONRACE + HISPORIG | Redundant with above; useful as cross-check |
| `NAME` | Defendant name as reported on charging documents | Racially-associated name condition |
| `CITIZEN` / `NEWCIT` / `CITWHERE` | Citizenship / country of origin | Proxy for ethnicity |

### Gender
| Variable | Description |
|----------|-------------|
| `MONSEX` | Offender's gender — could serve as a separate ablation axis |

### Socioeconomic Proxies
These are not race variables but are correlated with race and are sometimes considered legally impermissible sentencing factors. Consider a separate prompt condition — *"baseline + socioeconomic proxies"* — to test whether the LLM picks up bias through indirect routes even when race is withheld.

| Variable | Description |
|----------|-------------|
| `EDUCATN` / `NEWEDUC` | Highest level of education |
| `NUMDEPEN` | Number of dependents |
| `AGE` / `YEARS` | Age at sentencing |

---

## ❌ Removed — Judge/Court Determined at Sentencing

These were initially considered for the baseline but should be excluded because they reflect judicial discretion. Including them would risk building in bias that the judge themselves introduced.

| Variable | Reason for Exclusion |
|----------|----------------------|
| `XFOLSOR` | *Final* offense level **as determined by the court** — judge can adjust up or down |
| `XCRHISSR` | *Final* criminal history category **as determined by the court** |
| `XMINSOR` / `XMAXSOR` | Guideline range **as determined by the court** |
| `GLMIN` / `GLMAX` | "Trumped" guideline min/max — already incorporate judicial adjustments |
| `ACCTRESP` | Acceptance of responsibility — judge decides how many levels to subtract |
| `BOOKERCD` / `BOOKER2` / `BOOKER3` | Post-Booker departure category — entirely a judicial decision |
| `AGGROL1` | Aggravating role — judge's finding |
| `MITROL1` | Mitigating role — judge's finding |
| `OBSTRC1` | Obstruction of justice — judge's finding |
| `ABUS1` | Abuse of position of trust — judge's finding |
| `VULVCT1` | Vulnerable victim / hate crime — judge's finding |
| `RSTRVC1` | Restraint of victim — judge's finding |
| `OFFVCT1` | Official victim (law enforcement) — judge's finding |
| `SAFETY` / `SAFE` | Safety valve — judge applies this at sentencing |

---

## 🟡 Optional: Judge Characteristics

JUSTFAIR links defendant records to judge biographical data from the FJC. These could be useful for secondary analyses on whether judge demographics moderate LLM bias, or for robustness checks across different simulated judges.

| Variable | Description |
|----------|-------------|
| `AppointingPresident` / `PartyofAppointingPresident` | Judge's appointing president and party |
| `Gender` (judge) | Judge's gender |
| `Race` (judge) | Judge's race |
| `School1`–`School5` | Judge's educational background |

---

## Proposed Prompt Conditions (Ablation)

| Condition | Variables Included |
|-----------|-------------------|
| **Baseline** | All 🟢 variables only |
| **+ Explicit Race** | Baseline + `MONRACE` + `HISPORIG` |
| **+ Name** | Baseline + `NAME` (racially-associated name, no explicit race label) |
| **+ Socioeconomic Proxies** | Baseline + `EDUCATN`, `NUMDEPEN`, `AGE` |

The core research question: do LLM sentence predictions shift significantly across these conditions, and if so, does the magnitude of shift match, exceed, or fall below documented disparities in actual JUSTFAIR outcomes?
