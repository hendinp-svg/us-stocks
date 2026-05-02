# Equity Monitor · Live Universe

A bias-minimized US stock screener that runs entirely in the browser. Single HTML file, deployable to GitHub Pages in under 2 minutes.

## What changed in this version

The previous build had a hardcoded watchlist of 20 stocks. This one has none. Every page load:

1. Fetches the **current S&P 500 constituent list** from a public open-data source on GitHub
2. Picks a sample using **stratified random sampling** across all 11 GICS sectors
3. Scores each sampled stock on five dimensions and adds it to a persistent pool
4. Surfaces the top 3 picks from the entire scored pool

Nothing is preselected. The dashboard does not know which stocks you should care about until it scores them.

## How bias minimization works

There are three places where bias can creep into a stock screener: in the universe, in the candidate selection, and in the scoring.

**Universe.** The S&P 500 constituents file is fetched live from `datasets/s-and-p-500-companies` on GitHub. If a name was added or removed by the index committee, the dashboard reflects that on next reload. The list is cached locally for 24 hours to be polite to the source. Click "Fresh Scan" to force a refetch.

**Candidate selection.** Each scan picks N stocks (default 25) by round-robin across GICS sectors, with random ordering inside each sector. Sectors are also shuffled. The result is that no single sector dominates a sample, and no stock is favored over another within its sector. Stocks already scored within the cache window (default 24h) are skipped, so each scan extends coverage rather than re-scoring the same names.

**Scoring.** The five-factor composite is deterministic and transparent. Same inputs always produce the same output. Weights are visible in the methodology section.

The honest limitation: the S&P 500 itself is a curation. It excludes small caps, micro caps, and most international names by definition. If you want broader coverage, you would need a paid API tier with bulk endpoints. Within the S&P 500, this dashboard adds no further selection bias.

## Setup

1. Sign up for a free Finnhub account at https://finnhub.io/register
2. Copy your API token from the dashboard
3. Open `index.html`, paste the token in the welcome screen
4. Click "Start Scan"

Your API key is stored in browser localStorage only. Nothing is sent anywhere except Finnhub's API.

## Deploying to GitHub Pages

```bash
git init
git add index.html README.md
git commit -m "Initial commit"
git remote add origin git@github.com:USERNAME/REPO.git
git push -u origin main
```

In the repo settings, enable Pages on the main branch. The dashboard will be live at `https://USERNAME.github.io/REPO/`.

## How long does a scan take

Finnhub free tier allows 60 requests per minute. The dashboard makes 4 API calls per ticker (quote, profile, fundamentals, recommendations). Default sample size of 25 needs roughly 100 calls.

| Sample size | Approx time | Calls |
|---|---|---|
| 10 | 45 seconds | 40 |
| 25 (default) | 100 seconds | 100 |
| 50 | 3.5 minutes | 200 |

Coverage compounds across scans. With the default sample size, full S&P 500 coverage takes about 20 scans. You can run them whenever you want; results stay cached for 24 hours.

## Composite score

The composite is a weighted average of five sub-scores, each on a 0-100 scale.

```
Momentum  25%   52-week range position + 13/26/52w returns (penalty if >90% of high)
Quality   30%   Net margin, ROE, ROA, current ratio
Value     15%   PE TTM, PB, PS
Growth    20%   5y revenue and EPS CAGR + TTM growth
Sentiment 10%   Analyst recommendation distribution
```

Sub-scores tier by color: green ≥70, gold ≥55, paper ≥40, mute below.

## Moat detection

Replaces the old hardcoded moat map with a quantitative proxy applied uniformly to every stock:

- **Wide moat:** gross margin >40% AND ROE >15% AND net margin >15%
- **Narrow moat:** any 2 of the 3
- **None detected:** 0 or 1 of the 3

This is an imperfect proxy. A capital-intensive utility with a true regulatory moat may not pass these thresholds. A high-margin software company without durable advantages might. The benefit is that the same rule applies to all 500+ stocks without manual tagging.

## Macro strip

Six tiles at the top show ETF proxies for major asset classes: SPY (S&P 500), QQQ (Nasdaq), IEF (10-year Treasury bond price as a rate proxy), UUP (US dollar), GLD (gold), USO (crude oil). The free tier does not give direct access to the Fed funds rate, VIX, or macro indices, so ETFs serve as the closest available signal.

## Troubleshooting

**"BAD API KEY"** · Your token is wrong or has been revoked. Open Settings and paste a fresh one from finnhub.io.

**"RATE LIMITED · WAIT 60s"** · You hit the 60 req/min cap. Wait a minute and click Score More.

**"INIT FAILED · UNIVERSE_FETCH"** · GitHub raw.githubusercontent.com was unreachable. Try again, or check your network. The constituents source is at `https://raw.githubusercontent.com/datasets/s-and-p-500-companies/main/data/constituents.csv`.

**Scan looks stuck** · Open browser console (F12). Each ticker logs its progress. If you see auth errors, your key may be exhausted.

## Files

- `index.html` · The entire dashboard. Single file, no build step, no dependencies beyond a Finnhub key.

## Disclaimer

This is not financial advice. The composite score is a deterministic ranking on observable metrics. It cannot account for regime shifts, fraud risk, regulatory changes, technology disruption, or qualitative factors. Verify every data point before any investment decision.
