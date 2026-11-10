# Following the Money behind Polymarket’s Iran War Bets

**A graph analysis of winning wallets on Polymarket’s 2026 US–Iran markets**

![Python](https://img.shields.io/badge/python-3.10%2B-blue)
![NetworkX](https://img.shields.io/badge/graphs-NetworkX-orange)

Graph Analytics course project — November 2026

---

## Overview

Public investigations reported that some Polymarket accounts may have profited from advance knowledge of the 2026 US–Iran conflict. This project asks a simple question: **do the biggest winners of Polymarket’s Iran markets behave like independent bettors, or like groups of linked wallets?**

Using only public data, the notebook:

1. finds the Iran-related markets that paid out between January and April 2026,
2. downloads every trade and computes the profit of every wallet,
3. reconstructs the money flows around the winners,
4. models these flows as a **directed, weighted graph** and analyses it with classic graph analytics (connectivity, distances, centrality, communities, robustness, Laplacian spectrum),
5. combines graph and betting signals into a transparent **suspicion score** and detects **clusters of linked winners**.

> ⚠️ **Disclaimer.** This project detects *atypical on-chain patterns*, not proof of insider trading. Wallet addresses are pseudonymous, holding several wallets is legitimate, and no address is linked to a real person. Results should be read as signals, never as accusations.

## Key results

| | |
|---|---|
| Markets scanned | 1,080 closed events, 2,233 Iran-related markets, 1,131 paid out between Jan and Apr 2026 |
| Markets analysed | The 5 largest, 397 M$ wagered |
| Trades / wallets | 926,658 trades by 82,596 wallets |
| Winners | 150 wallets with ≥ 1,000 $ profit, 16.3 M$ in total; the top 10 capture 19.7% of all profits |
| Money flows | 1,185 of 55,291 activity rows (about 2%) linked to a counterparty |
| Money-flow graph | 214 wallets, 326 directed edges (47 winners), density 0.0072, 38.4 M$ of transfers |
| Connectivity | 2 weak components, the largest holding 99.1% of wallets; 169 strong components, 6 of them money loops |
| Distances | diameter 7, radius 4, average shortest path 3.26 (largest component) |
| Local structure | average clustering 0.047, transitivity 0.029 |
| Spectrum | Fiedler value 0.093 (normalised Laplacian, largest component) |
| Communities | Louvain: 10 communities, modularity 0.59, stable over 10 seeds (mean NMI 0.96); label propagation: 15 communities, modularity 0.36; NMI between the two 0.56 |
| Bridges | 153 of 293 undirected edges disconnect the graph when removed |

**Main findings**

- The graph is sparse, heavy-tailed and modular, held together by a few hubs. The most central wallet, `0xc011…2dfb`, is in the top 10 of all five centrality measures but only receives money (68 incoming edges, none outgoing): an **exit point** such as an exchange or a bridge.
- Centrality and profit are **not linearly correlated** (Pearson not significant), but they are **moderately rank-correlated** (Spearman 0.21 to 0.42, all significant, on 140 wallets with a known profit): better-connected wallets tend to rank higher in profit, without big winners being the most central.
- **60 of 150 winners** were first funded less than a week before their first Iran bet.
- **15 winners score 4 or more** on the suspicion score and earned 4.3 M$, about 26% of all winners’ profit.
- **Two clusters of linked winners** cover 24 of the 47 winners in the graph:
  - a large, weak cluster of 20 winners (2.6 M$) held together by `0xd152…2150`, a wallet sending small amounts to many wallets (likely a service wallet);
  - a strong cluster of **4 winners earning 2.0 M$**, funded by a single wallet, `0xf70d…dbef`, on the same Khamenei markets, which Louvain isolates as its own community (13 wallets).
- No transfer to a related wallet was detected within 14 days after resolution.

## Data sources

All data is public and no API key is required with the default settings.

| Source | Used for |
|---|---|
| [Polymarket Gamma API](https://docs.polymarket.com) | Markets, outcomes, resolution times, volumes (keyset pagination on `/events/keyset`) |
| [Polymarket Data API v2](https://docs.polymarket.com) | Every trade (`/v2/trades`) and each wallet’s activity (deposits, withdrawals, transfers) |
| Public Polygon RPC nodes | Transaction receipts, to find the counterparty of each deposit or withdrawal |
| Etherscan V2 API *(optional)* | Full stablecoin transfer history (requires a free API key) |

> The Polymarket Data API v1 routes were retired in October 2026; the notebook uses the v2 endpoints.

## Repository structure

The repository contains the main Jupyter notebook and this README. When the notebook runs, it exports its results:

| Output | Content |
|---|---|
| `winners_ranked.csv` | Winners with their profit, signals and suspicion score |
| `winner_clusters.csv` | Clusters of linked winners |
| `redistribution_flows.csv` | Transfers sent by winners to related wallets after resolution |
| `communities.csv` | Louvain communities and their winners |
| `money_flow_graph.graphml` | The money-flow graph, readable by NetworkX or Gephi |
| `polymarket_iran_graph.graphml` | The cleaned graph, saved right after construction |
| `polymarket_cache_backup.zip` | A compressed copy of the download cache, to rerun the analysis without downloading again |

## Getting started

### Run on Google Colab (recommended)

1. Go to [Google Colab](https://colab.research.google.com), choose **File → Open notebook → GitHub**, paste the URL of this repository and select the notebook. You can also download the `.ipynb` file and upload it in Colab.
2. Run the cells from top to bottom.
3. The export cell saves results to `/content/drive/MyDrive/graph_results/`, so Colab will ask for access to your Google Drive. Change the paths in section 5.3 if you prefer to save locally.

### Run locally

Download or clone this repository (green **Code** button on GitHub), then install the dependencies and open the notebook:

```bash
pip install networkx pandas numpy scipy scikit-learn matplotlib requests jupyter
jupyter notebook
```

When running locally, remove or skip the Google Drive mount (`from google.colab import drive`) and update the export paths.

### Test without downloading anything

Set `DATA_MODE = "synthetic"` in the configuration cell. The notebook then generates a realistic fake dataset with **planted suspicious clusters** and runs the full analysis in a few seconds, which is useful to check that everything works and that the method finds what it should.

## Configuration

All parameters are in the configuration cell (section 0). The most important ones:

| Parameter | Default | Meaning |
|---|---|---|
| `DATA_MODE` | `"live"` | `"live"` (real data) or `"synthetic"` (test data) |
| `TAG_SLUGS` | `["iran", "middle-east", "geopolitics"]` | Polymarket tags to scan |
| `KEYWORDS` | `["iran", "khamenei", "tehran", "hormuz"]` | Keywords a market title must contain |
| `START_DATE`, `END_DATE` | `2026-01-01`, `2026-04-30` | Markets must have **paid out** within this window |
| `EVENT_SLUGS` | `[]` | Optional list of event slugs to skip the scan |
| `TOP_N_MARKETS` | `5` | Keep only the N markets with the largest volume |
| `MAX_TRADES_PER_MARKET` | `200,000` | Download cap per market |
| `MIN_TRADE_USDC` | `0` | Skip trades below this amount (faster download) |
| `MIN_PROFIT` | `1,000` | Minimum profit (USD) to count as a winner |
| `TOP_N_WINNERS` | `150` | Number of winners kept |
| `TRANSFER_SOURCE` | `"polymarket"` | `"polymarket"` (no key), `"etherscan"` (needs a key) or `"csv"` |
| `ACTIVITY_WALLETS` | `500` | Wallets whose activity is downloaded |
| `MAX_RECEIPTS` | `20,000` | Maximum number of transaction receipts checked on public Polygon nodes |
| `HUB_DEGREE_THRESHOLD` | `200` | Addresses with more counterparties are treated as infrastructure |
| `FRESH_WALLET_DAYS` | `7` | Threshold for a “fresh” wallet |
| `LONGSHOT_PRICE`, `LONGSHOT_MIN_STAKE` | `0.35`, `2,500` | Definition of a long-shot win |
| `FUNDING_HOPS` | `2` | Depth of the search for common funders |
| `REDISTRIBUTION_WINDOW_DAYS` | `14` | Window after resolution for redistribution |
| `SEED` | `42` | Random seed for reproducibility |

Downloaded data is **cached** in `polymarket_cache/` (markets once, trades market by market), so an interrupted run resumes where it stopped. Delete the cache folder to force a fresh download.

## Notebook structure

| Section | Content |
|---|---|
| **0. Setup** | Imports and configuration |
| **1. Data collection** | Market discovery, trades, wallet profits, money flows |
| **2. Graph construction** | Directed weighted graph, infrastructure filtering, undirected version |
| **3. Graph statistics** | Structure, degree, connectivity, distances, clustering, centrality (with top-k overlap, edge betweenness, correlation with profit), communities (Louvain vs label propagation, NMI, community graph), robustness, Laplacian spectrum |
| **4. Suspicious patterns** | Fresh wallets, long-shot wins, linked winners, redistribution of gains, suspicion score |
| **5. Results** | Cluster visualisation, summary table, export, conclusion |

### Methodological notes

- **Profit** = net cash flow (sales − purchases) + 1 $ per winning share held at resolution.
- The directed graph is **not strongly connected** (money mostly flows one way), so distances, eigenvector centrality and the Fiedler value are computed on the **undirected largest connected component**. Direction is kept for in/out-degrees, PageRank, strongly connected components and the search for funders.
- In the undirected graph, edges are weighted by `log(1 + amount)` to limit the influence of a few very large transfers.

## Reusing the pipeline on another topic

The pipeline is not specific to Iran. To study another market, change:

```python
TAG_SLUGS = ["elections"]
KEYWORDS = ["president", "election"]
START_DATE, END_DATE = "2026-01-01", "2026-06-30"
TOP_N_MARKETS = 5
```

and rerun the notebook. Any market where some participants may know the outcome in advance is a good candidate: elections, central-bank decisions, sports transfers, company earnings, award ceremonies or product launches.

## Limitations

- Only **1,185 of 55,291** activity rows (about 2%) could be linked to a counterparty: at most 20,000 transaction receipts are checked (`MAX_RECEIPTS`), and most deposits and withdrawals go through exchanges or bridges, which hide the real sender. The detected links are therefore a lower bound.
- Three of the five markets reached the 200,000-trade cap; the API returns the most recent trades first, so their oldest trades are missing.
- With `HUB_DEGREE_THRESHOLD = 200`, some exchanges or bridges remain in the graph and can create spurious links on a graph of this size.
- API fields and endpoints may change; check the [Polymarket documentation](https://docs.polymarket.com) if a request fails.

## Legal notice

Prediction markets such as Polymarket are not authorised in several countries, including France. This repository only **reads public data** for research and educational purposes; it does not place bets and does not encourage anyone to use the platform.

## Acknowledgements

Parts of the centrality and community detection code are adapted from the course practicals (metro network centrality and community detection).
