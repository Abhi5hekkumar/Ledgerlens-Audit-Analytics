# LedgerLens

**Finds suspicious entries in a company's accounting records using SQL and Python.**

Auditors check every journal entry (one line in the accounting records) to make sure nothing is wrong or dishonest. LedgerLens does this automatically and gives a short list of entries to check first.



## How it works

1. **Six SQL rules** look for warning signs in each entry.
2. **Benford's Law** checks each person's amounts. In real data, amounts starting with 1 are common and those starting with 9 are rare. Made-up numbers often break this pattern.
3. **A risk score** adds points for each warning sign. More points means check sooner.

| Rule | Catches |
|---|---|
| Weekend or holiday | Entry made when the office was closed |
| Round amount | Amounts like Rs 2,00,000 |
| Just below the limit | Amount just under the Rs 5 lakh approval limit |
| Same person approved | The person who made the entry also approved it |
| Duplicate | The same entry booked twice |
| Unusual accounts | A very rare combination of accounts |


## Results

| | |
|---|---|
| Entries checked | 20,080 |
| Shortlist to review (High + Medium) | **619** (3 in 100) |
| High risk | 259 entries, 94 in 100 really had a problem |
| Medium risk | 360 entries, 88 in 100 really had a problem |
| Unusual number pattern | 1 of 20 people (U07): 29% of amounts start with 7, 8 or 9, against 16% expected |


## Run it

Install [Python 3](https://www.python.org/downloads/), open a terminal in the project folder, and run (on Mac use `python3`):

```bash
pip install -r requirements.txt
python src/run_pipeline.py
```


## Keep in mind

- The problems were hidden to match my tests, so these results are cleaner than real data would give.
- The limits and point values are my assumptions.
- A warning sign is not proof of fraud. It means a person should take a closer look.

**Tools:** SQL (SQLite), Python (pandas, NumPy, matplotlib), Jupyter | **License:** MIT
