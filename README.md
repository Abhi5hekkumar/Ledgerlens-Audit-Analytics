# LedgerLens - Audit Analytics

**Journal entry risk detection using SQL, Python (Benford's Law, Isolation Forest) and a Power BI dashboard.**

LedgerLens mimics the journal entry testing an audit team performs on a general ledger. It screens a full year of entries for red flags, scores every entry for risk, and produces a ranked review queue plus an audit-style findings memo.

> **Data note:** real general ledgers are confidential, so this project uses a **synthetic** dataset of about 80,000 journal entries for FY 2025-26 (April to March). About 2.9% of entries carry deliberately planted anomalies with known labels, which lets every method be measured against ground truth. No real company or person is represented. Because the anomalies were built to resemble the tests, the accuracy figures below are optimistic compared with real data.

## Key results

| Result | Value |
|---|---|
| Journal entries tested (full population) | 80,333 |
| Planted anomalies | 2,331 (2.9%) |
| SQL rule-based tests: recall / precision | 85.4% / 93.8% |
| Preparers failing the Benford test on manual entries | 3 of 30 |
| Entries to review (High + Medium tier) | 3,781 (4.7% of all entries) |
| Planted anomalies covered by that review queue | 99.7% |
| High tier: entries / share truly anomalous | 749 / 95.5% |

The SQL rules cannot see fabricated amounts. The Benford preparer test showed where they came from, and the Isolation Forest then flagged all 333 of them.

![Review queue](reports/figures/05_risk_tiers.png)

## How it works

1. **Generate data** (`src/generate_data.py`): about 80,000 realistic journal entries (accounts, preparers, approvers, month-end volume spikes, fixed rent) plus seven types of planted anomaly.
2. **SQL red-flag tests** (`sql/`): six classic audit tests written as one SQLite view, using window functions (`ROW_NUMBER`, `RANK`, running totals) and CTEs.
3. **Benford's Law** (`src/benford_analysis.py`): first-digit test overall and per preparer, using Nigrini's MAD conformity bands and a Bonferroni-adjusted chi-square test.
4. **Isolation Forest** (`src/anomaly_model.py`): unsupervised model on amount, timing, approval and account-pair features, combined with the SQL flags into High / Medium / Low tiers.
5. **Dashboard data** (`src/build_dashboard_data.py`): a star schema ready for Power BI (see `dashboard/POWER_BI_GUIDE.md`).
6. **Findings memo** (`src/generate_memo.py`): an audit-style memo whose numbers are read straight from the results.

### The six SQL tests

| Test | What it catches |
|---|---|
| Weekend / holiday posting | Manual entries posted when the office is closed |
| Round-number amount | Multiples of Rs 50,000 from Rs 1 lakh up (rent excluded) |
| Just below approval threshold | Manual entries within 6% under the Rs 5 lakh limit |
| Preparer = approver | Segregation-of-duties breach |
| Duplicate entry | Same date, accounts, amount, narration and preparer |
| Unusual account pair | Debit/credit combinations making up under 0.2% of entries |

### Results by test

![SQL test performance](reports/figures/03_sql_test_performance.png)

### Benford's Law: population vs preparer

The population as a whole conforms (MAD 0.0026), but three preparers do not.

![Benford by preparer](reports/figures/02_benford_by_preparer.png)

## Quick start

```bash
git clone https://github.com/Abhi5hekkumar/Ledgerlens-Audit-Analytics.git
cd Ledgerlens-Audit-Analytics
pip install -r requirements.txt
python src/run_pipeline.py
```

The pipeline takes about 20 seconds and rebuilds every file in `data/`, `reports/` and `dashboard/powerbi_data/`. To run one step at a time, see the scripts in `src/`. Options for the generator: `python src/generate_data.py --rows 100000 --seed 7`.

For a guided tour with outputs, open `notebooks/LedgerLens_walkthrough.ipynb`.

## Project structure

```
Ledgerlens-Audit-Analytics/
├── data/                       # journal entries, SQL flags, risk scores
│   └── sql_outputs/            # results of each analysis query
├── sql/
│   ├── 01_red_flag_tests.sql   # the six tests as one view
│   └── 02_analysis_queries.sql # trends, preparer ranking, amount bands
├── src/
│   ├── generate_data.py
│   ├── load_to_sqlite.py
│   ├── run_sql_tests.py
│   ├── benford_analysis.py
│   ├── anomaly_model.py
│   ├── build_dashboard_data.py
│   ├── make_charts.py
│   ├── generate_memo.py
│   └── run_pipeline.py
├── notebooks/LedgerLens_walkthrough.ipynb
├── dashboard/
│   ├── powerbi_data/           # star-schema CSVs for Power BI
│   └── POWER_BI_GUIDE.md       # step-by-step build, with DAX measures
├── reports/
│   ├── audit_findings_memo.md  # the one-page style audit memo
│   └── figures/                # charts used in this README
├── requirements.txt
└── LICENSE
```

## Findings memo

`reports/audit_findings_memo.md` reads like the output of a real journal entry testing procedure: scope, procedures, results, the highest-risk entries, seven recommended audit responses and limitations.

## Limitations

- The data is synthetic and the anomalies mirror the tests, so precision and recall would be lower on a real ledger, where true problems are rarer and normal activity is noisier.
- The approval limit, holiday list and flag thresholds are illustrative and would come from the entity's policies.
- A flag is a reason to look, not a conclusion. Every flagged entry needs an auditor's judgement and supporting evidence.

## Possible extensions

- Add vendor master data to test for duplicate vendors and payments.
- Add prior-year data for trend and period-end comparison.
- Replace the fixed rules with a supervised model once real labelled outcomes exist.

## Tech stack

SQL (SQLite), Python (pandas, NumPy, SciPy, scikit-learn, matplotlib), Jupyter, Power BI

## License

MIT
